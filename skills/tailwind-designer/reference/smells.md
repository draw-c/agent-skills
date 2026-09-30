# Design smells — "It's off and I can't say why"

Name the smell, then apply the fix. Do not stack fixes — pick the one or two that restore hierarchy.

## Layout

- **Broken vertical rhythm.** Mix of 13px / 17px / 22px or arbitrary `p-[13px]`. Fix: one scale (4/8/12/16/24/32/48/64) and stay on it.
- **Fake centering.** Mathematically centered, optically low. Fix: `text-balance` on heads; optical nudge only after the type is right.
- **Grid exhaustion.** Every region is a 3-column grid. Fix: vary rhythm. Asymmetry (1–2, 2–1, full-bleed) feels designed.
- **Edge-to-edge text.** Body kissing the container. Fix: real padding and `max-w-prose` / `max-w-2xl` on prose. Never center long-form copy.
- **The grid is fighting.** Overflow, truncated titles, `space-y-*` and `gap-*` on the same node, `h-screen` under mobile chrome, card-in-card. Fix: shrinking children get `min-w-0`. One spacing method per container. `min-h-dvh`. Flatten nested cards to `divide-y` or inner `gap`.

## Typography

- **Flat hierarchy.** Headline and body too close in size or weight. Fix: bigger spread. If body is 16px, display is 32px+. Three sizes max in a region (title, body, meta).
- **Letter-spacing wrong.** Display without `tracking-tight`; all-caps without `tracking-wide`; body with tight tracking. Fix: tracking follows size and case.
- **Line-height off.** Display too loose or body too tight. Fix: display `leading-none`–`leading-tight`; body `leading-relaxed`.
- **Too many weights.** 4+ weights is noise, not hierarchy. Fix: 2–3 total per page.
- **Type has no voice.** Everything `text-sm font-medium`; centered blog paragraphs. Fix: heads `font-semibold` + `text-balance`; body `text-pretty`, left-aligned; meta `text-muted-foreground`.
- **Numbers look amateur.** Proportional figures jumping in KPIs; too many decimals; green/red without a sign. Fix: `tabular-nums tracking-tight`. Trim precision. Trend is `+4.2%` plus color, never color alone.

## Color

- **Accent chaos.** Brand color in 12 places, meaning nothing. Fix: 10% rule. One chromatic accent. If it's everywhere, it's nowhere.
- **Contrast halo.** Text on a busy photo or pattern. Fix: scrim (`bg-black/40`–`/55`), not hope. Check contrast of scrim + text.
- **Muddy neutrals.** Gray that leans brown next to a blue-gray border. Fix: one ramp (zinc or slate or stone). OKLCH with a shared hue when minting tokens.
- **Inverted dark mode.** Light mode flipped; same L on primary; shadows like dirt. Fix: retune L/C per theme (`reference/color.md`). Lower chroma in dark. Prefer hairline rings over drop shadows.
- **Contrast is lying.** `text-muted-foreground` as the label; mint/teal brand as body text; `border-white/5` on `bg-zinc-950`. Fix: labels `text-foreground`. Brand fills are backgrounds unless the darker sibling passes 4.5:1. Dark borders go up a step on the ramp.

## Depth

- **Shadow vomit.** Too many shadows, each too strong (`shadow-xl` on a tiny card). Fix: two layers max — one contact, one ambient — or `shadow-sm` plus `ring-1 ring-black/5`.
- **Border addiction.** Everything boxed. Fix: proximity and whitespace first. Borders last. One elevation language per surface.
- **Flat soup.** Every region the same z-plane and the same `bg-background`. Fix: muted tints, hairline rings, blur only on overlays.
- **Two brands.** Arbitrary hex beside tokens; `rounded-md` next to `rounded-3xl`. Fix: existing CSS variables / `@theme`. Controls `rounded-md`, surfaces `rounded-xl`. Two typefaces max.

## Motion

- **Transition-all.** Animating layout you didn't mean to. Fix: `transition-colors`, `transition-transform`, `transition-shadow`.
- **Same duration everywhere.** Mechanical. Fix: 150ms color/opacity, 200–250ms transform, 300ms drawers. Ease-out in, ease-in out.
- **Hover without focus.** Mouse-only. Fix: every `hover:` has `focus-visible:`. Never `outline-none` without a ring.
- **Motion as decoration.** `hover:-translate-y-1` on a dense grid; infinite bounce on a CTA; no `motion-reduce:`. Fix: dashboard cards change shadow, not position. `motion-reduce:transition-none` / `animate-none`.

## Interaction

- **Missing states.** No disabled, loading, or error. Fix: default, hover, focus-visible, active, disabled, loading — skip none.
- **Dead affordances.** Looks clickable and isn't, or static text has a hover fill. Fix: real `<button>` / `<a href>`. Hover only on things that do something.
- **No feedback.** Click, then 400ms of nothing. Fix: busy spinner with `aria-busy`, or optimistic UI. Keep the control's width.
- **Emoji as UI.** Emoji in tabs, badges, buttons, headers. Fix: Lucide (or equivalent) at `size-4`. Logo marks sit on the background — no square tiles.

## Space

- **Whitespace with no purpose.** Gaps that don't group or separate. Fix: space is hierarchy. If it doesn't mean something, delete it.
- **Cramped containers.** Content kissing edges. Fix: padding as breathing room, not a cage.
- **Uniform spacing.** Every gap is `gap-4`. Fix: section > group > element (e.g. 32 / 16 / 8).
- **Density clash.** Dashboard with `py-16` heroes; marketing with `gap-2` cards; mixed padding in one card. Fix: pick a density. Dashboard: `gap-3/4`, `text-sm`, `p-4`. Marketing: `gap-8/12`, `p-6 md:p-8`. Same padding on header/body/footer.

## The fix-everything pass

1. Squint. What does the eye hit first? Should it?
2. Grayscale. If it collapses, you leaned on color.
3. Kill animation. Does it still feel designed?
4. Tab through. Is focus obvious everywhere?
5. 375px. Designed, or merely fitting?
6. Delete anything you can't justify in one sentence.
