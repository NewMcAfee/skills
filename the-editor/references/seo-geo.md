# SEO/GEO — Diretrizes de Otimização para The Build

## Princípio

Todo episódio do The Build é escrito para ser encontrado no Google (SEO) e citado por IAs (GEO). Os dois objetivos não conflitam — conteúdo com dado próprio, estrutura clara e linguagem direta serve ambos. O Sensei não escreve para robôs; escreve para operadores. O SEO é consequência do conteúdo bom, não objetivo separado.

---

## Título

- 40-60 caracteres
- Keyword natural, não forçada
- Formato que performa: número + resultado + contexto

**Exemplos por pilar:**
| Pilar | Exemplo de título |
|-------|------------------|
| Estudo de Caso | "CAC de R$700 para R$180: o que cohort revelou" |
| Verdade Nua e Crua | "Por que ROAS alto não significa negócio lucrativo" |
| Breaking News | "Meta mudou o algoritmo do Reels: o que muda na prática" |
| Behind The Business | "A TGS chegou a 500 assinantes: o que funcionou" |
| O Arsenal | "Teoria das Restrições: como achar o gargalo real" |
| O Caminho | "14 horas de trabalho e nenhum resultado: o erro que repeti" |

---

## Primeiro Parágrafo (meta description natural)

O primeiro parágrafo funciona como meta description para Google e como preview para IAs. Deve conter:
- A dor ou situação central (o que estava errado)
- O contexto mínimo (quem, onde — sem revelar cliente)
- A promessa do episódio (o que o leitor vai entender)

Comprimento como meta description isolada: 150-160 caracteres.

**Exemplo:**
> "CAC de R$700 para R$180 em 45 dias. Mesmo orçamento, mesmo público, mesma equipe. A diferença foi uma pergunta que ninguém havia feito."

---

## Subtítulos (H2 e H3)

- Formatar como perguntas reais quando possível: "O que o dado revelou?" > "O Diagnóstico"
- Cada H2 é uma unidade completa de significado — IAs extraem H2s como snippets independentes
- Evitar subtítulos genéricos que não indicam conteúdo: "Conclusão", "Resultado", "Análise"
- H3 quando uma seção tem subdivisão legítima — não usar para estilizar parágrafos

---

## Dado Próprio (peso máximo em GEO)

IAs preferem citar fontes com dados originais. Dados do Sensei têm peso GEO superior a dados de terceiros — especialmente dados de clientes anonimizados, dados da TGS e benchmarks próprios.

**Como apresentar dados para máximo impacto GEO:**
- Com contexto: "Em 45 dias, com o mesmo orçamento de mídia..."
- Em comparação direta: "Antes: CAC R$700 | Depois: CAC R$180 | Tempo: 45 dias"
- Com especificidade: "70% dos leads estavam sendo descartados — de um total de 847 leads/mês"

Se citar dado de terceiro, nomear a fonte inline (não em rodapé): "Segundo relatório Substack Q1 2026, a plataforma atingiu 8,4M assinantes pagos."

---

## Slug

Editar manualmente no Substack — nunca usar o gerado automaticamente.

**Formato:** `keyword-principal-contexto` (máx 5 palavras, sem stop words)
**Stop words a remover:** de, o, a, para, com, que, em, e, por, ao

**Exemplos:**
- ✓ `cac-cohort-reducao-45-dias`
- ✓ `leads-descartados-comercial-follow-up`
- ✓ `roas-alto-negocio-prejuizo`
- ✗ `como-eu-reduzi-o-cac-de-r700-para-r180` (longo, com stop words)
- ✗ `episodio-3` (sem keyword)

---

## Tags Substack

4-6 tags por episódio. Tags afetam descoberta no Substack e no Search.

**Tags fixas (sempre incluir pelo menos 2 dessas):**
- `growth`
- `negócios`
- `empreendedorismo`

**Tags específicas por pilar:**

| Pilar | Tags recomendadas |
|-------|------------------|
| Estudo de Caso | `cases`, `dados`, `marketing` |
| Verdade Nua e Crua | `marketing`, `estratégia`, `análise` |
| Breaking News | `tendências`, `IA`, `plataformas` |
| Behind The Business | `building-in-public`, `saas`, `tgs` |
| O Arsenal | `frameworks`, `ferramentas`, `produtividade` |
| O Caminho | `mentalidade`, `liderança`, `filosofia` |

---

## SEO Description

Campo preenchido manualmente no Substack (não é o primeiro parágrafo, é um campo separado).

- 150-160 caracteres exatos
- Contém: dado central + promessa do episódio
- Funciona como razão para clicar no email ou no link compartilhado
- Escrita como frase completa, não como lista de keywords

**Exemplo:**
> "CAC cortado de R$700 para R$180 em 45 dias. Análise de cohort revelou o que o dashboard de mídia escondia — e o que fazer diferente."
> (147 caracteres ✓)

---

## Links Internos (cluster temático)

- Linkar para episódios anteriores do The Build quando relevante
- Constrói cluster temático — Google vê profundidade no assunto e favorece episódios relacionados
- Máximo 3-4 links por episódio — excesso dilui autoridade e dispersa o CTA
- Texto descritivo do link, nunca "clique aqui": "como mostrei no episódio sobre cohort" > "clique aqui"

---

## Estrutura GEO-friendly

IAs (ChatGPT, Perplexity, Claude) extraem conteúdo estruturado para citação. Para maximizar chances de citação:

- **Começo de seção com o conceito mais importante** — não guardar a conclusão para o final
- **Conclusão clara ao final de cada seção** — IAs extraem seções completas
- **Dados numéricos com unidade e contexto** — "47% de redução no CAC em 45 dias" é citável; "redução significativa" não é
- **Evitar linguagem vaga** — "muito", "bastante", "consideravelmente" não geram snippets

**O que IAs preferem citar do The Build:**
1. Dados comparativos antes/depois com dado real
2. Conclusões contraintuitivas com evidência (pilar Verdade Nua e Crua)
3. Frameworks aplicados com resultado mensurável (pilar Arsenal)
4. Bastidores com métricas da TGS (pilar Behind The Business)
