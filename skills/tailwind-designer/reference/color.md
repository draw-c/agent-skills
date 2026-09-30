# Color

Color is a ramp plus one accent. Pick neutrals first, then a primary that survives both light and dark backgrounds.

## OKLCH

Prefer OKLCH for tokens (perceptual lightness is independent of hue). Tailwind v4 `@theme --color-*` and modern browsers accept it.

```css
--color-primary: oklch(0.55 0.12 230); /* L C H */
```

- **L** (0–1): perceived lightness. Body text on white ≈ 0.25–0.35; body text on dark ≈ 0.85–0.97.
- **C**: chroma. Neutrals stay 0.004–0.03. Accents 0.10–0.18. Above ~0.22 looks neon and fails on large fills.
- **H**: hue angle. Keep a family (±10) for hover/active of the same role.

When the project is already hex/HSL (shadcn CSS variables), **do not convert everything in one PR**. Add new tokens in the existing format, or wrap hex in CSS variables the utilities already reference.

## Neutral ramp

Build surfaces from one ramp (zinc, slate, or stone). Do not mix zinc borders with slate backgrounds.

| Role | Light (approx L) | Dark (approx L) | Typical utility |
|------|------------------|-----------------|-----------------|
| Page background | 0.98–0.99 | 0.16–0.20 | `bg-background` |
| Raised card | 1.0 / 0.99 | 0.20–0.24 | `bg-card` |
| Muted fill | 0.96–0.97 | 0.25–0.28 | `bg-muted` |
| Border | 0.90–0.92 | 0.30–0.34 | `border-border` |
| Muted text | 0.45–0.55 | 0.65–0.75 | `text-muted-foreground` |
| Body text | 0.20–0.28 | 0.93–0.97 | `text-foreground` |

Hover on a fill is one step on the ramp (`hover:bg-muted`), not a new hue.

## Accent rules

- **One primary** for the main CTA and key active nav.
- Primary on a button must pair with `primary-foreground` that meets 4.5:1.
- Teal / mint brand fills (high L, moderate C) often fail as **text**. Use them as backgrounds, sidebar, or charts; use a darker sibling for text/icons.
- Semantic colors are not decorative: success, warning, destructive. Do not reuse destructive for “attention” marketing.
- Charts: 4–6 hues, similar L/C so no series screams. Grid lines stay `border` / muted, not black.

## Dark mode pairing

Every component that sets a color sets the dark pair in the same class string **or** via tokens so one class works in both themes.

Token approach (preferred):

```css
:root { --color-background: oklch(0.99 0.005 247); }
.dark { --color-background: oklch(0.18 0.02 260); }
```

```html
<div class="bg-background text-foreground">
```

If tokens do not exist (one-off):

```html
<div class="bg-zinc-50 text-zinc-900 dark:bg-zinc-950 dark:text-zinc-50">
```

Never ship `bg-white text-black` without `dark:` unless the product is light-only.

Borders in dark mode should be lighter than the background (`oklch` L +0.08 to +0.12), not pure white at 10% unless the rest of the system already uses alpha borders.

## Contrast (WCAG AA)

| Text | Minimum vs background |
|------|------------------------|
| Body (< 18px / < 14px bold) | 4.5:1 |
| Large text | 3:1 |
| UI icons and input borders that convey state | 3:1 |

`text-muted-foreground` is the usual failure. If muted text is the **only** label, darken it. Placeholders may be lighter; they are not labels.

Disabled: `opacity-50` plus `pointer-events-none` is acceptable; do not rely on color alone for “you cannot click this.”

## Alpha and overlays

- Overlays: `bg-black/50` or `bg-zinc-950/60` with `backdrop-blur-sm` optional. Keep blur light (sm) on dashboards.
- Hover on primary: `hover:bg-primary/90` if primary is a CSS color; or a dedicated `--color-primary-hover` at L ±0.04.
- Do not stack `bg-white/5` on `bg-white/5` parents until the surface is unreadable.

## Project notes

If the app documents a brand (e.g. teal sidebar, slate body):

- Put brand on **named regions** (sidebar, chart, active pill), not on every border.
- Page canvas stays near-white / near-zinc; cards `bg-white` / `bg-card` with `border-border`.
- Status pills: solid-light background + darker text (`bg-emerald-500/10 text-emerald-700 dark:text-emerald-400`).

See `examples/` for how tokens land on buttons, cards, and inputs.
