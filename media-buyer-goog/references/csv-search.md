# CSV Google Ads Editor — Search e Display

## Estrutura do arquivo

O CSV do Google Ads Editor é um arquivo flat com todas as entidades misturadas. O tipo de entidade é inferido pelas colunas preenchidas. Siga esta ordem de linhas:

1. Linha de cabeçalho (obrigatória)
2. Linhas de campanha
3. Linhas de grupo (Ad group)
4. Linhas de keyword
5. Linhas de anúncio RSA
6. Linhas de negative keyword (nível campanha)
7. Repetir para próxima campanha

---

## Cabeçalho obrigatório (Search / Display)

```
Campaign,Campaign type,Campaign status,Budget,Bid strategy type,Networks,Languages,Location,Ad group,Ad group status,Max CPC,Ad group type,Keyword,Criterion type,Status,Ad type,Final URL,Path 1,Path 2,Headline 1,Headline 2,Headline 3,Headline 4,Headline 5,Headline 6,Headline 7,Headline 8,Headline 9,Headline 10,Headline 11,Headline 12,Headline 13,Headline 14,Headline 15,Headline 1 position,Headline 2 position,Headline 3 position,Description 1,Description 2,Description 3,Description 4,Description 1 position,Description 2 position
```

---

## Linha de Campanha

| Coluna | Valor | Notas |
|--------|-------|-------|
| `Campaign` | `TEST_GOOG_LEAD_ProdutoX_LandingPage` | Nome Dante |
| `Campaign type` | `Search Network` ou `Display Network` | Exato — case sensitive |
| `Campaign status` | `Enabled` ou `Paused` | Sempre `Enabled` para novas |
| `Budget` | `50` | Valor diário em número puro (sem R$) |
| `Bid strategy type` | `Maximize conversions` ou `Target CPA` ou `Manual CPC` | Ver abaixo |
| `Networks` | `Search Network` ou `Display Network` | Igual ao Campaign type |
| `Languages` | `Portuguese (Brazil)` | Exato com parênteses |
| `Location` | `Brazil` | Ou cidade/estado por extenso em inglês |

**Bid strategy por maturidade de conta:**
- Conta nova (0 conversões): `Manual CPC`
- Conta com 30–50 conversões/mês: `Maximize conversions`
- Conta com 50+ e CPA definido: `Target CPA`

Exemplo de linha completa de campanha:
```
TEST_GOOG_LEAD_BootcampVendas_LandingPage,Search Network,Enabled,80,Maximize conversions,Search Network,Portuguese (Brazil),Brazil,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
```
*Colunas de Ad group, Keyword e Ad ficam vazias em linhas de campanha.*

---

## Linha de Grupo de Anúncios

| Coluna | Valor | Notas |
|--------|-------|-------|
| `Campaign` | nome da campanha pai | Deve corresponder exatamente |
| `Ad group` | `GRP-ICP-A-PROD-1_SOLUTION_001` | Nome Dante adaptado |
| `Ad group status` | `Enabled` | |
| `Max CPC` | `2.5` | Lance padrão do grupo — ponto decimal |
| `Ad group type` | `Standard` (Search) ou `Default` (Display) | |

Exemplo:
```
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,GRP-ICP-A-PROD-1_SOLUTION_001,Enabled,2.5,Standard,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
```

---

## Linha de Keyword

| Coluna | Valor | Notas |
|--------|-------|-------|
| `Campaign` | nome da campanha pai | |
| `Ad group` | nome do grupo pai | |
| `Keyword` | `bootcamp de vendas online` | Texto da keyword sem colchetes/aspas |
| `Criterion type` | `Exact`, `Phrase`, `Broad`, `Negative exact`, `Negative phrase`, `Negative broad` | Exato — case sensitive |
| `Status` | `Enabled` | |
| `Max CPC` | (opcional) `3.0` | Omitir para herdar do grupo |

**Atenção sobre Criterion type:**
- `Exact` — não inclua colchetes no campo Keyword
- `Phrase` — não inclua aspas no campo Keyword
- `Broad` — keyword sem modificadores
- Negativas: use `Negative exact`, `Negative phrase` ou `Negative broad`

Exemplo — linha de keyword:
```
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,GRP-ICP-A-PROD-1_SOLUTION_001,,,,bootcamp de vendas online,Exact,Enabled,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
```

---

## Linha de RSA (Responsive Search Ad)

