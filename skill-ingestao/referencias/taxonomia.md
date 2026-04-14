# Guia de Taxonomia e Nomenclatura: Agência Growth IA

Este documento estabelece o padrão universal e obrigatório de nomenclatura para todos os ativos, testes e campanhas. A adesão estrita a este padrão garante que as Skills de IA (Arquiteto de Testes, Guardião do Log, Data Miner e Media Buyer) operem com precisão matemática e sem alucinações.

---

## 1\. Princípios de Design do Sistema

1. **Machine-Readable:** Uso de underscores `_` como separadores universais para facilitar a extração de dados.  
2. **Hierarquia Lógica:** Separação estrita de variáveis entre Campanha (Estratégia), Conjunto (Público) e Anúncio (Criativo).  
3. **Escalabilidade:** Uso de IDs sequenciais com separadores de hífen para evitar sobreposição de categorias.  
4. **CamelCase:** Textos personalizados devem ser escritos sem espaços, com a primeira letra de cada palavra em maiúscula (Ex: `BootcampVendas`).

---

## 2\. Nomenclatura de Campanha (Nível Estratégico)

A campanha define o **"O quê"**, **"Onde"** e **"Para quê"**.

**Sintaxe:** `[STATUS]_[CANAL]_[OBJETIVO]_[CATEGORIA/PRODUTO]_[TIPO-DESTINO]`

### Componentes:

- **STATUS:** `HALL` (Campanhas validadas), `TEST` (Testes), `EVER` (Evergreen/Perpétuo).  
- **CANAL:** `META`, `GOOG`, `LINK`, `TIKT`.  
- **OBJETIVO:** `LEAD`, `SALES`, `TRFC` (Tráfego), `AWAR` (Reconhecimento), `MSNG` (Mensagem).  
- **CATEGORIA:** Nome do produto ou segmento (Ex: `BootcampVendas`).  
- **TIPO-DESTINO:** `FormNativo`, `WhatsApp`, `LandingPage`, `Site`, `Ecom`, `App`.

**Exemplo:** `TEST_META_LEAD_BootcampVendas_FormNativo`

---

## 3\. Nomenclatura de Conjunto de Anúncios (Nível Público)

O conjunto define o **"Para Quem"** e isola a variável de **"Formato"**.

**Sintaxe:** `AUD-[HIERARQUIA]-[ID]_[FUNIL]_[TIPO-ALVO]_[NOME-PUBLICO]_[GEO]_[DEMO]_[FORMATO]`

### A Regra da Cascata (Waterfall 0-9):

O primeiro dígito do ID (`AUD-X-...`) determina a hierarquia de exclusão obrigatória para evitar sobreposição de leilão:

- **0-1 (CLIENTES):** Ativos (0) e Inativos (1).  
- **2-3 (LEADS):** SQLs (2) e MQLs (3).  
- **4 (QUENTE):** Visitantes de Site/LP.  
- **5-6 (MORNO):** Assistiram Ads (5) ou Engajamento em Redes (6).  
- **7-9 (FRIO):** Semelhantes/LAL (7), Perfil Detalhado/Interesses (8), Aberto/Broad (9).

### Componentes:

- **FUNIL:** `COLD` (Frio), `WARM` (Morno), `HOT` (Quente).  
- **TIPO-ALVO:** `LAL` (Lookalike), `INT` (Interesses), `RTG` (Retargeting), `CRM` (Listas), `BROAD` (Aberto).  
- **FORMATO:** `STA` (Apenas Estáticos), `VID` (Apenas Vídeos), `MIX` (Misto).

**Exemplo:** `AUD-7-001_COLD_LAL_Clientes_BR_25-60-HM_VID`

---

## 4\. Nomenclatura de Anúncio (Nível Criativo/KV)

O anúncio é o **"Código de Barras"** do ativo visual. O detalhamento completo (headline, copy) reside no `GROWTH_LOG.md`.

**Sintaxe:** `AD-[ID]_[FORMATO]_[CONSCIENCIA]_[GANCHO]_[AVATAR]_[VARIAÇÃO]`

### Componentes:

- **FORMATO:** `STA` (Estático), `VID` (Vídeo), `CAR` (Carrossel), `MOT` (Motion).  
- **CONSCIENCIA:**  
  - `UNC` (Inconsciente)  
  - `PRB` (Consciente do Problema)  
  - `SOL` (Consciente da Solução)  
  - `PRO` (Avaliando Fornecedor/Produto)  
  - `MIX` (Múltiplos Níveis)  
- **GANCHO (Hook):** `Loss` (Aversão à perda), `Proof` (Prova Social), `Story` (História), `Error` (Alerta de Erro), `Fact` (Especificidade).  
- **AVATAR:** Nome do porta-voz ou persona (Ex: `CamilaFarani`).  
- **VARIAÇÃO:** Resumo curto da headline ou tema visual (Ex: `PerdendoDinheiro`).

**Exemplo:** `AD-015_STA_UNCPRB_Loss_CamilaFarani_PerdendoDinheiro`

---

## 5\. Dicionário de Tags e Abreviações

| Categoria | Tag | Significado |
| :---- | :---- | :---- |
| Formato | `STA` | Static / Banner / Imagem Única |
| Formato | `VID` | Vídeo / Reels / Pessoas Reais / IA Avatar |
| Consciência | `UNCPRB` | Inconsciente e Consciente do Problema |
| Gancho | `Contr` | Contrariante / Quebra de Padrão |
| CTA | `CONS` | Consultivo |
| CTA | `DIR` | Direto / Oferta |

---

## 6\. Governança do Sistema

1. **O Guardião do Log:** Esta Skill deve rejeitar qualquer registro no `GROWTH_LOG.md` que não siga este padrão.  
2. **O Media Buyer:** Deve gerar automaticamente as exclusões de público baseando-se no dígito de Hierarquia (Ex: AUD-9 exclui AUD 0 a 8).  
3. **O Data Miner:** Deve usar o separador `_` para decompor os nomes de campanha em dimensões de análise.

---

*Documento Versão 1.0 \- Estrutura consolidada para Agência Growth IA.*  
