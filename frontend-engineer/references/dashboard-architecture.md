# Dante Dashboard BI — Arquitetura Frontend

## Contexto

O Dashboard BI replica e evolui o GrowthTracking (`growhtracking.lovable.app`) — mesmo operador construiu, mesmas seções, mas agora com dados reais do Supabase via Validator API. Não é um dashboard genérico: é a visão operacional do coordenador V4.

---

## Seções do Dashboard

| Seção | Descrição | Tipo de dado | Chart |
|-------|-----------|--------------|-------|
| Resultado de Negócio | KPIs principais (investimento, leads, oportunidades, receita) | Agregado por período | Cards com trend |
| Funil de Geração de Demanda | Impressões → Cliques → Leads → Oportunidades → Vendas | Calculado on-demand | Funnel chart ou bar |
| Custos por Etapa | CPM, CPC, CPL, CPO, CAC | Calculado on-demand | Bar chart |
| Taxas de Conversão | CTR, Lead Rate, Opp Rate, Close Rate | Calculado | Cards + mini chart |
| Velocidade do Funil | Tempo médio entre etapas | Calculado | Bar horizontal |
| Evolução | Métricas ao longo do tempo | Série temporal | LineChart Recharts |
| Análise de Safra (Cohort) | Cohort de leads por período de entrada vs conversão | Tabela cruzada | Tabela com heatmap CSS |
| Projeção de Resultados | Projeção de receita com base em histórico | Calculado | AreaChart com banda de confiança |

---

## Fluxo de Dados

```
Supabase (PostgreSQL)
  ↓ REST API (Validator FastAPI)
  ↓ Derivações calculadas on-demand (não persistidas)
TanStack Query (cache no cliente)
  ↓ hooks customizados por seção
Componentes React
  ↓ props tipadas
Recharts / Tabelas
```

**Regra crítica:** dados derivados (taxas, custos por etapa, projeções) são calculados no backend e entregues prontos. O frontend não faz matemática de negócio.

---

## Multi-projeto

O operador navega entre projetos sem recarregar a página. Cada projeto tem `project_id` como chave de isolamento.

**Padrão obrigatório:**

```tsx
// Contexto global no topo da app
const ProjectContext = createContext<{ projectId: string; setProjectId: (id: string) => void } | null>(null);

export function useProjectContext() {
  const ctx = useContext(ProjectContext);
  if (!ctx) throw new Error('useProjectContext must be used inside ProjectProvider');
  return ctx;
}

// Query key sempre com projectId
const { data } = useQuery({
  queryKey: ['funnel', projectId, { startDate, endDate, granularity }],
  queryFn: () => fetchFunnel(projectId, filters),
  staleTime: 5 * 60 * 1000,
});
```

Trocar `projectId` invalida automaticamente todas as queries que o usam na key. Não precisa de `queryClient.invalidateQueries` manual.

---

## Estratégia de Fetch por Seção

| Seção | staleTime | Refetch strategy |
|-------|-----------|------------------|
| Resultado de Negócio | 5min | On project change |
| Funil | 5min | On filter change |
| Evolução | 10min | On date range change |
| Safra/Cohort | 15min | On demand (botão refresh) |
| Projeção | 15min | On demand |
| Projetos (lista nav) | 10min | On mount |

---

## Estrutura de Arquivos Sugerida

```
src/
├── components/
│   ├── dashboard/
│   │   ├── BusinessResults/
│   │   │   ├── index.tsx          # Componente raiz da seção
│   │   │   ├── KPICard.tsx        # Card individual
│   │   │   └── useBizResults.ts   # Hook + query
│   │   ├── FunnelSection/
│   │   ├── CostSection/
│   │   ├── ConversionRates/
│   │   ├── FunnelVelocity/
│   │   ├── EvolutionChart/
│   │   ├── CohortAnalysis/        # Tabela com heatmap
│   │   └── ProjectionChart/
│   └── ui/                        # Componentes do design system
│       ├── Card.tsx
│       ├── KPIBadge.tsx
│       ├── SectionHeader.tsx
│       └── EmptyState.tsx
├── contexts/
│   └── ProjectContext.tsx
├── hooks/
│   └── useProjectContext.ts
└── lib/
    └── queryClient.ts             # Configuração global do TanStack Query
```

---

## Configuração Global do TanStack Query

```tsx
// lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // 5min padrão
      gcTime: 10 * 60 * 1000,        // 10min no cache antes de GC
      retry: 1,                       // 1 retry em erro de rede
      refetchOnWindowFocus: false,    // dashboard analítico — sem refetch ao focar janela
    },
  },
});
```

---

## Cohort / Análise de Safra

A seção de Safra é uma **matriz de tabela com coloração condicional** — não um gráfico. Recharts não é a ferramenta aqui.

Estrutura esperada do dado:
```ts
type CohortRow = {
  period: string;         // "Jan/24", "Fev/24" — mês de entrada do lead
  total_leads: number;
  converted_m1: number;   // convertidos no mês 1 após entrada
  converted_m2: number;
  converted_m3: number;
  // ...
  rate_m1: number;        // taxa de conversão mês 1 (vem do backend)
  rate_m2: number;
};
```

Renderização: tabela HTML com `background-color` proporcional à taxa usando escala Copper (50→500). Zero CSS inline — usa variáveis CSS dos tokens.

---

## Estados Padrão por Seção

Todo wrapper de seção deve seguir este padrão:

```tsx
function SectionWrapper({ children, isLoading, isError, isEmpty, onRetry }) {
  if (isLoading) return <SectionSkeleton />;
  if (isError) return <SectionError onRetry={onRetry} />;
  if (isEmpty) return <EmptyState message="Nenhum dado para o período selecionado" />;
  return children;
}
```

`SectionSkeleton` usa dimensões fixas para evitar layout shift (CLS). Nunca usar `height: auto` em skeleton.
