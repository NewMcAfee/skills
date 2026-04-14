---
name: obsidian
description: Gerencia contexto de forma inteligente em duas dimensões: comprime o contexto intra-sessão preservando apenas o irreversível, e persiste memória inter-sessão em .claude/context.md no projeto. Ative com /obsidian quando o contexto estiver se aproximando do limite, ao final de uma sessão com trabalho significativo, ou ao iniciar uma sessão em projeto com histórico. NÃO use para projetos de sessão única ou tarefas descartáveis.
allowed-tools: Read,Write,Bash
---

# Obsidian — Gestão de Contexto para Claude Code

O contexto é o recurso mais escasso de uma sessão. O critério central desta skill é **irreversibilidade**: informação irreversível (existe só na conversa, não pode ser re-derivada lendo o código) deve ser preservada; informação recuperável (pode ser re-obtida com uma ferramenta) pode ser descartada.

Claude Code faz compaction automático em ~83% de capacidade. Este critério garante que o que sobrevive à compaction seja o que realmente importa — e que a memória entre sessões capture o mesmo conjunto.

---

## Modo 1 — Compressão Intra-Sessão

**Quando usar:** contexto acima de 60% de capacidade, ou antes de iniciar subtarefa longa.

### O que é irreversível (SEMPRE preservar)
- Decisões arquiteturais e o porquê — ex: "escolhemos X em vez de Y porque Z"
- Bugs diagnosticados: mensagem de erro exata + componente afetado + causa-raiz
- Preferências e constraints declarados pelo usuário nesta sessão
- O que foi tentado e falhou (com a razão do fracasso)
- Lista de arquivos modificados (nomes, não conteúdo)
- Próximos passos pendentes

### O que é recuperável (pode descartar)
- Outputs de Read, Bash, Grep já processados e sem referência posterior
- Histórico de navegação (ls, find, exploração sem achados relevantes)
- Raciocínio intermediário de abordagens descartadas
- Boilerplate de respostas (confirmações, "entendido", transições)
- Conteúdo de arquivos que não foram modificados e podem ser re-lidos

### Workflow de compressão

1. **Leia** `.claude/context.md` se existir — essa é a base, não recomece do zero
2. **Construa** o sumário estruturado com estas seções exatas:

```markdown
## Estado em [data/hora]

### Objetivo da sessão
[1-2 frases: o que estamos tentando fazer]

### Decisões tomadas
- [decisão]: [rationale em 1 frase]

### Arquivos modificados
- `path/to/file` — [o que mudou]

### Erros e diagnósticos
- [erro exato ou descrição]: causa → [causa-raiz]; status → [resolvido|pendente]

### Tentativas fracassadas
- [abordagem]: falhou porque [razão] — não tentar novamente

### Próximos passos
- [ ] [ação concreta]
```

3. **Não resuma tudo** — processe apenas o span de contexto que será descartado, merge com sumário existente
4. **Ao usar `/compact`**, injete o sumário como primeiro bloco da instrução de compactação

---

## Modo 2 — Memória Inter-Sessão

**Quando usar:** ao encerrar uma sessão com trabalho significativo, ou ao iniciar sessão em projeto com histórico.

### Arquivo de memória: `.claude/context.md`

Crie ou atualize `.claude/context.md` na raiz do projeto com este schema:

```markdown
# Contexto do Projeto — [Nome do Projeto]
_Atualizado: [data]_

## Arquitetura e Decisões Permanentes
<!-- Decisões que não mudam: stack, padrões, constraints não-negociáveis -->
- [decisão permanente e rationale]

## Estado Atual do Trabalho
<!-- O que está em andamento agora -->
- **Branch/área**: [nome]
- **Objetivo**: [o que está sendo construído/corrigido]
- **Progresso**: [onde parou]

## Padrões Estabelecidos Neste Projeto
<!-- Convenções específicas descobertas na prática, não óbvias pelo código -->
- [padrão]: [como aplicar]

## Erros e Armadilhas Conhecidas
<!-- O que já quebrou e por quê — para não repetir -->
- [problema]: [causa] → [solução ou workaround]

## Tentativas Fracassadas
<!-- Abordagens que não funcionam neste projeto -->
- [abordagem]: falhou porque [razão]

## Dependências e Contexto Externo
<!-- Serviços, APIs, variáveis de ambiente necessárias -->
- [dependência]: [detalhes relevantes]
```

