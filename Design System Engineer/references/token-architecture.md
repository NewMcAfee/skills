# Arquitetura de Tokens — 3 Camadas

## Por que 3 camadas

Um design system com tokens num único nível (ex: aplicar `Primary/500` diretamente num botão) quebra quando:
- A marca muda de azul para verde — cada componente precisa ser atualizado manualmente
- Dark mode é adicionado — é preciso criar uma cópia completa de todos os componentes
- Uma nova marca é adicionada ao sistema — o sistema não tem onde encaixar

A arquitetura de 3 camadas resolve isso: mudando um primitivo, todos os aliases atualizam automaticamente. Trocando os valores dos modos Light/Dark nos semânticos, todo o sistema muda de tema sem tocar em nenhum componente.

---

## Camada 1 — Primitivos (Raw Values)

**O que são:** Os valores brutos do sistema. A paleta completa de opções — não decisões.

**Regra crítica:** Primitivos nunca são aplicados diretamente a componentes. Eles são a fonte dos aliases semânticos. No Figma, oculte os primitivos da publicação.

**Nomenclatura:** `property/color/step`

```
Primitivos de cor:
Primary/50 → Primary/950      (11 tons)
Secondary/50 → Secondary/950  (11 tons, se houver)
Neutral/50 → Neutral/950      (11 tons)
Success/50 → Success/950      (11 tons)
Warning/50 → Warning/950      (11 tons)
Error/50 → Error/950          (11 tons)
Info/50 → Info/950            (11 tons)

Primitivos de espaçamento:
Spacing/1  = 4px
Spacing/2  = 8px
Spacing/3  = 12px
Spacing/4  = 16px
Spacing/5  = 20px
Spacing/6  = 24px
Spacing/8  = 32px
Spacing/10 = 40px
Spacing/12 = 48px
Spacing/16 = 64px
Spacing/20 = 80px
Spacing/24 = 96px

Primitivos de border-radius:
Radius/None = 0px
Radius/SM   = [base × 0.5]px
Radius/MD   = [base]px          ← base definido pelo caminho visual
Radius/LG   = [base × 1.5]px
Radius/XL   = [base × 2]px
Radius/Pill = 9999px
```

---

## Camada 2 — Semânticos (Intent/Decision)

**O que são:** Aliases que dão significado aos primitivos. Definem onde e como uma cor deve ser usada — não qual cor é.

**Regra crítica:** Semânticos têm modos (Light e Dark). Em cada modo, o alias aponta para um primitivo diferente. É aqui que o dark mode acontece — não nos componentes.

**Nomenclatura:** `Color/Category/Role`

```
Superfícies:
Color/Background/Default      Light: Neutral/50    Dark: Neutral/950
Color/Background/Subtle       Light: Neutral/100   Dark: Neutral/900
Color/Background/Muted        Light: Neutral/200   Dark: Neutral/800
Color/Surface/Card            Light: #FFFFFF       Dark: Neutral/900
Color/Surface/Overlay         Light: rgba(0,0,0,0.05) Dark: rgba(255,255,255,0.05)

Texto:
Color/Foreground/Default      Light: Neutral/900   Dark: Neutral/50
Color/Foreground/Subtle       Light: Neutral/600   Dark: Neutral/400
Color/Foreground/Disabled     Light: Neutral/400   Dark: Neutral/600
Color/Foreground/Inverse      Light: Neutral/50    Dark: Neutral/900

Marca:
Color/Brand/Default           Light: Primary/600   Dark: Primary/400
Color/Brand/Hover             Light: Primary/700   Dark: Primary/300
Color/Brand/Subtle            Light: Primary/100   Dark: Primary/900
Color/Brand/Foreground        Light: #FFFFFF       Dark: Primary/950

Bordas:
Color/Border/Default          Light: Neutral/200   Dark: Neutral/800
Color/Border/Input            Light: Neutral/300   Dark: Neutral/700
Color/Border/Focus            Light: Primary/500   Dark: Primary/400

Semânticas:
Color/Semantic/Success        Light: Success/600   Dark: Success/400
Color/Semantic/Success/Subtle Light: Success/100   Dark: Success/900
Color/Semantic/Warning        Light: Warning/600   Dark: Warning/400
Color/Semantic/Warning/Subtle Light: Warning/100   Dark: Warning/900
Color/Semantic/Error          Light: Error/600     Dark: Error/400
Color/Semantic/Error/Subtle   Light: Error/100     Dark: Error/900
Color/Semantic/Info           Light: Info/600      Dark: Info/400
Color/Semantic/Info/Subtle    Light: Info/100      Dark: Info/900
```

