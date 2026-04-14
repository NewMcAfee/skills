# Padrões de Migration — Data Engineer Dante

Padrões documentados para migrations seguras no Supabase/PostgreSQL em produção.

---

## PADRÕES SEGUROS

### M001 — ADD COLUMN nullable (sem lock)
```sql
-- 006_exemplo.sql
-- Adicionar coluna opcional: instantâneo, sem lock
ALTER TABLE performance_metrics 
ADD COLUMN IF NOT EXISTS source_platform TEXT;

-- Se precisar NOT NULL, fazer em 3 passos:
-- Passo 1: ADD nullable (sem lock)
ALTER TABLE performance_metrics ADD COLUMN IF NOT EXISTS source_platform TEXT;
-- Passo 2: backfill (executar em batches se tabela grande)
UPDATE performance_metrics SET source_platform = 'meta' WHERE canal = 'META' AND source_platform IS NULL;
UPDATE performance_metrics SET source_platform = 'google' WHERE canal = 'GOOG' AND source_platform IS NULL;
-- Passo 3: NOT NULL constraint (lock curto via NOT VALID trick em PG 18+)
ALTER TABLE performance_metrics ALTER COLUMN source_platform SET NOT NULL;
```

### M002 — ADD ENUM VALUE (sem rebuild)
```sql
-- PostgreSQL permite adicionar ENUM values sem rebuild completo
ALTER TYPE tracking_completeness ADD VALUE IF NOT EXISTS 'third_party';
-- CUIDADO: não é reversível sem DDL complexo. Confirme o valor antes.
```

### M003 — CREATE INDEX CONCURRENTLY (sem lock de write)
```sql
-- Sempre CONCURRENTLY em tabela com dados
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_metrics_source_platform
ON performance_metrics(project_id, source_platform)
WHERE source_platform IS NOT NULL;

-- Para reconstruir índice existente sem lock:
REINDEX CONCURRENTLY INDEX idx_metrics_project_period;
```

### M004 — Nova tabela com padrão Dante completo
```sql
-- Template: nova tabela seguindo padrão 004/005
CREATE TABLE IF NOT EXISTS nova_tabela (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE RESTRICT,
  version_id UUID NOT NULL REFERENCES versions(id) ON DELETE RESTRICT,
  icp_product_map_id UUID REFERENCES icp_product_map(id) ON DELETE RESTRICT,  -- se dados de performance
  
  -- campos de negócio
  nome TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'draft',
  
  -- auditoria
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Trigger de updated_at (padrão Dante)
CREATE TRIGGER set_updated_at_nova_tabela
  BEFORE UPDATE ON nova_tabela
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Índices obrigatórios
CREATE INDEX IF NOT EXISTS idx_nova_tabela_project ON nova_tabela(project_id, status);

-- RLS obrigatória
ALTER TABLE nova_tabela ENABLE ROW LEVEL SECURITY;
CREATE POLICY nova_tabela_project_policy ON nova_tabela
  USING (has_project_access(project_id));
```

### M005 — CHECK CONSTRAINT segura (NOT VALID)
```sql
-- Adicionar constraint sem varrer dados existentes imediatamente
ALTER TABLE governance_events 
ADD CONSTRAINT chk_urgency_range CHECK (urgency BETWEEN 1 AND 5) NOT VALID;

-- Validar em transação separada (SHARE UPDATE EXCLUSIVE — permite reads/writes)
ALTER TABLE governance_events VALIDATE CONSTRAINT chk_urgency_range;
```

### M006 — Renomear coluna com expand-contract
```sql
-- NUNCA: ALTER TABLE RENAME COLUMN em tabela com FK ou em produção ativa
-- CORRETO: expand-contract

-- Passo 1: ADD nova coluna
ALTER TABLE performance_metrics ADD COLUMN canal_v2 TEXT;

-- Passo 2: backfill
UPDATE performance_metrics SET canal_v2 = canal::TEXT WHERE canal_v2 IS NULL;

-- Passo 3: atualizar aplicação para escrever em canal_v2
-- Passo 4: atualizar aplicação para ler de canal_v2

-- Passo 5 (depois de validar): DROP coluna antiga
ALTER TABLE performance_metrics DROP COLUMN canal;  -- apenas após validação completa
```