### Workflow de persistência

**Ao encerrar sessão:**
1. Se `.claude/context.md` não existe, crie-o
2. Atualize apenas as seções afetadas pela sessão — não reescreva o que não mudou
3. Na seção "Estado Atual do Trabalho", registre exatamente onde parou
4. Adicione à seção "Erros e Armadilhas" qualquer falha nova com causa-raiz

**Ao iniciar sessão:**
1. Leia `.claude/context.md`
2. Informe o usuário: "Contexto carregado: [objetivo atual], [N decisões registradas], [próximos passos]"
3. Não re-explore o codebase para informações já registradas — confie no contexto

### Bloco no CLAUDE.md do projeto

Se o projeto tem CLAUDE.md, adicione este bloco (se ainda não existir):

```markdown
## Contexto Persistente

Sempre leia `.claude/context.md` ao iniciar uma sessão neste projeto.
Sempre atualize `.claude/context.md` ao encerrar sessão com trabalho significativo.
```

---

## Modo 3 — Inicialização de Projeto

Para projetos sem `.claude/context.md` ainda:

1. Pergunte ao usuário: "Qual é o objetivo principal deste projeto?"
2. Explore minimamente: `ls`, leia README se existir, leia CLAUDE.md se existir
3. Crie `.claude/context.md` com o schema acima preenchido com o que já é conhecido
4. Adicione bloco no CLAUDE.md do projeto (crie CLAUDE.md se não existir)

---

## Anti-patterns

- **Não resuma tudo ao comprimir** — processar o span inteiro gasta mais tokens do que economiza; processe apenas o que vai ser descartado
- **Não guarde outputs de ferramentas** — Read/Bash/Grep podem ser re-executados; guardar seus outputs é o principal desperdício de contexto
- **Não use `.claude/context.md` como log de sessão** — é estado atual, não histórico; informação supersedida deve ser substituída, não acumulada
- **Não confunda recuperável com irrelevante** — um arquivo não modificado é recuperável (re-leia quando precisar); uma decisão de design não está em arquivo nenhum
- **Não reescreva o CLAUDE.md do projeto inteiro** — apenas adicione o bloco de contexto persistente; não altere o que o usuário já escreveu

---

## Avaliação

### Cenário 1: Compressão após debugging longo
**Input**: sessão com 80+ tool calls de debugging, contexto em ~65%
**Comportamento esperado**:
- [ ] Identifica e preserva: mensagem de erro exata, causa-raiz encontrada, arquivo corrigido
- [ ] Descarta: outputs de Read e Bash anteriores ao diagnóstico
- [ ] Produz sumário com seções preenchidas, não genéricas
- [ ] Não perde o próximo passo pendente

### Cenário 2: Retomada de sessão
**Input**: nova sessão, projeto com `.claude/context.md` existente
**Comportamento esperado**:
- [ ] Lê `.claude/context.md` antes de qualquer exploração
- [ ] Informa resumo do estado ao usuário
- [ ] Não re-explora arquivos já listados em "Arquivos modificados"
- [ ] Retoma do "Próximos passos" sem perguntar "por onde começamos?"

### Cenário 3: Registro de decisão arquitetural
**Input**: usuário decide usar abordagem X em vez de Y durante a sessão
**Comportamento esperado**:
- [ ] Registra em "Decisões tomadas" com rationale
- [ ] Se Y foi tentado e falhou, registra em "Tentativas fracassadas"
- [ ] Atualiza `.claude/context.md` ao encerrar
- [ ] Decisão persiste e aparece em sessão futura
