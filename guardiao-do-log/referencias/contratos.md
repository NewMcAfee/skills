# Contratos do Validator — Guardião do Log

## Base URL
- **Bloco 2 (direto):** `http://localhost/validator`
- **Bloco 3 (N8N):** URL do webhook N8N (fornecida pelo operador)

## ENUMs aceitos

### Campanha
| Campo | Valores válidos |
|---|---|
| STATUS | `HALL`, `TEST`, `EVER` |
| CANAL | `META`, `GOOG`, `LINK`, `TIKT` |
| OBJETIVO | `LEAD`, `SALES`, `TRFC`, `AWAR`, `MSNG` |
| PRODUTO | CamelCase sem espaços (ex: `BootcampVendas`) |
| DESTINO | `FormNativo`, `WhatsApp`, `LandingPage`, `Site`, `Ecom`, `App` |

### Conjunto
| Campo | Valores válidos |
|---|---|
| HIERARQUIA | Dígito 0–9 |
| ID | Alfanumérico com zero-padding (ex: `001`) |
| FUNIL | `COLD`, `WARM`, `HOT` |
| TIPO-ALVO | `LAL`, `INT`, `RTG`, `CRM`, `BROAD` |
| PÚBLICO | CamelCase (ex: `Clientes`) |
| GEO | Sigla do país/região (ex: `BR`) |
| DEMO | Formato `IDADEMIN-IDADEMAX-GENERO` (ex: `25-60-HM`) |
| FORMATO | `STA`, `VID`, `CAR`, `MOT`, `MIX` |

### Anúncio
| Campo | Valores válidos |
|---|---|
| ID | Numérico com zero-padding, mín. 3 dígitos (ex: `015`) |
| FORMATO | `STA`, `VID`, `CAR`, `MOT`, `MIX` |
| CONSCIÊNCIA | `UNC`, `PRB`, `SOL`, `PRO`, `MIX` |
| GANCHO | `Loss`, `Proof`, `Story`, `Error`, `Fact`, `Contr` |
| AVATAR | CamelCase (ex: `CamilaFarani`) |
| VARIAÇÃO | CamelCase (ex: `PerdendoDinheiro`) |

---

## Cascata de Exclusões (Conjunto)

| Hierarquia | Nível | Exclui automaticamente |
|---|---|---|
| 0 | Clientes Ativos | nenhum |
| 1 | Clientes Inativos | AUD-0 |
| 2 | SQL | AUD-0, AUD-1 |
| 3 | MQL | AUD-0, AUD-1, AUD-2 |
| 4 | Quente (Site/LP) | AUD-0 a AUD-3 |
| 5 | Morno (Assistiram Ads) | AUD-0 a AUD-4 |
| 6 | Morno (Engajamento) | AUD-0 a AUD-5 |
| 7 | Frio LAL/Semelhantes | AUD-0 a AUD-6 |
| 8 | Frio Interesses | AUD-0 a AUD-7 |
| 9 | Frio Aberto/Broad | AUD-0 a AUD-8 |

---

## Payload — POST /validate/campaign

```json
{
  "campaign_name": "TEST_META_LEAD_BootcampVendas_FormNativo",
  "project_id": "uuid",
  "version_id": "uuid",
  "icp_product_map_id": "uuid",
  "platform_id": null,
  "launched_at": null,
  "notes": null
}
```
**Resposta 201:**
```json
{ "created": { "id": "uuid", "campaign_name": "...", ... } }
```

---

## Payload — POST /validate/ad_set

```json
{
  "ad_set_name": "AUD-7-001_COLD_LAL_Clientes_BR_25-60-HM_VID",
  "campaign_id": "uuid",
  "project_id": "uuid",
  "version_id": "uuid",
  "platform_id": null,
  "launched_at": null,
  "daily_budget": null
}
```
**Resposta 201:**
```json
{
  "created": { "id": "uuid", "ad_set_name": "...", ... },
  "excluded_hierarchy_digits": [0, 1, 2, 3, 4, 5, 6]
}
```

---

## Payload — POST /validate/ad

```json
{
  "ad_name": "AD-015_STA_UNC_Loss_CamilaFarani_PerdendoDinheiro",
  "ad_set_id": "uuid",
  "project_id": "uuid",
  "version_id": "uuid",
  "icp_product_map_id": "uuid",
  "headline": null,
  "primary_text": null,
  "cta_type": null,
  "platform_id": null,
  "launched_at": null
}
```
**Resposta 201:**
```json
{ "created": { "id": "uuid", "ad_name": "...", ... } }
```

---

## Erros estruturados (HTTP 422)

```json
{
  "error": "Campo 'status' inválido: 'ATIVO'. Valores válidos: ['HALL', 'TEST', 'EVER']",
  "field": "status",
  "suggestion": "Valores válidos: ['HALL', 'TEST', 'EVER']"
}
```

O campo `field` identifica exatamente onde está o problema.
O campo `suggestion` contém os valores aceitos ou a orientação de correção.
