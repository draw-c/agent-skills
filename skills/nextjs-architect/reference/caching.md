# Caching

Three different “caches.” Name which one you mean or you will invalidate the wrong thing.

| Layer | What it stores | Identity | Freshness |
|-------|----------------|----------|-----------|
| React `cache()` | Per-request memo of a function | Function + args, one render | Request only. Not shared across users. |
| Next data / full-route cache | `fetch` result, `'use cache'` output, RSC payload | Cache key + tags | TTL / `cacheLife` / on-demand tags |
| Client (router + SWR/TanStack) | RSC payload, browser query cache | URL / query key | Prefetch + library revalidate |

`cache()` does **not** replace tags. Tags do **not** dedupe two `getUser()` calls in one RSC tree. Use both: `cache()` for one render, tags for cross-request reuse.

## Tag taxonomy

Put names in `lib/cache/tags.ts`. One helper per entity. Never stringify ad-hoc in five files.

```ts
export const tagProduct = (id: string) => `product:${id}`;
export const tagCatalog = () => `catalog`;
export const tagUser = (id: string) => `user:${id}`;
```

Rules:

- Tag the **entity you mutate**, plus any **aggregate** that includes it (`product:123` and `catalog`).
- Do not tag `"user"` for all users. Tag `user:${id}`.
- List tags next to the DAL function that reads, not only next to the action that writes.

## Choosing a strategy

1. **Must be correct for this user, now** (session, cart, worksheet after save) → uncached or `updateTag` after the mutation. Wrap in `<Suspense>`.
2. **Shared, slowly changing** (catalog, team names, CMS) → `'use cache'` + `cacheLife` + tags (or `unstable_cache` / `fetch` `next.tags` if Cache Components is off).
3. **One render, many callers** (auth in layout + page + widget) → React `cache()` around the DAL read.
4. **Browser list filters** → client library (SWR/TanStack) **in addition to** server cache, not instead of it. Keep keys aligned with tags.

## Next 15 / no Cache Components

```ts
import { unstable_cache } from 'next/cache';

export const getProduct = (id: string) =>
  unstable_cache(
    async () => db.products.findUnique({ where: { id } }),
    [tagProduct(id)],
    { revalidate: 3600, tags: [tagProduct(id), tagCatalog()] },
  )();
```

`fetch` options: `cache: 'force-cache' | 'no-store'`, `next: { revalidate, tags }`. **Do not assume GET is cached** — default has shifted across versions. Be explicit.

Invalidate:

```ts
revalidateTag(tagProduct(id)); // 15: single arg OK
```

## Next 16 + `cacheComponents`

See `reference/next-16.md`. Opt in with `'use cache'`. `revalidateTag(tag, 'max')`. `updateTag` for read-your-writes.

Do not export `dynamic` / `revalidate` / `fetchCache` on the segment.

## Mutations

Every Server Action that changes user-visible data:

1. Authz in the DAL (`server-only`), not only in the UI.
2. Mutate.
3. Invalidate the same tags the readers used.
4. Return a serializable result (not a Prisma row).

Prefer tags over `revalidatePath`. Path invalidation is for “this URL is a snowflake” or third-party webhooks that cannot name an entity.

## Waterfalls vs cache misses

- Sequential `await getA(); await getB(a.id)` is a **data dependency**, not a cache bug. If `getB` only needs `a.id` and you already have it from the URL, fetch in parallel.
- Nested Suspense: cached shell outside, personalized inside. Do not put the catalog fetch **inside** the session boundary.
- Layouts that call `cookies()` make the **layout** dynamic. Push session UI into a child behind Suspense.

## What not to cache

- Secrets, CSRF tokens, one-time codes
- Responses that differ by `Set-Cookie` / `Authorization` unless the cache key includes that identity
- Errors (do not `unstable_cache` a function that throws NotFound as the happy path without handling)

## Verify

- Mutation: save → reload → user sees their write (`updateTag`) or public page updates within SWR window (`revalidateTag`).
- Dev: with Cache Components, instant-navigation errors name the blocking read.
- Prod: cache hit rate / tag churn. A tag that invalidates on every request is not a cache.
