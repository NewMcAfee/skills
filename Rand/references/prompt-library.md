# Prompt Library — Rand Visual Director

## Regra de uso — leia antes de qualquer consulta

Esta biblioteca existe para economizar tempo na **estrutura técnica** dos prompts — parâmetros, exclusões, modificadores de formato, comportamentos conhecidos de cada ferramenta.

**Ela não serve como ponto de partida criativo.** O ponto de partida criativo é sempre, sem exceção:
1. O Caminho Visual aprovado no Passo 3
2. O brandscape mapeado no Passo 2
3. O arquétipo do Brand Core do Campbell

Usar um prompt anterior como base criativa é o caminho mais rápido para fazer logos que se parecem entre si. Consulte a biblioteca para aprender como a ferramenta se comporta — não para repetir o que funcionou esteticamente em outro projeto.

**Fluxo correto:**
1. Defina a direção criativa a partir do Caminho Visual aprovado
2. Consulte a biblioteca para ver como prompts similares foram estruturados tecnicamente
3. Escreva um prompt novo que traduz a direção criativa do projeto atual
4. Ao final do projeto, adicione o aprendizado com o formato abaixo

---

## Parâmetros técnicos por ferramenta

### Ideogram v3 — wordmarks e logos com texto integrado
- Melhor para: wordmarks, logos onde o nome da marca aparece no arquivo gerado
- Parâmetro de estilo: `--style design` para logos flat e vetoriais
- Fundo branco explícito garante melhor isolamento do logo
- Evitar: `realistic`, `photo`, `3D` — aumentam a chance de detalhes que comprometem escalabilidade
- Limitação conhecida: menos controle artístico que Midjourney para marcas altamente expressivas

### Midjourney v6.1 — símbolos, ícones e lettermarks
- Melhor para: símbolos puros, ícones, lettermarks sem texto corrido
- Parâmetros recomendados: `--v 6.1 --style raw --ar 1:1`
- `--style raw` reduz a tendência do modelo de adicionar tratamentos artísticos não solicitados
- `--no text` é obrigatório — Midjourney distorce texto de forma imprevisível
- Exclusões sempre úteis: `--no gradients --no shadows --no 3D --no realistic details --no watermark`
- Limitação conhecida: não renderiza texto confiável — nunca use para wordmarks

---

## Formato de entrada para novos aprendizados

Adicione ao final desta biblioteca após cada projeto finalizado. Sem dados confidenciais do cliente — setor e arquétipo são suficientes.

```markdown
---
**Projeto:** [setor — ex: "fintech B2B" ou "suplementos esportivos"]
**Data:** [mês/ano]
**Ferramenta:** Ideogram v3 / Midjourney v6.1
**Tipo de logo:** wordmark / símbolo / lettermark / combinação
**Arquétipo:** [arquétipo do Brand Core]
**Caminho Visual:** [nome do caminho aprovado]

**Prompt aprovado:**
[prompt exato que gerou o resultado aprovado pelos 8 critérios]

**Resultado:** ✅ Aprovado direto / 🔄 Aprovado após ajuste

**Ajuste feito (se houver):**
[o que mudou entre o prompt inicial e o aprovado, e por quê]

**Aprendizado técnico:**
[o que esse prompt ensina sobre o comportamento da ferramenta — não sobre estética]

**O que NÃO replicar criativamente:**
[elementos visuais específicos desse logo que não devem aparecer em outros projetos]
```

---

## Entradas

*Biblioteca vazia no início. Adicione a primeira entrada ao finalizar o primeiro projeto com o Rand.*
