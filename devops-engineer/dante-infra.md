# Dante — Referência de Infraestrutura

**Atualizado:** 2026-04-12

---

## VPS

| Campo | Valor |
|-------|-------|
| IP | `187.127.0.135` |
| OS | Ubuntu (Linux) |
| Root do projeto | `/root/dante/` |
| Usuário operacional | `root` |

---

## Containers Dante (projeto principal)

| Container | Imagem | Porta externa | Porta interna | Rede |
|-----------|--------|--------------|---------------|------|
| `dante_n8n` | `n8nio/n8n:latest` | `5679` | `5678` | `dante_internal` |
| `dante_validator` | build local (FastAPI) | — (expose only) | `8000` | `dante_internal` |
| `dante_nginx` | `nginx:1.27-alpine` | — (expose only) | `80` | `dante_internal` |

**Outros containers no VPS (não gerenciados pelo compose do Dante):**

| Container | Porta | Projeto |
|-----------|-------|---------|
| `n8n-n8n-1` | `5678` | TheGrowthSensei N8N (separado) |
| `nginx-proxy-manager-app-1` | `80`, `81`, `443` | NPM Global — SSL e proxy reverso |

> Atenção: porta `5679` foi escolhida para o dante_n8n justamente para evitar conflito com o `n8n-n8n-1` que ocupa a `5678`.

---

## Rede Docker

- **Nome:** `dante_internal`
- **Driver:** `bridge`
- **IPv6:** não configurado
- **Resolução DNS interna:** containers se encontram pelo nome do serviço no compose (ex: `http://validator:8000`, `http://nginx/validator/`)
- **Armadilha IPv6:** redes Docker bridge sem IPv6 causam `ENETUNREACH` quando o DNS de destino resolve para IPv6 (problema documentado com Supabase sem pooler)

---

## Nginx Proxy Manager (NPM)

- **UI Admin:** `http://187.127.0.135:81`
- **Proxy hosts ativos:**
  - ID 1: `n8n.thegrowthsensei.com.br` + `thegrowthsensei.com.br` → `:5678` (SSL ativo, cert ID 4)
  - ID 2: `dante-n8n.thegrowthsensei.com.br` → `:5679` (SSL ativo, cert ID 6, expira 2026-07-11, HTTPS forçado)
- **Dados persistidos:** `/root/nginx-proxy-manager/data/`
- **Certificados:** `/root/nginx-proxy-manager/letsencrypt/`

---

## Nginx Interno (dante_nginx)

Arquivo de config: `/root/dante/infrastructure/nginx/dante.conf`

```nginx
server {
    listen 80;
    server_name _;

    location /validator/ {
        proxy_pass http://validator:8000/;
        proxy_read_timeout 30s;
        proxy_connect_timeout 5s;
    }

    location /nginx-health {
        return 200 "ok\n";
    }
}
```

Acesso interno: `http://nginx/validator/`
Health check: `http://nginx/nginx-health`

---

## Estrutura de Diretórios no VPS

```
/root/
├── dante/
│   ├── infrastructure/
│   │   ├── docker-compose.yml     ← orquestra dante_n8n, dante_validator, dante_nginx
│   │   └── nginx/dante.conf       ← config do nginx interno
│   ├── services/
│   │   ├── validator/             ← FastAPI — Dockerfile + .env aqui
│   │   └── data_miner/            ← CLI Python — acionado via N8N Execute Command
│   └── n8n/flows/                 ← flows exportados (montado :ro no container)
├── n8n/docker-compose.yml         ← N8N do TheGrowthSensei (stack separada)
└── nginx-proxy-manager/
    ├── docker-compose.yml
    ├── data/                       ← SQLite do NPM + configs nginx geradas
    └── letsencrypt/                ← certificados SSL
```

---

## docker-compose.yml — Configurações Críticas

Localização: `/root/dante/infrastructure/docker-compose.yml`

**Configurações não óbvias a preservar:**

