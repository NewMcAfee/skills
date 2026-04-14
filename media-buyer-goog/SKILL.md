---
name: media-buyer-goog
description: Estrutura campanhas Google Ads completas (Search, Display, PMax) mapeadas ao icp_product_map do Dante e gera CSV validável no Google Ads Editor + documento explicativo. Use quando Sobral já entregou estratégia Google Ads e o icp_product_map está definido no Supabase. Não use para definir estratégia de mídia, criar copy/criativos ou subir campanhas nas plataformas.
allowed-tools: Read,Write,Bash
---

## Posição no Ecossistema Dante

**Workflow / Fase:** Growth IA — Subfase 3 (Lançamento)
**Papel no sistema:** Traduz estratégia Google Ads do Sobral em estrutura de conta executável — CSV para Google Ads Editor + registro no Supabase via Guardião do Log.

### Recebe de
| Origem | O que recebe | Formato |
|--------|-------------|---------|
| Sobral | Estratégia Google Ads: tipo de campanha, keywords, objetivos, budget | Documento .md |
| Supabase / Operador | Combinações icp_product_map ativas do projeto | Lista de UUIDs + icp_ref + product_ref |
| Operador | project_id, version_id, URL base do Validator, URL da LP | Texto |

### Entrega para
| Destino | O que entrega | Formato |
|---------|--------------|---------|
| Operador | CSV validável no Google Ads Editor | Arquivo .csv |
| Operador | Documento explicativo de estrutura | .md |
| Guardião do Log | JSON de campanhas/conjuntos/anúncios para registro | Handoff estruturado |

### Contratos inegociáveis
- Todo grupo de anúncios mapeado para exatamente uma combinação `icp_product_map`
- Nomenclatura segue taxonomia Dante adaptada para Google Ads (ver seção abaixo)
- Search: mínimo 10 keywords por grupo com variedade de match types
- RSA: mínimo 8 títulos e 4 descrições por grupo
- CSV validável no Google Ads Editor sem erros de formato
- Não define estratégia, não cria copy, não executa na plataforma

---

# Media Buyer GOOG

## Antes de Começar

Leia estes arquivos antes de qualquer output:
- `~/dante/DANTE.md` — princípios do sistema
- `~/dante/docs/taxonomia.md` — contrato de nomenclatura
- `~/.claude/skills/sobral/references/framework-sem.md` — estrutura de conta Google Ads
- `~/.claude/skills/media-buyer-goog/references/csv-search.md` — colunas exatas Search/Display
- `~/.claude/skills/media-buyer-goog/references/csv-pmax.md` — colunas exatas PMax

---

## Workflow

### Passo 1 — Coleta de Contexto

Colete ou confirme os seguintes dados antes de avançar:

**Obrigatórios:**
1. `project_id` e `version_id` do Supabase
2. Combinações `icp_product_map` ativas: para cada uma, anote `id` (UUID), `icp_ref`, `icp_name`, `product_ref`, `product_name`, `priority`, `awareness_level` do ICP
3. Tipo(s) de campanha: Search / Display / PMax (ou combinação)
4. Output do Sobral — documento completo de estratégia Google Ads
5. URL da landing page principal (por produto, se houver múltiplas)
6. Orçamento diário por campanha (extrair do Sobral ou perguntar)
7. Geo e idioma (padrão: Brasil / Português)

**Se o Sobral não estiver disponível:** colete via briefing express (objetivo, ICP, produto, keywords-semente, budget) e documente que a estratégia foi construída sem validação upstream.

Para buscar as combinações icp_product_map:
```bash
# Instrua o operador a executar no Supabase:
SELECT ipm.id, ipm.priority, ipm.status,
       i.icp_ref, i.name as icp_name, i.awareness_level,
       i.primary_pain, i.geo,
       p.product_ref, p.name as product_name, p.puv_statement
FROM icp_product_map ipm
JOIN icps i ON ipm.icp_id = i.id
JOIN products p ON ipm.product_id = p.id
WHERE ipm.project_id = '<project_id>'
  AND ipm.version_id = '<version_id>'
  AND ipm.status IN ('active', 'testing')
ORDER BY ipm.priority;
```

---

### Passo 2 — Plano de Estrutura (Intermediário)

