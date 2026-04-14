# Framework: Unit Economics, Corporate Finance & North Star Metric

## Por que Unit Economics importa

Unit economics responde à pergunta mais importante de qualquer negócio: **"Quanto ganho por cada real que invisto em crescimento?"** Sem essa resposta, qualquer decisão de budget é aposta.

---

## 1. CAC — Customer Acquisition Cost

**Fórmula básica:**
```
CAC = (Gastos de Marketing + Gastos de Vendas) / Novos Clientes no Período
```

**Variações importantes:**

| Tipo | Inclui | Quando usar |
|---|---|---|
| CAC Blended | Todo gasto de mktg+vendas | Visão geral de saúde |
| CAC Paid | Apenas canais pagos | Para decisões de mídia |
| CAC Orgânico | Apenas canais orgânicos | Para ROI de conteúdo/SEO |
| CAC por Canal | Gasto do canal / clientes do canal | Para alocar budget |

**Benchmarks CAC Payback por ACV (2024-2025):**
| ACV | Payback ideal | Payback mediano |
|---|---|---|
| < R$5K | < 6 meses | 9 meses |
| R$5K–R$50K | < 12 meses | 15 meses |
| R$50K–R$200K | < 18 meses | 22 meses |
| > R$200K | < 24 meses | 28 meses |

---

## 2. LTV — Lifetime Value

**Fórmula básica (modelo recorrente):**
```
LTV = (Ticket Médio × Margem Bruta) / Churn Rate Mensal
```

**Exemplo:**
- Ticket: R$1.000/mês | Margem Bruta: 70% | Churn: 2%/mês
- LTV = (R$1.000 × 0,70) / 0,02 = R$35.000

**Armadilhas comuns:**
- Usar churn anual quando deveria ser mensal (superestima LTV em 12x)
- Não incluir margem bruta (superestima LTV em 1/margem)
- Ignorar expansion revenue (subestima LTV quando NRR > 100%)

**LTV com expansão:**
```
LTV ajustado = (Ticket Médio × Margem Bruta) / (Churn - Expansion Rate)
```
Se expansion rate > churn rate → LTV é matematicamente infinito → use payback como métrica primária.

---

## 3. LTV:CAC — A Métrica de Saúde do Negócio

| Ratio | Diagnóstico | Ação |
|---|---|---|
| < 1:1 | Negócio destrói valor — cada cliente custa mais do que gera | Parar de crescer; revisar modelo |
| 1:1 a 2:1 | Marginal — dificilmente sustentável com overhead | Reduzir CAC ou aumentar LTV urgentemente |
| 3:1 | Benchmark saudável — referência do mercado | Manter e escalar com disciplina |
| > 5:1 | Pode estar sub-investindo em crescimento | Considerar acelerar aquisição |

**Benchmark LTV:CAC 2024-2025:** mediana SaaS = 3,6:1

---

## 4. CAC Payback Period

**Fórmula:**
```
Payback (meses) = CAC / (Ticket Médio × Margem Bruta)
```

**Por que é mais útil que LTV:CAC em estágios iniciais:** LTV é uma projeção; Payback é uma certeza. Um negócio com Payback de 8 meses sobrevive a crises; um negócio com Payback de 24 meses precisa de capital constante.

**Relação com decisão de GTM Motion (para César — Bloco 3):**
| ACV e Payback | Implicação de Motion |
|---|---|
| ACV < R$5K + Payback > 12m | SLG financeiramente inviável (SDR ≈ R$5-8K/mês) → PLG/MLG |
| ACV R$5K–R$50K | Híbrido pode funcionar se inside sales for eficiente |
| ACV > R$50K | SLG com ciclo consultivo geralmente suportável |

---

## 5. Contribution Margin

**Fórmula:**
```
Contribution Margin = (Receita - COGS Variável) / Receita
```

Indica eficiência operacional antes de overhead. Negócios SaaS saudáveis têm CM > 60-70%.

**Rule of 40:**
```
Rule of 40 Score = Crescimento de Receita (%) + Margem EBITDA (%)
```
- Score ≥ 40%: negócio saudável (equilíbrio crescimento + rentabilidade)
- Score ≥ 60%: top quartil do mercado
- Score < 20%: atenção — crescimento não está sendo acompanhado por eficiência

**Magic Number (eficiência de aquisição):**
```
Magic Number = (MRR atual - MRR período anterior) × 4 / Gastos S&M período anterior
```
- > 1,0: escalar agressivamente — cada R$1 gasto em S&M retorna > R$1 em ARR
- 0,75–1,0: escalar com cuidado
- < 0,5: problema estrutural — revisar ICP ou processo de vendas

---

## 6. North Star Metric (NSM)

**Definição:** A única métrica que captura o valor core que o produto entrega ao cliente E é leading indicator de receita recorrente. Muda menos que OKRs e alinha toda a empresa.

**Stack completo de métricas:**
```
NSM → "qual valor estamos entregando?" (semestral/anual)
OKR → "como avançamos no NSM este trimestre?" (trimestral)
KPI → "como operamos bem hoje?" (semanal/mensal)
```

**Como escolher a NSM:**

| Tipo de negócio | NSM candidata | Por quê |
|---|---|---|
| SaaS enterprise | MRR ativo ou ARR | Receita recorrente como proxy de valor |
| SaaS self-serve | Usuários ativos pagantes / Trial-to-paid | Adoção como proxy |
| Marketplace | GMV ou # transações ativas | Volume de valor trocado |
| Consumo/API | Unidades consumidas (calls, GB, eventos) | Uso como proxy de valor |
| Serviço/consultoria | # clientes com NPS > 8 ativos | Satisfação como proxy de retenção |

**Critérios de uma boa NSM:**
1. Conecta o valor entregue ao cliente com receita recorrente
2. É leading indicator (muda antes da receita)
3. Toda a empresa pode influenciá-la (não só 1 time)
4. É mensurável semanalmente

**NSM ruim:** receita total (lagging, não leading), NPS isolado (não conecta com receita), número de logins (vaidade, não valor).

---

## 7. Breakeven e Viabilidade de Budget

**Fórmula de Clientes para Breakeven:**
```
Clientes para breakeven = Custos Fixos Mensais / (Ticket Médio × Margem Bruta)
```

**CAC Máximo Tolerável:**
```
CAC máximo = LTV / 3  (para manter LTV:CAC ≥ 3:1)
```

**Budget Máximo de Aquisição:**
```
Budget máximo = CAC máximo × Meta de novos clientes no período
```

**Verificação de viabilidade do plano de mídia:**
1. Calcule CAC máximo tolerável
2. Estime CAC esperado do canal (baseado em benchmarks ou histórico)
3. Se CAC esperado > CAC máximo → o canal não é viável no volume atual → escale apenas se houver caminho para reduzir CAC
4. Se Budget disponível < Budget necessário → ajuste a meta de clientes, não o CAC máximo
