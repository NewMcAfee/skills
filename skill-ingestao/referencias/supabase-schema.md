# Schema Supabase — Dante (Growth IA OS)
**Versão:** 2.0  
**Status:** Documento de referência para construção do banco  
**Quem usa este documento:** Claude (via skill ou Claude Code) ao criar ou migrar o banco de dados

---

## Instruções para o Claude que vai construir isso

Este documento é o contrato do banco de dados. Antes de criar qualquer tabela:

1. Leia o documento inteiro antes de escrever uma linha de SQL
2. Respeite a ordem de criação definida na seção "Ordem de criação"
3. Toda FK deve ter `ON DELETE RESTRICT` salvo indicação contrária — nunca deletar dados históricos
4. Toda tabela tem `created_at TIMESTAMPTZ DEFAULT NOW()` e `updated_at TIMESTAMPTZ DEFAULT NOW()`, exceto tabelas explicitamente marcadas como imutáveis
5. Crie um trigger `set_updated_at` para atualizar `updated_at` automaticamente em todas as tabelas que o possuem
6. Ative Row Level Security (RLS) em todas as tabelas — política padrão: usuário autenticado vê apenas projetos onde `project_id` pertence à sua organização
7. O campo `version_id` é a espinha dorsal do versionamento — toda tabela que carrega conteúdo estratégico do projeto referencia `versions.id`

---

## Princípios de design

**Documentos vivos, histórico imutável.** Os 9 documentos do planejamento são revisados a cada 3–6 meses. O banco não sobrescreve — cria uma nova versão. Testes, campanhas e assets referenciam a versão vigente na época em que foram criados.

**ICP ≠ segmentação de público.** O ICP define quem é o cliente ideal — estratégia. O público do conjunto de anúncios define como a plataforma vai encontrar pessoas — tática. Um mesmo ICP pode ser alcançado via LAL, interesse, retargeting ou broad.

**Múltiplos produtos, múltiplos ICPs, combinações explícitas.** Um projeto pode ter N produtos com PUVs distintas e M ICPs ativos. A tabela `icp_product_map` registra quais combinações são estratégicas e qual é a prioridade de cada uma. Campanhas e testes referenciam tanto o ICP quanto o produto — nunca apenas um dos dois.

**Dados brutos no banco, derivados calculados on-demand.** A tabela `performance_metrics` armazena apenas dados brutos com granularidade de tempo + nível hierárquico (campanha / conjunto / anúncio). Taxas de conversão, custos unitários e velocidade de funil são calculados pelo Data Miner na hora da análise — nunca persistidos, exceto como snapshot no momento de conclusão de um teste.

**Dimensionalidade obrigatória nas métricas.** Todo registro de performance carrega: período (data de início e fim), granularidade (diária/semanal/mensal/trimestral), e nível hierárquico (campanha/conjunto/anúncio). Sem essas três dimensões, nenhuma análise temporal ou comparativa é possível.

**Campos no banco = campos que algum componente lê programaticamente.** Conteúdo narrativo fica nos documentos do Cowork.

---

## Enums globais

```sql
CREATE TYPE canal_type AS ENUM ('META', 'GOOG', 'LINK', 'TIKT');
CREATE TYPE status_campanha AS ENUM ('HALL', 'TEST', 'EVER');
CREATE TYPE objetivo_type AS ENUM ('LEAD', 'SALES', 'TRFC', 'AWAR', 'MSNG');
CREATE TYPE destino_type AS ENUM ('FormNativo', 'WhatsApp', 'LandingPage', 'Site', 'Ecom', 'App');
CREATE TYPE funil_type AS ENUM ('COLD', 'WARM', 'HOT');
CREATE TYPE audience_type AS ENUM ('LAL', 'INT', 'RTG', 'CRM', 'BROAD');
CREATE TYPE formato_type AS ENUM ('STA', 'VID', 'CAR', 'MOT', 'MIX');
CREATE TYPE consciencia_type AS ENUM ('UNC', 'PRB', 'SOL', 'PRO', 'MIX');
CREATE TYPE gancho_type AS ENUM ('Loss', 'Proof', 'Story', 'Error', 'Fact', 'Contr');
CREATE TYPE test_status AS ENUM ('draft', 'active', 'reading', 'validated', 'inconclusive', 'archived');
CREATE TYPE icp_status AS ENUM ('active', 'testing', 'archived');
CREATE TYPE gender_type AS ENUM ('M', 'F', 'all');
CREATE TYPE awareness_level AS ENUM ('UNC', 'PRB', 'SOL', 'PRO', 'MIX');
CREATE TYPE metric_level AS ENUM ('campaign', 'ad_set', 'ad');
CREATE TYPE metric_granularity AS ENUM ('daily', 'weekly', 'monthly', 'quarterly', 'custom');
CREATE TYPE tracking_completeness AS ENUM ('platform_only', 'platform_crm');
-- platform_only: dados disponíveis até lead/MQL (tracking não integrado com CRM)
-- platform_crm:  funil completo disponível (tracking integrado com CRM)
```

