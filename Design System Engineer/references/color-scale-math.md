# Matemática de Escala de Cor

## Por que não HSL

HSL (Hue, Saturation, Lightness) não é perceptualmente uniforme. Azul e amarelo com `lightness: 50%` têm luminâncias radicalmente diferentes — o azul parece muito mais escuro. Isso torna impossível criar escalas onde `Primary/500` de uma cor qualquer tenha o mesmo contraste que `Primary/500` de outra.

**O problema prático:** Se você define `color-text-primary: Primary/700` na escala de azul, e depois troca a marca para verde, `Primary/700` verde pode ser muito mais claro e não passar WCAG AA. Com HSL, cada troca de cor exige revalidar todos os pares manualmente.

**A solução:** OKLCH ou HSLuv — espaços de cor perceptualmente uniformes onde Lightness reflete a percepção humana real.

---

## A Regra do 500

O critério de qualidade de uma escala bem construída:

> **Tom 500 deve passar WCAG AA (4.5:1) tanto sobre branco (#FFFFFF) quanto sobre preto (#000000).**

Se o 500 passa nos dois, você garante:
- Tons 600-950 são seguros sobre branco (para texto em light mode)
- Tons 50-400 são seguros sobre preto (para texto em dark mode)
- A escala tem um "ponto de virada" no meio, permitindo o espelhamento automático entre light e dark

Verifique com: https://accessiblepalette.com ou calculando via fórmula WCAG:
`Contraste = (L1 + 0.05) / (L2 + 0.05)` onde L é a luminância relativa (0-1).

---

## Algoritmo de geração — passo a passo

### 1. Identifique a cor âncora (brand color)

Extraia o hex da logo ou do Pacote do Rand. Converta para OKLCH:
- Ferramentas: oklch.com, css.land/lch, ou use a função `oklch()` do CSS moderno
- Anote: L (lightness 0-1), C (chroma), H (hue 0-360)

### 2. Defina os targets de lightness para cada step

Use esta tabela de lightness em OKLCH (valores aproximados — ajuste para a cor específica):

| Step | Lightness alvo | Uso típico |
|------|---------------|------------|
| 50   | 0.97          | Fundo sutil, hover states suaves |
| 100  | 0.93          | Fundo de badges, chips |
| 200  | 0.85          | Bordas suaves, hover backgrounds |
| 300  | 0.75          | Bordas, ícones desabilitados |
| 400  | 0.63          | Textos secundários em dark mode |
| 500  | 0.53          | **Âncora** — deve passar 4.5:1 sobre branco E preto |
| 600  | 0.43          | Textos em light mode, links |
| 700  | 0.34          | Botões primários, CTAs |
| 800  | 0.26          | Hover de botões primários |
| 900  | 0.18          | Textos fortes, headings |
| 950  | 0.12          | Near-black para backgrounds dark |

**Importante:** Esses são targets — ajuste o C (chroma) para manter a saturação visualmente consistente enquanto varia o L.

### 3. Ajuste o hue shift (efeito Abney)

Algumas cores mudam percepção de matiz quando ficam mais claras ou escuras (efeito Abney). Exemplos comuns:
- Azul tende para roxo quando muito escuro → shift H +5 a +10° nos tons 800-950
- Amarelo tende para verde quando muito claro → shift H -5° nos tons 50-100
- Laranja tende para marrom sem hue shift → manter H ou shift -3° nos escuros

### 4. Valide o contraste em cada step

Para cada tom, calcule o contraste sobre #FFF e #000:

```
Luminância de uma cor hex:
1. Normalize os canais: r/255, g/255, b/255
2. Para cada canal: se valor ≤ 0.03928, c = valor/12.92; senão, c = ((valor+0.055)/1.055)^2.4
3. L = 0.2126×r + 0.7152×g + 0.0722×b
4. Contraste vs branco: (1.05) / (L + 0.05)
5. Contraste vs preto: (L + 0.05) / (0.05)
```

**Critérios WCAG 2.2 obrigatórios:**
- Texto normal: ≥ 4.5:1
- Texto grande (≥18px/700 ou ≥24px/400): ≥ 3:1
- Elementos gráficos e ícones: ≥ 3:1
- UI components (bordas de input, etc.): ≥ 3:1

### 5. Documente o contraste de cada step

| Step | Hex | Contraste vs #FFF | Contraste vs #000 | WCAG Status |
|------|-----|-------------------|-------------------|-------------|
| 50   | #   | X:1               | X:1               | BG Only |
| 100  | #   | X:1               | X:1               | BG Only |
| ...  | ... | ...               | ...               | ... |
| 500  | #   | ≥4.5:1            | ≥4.5:1            | ✅ AA Pass ambos |
| 600  | #   | ≥4.5:1            | X:1               | ✅ AA Text Light |
| ...  | ... | ...               | ...               | ... |

---

## Derivando neutros da cor primária

Neutros derivados da primária criam coerência — os cinzas "respiram" na mesma família que a marca.

**Processo:**
1. Tome o H (hue) da primária
2. Reduza C (chroma) para 0.02-0.04 (quase cinza, mas levemente tingido)
3. Gere a escala 50-950 com a mesma tabela de lightness dos primitivos
4. Resultado: escala de neutros que "pertence" à marca

Exemplo: primária azul-marinho H=240° → neutros com leve toque azulado, não cinza genérico.

---

## Cores semânticas — pontos de partida

Use esses valores de hue como ponto de partida, depois gere a escala completa:

| Semântica | H (hue OKLCH) | Cor descritiva |
|-----------|--------------|----------------|
| Success   | ~145°        | Verde médio |
| Warning   | ~75°         | Âmbar/laranja |
| Error     | ~25°         | Vermelho |
| Info      | ~240°        | Azul (mesmo H da primária se for azul, senão azul neutro) |

Ajuste C (chroma) para que o tom 500 de cada semântica passe a regra do 500.

---

## Ferramentas úteis

- **Geração e visualização:** https://accessiblepalette.com (suporta LCh)
- **Conversor OKLCH:** https://oklch.com
- **Verificação de contraste:** https://webaim.org/resources/contrastchecker
- **Referência de escalas de grandes sistemas:** Tailwind CSS, Radix Colors, Material Design 3

---

## Checklist de qualidade da escala

Antes de avançar para os tokens semânticos, verifique:

- [ ] Tom 500 passa 4.5:1 sobre branco?
- [ ] Tom 500 passa 4.5:1 sobre preto?
- [ ] Tons 600-700 passam 4.5:1 sobre branco? (para textos e CTAs em light mode)
- [ ] Tons 300-400 passam 4.5:1 sobre preto? (para textos e ícones em dark mode)
- [ ] Os neutros derivam do hue da primária?
- [ ] Cores semânticas passam a regra do 500?
- [ ] A escala foi gerada em OKLCH ou HSLuv (não HSL simples)?
