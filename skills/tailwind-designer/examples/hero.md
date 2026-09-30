# Hero

Marketing heroes are generous; app “heroes” (page headers) stay compact. Choose the density from context — do not drop a marketing hero on a dashboard.

## Marketing hero

```html
<section class="relative overflow-hidden">
  <div class="mx-auto flex min-h-[28rem] max-w-6xl flex-col justify-center gap-8 px-6 py-20 md:min-h-[32rem] md:px-8">
    <p class="text-sm font-medium uppercase tracking-wider text-muted-foreground">
      Sports betting operations
    </p>
    <h1 class="max-w-3xl text-balance text-4xl font-semibold tracking-tight text-foreground md:text-5xl">
      See every line, book, and edge in one workspace
    </h1>
    <p class="max-w-xl text-pretty text-base text-muted-foreground md:text-lg">
      Live odds, worksheets, and settlement without switching tabs.
    </p>
    <div class="flex flex-wrap items-center gap-3">
      <a href="/signup" class="inline-flex h-10 items-center rounded-md bg-primary px-4 text-sm font-medium text-primary-foreground ...">
        Start workspace
      </a>
      <a href="/docs" class="inline-flex h-10 items-center rounded-md border border-border bg-background px-4 text-sm font-medium ...">
        Documentation
      </a>
    </div>
  </div>
</section>
```

Notes:

- `text-balance` on the heading; `text-pretty` on the supporting paragraph
- Max width on copy (`max-w-3xl` / `max-w-xl`) so lines do not exceed ~70 characters
- CTAs: one primary, one outline. No third competing button
- Vertical padding `py-16`–`py-24`; do not use `h-screen` unless the brand page is truly a single viewport

## With background image

Put a scrim under type. Never set light text directly on a photo.

```html
<section class="relative isolate min-h-[28rem]">
  <img
    src="..."
    alt=""
    class="absolute inset-0 -z-20 size-full object-cover"
  />
  <div class="absolute inset-0 -z-10 bg-zinc-950/55"></div>
  <div class="mx-auto flex min-h-[28rem] max-w-6xl flex-col justify-center px-6 py-20 text-white">
    <h1 class="max-w-3xl text-balance text-4xl font-semibold tracking-tight">...</h1>
  </div>
</section>
```

Check contrast of the **scrim + text**, not the photo.

## App page header (not a hero)

Dashboards should use a compact header, not a marketing block.

```html
<header class="flex flex-wrap items-center justify-between gap-3 border-b border-border pb-4">
  <div class="min-w-0">
    <h1 class="text-xl font-semibold tracking-tight">Worksheet</h1>
    <p class="text-sm text-muted-foreground">Tonight’s board · ET</p>
  </div>
  <div class="flex items-center gap-2">
    <!-- filters / primary action -->
  </div>
</header>
```

## Split hero (copy + visual)

```html
<section class="mx-auto grid max-w-6xl items-center gap-10 px-6 py-16 md:grid-cols-2 md:gap-16">
  <div class="space-y-6">
    <h1 class="text-balance text-4xl font-semibold tracking-tight">...</h1>
    <p class="text-pretty text-muted-foreground">...</p>
    <div class="flex flex-wrap gap-3"><!-- CTAs --></div>
  </div>
  <div class="min-w-0">
    <img src="..." alt="Product screenshot" class="w-full rounded-xl border border-border shadow-sm" />
  </div>
</section>
```

Product shots: `rounded-xl border shadow-sm`, not a heavy drop shadow or fake browser chrome unless it exists in the brand.

## Rules

- Heading hierarchy: the hero title is `h1` if it is the page title
- Motion: optional fade is fine; do not autoplay large loops. `motion-reduce:animate-none`
- Full-bleed backgrounds belong on `<section>`, not on `body` in an authenticated shell
- No emoji in headlines or CTA labels
- On small screens, stack CTAs with `flex-wrap` and full-width only if they are the sole actions (`w-full sm:w-auto`)
