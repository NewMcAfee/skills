# CSV Bulk Import — Meta Ads Manager

## Estrutura do Arquivo

O CSV usa um modelo **flat com coluna de tipo por linha**. Cada linha representa uma entidade. A hierarquia é mantida por referência de nome: Ad Sets referenciam o nome da Campanha pai; Ads referenciam o nome do Ad Set pai.

---

## Colunas Obrigatórias

```
Row Type | Campaign Name | Ad Set Name | Ad Name | Campaign Objective | Campaign Status | Campaign Daily Budget (BRL) | Campaign Budget Optimization | Bid Strategy | Ad Set Status | Ad Set Daily Budget (BRL) | Start Date | End Date | Age Min | Age Max | Gender | Locations | Custom Audiences | Excluded Custom Audiences | Placements | Optimization Goal | Ad Status | Ad Format | Primary Text | Headline | Description | CTA Type | Website URL | Creative URL | icp_product_map_id
```

---

## Valores por Coluna

| Coluna | Tipo de linha | Valores |
|--------|--------------|---------|
| Row Type | Todos | `Campaign` · `Ad Set` · `Ad` |
| Campaign Objective | Campaign | `OUTCOME_LEADS` · `OUTCOME_SALES` · `OUTCOME_TRAFFIC` · `OUTCOME_AWARENESS` · `OUTCOME_ENGAGEMENT` |
| Campaign Status | Campaign | `ACTIVE` · `PAUSED` |
| Campaign Budget Optimization | Campaign | `ON` · `OFF` |
| Bid Strategy | Campaign / Ad Set | `LOWEST_COST_WITHOUT_CAP` · `LOWEST_COST_WITH_BID_CAP` · `COST_CAP` |
| Ad Set Status | Ad Set | `ACTIVE` · `PAUSED` |
| Start Date / End Date | Ad Set | Formato `MM/DD/YYYY HH:MM` |
| Age Min / Age Max | Ad Set | Inteiros (mín 18, máx 65) |
| Gender | Ad Set | `All` · `Male` · `Female` |
| Locations | Ad Set | Código ISO-3166 ou região (ex: `BR`, `SP`, `RJ`) — múltiplos separados por `\|` |
| Custom Audiences | Ad Set | Nome descritivo das audiências incluídas — separados por `\|` |
| Excluded Custom Audiences | Ad Set | Gerado pelo waterfall — separados por `\|` |
| Placements | Ad Set | `Automatic Placements` · `Feed` · `Reels` · `Stories` |
| Optimization Goal | Ad Set | `LEAD_GENERATION` · `OFFSITE_CONVERSIONS` · `LINK_CLICKS` · `IMPRESSIONS` |
| Ad Status | Ad | `ACTIVE` · `PAUSED` |
| Ad Format | Ad | `SINGLE_IMAGE` · `VIDEO` · `CAROUSEL` |
| CTA Type | Ad | `LEARN_MORE` · `SIGN_UP` · `GET_QUOTE` · `CONTACT_US` · `SEND_MESSAGE` |
| Creative URL | Ad | URL do asset — `[PLACEHOLDER]` para anúncios a preencher |
| icp_product_map_id | Ad Set + Ad | UUID da combinação ICP+Produto — **campo obrigatório Dante** |

---

## Template de Exemplo (1 campanha completa)

