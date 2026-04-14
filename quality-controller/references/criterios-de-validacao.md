# Critérios de Validação por Tipo de Output

Referência detalhada para as 5 camadas. Consulte quando o contexto pedir granularidade maior que o SKILL.md oferece.

---

## Copy (Escriba)

### Camada 1 — Coerência Estratégica
- O `icp_product_map_id` referenciado no briefing é o que a copy endereça?
- O nível de consciência (UNC/PRB/SOL/PRO/MIX) está correto para o formato e canal?
- A PUV declarada no Da Vinci está presente — não foi substituída por benefício genérico?
- O gancho ativa a dor ou desejo correto para o avatar declarado?
- A copy não faz promessas que o produto não entrega (verificar na PUV)?

### Camada 2 — Clareza de Mensagem
- Headline comunica o benefício central em menos de 8 segundos de leitura?
- O CTA instrui uma ação específica (não "saiba mais" genérico quando há ação concreta)?
- Há uma única mensagem central — não duas promessas disputando atenção?
- O leitor sabe o que fazer ao terminar de ler?

### Camada 3 — Consistência de Marca
- Tom de voz alinhado com os atributos declarados no Brand Core?
- Vocabulário proibido ausente (verificar Copy System se existir)?
- Nível de formalidade consistente com outros materiais aprovados?

### Camada 4 — Solidez Operacional
- Caracteres dentro do limite da plataforma (META: headline ≤ 40 / texto ≤ 125; Google: título ≤ 30 / descrição ≤ 90)?
- Links ou menções de URL corretos e funcionais?
- Nenhuma sensibilidade da conta violada (temas, promessas, linguagem)?
- Para e-mail: assunto dentro de 50 caracteres para mobile?

### Camada 5 — Qualidade Percebida
- Gramática e ortografia corretas?
- Pontuação consistente (não mistura estilos)?
- Copy não soa gerada por IA sem revisão humana (construções artificiais, repetição de palavras)?
- Nível de acabamento compatível com o estágio declarado (rascunho vs. final)?

---

## Asset Visual (Takehiko)

### Camada 1 — Coerência Estratégica
- A direção criativa executa o gancho declarado no Briefing Criativo?
- O avatar visualmente representado corresponde ao ICP do `icp_product_map`?
- O asset serve ao formato e nível de consciência declarados?

### Camada 2 — Clareza de Mensagem
- Existe um único focal point visual — o olhar sabe onde ir primeiro?
- Hierarquia tipográfica: headline visível antes de qualquer elemento secundário?
- Em mobile (preview reduzido), a mensagem principal ainda é legível?
- Para carrossel: Slide 1 funciona como standalone (gancho suficiente sem os demais)?

### Camada 3 — Consistência de Marca
- Paleta de cores usa os valores HEX exatos do Design System?
- Tipografia segue as famílias e pesos definidos no Design System?
- Logo aplicado segundo as guidelines (espaçamento mínimo, versão correta)?
- Grid e espaçamento respeitam os tokens definidos?

### Camada 4 — Solidez Operacional
- Dimensões corretas para a plataforma (ver tabela abaixo)?
- Formato de arquivo correto (PNG/JPG/MP4 conforme plataforma)?
- Peso dentro do limite aceitável (META: <30MB vídeo, <30MB imagem)?
- `AD-ID` e `test_id` documentados no handoff?
- Texto sobre imagem respeita a regra de 20% de área (META)?

**Specs de referência:**
| Plataforma | Formato | Dimensão |
|---|---|---|
| META Feed | Quadrado | 1080x1080px |
| META Feed | Vertical | 1080x1350px |
| META Stories/Reels | Vertical | 1080x1920px |
| Google Display | Retângulo | 300x250px |
| Google Display | Leaderboard | 728x90px |
| Google Display | Half Page | 300x600px |
| YouTube | Horizontal | 1920x1080px |
| LinkedIn | Horizontal | 1200x627px |

### Camada 5 — Qualidade Percebida
- Imagens sem pixelação ou compressão visível?
- Elementos alinhados (sem deslocamento casual)?
- Cores consistentes (sem variações de tom não intencionais)?
- Para vídeo: cortes limpos, áudio sem ruído, legendas sincronizadas?

---

## Output Estratégico (Documento)

### Camada 1 — Coerência Estratégica
- O documento responde ao objetivo declarado no briefing de origem?
- As recomendações são consistentes com os dados apresentados?
- Há coerência interna (as conclusões seguem das premissas)?

### Camada 2 — Clareza de Mensagem
- O leitor entende a recomendação principal sem ler o documento inteiro?
- Existe sumário executivo ou destaque da conclusão principal?
- Jargão explicado quando necessário para o nível do interlocutor?

### Camada 3 — Consistência de Marca
- Tom adequado ao nível de relacionamento com o cliente (técnico vs. executivo)?
- Formatação consistente com outros documentos da conta?

### Camada 4 — Solidez Operacional
- Dados referenciados com fonte identificável?
- Métricas calculadas corretamente (verificar cálculos críticos)?
- Recomendações acionáveis — não vagas demais para implementar?
- Dependências e riscos declarados?

### Camada 5 — Qualidade Percebida
- Formatação consistente (títulos, bullets, tabelas no mesmo estilo)?
- Sem erros de digitação ou nomenclatura inconsistente?
- Nível de profissionalismo compatível com a entrega ao cliente?

---

## Escala de Decisão: Erro vs. Opinião

Use este teste para qualquer achado antes de classificar:

| Pergunta | Se SIM | Se NÃO |
|---------|--------|--------|
| Viola critério documentado (Brand Core, Design System, spec de plataforma)? | Crítico ou Importante | Vai para próxima |
| Compromete o objetivo do briefing (ICP errado, PUV ausente, dado errado)? | Crítico | Vai para próxima |
| Viola sensibilidade conhecida da conta? | Crítico | Vai para próxima |
| Compromete clareza para o ICP (mensagem ambígua ou ausente)? | Importante | Vai para próxima |
| É inconsistência com padrão estabelecido mas não documentado? | Recomendado | Não é achado |
| É preferência sobre algo igualmente válido? | Não é achado | — |
