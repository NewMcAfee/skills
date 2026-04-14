# Dante — Mapa de Schema para Data Engineer

## Estado Atual: 5 Migrations Aplicadas

```
001 (supabase-schema.sql)     — schema base: 15 tabelas, 13 ENUMs, 35 índices, RLS
002 (performance_metrics_crm) — ADD COLUMN attribution_model + UNIQUE com tracking
003 (performance_metrics_cohort) — ADD COLUMNS cohort + velocidade + índice parcial
004 (governanca_ia)           — 4 tabelas novas, 8 ENUMs, JSONB em governance_events
005 (design_ia)               — 3 tabelas novas, 4 ENUMs
fn_data_analysis.sql          — RPC: fn_data_analysis() agrega por icp_product_map
```

**Próxima migration: `006_*.sql`**

---

## Hierarquia de Tabelas

```
organizations
  └─ projects (org_id FK)
       ├─ versions (project_id FK)
       │    └─ planning_data (project_id, version_id FK)
       ├─ products (project_id, version_id FK)
       ├─ icps (project_id, version_id FK)
       ├─ icp_product_map (project_id, version_id, icp_id FK, product_id FK)
       │    └─ [eixo central de toda análise]
       ├─ campaigns (project_id, version_id, product_id FK)
       │    └─ ad_sets (campaign_id FK, project_id, version_id FK)
       │         └─ ads (ad_set_id FK, project_id, version_id, icp_product_id FK, asset_id FK)
       ├─ performance_metrics (project_id, version_id, campaign_id FK, ad_set_id FK, ad_id FK)
       ├─ tests (project_id, version_id, icp_product_id FK, campaign_id FK)
       ├─ assets (project_id, version_id, ad_id FK)
       ├─ revision_events (project_id)
       ├─ account_health [004] (project_id FK)
       ├─ truths_and_traps [004] (project_id FK)
       ├─ execution_backlog [004] (project_id FK, parent_id self-ref FK)
       ├─ governance_events [004] (project_id FK, target_workflow ENUM)
       ├─ brand_audits [005] (project_id FK)
       ├─ production_queue [005] (project_id FK, test_ref FK → tests)
       └─ asset_versions [005] (project_id FK, asset_id FK)
```

---

## `performance_metrics` — Tabela Central de Dados

**Princípio:** apenas métricas brutas. Derivados (CTR, CPL, ROAS) calculados on-demand.

### Dimensões

| Dimensão | Campos | Valores |
|----------|--------|---------|
| Período | period_start (DATE), period_end (DATE), granularity | daily, weekly, monthly, quarterly, custom |
| Nível | level, campaign_id, ad_set_id, ad_id | campaign, ad_set, ad |
| Origem | tracking, canal | platform_only, platform_crm, cohort; META, GOOG, etc |

### Métricas brutas

| Funil | Campos |
|-------|--------|
| Topo | investment (NUMERIC), reach (INT), impressions (INT), clicks (INT) |
| Meio | leads (INT), mqls (INT), sqls (INT) |
| Fundo | negotiations (INT), won (INT), lost (INT), revenue (NUMERIC) |
| Cohort [003] | cohort_period (DATE), observation_period (DATE), cohort_granularity, cohort_status |
| Velocidade [003] | days_to_mql, days_to_sql, days_to_deal, days_to_close (INT — mediana) |

### UNIQUE constraint (crítica)

```sql
UNIQUE (period_start, period_end, granularity, level, campaign_id, ad_set_id, ad_id, tracking)
```
Colunas `campaign_id`, `ad_set_id`, `ad_id` são nullable — o nível determina quais são NOT NULL.

**Armadilha:** Adicionar campo a esse UNIQUE em migration posterior requer DROP + recreate da constraint. Em tabela com dados, isso é `ACCESS EXCLUSIVE` lock. Planeje antes de crescer o UNIQUE.

### Índices existentes

```sql
idx_metrics_project_period  ON (project_id, period_start, period_end)
idx_metrics_level           ON (level, campaign_id, ad_set_id, ad_id)
idx_metrics_granularity     ON (project_id, granularity)
idx_metrics_canal           ON (project_id, canal)
idx_metrics_cohort          ON (project_id, cohort_period, observation_period) WHERE cohort_period IS NOT NULL
```

---

## `icp_product_map` — Eixo de Toda Análise

