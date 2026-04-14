# Padrões de Componente — Dante Dashboard

## Padrão Base: Hook + Componente Separados

Nunca misturar lógica de fetch com JSX no mesmo arquivo. Sempre:
- `useSectionData.ts` — TanStack Query, transformação com useMemo, tipos
- `SectionComponent.tsx` — JSX puro, recebe props, não sabe de onde vêm

```tsx
// hooks/useFunnelData.ts
import { useQuery } from '@tanstack/react-query';
import { useMemo } from 'react';
import { useProjectContext } from '@/contexts/ProjectContext';

export function useFunnelData(filters: FunnelFilters) {
  const { projectId } = useProjectContext();

  const { data, isLoading, isError, refetch } = useQuery({
    queryKey: ['funnel', projectId, filters],
    queryFn: () => fetchFunnel(projectId, filters),
    staleTime: 5 * 60 * 1000,
    enabled: !!projectId,   // não roda sem projectId
  });

  // Transformação fora do JSX — memoizada
  const chartData = useMemo(() => {
    if (!data) return [];
    return data.stages.map(stage => ({
      name: stage.label,
      value: stage.count,
      rate: stage.conversion_rate,
    }));
  }, [data]);

  return { chartData, isLoading, isError, refetch };
}
```

---

## Padrão de Estados Obrigatórios

```tsx
// components/dashboard/FunnelSection/index.tsx
import { useFunnelData } from '@/hooks/useFunnelData';
import { FunnelChart } from './FunnelChart';
import { SectionSkeleton } from '@/components/ui/SectionSkeleton';
import { SectionError } from '@/components/ui/SectionError';
import { EmptyState } from '@/components/ui/EmptyState';

export function FunnelSection({ filters }: { filters: FunnelFilters }) {
  const { chartData, isLoading, isError, refetch } = useFunnelData(filters);

  if (isLoading) return <SectionSkeleton height={320} />;
  if (isError) return <SectionError onRetry={refetch} />;
  if (chartData.length === 0) return (
    <EmptyState message="Nenhum dado de funil para o período selecionado" />
  );

  return <FunnelChart data={chartData} />;
}
```

**Regra:** o `isLoading` vem do TanStack Query, não de `useState(true)`. Nunca gerenciar loading manualmente.

---

## Padrão Recharts com Tokens Dante

```tsx
// components/dashboard/EvolutionChart/EvolutionChart.tsx
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid,
  Tooltip, ResponsiveContainer, Legend
} from 'recharts';

const DANTE_COLORS = {
  primary: 'var(--color-primary-500)',     // Copper #A0522D
  secondary: 'var(--color-neutral-600)',   // Cinza médio
  grid: 'var(--color-neutral-200)',        // Bordas
};

export function EvolutionChart({ data }: { data: EvolutionPoint[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={data} margin={{ top: 8, right: 16, left: 0, bottom: 0 }}>
        <CartesianGrid strokeDasharray="3 3" stroke={DANTE_COLORS.grid} />
        <XAxis
          dataKey="period"
          tick={{ fontFamily: 'DM Sans', fontSize: 12 }}
        />
        <YAxis
          tick={{ fontFamily: 'DM Sans', fontSize: 12 }}
          tickFormatter={(v) => `R$${(v / 1000).toFixed(0)}k`}
        />
        <Tooltip
          contentStyle={{
            fontFamily: 'DM Sans',
            borderRadius: '6px',
            border: '1px solid var(--color-neutral-200)',
            backgroundColor: 'var(--color-neutral-50)',
          }}
        />
        <Line
          type="monotone"
          dataKey="revenue"
          stroke={DANTE_COLORS.primary}
          strokeWidth={2}
          dot={false}             // sem pontos em série densa
          activeDot={{ r: 4 }}
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

**Regras:**
- `ResponsiveContainer` sempre — nunca width/height fixo em px
- `dot={false}` para séries com > 30 pontos — melhora performance e legibilidade
- Tooltip com tokens, não valores hardcoded
- `tickFormatter` para formatar valores no eixo Y sem poluir os dados

---

## Padrão de Cohort (Safra) — Tabela com Heatmap CSS

```tsx
// components/dashboard/CohortAnalysis/CohortTable.tsx

function getCohortCellClass(rate: number): string {
  if (rate >= 0.3) return 'bg-primary-500 text-white';
  if (rate >= 0.2) return 'bg-primary-300 text-neutral-950';
  if (rate >= 0.1) return 'bg-primary-100 text-neutral-950';
  return 'bg-neutral-100 text-neutral-600';
}

