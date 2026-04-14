---
name: performance-ingestor
description: Identifica o formato de exports de plataformas de mídia (Meta Ads Manager, Google Ads), mapeia colunas para o schema do Dante e gera o payload de configuração para o serviço data_miner executar a ingestão em performance_metrics via Validator. É a etapa zero do loop analítico. Ativar quando o operador precisar ingerir dados de performance de uma campanha. NÃO ativar para análise — isso é escopo do Data Miner e do Growth Lead.
allowed-tools: Read,Write,Bash
---

## Posição no Ecossistema Dante

**Workflow / Fase:** Growth IA — Loop (Etapa 0 — Ingestão de Performance)
**Papel no sistema:** Etapa zero do loop analítico — determina o mapeamento entre o export da plataforma e o schema Dante, e dispara o serviço Python que faz o parse e a escrita em `performance_metrics`.

### Recebe de
| Origem | O que recebe | Formato |
|--------|-------------|---------|
| Operador | Caminho para o arquivo de export da plataforma | Path para CSV |
| Operador | Contexto: período, granularidade desejada | Texto livre |

### Entrega para
| Destino | O que entrega | Formato |
|---------|--------------|---------|
| data_miner service (Python, via N8N) | Payload de configuração com mapeamento de colunas, período, granularidade | JSON estruturado |
| Operador | Relatório de ingestão: registros inseridos, ignorados, warnings | Texto estruturado |
| Data Miner skill (próxima etapa) | `performance_metrics` populada e pronta para análise | Estado no banco |

### Contratos
- Lê no máximo `head -n 4` do CSV — inspeciona estrutura, nunca dados em massa
- Nunca escreve no banco diretamente — parse e escrita são responsabilidade do serviço data_miner
- Não avança sem confirmar: plataforma, período, mapeamento de colunas e idempotência
- Campanhas precisam estar registradas via Guardião do Log — sem isso o serviço não resolve as referências

---

# Performance Ingestor — Dante Growth IA OS

## Contexto

O loop analítico pressupõe que `performance_metrics` está populada. Esta skill é a porta de entrada desses dados.

**Princípio central: LLM não processa dados brutos.** A skill inspeciona apenas os headers do arquivo para identificar o formato e gerar o mapeamento. O parse linha a linha, a validação de tipos e a escrita via Validator são feitos pelo serviço `data_miner` (Python, VPS).

```
Plataforma (export .csv)
  → Performance Ingestor (LLM)    ← inspeciona headers, gera mapeamento e payload
  → N8N                           ← orquestra o job
  → data_miner service (Python)   ← parseia, valida, POST no Validator
  → Validator API                 ← barreira entre serviço e banco
  → performance_metrics            ← banco populado
  → Data Miner (skill LLM)        ← lê banco, produz JSON limpo para Growth Lead
```

**Dependências de infraestrutura** (verificar antes de usar em produção):
- N8N flow `performance-ingest` — precisa ser criado (contrato: recebe payload desta skill)
- Validator endpoint `/validate/performance_metrics` — precisa ser implementado no Validator API

Se alguma dessas ainda não existe, as Fases 1–4 funcionam normalmente (mapeamento e payload são gerados). A Fase 5 falha — mas o payload gerado é o insumo para o Claude Code construir a infra.

---

## Pré-requisitos

Ao ser acionada, confirme quatro informações:

> "Para iniciar a ingestão, preciso de:
> 1. **Plataforma:** Meta Ads Manager ou Google Ads?
> 2. **Nível do export:** campanha / conjunto (ad set) / anúncio?
> 3. **Período:** data início e data fim dos dados no arquivo?
> 4. **Caminho do arquivo:** qual é o path completo do CSV?"

Se qualquer uma estiver faltando, **não avance**.

**Pergunta adicional obrigatória antes de disparar:**
> "Este período já foi ingerido antes? (sim / não / não sei)"

Se sim: confirme com o operador que o data_miner fará upsert (não duplicará). Se o serviço não for idempotente ainda, avise o risco antes de disparar.

---

## Fase 1 — Inspeção do arquivo

Leia o arquivo de mapeamentos de referência:

```
referencias/column-mappings.md
```

Em seguida, leia **apenas** as primeiras 4 linhas do CSV:

```bash
head -n 4 "<path_do_arquivo>"
```

Com base nos headers:
1. Identifique a plataforma pelo padrão de colunas — Meta e Google têm nomes distintos
2. Confirme o nível de granularidade (presença de `Ad name` vs apenas `Campaign`)
3. Verifique colunas obrigatórias: `investment` e `impressions` devem ter correspondência
4. Identifique colunas derivadas (CPL, CPC, ROAS, CTR) — serão ignoradas
5. Registre colunas sem mapeamento no arquivo de referência — são warnings, não bloqueios

Se os headers não forem reconhecidos com confiança, exiba-os ao operador e pergunte antes de continuar.

---

## Fase 2 — Mapeamento de colunas

Use o arquivo `referencias/column-mappings.md` como fonte canônica dos mapeamentos.

Para cada campo obrigatório (`investment`, `impressions`, `period_start`, referência de campanha): identifique a coluna correspondente no export.

