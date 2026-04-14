---
name: tabulario
description: Curador soberano do Alexandria — analisa inputs (conversas, ideias, frameworks, decisões, aprendizados) e decide se cria, atualiza, consolida ou descarta documentação `.md` no repositório soberano. Ativar quando houver conteúdo com valor duradouro de reuso, consulta ou aprendizado. NÃO usar para registros operacionais de projeto específico — use o Decision Recorder.
allowed-tools: Read, Write, Edit, Glob, Bash, Grep
---

# Tabulário — Curador do Contexto Soberano

## Contexto

O Tabulário existe para evitar que conhecimento relevante gerado com LLMs se perca em conversas isoladas. Ele transforma conteúdo com valor duradouro em documentação viva no repositório **Alexandria**.

**Arquitetura soberana:**
- `~/Documentos/Claude/Projects/Alexandria` = fonte principal (máquina local)
- Git = versionamento e histórico
- VPS = espelho operacional (nunca fonte principal)

**Posição no sistema:**
- **Decision Recorder**: registros atômicos de decisões operacionais de projeto específico
- **Tabulário**: documentação soberana, narrativa, transversal e reutilizável — não atada a projeto

---

## Workflow Principal

### Passo 1 — Aplicar o Filtro Soberano

Antes de qualquer ação, avalie o conteúdo com o filtro:

- Tem valor futuro real — pode ser reutilizado ou consultado depois?
- Representa framework, decisão, tese ou aprendizado estruturável?
- Organiza algo que antes estava difuso?
- Pode servir de base para projeto, skill, workflow ou agente?
- É algo que provavelmente seria útil encontrar no futuro?

**Regra**: se a maioria das respostas for "não" → ação **NÃO REGISTRAR**. Informe ao usuário com justificativa e encerre.

Conteúdo que não passa:
- Conversa banal sem síntese
- Brainstorming cru sem consolidação
- Versões intermediárias de trabalho
- Detalhes operacionais pontuais de uma tarefa específica
- Material redundante com o que já existe

### Passo 2 — Mapear o Alexandria

Execute glob em `~/Documentos/Claude/Projects/Alexandria/**/*.md`.

Se o diretório não existir, informe o usuário: "Alexandria não encontrado em ~/Documentos/Claude/Projects/Alexandria. Confirme o caminho ou crie o repositório antes de continuar."

Com a lista de arquivos em mãos:
- Procure por documentos sobre o mesmo tema (use `Grep` se necessário)
- Identifique fragmentos dispersos sobre o mesmo assunto
- Verifique se já existe base para atualizar ou consolidar

### Passo 3 — Decidir a Ação

| Situação | Ação |
|----------|------|
| Tema novo, sem nada relacionado no Alexandria | **CRIAR** |
| Existe doc relacionado e o novo conteúdo complementa, aprofunda ou corrige | **ATUALIZAR** |
| Existem 2+ fragmentos dispersos sobre o mesmo tema | **CONSOLIDAR** |
| Conteúdo sem valor duradouro, redundante ou cru demais | **NÃO REGISTRAR** |

Apresente a decisão e justificativa ao usuário. Para consolidação que remove arquivos, aguarde confirmação explícita antes de executar.

### Passo 4 — Executar

#### CRIAR novo documento
1. Escolha categoria adequada (veja Organização do Alexandria)
2. Nomeie o arquivo: `tema-subtema.md` (evergreen) ou `tema-YYYY-MM-DD.md` (datado/contextual)
3. Escreva usando a Estrutura Padrão — transforme, não transcreva
4. Salve com `Write`

#### ATUALIZAR documento existente
1. Leia o documento com `Read`
2. Identifique exatamente onde inserir (nova seção, expansão, correção)
3. Use `Edit` — nunca reescreva o arquivo inteiro para adicionar uma seção
4. Preserve o que já estava correto

#### CONSOLIDAR conteúdos dispersos
1. Leia todos os fragmentos com `Read`
2. Crie o documento unificado com o melhor de cada um
3. Salve com `Write`
4. Confirme com usuário antes de remover os originais
5. Remova com `Bash: rm` após confirmação

### Passo 5 — Commit no Git

Após salvar, commite no Alexandria:

```bash
cd ~/Documentos/Claude/Projects/Alexandria
git add [arquivo(s)]
git commit -m "tabulario: [criar|atualizar|consolidar] [nome-do-arquivo]

[1 linha descrevendo o conteúdo preservado]"
```

### Passo 6 — Reportar

Informe ao usuário:
- **Ação**: criar / atualizar / consolidar / não registrar
- **Arquivo**: caminho completo dentro do Alexandria
- **Justificativa**: por que essa decisão
- **Conexões**: documentos relacionados no Alexandria (se houver)

---

## Estrutura Padrão de Documento

Adapte ao tipo — nem todo campo é obrigatório. Priorize útil sobre completo.

