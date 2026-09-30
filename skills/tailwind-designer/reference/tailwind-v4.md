# Tailwind CSS v4

Use this file when the project is on Tailwind v4 (CSS-first). If `package.json` has `tailwindcss` ^3, or the app uses `tailwind.config.js` + PostCSS `tailwindcss` plugin, **do not apply this file** — stay on the JS config.

## How to detect v4

- `tailwindcss` ^4 in package.json
- CSS entry contains `@import "tailwindcss"`
- `@theme`, `@utility`, `@plugin`, `@source` in CSS
- Vite/Next using `@tailwindcss/vite` or `@tailwindcss/postcss` instead of `tailwindcss: {}` in PostCSS

v3 entry still looks like:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## CSS-first configuration

v4 is configured in CSS. Import the framework once, then declare tokens in `@theme`.

```css
@import "tailwindcss";

@theme {
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "SF Mono", ui-monospace, monospace;

  --color-background: oklch(0.985 0.002 247);
  --color-foreground: oklch(0.21 0.02 260);
  --color-muted: oklch(0.97 0.005 247);
  --color-muted-foreground: oklch(0.55 0.02 260);
  --color-border: oklch(0.91 0.01 247);
  --color-ring: oklch(0.55 0.12 230);
  --color-primary: oklch(0.55 0.12 230);
  --color-primary-foreground: oklch(0.99 0.005 230);
  --color-destructive: oklch(0.55 0.2 25);

  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;

  --shadow-card: 0 1px 2px oklch(0.2 0.02 260 / 0.06);
}

@layer base {
  html {
    @apply bg-background text-foreground antialiased;
  }
  * {
    @apply border-border;
  }
}
```

Theme keys map to utilities:

| Token | Utility |
|-------|---------|
| `--color-primary` | `bg-primary`, `text-primary`, `border-primary` |
| `--color-primary-foreground` | `text-primary-foreground` |
| `--radius-lg` | `rounded-lg` |
| `--font-sans` | `font-sans` |
| `--shadow-card` | `shadow-card` |

Namespaces that generate utilities include `--color-*`, `--font-*`, `--text-*`, `--spacing-*`, `--radius-*`, `--shadow-*`, `--breakpoint-*`, `--ease-*`, `--animate-*`.

## Custom utilities

```css
@utility stack {
  display: flex;
  flex-direction: column;
  gap: --spacing(4);
}

@utility cluster {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: --spacing(2);
}

@utility focus-ring {
  outline: 2px solid transparent;
  outline-offset: 2px;
  &:focus-visible {
    outline-color: var(--color-ring);
  }
}
```

Use `@utility` for repeated, named patterns. Do not recreate core utilities (`flex`, `grid`, `p-4`).

## Dark mode in v4

Default is `prefers-color-scheme`. For a `.dark` class (shadcn-style):

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-background: oklch(0.985 0.002 247);
  --color-foreground: oklch(0.21 0.02 260);
}

@layer base {
  .dark {
    --color-background: oklch(0.18 0.02 260);
    --color-foreground: oklch(0.97 0.005 247);
  }
}
```

Override tokens on `.dark`; do not duplicate entire component class lists unless the pair cannot be expressed as a token.

## `@source` and scanning

v4 scans source files automatically. If classes are built in unusual paths (monorepo packages, generated files):

```css
@source "../packages/ui/**/*.tsx";
@source inline("bg-primary text-primary-foreground");
```

## `@plugin` (JS plugins)

```css
@plugin "@tailwindcss/typography";
@plugin "tailwindcss-animate";
```

v3 `require('tailwindcss-animate')` in `tailwind.config.js` becomes `@plugin` in CSS.

## Migration notes (v3 → v4)

- Replace `@tailwind base/components/utilities` with `@import "tailwindcss"`.
- Move `theme.extend` colors, fonts, radii into `@theme`.
- `darkMode: 'class'` → `@custom-variant dark (&:where(.dark, .dark *));`
- `content: [...]` → `@source` only when defaults miss files.
- Arbitrary values still work: `w-[18rem]`, `bg-[oklch(0.7_0.1_230)]`.
- Some deprecated utilities were removed; prefer `min-h-dvh` over old `min-h-screen` hacks on mobile, `size-*` over `w-* h-*` pairs.

## Mixing with v3 projects

This workspace's Next app may still be on Tailwind 3 (`tailwind.config.ts` + PostCSS). In that case:

- Put tokens in `theme.extend` (colors as CSS variables if shadcn).
- Do **not** add `@theme` blocks to `globals.css`.
- Component examples in this skill still apply; only the token plumbing differs.
