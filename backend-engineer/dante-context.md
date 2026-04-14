# Dante — Contexto Arquitetural para Backend Engineer

## Princípio Central

**O Validator é a lei.** Nada escreve no Supabase sem passar por ele. Qualquer proposta que bypasse o Validator é recusada imediatamente — não importa o motivo de conveniência.

---

## Mapa de Serviços

```
N8N (dante_n8n:5678)
  ↓ HTTP interno
Nginx (dante_nginx:80)  →  /validator/* → validator:8000
  ↓
Validator (dante_validator:8000) [FastAPI]
  ↓ httpx.AsyncClient (service_role_key)
Supabase REST API
  ↓
PostgreSQL (14 tabelas)
```

**Data Miner (miner.py)**: CLI Python, acionado via N8N Execute Command. Não é um daemon. Lê CSVs de plataforma (Meta/Google) e CRM (HubSpot). Chama o Validator via HTTP para lookup e ingestão.

**Meta Connector**: CLI Python, mesmo padrão do miner.

---

## Validator — Padrões de Código

### Estrutura de endpoint padrão

```python
@app.post("/validate/campaign", status_code=201)
async def create_campaign(data: CampaignInput, request: Request):
    db: SupabaseClient = request.app.state.db
    payload = await validate_campaign(data, db)
    result = await db.insert_campaign(payload)
    return {"created": result}
```

**Regras:**
- `status_code=201` em POST de criação
- `db` vem do `request.app.state` — injetado no lifespan, não instanciado por request
- `async def` obrigatório — todas as chamadas são await
- Retorna `{"created": result}` para criação, objeto direto para lookup

### Modelos Pydantic

```python
class CampaignInput(BaseModel):
    campaign_name: str
    project_id: str
    version_id: str
    icp_product_map_id: str
    platform_id: str | None = None      # opcionais com None explícito
    launched_at: str | None = None
    notes: str | None = None
```

**Regras:**
- `str | None = None` para opcionais — nunca `Optional[str]` (legacy Pydantic v1)
- Campos de data como `str` — o Supabase valida formato DATE/TIMESTAMP
- IDs como `str` — UUIDs são strings para o Pydantic

### Validação 3 camadas

```python
async def validate_campaign(data: CampaignInput, db: SupabaseClient) -> dict:
    # Camada 1: schema já foi validado pelo Pydantic na entrada do endpoint

    # Camada 2: enum/taxonomia
    parse_result = parse_campaign_name(data.campaign_name)
    if not parse_result.valid:
        raise ValidationError(parse_result.error, parse_result.field, parse_result.suggestion)

    # Camada 3: FKs no banco
    if not await db.record_exists("icp_product_map", data.icp_product_map_id):
        raise ValidationError("icp_product_map_id não encontrado", "icp_product_map_id",
                               "Crie o icp_product_map antes de criar a campanha")

    return {**parse_result.fields, "project_id": data.project_id, ...}
```

### Tratamento de erro padrão

```python
# Exceção de domínio
class ValidationError(Exception):
    def __init__(self, error: str, field: str, suggestion: str):
        self.error = error
        self.field = field
        self.suggestion = suggestion

class DbError(Exception):
    def __init__(self, error: str, field: str, suggestion: str): ...

# Handlers registrados no app
@app.exception_handler(ValidationError)
async def validation_handler(request, exc):
    return JSONResponse(status_code=422,
                        content={"error": exc.error, "field": exc.field, "suggestion": exc.suggestion})

@app.exception_handler(DbError)
async def db_handler(request, exc):
    return JSONResponse(status_code=422, ...)

@app.exception_handler(Exception)
async def generic_handler(request, exc):
    return JSONResponse(status_code=500,
                        content={"error": "Erro interno", "field": "", "suggestion": "Contate suporte"})
```

**Nunca** retornar stacktrace. **Sempre** `{error, field, suggestion}`.

### Cliente Supabase

```python
class SupabaseClient:
    def __init__(self):
        self.base_url = os.getenv("SUPABASE_URL")
        self.headers = {
            "apikey": os.getenv("SUPABASE_SERVICE_ROLE_KEY"),
            "Authorization": f"Bearer {os.getenv('SUPABASE_SERVICE_ROLE_KEY')}",
            "Content-Type": "application/json",
            "Prefer": "return=representation",
        }
        self.client = httpx.AsyncClient(base_url=self.base_url, headers=self.headers)

    async def _insert(self, table: str, payload: dict) -> dict:
        clean = {k: v for k, v in payload.items() if v is not None}  # remove None → Supabase usa DEFAULT
        resp = await self.client.post(f"/rest/v1/{table}", json=clean)
        if resp.status_code not in (200, 201):
            self._handle_pg_error(resp.json())
        return resp.json()[0]
```

