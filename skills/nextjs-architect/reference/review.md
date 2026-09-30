# Architecture review

Use this when the user asks to review, audit, or “why is this slow.” Work top-down. Cite file paths. Use the SKILL.md output format (Assessment / Recommendation / Trade-offs / Verification).

Do not propose Pages Router or `getServerSideProps`.

## 1. Version and shape

- [ ] Next major from `package.json` (15 vs 16). Cache APIs must match (`reference/next-16.md`).
- [ ] App Router only, or a Pages island? Flag any `pages/`.
- [ ] Real routes (`app/**/page.tsx`) vs SPA-on-one-page (this repo: `/` + client tabs). If SPA, say the RSC/cache story does not apply until split.

## 2. Rendering boundaries

- [ ] `"use client"` on `page.tsx` or root `layout.tsx`? Push to leaves.
- [ ] Client files that import DAL / ORM / env secrets? Add `server-only` and break the import.
- [ ] Barrels that re-export client + server together? Split or delete.
- [ ] Raw DB rows passed to client components? Map DTOs.
- [ ] `params` / `searchParams` / `cookies()` / `headers()` awaited?
- [ ] Server children passed as `children` into client wrappers (good) vs imported into client files (bad).

## 3. Data and cache

- [ ] First paint fetched on the server, not `useEffect`.
- [ ] Independent reads in `Promise.all` (or equivalent).
- [ ] React `cache()` on request-scoped DAL used in multiple components.
- [ ] Tag helpers exist; readers and mutations share the same tags.
- [ ] Mutations call `updateTag` / `revalidateTag` (with profile on 16) — not only `revalidatePath('/')`.
- [ ] Route Handlers not used as an inner hop from RSC.
- [ ] If `cacheComponents: true`: no segment `dynamic`/`revalidate`/`fetchCache`; request-time reads behind Suspense.

## 4. Streaming and failure

- [ ] `loading.tsx` for segments that await.
- [ ] Inner `<Suspense>` around slow/personalized blocks.
- [ ] `error.tsx` on fetching segments; `global-error.tsx` at root.
- [ ] `not-found()` + `not-found.tsx` for missing entities.
- [ ] Skeletons match layout; no emoji.

## 5. Bundle and UX platform

- [ ] `next/font`, `next/image`, `next/script` used.
- [ ] Heavy widgets dynamic-imported; `{ ssr: false }` only where required, never to “disable RSC.”
- [ ] `typedRoutes` / `reactCompiler` considered for Next 16.
- [ ] No `next build` while `next dev` is up; verify with `tsc --noEmit` and analyzer when the question is size.

## 6. Security at the boundary

- [ ] Server Actions authenticated and authorized in DAL.
- [ ] Client-safe DTOs (no secrets, hashes, internal flags).
- [ ] `server-only` on data layer.

## How to write the findings

Rank by **architectural risk**, not nitpicks:

1. Boundary leaks (client importing server IO)
2. Wrong cache contract (stale money/odds, or no cache on hot public data)
3. Waterfalls / blocked shells
4. Bundle (root `"use client"`, barrels)
5. Missing error/loading
6. Compiler/memo noise

Each finding: file path, what happens in production, the fix, cost of the fix.

## Verification (pick what matches the change)

- Types: `npx tsc --noEmit` (from `web/`).
- Boundary: build-time `'server-only'` error gone; `"use client"` count reduced on the route.
- Cache: mutate once, confirm the tagged UI updates (immediate vs SWR).
- Bundle: analyzer — expected packages absent from client graph.
- UX: 375px, keyboard, reduced motion (pair with `tailwind-designer` if UI).