Para campos opcionais (`reach`, `clicks`, `leads`): mapear quando presentes, `null` quando ausentes.

**Campos nunca presentes em exports de plataforma** (sempre `null` — vêm de ingestão de CRM):
`mqls`, `sqls`, `negotiations`, `won`, `lost`, `revenue`

**Edge case — Google Ads sem coluna de data:**
Se o export não contiver `Day`, use as datas informadas pelo operador nos pré-requisitos como `period_start` e `period_end` para todos os registros.

**Se um header não estiver no arquivo de referência (schema drift):**
1. Exiba o header desconhecido ao operador
2. Pergunte qual campo do schema corresponde
3. Inclua no payload como `custom_mapping`
4. Após ingestão bem-sucedida, atualize `referencias/column-mappings.md` com o novo nome

---

## Fase 3 — Confirmação do mapeamento

Exiba o mapeamento antes de gerar o payload:

```
=== MAPEAMENTO IDENTIFICADO ===

Arquivo:        [nome do arquivo]
Plataforma:     Meta Ads Manager | Google Ads
Nível:          campanha | conjunto | anúncio
Período:        [data início] → [data fim]

CAMPOS MAPEADOS
✅ investment    ← "[coluna identificada]"
✅ impressions   ← "[coluna identificada]"
✅ period_start  ← "[coluna identificada]"
✅ period_end    ← "[coluna identificada]"
   reach         ← "[coluna identificada]" | N/A — será null
   clicks        ← "[coluna identificada]" | N/A — será null
   leads         ← "[coluna identificada]" | N/A — será null

CAMPOS NULL (escopo de ingestão de CRM — não de plataforma)
—  mqls, sqls, negotiations, won, lost, revenue

COLUNAS IGNORADAS (derivados)
—  [lista]

Confirma o mapeamento? (sim / ajustar [campo → coluna])
```

Se o operador pedir ajuste, corrija o campo específico — não refaça a inspeção.

---

## Fase 4 — Geração do payload

Com o mapeamento confirmado, gere o JSON de configuração:

```json
{
  "job_id": "perf-ingest-[plataforma]-[YYYYMMDD-HHmm]",
  "source": {
    "platform": "meta" | "google",
    "file_path": "[path absoluto do CSV]",
    "level": "campaign" | "ad_set" | "ad",
    "period_start": "YYYY-MM-DD",
    "period_end": "YYYY-MM-DD"
  },
  "column_mapping": {
    "period_start": "[coluna]",
    "period_end": "[coluna ou null]",
    "campaign_ref": "[coluna com nome da campanha]",
    "ad_set_ref": "[coluna ou null]",
    "ad_ref": "[coluna ou null]",
    "investment": "[coluna]",
    "impressions": "[coluna]",
    "reach": "[coluna ou null]",
    "clicks": "[coluna ou null]",
    "leads": "[coluna ou null]"
  },
  "custom_mapping": {},
  "validator_endpoint": "/validate/performance_metrics",
  "on_duplicate": "upsert",
  "on_error": "log_and_continue"
}
```

Salve o arquivo de configuração:

```python
import json
payload = { ... }
with open('/tmp/perf-ingest-[timestamp].json', 'w') as f:
    json.dump(payload, f, indent=2)
```

---

## Fase 5 — Disparo do job

Acione o serviço via N8N webhook:

```bash
curl -s -X POST "http://localhost:5678/webhook/performance-ingest" \
  -H "Content-Type: application/json" \
  -d @/tmp/perf-ingest-[timestamp].json
```

Aguarde o retorno do job:

```json
{
  "job_id": "...",
  "status": "completed" | "partial" | "failed",
  "records_processed": N,
  "records_inserted": N,
  "records_updated": N,
  "records_skipped": N,
  "unresolved_campaigns": [...],
  "warnings": [...],
  "errors": [...]
}
```

---

## Fase 6 — Relatório ao operador

```
=== INGESTÃO CONCLUÍDA ===

Job:                   [job_id]
Plataforma:            [Meta / Google] — nível [campanha/conjunto/anúncio]
Período ingerido:      [início] → [fim]
Registros no CSV:      [total]
✅ Inseridos (novos):   [N]
🔄 Atualizados (upsert): [N]
⏭️  Ignorados:           [N]

[Se unresolved_campaigns:]
⚠️  CAMPANHAS NÃO ENCONTRADAS NO BANCO:
  - [nome da campanha]
  Causa: campanha não registrada via Guardião do Log.
  Ação: registre via Guardião do Log e re-ingeste apenas esse período.

[Se warnings:]
⚠️  AVISOS: [descrição] — [N registros afetados]

[Se erros:]
❌ ERROS: [descrição e ação]

Próximos passos:
→ performance_metrics populada para [período]
→ Acione o Data Miner com project_id e version_id para produzir o JSON de análise
→ Em seguida, acione o Growth Lead
```

---

## Tratamento de erros

### N8N inacessível
> "Não foi possível conectar ao N8N em `http://localhost:5678`. Verifique se o container está ativo: `docker ps`"

