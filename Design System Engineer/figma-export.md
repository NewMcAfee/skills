# Figma Export — Templates e Padrões

## Estrutura do script Plugin API

O script deve criar os elementos na seguinte ordem (dependências primeiro):

```
1. Coleção "Primitivos" → variáveis de cor (ocultas da publicação)
2. Coleção "Tokens" → aliases semânticos com modos Light e Dark
3. Coleção "Spacing" → variáveis de espaçamento e border-radius
4. Color Styles → espelham primitivos (para compatibilidade com elementos não-componente)
5. Text Styles → escala tipográfica completa
6. Effect Styles → sombras 4 níveis
```

---

## Template base do script

```javascript
// Design System Export Script
// Gerado pelo DSE — cole em Plugins > Development > New Plugin > code.ts

figma.showUI(__html__, { width: 400, height: 300 });

async function createDesignSystem() {
  
  // ══════════════════════════════════════════
  // TOKENS — substitua pelos valores calculados
  // ══════════════════════════════════════════
  
  const BRAND = {
    name: "{{BRAND_NAME}}",
    primaryColor: "{{PRIMARY_HEX}}", // ex: "#0A1628"
    headingFont: "{{HEADING_FONT}}",  // ex: "Space Grotesk"
    bodyFont: "{{BODY_FONT}}",        // ex: "DM Sans"
    baseRadius: {{BASE_RADIUS}},      // ex: 8
  };
  
  // Primitivos de cor — gerados pelo DSE
  const primitives = {
    primary: {
      "50":  "{{PRIMARY_50}}",
      "100": "{{PRIMARY_100}}",
      "200": "{{PRIMARY_200}}",
      "300": "{{PRIMARY_300}}",
      "400": "{{PRIMARY_400}}",
      "500": "{{PRIMARY_500}}",
      "600": "{{PRIMARY_600}}",
      "700": "{{PRIMARY_700}}",
      "800": "{{PRIMARY_800}}",
      "900": "{{PRIMARY_900}}",
      "950": "{{PRIMARY_950}}",
    },
    neutral: { /* mesma estrutura */ },
    success: { /* mesma estrutura */ },
    warning: { /* mesma estrutura */ },
    error:   { /* mesma estrutura */ },
    info:    { /* mesma estrutura */ },
  };

  // Tokens semânticos — Light mode (valores de exemplo)
  const semanticLight = {
    "Color/Background/Default":      primitives.neutral["50"],
    "Color/Background/Subtle":       primitives.neutral["100"],
    "Color/Background/Muted":        primitives.neutral["200"],
    "Color/Surface/Card":            "#FFFFFF",
    "Color/Foreground/Default":      primitives.neutral["900"],
    "Color/Foreground/Subtle":       primitives.neutral["600"],
    "Color/Foreground/Disabled":     primitives.neutral["400"],
    "Color/Foreground/Inverse":      primitives.neutral["50"],
    "Color/Brand/Default":           primitives.primary["600"],
    "Color/Brand/Hover":             primitives.primary["700"],
    "Color/Brand/Subtle":            primitives.primary["100"],
    "Color/Brand/Foreground":        "#FFFFFF",
    "Color/Border/Default":          primitives.neutral["200"],
    "Color/Border/Input":            primitives.neutral["300"],
    "Color/Border/Focus":            primitives.primary["500"],
    "Color/Semantic/Success":        primitives.success["600"],
    "Color/Semantic/Success/Subtle": primitives.success["100"],
    "Color/Semantic/Warning":        primitives.warning["600"],
    "Color/Semantic/Warning/Subtle": primitives.warning["100"],
    "Color/Semantic/Error":          primitives.error["600"],
    "Color/Semantic/Error/Subtle":   primitives.error["100"],
    "Color/Semantic/Info":           primitives.info["600"],
    "Color/Semantic/Info/Subtle":    primitives.info["100"],
  };

  // Tokens semânticos — Dark mode
  const semanticDark = {
    "Color/Background/Default":      primitives.neutral["950"],
    "Color/Background/Subtle":       primitives.neutral["900"],
    "Color/Background/Muted":        primitives.neutral["800"],
    "Color/Surface/Card":            primitives.neutral["900"],
    "Color/Foreground/Default":      primitives.neutral["50"],
    "Color/Foreground/Subtle":       primitives.neutral["400"],
    "Color/Foreground/Disabled":     primitives.neutral["600"],
    "Color/Foreground/Inverse":      primitives.neutral["900"],
    "Color/Brand/Default":           primitives.primary["400"],
    "Color/Brand/Hover":             primitives.primary["300"],
    "Color/Brand/Subtle":            primitives.primary["900"],
    "Color/Brand/Foreground":        primitives.primary["950"],
    "Color/Border/Default":          primitives.neutral["800"],
    "Color/Border/Input":            primitives.neutral["700"],
    "Color/Border/Focus":            primitives.primary["400"],
    "Color/Semantic/Success":        primitives.success["400"],
    "Color/Semantic/Success/Subtle": primitives.success["900"],
    "Color/Semantic/Warning":        primitives.warning["400"],
    "Color/Semantic/Warning/Subtle": primitives.warning["900"],
    "Color/Semantic/Error":          primitives.error["400"],
    "Color/Semantic/Error/Subtle":   primitives.error["900"],
    "Color/Semantic/Info":           primitives.info["400"],
    "Color/Semantic/Info/Subtle":    primitives.info["900"],
  };

  // ══════════════════════════════════════════
  // HELPER: hex para RGB Figma (0-1)
  // ══════════════════════════════════════════
  function hexToRGB(hex) {
    const h = hex.replace('#', '');
    return {
      r: parseInt(h.substring(0,2), 16) / 255,
      g: parseInt(h.substring(2,4), 16) / 255,
      b: parseInt(h.substring(4,6), 16) / 255,
    };
  }

  // ══════════════════════════════════════════
  // 1. COLEÇÃO PRIMITIVOS (oculta da publicação)
  // ══════════════════════════════════════════
  const primCollection = figma.variables.createVariableCollection("Primitivos");
  primCollection.renameMode(primCollection.modes[0].modeId, "Value");
  
  const primVars = {};
  
  for (const [colorName, scale] of Object.entries(primitives)) {
    for (const [step, hex] of Object.entries(scale)) {
      const varName = `${colorName.charAt(0).toUpperCase() + colorName.slice(1)}/${step}`;
      const variable = figma.variables.createVariable(varName, primCollection, "COLOR");
      variable.setValueForMode(primCollection.modes[0].modeId, hexToRGB(hex));
      // Ocultar da publicação — não aplicar diretamente em componentes
      variable.hiddenFromPublishing = true;
      variable.scopes = ["ALL_SCOPES"]; // disponível internamente para aliases
      primVars[varName] = variable;
    }
  }

  // ══════════════════════════════════════════
  // 2. COLEÇÃO TOKENS SEMÂNTICOS (modos Light + Dark)
  // ══════════════════════════════════════════
  const tokenCollection = figma.variables.createVariableCollection("Tokens");
  tokenCollection.renameMode(tokenCollection.modes[0].modeId, "Light");
  const darkModeId = tokenCollection.addMode("Dark");
  const lightModeId = tokenCollection.modes[0].modeId;

  for (const tokenName of Object.keys(semanticLight)) {
    const variable = figma.variables.createVariable(tokenName, tokenCollection, "COLOR");
    variable.setValueForMode(lightModeId, hexToRGB(semanticLight[tokenName]));
    variable.setValueForMode(darkModeId, hexToRGB(semanticDark[tokenName]));
  }

  // ══════════════════════════════════════════
  // 3. COLEÇÃO SPACING + RADIUS
  // ══════════════════════════════════════════
  const spacingCollection = figma.variables.createVariableCollection("Spacing");
  spacingCollection.renameMode(spacingCollection.modes[0].modeId, "Value");
  const spacingModeId = spacingCollection.modes[0].modeId;

  const spacingScale = [
    ["Spacing/1", 4], ["Spacing/2", 8], ["Spacing/3", 12], ["Spacing/4", 16],
    ["Spacing/5", 20], ["Spacing/6", 24], ["Spacing/8", 32], ["Spacing/10", 40],
    ["Spacing/12", 48], ["Spacing/16", 64], ["Spacing/20", 80], ["Spacing/24", 96],
  ];
  
  const baseRadius = BRAND.baseRadius;
  const radiusScale = [
    ["Radius/None", 0],
    ["Radius/SM",   Math.round(baseRadius * 0.5)],
    ["Radius/MD",   baseRadius],
    ["Radius/LG",   Math.round(baseRadius * 1.5)],
    ["Radius/XL",   baseRadius * 2],
    ["Radius/Pill", 9999],
  ];

  for (const [name, value] of [...spacingScale, ...radiusScale]) {
    const variable = figma.variables.createVariable(name, spacingCollection, "FLOAT");
    variable.setValueForMode(spacingModeId, value);
  }

  // ══════════════════════════════════════════
  // 4. COLOR STYLES (para compatibilidade)
  // ══════════════════════════════════════════
  const colorStylesData = {
    // Primária
    ...Object.fromEntries(
      Object.entries(primitives.primary).map(([step, hex]) =>
        [`Primary/${step}`, hex]
      )
    ),
    // Neutros
    ...Object.fromEntries(
      Object.entries(primitives.neutral).map(([step, hex]) =>
        [`Neutral/${step}`, hex]
      )
    ),
    // Semânticas
    "Semantic/Success": primitives.success["500"],
    "Semantic/Warning": primitives.warning["500"],
    "Semantic/Error":   primitives.error["500"],
    "Semantic/Info":    primitives.info["500"],
  };

  for (const [name, hex] of Object.entries(colorStylesData)) {
    const style = figma.createPaintStyle();
    style.name = name;
    style.paints = [{ type: 'SOLID', color: hexToRGB(hex) }];
  }

  // ══════════════════════════════════════════
  // 5. TEXT STYLES
  // ══════════════════════════════════════════
  // Base 16px, razão Major Third (1.25) — ajuste conforme análise paramétrica
  const BASE = 16;
  const RATIO = {{TYPE_RATIO}}; // ex: 1.25 (Major Third) ou 1.333 (Perfect Fourth)
  
  const typeScale = [
    { name: "Heading/H1", size: Math.round(BASE * Math.pow(RATIO, 5)), weight: 700, lineHeight: 1.2, font: BRAND.headingFont },
    { name: "Heading/H2", size: Math.round(BASE * Math.pow(RATIO, 4)), weight: 600, lineHeight: 1.2, font: BRAND.headingFont },
    { name: "Heading/H3", size: Math.round(BASE * Math.pow(RATIO, 3)), weight: 600, lineHeight: 1.25, font: BRAND.headingFont },
    { name: "Heading/H4", size: Math.round(BASE * Math.pow(RATIO, 2)), weight: 500, lineHeight: 1.3, font: BRAND.headingFont },
    { name: "Heading/H5", size: Math.round(BASE * RATIO),              weight: 500, lineHeight: 1.35, font: BRAND.headingFont },
    { name: "Heading/H6", size: BASE,                                  weight: 600, lineHeight: 1.4, font: BRAND.headingFont },
    { name: "Body/Large",   size: BASE,                                weight: 400, lineHeight: 1.5, font: BRAND.bodyFont },
    { name: "Body/Regular", size: Math.round(BASE / RATIO),            weight: 400, lineHeight: 1.5, font: BRAND.bodyFont },
    { name: "Body/Small",   size: Math.round(BASE / Math.pow(RATIO,2)),weight: 400, lineHeight: 1.5, font: BRAND.bodyFont },
    { name: "Body/XSmall",  size: Math.round(BASE / Math.pow(RATIO,3)),weight: 400, lineHeight: 1.5, font: BRAND.bodyFont },
    { name: "Label/Large",  size: Math.round(BASE / RATIO),            weight: 500, lineHeight: 1.3, font: BRAND.bodyFont },
    { name: "Label/Regular",size: Math.round(BASE / Math.pow(RATIO,2)),weight: 500, lineHeight: 1.3, font: BRAND.bodyFont },
  ];

  for (const { name, size, weight, lineHeight, font } of typeScale) {
    await figma.loadFontAsync({ family: font, style: weight === 700 ? "Bold" : weight === 600 ? "SemiBold" : weight === 500 ? "Medium" : "Regular" });
    const style = figma.createTextStyle();
    style.name = name;
    style.fontName = { family: font, style: weight === 700 ? "Bold" : weight === 600 ? "SemiBold" : weight === 500 ? "Medium" : "Regular" };
    style.fontSize = size;
    style.lineHeight = { unit: "PIXELS", value: Math.round(size * lineHeight / 4) * 4 }; // arredonda ao múltiplo de 4px
  }

  // ══════════════════════════════════════════
  // 6. EFFECT STYLES (sombras)
  // ══════════════════════════════════════════
  const shadows = [
    { name: "Shadow/SM", offsetX: 0, offsetY: 1, blur: 4,  spread: 0, opacity: 0.08 },
    { name: "Shadow/MD", offsetX: 0, offsetY: 4, blur: 8,  spread: 0, opacity: 0.10 },
    { name: "Shadow/LG", offsetX: 0, offsetY: 8, blur: 16, spread: 0, opacity: 0.12 },
    { name: "Shadow/XL", offsetX: 0, offsetY: 16,blur: 24, spread: 0, opacity: 0.14 },
  ];

  for (const { name, offsetX, offsetY, blur, spread, opacity } of shadows) {
    const style = figma.createEffectStyle();
    style.name = name;
    style.effects = [{
      type: "DROP_SHADOW",
      color: { r: 0, g: 0, b: 0, a: opacity },
      offset: { x: offsetX, y: offsetY },
      radius: blur,
      spread: spread,
      visible: true,
      blendMode: "NORMAL",
    }];
  }

  figma.closePlugin("✅ Design System criado com sucesso!");
}

createDesignSystem().catch(err => figma.closePlugin(`❌ Erro: ${err.message}`));
```