---

## PADRÕES PERIGOSOS (documentados para rejeitar)

### D001 — ADD COLUMN NOT NULL sem DEFAULT em tabela com dados
```sql
-- ERRADO: trava access exclusive até reescrever TODA a tabela
ALTER TABLE performance_metrics ADD COLUMN canal_v2 TEXT NOT NULL;
-- Em tabela com 1M linhas: pode durar minutos. Todos os writes bloqueados.
```

### D002 — CREATE INDEX sem CONCURRENTLY
```sql
-- ERRADO: bloqueia writes pelo tempo do build do índice
CREATE INDEX idx_novo ON performance_metrics(project_id, canal);
-- Corrigir: adicionar CONCURRENTLY
```

### D003 — DROP COLUMN sem verificar dependências
```sql
-- ERRADO: verificar antes se há views, functions, ou aplicação usando
ALTER TABLE performance_metrics DROP COLUMN canal;
-- Checar antes: SELECT * FROM information_schema.columns WHERE column_name = 'canal';
-- E: SELECT * FROM pg_depend WHERE ... (para views/functions)
```

### D004 — Alterar ENUM existente (remover valor)
```sql
-- ERRADO: não existe DROP VALUE no PostgreSQL
-- Para remover valor de ENUM: rebuild completo da tabela
-- Alternativa: migrar para TEXT com CHECK constraint
```

### D005 — Alterar migration já aplicada
```sql
-- ERRADO: editar 002_performance_metrics_crm.sql depois de aplicada
-- Migrations 001-005 são contratos imutáveis.
-- CORRETO: criar 006_correcao.sql
```

### D006 — DROP TABLE com dados de produção
```sql
-- SEMPRE confirmar com usuário antes de DROP TABLE
-- Se precisar "remover" tabela: renomear para _deprecated_ e arquivar
ALTER TABLE tabela_antiga RENAME TO _deprecated_tabela_antiga;
```

---

## TEMPLATE DE MIGRATION

```sql
-- NNN_descricao_curta.sql
-- Data: YYYY-MM-DD
-- Objetivo: [uma linha descrevendo o objetivo]
-- Reversão: [como desfazer — ou "não reversível: campo removido"]
-- Dependências: [migrations anteriores que precisam estar aplicadas]

BEGIN;

-- [SQL da migration]

-- Verificação (opcional mas recomendada)
DO $$
BEGIN
  -- Assert que a migration foi aplicada corretamente
  IF NOT EXISTS (SELECT 1 FROM information_schema.columns 
                 WHERE table_name = 'X' AND column_name = 'Y') THEN
    RAISE EXCEPTION 'Migration falhou: coluna Y não encontrada em X';
  END IF;
END $$;

COMMIT;
```

---

## CHECKLIST PRÉ-MIGRATION

Antes de escrever qualquer migration para produção:

```
[ ] Número sequencial: próxima é 006_
[ ] Nome descritivo: 006_adiciona_canal_v2_performance_metrics.sql
[ ] BEGIN/COMMIT wrapping
[ ] IF NOT EXISTS / IF EXISTS em todos os DDL
[ ] ADD COLUMN: nullable ou com DEFAULT
[ ] ÍNDICES: CONCURRENTLY
[ ] CONSTRAINTS: NOT VALID + VALIDATE separado se tabela grande
[ ] RLS: nova tabela tem ENABLE ROW LEVEL SECURITY + POLICY
[ ] Trigger updated_at: nova tabela tem trigger padrão
[ ] icp_product_map_id: tabela de análise referencia o eixo central
[ ] Rollback documentado no cabeçalho
[ ] Testado em schema de staging antes de produção
```
