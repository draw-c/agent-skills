# Performance

Measure the boundary and the bundle before adding memo. RSC and the React Compiler removed most of the old “wrap everything in `useCallback`” playbook.

## Budgets

Think per **route client graph**, not per repo.

- `"use client"` at a layout taxes every child route.
- A barrel `components/index.ts` that re-exports client and server widgets pulls client code into anyone who import one export.
- Icon packs: import `{ Zap } from 'lucide-react'` (tree-shaken). Do not import a barrel of 300 icons in a server file that then gets imported by a client leaf.

## First-paint data

| Pattern | Cost |
|---------|------|
| Async Server Component `await` | Server time, zero client JS for the data path |
| `useEffect` + `fetch` | Extra RT, loading flash, SEO/empty HTML |
| RSC `fetch` to your own `/api/*` | Extra hop, loses `cookies()` forwarding unless you duplicate headers |
| Sequential `await` of independent reads | Waterfall on TTFB |

Fix order: move fetch to the server → `Promise.all` → Suspense the slow part → then look at IO.

## Streaming

- `loading.tsx` = one Suspense around the whole segment. Use it, then add **inner** `<Suspense>` for slow widgets so the shell and fast data are not blocked.
- Skeleton must match layout (avoid CLS). No emoji. Same density as the real UI.
- Cache Components: request-time APIs behind a boundary or the instant-navigation check fails.

## Compiler vs manual memo

With `reactCompiler: true` (Next 16):

- Do not add `useMemo` / `useCallback` / `React.memo` unless a profiler shows a remaining hot path the compiler missed.
- Stale-closure bugs from hand-rolled memo are worse than an extra render on a leaf button.

Without the compiler, memoize only stable callbacks passed into **memoized expensive children**, not every handler.

## Images, fonts, scripts

- `next/font` — no layout shift from Google CSS.
- `next/image` — size (width/height or `fill` + parent `relative`), `sizes` for responsive, `priority` only on LCP.
- `next/script` — `afterInteractive` / `lazyOnload` for third parties. Never a blocking `<script>` in `layout.tsx` for analytics.

Team logos: `Image` with transparent PNG/SVG, **no** square tile wrappers (workspace rule).

## `next/dynamic`

Use for heavy **client** widgets (charts, PDF, editors):

```tsx
const OddsChart = dynamic(() => import('./odds-chart'), {
  loading: () => <ChartSkeleton />,
  ssr: true, // default; keep SSR unless the module crashes without window
});
```

`ssr: false` only when the import graph touches `window` at load. Not a substitute for fixing the module. Illegal / broken as a way to disable SSR for a Server Component.

## Bundle audit

1. `@next/bundle-analyzer` or Turbopack generate a client graph for the route.
2. Search `"use client"` — each hit is a root of a client subgraph.
3. Check unexpected packages in that subgraph (ORM, Node `fs`, markdown, moment, lodash barrel).
4. Split: move data libs behind `server-only`; split “table” (server) from “row actions” (client).

This repo: prefer `npx tsc --noEmit` over `next build` while `next dev` is running.

## LCP / CLS / INP (App Router)

- LCP: server-rendered heading or priority image in the first chunk, not a client-only hero.
- CLS: font + image dimensions + skeletons that reserve height.
- INP: keep client leaves small; `useTransition` for non-urgent actions; do not hydrate a dashboard-worth of listeners for one button.

## This repo’s SPA shell

A single client page with tab state:

- Fast local UX, **no** per-tab RSC, **no** granular `loading.tsx`, **full** client graph on first load.
- Performance work is then: split routes, or dynamic-import tab panels, or keep the shell client but fetch tab data via RSC children if you reintroduce server page composition.

Say which strategy you are on. Do not mix “we are App Router” rhetoric with an SPA bundle.