| Coluna | Valor | Notas |
|--------|-------|-------|
| `Campaign` | nome da campanha pai | |
| `Ad group` | nome do grupo pai | |
| `Ad type` | `Responsive search ad` | Exato — não abreviar |
| `Status` | `Enabled` | |
| `Final URL` | `https://seusite.com/pagina` | URL completa com https |
| `Path 1` | `bootcamp` | Opcional, até 15 chars |
| `Path 2` | `vendas` | Opcional, até 15 chars |
| `Headline 1`–`Headline 15` | texto do título | Até 30 chars cada — mínimo 3, meta 8–15 |
| `Headline 1 position`–`Headline 3 position` | `1`, `2`, `3` ou vazio | Preencher apenas para pins |
| `Description 1`–`Description 4` | texto da descrição | Até 90 chars cada — mínimo 2, meta 4 |
| `Description 1 position` / `Description 2 position` | `1`, `2` ou vazio | Pins opcionais |

**Limite de caracteres — nunca ultrapassar:**
- Headline: 30 caracteres (incluindo espaços)
- Description: 90 caracteres (incluindo espaços)
- Path 1/2: 15 caracteres cada

Exemplo de linha RSA (simplificado — apenas 3 headlines e 2 descrições mostrados):
```
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,GRP-ICP-A-PROD-1_SOLUTION_001,,,,,,,Responsive search ad,Enabled,https://site.com/bootcamp,bootcamp,vendas,Bootcamp de Vendas Online,Resultados em 30 Dias,Para Profissionais B2B,,,,,,,,,,,,,,,,,,,,,,,2,,,Somos referência em vendas consultivas há 8 anos.,Aumente sua taxa de fechamento sem aumentar a equipe.,,,,
```

---

## Negative Keywords de Nível de Campanha

Use linhas de keyword com `Criterion type` = `Negative exact` ou `Negative phrase`, mas com a coluna `Ad group` vazia. Isso aplica a negativa no nível de campanha.

Exemplo:
```
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,,,,,grátis,Negative exact,Enabled,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,,,,,vaga de emprego,Negative phrase,Enabled,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
```

---

## Template CSV Completo — Exemplo Funcional

```csv
Campaign,Campaign type,Campaign status,Budget,Bid strategy type,Networks,Languages,Location,Ad group,Ad group status,Max CPC,Ad group type,Keyword,Criterion type,Status,Ad type,Final URL,Path 1,Path 2,Headline 1,Headline 2,Headline 3,Headline 4,Headline 5,Headline 6,Headline 7,Headline 8,Description 1,Description 2,Description 3,Description 4
TEST_GOOG_LEAD_BootcampVendas_LandingPage,Search Network,Enabled,80,Maximize conversions,Search Network,Portuguese (Brazil),Brazil,,,,,,,,,,,,,,,,,,,,,,,,
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,GRP-ICP-A-PROD-1_SOLUTION_001,Enabled,2.5,Standard,,,,,,,,,,,,,,,,,,,,,
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,GRP-ICP-A-PROD-1_SOLUTION_001,,,,bootcamp de vendas,Exact,Enabled,,,,,,,,,,,,,,,,,,,,,
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,GRP-ICP-A-PROD-1_SOLUTION_001,,,,treinamento de vendas b2b,Phrase,Enabled,,,,,,,,,,,,,,,,,,,,,
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,GRP-ICP-A-PROD-1_SOLUTION_001,,,,,Responsive search ad,Enabled,https://site.com/bootcamp,bootcamp,vendas,Título 1 aqui,Título 2 aqui,Título 3 aqui,Título 4 aqui,Título 5 aqui,Título 6 aqui,Título 7 aqui,Título 8 aqui,Descrição 1 aqui (máx 90 chars),Descrição 2 aqui,Descrição 3 aqui,Descrição 4 aqui
TEST_GOOG_LEAD_BootcampVendas_LandingPage,,,,,,,,,,,,grátis,Negative exact,Enabled,,,,,,,,,,,,,,,,,,,,,
```

---

## Checklist antes de salvar o CSV

- [ ] Arquivo em UTF-8
- [ ] Separador: vírgula
- [ ] Primeira linha: cabeçalho com nomes exatos das colunas
- [ ] `Campaign type` com maiúscula em cada palavra: `Search Network`
- [ ] `Ad type`: `Responsive search ad` (apenas R maiúsculo)
- [ ] `Criterion type`: `Exact`, `Phrase`, `Broad`, `Negative exact`, `Negative phrase` (case exato)
- [ ] `Status` e `Campaign status` e `Ad group status`: `Enabled` ou `Paused`
- [ ] Números decimais com ponto (não vírgula)
- [ ] Headlines ≤ 30 chars, Descriptions ≤ 90 chars
- [ ] Toda campanha GOOG tem `Networks` = mesmo valor de `Campaign type`
- [ ] Linhas de campanha não têm colunas de Ad group/Keyword/Ad preenchidas
