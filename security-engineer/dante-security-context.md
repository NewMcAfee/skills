# Dante — Contexto de Segurança

> Leia este arquivo antes de qualquer auditoria de segurança no ecossistema Dante.

## Arquitetura de Segurança

```
[N8N / LLM / Dashboard]
         |
    [Validator API]        ← única porta de entrada ao banco
    FastAPI + Pydantic     ← usa service_role_key (bypassa RLS)
         |
    [Supabase]             ← RLS ativado em todas as tabelas
    PostgreSQL             ← policies por org_id
```

**Princípio fundamental:** Nenhum cliente (N8N, LLM, dashboard React) acessa o Supabase diretamente. O Validator valida, transforma e autoriza antes de qualquer operação no banco.

---

## Credenciais e Chaves

| Credencial | Onde deve estar | Onde NUNCA deve estar |
|------------|----------------|----------------------|
| `SUPABASE_URL` | `.env` do Validator | código, log, chat |
| `SUPABASE_SERVICE_ROLE_KEY` | `.env` do Validator | frontend, N8N env visível, log |
| `SUPABASE_ANON_KEY` | `.env` do Dashboard (React) | log com payload, chat |
| `META_ACCESS_TOKEN` | `.env` do serviço CAPI | código, log, N8N expression visível |
| `META_PIXEL_ID` | `.env` | hardcoded em qualquer arquivo |
| Chaves N8N | `.env` | git, log |

**service_role_key vs anon_key:**
- `service_role_key` → bypassa RLS completamente. Só no Validator (backend). Vazamento = desastre multi-tenant.
- `anon_key` → respeita RLS. Pode estar no frontend com restrições. Nunca logar com payload de usuário.

---

## Row Level Security (RLS)

**Estado atual:** RLS ativado em todas as 14 tabelas por `org_id`.

**Padrão de policy:**
```sql
-- SELECT: org vê só seus dados
CREATE POLICY "org_select" ON tabela
  FOR SELECT USING (org_id = auth.jwt()->>'org_id');

-- INSERT: org insere só com seu org_id
CREATE POLICY "org_insert" ON tabela
  FOR INSERT WITH CHECK (org_id = auth.jwt()->>'org_id');
```

**Atenção crítica:**
- Nunca usar `user_metadata` em RLS — usuário autenticado pode alterar esse campo
- Sempre usar `auth.jwt()` claim específico ou `auth.uid()` com join em tabela de membros
- RLS desativado = qualquer org vê dados de todas as orgs (multi-tenant violado)

**Auditoria de migration:** Toda migration que toca schema deve incluir verificação:
```sql
SELECT tablename, rowsecurity FROM pg_tables 
WHERE schemaname = 'public' AND rowsecurity = false;
```
Resultado deve ser vazio. Se não for — alerta CRÍTICO.

---

## Dados CAPI (Meta Conversions API)

**Campos com PII que devem ser hasheados ANTES de enviar para Meta:**
- `em` → email: lowercase + strip → SHA256
- `ph` → phone: apenas dígitos, E.164 sem `+` → SHA256
- `fn` → first name: lowercase → SHA256
- `ln` → last name: lowercase → SHA256
- `ge` → gender: lowercase (`m`/`f`) → SHA256
- `db` → date of birth: `YYYYMMDD` → SHA256

**Campos que NÃO devem ser hasheados (Meta exige em claro):**
- `client_ip_address` — IP do usuário
- `client_user_agent` — user agent do browser
- `fbp` — cookie `_fbp` do Meta Pixel
- `fbc` — cookie `_fbc` / `fbclid` da URL

**No banco Dante:** PII armazenado em texto puro (para operabilidade). Hashing ocorre apenas no momento de envio para Meta.

**Acesso restrito:** `client_ip_address`, `client_user_agent`, `fbclid`, `fbp` são dados sensíveis — acesso via RLS restrito a `org_id` dono do dado. Nunca expor em APIs públicas ou logs.

---

## Componentes e Surface de Ataque

### Validator (FastAPI)
- Porta: interna (docker network `dante_internal`)
- Não exposto diretamente à internet — atrás de Nginx
- Usa `service_role_key` — toda escrita no banco passa aqui
- Pontos de atenção: input validation, SQL injection via parâmetros, rate limiting de endpoints críticos

### N8N
- Porta: deve estar atrás de auth (Basic Auth ou OAuth2)
- Desativar auth em produção é inaceitável — qualquer IP com acesso à porta pode acionar workflows
- Variáveis de ambiente do N8N são visíveis no painel se não configuradas como `secret` — atenção ao `META_ACCESS_TOKEN`

### Dashboard (React)
- Usa `anon_key` do Supabase — respeita RLS
- Nunca incluir `service_role_key` no bundle do frontend
- CORS configurado para domínio específico do dashboard, não `*`

### Supabase
- RLS é a barreira principal de multi-tenant
- Policies indexadas para evitar full scan (performance + surface de tempo de execução)
- Logs do Supabase podem conter queries — verificar se campos sensíveis aparecem em erros

---

## Multi-tenant: Invariante Crítica

Dante é projetado para múltiplas organizações. O isolamento é garantido por:

1. **RLS por `org_id`** — nível de banco, não pode ser bypassado sem `service_role_key`
2. **Validator filtra por `org_id`** do JWT — nível de aplicação
3. **Sem `service_role_key` no frontend** — nenhum cliente pode bypasassar RLS

**Risco principal:** Se o Validator escreve no banco sem incluir `org_id` no payload, a policy de INSERT falha ou insere com `org_id` errado. Toda query de escrita deve propagar `org_id` do JWT.

---

## LGPD / GDPR — Pontos Relevantes

- PII de usuários finais (leads dos clientes do Dante): email, phone, IP
- Base legal para CAPI: legítimo interesse ou consentimento do titular
- Retenção: não definida formalmente — atenção ao crescimento ilimitado de tabela de eventos
- Direito ao esquecimento: se solicitado, `client_ip_address` e dados de identificação devem ser removíveis
- Incidente de segurança: 72h para notificação à ANPD (Brasil) / autoridade competente (EU)
