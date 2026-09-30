# Button

Primary action is solid; secondary is outline or muted fill; destructive is rare. One primary per visible cluster.

## Anatomy

Layout → type → color → states → motion. Always include `disabled` and `focus-visible`.

```html
<button
  type="button"
  class="inline-flex items-center justify-center gap-2
         rounded-md px-3 py-2 text-sm font-medium
         bg-primary text-primary-foreground shadow-sm
         hover:bg-primary/90 active:translate-y-px
         focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring
         focus-visible:ring-offset-2 focus-visible:ring-offset-background
         disabled:pointer-events-none disabled:opacity-50
         transition-colors duration-150 ease-out"
>
  Save changes
</button>
```

If the project has no `primary` tokens, use an explicit ramp and pair dark mode:

```html
<button
  type="button"
  class="inline-flex items-center justify-center gap-2 rounded-md px-3 py-2
         text-sm font-medium bg-zinc-900 text-zinc-50
         hover:bg-zinc-800 dark:bg-zinc-100 dark:text-zinc-900 dark:hover:bg-zinc-200
         focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-zinc-400
         disabled:pointer-events-none disabled:opacity-50
         transition-colors duration-150"
>
  Save changes
</button>
```

## Variants

| Variant | Surface | Use |
|---------|---------|-----|
| Primary | `bg-primary text-primary-foreground` | One main action |
| Secondary | `bg-muted text-foreground hover:bg-muted/80` | Adjacent actions |
| Outline | `border border-border bg-background hover:bg-muted` | Low emphasis |
| Ghost | `hover:bg-muted` | Toolbars, icon actions |
| Destructive | `bg-destructive text-destructive-foreground` | Delete; confirm in a dialog |

Sizes:

- `sm`: `h-8 px-2.5 text-xs`
- `md` (default): `h-9 px-3 text-sm`
- `lg`: `h-10 px-4 text-sm`
- Icon: `size-9` (md) or `size-8` (sm), `p-0`

## With icon

```html
<button type="button" class="inline-flex h-9 items-center gap-2 rounded-md bg-primary px-3 text-sm font-medium text-primary-foreground ...">
  <svg class="size-4 shrink-0" aria-hidden="true"><!-- Lucide --></svg>
  Export
</button>
```

Icon-only:

```html
<button type="button" aria-label="Open menu" class="inline-flex size-9 items-center justify-center rounded-md ...">
  <svg class="size-4" aria-hidden="true"></svg>
</button>
```

## Loading

Keep the original width (`min-w` or disabled label swap). Spinner is decorative.

```html
<button type="button" disabled aria-busy="true" class="...">
  <svg class="size-4 animate-spin" aria-hidden="true"></svg>
  <span>Saving</span>
</button>
```

## Rules

- `type="button"` unless it submits a form (`submit`)
- Do not wrap a button in another button or nest interactive elements
- Do not use emoji. Lucide (or equivalent) at `size-4`
- Width: `w-full` only in stacked mobile forms, not in header toolbars
- Brand teals that fail contrast: use as `bg-*` only with a darker `text-*` pair, or invert (dark text on light teal)