```sql
icp_product_map (
  id UUID PRIMARY KEY,
  project_id UUID FK → projects,
  version_id UUID FK → versions,
  icp_id UUID FK → icps,
  product_id UUID FK → products,
  priority INT,
  status TEXT,
  strategic_notes TEXT,
  target_cpl NUMERIC,
  target_cpa NUMERIC,
  target_roas NUMERIC,
  UNIQUE (icp_id, product_id, version_id)
)
```

**Toda tabela de dados novos deve referenciar `icp_product_map_id` ou ser cruzável via `ads.icp_product_id`.**

---

## JSONB no Schema

**Apenas 1 campo JSONB:** `governance_events.payload`

```sql
payload JSONB  -- metadados variáveis por tipo de evento
-- Exemplo: {"metric": "CPL", "current": 45.20, "target": 35.00}
```

Sem índice GIN por enquanto. Se `payload->>'metric'` virar filtro frequente de dashboard:
```sql
CREATE INDEX CONCURRENTLY idx_gov_payload_gin 
ON governance_events USING GIN (payload);
```

**Nota:** O briefing menciona `qualification` e `capi_data` como JSONB futuros. Quando chegarem, aplicar mesma regra: índice GIN se virar filtro.

---

## RLS — Estrutura de Segurança

```sql
-- Função 1: membro da org? (SECURITY DEFINER + STABLE = cached por query)
CREATE FUNCTION is_org_member(p_org_id UUID) RETURNS BOOLEAN
SECURITY DEFINER STABLE LANGUAGE sql AS $$
  SELECT EXISTS (SELECT 1 FROM organization_members
                 WHERE org_id = p_org_id AND user_id = auth.uid())
$$;

-- Função 2: acesso ao projeto via org?
CREATE FUNCTION has_project_access(p_project_id UUID) RETURNS BOOLEAN
SECURITY DEFINER STABLE LANGUAGE sql AS $$
  SELECT EXISTS (SELECT 1 FROM projects p
                 JOIN organization_members om ON om.org_id = p.org_id
                 WHERE p.id = p_project_id AND om.user_id = auth.uid())
$$;
```

**Todas as 17+ tabelas têm RLS com `USING (has_project_access(project_id))`.**

**Custo RLS em analytics:** `has_project_access()` faz JOIN por linha avaliada. Em table scan de `performance_metrics` com 1M+ linhas, o custo pode dominar a query. Mitigação: o Validator usa `service_role_key` (bypassa RLS). Dashboards externos via `anon_key` precisam de projeto_id indexado + RLS otimizado.

**Índice necessário para RLS eficiente:**
```sql
-- Já existe em organization_members:
CREATE INDEX idx_org_members_user ON organization_members(user_id);
CREATE INDEX idx_org_members_org ON organization_members(org_id);
-- Garantir que project_id está indexado em cada tabela com RLS — já está via idx_*_project_version
```

---

## Tabelas com Imutabilidade de Conteúdo

| Tabela | Imutável? | Mutável? |
|--------|-----------|----------|
| `performance_metrics` | Conteúdo de métricas | — |
| `account_health` | Após created_at | — |
| `truths_and_traps` | Conteúdo (entry_text, etc) | is_active, superseded_by |
| `governance_events` | Conteúdo principal | status, resolved_at, payload |
| `asset_versions` | Conteúdo do asset | feedback, status |

Imutabilidade é padrão arquitetural — não é enforced por trigger hoje, mas é contrato de uso. Migrations que adicionam `ON UPDATE CASCADE` ou `SET DEFAULT` em tabelas imutáveis precisam de discussão.

---

## RICE Score — Coluna Gerada

`execution_backlog` tem RICE score como coluna gerada:
```sql
rice_score NUMERIC GENERATED ALWAYS AS (
  CASE WHEN confidence > 0 AND effort > 0
    THEN (reach * impact * (confidence / 100.0)) / effort
    ELSE 0
  END
) STORED
```

**Exemplo de derivado correto:** o valor depende só de colunas da mesma linha, sem JOIN. Pode ser GENERATED STORED. CTR/CPL/ROAS não podem porque precisam de múltiplas linhas ou dados de outra tabela.

---

## Caminho dos Arquivos de Schema

```
/root/dante/infrastructure/supabase-schema.sql       — schema base (001)
/root/dante/infrastructure/migrations/
  ├── 002_performance_metrics_crm.sql
  ├── 003_performance_metrics_cohort.sql
  ├── 004_governanca_ia.sql
  ├── 005_design_ia.sql
  └── fn_data_analysis.sql
```
