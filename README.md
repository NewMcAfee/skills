# Skills

Coleção de skills para o **Claude Code** — cada pasta é uma skill com seu `SKILL.md` e ativação por contexto.

## Instalação

```bash
# clone dentro de ~/.claude/skills/ (ou do diretório de skills do seu projeto)
cd ~/.claude/skills
git clone https://github.com/NewMcAfee/skills.git .
```

Cada subpasta contém um `SKILL.md` com frontmatter (`name`, `description`). O Claude Code carrega e seleciona a skill automaticamente com base na descrição — mas este índice ajuda a saber **quando chamar cada uma** sem precisar abrir os arquivos.

---

## Como ler este índice

| Coluna | O que é |
|---|---|
| **Skill** | Pasta no repositório (link direto) |
| **Quando usar** | Gatilho prático — se o que você vai fazer cai aqui, é essa |
| **Quando NÃO usar** | Delimita o escopo contra skills vizinhas |

---

## 🧭 Estratégia & Discovery

Investigação inicial, mercado, ICP, posicionamento de produto, diagnóstico e plano executivo.

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`socrates`](./socrates) | Kickoff com cliente novo: preparar perguntas e consolidar briefing/debriefing em Markdown | Projetos já em execução com briefing pronto |
| [`oraculo`](./oraculo) | Pesquisa de mercado, análise competitiva, tendências setoriais (sempre com web search) | ICP (→ `michelangelo`) ou posicionamento (→ `davinci`) |
| [`michelangelo`](./michelangelo) | Construir/refinar ICP, mapear comitê de decisão, dores e desejos do cliente-alvo | Pesquisa de mercado ampla (→ `oraculo`) |
| [`davinci`](./davinci) | Proposta única de valor + oferta de ativação a partir do ICP (processo de 8 passos) | Antes de ter ICP pronto |
| [`arquimedes`](./arquimedes) | Diagnóstico estratégico: restrição principal, North Star, unit economics, breakeven. Também sparring crítico de outros outputs | Execução em plataforma, criação de copy ou análise de dados brutos |
| [`edison`](./edison) | Criar produto digital do zero: visão → briefing completo → MVP pronto pra construção | Refinamento de produto já em produção |
| [`cesar`](./cesar) | Sintetizar os 6 outputs anteriores em **Plano de GTM** + apresentação executiva (até 1h30) | Antes de ter o Pacote 1 completo |

## 🎨 Marca & Identidade Visual

Da estratégia de marca (quem somos) até logo, tipografia e design system.

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`Campbell`](./Campbell) | Construir ou auditar **Brand Core**: propósito, arquétipo, tom de voz, posicionamento | Pesquisa (→ `oraculo`), ICP (→ `michelangelo`), visual (→ `Rand`) |
| [`Rand`](./Rand) | Traduzir Brand Core em direção de arte: brandscape, caminhos visuais, logo + tipografia | Estratégia (→ `Campbell`), paleta com escalas (→ `Design System Engineer`) |
| [`Design System Engineer`](./Design%20System%20Engineer) | Design system parametrizado: tokens OKLCH, 3 camadas (primitivos → semânticos → componentes), preview HTML + script Figma | Estratégia (→ `Campbell`), logo/tipografia (→ `Rand`) |
| [`brand`](./brand) | Fundamentos gerais de brand voice, messaging e consistência de marca | Sistemas tokenizados (→ `Design System Engineer`) |
| [`design`](./design) | Skill guarda-chuva: logos (55 estilos), CIP (50 entregáveis), slides, banners, ícones, social photos | Tarefas já cobertas por skills específicas (prefira elas) |
| [`design-system`](./design-system) | Tokens 3-camadas + apresentações brand-compliant | Design system completo com preview e Figma (→ `Design System Engineer`) |

## ✍️ Copy & Conteúdo