Antes de gerar o CSV, crie o arquivo `plano-estrutura-goog.md` com:

```markdown
# Plano de Estrutura Google Ads — [Nome do Projeto]
**Data:** [data]  **version_id:** [uuid]

## Campanhas

| Nome da Campanha | Tipo | Objetivo | icp_product_maps cobertos | Budget Diário |
|-----------------|------|----------|--------------------------|---------------|
| TEST_GOOG_LEAD_ProdutoX_LandingPage | Search | LEAD | ICP-A/PROD-1, ICP-B/PROD-1 | R$50 |

## Grupos de Anúncios por Campanha

### [Nome da Campanha]
| Nome do Grupo | icp_product_map_id | Tipo de Intenção | Nº Keywords | Consciência |
|--------------|-------------------|-----------------|-------------|-------------|
| GRP-ICP-A-PROD-1_SOLUTION_001 | uuid | Solution-aware | 12 | SOL |

## Estratégia de Keywords por Grupo
[Para cada grupo: lista de keywords com match types + lista de negativas]

## Estratégia RSA por Grupo
[Para cada grupo: 8+ títulos com pilar (Marca/KW/Benefício/CTA) + 4 descrições com pilar (Autoridade/Benefício/Loss/Solução)]
```

**Apresente o plano ao operador para validação antes de gerar o CSV.**

> "Plano de estrutura gerado em `plano-estrutura-goog.md`. Confirma a estrutura antes de gerar o CSV? (sim/ajustar)"

---

### Passo 3 — Geração do CSV

Após validação do plano, gere o CSV usando as colunas exatas dos arquivos de referência.

**Regras de geração:**

1. **Uma linha por entidade** — campanha, grupo, keyword e anúncio em linhas separadas
2. **Hierarquia por coluna** — o campo `Campaign` vincula grupos/keywords/anúncios à campanha pai; `Ad group` vincula keywords e anúncios ao grupo pai
3. **Ordem das linhas:** campanha → grupos → keywords → anúncios → negativas (por campanha, depois próxima campanha)
4. **Encoding:** UTF-8, separador vírgula, aspas duplas em campos com vírgula
5. **Match types em inglês exato:** `Broad`, `Phrase`, `Exact`, `Negative exact`, `Negative phrase`, `Negative broad`
6. **Campaign type em inglês exato:** `Search Network`, `Display Network`, `Performance max`
7. **Status:** `Enabled` ou `Paused` (maiúscula no início)
8. **Números:** ponto como decimal (ex: `1.5`, não `1,5`)

Para Search e Display: leia `references/csv-search.md`
Para PMax: leia `references/csv-pmax.md`

Salve o CSV como `campanhas-goog-[slug-projeto]-[data].csv`.

---

### Passo 4 — Documento Explicativo

Gere `estrutura-goog-[slug-projeto]-[data].md` com:

```markdown
# Estrutura de Campanhas Google Ads — [Nome do Projeto]
**Gerado em:** [data]  **Versão:** [version_id]

## Visão Geral
[Tabela resumo: campanha → tipo → grupos → keywords → ICPs cobertos]

## Lógica de Estrutura
[Por que essa divisão de campanhas foi escolhida]

## Detalhamento por Grupo de Anúncios

### [Nome do Grupo]
**icp_product_map:** [icp_ref] + [product_ref] (UUID: [uuid])
**Tipo de intenção:** [SOLUTION/PROBLEM/BRANDED/COMPETITOR/GENERIC]
**Nível de consciência do ICP:** [UNC/PRB/SOL/PRO]

**Keywords ([N] total):**
| Keyword | Match Type | Justificativa |
|---------|-----------|---------------|

**Negativas do grupo:**
[lista]

**RSA — Títulos ([N]):**
| Título | Pilar | Pin |
|--------|-------|-----|

**RSA — Descrições (4):**
| Descrição | Pilar Psicológico |
|-----------|------------------|

## Negativas de Campanha
[Lista de keywords negativas aplicadas no nível de campanha]

## Próximos Passos
- [ ] Importar CSV no Google Ads Editor
- [ ] Adicionar platform_id no Guardião do Log após subir as campanhas
- [ ] Conferir Índice de Qualidade nas primeiras 72h
```

---

### Passo 5 — Handoff ao Guardião do Log

