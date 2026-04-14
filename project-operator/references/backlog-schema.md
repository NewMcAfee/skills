# Schema de Backlog — Project Operator

## Campos Obrigatórios de Todo Item

Nenhum item entra no backlog ativo sem estes 6 campos. Item incompleto vai para rascunho.

| Campo | Descrição | Exemplos |
|-------|-----------|---------|
| **Objetivo** | O que este item realiza — em 1 frase | "Validar tracking antes da apresentação ao cliente" |
| **Prioridade** | MoSCoW + RICE score | Must (21.6) / Should (9.2) / Could (4.1) |
| **Dependências** | O que precisa estar pronto antes | "Briefing Criativo test_id:xxx" / "nenhuma" |
| **Dono** | Quem executa — pessoa ou agente nomeado | "Escriba" / "João (mídia)" / "operador" |
| **Prazo** | Data limite dentro do ciclo | "2026-04-15" |
| **Critério de done** | O que significa estar pronto — verificável | "LP publicada e sem erro de tracking" |

## Campos Completos do Backlog

```
ID:                     B001 (sequencial por projeto)
Nome:                   [nome curto e descritivo]
Tipo:                   iniciativa | entregável | tarefa | briefing
Origem:                 [Tipo A–E: estratégico | conta | operacional | analítico | revisão]
Objetivo:               [o que realiza — 1 frase]
Categoria:              tracking | criativo | copy | mídia | análise | governança | infraestrutura | cliente
Hipótese relacionada:   [test_id se houver] | nenhuma
Impacto esperado:       [o que muda quando isso estiver pronto]
RICE Score:             [número calculado]
  Reach:                [score 1–10]
  Impact:               [0.25 / 0.5 / 1 / 2 / 3]
  Confidence:           [% como decimal: 0.3 | 0.5 | 0.7 | 0.9 | 1.0]
  Effort:               [0.5 | 1 | 2 | 4 | 8]
MoSCoW:                 Must | Should | Could | Won't Now
Dependências:           [IDs de itens ou condições externas]
Dono:                   [pessoa ou agente]
Prazo:                  [YYYY-MM-DD]
Status:                 backlog | em andamento | bloqueado | concluído | cancelado
Critério de done:       [verificável, sem ambiguidade]
Precisa de QC?:         sim | não — [se sim: qual tipo de revisão]
Observações:            [contexto adicional relevante]
```

## Tipos de Objeto — Diferenças Práticas

### Iniciativa
- Nível mais alto de abstração
- Não é executável diretamente — precisa ser quebrada em entregáveis
- Tem objetivo estratégico claro
- Exemplo: "Estruturar o tracking do projeto"

### Entregável
- Saída concreta e verificável
- Filho de uma iniciativa
- Tem critério de done específico e objetivo
- Exemplo: "Measurement plan aprovado" / "GTM configurado e validado"

### Tarefa
- Unidade operacional atômica
- Executável por uma pessoa ou agente sem perguntas adicionais
- Filho de um entregável
- Exemplo: "Revisar nomenclatura dos eventos no GTM" / "Subir planilha de combinações no Guardião"

### Briefing
- Instrução estruturada para execução por pessoa ou agente
- Filho de uma tarefa ou entregável
- Deve ser autossuficiente — quem recebe não precisa perguntar mais nada
- Ver `references/briefing-templates.md` para os formatos

## Categorias de Item — Definições

| Categoria | Quando usar |
|-----------|-------------|
| tracking | Eventos, GTM, Measurement Plan, Data Miner, validações |
| criativo | Assets, anúncios, vídeos, peças de campanha |
| copy | Textos, landing pages, e-mails, copies de anúncio |
| mídia | Campanhas, conjuntos, segmentações, budget |
| análise | Diagnósticos, relatórios, leituras de performance |
| governança | Backlog, briefings, decisões, memória do projeto |
| infraestrutura | Banco, Validator, N8N, dashboard, integrações |
| cliente | Entregas para o cliente, apresentações, check-ins |

## Origens de Item — Tipos A–E

| Tipo | Origem | Hierarquia de confiança |
|------|--------|------------------------|
| A | Estratégia e planejamento (GTM Plan, briefing, PUV) | Alta — fundação do projeto |
| B | Conta (Client Success Strategist: riscos, próximos passos, promessas) | Mais alta — sinal do cliente |
| C | Operacional (erros, blockers, atrasos, devoluções do QC) | Média — fato operacional |
| D | Analítico (Growth Lead: alertas, recomendações, Arquiteto de Testes: novos testes) | Média — dado de performance |
| E | Revisão de fase (replanejamento, mudança de step) | Alta — decisão estratégica |

## Estados do Backlog

| Status | Definição |
|--------|-----------|
| backlog | Aguardando ciclo — não está no ciclo atual |
| em andamento | No ciclo atual, dono trabalhando |
| bloqueado | Dependência não resolvida — registre qual |
| concluído | Critério de done atingido — marque a data |
| cancelado | Fora do escopo — registre o motivo |

## Exemplo de Backlog Estruturado

```
ID: B001
Nome: Validar tracking de leads antes da apresentação
Tipo: tarefa
Origem: Tipo B (Client Success Strategist: cliente quer ver dados na reunião de quinta)
Objetivo: Garantir que os eventos de lead estão disparando corretamente antes de mostrar dados ao cliente
Categoria: tracking
RICE: 18.9 (Reach:8 × Impact:3 × Confidence:0.9 / Effort:1.28)
MoSCoW: Must
Dependências: GTM configurado (B003 — concluído)
Dono: operador
Prazo: 2026-04-14
Status: em andamento
Critério de done: 100% dos eventos de lead disparando sem erro no GTM Preview + dados chegando no Supabase
Precisa de QC?: sim — revisão operacional antes de apresentar ao cliente
```