export function CohortTable({ rows }: { rows: CohortRow[] }) {
  const months = ['M+1', 'M+2', 'M+3', 'M+4', 'M+5', 'M+6'];

  return (
    <div className="overflow-x-auto">
      <table className="w-full text-sm font-dm-sans border-collapse">
        <thead>
          <tr className="bg-neutral-100">
            <th className="px-4 py-2 text-left text-neutral-600 font-medium">Safra</th>
            <th className="px-4 py-2 text-right text-neutral-600 font-medium">Leads</th>
            {months.map(m => (
              <th key={m} className="px-4 py-2 text-center text-neutral-600 font-medium">{m}</th>
            ))}
          </tr>
        </thead>
        <tbody>
          {rows.map(row => (
            <tr key={row.period} className="border-t border-neutral-200">
              <td className="px-4 py-2 font-medium">{row.period}</td>
              <td className="px-4 py-2 text-right">{row.total_leads.toLocaleString('pt-BR')}</td>
              {[row.rate_m1, row.rate_m2, row.rate_m3, row.rate_m4, row.rate_m5, row.rate_m6].map((rate, i) => (
                <td
                  key={i}
                  className={`px-4 py-2 text-center rounded ${getCohortCellClass(rate ?? 0)}`}
                >
                  {rate != null ? `${(rate * 100).toFixed(1)}%` : '—'}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## Padrão de KPI Card

```tsx
// components/ui/KPICard.tsx
interface KPICardProps {
  label: string;
  value: string;
  trend?: { direction: 'up' | 'down' | 'neutral'; label: string };
  isHighlight?: boolean;  // usa cor Copper para valor
}

export function KPICard({ label, value, trend, isHighlight = false }: KPICardProps) {
  return (
    <div className="bg-neutral-50 border border-neutral-200 rounded-md p-4 shadow-[0_2px_8px_rgba(17,17,17,0.06)]">
      <p className="text-xs font-medium text-neutral-600 uppercase tracking-wide font-dm-sans">
        {label}
      </p>
      <p className={`text-2xl font-bold mt-1 font-fraunces ${isHighlight ? 'text-primary-500' : 'text-neutral-950'}`}>
        {value}
      </p>
      {trend && (
        <p className={`text-xs mt-1 font-dm-sans ${
          trend.direction === 'up' ? 'text-success-700' :
          trend.direction === 'down' ? 'text-error-700' :
          'text-neutral-600'
        }`}>
          {trend.label}
        </p>
      )}
    </div>
  );
}
```

---

## Padrão de Skeleton (evitar Layout Shift)

```tsx
// components/ui/SectionSkeleton.tsx
export function SectionSkeleton({ height = 200 }: { height?: number }) {
  return (
    <div
      className="animate-pulse bg-neutral-100 rounded-md"
      style={{ height: `${height}px` }}   // altura fixa para evitar CLS
      aria-label="Carregando..."
      role="status"
    />
  );
}
```

**Regra:** altura do skeleton deve ser igual à altura do componente real. Medir o componente e fixar.

---

## Padrão de Estado Vazio

```tsx
// components/ui/EmptyState.tsx
export function EmptyState({ message }: { message: string }) {
  return (
    <div className="flex flex-col items-center justify-center py-12 text-neutral-600">
      <p className="font-dm-sans text-sm">{message}</p>
    </div>
  );
}
```

Sem ícone complexo, sem ilustração — o Dante é direto.

---

## Padrão de Lazy Loading por Seção

```tsx
// app/Dashboard.tsx
import { lazy, Suspense } from 'react';
import { SectionSkeleton } from '@/components/ui/SectionSkeleton';

// Seções pesadas carregam só quando visíveis
const CohortAnalysis = lazy(() => import('@/components/dashboard/CohortAnalysis'));
const ProjectionChart = lazy(() => import('@/components/dashboard/ProjectionChart'));

export function Dashboard() {
  return (
    <main>
      {/* Seções acima do fold — carregam imediatamente */}
      <BusinessResults />
      <FunnelSection />

      {/* Seções pesadas — lazy */}
      <Suspense fallback={<SectionSkeleton height={400} />}>
        <CohortAnalysis />
      </Suspense>
      <Suspense fallback={<SectionSkeleton height={300} />}>
        <ProjectionChart />
      </Suspense>
    </main>
  );
}
```

---

## React.memo para Charts em Grid

```tsx
// Sem memo — todo filtro re-renderiza todos os 6 charts
function DashboardGrid({ filters }) {
  return (
    <div className="grid grid-cols-3 gap-4">
      <FunnelChart data={funnelData} />
      <CostChart data={costData} />
      {/* ... */}
    </div>
  );
}

// Com memo — só re-renderiza o chart cujos dados mudaram
const FunnelChart = React.memo(function FunnelChart({ data }) {
  return <RechartsWrapper data={data} />;
});
```

Aplicar `React.memo` em todo componente de chart que recebe dados como prop e está dentro de um grid.
