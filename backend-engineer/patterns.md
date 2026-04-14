# Padrões Aprovados e Rejeitados — Backend Engineer Dante

Cada entrada documenta uma decisão técnica com justificativa. Use para calibrar recomendações futuras.

---

## APROVADOS

### P001 — httpx.AsyncClient compartilhado no lifespan
**Decisão:** Instanciar `SupabaseClient` uma vez no lifespan do FastAPI e injetar via `request.app.state.db`.
**Porquê:** Reutiliza pool de conexões TCP. Criar client por request é 3-5x mais lento e desperdiça file descriptors.
**Padrão:**
```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.db = SupabaseClient()  # uma vez
    yield
    await app.state.db.close()       # cleanup no shutdown
```

### P002 — Remover None antes de POST no Supabase
**Decisão:** `{k: v for k, v in payload.items() if v is not None}` antes de enviar.
**Porquê:** Supabase REST aplica DEFAULT nos campos ausentes. Enviar `null` explicitamente sobrescreve DEFAULT — comportamento inesperado em campos como `launched_at`.

### P003 — ParseResult como dataclass para validação de taxonomia
**Decisão:** Validadores de nome retornam `ParseResult(valid, fields, error, field, suggestion)`.
**Porquê:** Separa parsing de validação. Permite testar parsing independente. Evita raise/catch em lógica de negócio — o raise fica no handler do endpoint.

### P004 — `str | None = None` para opcionais em Pydantic v2
**Decisão:** Usar `str | None = None` em vez de `Optional[str] = None`.
**Porquê:** `Optional` é alias de `Union[X, None]` — verboso e Pydantic v1. Com Pydantic v2, `str | None` é o padrão idiomático e mais rápido de parsear.

### P005 — service_role_key em todos os requests ao Supabase
**Decisão:** Validator usa `SUPABASE_SERVICE_ROLE_KEY` em header `apikey` E `Authorization`.
**Porquê:** service_role_key ignora RLS — necessário para operações de write que LLMs disparam. anon_key seria bloqueada pelas policies.

### P006 — CLIs Python sem async
**Decisão:** miner.py usa `requests` (síncrono), não `httpx` async.
**Porquê:** CLI roda em processo único, sem concorrência. Async seria overhead de event loop sem benefício. Sync é mais simples de debugar e o N8N não se beneficia de paralelismo no Execute Command.

### P007 — Exclusão hierárquica em ad_sets
**Decisão:** `excluded_hierarchy_digits` calculado pelo Validator na criação do ad_set.
**Porquê:** Garante integridade da hierarquia de audiência independente de quem cria. AUD-7 sempre exclui 0-6 — regra de negócio que não pode depender do cliente.

---

## REJEITADOS

### R001 — Endpoint GET com body
**Proposta:** GET /lookup/campaign com body JSON para evitar query params longos.
**Rejeição:** GET com body viola RFC 9110. Alguns clientes (incluindo httpx por default) ignoram body em GET. Query params têm limite de ~8KB — suficiente para UUIDs.
**Alternativa:** Query params para lookup, POST apenas para operações com side effects.

### R002 — Autenticação por token no Validator
**Proposta:** Adicionar Bearer token na API do Validator para segurança.
**Rejeição:** Validator roda exclusivamente na rede `dante_internal` — sem exposição externa. Adicionar autenticação é complexidade sem modelo de ameaça justificado. Se a rede interna for comprometida, token não protege nada.
**Condição de revisão:** Se o Validator precisar ser exposto externamente (não planejado).

### R003 — Múltiplas instâncias do Validator com workers
**Proposta:** `uvicorn main:app --workers 4` para aumentar throughput.
**Rejeição:** Validator é stateless exceto pelo `SupabaseClient` no `app.state`. Com workers, cada processo tem seu próprio client — multiplicar conexões ao Supabase sem necessidade. Solução correta: um worker com async (já implementado) aguenta >200 req/s para esse volume.

### R004 — Retry automático no Validator para erros do Supabase
**Proposta:** Retry com backoff no `_insert` quando Supabase retorna 503.
**Rejeição:** Validator não deve tentar mascarar indisponibilidade do Supabase. O LLM que chama o Validator precisa saber que o banco está fora para tomar decisão (tentar depois, alertar usuário). Retry silencioso atrasa o feedback loop.
**Alternativa:** Retry deve estar no N8N (retry de workflow), não no Validator.

### R005 — Escrever diretamente na REST API do Supabase de workflow N8N
**Proposta:** N8N HTTP Request direto para `supabase.co/rest/v1/campaigns` sem passar pelo Validator.
**Rejeição:** Bypassa validação de schema, taxonomia e FK. Um registro inválido no banco quebra todos os relatórios downstream. O Validator existe exatamente para prevenir isso.

### R006 — Cache em memória de lookups no Validator
**Proposta:** Dict em memória para cachear resultados de `record_exists`.
**Rejeição:** Validator pode ter múltiplos workers (futuro). Estado em memória não é compartilhado entre processos. Além disso, Supabase tem latência <5ms na rede interna — cache não resolve problema real de performance para esse volume.

---

## EM AVALIAÇÃO

### E001 — Paginação em endpoints de lookup
**Contexto:** Lookup retorna primeiro match. Se projeto tiver nomes ambíguos, pode retornar resultado errado.
**Status:** Aguarda evidência de colisão em produção antes de adicionar complexidade.
**Critério de decisão:** >1 caso reportado de match errado por ambiguidade de nome.

### E002 — Webhook do Supabase para invalidar lookups
**Contexto:** Se uma campanha é deletada/arquivada, lookup ainda a encontra.
**Status:** Aguarda definição de política de arquivamento de dados (ainda não existe).