### N8N flow inexistente (webhook 404)
> "O flow `performance-ingest` ainda não existe no N8N. Este flow precisa ser criado — o payload gerado na Fase 4 define o contrato de entrada. Passe o payload para o Claude Code construir o flow."

### Validator endpoint inexistente (404 no Validator)
> "O endpoint `/validate/performance_metrics` ainda não existe no Validator API. Este endpoint precisa ser implementado. O payload gerado define o contrato esperado — passe para o Claude Code implementar."

### Colunas obrigatórias ausentes
Se `investment` ou `impressions` não mapearem, bloqueie imediatamente:
> "As colunas obrigatórias [X, Y] não foram encontradas nos headers. Verifique se o export foi gerado com as colunas corretas. Não é possível prosseguir."

### Campanha não encontrada no banco
Ingestão parcial é válida. Registre o warning, continue para os registros que resolvem.

---

## Anti-patterns

**Não faça:**
- Ler mais de `head -n 4` do CSV — mesmo que a identificação seja ambígua. Se não bastar, pergunte ao operador
- Assumir o mapeamento sem exibir a confirmação da Fase 3 — o operador precisa aprovar antes do disparo
- Disparar sem perguntar sobre re-ingestão do período — duplicatas silenciosas são o pior tipo de erro de dados
- Criar campanha no banco quando não encontrada — nunca. Oriente o operador a usar o Guardião do Log
- Atualizar `column-mappings.md` antes da ingestão ser confirmada bem-sucedida — só atualizar após HTTP 201

**Por quê cada anti-pattern importa:**
- Leitura em massa viola o princípio de que LLM não processa dados brutos
- Mapeamento sem confirmação gera dados incorretos silenciosamente
- Duplicatas de performance distorcem todas as análises do Growth Lead
- Criar campanha fora do fluxo do Guardião quebra a auditabilidade da taxonomia

---

## Regras invioláveis

1. **LLM lê no máximo `head -n 4` do CSV** — inspeciona estrutura, nunca dados em massa
2. **Nunca escreve no banco diretamente** — toda escrita é via data_miner → Validator
3. **Nunca persiste derivados** — CPL, ROAS, CTR, CPC são ignorados
4. **Nunca avança sem confirmação do mapeamento** — operador aprova antes do disparo
5. **Sempre pergunta sobre re-ingestão** — ingestão duplicada é erro de dados silencioso
6. **Ingestão parcial é comportamento válido** — registros com erro são logados, os demais são inseridos
7. **Schema drift → atualizar `column-mappings.md` após ingestão bem-sucedida** — nunca antes

---

## Cenários de avaliação

### Cenário 1 — Meta Ads, nível anúncio, export padrão

**Input:** Operador fornece CSV do Meta Ads Manager, granularidade por anúncio, período 7 dias, primeira vez ingerindo.

**Comportamento esperado:**
- [ ] Lê `referencias/column-mappings.md` antes de inspecionar o CSV
- [ ] Lê apenas `head -n 4` do arquivo
- [ ] Identifica plataforma Meta pelos headers
- [ ] Mapeia `Amount spent (BRL)` → investment, `Leads` → leads, `Link clicks` → clicks
- [ ] Pergunta sobre re-ingestão — operador confirma que é a primeira vez
- [ ] Exibe resumo do mapeamento e aguarda confirmação
- [ ] Salva JSON de configuração em `/tmp/`
- [ ] Aciona webhook N8N
- [ ] Exibe relatório com inseridos/atualizados/ignorados

### Cenário 2 — Google Ads sem coluna de data (período agregado)

**Input:** Export Google Ads sem coluna `Day`, dados agregados do mês inteiro.

**Comportamento esperado:**
- [ ] Identifica ausência de coluna de data
- [ ] Usa as datas dos pré-requisitos como `period_start` e `period_end` para todos os registros
- [ ] Menciona isso explicitamente no resumo da Fase 3
- [ ] Gera payload com `period_start` e `period_end` fixos (não mapeados de coluna)

### Cenário 3 — Campanha não registrada + re-ingestão de período

**Input:** Re-ingestão de período já ingerido + export contém uma campanha nova não registrada.

**Comportamento esperado:**
- [ ] Pergunta sobre re-ingestão — operador confirma que sim
- [ ] Informa que o data_miner fará upsert (não duplicará registros existentes)
- [ ] Após disparo, relatório mostra `records_updated` > 0 além de `records_inserted`
- [ ] Campanha não encontrada aparece em `unresolved_campaigns`
- [ ] Skill orienta a registrar via Guardião do Log e re-ingerir só esse período

### Cenário 4 — Infra ainda não construída

**Input:** N8N flow `performance-ingest` ou endpoint `/validate/performance_metrics` ainda não existem.

**Comportamento esperado:**
- [ ] Executa Fases 1–4 normalmente (independentes da infra)
- [ ] Na Fase 5, recebe 404
- [ ] Identifica se é o N8N (porta 5678) ou o Validator (porta 8000) pelo endpoint
- [ ] Informa qual componente está faltando
- [ ] Entrega o payload JSON gerado como insumo para o Claude Code construir a infra
- [ ] Não descarta o mapeamento — o trabalho das Fases 1–4 não se perde
