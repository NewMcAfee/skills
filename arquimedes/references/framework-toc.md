# Framework: Teoria das Restrições (Theory of Constraints)

## Conceito Central

Todo sistema tem exatamente **uma restrição** que limita seu resultado global. Melhorar qualquer parte que não seja a restrição não melhora o sistema — apenas cria falsa sensação de progresso.

**Erro mais comum:** confundir a restrição aparente (o que todo mundo reclama) com a restrição real (o que de fato limita o throughput).

---

## Os 5 Passos POOGI (Process of Ongoing Improvement)

| Passo | Ação | Pergunta Central |
|---|---|---|
| 1. Identificar | Encontre o gargalo real | "O que está impedindo o sistema de gerar mais resultado?" |
| 2. Explorar | Extraia o máximo da restrição atual | "Como aproveitamos 100% da capacidade do gargalo?" |
| 3. Subordinar | Sincronize o resto ao gargalo | "Tudo no sistema está servindo ao gargalo ou criando ruído?" |
| 4. Elevar | Aumente a capacidade do gargalo | "Como expandimos o gargalo quando ele estiver 100% explorado?" |
| 5. Repetir | Volte ao Passo 1 | "Agora que resolvemos, qual é a próxima restrição?" |

**Atenção no Passo 5:** ao elevar uma restrição, a próxima restrição emerge em outro lugar. O processo nunca termina — é contínuo.

---

## Métricas ToC (Throughput Accounting)

Substituem a contabilidade de custo tradicional, que incentiva sub-otimizações locais.

| Métrica | Definição | Foco |
|---|---|---|
| **Throughput (T)** | Taxa com que o sistema gera dinheiro através de vendas | Maximizar |
| **Inventory (I)** | Dinheiro investido em ativos para gerar Throughput | Minimizar |
| **Operating Expense (OE)** | Dinheiro gasto para converter Inventory em Throughput | Minimizar |

**Resultado (Net Profit):** T − OE  
**ROI:** (T − OE) / I

**Implicação prática:** antes de contratar mais pessoas ou comprar mais ferramentas (↑I, ↑OE), pergunte se isso aumenta T. Se não aumentar, é desperdício.

---

## Thinking Processes (TP) — Ferramentas de Diagnóstico

| Ferramenta | Para que serve |
|---|---|
| **Current Reality Tree (CRT)** | Mapeia causas e efeitos do problema atual — identifica a causa-raiz |
| **Evaporating Cloud (EC)** | Resolve conflitos aparentes encontrando o pressuposto incorreto |
| **Future Reality Tree (FRT)** | Valida se a solução proposta realmente resolve o problema sem criar novos |
| **Prerequisite Tree (PRT)** | Identifica obstáculos para implementar a solução |
| **Transition Tree (TrT)** | Monta o plano de implementação passo a passo |

Para uso no pipeline: CRT (diagnóstico) e EC (conflitos estratégicos) são os mais aplicáveis.

---

## Aplicação em Funil de Receita

### Identificando a Restrição no Funil

```
Awareness → Lead → MQL → SQL → Oportunidade → Cliente → Retenção
```

Calcule a taxa de conversão entre cada etapa. **A menor taxa relativa é o gargalo.**

| Tipo de Restrição | Sintoma | Solução (Passo 4) |
|---|---|---|
| Topo do funil (awareness) | Poucos leads entrando | Mais budget/alcance em mídia |
| Meio do funil (qualificação) | Leads não viram SQL | Melhor ICP, qualificação, nurturing |
| Fundo do funil (fechamento) | Oportunidades não fecham | Processo de vendas, oferta, pricing |
| Pós-venda (retenção) | Churn alto | Onboarding, CS, produto |

**Erro clássico:** investir em aquisição quando a restrição é retenção. Encher o funil com buraco no fundo.

### Subordinando ao Gargalo (Passo 3)

Se a restrição é o SQL-to-close (vendas não fecham):
- ❌ Errado: contratar mais SDRs para gerar mais leads
- ✅ Certo: focar SDRs só em oportunidades que já mostraram fit; melhorar playbook de fechamento

Se a restrição é awareness (poucas pessoas conhecem):
- ❌ Errado: otimizar landing page indefinidamente
- ✅ Certo: aumentar alcance de topo — mídia paga, PR, distribuição de conteúdo

---

## Perguntas de Diagnóstico Rápido

1. "Em qual etapa do funil o volume cai mais drasticamente?"
2. "Qual é a taxa de conversão de cada etapa? Qual está mais longe do benchmark?"
3. "Se você dobrasse a capacidade do que você está otimizando hoje, o resultado dobraria? Se não, você está no lugar errado."
4. "O que acontece se o gargalo não for resolvido nos próximos 90 dias?"
