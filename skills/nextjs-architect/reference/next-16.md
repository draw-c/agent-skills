# Next.js 16

This workspace's web app is Next 16 (`web/package.json`). Prefer these APIs over v14/v15 defaults. Detect the major from `package.json` before copying patterns from older blogs.

## What changed (architecturally)

| Before (15 / early 16) | After (16 with Cache Components) |
|------------------------|----------------------------------|
| Implicit `fetch` cache + segment `dynamic` / `revalidate` / `fetchCache` | Opt **in** with `'use cache'` + `cacheLife` / `cacheTag` |
| `experimental.dynamicIO` / `experimental.useCache` / experimental PPR | `cacheComponents: true` (PPR is this model) |
| `revalidateTag('posts')` | `revalidateTag('posts', 'max')` — profile required |
| Manual memo as default | `reactCompiler: true` |
| Sync `params` / `searchParams` | Always `Promise<…>` — `await` them |

Do not enable `cacheComponents` and keep `export const dynamic = 'force-dynamic'` (or `revalidate` / `fetchCache`) on the same segment. Next will error. Isolate request-time work instead.

## Enable Cache Components

```ts
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  cacheComponents: true,
  reactCompiler: true,
  typedRoutes: true,
};

export default nextConfig;
```

With the flag on, Next prerenders the static/cached shell and validates in dev that navigation can stay instant. Anything that would block that shell (cookies, uncached IO, `searchParams`) must sit behind `<Suspense>` or inside a `'use cache'` function with an explicit life.

## `'use cache'` instead of `unstable_cache`

`SKILL.md` still shows `unstable_cache` because it works without the flag and on Next 15. **On Next 16 with `cacheComponents`, prefer:**

```ts
import { cacheLife, cacheTag } from 'next/cache';
import { tagProduct, tagCatalog } from '@/lib/cache/tags';

export async function getProduct(id: string) {
  'use cache';
  cacheLife('hours');
  cacheTag(tagProduct(id), tagCatalog());

  const product = await db.products.findUnique({ where: { id } });
  if (!product) return null;
  return product;
}
```

- `'use cache'` is a directive inside the function/component, not a wrapper HOF.
- Call `cacheLife` / `cacheTag` in the same function (not after the fact).
- Do **not** invoke Server Actions inside a cached component. Pass the action reference down to a Client Component.

Without `cacheComponents`, keep `unstable_cache` + `next.tags` / `fetch` options from `SKILL.md`. Do not mix both styles on one data function.

## Invalidation on 16

| API | When | Semantics |
|-----|------|-----------|
| `updateTag(tag)` | Server Action after a mutation the **user must see now** | Next read waits for fresh data. Throws outside a Server Action. |
| `revalidateTag(tag, 'max')` | SWR is OK (catalogs, public pages, webhooks) | Stale served while revalidate runs. **Profile is required.** |
| `revalidatePath(path)` | Last resort when you cannot tag | Coarse. Do not `revalidatePath('/')` as a hammer. |
| `refresh()` | Dynamic (uncached) reads after a mutation | Complements tags; does not replace them. |

```ts
'use server';
import { updateTag, revalidateTag } from 'next/cache';
import { tagProduct, tagCatalog } from '@/lib/cache/tags';

export async function updateProduct(id: string, data: ProductInput) {
  await db.products.update({ where: { id }, data });
  updateTag(tagProduct(id));           // editor sees their save
  revalidateTag(tagCatalog(), 'max');  // public grid can lag
}
```

Single-arg `revalidateTag('posts')` is deprecated (TypeScript error on current types). Use `'max'` or `{ expire: 0 }` if you truly need blocking revalidate.

## Request-time APIs

Always `await`: `params`, `searchParams`, `cookies()`, `headers()`, `draftMode()`.

With Cache Components, reading these **outside** a Suspense boundary can fail the instant-navigation check. Pattern:

```tsx
export default function Page() {
  return (
    <>
      <CachedChrome />
      <Suspense fallback={<DashboardSkeleton />}>
        <Dashboard />  {/* cookies / session / searchParams */}
      </Suspense>
    </>
  );
}
```

## Middleware → Proxy

If you hit a middleware deprecation in 16, follow the version upgrade guide (`proxy` convention). Do not invent a second auth layer in Client Components.

## This repo

- App currently behaves as an SPA at `/` (client tabs, not `/worksheet` URLs). Calling that out is part of an architecture review — per-route RSC and `loading.tsx` do not apply until routes are split.
- Never run `next build` while `next dev` is running. Use `npx tsc --noEmit`.