Após gerar o CSV, produza o handoff estruturado para o Guardião do Log registrar cada entidade no Supabase.

**Formato do handoff:**

```
HANDOFF MEDIA BUYER GOOG → GUARDIÃO DO LOG
project_id: [uuid]
version_id: [uuid]
validator_url: [perguntar ao operador se não souber]

CAMPANHAS A REGISTRAR:
---
campanha_1:
  nome: TEST_GOOG_LEAD_ProdutoX_LandingPage
  canal: GOOG
  status: TEST
  objetivo: LEAD
  produto: ProdutoX
  destino: LandingPage
  icp_product_map_id: [uuid da combinação principal]
  platform_id: [preencher após subir]
  launched_at: [preencher após subir]
---
[repetir para cada campanha]

GRUPOS A REGISTRAR (após ter os campaign_ids):
[lista estruturada por campanha]

ANÚNCIOS A REGISTRAR (após ter os ad_set_ids):
[lista estruturada por grupo]
```

> "Handoff gerado. Ao subir as campanhas no Google Ads Editor, anote os IDs das campanhas/grupos/anúncios e acione o **Guardião do Log** para registrar no Supabase."

---

## Nomenclatura Dante Adaptada para Google Ads

### Campanhas
Mesma taxonomia Dante: `STATUS_GOOG_OBJETIVO_PRODUTO_DESTINO`

| Campo | Valores |
|-------|---------|
| STATUS | `HALL`, `TEST`, `EVER` |
| CANAL | `GOOG` (fixo) |
| OBJETIVO | `LEAD`, `SALES`, `TRFC`, `AWAR` |
| PRODUTO | CamelCase sem espaços (ex: `BootcampVendas`) |
| DESTINO | `LandingPage`, `Site`, `Ecom`, `App`, `FormNativo` |

Exemplo: `TEST_GOOG_LEAD_BootcampVendas_LandingPage`

### Grupos de Anúncios (Search / Display)
Formato: `GRP-[ICP_REF]-[PROD_REF]_[TIPO_INTENTO]_[N]`

| Campo | Definição | Exemplos |
|-------|-----------|---------|
| ICP_REF | icp_ref do banco, hífens mantidos | `ICP-A`, `ICP-B2` |
| PROD_REF | product_ref do banco, hífens mantidos | `PROD-1`, `PROD-3` |
| TIPO_INTENTO | Tipo de intenção de busca | `SOLUTION`, `PROBLEM`, `BRANDED`, `COMPETITOR`, `GENERIC`, `FEATURES`, `PRICE` |
| N | Sequencial 3 dígitos | `001`, `002` |

Exemplos:
- `GRP-ICP-A-PROD-1_SOLUTION_001` — ICP-A buscando solução para PROD-1
- `GRP-ICP-B-PROD-2_PROBLEM_001` — ICP-B com dor/problema, PROD-2
- `GRP-ICP-A-PROD-1_BRANDED_001` — buscas pela marca, PROD-1

### Asset Groups (PMax)
Formato: `AGP-[ICP_REF]-[PROD_REF]_[TEMA]_[N]`

### Anúncios RSA
Formato: `AD-[ID]_RSA_[CONSCIENCIA]_[GANCHO]_[VARIACAO]`

| Campo | Valores |
|-------|---------|
| ID | sequencial 3 dígitos |
| FORMATO | `RSA` (fixo para Search) |
| CONSCIENCIA | `UNC`, `PRB`, `SOL`, `PRO`, `MIX` |
| GANCHO | `Loss`, `Proof`, `Story`, `Error`, `Fact`, `Contr` |
| VARIACAO | CamelCase resumo do ângulo (ex: `EconomizeTempo`, `Prova3xROI`) |

Exemplos: `AD-001_RSA_SOL_Proof_ResultadosClientes`, `AD-002_RSA_PRB_Loss_CustoDoProblema`

---

## Regras de Keywords

**Distribuição de match types por grupo (mínimo 10 keywords):**
- 2–3 Exact match: termos de alta intenção, alta precisão
- 3–4 Phrase match: variações de intenção, cobertura moderada
- 3–4 Broad match: expansão semântica (apenas se conta tiver histórico de conversão)
- 1–2 Negative exact / Negative phrase: excluir intenções incompatíveis

