# RSC and the client boundary

Server Components are the default. `"use client"` is a **module** boundary: that file and every module it **statically imports** join the client bundle. Imports of Server Components **as children/slots** do not.

## When a file must be client

Needs one of: `useState` / `useReducer`, `useEffect`, event handlers (`onClick`, …), browser-only APIs (`window`, `localStorage`), certain context providers.

Otherwise leave it a Server Component — including forms that only `action={serverAction}`.

## Leaves, not pages

```
page.tsx          Server — fetch, compose
  ProductView     Server — present data
    AddToCart     Client — click + useTransition
    Chart         Client — heavy viz (optional dynamic import)
```

A `"use client"` on `page.tsx` or root `layout.tsx` is almost always a mistake: data fetching, `next/font`, and secrets no longer belong there, and the bundle includes the whole tree.

Exception: a documented SPA shell (this repo's `/` tabs). Treat it as debt in reviews; do not pretend it has per-route RSC.

## Composition: server children of client wrappers

Client components can accept `children` (or named slots) that were **already created** on the server:

```tsx
// sidebar.tsx
'use client';
export function ClientSidebar({ children }: { children: React.ReactNode }) {
  const [open, setOpen] = useState(true);
  return <aside>{open ? children : null}</aside>;
}

// page.tsx (Server)
export default async function Page() {
  const user = await getUser();
  return (
    <ClientSidebar>
      <ServerUserCard user={user} />
    </ClientSidebar>
  );
}
```

`ServerUserCard` is not imported by `sidebar.tsx`, so it stays on the server. **Importing** `ServerUserCard` from a client file would pull it across the boundary (and fail if it is async / uses `server-only`).

## Serializable props

Across the boundary, props must be JSON-like plus a few React types:

**OK:** string, number, boolean, null, plain objects/arrays, `Date` is risky — prefer ISO string, Server Action references, `ReactNode` children.

**Not OK:** class instances, `Map`/`Set`, Prisma models with methods, functions (except Server Actions), `Decimal` / `bigint` unless converted.

Map to a DTO in the DAL before passing into a Client Component. Never pass a raw DB row (password hashes, internal flags, PII extras).

## `server-only` / `client-only`

```ts
// lib/data/users.ts
import 'server-only';
```

Put this on every DAL, ORM, env-secret, and token module. Build fails if a Client Component imports it — that is the point.

```ts
// lib/client/analytics.ts
import 'client-only';
```

Shared formatters (`lib/utils/format.ts`) take **neither** guard so both sides can import them. If a util imports `fs` or `window`, it is not shared — split the file.

## Server Actions

```ts
'use server';

import { updateProduct } from '@/lib/data/products';
import { updateTag } from 'next/cache';
import { tagProduct } from '@/lib/cache/tags';

export async function updateProductAction(id: string, data: ProductInput) {
  const result = await updateProduct(id, data); // authz inside DAL
  updateTag(tagProduct(id));
  return result; // serializable DTO
}
```

- Actions are public endpoints. Validate input. Authz in DAL, always.
- Keep the action thin: parse → DAL → tag.
- Do not call the action from a `'use cache'` function; pass it to a client leaf.

## `next/dynamic` and RSC

- `next/dynamic(..., { ssr: false })` is for **Client Component modules** that must not prerender (`window` at import time).
- Do **not** use `{ ssr: false }` **inside** a Server Component to “skip SSR” of the page. Import the client leaf normally; only the leaf hydrates.
- Prefer `dynamic` for heavy client widgets (charts, editors) so they are their own chunk — still a client leaf, not a page.

## Context providers

A client `ThemeProvider` in root `layout.tsx` makes **that module and its imports** client, not the whole server tree of `children`. Still: keep the provider tiny. Do not import a 200-line client design system into `providers.tsx` if the pages are mostly RSC.

Auth session for display: fetch on the server, pass a DTO into a small client menu. Do not put `useSession()` at the layout root unless the product is already a client SPA.

## Mental test

If you deleted every `"use client"` file, would the page still show the right **data** (minus buttons)? If no, fetching leaked to the client.