---

## Tabelas

### 1. organizations
Agrupa projetos por agência ou operação própria. Ponto de âncora do RLS.

```sql
CREATE TABLE organizations (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name          TEXT NOT NULL,
  slug          TEXT UNIQUE NOT NULL,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 2. projects
Um projeto = um cliente ou uma operação. Ponto de entrada de todo o sistema.

```sql
CREATE TABLE projects (
  id                 UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id             UUID NOT NULL REFERENCES organizations(id) ON DELETE RESTRICT,
  name               TEXT NOT NULL,
  slug               TEXT NOT NULL,
  status             TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'paused', 'archived')),
  current_version_id UUID,              -- FK circular adicionada após criar versions
  created_at         TIMESTAMPTZ DEFAULT NOW(),
  updated_at         TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (org_id, slug)
);
```

---

### 3. versions
Cada revisão do planejamento estratégico gera uma nova versão. Imutável após `sealed_at`.

```sql
CREATE TABLE versions (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id       UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_number   INTEGER NOT NULL,       -- 1, 2, 3… incrementado automaticamente
  label            TEXT,                   -- ex: "Q1 2025", "Revisão pós-sazonalidade"
  valid_from       DATE NOT NULL,
  valid_until      DATE,                   -- NULL = versão vigente atual
  sealed_at        TIMESTAMPTZ,            -- travada: não pode mais ser editada
  revision_summary TEXT,                   -- o que mudou em relação à versão anterior
  created_by       TEXT,                   -- operador ou 'skill:revisao-estrategica'
  created_at       TIMESTAMPTZ DEFAULT NOW(),
  updated_at       TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (project_id, version_number)
);

-- Fechar loop circular
ALTER TABLE projects
  ADD CONSTRAINT fk_current_version
  FOREIGN KEY (current_version_id) REFERENCES versions(id) ON DELETE SET NULL;
```

---

### 4. planning_data
Campos estruturados extraídos dos 9 documentos. Uma linha por versão por projeto.
Não substitui os documentos — complementa com dados consultáveis.
Métricas-alvo globais ficam aqui. Métricas por produto ficam em `products`.

```sql
CREATE TABLE planning_data (
  id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id                  UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id                  UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,

  -- Do Briefing do Projeto (doc 1)
  segment                     TEXT,         -- ex: "Construtoras médias", "SaaS B2B"
  sales_cycle                 TEXT,         -- ex: "30-90 dias", "imediato"

  -- Da Modelagem Financeira (doc 7) — nível projeto
  monthly_budget_total        NUMERIC(12,2),
  break_even_leads            INTEGER,

  -- Do Planejamento de Mídia (doc 8)
  active_channels             canal_type[],
  budget_meta                 NUMERIC(12,2),
  budget_goog                 NUMERIC(12,2),

  -- Do Go-to-Market Plan (doc 9)
  gtm_priority                TEXT,
  gtm_north_star              TEXT,

  -- Critérios de validação estatística (hardcoded no Runbook, refletido aqui)
  min_impressions_for_reading INTEGER DEFAULT 5000,
  min_leads_for_reading       INTEGER DEFAULT 30,
  min_days_for_reading        INTEGER DEFAULT 14,
  confidence_threshold        NUMERIC(4,2) DEFAULT 0.80,

  -- Configuração de tracking
  tracking_completeness       tracking_completeness NOT NULL DEFAULT 'platform_only',
  crm_integrated              BOOLEAN DEFAULT FALSE,

  created_at                  TIMESTAMPTZ DEFAULT NOW(),
  updated_at                  TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (project_id, version_id)
);
```

---

### 5. products
Um projeto pode ter múltiplos produtos com PUVs distintas.
Cada produto é versionado — uma revisão pode alterar a PUV, arquivar um produto ou lançar um novo.

```sql
CREATE TABLE products (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id          UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id          UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,
  product_ref         TEXT NOT NULL,        -- ex: "PROD-1", "PROD-2" — aparece nos briefings
  name                TEXT NOT NULL,        -- ex: "BootcampVendas", "Consultoria Enterprise"
  status              TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'archived')),

  -- Da PUV (doc 5) — por produto
  puv_statement       TEXT,                 -- proposta única de valor ≤ 280 chars
  main_differentiator TEXT,                 -- diferencial central em uma frase

  -- Métricas-alvo por produto (sobrescreve planning_data se preenchido)
  price_point         NUMERIC(12,2),        -- ticket médio ou preço
  target_cpl          NUMERIC(10,2),
  target_cpa          NUMERIC(10,2),
  target_roas         NUMERIC(6,2),

  -- Histórico
  archived_at         TIMESTAMPTZ,
  revision_notes      TEXT,

  created_at          TIMESTAMPTZ DEFAULT NOW(),
  updated_at          TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 6. icps