| Config | Valor | Motivo |
|--------|-------|--------|
| Porta N8N | `5679:5678` | Evita conflito com n8n-n8n-1 na 5678 |
| `restart` | `unless-stopped` | Respeita parada intencional |
| `N8N_BASIC_AUTH_ACTIVE` | `false` | HTTPS via NPM é a barreira de acesso |
| `N8N_SECURE_COOKIE` | `false` | Necessário quando NPM faz SSL termination |
| `extra_hosts` | `aws-1-sa-east-1.pooler.supabase.com:18.228.163.245` | Fix IPv4 — veja seção Supabase |
| Volume flows | `../n8n/flows:/home/node/flows:ro` | `:ro` obrigatório — flows não devem ser editados no container |
| Validator porta | apenas `expose: "8000"` | Sem `ports:` — acesso apenas via nginx interno |

---

## Supabase — Conexão e Quirks

**Problema raiz:** O DNS do Supabase resolve para IPv6 por padrão. A rede `dante_internal` (bridge sem IPv6) não tem rota para IPv6 externo → erro `ENETUNREACH`.

**Solução implementada:** Connection Pooler `aws-1` com IP fixado via `extra_hosts`.

| Campo | Valor |
|-------|-------|
| Host | `aws-1-sa-east-1.pooler.supabase.com` ← **aws-1**, não aws-0 |
| IP fixado no compose | `18.228.163.245` |
| Port | `5432` (Session Mode) |
| Database | `postgres` |
| User | `postgres.vvtihydurfxaqdtliuot` ← sufixo obrigatório |
| SSL | `require` — obrigatório |

**Armadilhas documentadas:**
- `aws-0` dá erro "Tenant or user not found" — usar `aws-1` obrigatoriamente
- Porta `6543` = Transaction Mode (sem prepared statements) — usar `5432`
- Se o IP do pooler mudar na AWS, o `extra_hosts` fica stale e a conexão quebra silenciosamente

---

## Secrets — Estado Atual

**Localização:** `services/validator/.env` (referenciado via `env_file:` no compose)

**Variáveis críticas:**
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY` ← nunca em código, nunca commitado

**Verificação antes de qualquer commit:**
```bash
git status services/validator/.env
git diff --staged -- "*.env"
```

**Roadmap:** evolução para secrets manager dedicado está na Fase 5 — não antecipar.

---

## Padrão para Novos Serviços

Novos conectores (ex: `meta_connector`, `google_connector`) seguem uma de duas opções:

### Opção A — Container dedicado (serviço com processo contínuo ou porta própria)

```yaml
meta_connector:
  build: ../services/meta_connector
  container_name: dante_meta_connector
  env_file:
    - ../services/meta_connector/.env
  expose:
    - "8001"
  networks:
    - dante_internal
  restart: unless-stopped
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8001/health"]
    interval: 30s
    timeout: 5s
    retries: 3
    start_period: 15s
```

### Opção B — Volume montado no N8N (script acionado sob demanda via Execute Command)

```yaml
# Adicionar ao serviço n8n existente:
volumes:
  - n8n_data:/home/node/.n8n
  - ../n8n/flows:/home/node/flows:ro
  - ../services/meta_connector:/home/node/connectors/meta:ro  # ← novo
```

**Regra de decisão:** se precisa de porta própria, daemon ou ciclo de vida independente → Opção A. Se é script acionado pelo N8N → Opção B. Dúvida? Sempre começa com B.

---

## Healthchecks — Referência

| Container | Teste | Intervalo |
|-----------|-------|-----------|
| `dante_n8n` | `wget -q -O- http://localhost:5678/healthz` | 30s |
| `dante_validator` | `curl -f http://localhost:8000/health` | 30s |
| `dante_nginx` | sem healthcheck configurado (verificar via `http://nginx/nginx-health` de outro container) | — |

---

## Comandos de Referência Rápida

```bash
# Subir um serviço específico (sem derrubar os outros)
cd /root/dante/infrastructure && docker compose up -d nome_do_servico

# Ver logs em tempo real
docker logs -f dante_n8n
docker logs -f dante_validator

# Inspecionar health de todos os containers Dante
docker inspect --format='{{.Name}}: {{.State.Health.Status}}' \
  dante_n8n dante_validator dante_nginx

# Reiniciar um container sem derrubar os outros
docker compose restart nome_do_servico

# Ver configuração efetiva do compose (com variáveis expandidas)
cd /root/dante/infrastructure && docker compose config

# Entrar no container para debug
docker exec -it dante_n8n sh
docker exec -it dante_validator bash
```
