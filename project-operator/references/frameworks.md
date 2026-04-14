# Frameworks de Priorização — Project Operator

## RICE — Cálculo e Adaptação para o Dante

### Fórmula padrão
```
RICE Score = (Reach × Impact × Confidence) / Effort
```

### Escalas adaptadas para contexto de growth/execução

**Reach — quantas áreas, etapas do funil, stakeholders ou entregáveis o item impacta**
| Score | Critério |
|-------|----------|
| 1 | Afeta apenas 1 tarefa ou 1 pessoa |
| 3 | Afeta 1 área ou workflow inteiro |
| 5 | Afeta 2 workflows |
| 8 | Afeta os 3 workflows (Growth IA + Design IA + Governança IA) |
| 10 | Afeta a conta inteira + cliente + próximas reuniões |

**Impact — quanto o item move resultado, clareza, retenção ou avanço do projeto**
| Score | Critério |
|-------|----------|
| 0.25 | Mínimo — small nice-to-have |
| 0.5 | Baixo — melhoria incremental |
| 1 | Médio — move o projeto de forma perceptível |
| 2 | Alto — destrava algo importante ou previne risco real |
| 3 | Massivo — impacto direto em resultado, conta ou fase do projeto |

**Confidence — nível de evidência de que este item vale a pena**
| % | Critério |
|---|---------|
| 30% | Apenas percepção interna — hipótese não validada |
| 50% | Há sinal operacional ou dado de contexto |
| 70% | Há dado de performance ou feedback do cliente |
| 90% | Há decisão explícita do operador ou dado concreto da conta |
| 100% | Comprometimento formal com o cliente ou prazo inegociável |

**Effort — custo operacional em pessoas × horas equivalentes**
| Score | Critério |
|-------|----------|
| 0.5 | Menos de 1h — tarefa rápida e clara |
| 1 | 1–4h — tarefa de um dia |
| 2 | 4–16h — iniciativa de 2–4 dias |
| 4 | 16–40h — iniciativa de semana |
| 8 | Mais de 40h — iniciativa multi-semana |

### Exemplo de cálculo
```
Item: Validar tracking antes de apresentação ao cliente
Reach: 8 (afeta múltiplos workflows + conta)
Impact: 3 (previne erro crítico com cliente)
Confidence: 90% (cliente pediu explicitamente)
Effort: 1 (4h de trabalho)

RICE = (8 × 3 × 0.9) / 1 = 21.6 → Must
```

### Regra de interpretação
- **Score > 15:** Must ou Should alto — entra no ciclo
- **Score 8–15:** Should ou Could — entra se houver capacidade
- **Score < 8:** Could ou Won't Now — agenda para o futuro ou elimina
- **Confidence < 40%:** sempre Could ou Won't Now — hipótese precisa de validação antes

---

## MoSCoW — Governança do Ciclo

### Definições no contexto Dante

**Must (máx. 3 por ciclo)**
- Sem isso o ciclo falha ou o projeto sofre consequência real
- Critério de done concreto e verificável
- Dono definido, prazo dentro do ciclo, sem dependência não resolvida
- Se 4+ candidatos a Must: negocie com o operador qual cede para Should

**Should**
- Alta importância, mas o ciclo não falha sem isso
- Entra no ciclo se capacidade permitir
- Candidato natural a Must do próximo ciclo

**Could**
- Desejável, mas pode esperar
- Entra apenas se todos os Must e Should forem concluídos antes

**Won't Now**
- Fora do ciclo com justificativa explícita
- Não eliminar da memória — revisar no próximo ciclo ou quando contexto mudar
- Registre o motivo: "muito cedo", "depende de X", "baixo impacto atual"

### Regras de governança do ciclo
1. Máximo 3 Must por ciclo — sem exceção
2. Se o operador pressionar por 4+ Must: apresente os RICE scores e peça que ele escolha os 3
3. Item bloqueado por dependência não resolvida nunca é Must — vai para Should
4. Item de Confidence < 40% nunca é Must — vai para Could no máximo
5. Projeto imaturo: os 3 Must devem pertencer a categorias de fundação (tracking, estrutura, clareza, documentação)
6. Projeto maduro: os Must podem incluir otimização, escala, abertura de novo canal

---

## Avaliação de Maturidade do Projeto

### Como identificar maturidade

**Projeto IMATURO — prioriza fundação**
Sinais:
- Tracking não está validado ou não existe
- Campanhas rodando sem nomenclatura correta ou sem dados
- ICP não definido formalmente
- Não há histórico de testes
- Operador ainda está descobrindo o que funciona
- Sem baseline de métricas estabelecido

Regra: Must = fundação. Nenhum item de sofisticação entra como Must.

**Projeto MADURO — pode priorizar otimização**
Sinais:
- Tracking validado, dados fluindo corretamente
- Histórico de testes com pelo menos 2–3 conclusões
- ICP e icp_product_map definidos e em uso
- Campanhas rodando com nomenclatura correta
- Baseline de métricas estabelecido
- Operador sabe o que funciona e quer escalar

Regra: Must pode incluir otimização, escala, abertura de canal, sofisticação de testes.

### Sinalização de desalinhamento
Se os Must propostos não condizem com a maturidade do projeto:
> "⚠ ALERTA DE MATURIDADE: Este ciclo está priorizando [otimização/escala] num projeto que ainda [não tem tracking validado / não tem histórico de testes / não tem ICP definido]. Recomendo substituir estes Must por [ação de fundação específica] antes de avançar."

---

## Hierarquia de Confiança das Fontes

Respeite esta hierarquia ao priorizar — fonte de maior confiança tem peso maior:

1. **Primária — Sinais do cliente** (check-ins, reuniões, feedback explícito)
   - Define clima da conta e leitura de urgência
   - Confidence mínima: 70–100%
   
2. **Secundária — Eventos operacionais** (erros, entregas, resultados, alertas de performance)
   - Ajusta prioridade com base em fatos
   - Confidence: 50–90% dependendo da solidez do dado
   
3. **Terciária — Percepção interna e hipóteses do time**
   - Entra como hipótese, nunca como verdade estabelecida
   - Confidence máxima: 40–50%

Nunca inverta essa hierarquia. Percepção interna não supera sinal explícito do cliente.
