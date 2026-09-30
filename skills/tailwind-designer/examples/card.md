# Card

A card is a surface: one radius, one border, one shadow level. Content inside uses stacks and gaps, not extra nested cards.

## Default (dashboard)

```html
<section
  class="rounded-xl border border-border bg-card text-card-foreground shadow-sm"
>
  <header class="flex items-start justify-between gap-3 border-b border-border px-5 py-4">
    <div class="min-w-0">
      <h2 class="text-base font-semibold tracking-tight">Open tickets</h2>
      <p class="mt-1 text-sm text-muted-foreground">Unresolved in the last 7 days</p>
    </div>
    <button type="button" class="...ghost sm">View all</button>
  </header>
  <div class="px-5 py-4">
    <!-- body: table, list, or metrics. gap-4 stack -->
  </div>
  <footer class="border-t border-border px-5 py-3 text-xs text-muted-foreground">
    Updated 2 minutes ago
  </footer>
</section>
```

Without tokens:

```html
<section class="rounded-xl border border-zinc-200 bg-white text-zinc-900 shadow-sm dark:border-zinc-800 dark:bg-zinc-950 dark:text-zinc-50">
```

## Padding and density

| Context | Padding | Radius | Shadow |
|---------|---------|--------|--------|
| Dashboard widget | `p-4` / header `px-5 py-4` | `rounded-xl` | `shadow-sm` |
| Marketing feature | `p-6 md:p-8` | `rounded-2xl` | `shadow-sm` or none + border |
| Compact KPI | `p-4` | `rounded-lg` | none or `shadow-sm` |
| Nested (avoid) | — | — | do not nest cards; use `divide-y` |

Sibling cards in a grid: `gap-4` or `gap-6`, equal min-height only if the row is a KPI set (`items-stretch`).

## KPI card

```html
<article class="rounded-xl border border-border bg-card p-4 shadow-sm">
  <p class="text-xs font-medium uppercase tracking-wide text-muted-foreground">Ticket sold</p>
  <p class="mt-2 text-2xl font-semibold tabular-nums tracking-tight">1,248</p>
  <p class="mt-1 text-xs text-emerald-600 dark:text-emerald-400">+4.2% vs last week</p>
</article>
```

Use `tabular-nums` on numbers. Trend color is accompanied by `+`/`-` text, not color alone.

## Interactive card (clickable)

Prefer an inner link or button over making the whole article a `<div onClick>`. If the card is a link:

```html
<a
  href="/games/123"
  class="block rounded-xl border border-border bg-card p-4 shadow-sm
         transition-shadow duration-200 hover:shadow-md
         focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring"
>
  ...
</a>
```

Do not animate `translate-y` on dense grids (causes sibling jump). Marketing grids may use `hover:shadow-md`.

## Media

```html
<article class="overflow-hidden rounded-xl border border-border bg-card shadow-sm">
  <img src="..." alt="" class="aspect-video w-full object-cover" />
  <div class="space-y-2 p-4">
    <h3 class="text-sm font-semibold">Title</h3>
    <p class="text-sm text-muted-foreground">Supporting line.</p>
  </div>
</article>
```

Team logos: `h-8 w-auto object-contain`, **no** rounded square tile or bordered badge unless requested.

## Rules

- One visual elevation per card (`border` **or** heavy shadow, not both fighting)
- Header title `font-semibold`; meta `text-muted-foreground text-sm`
- Overflow: `min-w-0` on text columns; `truncate` only when the full string is available via `title` or a detail view
- Do not put raw tables against the card edge without `overflow-x-auto`