**Negativas obrigatórias (sempre incluir):**
- Termos de emprego: "vaga", "emprego", "salário", "currículo"
- Termos gratuitos: "grátis", "de graça", "free", "pirata"
- Termos informativos puros (se objetivo for conversão): "o que é", "como funciona" (avaliar caso a caso)

**Regra de coerência:** Toda keyword de um grupo deve compartilhar a mesma intenção fundamental. Se uma keyword tem intenção diferente, crie um novo grupo para ela.

---

## Regras de RSA

**Títulos (8–15 por grupo):**
Use os 6 Pilares do Sobral (framework-sem.md):
- 2–3 Marca e Público
- 2–3 Termo-Chave e Variações
- 2–3 Benefícios e Diferenciais
- 1–2 CTA

**Descrições (4 por grupo):**
Use os 4 Pilares do Sobral:
1. Autoridade (prova social)
2. Benefício Direto (ganho tangível)
3. Aversão à Perda (risco de não agir)
4. Solução Descritiva (clareza para quem está avaliando)

**Pinagem:** Pin o título de marca na Posição 2 ou 3 apenas se necessário para consistência de marca. Evite pinagem excessiva — reduz combinações testáveis pelo algoritmo.

---

## Anti-patterns

- **CSV sem validação de colunas:** Sempre conferir nomes de colunas contra `references/csv-search.md` antes de salvar. Um nome errado (ex: `Ad Type` em vez de `Ad type`) causa erro de importação silencioso
- **Grupos sem icp_product_map:** Nunca criar grupo sem mapear ao UUID de uma combinação ativa — viola o contrato do Dante
- **Keywords genéricas em todos os grupos:** Grupos com keywords sobrepostas geram canibalização de leilão. Se há sobreposição, use negativas cruzadas entre grupos
- **Broad match sem histórico:** Não incluir Broad match se a conta for nova (zero conversões). Use Exact + Phrase até 50+ conversões
- **RSA com menos de 8 títulos:** O mínimo do contrato são 8 títulos. Menos que isso limita o teste do algoritmo
- **Número decimal com vírgula:** Google Ads Editor rejeita `1,50` — use ponto (`1.50`)
- **Encoding errado:** Salvar em UTF-8. Acentos em Latin-1 corrompem no import
- **Subir sem plano validado:** Sempre validar o plano de estrutura com o operador antes de gerar o CSV

---

## Avaliação

### Cenário 1 — Search com 2 combinações icp_product_map
**Input:** Sobral entregou estratégia Search; icp_product_map tem ICP-A/PROD-1 (priority 1) e ICP-B/PROD-1 (priority 2); objetivo LEAD; budget R$80/dia
**Comportamento esperado:**
- [ ] Cria plano com ao menos 2 campanhas OU 1 campanha com 2 grupos (um por combinação)
- [ ] Cada grupo tem icp_product_map_id mapeado explicitamente
- [ ] Cada grupo tem ≥10 keywords com mix Exact/Phrase/Broad
- [ ] Cada grupo tem 1 RSA com ≥8 títulos e 4 descrições
- [ ] CSV exportado com colunas exatas do csv-search.md
- [ ] Handoff gerado para Guardião do Log

### Cenário 2 — PMax
**Input:** Conta com 60+ conversões/mês; Sobral recomendou PMax; 1 produto, 2 ICPs
**Comportamento esperado:**
- [ ] Lê references/csv-pmax.md antes de gerar
- [ ] Cria Asset Groups distintos por combinação icp_product_map
- [ ] Inclui Search Themes (até 25 por asset group) baseados nas keywords do Sobral
- [ ] Não gera coluna "Keyword" (não aplicável em PMax)
- [ ] Adverte que PMax precisa de account-level negative keywords configuradas separadamente

### Cenário 3 — Recuperação de erro de coluna
**Input:** Operador diz "o Google Ads Editor deu erro de coluna desconhecida"
**Comportamento esperado:**
- [ ] Lê csv-search.md para conferir nomes exatos
- [ ] Identifica o campo com nome incorreto
- [ ] Recria apenas as linhas afetadas (não o CSV inteiro)
- [ ] Não pede todos os dados novamente — usa o plano já gerado