Fundação de voz, copy direta, roteiros de vídeo e distribuição de conteúdo.

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`escriba`](./escriba) | **Copy System** (Modo 0) ou hard copy: anúncios, LPs, e-mails, headlines (Modos 1–5) | Estratégia de produto (→ `davinci`) ou design visual (→ `takehiko-inoue`) |
| [`kubrick`](./kubrick) | Roteiros de vídeo: VSL, Reels/Shorts, YouTube, testimoniais, case studies | Copy estática (→ `escriba`) |
| [`bernbach`](./bernbach) | Transformar report de performance pronto em e-mail, PPT, LinkedIn, case study | Gerar o report em si (→ `newton`) |
| [`sensei`](./sensei) | Qualquer conteúdo na voz do **TheGrowthSensei** (posts, newsletters, roteiros) | Conteúdo fora dessa marca |
| [`the-editor`](./the-editor) | Newsletter **The Build**: episódios, pautas, Notes nos 6 pilares | Outras marcas / formatos não-editoriais |
| [`the-amplifier`](./the-amplifier) | Desdobrar episódio do The Build em Notes/LinkedIn/X/IG/Shorts | Produzir o episódio (→ `the-editor`) ou assets visuais (→ `takehiko-inoue`) |
| [`newton`](./newton) | Análise **ad-hoc** de CSV/planilha de campanha → relatório executivo | Análise estruturada por ICP×Produto (→ `growth-lead`) |

## 🖼️ Design & UI (execução visual e de interface)

Construção de interfaces, landing pages, banners, slides e assets visuais.

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`frontend-design`](./frontend-design) | Construir UIs distintas e polidas (componentes, páginas, dashboards) evitando cara genérica de IA | Code puro de lógica (sem design) |
| [`landing-page-designer`](./landing-page-designer) | Landing page single-file (launch, Product Hunt, GitHub Pages) sem dependências | App completo com rotas e estado (→ `frontend-design` / `tesla`) |
| [`ui-styling`](./ui-styling) | UI acessível com shadcn/ui + Tailwind, dark mode, tokens de tema | Decisões de estilo/paleta do zero (→ `ui-ux-pro-max`) |
| [`ui-ux-pro-max`](./ui-ux-pro-max) | Inteligência de UI/UX: 50+ estilos, 161 paletas, 57 pares tipográficos, 99 guidelines | Review contra guidelines (→ `web-design-guidelines`) |
| [`banner-design`](./banner-design) | Banners para social, ads, website hero, print (vários estilos e plataformas) | Vídeo animado (→ `kubrick` / `takehiko-inoue`) |
| [`slides`](./slides) | Apresentações HTML estratégicas com Chart.js e tokens de design | Slides brand puros sem narrativa (→ `design-system`) |
| [`web-design-guidelines`](./web-design-guidelines) | Review de UI pronta: acessibilidade, UX, boas práticas | Construir UI do zero (→ `frontend-design`) |
| [`ux-architect`](./ux-architect) | Admin panels B2B: inventário de telas, IA, flows, wireframe spec | Geração de código ou review visual |
| [`takehiko-inoue`](./takehiko-inoue) | Produzir assets reais via **Figma MCP**: brandbook, anúncios, posts, carrosséis, vídeo | Estratégia visual (→ `Rand`) ou copy (→ `escriba`) |

## 📣 Mídia Paga & Performance

Estratégia, estruturação e análise de campanhas Google/Meta no ecossistema Dante.

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`sobral`](./sobral) | Estratégia sênior de mídia paga (Google + Meta): estrutura, públicos, criativos, orçamento | Execução em plataforma ou criação de copy |
| [`media-buyer-goog`](./media-buyer-goog) | Estrutura de conta Google Ads (Search/Display/PMax) → CSV pro Google Ads Editor | Antes da estratégia do `sobral` ou sem `icp_product_map` |
| [`media-buyer-meta`](./media-buyer-meta) | Estrutura conta Meta (campanha→conjunto→anúncio) com exclusões waterfall → CSV bulk | Antes da estratégia do `sobral` |
| [`guardiao-do-log`](./guardiao-do-log) | Registrar campanhas/conjuntos/anúncios no Dante via Validator | Análise ou leitura de dados |
| [`performance-ingestor`](./performance-ingestor) | Ingerir export da plataforma para `performance_metrics` via `data_miner` | Análise dos dados ingeridos (→ `growth-lead` / `newton`) |
| [`growth-lead`](./growth-lead) | Diagnóstico semanal com JSON estruturado do Data Miner por `icp_product_map` | Dados brutos/ad-hoc (→ `newton`) |
| [`arquiteto-de-testes`](./arquiteto-de-testes) | Transformar diagnóstico do Growth Lead em hipóteses falsificáveis + briefing criativo | Análise de performance ou produção criativa |

## 📊 Tracking & Instrumentação

Do desenho do evento ao GTM configurado e validado.

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`instrumentation-engineer`](./instrumentation-engineer) | Measurement plan, `dataLayer.push()`, payloads N8N — **antes** do GTM | Configurar GTM/tags (→ `tracking-engineer`) |
| [`tracking-engineer`](./tracking-engineer) | GTM, GA4, Meta CAPI, Google Ads, Clarity + QA técnico — a partir do handoff da IE | Preparar a origem do evento (→ `instrumentation-engineer`) |

