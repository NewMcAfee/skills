# Templates de Documento — Tabulário

Referência para os tipos de documento que o Tabulário pode criar.
Use como ponto de partida — adapte ao conteúdo real, não force o template.

---

## framework

```markdown
# [Nome do Framework]

**Tema**: frameworks
**Data**: [YYYY-MM-DD]
**Tipo**: framework

## Problema que resolve
[Qual situação recorrente este framework ajuda a navegar]

## Estrutura
[Os componentes ou dimensões do framework — lista, tabela ou diagrama textual]

## Como usar
[Passo a passo ou condições de aplicação]

## Exemplos
[1-2 exemplos concretos do framework em ação]

## Limitações
[Onde este framework não se aplica ou falha]

## Relacionado
```

---

## modelo-operacional

```markdown
# [Nome do Modelo]

**Tema**: operacao
**Data**: [YYYY-MM-DD]
**Tipo**: modelo-operacional

## Contexto
[Qual processo ou operação este modelo governa]

## Fluxo
[Etapas em ordem — numeradas e imperativas]

## Inputs necessários
[O que precisa estar disponível para executar]

## Outputs esperados
[O que este modelo produz]

## Decisões críticas
[Pontos onde há bifurcação — critérios para cada caminho]

## Relacionado
```

---

## tese

```markdown
# [Afirmação da Tese — Frase Completa]

**Tema**: teses
**Data**: [YYYY-MM-DD]
**Tipo**: tese

## Contexto
[O que levou a formular esta tese]

## Argumento central
[O raciocínio que sustenta a tese — passo a passo]

## Evidências
[O que confirma ou sustenta — exemplos, padrões observados, analogias]

## Contrargumentos
[O que poderia refutar — e por que ainda sustento a tese]

## Implicações
[Se a tese for verdadeira, o que isso muda]

## Status
[ ] Em formação  [ ] Consolidada  [ ] Revisada em [data]

## Relacionado
```

---

## playbook

```markdown
# Playbook: [Situação ou Objetivo]

**Tema**: operacao
**Data**: [YYYY-MM-DD]
**Tipo**: playbook

## Quando usar
[Condições que ativam este playbook]

## Pré-condições
[O que precisa estar verdadeiro antes de começar]

## Sequência de ações
1. [Ação imperativa com critério de conclusão]
2. ...

## Decisões frequentes
| Situação | Ação recomendada |
|----------|-----------------|
| ... | ... |

## Erros comuns
[O que costuma dar errado e como evitar]

## Relacionado
```

---

## nota-de-arquitetura

```markdown
# [Decisão ou Componente de Arquitetura]

**Tema**: arquitetura
**Data**: [YYYY-MM-DD]
**Tipo**: nota-de-arquitetura

## Contexto
[Qual problema ou necessidade levou a esta decisão]

## Decisão
[O que foi decidido — específico e direto]

## Alternativas consideradas
[Outras opções avaliadas e por que foram descartadas]

## Consequências
[O que esta decisão implica — técnico e operacional]

## Status
[ ] Proposta  [x] Adotada  [ ] Revisada  [ ] Obsoleta

## Relacionado
```

---

## síntese

```markdown
# Síntese: [Tema ou Título da Conversa]

**Tema**: sinteses
**Data**: [YYYY-MM-DD]
**Tipo**: síntese
**Origem**: [Conversa sobre X em Claude, data]

## O que foi discutido
[1 parágrafo — tema central e escopo da conversa]

## Decisões tomadas
[Lista de decisões com brevidade]

## Insights relevantes
[O que emergiu de valor reutilizável]

## Frameworks ou modelos identificados
[Se houver — com link para doc específico se merecer]

## Próximos passos registrados
[Se houver comprometimentos ou desdobramentos]

## Relacionado
```

---

## prompt-estruturado

```markdown
# Prompt: [Nome Descritivo]

**Tema**: prompts
**Data**: [YYYY-MM-DD]
**Tipo**: prompt-estruturado

## Contexto de uso
[Quando usar este prompt — situação específica]

## Prompt

```
[Prompt completo aqui]
```

## Variáveis
| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| {X} | ... | ... |

## Resultado esperado
[O que este prompt deve produzir]

## Observações
[Ajustes úteis, limitações, versões testadas]

## Relacionado
```

---

## registro-de-skill

```markdown
# Skill: [Nome da Skill]

**Tema**: skills
**Data**: [YYYY-MM-DD]
**Tipo**: registro-de-skill

## Problema que resolve
[Por que esta skill foi criada]

## Função
[O que a skill faz — 2-3 linhas]

## Modos de uso
[Como invocar e com quais inputs]

## Decisões de design
[Escolhas não-óbvias feitas na construção e por quê]

## Limitações conhecidas
[O que a skill não faz bem ou ainda não suporta]

## Localização
[Caminho no repositório de skills]

## Relacionado
```

---

## aprendizado

```markdown
# Aprendizado: [Título Que Resume a Lição]

**Tema**: aprendizados
**Data**: [YYYY-MM-DD]
**Tipo**: aprendizado

## Contexto
[O que gerou este aprendizado — situação ou experimento]

## Lição
[O que aprendi — direto, sem floreio]

## Por que importa
[Impacto prático — onde isso muda como opero]

## Como aplicar
[Situações onde esta lição se aplica]

## O que mudei
[Se gerou mudança concreta — o que foi alterado]

## Relacionado
```
