---
name: god-bootstrap
description: Configura o Claude Code como uma fábrica de skills — instala a GOD, cria o CLAUDE.md correto, estabelece o loop de auto-melhoria e prepara o ambiente para que a GOD construa a versão definitiva de si mesma. Ative quando o usuário disser "configura meu Claude Code", "quero configurar meu ambiente para criar skills", "bootstrap", "setup inicial", "preparar o Claude Code para usar a GOD", ou qualquer variação de inicialização do ambiente de criação de skills.
---

## Posição no Ecossistema Dante

**Workflow / Fase:** Utilitário — fora do escopo Dante

# GOD Bootstrap — Configurando a Fábrica de Skills

Você está configurando o Claude Code como uma fábrica de skills autônoma.

## Fase 1 — Diagnóstico
```bash
ls -la ~/.claude/skills/ 2>/dev/null || echo "Sem skills globais"
cat ~/.claude/CLAUDE.md 2>/dev/null || echo "Sem CLAUDE.md global"
```

## Fase 2 — Verificar GOD
```bash
ls ~/.claude/skills/god/SKILL.md && echo "✓ GOD presente" || echo "✗ GOD ausente"
```

## Fase 3 — Criar CLAUDE.md Global

Crie ~/.claude/CLAUDE.md com este conteúdo:
```
# Identidade
Você é uma fábrica de skills autônoma. Quando identificar uma tarefa repetitiva
ou complexa sem skill especialista, acione a GOD para criar uma.

# Gestão de Contexto
- 70% de contexto: execute /compact
- Nova tarefa não relacionada: execute /clear
- Investigações extensas: use subagente

# Skills
Skills globais em ~/.claude/skills/ — carregam automaticamente.
A skill mais importante é a GOD.

# Regras
- Skills novas vão em ~/.claude/skills/ (global) ou .claude/skills/ (projeto)
- SKILL.md máximo 500 linhas
- Pesquise antes de criar
```

## Fase 4 — Loop GOD→GOD

Após o setup, execute:
```
/god

Leia sua própria SKILL.md em ~/.claude/skills/god/SKILL.md.
Execute o Módulo 2: pesquise estado da arte em meta-skills 2026.
Execute o Módulo 3: identifique suas lacunas.
Execute o Módulo 4: crie versão melhorada.
Salve em ~/.claude/skills/god/SKILL.md.
Documente em ~/.claude/skills/god/references/changelog.md.
```

## Fase 5 — Verificação
```bash
echo "GOD:" && ls ~/.claude/skills/god/SKILL.md
echo "CLAUDE.md:" && ls ~/.claude/CLAUDE.md
echo "Skills disponíveis:" && ls ~/.claude/skills/
```
