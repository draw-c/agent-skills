# Motion

Motion clarifies cause and effect. It must be short, consistent, and optional (`prefers-reduced-motion`).

## Defaults

| Kind | Duration | Easing | Utilities |
|------|----------|--------|-----------|
| Color, opacity, shadow | 150ms | out | `duration-150 ease-out` |
| Transform (hover lift, press) | 150–200ms | out | `duration-200 ease-out` |
| Overlay / popover | 150–200ms | out (in), in (out) | `duration-200` |
| Drawer / sheet | 250–300ms | out | `duration-300 ease-out` |
| Page / large layout | 300ms max | out | avoid except shared-element |

Use Tailwind’s scale, not one-off `duration-[183ms]`.

Standard easings:

- Enter / hover: `ease-out` (start fast, settle)
- Exit: `ease-in` (or `ease-out` if very short)
- Looping: `ease-in-out` or `linear` (spinners)

CSS equivalents when defining `@theme`:

```css
@theme {
  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
}
```

## What should move

**Do animate**

- Hover color and `shadow` on buttons/cards
- `translate-y` 1px on active (`active:translate-y-px`)
- Opacity + slight translate on menus, toasts, dialogs
- Accordion height (grid `grid-rows-[0fr]` / `[1fr]` or Radix height)
- Skeleton shimmer (opacity or background-position)

**Do not animate**

- Layout shifts (`width`/`height`/`top` on large regions) — prefer `transform` and `opacity`
- Infinite bounce on important CTAs
- Parallax that fights scrolling
- Different durations on sibling controls in the same toolbar

## Recipes

### Interactive control

```html
<button
  class="transition-colors duration-150 ease-out
         hover:bg-primary/90 active:translate-y-px
         disabled:pointer-events-none disabled:opacity-50"
>
  Save
</button>
```

Prefer `transition-colors` or `transition-shadow` over `transition-all` on dense UIs (avoids accidental layout transitions).

### Card hover lift

```html
<article
  class="rounded-xl border border-border bg-card shadow-sm
         transition-shadow duration-200 ease-out
         hover:shadow-md"
>
  ...
</article>
```

Lift with shadow, not `hover:-translate-y-1`, unless the surface is a marketing card with room to move. Dashboard tables should not jump.

### Fade + rise enter

```html
<div
  class="animate-in fade-in slide-in-from-bottom-2 duration-200
         motion-reduce:animate-none"
>
  ...
</div>
```

If `tailwindcss-animate` (v3) or v4 `--animate-*` is unavailable:

```html
<div class="opacity-0 translate-y-1 transition duration-200 ease-out data-[open]:opacity-100 data-[open]:translate-y-0">
```

### Overlay (dialog)

- Backdrop: `opacity` 150ms
- Panel: opacity + `scale-[0.98]` → `scale-100`, 200ms `ease-out`
- Exit faster than enter (150ms) so dismiss feels snappy

### Reduced motion

Always pair decorative animation:

```html
<div class="transition-transform duration-200 motion-reduce:transition-none motion-reduce:transform-none">
```

For CSS keyframes:

```css
@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

Prefer the targeted `motion-reduce:` utilities over a global nuke when the product still needs opacity fades.

## Loading motion

- Spinners: `animate-spin`, `size-4`, muted color, `aria-hidden="true"` with an `sr-only` status
- Skeletons: `animate-pulse bg-muted rounded-md`; never pulse text that is already real content
- Progress: `transition-[width] duration-300 ease-out` on the bar only

## Checklist

- [ ] Duration ≤ 300ms except rare full-page transitions
- [ ] `transition-all` not used on large trees
- [ ] Hover/focus not the only way to understand state
- [ ] `motion-reduce:` or equivalent on decorative motion
- [ ] Enter/exit do not trap focus before animation ends
