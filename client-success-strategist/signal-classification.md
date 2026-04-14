# Classificação e Peso de Sinais — Client Success Strategist

## O Problema que a Classificação Resolve

Times de growth são tendenciosos para otimismo. Quando o cliente não reclama, o time interpreta como satisfação. Quando o cliente faz uma pergunta difícil, o time interpreta como "ele está engajado". Essa contaminação narrativa é o principal inimigo da governança de conta.

A classificação hierárquica força separação antes da interpretação.

---

## Fontes e Pesos

### PRIMÁRIA (peso 1.0) — Sinais do Cliente

O que conta como sinal primário:
- **Afirmação direta:** o cliente disse X sobre resultado, qualidade, relacionamento ou expectativa
- **Pergunta estratégica:** o cliente questionou algo que revela preocupação ou interesse (ex: "quanto tempo para vermos resultado?", "vocês já fizeram isso para outros clientes?")
- **Comportamento observável:** cliente chegou 20 min atrasado na call, cancelou reunião de última hora, não abriu os relatórios enviados, trouxe novo stakeholder sem avisar
- **Silêncio qualificado:** cliente não respondeu a pergunta direta sobre satisfação — isso é dado, não ausência de dado
- **Tom e energia:** mudança de padrão de comunicação em relação ao histórico (ficou mais formal, menos entusiasmado, mais objetivo)

O que NÃO é sinal primário:
- Interpretação do time sobre o comportamento do cliente ("ele parecia feliz")
- Inferência baseada em métricas sem validação do cliente
- "Não reclamou" — ausência de reclamação não é satisfação confirmada

**Regra de ouro primária:** se o cliente não disse ou demonstrou diretamente, não é primário.

---

### SECUNDÁRIA (peso 0.6) — Eventos Operacionais

O que conta como evento secundário:
- **Resultado de campanha:** ROAS, CPL, leads gerados, taxa de conversão — vs. metas acordadas
- **Status de entrega:** entregue no prazo / com atraso / refeito / não entregue
- **Incidentes:** erro técnico, falha de processo, dado incorreto enviado ao cliente
- **Mudanças de contexto:** budget cortado, escopo alterado, prazo movido — independente da causa
- **Histórico de interação:** frequência de check-ins, abertura de e-mails, participação em reuniões

Como usar eventos secundários:
- Ajuste o score das dimensões afetadas
- **Não** derive leitura de clima sem sinal primário correspondente
- Use para identificar o que precisa ser validado com o cliente — não para substituir essa validação

---

### TERCIÁRIA (peso 0.3) — Percepção Interna

O que conta como percepção interna:
- O que o gestor/time acha sobre o estado da conta
- Intuições sobre o humor do cliente
- Avaliações subjetivas de reunião ("achei que ele gostou")
- Hipóteses sobre o que o cliente está pensando

Regras estritas para fonte terciária:
1. **Nunca define clima** — só informa hipóteses
2. **Nunca sobrescreve primário** — se conflitar com sinal do cliente, o primário prevalece sempre
3. **Sempre rotulada explicitamente:** `⚠️ Hipótese — percepção interna — requer validação`
4. **Tem validade temporal:** se uma hipótese terciária não for validada no próximo check-in, ela se torna risco de gap de informação

**Exceção de elevação:** percepção interna pode ser promovida a secundária SE há um evento operacional concreto que a sustente. Ex: "time acha que cliente está insatisfeito" (terciária) + "cliente não abriu nenhum relatório nos últimos 30 dias" (secundária) = hipótese terciária com evidência secundária — mas ainda não é primária.

---

## Protocolo de Classificação

Ao receber qualquer input, passe cada sinal por estas perguntas em sequência:

```
1. O cliente disse ou demonstrou isso diretamente? 
   → SIM: PRIMÁRIA
   → NÃO: vai para 2

2. Isso é um evento mensurável ou fato operacional?
   → SIM: SECUNDÁRIA
   → NÃO: vai para 3

3. Isso é o que o time acha, interpreta ou sente?
   → SIM: TERCIÁRIA
   → NÃO: descarte — não é dado, é ruído
```

---

## Sinais de Risco por Categoria

### Risco de Retenção (alta criticidade)
Sinais primários que indicam risco de não renovação ou saída:
- Menção a "avaliar alternativas" ou "ver outras opções"
- Pergunta sobre cláusulas de saída do contrato
- Solicitação de dados para exportação ou auditoria externa
- Tom de encerramento em comunicação ("obrigado por tudo")
- Prazo de renovação se aproximando + engajamento reduzido (secundário + comportamental)

### Risco de Relacionamento
- Mudança de stakeholder sem apresentação formal do novo contato
- Inclusão de pessoa de nível mais alto sem contexto
- Redução da frequência de check-ins iniciada pelo cliente
- Respostas monossilábicas onde antes havia diálogo
- Cliente parou de compartilhar contexto estratégico

### Risco de Resultado
- Questionamento direto sobre metas ou ROI
- Comparação negativa com período anterior ou com concorrente
- Solicitação de revisão de expectativas ou repriorização
- "Precisamos ver resultado até X" com urgência explícita

### Risco de Operação
- Reclamação sobre processo, comunicação ou execução — mesmo que resolvida
- Pedido de mais transparência, relatórios ou acesso
- Apontamento de erro pelo cliente antes do time identificar

---

## Sinais de Oportunidade

### Expansão de Escopo
- Cliente menciona problema que não está no escopo atual ("precisamos melhorar X também")
- Crescimento do negócio do cliente relatado ("estamos abrindo nova unidade")
- Pergunta sobre capacidade do time para outros projetos

### Upsell / Novos Serviços
- Cliente pede algo que não está no contrato atual
- Pergunta sobre outros serviços ou capacidades do time
- Indicação espontânea para outro setor ou empresa

### Aprofundamento de Relacionamento
- Cliente pede opinião estratégica além do escopo
- Convite para participar de reuniões internas
- Apresentação a stakeholders de nível mais alto

---

## O Que Fazer com Contradições

**Cenário:** sinal primário positivo + percepção interna negativa
→ Score pela fonte primária. Registre a percepção como `⚠️ Hipótese — investigar no próximo check-in`. Não resolva a contradição — ela é a próxima pergunta a fazer ao cliente.

**Cenário:** sinal primário positivo + evento secundário negativo
→ Score balanceado. Leitura de clima segue o primário, mas risco secundário entra como alerta. Próximo passo: validar com cliente se ele percebe o evento operacional.

**Cenário:** ausência de sinal primário recente (> 30 dias sem check-in)
→ Marque como "sem sinal primário suficiente". Não derive clima. Urgência: agendar check-in. O silêncio prolongado é em si um sinal de risco.
