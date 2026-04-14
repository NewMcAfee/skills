# GOD Changelog

## v2.0 — 2026-03-27 (Loop GOD→GOD)

### Pesquisa realizada
- DSPy, GEPA, TextGrad, OPRO, APE — frameworks de meta-prompt 2026
- Anthropic canonical skill authoring patterns (Skill authoring best practices)
- MAST Taxonomy — 14 modos de falha em sistemas multi-agente (NeurIPS 2025)
- Context Engineering vs Prompt Engineering (Lütke/Schmid 2025)
- awesome-claude-code community skills

### Lacunas identificadas na v1.0
1. Sem framework de avaliação — skills criadas sem cenários de teste
2. Sem calibração de grau de liberdade — instruções sempre no mesmo nível de prescrição
3. Sem progressive disclosure — SKILL.md tratado como dump em vez de nav layer
4. Sem custo de contexto (~1500 tokens/invocação) — verbosidade não era penalizada
5. Sem padrão Claude A/B — nenhum teste com instância fresca
6. Sem anti-patterns documentados — não ensinava o que evitar
7. Sem context engineering — não distinguia "instrução errada" de "contexto incompleto"
8. Module 2 sem estrutura de investigação — busca vaga demais
9. Entrega final sem cenários de avaliação prontos

### Adições em v2.0
- **Módulo 5** (Validação): cenários de avaliação, teste Claude A/B, checklist de qualidade
- **Seção 4.3**: calibração de grau de liberdade com tabela de decisão
- **Seção 4.1**: estrutura de diretório com progressive disclosure explícita
- **Seção 4.2**: regras de description (terceira pessoa, trigger, ação-orientada)
- **Seção 4.5**: loop de feedback obrigatório para skills complexas
- **Princípios** expandidos: context engineering, custo de tokens, falha documentada
- **Module 2** reestruturado: 5 áreas específicas de investigação incluindo modos de falha
- **Module 1** expandido: escopo negativo e critérios de sucesso observáveis
- **Auto-melhoria**: loop GOD→GOD documentado na própria skill

## v1.0 — 2026-03-27 (Versão inicial)
- 4 módulos: Radar, Explorador, Crítico, Forjador
- Princípios básicos de criação de skills
- Entrega: SKILL.md + resumo + exemplo simulado
