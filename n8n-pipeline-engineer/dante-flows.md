# Dante — Contexto de Flows N8N

## Infraestrutura

| Item | Valor |
|------|-------|
| URL | `https://dante-n8n.thegrowthsensei.com.br` |
| Container | `dante_n8n` |
| Porta interna | 5679 |
| Rede Docker | `dante_internal` |
| Versão N8N | **2.13.4** |

**IMPORTANTE:** Não confundir com o N8N Geral em `n8n.thegrowthsensei.com.br` — são instâncias separadas.

---

## Topologia de Rede Interna

```
[Chamador externo]
       ↓ HTTPS
    [Nginx]
       ↓ http://nginx/validator/   (rede dante_internal)
    [dante_validator]  ← FastAPI na porta 8000
       ↓ Supabase Postgres (pooler aws-1, Session Mode)
    [Supabase]

[N8N] → acessa Validator em http://nginx/validator/
[N8N] → acessa Supabase Postgres diretamente APENAS para leituras (idempotency, lookup)
```

**Regra de ouro:** Todo write passa pelo Validator. Leituras via nó Postgres são permitidas.

---

## Flows Existentes

### `guardiao-validator.json`
- **Trigger:** POST `/webhook/guardiao`
- **Função:** Recebe logs do Guardião do Log → health check no Validator → valida payload → persiste no Supabase
- **Modo:** Síncrono (`responseMode: responseNode`)
- **Padrão de nós:** Webhook → Health Check → IF Healthy → [POST Validator] → IF 200 → Respond

### `performance-ingest.json`
- **Trigger:** POST `/webhook/performance-ingest`
- **Função:** Ingere dados de performance de mídia paga → normaliza → envia ao Validator
- **Modo:** Síncrono

### `data-analysis.json`
- **Trigger:** Schedule Trigger
- **Função:** Análise periódica de dados — agrega métricas, gera relatórios
- **Modo:** Assíncrono (sem Respond to Webhook)

---

## Validator — Endpoints Relevantes

Base URL interna: `http://nginx/validator`

| Endpoint | Método | Descrição |
|----------|--------|-----------|
| `/health` | GET | Health check — retorna 200 se ok |
| `/log` | POST | Persiste log do Guardião |
| `/performance` | POST | Persiste dados de performance |
| `/campaign` | POST | Valida e persiste campanha |

Todos retornam `{error, field, suggestion}` em caso de falha (422) — use o corpo diretamente no Respond to Webhook.

---

## Credenciais Configuradas no N8N Dante

| Nome da credencial | Tipo | Uso |
|--------------------|------|-----|
| `Supabase Postgres` | Postgres | Leituras diretas (idempotency, lookup) |
| Supabase REST (se configurado) | HTTP Header Auth | NÃO usar — passe pelo Validator |

**Session Mode** no pooler Supabase: suporta prepared statements (`$1, $2, ...`). Use sempre queries parametrizadas.

---

## Constraints de Versão — N8N 2.13.4

### Nós com typeVersion correto

```json
"type": "n8n-nodes-base.webhook",        "typeVersion": 2
"type": "n8n-nodes-base.httpRequest",    "typeVersion": 4.2
"type": "n8n-nodes-base.if",             "typeVersion": 2.1
"type": "n8n-nodes-base.postgres",       "typeVersion": 2.5
"type": "n8n-nodes-base.code",           "typeVersion": 2
"type": "n8n-nodes-base.respondToWebhook", "typeVersion": 1.1
"type": "n8n-nodes-base.set",            "typeVersion": 3.4
"type": "n8n-nodes-base.splitInBatches", "typeVersion": 3
"type": "n8n-nodes-base.executeWorkflow","typeVersion": 1.1
"type": "n8n-nodes-base.scheduleTrigger","typeVersion": 1.1
"type": "n8n-nodes-base.errorTrigger",   "typeVersion": 1
"type": "n8n-nodes-base.merge",          "typeVersion": 3
```

### Bug conhecido — Switch typeVersion 3.2

O nó Switch com typeVersion 3.2 **não existe** no N8N 2.13.4. Se um JSON importado contém esse nó, o workflow importa sem erro mas falha silenciosamente ao executar.

**Substituto obrigatório:** IF encadeados.

```
IF condição A → true path A
  false → IF condição B → true path B
    false → default path
```

---

## Deploy — Procedimento Completo

### Importar flow existente

```bash
# Flow já presente no import-flow.sh:
bash /root/dante/n8n/import-flow.sh

# Qualquer flow:
docker exec dante_n8n n8n import:workflow \
  --input=/home/node/flows/<nome>.json
```

O volume Docker mapeia `/root/dante/n8n/flows/` → `/home/node/flows/` no container.

### Exportar flow para versionamento

```bash
# Listar IDs dos flows ativos:
docker exec dante_n8n n8n list:workflow

# Exportar:
docker exec dante_n8n n8n export:workflow \
  --id=<ID> \
  --output=/home/node/flows/<nome>.json

# Copiar para host (se necessário):
# O volume já sincroniza — o arquivo estará em /root/dante/n8n/flows/
```

### Verificar saúde do N8N

```bash
docker ps --filter name=dante_n8n
curl http://localhost:5679/healthz
```

---

## Padrão de Idempotência (Postgres-based)

Sem Redis no Dante, a idempotência usa Postgres:

```sql
-- Tabela de controle (criar se não existir):
CREATE TABLE IF NOT EXISTS webhook_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  idempotency_key TEXT NOT NULL,
  workflow_name TEXT NOT NULL,
  status TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE UNIQUE INDEX IF NOT EXISTS idx_webhook_log_key 
  ON webhook_log(idempotency_key);
```

**Flow N8N:**
1. Code node extrai `idempotency_key` do payload
2. Postgres node: `SELECT id FROM webhook_log WHERE idempotency_key = $1`
3. IF: resultado encontrado → Respond 200 (já processado)
4. IF: não encontrado → Postgres INSERT + processar + Respond

---

## Error Workflow Global — dante-error-handler

Configurar em todos os flows de produção:
- Settings → Error Workflow → selecionar `dante-error-handler`

Estrutura do `dante-error-handler.json`:
```
Error Trigger
  → Set: extrair $execution.error, $workflow.name, $execution.id
  → Postgres: INSERT INTO execution_errors (...)
  → HTTP Request: POST Slack webhook com contexto
```
