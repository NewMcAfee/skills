# N8N — Padrões Aprovados e Rejeitados no Dante

## Padrões Aprovados

### P-01: Health Check antes de processar
**Contexto:** Qualquer flow que depende do Validator
**Padrão:**
```
Webhook → HTTP GET /validator/health (neverError: true, fullResponse: true)
  → IF statusCode === 200 → processar
  → IF statusCode !== 200 → Respond 503 imediatamente
```
**Por quê:** O Validator pode estar com cold start. Falhar rápido é melhor que timeout de 120s no caller.

---

### P-02: HTTP Request com captura manual de status
**Padrão:**
```json
{
  "options": {
    "response": {
      "response": {
        "neverError": true,
        "fullResponse": true
      }
    }
  }
}
```
**Por quê:** Com `neverError: false` (padrão), qualquer 4xx/5xx joga o flow para o Error Trigger sem chance de tratar. `neverError: true` permite IF no status code.

---

### P-03: Respond to Webhook em todos os branches
**Padrão:** Toda bifurcação (IF true + IF false) termina com Respond to Webhook.
**Por quê:** Branch sem Respond deixa o caller esperando até timeout (120s padrão do N8N).

---

### P-04: Code node para lógica com 3+ condicionais
**Quando usar:** Substituir 5+ nós nativos por 1 Code node quando a lógica tem condicionais aninhadas ou transformações complexas.
**Exemplo:**
```javascript
// Normalização de payload — substituindo Set + IF + IF + Set
const raw = items[0].json;
return [{
  json: {
    campaign_name: raw.campaign_name?.trim().toUpperCase(),
    project_id: raw.project_id,
    type: raw.type || 'campaign',
    validated_at: new Date().toISOString()
  }
}];
```

---

### P-05: Idempotência via Postgres (Postgres-based, sem Redis)
**Padrão:**
1. Code node: extrair/gerar `idempotency_key`
2. Postgres node: `SELECT id FROM webhook_log WHERE idempotency_key = $1`
3. IF: rows > 0 → Respond 200 (skip)
4. IF: rows = 0 → processar + `INSERT INTO webhook_log (...)`

---

### P-06: Error Trigger global + notificação
**Padrão:** Todos os flows de produção apontam para `dante-error-handler` em Settings.
**dante-error-handler.json:**
```
Error Trigger
  → Code node: formatar mensagem com $execution.error + $workflow.name
  → Postgres: INSERT INTO execution_errors
  → HTTP Request: POST Slack webhook
```

---

### P-07: IF encadeado em vez de Switch
**Contexto:** N8N 2.13.4 não suporta Switch typeVersion 3.2
**Padrão:**
```
IF: condição A → true: caminho A
                false → IF: condição B → true: caminho B
                                         false: caminho default
```

---

## Padrões Rejeitados

### R-01: Switch typeVersion 3.2
**Problema:** Não existe no N8N 2.13.4. Importa sem erro, falha silenciosamente em runtime.
**Substituto:** IF encadeados (P-07).

---

### R-02: Escrita direta no Supabase via nó Postgres
**Problema:** Bypassa toda a camada de validação do Validator (Pydantic, taxonomia, FK check).
**Substituto:** HTTP POST para `http://nginx/validator/<endpoint>`.

---

### R-03: Credencial hardcoded em Code node
**Problema:** Exposta em logs de execução e no JSON exportado para Git.
**Substituto:** `$env.NOME_DA_VAR` ou credencial cadastrada no N8N e referenciada pelo nó.

---

### R-04: Webhook síncrono sem timeout no HTTP Request
**Problema:** Validator pode demorar na inicialização. Sem timeout, o flow trava por 120s.
**Substituto:** Sempre setar `timeout: 10000` (10s) em HTTP Requests para serviços internos.

---

### R-05: Loop Over Items sem validação do input
**Problema:** Se `items` vier vazio ou malformado, o loop pode processar 0 vezes (ok) ou entrar em estado inconsistente.
**Substituto:** Code node antes do loop validando que `items.length > 0` e que a estrutura esperada está presente.

---

### R-06: Flow sem Error Workflow configurado
**Problema:** Falhas de produção passam silenciosamente sem alerta.
**Substituto:** Configurar Error Workflow → `dante-error-handler` em todo flow ativado.

---

### R-07: Lógica de negócio no nó Webhook Trigger
**Problema:** Webhook Trigger só recebe e roteia — qualquer expressão complexa no `path` ou `responseMode` mistura responsabilidades e dificulta debug.
**Substituto:** Webhook Trigger limpo → Code node de normalização → IF de guarda.