**Regras:**
- `service_role_key` sempre — ignora RLS, tem permissão total
- `Prefer: return=representation` — retorna o registro inserido
- Remover `None` antes de POST — Supabase aplica DEFAULT nos campos ausentes
- `_handle_pg_error` mapeia PG error codes para `DbError` com mensagens legíveis

---

## Schema de Tabelas (referência rápida)

| Tabela | Chaves principais | Restrições importantes |
|--------|-------------------|----------------------|
| `organizations` | id, name, slug | slug UNIQUE |
| `projects` | id, org_id, name | org_id FK |
| `versions` | id, project_id, version_number | project_id FK |
| `icp_product_map` | id, project_id, icp_id, product_id | (icp_id, product_id) UNIQUE por versão |
| `campaigns` | id, project_id, version_id | campaign_name UNIQUE por projeto |
| `ad_sets` | id, campaign_id, project_id | ad_set_name UNIQUE por projeto |
| `ads` | id, ad_set_id, project_id, icp_product_id | ad_name UNIQUE por projeto |
| `performance_metrics` | id, project_id, version_id | (period_start, period_end, granularity, level, campaign_id, ad_set_id, ad_id) UNIQUE |

Hierarquia: `organizations → projects → versions → icp_product_map → campaigns → ad_sets → ads`

---

## Data Miner — Padrões de CLI

```python
# Argparse padrão
parser = argparse.ArgumentParser()
parser.add_argument("--mode", type=int, choices=[1, 2, 3], required=True)
parser.add_argument("--platform", choices=["meta", "google"], required=True)
parser.add_argument("--file", required=True)
parser.add_argument("--project-id", required=True)
parser.add_argument("--version-id", required=True)
parser.add_argument("--validator-url", default="http://validator:8000")

# Chamadas ao Validator
def lookup_campaign(name, project_id, validator_url):
    resp = requests.get(f"{validator_url}/lookup/campaign",
                        params={"name": name, "project_id": project_id})
    if resp.status_code == 404:
        return None
    return resp.json()
```

**Regras:**
- `requests` (síncrono) — CLI roda em thread única, não precisa de async
- `--validator-url` configurável — padrão aponta para `validator:8000` na rede interna
- Logs para stdout — N8N captura via Execute Command
- Relatório final em JSON para stdout — N8N pode parsear

---

## Infra Docker

```yaml
# Rede interna — sem exposição direta
networks:
  dante_internal:
    driver: bridge

# Serviços novos entram assim:
novo_servico:
  build: ../services/novo_servico
  container_name: dante_novo_servico
  env_file: ../services/novo_servico/.env
  expose: [8001]                  # não "ports" — só interno
  networks: [dante_internal]
  depends_on: [validator]         # se depende do Validator
```

**Regras:**
- `expose` (interno) ≠ `ports` (externo). Serviços Python usam `expose`.
- N8N usa `extra_hosts` para resolver DNS do Supabase como IPv4. Serviços novos podem precisar do mesmo.
- Health check obrigatório em daemons. CLIs não precisam.
- `restart: unless-stopped` para daemons, não para CLIs.

---

## ENUMs de Taxonomia

```python
# Canal
CanalType: META, GOOG, LINK, TIKT

# Funil
FunilType: COLD, WARM, HOT

# Formato
FormatoType: STA, VID, CAR, MOT, MIX

# Consciência
ConscienciaType: UNC, PRB, SOL, PRO, MIX

# Gancho
GanchoType: Loss, Proof, Story, Error, Fact, Contr
```

Taxonomia completa em `/root/dante/services/validator/taxonomy.py`.

---

## Dependências atuais do Validator

```
fastapi==0.115.6
uvicorn[standard]==0.32.1
pydantic==2.10.4          # v2 — usar str | None, não Optional[str]
httpx==0.28.1             # async HTTP client
python-dotenv==1.0.1
```

Adicionar dependência nova requer rebuild do container. Custo real — considere antes de recomendar.