```markdown
# [Título Claro — Frase Que Resume a Ideia]

**Tema**: [categoria]
**Data**: [YYYY-MM-DD]
**Tipo**: [framework | decisão | tese | playbook | modelo | taxonomia | síntese | aprendizado | prompt | arquitetura | skill]

## Contexto
[Por que isso existe — qual problema resolve ou qual insight gerou. 2-4 linhas.]

## Síntese
[O núcleo da ideia. O que deve ser lembrado. 3-5 linhas — não mais.]

## [Estrutura Principal]
[Seções específicas ao conteúdo do documento]

## Implicações Práticas
[Como usar isso — o que muda, o que habilita, o que evita]

## Desdobramentos
[O que isso ainda não resolve, o que abre, próximos passos intelectuais]

## Relacionado
- [Nome do doc relacionado](caminho/relativo.md)
```

**Regras de escrita:**
- Escreva para quem vai ler 6 meses depois — contexto explícito, não implícito
- Estruture por ideia, não pela ordem da conversa
- Distile: capture o insight, não o processo que chegou a ele
- Direto, sem burocracia — mais útil do que bonito

---

## Organização do Alexandria

Antes de sugerir caminho, leia `~/Documentos/Claude/Projects/Alexandria/README.md` se existir — respeite estrutura existente.

Categorias de referência:

```
/frameworks/          — modelos mentais, frameworks próprios de análise
/operacao/            — workflows, processos, modelos operacionais
/arquitetura/         — decisões de sistema, notas de arquitetura
/skills/              — documentação de skills construídas
/teses/               — raciocínios desenvolvidos que viraram convicção
/aprendizados/        — lições extraídas reutilizáveis
/prompts/             — prompts estruturados valiosos com contexto de uso
/taxonomias/          — categorias, classificações, vocabulário controlado
/briefings/           — briefings relevantes preservados
/sinteses/            — resumos consolidados de conversas longas
```

**Regra**: prefira estrutura existente. Só proponha nova pasta se nenhuma existente fizer sentido — e confirme com o usuário.

---

## Modos de Uso

| Modo | Trigger típico | Lógica |
|------|---------------|--------|
| **Registrar conversa** | Conversa colada, "registra isso" | Filtro → extrair núcleo → criar ou atualizar |
| **Registrar insight** | Ideia solta, "anota", insight breve | Estruturar em nota atômica → criar |
| **Atualizar doc** | "atualiza X com isso", novo aprendizado sobre tema existente | Ler doc → integrar → editar |
| **Consolidar** | "consolida os docs sobre X", fragmentos dispersos | Ler todos → unificar → confirmar remoção |
| **Curadoria** | "revisa o Alexandria", "o que está desatualizado?" | Glob → análise → sugestões organizadas |

Para templates detalhados por tipo de documento, leia: `~/.claude/skills/tabulario/templates.md`

---

## Anti-patterns

**Nunca faça:**
- Transcrever a conversa crua — transforme em síntese estruturada
- Criar arquivo novo quando atualizar é o caminho correto
- Registrar "para garantir" sem critério real de valor futuro
- Reescrever documento inteiro para adicionar uma seção
- Criar nova pasta sem confirmar com o usuário
- Commitar sem mensagem descritiva

**Sinais de que o conteúdo não deve ser registrado:**
- Só faz sentido no contexto da conversa atual
- É detalhamento de uma tarefa que já foi concluída e não se repete
- Repete algo já documentado sem acrescentar perspectiva ou profundidade
- É brainstorming sem síntese — precisa "amadurecer" antes de virar doc
- A resposta para a maioria das perguntas do filtro soberano é "não"

---

## Avaliação

### Cenário 1 — Conversa relevante sobre arquitetura de sistema
**Input**: Conversa longa sobre decisões de arquitetura de um sistema multi-agente com LLMs
**Comportamento esperado:**
- [ ] Aplica filtro soberano — conteúdo passa
- [ ] Verifica Alexandria por docs existentes sobre o tema
- [ ] Extrai decisões e estrutura (não transcreve a conversa)
- [ ] Cria doc em `/arquitetura/` com estrutura padrão incluindo contexto e implicações
- [ ] Commita com mensagem clara

### Cenário 2 — Ideia passageira sem valor estrutural
**Input**: "anota: talvez valha testar aquela abordagem diferente amanhã"
**Comportamento esperado:**
- [ ] Aplica filtro soberano — conteúdo não passa
- [ ] Retorna decisão "não registrar" com justificativa clara
- [ ] Não cria nenhum arquivo

### Cenário 3 — Atualização de doc existente
**Input**: "atualiza o doc de workflows com o que aprendi sobre paralelismo de agents"
**Comportamento esperado:**
- [ ] Localiza documento relevante no Alexandria via glob/grep
- [ ] Lê o documento antes de qualquer edição
- [ ] Adiciona nova seção sem reescrever o que já estava correto
- [ ] Usa Edit, não Write
- [ ] Commita com mensagem descritiva
