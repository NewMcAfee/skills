# Taxonomia Dante — Referência para Media Buyer META

Fonte canônica: `dante/skills/skill-ingestao/referencias/taxonomia.md`

---

## Nível 1 — Campanha

**Sintaxe:** `{STATUS}_{CANAL}_{OBJETIVO}_{PRODUTO}_{DESTINO}`

| Campo | Valores aceitos |
|-------|----------------|
| STATUS | `HALL` (validada) · `TEST` (teste) · `EVER` (evergreen) |
| CANAL | `META` · `GOOG` · `LINK` · `TIKT` |
| OBJETIVO | `LEAD` · `SALES` · `TRFC` · `AWAR` · `MSNG` |
| PRODUTO | CamelCase, sem espaços (ex: `BootcampVendas`) |
| DESTINO | `FormNativo` · `WhatsApp` · `LandingPage` · `Site` · `Ecom` · `App` |

**Exemplo:** `TEST_META_LEAD_BootcampVendas_FormNativo`

---

## Nível 2 — Conjunto de Anúncios

**Sintaxe:** `AUD-{HIERARQUIA}-{ID_SEQ}_{FUNIL}_{TIPO}_{PÚBLICO}_{GEO}_{DEMO}_{FORMATO}`

| Campo | Valores aceitos |
|-------|----------------|
| HIERARQUIA | Dígito 0–9 (ver tabela waterfall) |
| ID_SEQ | Sequencial 3 dígitos: `001`, `002`… |
| FUNIL | `COLD` · `WARM` · `HOT` |
| TIPO | `LAL` · `INT` · `RTG` · `CRM` · `BROAD` |
| PÚBLICO | CamelCase (ex: `Clientes`, `GerenteMktB2B`) |
| GEO | Código geográfico (ex: `BR`, `SP`, `LAM`) |
| DEMO | `{IDADEMIN}-{IDADEMAX}-{GENERO}` — Gênero: `M` · `F` · `HM` |
| FORMATO | `STA` · `VID` · `CAR` · `MOT` · `MIX` |

**Exemplo:** `AUD-7-001_COLD_LAL_Clientes_BR_25-60-HM_VID`

### Mapeamento temperatura → dígito (regra da cascata)

| Dígito | Temperatura | Tipo de público |
|--------|-------------|-----------------|
| 0 | BOF/Reativação | Clientes Ativos (CRM) |
| 1 | BOF/Reativação | Clientes Inativos (CRM) |
| 2 | BOF/Hot | SQLs (CRM) |
| 3 | MOF/Hot | MQLs (CRM) |
| 4 | MOF/Quente | Visitantes Site/LP (pixel) |
| 5 | MOF/Morno | Video View 50%+ / Assistiram Ads |
| 6 | MOF/Morno | Engajamento em Redes (curtidas, comments, follows) |
| 7 | TOF/Frio | Semelhantes/LAL (a partir de lista CRM ou pixel) |
| 8 | TOF/Frio | Direcionamento Detalhado / Interesses |
| 9 | TOF/Frio | Aberto / Broad |

---

## Nível 3 — Anúncio

**Sintaxe:** `AD-{ID_SEQ}_{FORMATO}_{CONSCIÊNCIA}_{GANCHO}_{AVATAR}_{VARIAÇÃO}`

| Campo | Valores aceitos |
|-------|----------------|
| ID_SEQ | Sequencial 3 dígitos: `001`, `015`… |
| FORMATO | `STA` · `VID` · `CAR` · `MOT` · `MIX` |
| CONSCIÊNCIA | `UNC` · `PRB` · `SOL` · `PRO` · `MIX` |
| GANCHO | `Loss` · `Proof` · `Story` · `Error` · `Fact` · `Contr` |
| AVATAR | CamelCase (ex: `CamilaFarani`, `FundadorV4`) |
| VARIAÇÃO | CamelCase, tema visual/headline (ex: `PerdendoDinheiro`, `Prova3xROI`) |

**Exemplo:** `AD-015_STA_UNC_Loss_CamilaFarani_PerdendoDinheiro`

---

## Regra de Ouro

Toda campanha, conjunto e anúncio **deve referenciar `icp_product_map_id`** — nunca ICP ou produto isolados.