Perfis de cliente ideal. Múltiplos ICPs por projeto e por versão.
ICP ≠ segmentação de público — ver princípios de design.

```sql
CREATE TABLE icps (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id       UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id       UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,
  icp_ref          TEXT NOT NULL,     -- ex: "ICP-A", "ICP-B2"
  name             TEXT NOT NULL,     -- ex: "Construtora Média SP"
  status           icp_status NOT NULL DEFAULT 'active',

  -- Perfil estratégico
  awareness_level  awareness_level NOT NULL,
  primary_pain     TEXT NOT NULL,     -- ≤ 120 chars
  primary_desire   TEXT NOT NULL,     -- ≤ 120 chars
  main_objection   TEXT,              -- ≤ 120 chars
  buying_trigger   TEXT,

  -- Perfil demográfico (contexto para Media Buyer e Arquiteto)
  age_min          INTEGER,
  age_max          INTEGER,
  gender           gender_type DEFAULT 'all',
  geo              TEXT[],            -- ex: ARRAY['BR-SP', 'BR-RJ']
  income_range     TEXT,

  -- Métricas-alvo específicas deste ICP
  target_cpl       NUMERIC(10,2),
  target_cpa       NUMERIC(10,2),
  min_sample_leads INTEGER DEFAULT 30,

  -- Histórico
  archived_at      TIMESTAMPTZ,
  revision_notes   TEXT,

  created_at       TIMESTAMPTZ DEFAULT NOW(),
  updated_at       TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 7. icp_product_map
Combinações estratégicas de ICP + Produto. Define quais pares são ativos,
qual é a prioridade e qual é a lógica estratégica de cada combinação.
Campanhas e testes referenciam tanto `icp_id` quanto `product_id` via esta tabela.

```sql
CREATE TABLE icp_product_map (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id      UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id      UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,
  icp_id          UUID NOT NULL REFERENCES icps(id) ON DELETE RESTRICT,
  product_id      UUID NOT NULL REFERENCES products(id) ON DELETE RESTRICT,

  priority        INTEGER NOT NULL DEFAULT 1,   -- 1 = maior prioridade
  status          TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'testing', 'archived')),

  -- Contexto estratégico da combinação
  strategic_notes TEXT,   -- ex: "ICP-A converte melhor no PROD-1, testar PROD-2 em Q2"

  -- Métricas-alvo específicas desta combinação (sobrescreve icp e product se preenchido)
  target_cpl      NUMERIC(10,2),
  target_cpa      NUMERIC(10,2),
  target_roas     NUMERIC(6,2),

  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (icp_id, product_id, version_id)
);
```

---

### 8. campaigns
Nível estratégico. Nome gerado pelo Guardião do Log, validado pelo Validator.

```sql
CREATE TABLE campaigns (
  id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id     UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id     UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,
  product_id     UUID REFERENCES products(id),   -- qual produto esta campanha promove

  -- Taxonomia: STATUS_CANAL_OBJETIVO_PRODUTO_DESTINO
  status_camp    status_campanha NOT NULL,
  canal          canal_type NOT NULL,
  objetivo       objetivo_type NOT NULL,
  produto        TEXT NOT NULL,                  -- CamelCase, sem espaços — ex: "BootcampVendas"
  destino        destino_type NOT NULL,

  campaign_name  TEXT UNIQUE NOT NULL,           -- nome gerado e validado
  platform_id    TEXT,                           -- ID na plataforma (Meta ou Google)

  launched_at    DATE,
  paused_at      DATE,
  notes          TEXT,

  created_at     TIMESTAMPTZ DEFAULT NOW(),
  updated_at     TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 9. ad_sets
Nível de público. Hierarquia de exclusão implementada pelo script de exclusão.

```sql
CREATE TABLE ad_sets (
  id                        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id               UUID NOT NULL REFERENCES campaigns(id) ON DELETE RESTRICT,
  project_id                UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id                UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,

  -- Taxonomia: AUD-HIERARQUIA-ID_FUNIL_TIPO_PUBLICO_GEO_DEMO_FORMATO
  hierarchy_digit           INTEGER NOT NULL CHECK (hierarchy_digit BETWEEN 0 AND 9),
  seq_id                    TEXT NOT NULL,       -- ex: "001"
  funil                     funil_type NOT NULL,
  audience_type             audience_type NOT NULL,
  audience_name             TEXT NOT NULL,
  geo                       TEXT NOT NULL,
  demo                      TEXT NOT NULL,
  formato                   formato_type NOT NULL,

  ad_set_name               TEXT NOT NULL,
  excluded_hierarchy_digits INTEGER[],           -- preenchido pelo script de exclusão

  platform_id               TEXT,
  launched_at               DATE,
  paused_at                 DATE,
  daily_budget              NUMERIC(10,2),

  created_at                TIMESTAMPTZ DEFAULT NOW(),
  updated_at                TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 10. ads
Nível criativo. Referencia ICP e produto via `icp_product_map`.

```sql
CREATE TABLE ads (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ad_set_id       UUID NOT NULL REFERENCES ad_sets(id) ON DELETE RESTRICT,
  project_id      UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id      UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,
  icp_product_id  UUID REFERENCES icp_product_map(id),  -- combinação ICP+Produto deste anúncio
  asset_id        UUID,                                  -- FK → assets.id (adicionada após criar assets)

  -- Taxonomia: AD-ID_FORMATO_CONSCIENCIA_GANCHO_AVATAR_VARIACAO
  seq_id          TEXT NOT NULL,
  formato         formato_type NOT NULL,
  consciencia     consciencia_type NOT NULL,
  gancho          gancho_type NOT NULL,
  avatar          TEXT NOT NULL,       -- CamelCase — ex: "CamilaFarani"
  variacao        TEXT NOT NULL,       -- CamelCase — ex: "PerdendoDinheiro"

  ad_name         TEXT NOT NULL,

  -- Copy
  headline        TEXT,
  primary_text    TEXT,
  cta_type        TEXT,               -- ex: "CONS", "DIR"

  platform_id     TEXT,
  launched_at     DATE,
  paused_at       DATE,

  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 11. performance_metrics
**Coração do Data Miner.** Armazena dados brutos de performance com três dimensões obrigatórias:
período + granularidade + nível hierárquico.

O Data Miner popula esta tabela. O Growth Lead lê desta tabela.
Derivados (taxas, custos unitários, velocidade) são calculados on-demand — nunca armazenados aqui.
Exceção: campos `snapshot_*` em `tests`, que preservam derivados no momento de conclusão de um teste.

```sql
CREATE TABLE performance_metrics (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id      UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id      UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,

  -- DIMENSÃO 1: Período
  period_start    DATE NOT NULL,
  period_end      DATE NOT NULL,
  granularity     metric_granularity NOT NULL,   -- daily | weekly | monthly | quarterly | custom

  -- DIMENSÃO 2: Nível hierárquico
  level           metric_level NOT NULL,         -- campaign | ad_set | ad
  campaign_id     UUID REFERENCES campaigns(id),
  ad_set_id       UUID REFERENCES ad_sets(id),
  ad_id           UUID REFERENCES ads(id),
  -- Regra: preencher apenas o ID correspondente ao level.
  -- level=campaign → campaign_id preenchido, ad_set_id e ad_id nulos.
  -- level=ad_set   → campaign_id e ad_set_id preenchidos, ad_id nulo.
  -- level=ad       → todos os três preenchidos.

  -- DIMENSÃO 3: Origem dos dados
  tracking        tracking_completeness NOT NULL,  -- platform_only | platform_crm
  canal           canal_type NOT NULL,

  -- DADOS BRUTOS — Topo de funil (sempre disponíveis)
  investment      NUMERIC(12,2),      -- investimento no período
  reach           INTEGER,            -- alcance
  impressions     INTEGER,
  clicks          INTEGER,

  -- DADOS BRUTOS — Meio de funil (disponíveis quando tracking ok)
  leads           INTEGER,
  mqls            INTEGER,            -- leads qualificados pelo marketing
  sqls            INTEGER,            -- leads qualificados por vendas

  -- DADOS BRUTOS — Fundo de funil (disponíveis quando CRM integrado)
  negotiations    INTEGER,            -- em negociação
  won             INTEGER,            -- ganho/fechado
  lost            INTEGER,            -- perdido
  revenue         NUMERIC(12,2),      -- valor gerado no período

  -- Campos nulos quando tracking_completeness = 'platform_only':
  -- mqls, sqls, negotiations, won, lost, revenue

  -- Metadados de importação
  imported_at     TIMESTAMPTZ DEFAULT NOW(),
  import_source   TEXT,               -- ex: "meta_ads_export_2025-01-15", "crm_export_2025-01-15"
  import_notes    TEXT,               -- observações sobre a qualidade dos dados

  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),

  -- Evitar duplicatas: um registro por período + nível + entidade
  UNIQUE (period_start, period_end, granularity, level, campaign_id, ad_set_id, ad_id)
);
```

---

### 12. tests
Experimentos formais. Referencia a combinação ICP+Produto via `icp_product_map`.
Resultados brutos são lidos de `performance_metrics`.
Snapshots de derivados preservam o estado no momento da conclusão.

```sql
CREATE TABLE tests (
  id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id           UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id           UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,
  icp_product_id       UUID REFERENCES icp_product_map(id),  -- ICP+Produto deste teste
  campaign_id          UUID REFERENCES campaigns(id),

  -- Hipótese
  hypothesis           TEXT NOT NULL,        -- "Se X, então Y porque Z"
  variable_tested      TEXT NOT NULL,        -- o que está sendo testado
  control_ad_id        UUID REFERENCES ads(id),
  variant_ad_id        UUID REFERENCES ads(id),

  -- Critérios de validação (copiados de planning_data/icp_product_map no momento da criação)
  metric_primary       TEXT NOT NULL,        -- ex: "CPL", "CTR", "ROAS", "lead>sql"
  validation_target    NUMERIC(10,2),
  min_impressions      INTEGER NOT NULL,
  min_leads            INTEGER NOT NULL,
  min_days             INTEGER NOT NULL,
  confidence_threshold NUMERIC(4,2) NOT NULL,

  -- Status
  status               test_status NOT NULL DEFAULT 'draft',
  started_at           DATE,
  ended_at             DATE,

  -- Período de leitura (referencia performance_metrics)
  metrics_period_start DATE,
  metrics_period_end   DATE,

  -- SNAPSHOTS de derivados no momento de conclusão
  -- (calculados pelo Data Miner, persistidos aqui como registro histórico)
  snapshot_investment   NUMERIC(12,2),
  snapshot_impressions  INTEGER,
  snapshot_clicks       INTEGER,
  snapshot_leads        INTEGER,
  snapshot_mqls         INTEGER,
  snapshot_sqls         INTEGER,
  snapshot_negotiations INTEGER,
  snapshot_won          INTEGER,
  snapshot_lost         INTEGER,
  snapshot_revenue      NUMERIC(12,2),
  -- Taxas calculadas no momento da conclusão
  snapshot_ctr          NUMERIC(8,4),        -- cliques / impressões
  snapshot_cpl          NUMERIC(10,2),       -- investimento / leads
  snapshot_cpmql        NUMERIC(10,2),       -- investimento / MQLs
  snapshot_cpsql        NUMERIC(10,2),       -- investimento / SQLs
  snapshot_cac          NUMERIC(10,2),       -- investimento / won
  snapshot_roas         NUMERIC(8,4),        -- revenue / investimento
  snapshot_lead_to_mql  NUMERIC(6,4),        -- mqls / leads
  snapshot_lead_to_sql  NUMERIC(6,4),        -- sqls / leads
  snapshot_lead_to_won  NUMERIC(6,4),        -- won / leads
  snapshot_sql_to_won   NUMERIC(6,4),        -- won / sqls
  -- Velocidade em dias (calculada a partir do CRM quando disponível)
  snapshot_days_lead_to_won  NUMERIC(8,2),
  snapshot_days_lead_to_sql  NUMERIC(8,2),
  snapshot_days_neg_to_won   NUMERIC(8,2),

  -- Conclusão
  conclusion           TEXT,
  next_hypothesis      TEXT,
  validated_at         TIMESTAMPTZ,

  created_at           TIMESTAMPTZ DEFAULT NOW(),
  updated_at           TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 13. assets
Assets físicos do sistema de design. Ponte entre growth e design.

```sql
CREATE TABLE assets (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id      UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id      UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,
  ad_id           UUID REFERENCES ads(id),

  asset_code      TEXT NOT NULL,         -- AD-ID completo da taxonomia
  formato         formato_type NOT NULL,

  briefing_source TEXT,                  -- "arquiteto-de-testes" | "growth-lead" | "manual"
  briefing_ref    UUID REFERENCES tests(id),

  storage_url     TEXT,
  figma_node_id   TEXT,
  dimensions      TEXT,                  -- ex: "1080x1080"
  platform        canal_type,

  status          TEXT DEFAULT 'draft' CHECK (status IN ('draft', 'approved', 'active', 'archived')),
  approved_at     TIMESTAMPTZ,
  approved_by     TEXT,

  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Fechar loop: ads.asset_id → assets.id
ALTER TABLE ads
  ADD CONSTRAINT fk_asset
  FOREIGN KEY (asset_id) REFERENCES assets(id) ON DELETE SET NULL;
```

---

### 14. revision_events
Log imutável de cada ciclo de revisão. Sem `updated_at` — nunca editado.

```sql
CREATE TABLE revision_events (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id            UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  from_version_id       UUID REFERENCES versions(id),
  to_version_id         UUID NOT NULL REFERENCES versions(id),

  changed_fields        TEXT[],
  icps_added            TEXT[],
  icps_archived         TEXT[],
  products_added        TEXT[],
  products_archived     TEXT[],
  combinations_added    TEXT[],          -- ex: ARRAY['ICP-A+PROD-2']
  combinations_archived TEXT[],
  budget_delta          NUMERIC(12,2),

  revision_rationale    TEXT NOT NULL,
  key_learnings         TEXT,
  conducted_by          TEXT,
  conducted_with_client BOOLEAN DEFAULT FALSE,

  created_at            TIMESTAMPTZ DEFAULT NOW()
);
```

---

## Índices recomendados

```sql
-- Hierarquia principal
CREATE INDEX idx_campaigns_project_version ON campaigns(project_id, version_id);
CREATE INDEX idx_campaigns_product ON campaigns(product_id);
CREATE INDEX idx_ad_sets_campaign ON ad_sets(campaign_id);
CREATE INDEX idx_ads_ad_set ON ads(ad_set_id);
CREATE INDEX idx_ads_icp_product ON ads(icp_product_id);

-- Tests
CREATE INDEX idx_tests_project_version ON tests(project_id, version_id);
CREATE INDEX idx_tests_icp_product ON tests(icp_product_id);
CREATE INDEX idx_tests_status ON tests(status);

-- ICPs e produtos
CREATE INDEX idx_icps_project_version ON icps(project_id, version_id);
CREATE INDEX idx_icps_status ON icps(project_id, status);
CREATE INDEX idx_products_project_version ON products(project_id, version_id);
CREATE INDEX idx_icp_product_map_version ON icp_product_map(project_id, version_id);

-- Performance metrics — as queries mais pesadas do sistema
CREATE INDEX idx_metrics_project_period ON performance_metrics(project_id, period_start, period_end);
CREATE INDEX idx_metrics_level ON performance_metrics(level, campaign_id, ad_set_id, ad_id);
CREATE INDEX idx_metrics_granularity ON performance_metrics(project_id, granularity);
CREATE INDEX idx_metrics_canal ON performance_metrics(project_id, canal);

-- Assets e revisões
CREATE INDEX idx_assets_ad ON assets(ad_id);
CREATE INDEX idx_revision_events_project ON revision_events(project_id);
```

---

## Trigger: updated_at automático

```sql
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DO $$
DECLARE t TEXT;
BEGIN
  FOREACH t IN ARRAY ARRAY[
    'organizations', 'projects', 'versions', 'planning_data',
    'products', 'icps', 'icp_product_map',
    'campaigns', 'ad_sets', 'ads',
    'tests', 'assets', 'performance_metrics'
  ]
  LOOP
    EXECUTE format(
      'CREATE TRIGGER trg_updated_at BEFORE UPDATE ON %I
       FOR EACH ROW EXECUTE FUNCTION set_updated_at()', t
    );
  END LOOP;
END $$;
```

---

## Ordem de criação das tabelas

```
1.  Enums globais
2.  organizations
3.  projects                          (sem current_version_id ainda)
4.  versions
5.  ALTER TABLE projects ADD CONSTRAINT fk_current_version
6.  planning_data
7.  products
8.  icps
9.  icp_product_map
10. campaigns
11. ad_sets
12. ads                               (sem asset_id ainda)
13. performance_metrics
14. tests
15. assets
16. ALTER TABLE ads ADD CONSTRAINT fk_asset
17. revision_events
18. Índices
19. Trigger set_updated_at
```

---

## Queries de referência para as skills

### Growth Lead: contexto completo da versão vigente
```sql
SELECT
  p.name AS project_name,
  v.version_number, v.label, v.valid_from,
  pd.gtm_north_star, pd.monthly_budget_total,
  pd.min_leads_for_reading, pd.confidence_threshold,
  pd.tracking_completeness,
  json_agg(DISTINCT jsonb_build_object(
    'icp_ref',          i.icp_ref,
    'icp_name',         i.name,
    'awareness_level',  i.awareness_level,
    'primary_pain',     i.primary_pain,
    'primary_desire',   i.primary_desire,
    'main_objection',   i.main_objection
  )) AS icps,
  json_agg(DISTINCT jsonb_build_object(
    'product_ref',      pr.product_ref,
    'product_name',     pr.name,
    'puv_statement',    pr.puv_statement,
    'target_cpl',       COALESCE(pr.target_cpl, pd.target_cpl),
    'target_roas',      COALESCE(pr.target_roas, pd.target_roas)
  )) AS products,
  json_agg(DISTINCT jsonb_build_object(
    'icp_ref',          i.icp_ref,
    'product_ref',      pr.product_ref,
    'priority',         ipm.priority,
    'target_cpl',       COALESCE(ipm.target_cpl, i.target_cpl, pr.target_cpl),
    'strategic_notes',  ipm.strategic_notes
  )) AS combinations
FROM projects p
JOIN versions v ON v.id = p.current_version_id
JOIN planning_data pd ON pd.version_id = v.id AND pd.project_id = p.id
JOIN icps i ON i.version_id = v.id AND i.project_id = p.id AND i.status = 'active'
JOIN products pr ON pr.version_id = v.id AND pr.project_id = p.id AND pr.status = 'active'
JOIN icp_product_map ipm ON ipm.icp_id = i.id AND ipm.product_id = pr.id AND ipm.status = 'active'
WHERE p.id = :project_id
GROUP BY p.name, v.version_number, v.label, v.valid_from,
         pd.gtm_north_star, pd.monthly_budget_total,
         pd.min_leads_for_reading, pd.confidence_threshold, pd.tracking_completeness;
```

### Data Miner: métricas brutas por nível e período
```sql
-- Exemplo: performance semanal de todos os conjuntos de uma campanha
SELECT
  pm.period_start, pm.period_end,
  ads.ad_set_name,
  pm.investment, pm.impressions, pm.clicks,
  pm.leads, pm.mqls, pm.sqls,
  pm.negotiations, pm.won, pm.lost, pm.revenue,
  pm.tracking
FROM performance_metrics pm
JOIN ad_sets ads ON ads.id = pm.ad_set_id
WHERE pm.campaign_id = :campaign_id
  AND pm.granularity = 'weekly'
  AND pm.level = 'ad_set'
ORDER BY pm.period_start, ads.ad_set_name;
```

### Arquiteto de Testes: combinações ICP+Produto sem testes suficientes
```sql
SELECT
  ipm.id AS icp_product_id,
  i.icp_ref, i.name AS icp_name, i.awareness_level, i.primary_pain,
  pr.product_ref, pr.name AS product_name, pr.puv_statement,
  ipm.priority, ipm.strategic_notes,
  COALESCE(ipm.target_cpl, i.target_cpl, pr.target_cpl) AS target_cpl,
  COUNT(t.id)                                              AS total_tests,
  COUNT(t.id) FILTER (WHERE t.status = 'validated')        AS validated_tests,
  COUNT(t.id) FILTER (WHERE t.status = 'active')           AS active_tests
FROM icp_product_map ipm
JOIN icps i     ON i.id = ipm.icp_id
JOIN products pr ON pr.id = ipm.product_id
LEFT JOIN tests t ON t.icp_product_id = ipm.id
WHERE ipm.project_id = :project_id
  AND ipm.version_id = :version_id
  AND ipm.status IN ('active', 'testing')
GROUP BY ipm.id, i.icp_ref, i.name, i.awareness_level, i.primary_pain,
         pr.product_ref, pr.name, pr.puv_statement, ipm.priority,
         ipm.strategic_notes, ipm.target_cpl, i.target_cpl, pr.target_cpl
ORDER BY ipm.priority, validated_tests ASC;
```

### Revisão estratégica: comparar combinações entre versões
```sql
SELECT
  i.icp_ref, i.name AS icp_name,
  pr.product_ref, pr.name AS product_name,
  ipm.status, ipm.priority,
  ipm.target_cpl,
  v.version_number, v.label,
  COUNT(t.id) FILTER (WHERE t.status = 'validated') AS validated_tests
FROM icp_product_map ipm
JOIN icps i     ON i.id = ipm.icp_id
JOIN products pr ON pr.id = ipm.product_id
JOIN versions v  ON v.id = ipm.version_id
LEFT JOIN tests t ON t.icp_product_id = ipm.id AND t.status = 'validated'
WHERE ipm.project_id = :project_id
GROUP BY i.icp_ref, i.name, pr.product_ref, pr.name,
         ipm.status, ipm.priority, ipm.target_cpl, v.version_number, v.label
ORDER BY v.version_number DESC, ipm.priority;
```

### Data Miner: análise de safra (cohort de leads por semana de entrada)
```sql
-- Agrupa leads por semana de criação e acompanha progressão no funil
SELECT
  DATE_TRUNC('week', pm.period_start) AS cohort_week,
  SUM(pm.leads)           AS leads_entrada,
  SUM(pm.mqls)            AS mqls,
  SUM(pm.sqls)            AS sqls,
  SUM(pm.won)             AS ganhos,
  SUM(pm.revenue)         AS receita,
  pm.canal
FROM performance_metrics pm
WHERE pm.project_id = :project_id
  AND pm.level = 'campaign'
  AND pm.granularity = 'weekly'
  AND pm.period_start BETWEEN :start_date AND :end_date
GROUP BY DATE_TRUNC('week', pm.period_start), pm.canal
ORDER BY cohort_week;
```

---

*Schema versão 2.0 — Dante (Growth IA OS)*
*Alterações em relação à v1.0:*
*— Adicionadas tabelas: `products`, `icp_product_map`, `performance_metrics`*
*— `tests` migrado de campos brutos para snapshots de derivados + referência a `performance_metrics`*
*— `ads` e `tests` passaram a referenciar `icp_product_map` em vez de `icps` diretamente*
*— `planning_data` removeu campos de PUV (movidos para `products`)*
*— `revision_events` adicionou campos de produtos e combinações*
*— Adicionados enums: `metric_level`, `metric_granularity`, `tracking_completeness`*
*— Próximo passo: executar este schema no Supabase e construir o Validator script*
