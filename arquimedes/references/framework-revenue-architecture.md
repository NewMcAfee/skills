# Framework: Revenue Architecture & Revenue Operations

## Conceito Central

Revenue Architecture (Winning by Design — Jacco van der Kooij) é o design sistemático de como uma empresa gera, entrega e expande receita de forma recorrente e previsível. Substitui o modelo linear "fechar deal" pelo modelo de **impacto recorrente** centrado no cliente.

---

## O Modelo Bow-Tie

O funil tradicional (só lado esquerdo) ignora metade da receita. O Bow-Tie mostra o modelo completo:

```
[AQUISIÇÃO]                    |    [EXPANSÃO]
Educate → Convince → Purchase  |  Onboard → Adopt → Expand
                              ↕
                    Mutual Commitment
                   (momento de contrato)
```

### Lado Esquerdo — Aquisição (Marketing + Vendas)
| Etapa | O que acontece | Métrica chave |
|---|---|---|
| **Educate** | Prospect descobre que tem um problema | Awareness rate, ICP reach |
| **Convince** | Prospect entende que sua solução resolve | MQL→SQL conversion |
| **Purchase** | Prospect torna-se cliente | Close rate, CAC, Sales cycle |

### Lado Direito — Expansão (CS + Produto)
| Etapa | O que acontece | Métrica chave |
|---|---|---|
| **Onboard** | Cliente atinge o primeiro momento de valor | Time-to-First-Impact |
| **Adopt** | Cliente usa a solução consistentemente | DAU/MAU, feature adoption |
| **Expand** | Cliente compra mais / renova / indica | NRR, Expansion MRR, NPS |

**Princípio fundamental:** o lado direito gera mais receita incremental do que o lado esquerdo na maioria dos modelos recorrentes. Empresas que investem só em aquisição e ignoram expansão têm churn que corrói o crescimento.

---

## SPICED — Framework de Execução GTM (Open Source)

Substitui BANT/MEDDIC para contextos de receita recorrente.

| Letra | Significado | Pergunta central |
|---|---|---|
| **S** — Situation | Contexto atual do cliente | "Como é sua operação hoje?" |
| **P** — Pain | Dor específica e quantificada | "O que esse problema está custando?" |
| **I** — Impact | Impacto do problema não resolvido | "Se não resolver, o que acontece?" |
| **C** — Critical Event | Data/evento que torna urgente | "Por que resolver isso agora?" |
| **E** — Decision | Processo de decisão e stakeholders | "Quem aprova? Quais critérios?" |

**Uso no pipeline:** SPICED não é um script de vendas — é um checklist de qualificação. Oportunidade sem Critical Event identificado tem alta probabilidade de travar no pipeline.

---

## Revenue Operations (RevOps)

RevOps é a função que alinha Marketing, Vendas e Customer Success sob **dados, processos e pipeline compartilhados** para gerar receita previsível.

### O que RevOps resolve
- Silos entre times que têm definições diferentes de "lead qualificado"
- Atribuição de receita por canal sem dados unificados
- Previsão de receita baseada em achismo, não em pipeline

### Métricas de Pipeline (RevOps)

| Métrica | Fórmula | O que indica |
|---|---|---|
| **Pipeline Velocity** | (# oportunidades × win rate × ACV) / sales cycle | Velocidade de geração de receita |
| **Coverage Ratio** | Pipeline total / meta do trimestre | Se há pipeline suficiente para bater a meta |
| **SQL-to-Opportunity** | SQLs que viram oportunidades reais | Qualidade da qualificação |
| **Deal Slippage** | % de deals que passaram da data de fechamento | Previsibilidade do forecast |
| **Lead Velocity Rate** | Crescimento MoM de leads qualificados | Leading indicator de receita futura |

**Regra de cobertura:** pipeline saudável = 3x a meta do trimestre em pipeline ativo. Se Coverage Ratio < 2x, a meta está em risco independentemente de qualquer outra ação.

### Alinhamento Marketing → Vendas → CS

```
Marketing gera leads → Vendas qualifica e fecha → CS retém e expande

Problema de alinhamento mais comum:
Marketing diz "gerou 1.000 leads"
Vendas diz "recebeu 1.000 contatos frios"
→ Definição de MQL não está acordada → gargalo na transição
```

Critério de MQL deve ser co-definido entre marketing e vendas baseado em dados históricos de close rate por segmento de lead.

---

## Revenue Design: 3 Fontes de Receita

Toda receita recorrente saudável tem 3 fontes. Empresa dependente de apenas uma é frágil.

| Fonte | O que é | Métrica |
|---|---|---|
| **Nova Aquisição** | Novos clientes | New MRR |
| **Expansão** | Upsell/cross-sell em base existente | Expansion MRR |
| **Retenção** | Renovação de contratos | Net Revenue Retention (NRR) |

**NRR > 100%:** base existente cresce sozinha (sem contar novos clientes). É o sinal mais forte de product-market fit em SaaS.

**Benchmark NRR:**
- < 90%: negócio com vazamento — aquisição apenas tapa o buraco
- 90–100%: neutro — cresce só com novos clientes
- 100–110%: bom — expansão compensa churn
- > 110%: excelente — benchmark de empresas top quartil

---

## Aplicação: Diagnosticando o Sistema de Receita

**Pergunta diagnóstica principal:** "Qual parte do bow-tie está quebrando?"

| Sintoma | Diagnóstico | Ação |
|---|---|---|
| Alto volume de leads, baixo SQL | Qualificação ruim ou ICP errado | Redefinir MQL; refinar ICP |
| Alto SQL, baixo close rate | Problema na proposta de valor ou processo de vendas | Revisar pitch, pricing, competição |
| Alto new MRR, alto churn | Problema no onboarding ou product-market fit | Acelerar time-to-value; revisar ICP |
| Baixo expansion MRR | CS não identificando oportunidades de expansão | Playbook de upsell; QBRs estruturadas |
| NRR < 100% | Base encolhendo — aquisição apenas compensa | Prioridade máxima: retenção > aquisição |
