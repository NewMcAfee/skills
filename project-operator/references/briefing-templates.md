# Templates de Briefing — Project Operator

## Princípio de briefing bem construído

Um briefing é bom quando quem recebe — pessoa ou agente — consegue executar sem fazer perguntas adicionais. Se o destinatário precisar voltar para esclarecer algo, o briefing falhou.

**Três perguntas de validação antes de emitir qualquer briefing:**
1. O output esperado está claro e verificável?
2. Quem recebe tem tudo que precisa para começar?
3. As restrições estão explícitas?

---

## Template 1 — Briefing para Pessoa

```
BRIEFING — [nome curto da tarefa/entregável]
ID do backlog: [B001]
Data: [YYYY-MM-DD]

OBJETIVO
[O que precisa existir como resultado — 1 frase]

ESCOPO
Incluído: [o que está dentro]
Excluído: [o que não é responsabilidade desta tarefa]

ENTREGA ESPERADA
Formato: [documento .md / planilha / arquivo no Figma / configuração no GTM / etc.]
Onde entregar: [link, pasta, slack, sistema]
Critério de done: [como saber que está pronto — verificável]

CONTEXTO
[O que a pessoa precisa saber para executar sem perguntar. Máx. 5 linhas. Não repita o que está no objetivo.]

DEPENDÊNCIAS
Precisa de: [o que deve estar pronto antes desta tarefa]
Bloqueia: [o que só começa após esta tarefa — se aplicável]

DONO
[Nome da pessoa]

PRAZO
[Data — YYYY-MM-DD]

VALIDAÇÃO
Passa pelo Quality Controller antes de entregar? [sim | não]
Se sim: qual tipo de revisão? [estratégica | criativa | operacional]
```

---

## Template 2 — Briefing para Agente de IA (Skill Dante)

```
BRIEFING — [nome da skill]
ID do backlog: [B001]
test_id: [uuid se relacionado a teste] | não aplicável
Data: [YYYY-MM-DD]

INPUT NECESSÁRIO
[O que o agente precisa receber para começar. Seja específico: formato, campos, referências.]

OUTPUT ESPERADO
[O que o agente deve entregar. Formato exato, estrutura, extensão.]

RESTRIÇÕES
- [O que não pode fazer ou incluir]
- [Sensibilidades da conta relevantes]
- [O que já foi tentado e não funcionou]

CONTEXTO MÍNIMO
[Apenas o que é necessário para o agente executar com inteligência. Não dump de dados — síntese.]
Fase do projeto: [GTM / Loop]
Maturidade: [imaturo / maduro]
Momento da conta: [breve leitura — 1 frase]

PRAZO
[Data — YYYY-MM-DD]

VALIDAÇÃO
Output vai para o Quality Controller? [sim | não]
Se sim: qual checklist aplicar? [estratégico | criativo | operacional]
```

---

## Briefings Específicos por Destinatário

### Briefing → Escriba (copy e textos)

```
BRIEFING ESCRIBA — [nome do material]
ID: [B001] | test_id: [se houver]

OBJETIVO DO MATERIAL
[O que o texto deve fazer — não o que é, mas o que realiza]

CANAL E FORMATO
Canal: [LP / anúncio / e-mail / post / etc.]
Formato: [headline + body / estrutura completa / variações]
Extensão: [número de palavras ou estrutura esperada]

PÚBLICO
ICP: [nome do perfil]
Nível de consciência: [UNC / PRB / SOL / PRO / MIX]
Dor ou desejo a acionar: [o que deve ressoar]

CONTEXTO ESTRATÉGICO
PUV: [proposta única de valor relevante]
Gancho: [ângulo de comunicação]
Tom: [formal / descontraído / urgente / aspiracional]

RESTRIÇÕES
- [o que não pode aparecer]
- [sensibilidades do cliente]
- [o que já foi testado e rejeitado]

PRAZO: [YYYY-MM-DD]
QC antes de subir: [sim | não]
```

### Briefing → Tracking Engineer / Instrumentation Engineer

```
BRIEFING TRACKING — [escopo da configuração]
ID: [B001]

O QUE PRECISA SER CONFIGURADO
[Descrição objetiva: quais eventos, quais propriedades, qual plataforma]

CONTEXTO
Measurement Plan de referência: [link ou documento]
Eventos prioritários: [lista com nome semântico + trigger]
Plataformas de destino: [GA4 / Meta Pixel / Google Ads / etc.]

CRITÉRIO DE DONE
[Como validar que está funcionando: Preview GTM sem erro / evento chegando no Supabase / etc.]

RESTRIÇÕES
- [o que não deve ser configurado ainda]
- [estrutura de nomenclatura a seguir: ver taxonomia]

PRAZO: [YYYY-MM-DD]
QC antes de apresentar ao cliente: [sim | não]
```

### Briefing → Quality Controller

```
BRIEFING QC — [nome do output para revisão]
ID: [B001]

OUTPUT A REVISAR
Tipo: [LP / criativo / copy / dashboard / documento estratégico / campanha]
Link ou localização: [onde encontrar]

CONTEXTO DA REVISÃO
Por que precisa de QC agora: [antes de publicar / antes de apresentar ao cliente / antes de subir]
Sensibilidades relevantes: [o que o cliente é crítico / o que já foi rejeitado antes]
Momento da conta: [leitura atual — 1 frase]

CHECKLIST PRIORITÁRIO
Verificar especialmente: [os pontos mais críticos para este output específico]

PRAZO PARA DEVOLUÇÃO: [YYYY-MM-DD]
```

### Sinal de Governança → Decision Recorder

```
SINAL DE GOVERNANÇA — Decision Recorder
Origem: Project Operator
Tipo: decisão de execução | mudança de rota | regra operacional | padrão recorrente | bloqueio recorrente

CONTEÚDO
[O que aconteceu ou foi decidido — fato, não interpretação]

IMPACTO
[O que muda na execução a partir deste registro]

RELEVÂNCIA FUTURA
[Por que isso deve ser lembrado — o que previne ou melhora]
```

### Sinal de Governança → Enablement Builder

```
SINAL DE GOVERNANÇA — Enablement Builder
Origem: Project Operator
Tipo de padrão: erro recorrente | processo repetido | checklist implícito | fluxo padronizável

PADRÃO IDENTIFICADO
[Descrição do que se repete — objetivo, sem julgamento]

FREQUÊNCIA
[Quantas vezes apareceu / em quantos projetos]

SUGESTÃO DE FORMATO
[SOP / checklist / template / playbook / nova skill]
Justificativa: [por que este formato é o adequado para este padrão]
```
