# Contratos do Validator — Skill de Ingestão

## Base URL
`http://localhost:8000` (interno ao VPS, via rede dante_internal)

## ENUMs aceitos

| Campo | Tabela | Valores válidos |
|---|---|---|
| status (project) | projects | `active`, `paused`, `archived` |
| status (product) | products | `active`, `archived` |
| status (icp) | icps | `active`, `testing`, `archived` |
| status (icp_map) | icp_product_map | `active`, `testing`, `archived` |
| awareness_level | icps | `UNC`, `PRB`, `SOL`, `PRO`, `MIX` |
| gender | icps | `M`, `F`, `all` |
| active_channels | planning_data | `META`, `GOOG`, `LINK`, `TIKT` (array) |
| tracking_completeness | planning_data | `platform_only`, `platform_crm` |

---

## POST /validate/organization

```json
{
  "name": "Nome da Organização",
  "slug": "nome-da-organizacao"
}
```

**Resposta 201:**
```json
{ "created": { "id": "uuid", "name": "...", "slug": "..." } }
```

**Resposta 409 (já existe):**
```json
{ "error": "Organization com slug 'nome-da-organizacao' já existe", "existing_id": "uuid" }
```

---

## POST /validate/project

```json
{
  "org_id": "uuid",
  "name": "Nome do Projeto",
  "slug": "nome-do-projeto",
  "status": "active"
}
```

**Resposta 201:**
```json
{ "created": { "id": "uuid", "org_id": "uuid", "name": "...", "slug": "..." } }
```

---

## POST /validate/version

```json
{
  "project_id": "uuid",
  "label": "Q1 2026",
  "valid_from": "2026-04-09",
  "revision_summary": "Onboarding inicial do projeto",
  "created_by": "skill:ingestao"
}
```

**Resposta 201:**
```json
{ "created": { "id": "uuid", "project_id": "uuid", "version_number": 1, "label": "...", "valid_from": "..." } }
```

---

## POST /validate/planning_data

```json
{
  "project_id": "uuid",
  "version_id": "uuid",
  "segment": "Construtoras médias",
  "sales_cycle": "30-90 dias",
  "monthly_budget_total": 15000.00,
  "break_even_leads": 120,
  "active_channels": ["META", "GOOG"],
  "budget_meta": 10000.00,
  "budget_goog": 5000.00,
  "gtm_priority": "Foco em COLD LAL para volume de leads qualificados",
  "gtm_north_star": "200 leads/mês com CPL abaixo de R$75",
  "min_impressions_for_reading": 5000,
  "min_leads_for_reading": 30,
  "min_days_for_reading": 14,
  "confidence_threshold": 0.80,
  "tracking_completeness": "platform_only",
  "crm_integrated": false
}
```

Use `null` para campos sem dados extraídos. Não omita campos do contrato.

**Resposta 201:**
```json
{ "created": { "id": "uuid", "project_id": "uuid", "version_id": "uuid", ... } }
```

---

## POST /validate/product

```json
{
  "project_id": "uuid",
  "version_id": "uuid",
  "product_ref": "PROD-1",
  "name": "BootcampVendas",
  "status": "active",
  "puv_statement": "Aprenda a fechar R$50k/mês em vendas consultivas em 8 semanas ou devolvemos seu investimento.",
  "main_differentiator": "Único bootcamp com simulações ao vivo com compradores reais",
  "price_point": 2997.00,
  "target_cpl": 80.00,
  "target_cpa": 1200.00,
  "target_roas": 2.50
}
```

Use `null` para métricas-alvo sem dados. `puv_statement` ≤ 280 chars.

**Resposta 201:**
```json
{ "created": { "id": "uuid", "product_ref": "PROD-1", "name": "..." } }
```

---

## POST /validate/icp

```json
{
  "project_id": "uuid",
  "version_id": "uuid",
  "icp_ref": "ICP-A",
  "name": "Vendedor B2B Experiente",
  "status": "active",
  "awareness_level": "PRB",
  "primary_pain": "Perde negócios para concorrentes mais baratos sem saber argumentar valor",
  "primary_desire": "Fechar contratos maiores sem precisar dar desconto",
  "main_objection": "Já tentei cursos antes e não funcionou",
  "buying_trigger": "Perdeu uma venda importante para concorrente mais barato",
  "age_min": 28,
  "age_max": 50,
  "gender": "all",
  "geo": ["BR-SP", "BR-RJ", "BR-MG"],
  "income_range": "R$5k–R$15k/mês",
  "target_cpl": 75.00,
  "target_cpa": null,
  "min_sample_leads": 30
}
```

`primary_pain`, `primary_desire`, `main_objection` ≤ 120 chars cada.
`geo` usa formato `PAÍS-ESTADO` (ex: `BR-SP`) ou `BR` para nacional.

**Resposta 201:**
```json
{ "created": { "id": "uuid", "icp_ref": "ICP-A", "name": "..." } }
```

---

## POST /validate/icp_product_map

```json
{
  "project_id": "uuid",
  "version_id": "uuid",
  "icp_id": "uuid-do-icp",
  "product_id": "uuid-do-product",
  "priority": 1,
  "status": "active",
  "strategic_notes": "ICP-A é o perfil mais validado para PROD-1. Testar PROD-2 em Q2.",
  "target_cpl": 75.00,
  "target_cpa": null,
  "target_roas": null
}
```

`priority`: 1 = maior prioridade. Use `null` para métricas não definidas para esta combinação.
Nunca use icp_ref ou product_ref — use sempre os UUIDs retornados pelo Validator.

**Resposta 201:**
```json
{ "created": { "id": "uuid", "icp_id": "uuid", "product_id": "uuid", "priority": 1 } }
```

---

## Erros estruturados (HTTP 422)

```json
{
  "error": "Campo 'awareness_level' inválido: 'CONSCIENTE'. Valores válidos: ['UNC', 'PRB', 'SOL', 'PRO', 'MIX']",
  "field": "awareness_level",
  "suggestion": "Valores válidos: ['UNC', 'PRB', 'SOL', 'PRO', 'MIX']"
}
```

Use `field` para identificar o campo e `suggestion` para a mensagem ao operador.
Corrija apenas o campo reportado — não recomece o fluxo.