## 🛠️ Engenharia (ecossistema Dante)

Agentes **adversariais** — questionam decisões técnicas antes de implementar.

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`backend-engineer`](./backend-engineer) | FastAPI/Pydantic/Supabase: endpoints, padrão de erro, auth, performance de API | UI, copy ou estratégia de negócio |
| [`frontend-engineer`](./frontend-engineer) | React 18+, dashboards BI, escolha de biblioteca, estados de loading/erro/vazio | Backend, banco ou estratégia |
| [`data-engineer`](./data-engineer) | Schema PostgreSQL/Supabase, índices, migrations, modelagem para BI | Lógica de aplicação ou UI |
| [`devops-engineer`](./devops-engineer) | VPS Ubuntu, Docker Compose, Nginx Proxy Manager, secrets — com rollback-first | Lógica de aplicação ou schema |
| [`security-engineer`](./security-engineer) | Auth, tokens/secrets, Supabase RLS, PII, endpoints sensíveis | UI, copy ou estratégia |
| [`n8n-pipeline-engineer`](./n8n-pipeline-engineer) | Flows N8N: design, webhooks, scheduling, error handling, idempotência | UI, copy, backend Python ou estratégia |

## 🎛️ Operação & Governança Dante

Governança de backlog, qualidade, memória e saúde de contas.

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`project-operator`](./project-operator) | Backlog 4-níveis, priorização RICE/MoSCoW, briefings executáveis, orquestração de ciclos | Análise de performance, decisões estratégicas ou qualidade |
| [`client-success-strategist`](./client-success-strategist) | Pós check-in/reunião: health score, leitura de clima, riscos, oportunidades | Decisões estratégicas ou qualidade de outputs |
| [`quality-controller`](./quality-controller) | Revisão em 5 camadas de outputs antes de virar entrega | Produzir copy/assets ou decidir estratégia |
| [`decision-recorder`](./decision-recorder) | Registrar/consultar memória operacional (Truths & Traps) | Análise de performance ou qualidade |
| [`enablement-builder`](./enablement-builder) | Transformar padrões recorrentes em checklists/SOPs/templates/playbooks | Executar tarefas ou criar skills (→ `god`) |
| [`skill-ingestao`](./skill-ingestao) | Onboarding: persistir planejamento estratégico nas 7 tabelas fundacionais do Dante | Registrar campanhas/métricas (→ `guardiao-do-log` / `performance-ingestor`) |

## 🧰 Produto digital & Prototipagem

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`tesla`](./tesla) | Criar prompts de alta performance para o **Lovable** (lovable.dev): MVP, LP, dashboard, SaaS | Construir direto em código (→ `frontend-design` / `landing-page-designer`) |

## 📚 Conhecimento & Memória

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`obsidian`](./obsidian) | Comprimir contexto intra-sessão e persistir memória inter-sessão em `.claude/context.md` | Projetos de sessão única |
| [`tabulario`](./tabulario) | Curadoria soberana de docs `.md` no repositório Alexandria (reuso, consulta, aprendizado) | Memória operacional de projeto (→ `decision-recorder`) |

## 🧬 Meta-skills (construir novas skills)

| Skill | Quando usar | Quando NÃO usar |
|---|---|---|
| [`god`](./god) | Forjar nova skill para problema **recorrente** que merece ferramenta dedicada | Tarefa única e descartável |
| [`god-bootstrap`](./god-bootstrap) | Setup inicial do Claude Code como fábrica de skills (instala `god`, prepara CLAUDE.md) | Ambiente já configurado |

---

## Regras de uso rápidas

1. **Descrição é o gatilho.** O Claude Code seleciona skill pelo campo `description:` no frontmatter. Se quiser forçar, cite o nome (ex.: "usa o `escriba`").
2. **Skills adversariais** (engenharia Dante) foram desenhadas pra **discordar** — se quiser validação sem atrito, não chame o `backend-engineer` / `data-engineer` / etc.
3. **Pipeline Dante** tem ordem: `socrates` → `oraculo` → `michelangelo` → `davinci` → `arquimedes` → `sobral` → `cesar`. Pular etapa costuma gerar output raso.
4. Skills de marca Dante seguem: `Campbell` (estratégia) → `Rand` (direção de arte) → `Design System Engineer` (tokens) → `takehiko-inoue` (produção).

---

_55 skills · atualizado automaticamente a partir do frontmatter de cada `SKILL.md`._