```csv
Row Type,Campaign Name,Ad Set Name,Ad Name,Campaign Objective,Campaign Status,Campaign Daily Budget (BRL),Campaign Budget Optimization,Bid Strategy,Ad Set Status,Ad Set Daily Budget (BRL),Start Date,End Date,Age Min,Age Max,Gender,Locations,Custom Audiences,Excluded Custom Audiences,Placements,Optimization Goal,Ad Status,Ad Format,Primary Text,Headline,Description,CTA Type,Website URL,Creative URL,icp_product_map_id
Campaign,TEST_META_LEAD_BootcampVendas_FormNativo,,,OUTCOME_LEADS,PAUSED,100.00,OFF,LOWEST_COST_WITHOUT_CAP,,,,,,,,,,,,,,,,,,,,,
Ad Set,TEST_META_LEAD_BootcampVendas_FormNativo,AUD-9-001_COLD_BROAD_GerenteMkt_BR_28-55-HM_VID,,,,,,,PAUSED,50.00,,,28,55,All,BR,,[EX] ClientesAtivos|[EX] ClientesInativos|[EX] SQLs|[EX] MQLs|[EX] VisitantesSite30d|[EX] VideoView50pct30d|[EX] Engajamento30d|[EX] LAL_Clientes|[EX] Interesses_GerenteMkt,Automatic Placements,LEAD_GENERATION,,,,,,,,uuid-do-icp-product-map
Ad Set,TEST_META_LEAD_BootcampVendas_FormNativo,AUD-7-002_COLD_LAL_GerenteMkt_BR_28-55-HM_VID,,,,,,,PAUSED,30.00,,,28,55,All,BR,,[EX] ClientesAtivos|[EX] ClientesInativos|[EX] SQLs|[EX] MQLs|[EX] VisitantesSite30d|[EX] Engajamento30d,Automatic Placements,LEAD_GENERATION,,,,,,,,uuid-do-icp-product-map
Ad Set,TEST_META_LEAD_BootcampVendas_FormNativo,AUD-4-003_HOT_RTG_VisitantesLP_BR_28-55-HM_MIX,,,,,,,PAUSED,20.00,,,28,55,All,BR,[EX] VisitantesSite30d,[EX] ClientesAtivos|[EX] ClientesInativos|[EX] SQLs|[EX] MQLs,Automatic Placements,LEAD_GENERATION,,,,,,,,uuid-do-icp-product-map
Ad,TEST_META_LEAD_BootcampVendas_FormNativo,AUD-9-001_COLD_BROAD_GerenteMkt_BR_28-55-HM_VID,AD-001_VID_UNC_Story_FundadorV4_DorOperacional,,,,,,,,,,,,,,,,,,,PAUSED,VIDEO,[PLACEHOLDER: texto primário — Story UNC],[PLACEHOLDER: headline],[PLACEHOLDER: descrição],LEARN_MORE,[PLACEHOLDER: URL destino],[PLACEHOLDER: URL do vídeo],uuid-do-icp-product-map
Ad,TEST_META_LEAD_BootcampVendas_FormNativo,AUD-7-002_COLD_LAL_GerenteMkt_BR_28-55-HM_VID,AD-002_VID_PRB_Proof_FundadorV4_Prova3xROI,,,,,,,,,,,,,,,,,,,PAUSED,VIDEO,[PLACEHOLDER: texto primário — Proof PRB],[PLACEHOLDER: headline],[PLACEHOLDER: descrição],LEARN_MORE,[PLACEHOLDER: URL destino],[PLACEHOLDER: URL do vídeo],uuid-do-icp-product-map
Ad,TEST_META_LEAD_BootcampVendas_FormNativo,AUD-4-003_HOT_RTG_VisitantesLP_BR_28-55-HM_MIX,AD-003_STA_SOL_Loss_FundadorV4_UltimaChance,,,,,,,,,,,,,,,,,,,PAUSED,SINGLE_IMAGE,[PLACEHOLDER: texto primário — Loss SOL],[PLACEHOLDER: headline],[PLACEHOLDER: descrição],GET_QUOTE,[PLACEHOLDER: URL destino],[PLACEHOLDER: URL da imagem],uuid-do-icp-product-map
```

---

## Regras de Formatação

1. **Status inicial:** Sempre `PAUSED` — o operador ativa manualmente após revisão no Ads Manager
2. **Budget:** Iniciar com valores do Sobral; se não definido, usar placeholder `[DEFINIR]`
3. **Datas:** Deixar em branco se não definidas — o Meta usará a data de upload como início
4. **icp_product_map_id:** Preencher em Ad Set e Ad — **nunca deixar vazio**
5. **Campos de criativo:** Sempre `[PLACEHOLDER: descrição]` — nunca inventar copy
6. **Separador de múltiplos valores:** Pipe `|` sem espaços

---

## Checklist pré-upload

- [ ] Todos os nomes validados contra taxonomia Dante
- [ ] Exclusões waterfall aplicadas em todos os conjuntos
- [ ] Todos os `[PLACEHOLDER]` de creative preenchidos com assets reais
- [ ] Audiências personalizadas (`[EX] ...`) criadas no Meta Business Manager
- [ ] `icp_product_map_id` preenchido em todos os Ad Sets e Ads
- [ ] Campanha com status `PAUSED` (ativar só após revisão no Ads Manager)
