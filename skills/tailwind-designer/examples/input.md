# Input

Label above field, helper or error below. Height aligns with buttons (`h-9` default) so forms and toolbars share a baseline.

## Text field

```html
<div class="grid gap-1.5">
  <label for="email" class="text-sm font-medium text-foreground">
    Email
  </label>
  <input
    id="email"
    name="email"
    type="email"
    autocomplete="email"
    placeholder="you@club.com"
    class="flex h-9 w-full rounded-md border border-input bg-background px-3
           text-sm text-foreground shadow-sm
           placeholder:text-muted-foreground
           focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring
           disabled:cursor-not-allowed disabled:opacity-50
           file:border-0 file:bg-transparent file:text-sm file:font-medium"
  />
  <p id="email-hint" class="text-xs text-muted-foreground">
    We’ll send a receipt to this address.
  </p>
</div>
```

Wire `aria-describedby="email-hint"` on the input when the hint exists.

## Error

```html
<div class="grid gap-1.5">
  <label for="stake" class="text-sm font-medium">Stake</label>
  <input
    id="stake"
    name="stake"
    type="text"
    inputmode="decimal"
    aria-invalid="true"
    aria-describedby="stake-error"
    class="h-9 rounded-md border border-destructive bg-background px-3 text-sm
           focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-destructive"
  />
  <p id="stake-error" class="text-xs text-destructive">Enter a number greater than 0.</p>
</div>
```

Do not rely on a red border alone.

## Horizontal (toolbar / filter)

```html
<div class="flex flex-wrap items-end gap-3">
  <div class="grid gap-1.5">
    <label for="q" class="text-xs font-medium text-muted-foreground">Search</label>
    <input id="q" type="search" class="h-9 w-64 rounded-md border border-input bg-background px-3 text-sm" />
  </div>
  <button type="submit" class="h-9 ...">Apply</button>
</div>
```

Labels can be `text-xs` in dense filters; they still exist in the DOM.

## Textarea and select

```html
<textarea
  id="notes"
  rows="4"
  class="min-h-20 w-full rounded-md border border-input bg-background px-3 py-2 text-sm
         focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring"
></textarea>

<select
  id="book"
  class="h-9 w-full rounded-md border border-input bg-background px-3 text-sm
         focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring"
>
  <option value="">Select book</option>
</select>
```

Native `<select>` is acceptable. Custom listboxes must implement keyboard support (prefer Radix / Headless UI).

## With leading icon

```html
<div class="relative">
  <svg class="pointer-events-none absolute left-2.5 top-1/2 size-4 -translate-y-1/2 text-muted-foreground" aria-hidden="true"></svg>
  <input type="search" aria-label="Search games" class="h-9 w-full rounded-md border border-input bg-background pl-8 pr-3 text-sm" />
</div>
```

If there is no visible label, `aria-label` is required.

## Checkbox / switch

```html
<label class="flex items-center gap-2 text-sm">
  <input type="checkbox" class="size-4 rounded border-input text-primary focus-visible:ring-ring" />
  Include graded bets
</label>
```

Hit target: the label wraps the control. Do not use a 12px box with a separate tiny caption.

## Rules

- Never use placeholder as the only label
- `disabled:opacity-50` + `disabled:cursor-not-allowed`
- Number-ish betting fields: `tabular-nums`, `inputmode="decimal"` where appropriate
- Autofill-safe: do not set a fixed dark `bg` that clashes with Chrome autofill without `shadow` inner fill if needed
- Dark mode: `bg-background` / `border-input`, not `bg-white` unless paired with `dark:bg-zinc-950`