**Nota sobre os steps:** Os steps exatos (Primary/600 no light, Primary/400 no dark) variam conforme a escala gerada. O critério é sempre: garanta contraste WCAG AA (4.5:1) no par foreground/background em ambos os modos antes de fixar o alias.

---

## Camada 3 — Componentes (Specific Usage)

**O que são:** Tokens específicos de componente que referenciam os semânticos. Usados diretamente nos componentes no Figma.

**Regra crítica:** Componentes referenciam semânticos, não primitivos. Isso garante que o dark mode funcione automaticamente em todos os componentes sem exceção.

**Nomenclatura:** `Component/Element/Property/State`

```
Exemplos:
Button/Primary/Background/Default  → Color/Brand/Default
Button/Primary/Background/Hover    → Color/Brand/Hover
Button/Primary/Foreground/Default  → Color/Brand/Foreground
Button/Primary/Border/Default      → transparent

Input/Background/Default           → Color/Background/Default
Input/Border/Default               → Color/Border/Input
Input/Border/Focus                 → Color/Border/Focus
Input/Foreground/Default           → Color/Foreground/Default
Input/Foreground/Placeholder       → Color/Foreground/Subtle

Card/Background/Default            → Color/Surface/Card
Card/Border/Default                → Color/Border/Default
Card/Shadow/Default                → Shadow/MD
```

---

## Estrutura no Figma

```
Variáveis — Coleções:

┌─────────────────────────────────────┐
│ Primitivos (OCULTO DA PUBLICAÇÃO)   │
│   Primary/50 → Primary/950          │
│   Neutral/50 → Neutral/950          │
│   [demais escalas]                  │
│   Spacing/1 → Spacing/24            │
│   Radius/None → Radius/Pill         │
└─────────────────────────────────────┘
              ↓ aliases
┌─────────────────────────────────────┐
│ Tokens (PUBLICADO — modos L/D)      │
│   Color/Background/Default          │
│   Color/Foreground/Default          │
│   Color/Brand/Default               │
│   Color/Border/Default              │
│   Color/Semantic/[Success/Warning…] │
└─────────────────────────────────────┘
              ↓ aliases
┌─────────────────────────────────────┐
│ Componentes (criados via script)    │
│   Button/Primary/Background/Default │
│   Input/Border/Focus                │
│   Card/Background/Default           │
└─────────────────────────────────────┘

Estilos (publicados para compatibilidade):
  Color Styles → espelham os primitivos (para uso em elementos não-componente)
  Text Styles  → H1-H6, Body, Labels
  Effect Styles → Shadow/SM-XL
```

---

## Regras de nomenclatura

1. Use `/` como separador de grupo no Figma — cria subgrupos automaticamente
2. PascalCase para grupos, camelCase para propriedades: `Color/Background/Default` ✅, `color-background-default` ❌
3. Semânticos descrevem intenção, não aparência: `Color/Brand/Subtle` ✅, `Color/LightBlue/100` ❌
4. Estados sempre no final: `/Default`, `/Hover`, `/Focus`, `/Disabled`, `/Active`
5. Nunca use nomes de cor em semânticos: `Color/Primary/500` como semântico ❌ — isso é primitivo, não semântico
