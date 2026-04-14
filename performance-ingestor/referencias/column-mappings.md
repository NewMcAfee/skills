# Mapeamento de Colunas — Performance Ingestor

Referência de mapeamento entre colunas dos exports de plataforma e campos do schema `performance_metrics`.

**Última atualização:** 2026-04-12

---

## Meta Ads Manager

Exports gerados em: Gerenciador de Anúncios → Relatórios → Exportar CSV

| Coluna no export Meta | Campo no schema | Obrigatório | Observações |
|----------------------|----------------|-------------|-------------|
| `Date` / `Reporting starts` | `period_start` | ✅ | Depende da configuração do relatório |
| `Reporting ends` | `period_end` | ✅ | Presente quando granularidade não é diária |
| `Campaign name` | referência → `campaigns.name` | ✅ | Resolvido para `campaign_id` pelo data_miner |
| `Ad Set Name` | referência → `ad_sets.name` | — | Presente no nível conjunto e anúncio |
| `Ad name` | referência → `ads.name` | — | Presente no nível anúncio |
| `Amount spent (BRL)` | `investment` | ✅ | Moeda varia por conta — ajustar se necessário |
| `Impressions` | `impressions` | ✅ | |
| `Reach` | `reach` | — | |
| `Link clicks` | `clicks` | — | Preferir sobre `Clicks (all)` |
| `Clicks (all)` | `clicks` | — | Fallback se `Link clicks` ausente |
| `Leads` | `leads` | — | Quando objetivo = Lead |
| `Results` | `leads` | — | Fallback quando `Leads` ausente e objetivo = Lead |

**Colunas derivadas — sempre ignorar:**
`Cost per result`, `CTR (all)`, `CPC (all)`, `CPM`, `Frequency`, `Result Rate`

---

## Google Ads

Exports gerados em: Google Ads → Relatórios → Baixar CSV

| Coluna no export Google | Campo no schema | Obrigatório | Observações |
|------------------------|----------------|-------------|-------------|
| `Day` | `period_start` e `period_end` | ✅ | Para granularidade diária, start = end = Day |
| `Campaign` | referência → `campaigns.name` | ✅ | |
| `Ad group` | referência → `ad_sets.name` | — | Presente no nível conjunto e anúncio |
| `Ad` / `Ad name` | referência → `ads.name` | — | Varia por tipo de export |
| `Cost` | `investment` | ✅ | Em moeda da conta |
| `Impr.` | `impressions` | ✅ | Abreviação padrão do Google |
| `Impressions` | `impressions` | ✅ | Alternativa por extenso |
| `Clicks` | `clicks` | — | |
| `Conversions` | `leads` | — | Quando ação de conversão = lead |

**Colunas derivadas — sempre ignorar:**
`Conv. rate`, `Avg. CPC`, `CTR`, `Avg. CPM`, `Avg. position`, `Cost / conv.`

**Edge case — export sem coluna de data (período agregado):**
Alguns relatórios do Google Ads não incluem a coluna `Day` quando exportam o período inteiro como uma linha por campanha. Nesse caso: usar as datas informadas pelo operador nos pré-requisitos como `period_start` e `period_end` para todos os registros.

---

## Campos do schema nunca presentes em exports de plataforma

Esses campos vêm exclusivamente de ingestão de CRM — sempre `null` na ingestão de performance:

`mqls`, `sqls`, `negotiations`, `won`, `lost`, `revenue`

---

## Schema drift — quando os nomes mudam

Meta e Google alteram nomes de colunas periodicamente (geralmente em atualizações trimestrais do painel). Quando um header não for encontrado neste mapeamento:

1. Exiba o header desconhecido ao operador
2. Pergunte qual campo do schema ele corresponde
3. Registre o mapeamento custom no payload (campo `custom_mapping`)
4. **Após ingestão bem-sucedida, atualize este arquivo com o novo nome de coluna**

O operador nunca deve precisar mapear a mesma coluna duas vezes.