---

## Placeholders para substituição

Antes de entregar o script ao usuário, substitua todos os `{{PLACEHOLDER}}` pelos valores calculados na análise paramétrica:

| Placeholder | O que substituir |
|---|---|
| `{{BRAND_NAME}}` | Nome da marca |
| `{{PRIMARY_HEX}}` | Hex da cor primária |
| `{{HEADING_FONT}}` | Nome da fonte Heading (ex: "Space Grotesk") |
| `{{BODY_FONT}}` | Nome da fonte Body (ex: "DM Sans") |
| `{{BASE_RADIUS}}` | Valor numérico do border-radius base (ex: 8) |
| `{{PRIMARY_50}}` a `{{PRIMARY_950}}` | Hexes calculados de cada step |
| `{{TYPE_RATIO}}` | Razão modular numérica (ex: 1.25 ou 1.333) |

Os hexes dos neutros, success, warning, error e info seguem o mesmo padrão.

---

## Checklist antes de entregar o script

- [ ] Todos os placeholders foram substituídos?
- [ ] Os hexes dos primitivos foram gerados em OKLCH/HSLuv (não HSL)?
- [ ] O tom 500 de cada escala passa a regra do 500?
- [ ] Os semânticos de dark mode foram calculados (não apenas invertidos)?
- [ ] A razão tipográfica foi definida (1.25 ou 1.333)?
- [ ] Os primitivos estão marcados como `hiddenFromPublishing: true`?
- [ ] Os text styles carregam as fontes com `figma.loadFontAsync` antes de criar?

---

## Instruções de execução (para entregar ao usuário)

```
Como executar o script no Figma:

1. Abra o Figma Desktop (não funciona no browser)
2. Abra o arquivo onde quer o design system
3. Menu: Plugins > Development > New Plugin
4. Selecione "Figma design" > "Empty canvas"
5. No arquivo code.ts que abrir, apague o conteúdo e cole o script
6. Salve (Cmd+S / Ctrl+S)
7. Menu: Plugins > Development > [Nome do plugin]
8. Execute — levará aproximadamente 5-10 segundos

O que será criado:
- Coleção "Primitivos" (oculta — não aparecer na publicação)
- Coleção "Tokens" com modos Light e Dark
- Coleção "Spacing" com escala de espaçamento e border-radius
- Color Styles, Text Styles e Effect Styles

Após executar:
- Publique a biblioteca: Assets > Publish changes
- Para usar os tokens semânticos: aplique variáveis da coleção "Tokens"
- Para mudar de Light para Dark: selecione um frame > Variables > mudar modo para "Dark"
```
