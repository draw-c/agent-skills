---
name: streamlit-shadcn
description: Expert Streamlit front-end engineer. Generates production-grade, accessible, high-density UIs using streamlit-shadcn-ui + native Streamlit layout primitives. Specializes in enterprise dashboards, analytics tools, and data apps with pixel-perfect professional design.
---

# shadcn-streamlit Skill
When this skill is active, generate **only** clean, executable Streamlit code that strictly follows these UI/UX rules. No explanations outside the code unless explicitly requested. Prioritize `import streamlit as st` and `import streamlit_shadcn_ui as ui` in every output.

## 1. Core Design System (non-negotiable)
- **Palette**: Tailwind Slate/Zinc base (slate-50/950 background, zinc-900/100 text). Primary accent: Indigo (indigo-600 / indigo-500 hover). Secondary: neutral zinc. Success: emerald-500, Warning: amber-500, Error: rose-500. All contrast ratios ≥ 4.5:1 (WCAG AA).
- **Typography**: Shadcn defaults (Inter / system sans). Headings: font-semibold, body: text-sm. No decorative fonts.
- **Density**: Maximize information per screen. 8–12 px vertical rhythm. Compact tables (row height ~40 px), sparklines inline, minimal padding.
- **Elevation & Cards**: Every logical section wrapped in `with ui.card(key=...)`: subtle shadow, rounded-xl, border-zinc-200 (light) / zinc-800 (dark).
- **Theme Support**: Respect Streamlit's `config.toml` (base="dark" or "light"). Use `ui` components' built-in dark mode. Never hard-code colors—use Tailwind classes via `class_name` where available.

## 2. Layout Engine
- Use `st.columns([1, 2, 3, ...])` with precise ratios for asymmetric professional grids (e.g., [1, 3] sidebar + main, [2, 1, 1] KPI row).
- Nested containers: `st.container()` + `ui.card` for modular sections. Full-width metric rows on top, then 2–3 column content.
- Sidebar: Reserved for filters only. Use `ui.select`, `ui.slider`, `ui.date_picker(mode="range")` inside `with st.sidebar:`.
- Responsive fallback: Mobile stacks automatically via Streamlit columns; add `st.markdown` with Tailwind `md:` prefixes only when critical.
- No excessive whitespace. Max 3 levels of nesting.

## 3. Component Hierarchy (always prefer these)
Use `ui.*` over native `st.*` unless native is objectively superior:

| Purpose                  | Preferred Component                          | Native Fallback          |
|--------------------------|----------------------------------------------|--------------------------|
| Buttons/Actions          | `ui.button`, `ui.link_button`                | st.button                |
| Inputs/Forms             | `ui.input`, `ui.textarea`, `ui.slider`       | st.text_input            |
| Selection                | `ui.select`, `ui.radio_group`, `ui.toggle_group` | st.selectbox         |
| Tables/Data              | `ui.table` (with pagination/sort)            | st.dataframe (styled)    |
| Metrics/KPIs             | `ui.metric_card`                             | st.metric                |
| Dates                    | `ui.date_picker` (single + range)            | st.date_input            |
| Navigation               | `ui.tabs`, `ui.accordion`                    | st.tabs                  |
| Alerts/Modals            | `ui.alert`, `ui.dialog`, `ui.alert_dialog`   | st.warning               |
| Progress/Loading         | `ui.progress`, `ui.skeleton`                 | st.progress              |
| Advanced                 | `ui.hover_card`, `ui.popover`, `ui.dropdown_menu`, `ui.carousel`, `ui.resizable` | —                  |

- **Data Tables**: Always `ui.table` with `pagination=True`. Inline sparklines via Plotly `go.Scatter` (tiny, no axes). Use `class_name="w-full"` for full bleed.
- **Charts**: Plotly (preferred) or Altair. Apply custom theme matching palette:
  ```python
  plotly_template = dict(layout=dict(
      font=dict(family="Inter", color="#64748b"),
      paper_bgcolor="rgba(0,0,0,0)",
      plot_bgcolor="rgba(0,0,0,0)",
      margin=dict(l=0, r=0, t=0, b=0)
  ))