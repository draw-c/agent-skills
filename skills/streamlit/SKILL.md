---
name: streamlit
description: Comprehensive Streamlit skill covering app structure, layout, widgets, data visualization, session state, caching, and deployment. Use for building any Streamlit web application from simple dashboards to multi-page production apps.
---

# Streamlit

Streamlit turns Python scripts into shareable web apps. This skill covers the complete API surface: layout, widgets, data display, charts, media, state management, caching, and deployment patterns.

## When to use this skill

- Building or modifying any Streamlit application.
- Adding UI elements: text, inputs, buttons, tables, charts, images, video.
- Structuring multi-page apps, sidebars, columns, tabs, or expandable sections.
- Persisting data across reruns with session state.
- Optimizing performance with caching.
- Managing secrets and deploying to Streamlit Cloud or other platforms.

## 1. App Structure & Configuration

```python
import streamlit as st

st.set_page_config(
    page_title="My App",
    page_icon="🚀",
    layout="centered",              # or "wide"
    initial_sidebar_state="expanded",  # or "collapsed"
    menu_items={
        "Get Help": "https://...",
        "Report a bug": "https://...",
        "About": "# About this app"
    }
)
```

**Rules:**
- `st.set_page_config` must be the **first** Streamlit command in the script. Call it exactly once.
- `layout="wide"` uses the full browser width; `centered` constrains content to a centered column.

## 2. Layout Containers

| API | Purpose |
|-----|---------|
| `st.sidebar` | Persistent sidebar panel. Attach any widget or display element via `st.sidebar.slider(...)`, `st.sidebar.write(...)`, etc. |
| `st.columns(spec)` | Horizontal layout. `spec` is a list of weights, e.g., `[1, 2, 1]`. Returns a list of container objects. |
| `st.tabs(labels)` | Tabbed interface. Returns a list of containers; write into each. |
| `st.expander(label)` | Collapsible section. Returns a container. |
| `st.container()` | Invisible bounded container for grouping. |
| `st.empty()` | Single-element container for dynamic insertion/replacement. |

**Column example:**
```python
col1, col2 = st.columns([1, 2])
with col1:
    st.button("Left")
with col2:
    st.button("Right")
```

## 3. Text & Display Elements

| API | Purpose |
|-----|---------|
| `st.title(body)` | Main app title (H1). |
| `st.header(body)` | Section header (H2). |
| `st.subheader(body)` | Sub-header (H3). |
| `st.markdown(body, unsafe_allow_html=False)` | Render Markdown. |
| `st.caption(body)` | Small muted text. |
| `st.code(body, language="python")` | Formatted code block. |
| `st.latex(body)` | LaTeX math rendering. |
| `st.divider()` | Horizontal rule. |
| `st.error` / `st.warning` / `st.info` / `st.success` / `st.exception` | Colored callout banners. |

## 4. Input Widgets

Every widget returns its current value and triggers a full script rerun on interaction.

| API | Returns | Key Params |
|-----|---------|------------|
| `st.button(label, key=None)` | `bool` | `on_click`, `type` ("primary"/"secondary"), `disabled` |
| `st.checkbox(label, value=False)` | `bool` | `on_change`, `disabled` |
| `st.radio(label, options, index=0)` | Selected option | `horizontal`, `on_change` |
| `st.selectbox(label, options, index=0)` | Selected option | `on_change`, `disabled` |
| `st.multiselect(label, options, default=None)` | `list` | `max_selections`, `on_change` |
| `st.slider(label, min, max, value)` | Number or tuple | `step`, `format` |
| `st.select_slider(label, options)` | Selected option | `value` (default) |
| `st.text_input(label, value="")` | `str` | `max_chars`, `type` ("password"), `autocomplete` |
| `st.number_input(label, min, max, value)` | `int` or `float` | `step`, `format` |
| `st.text_area(label, value="")` | `str` | `height`, `max_chars` |
| `st.date_input(label, value="today")` | `datetime.date` or tuple | `min_value`, `max_value` |
| `st.time_input(label, value="now")` | `datetime.time` | `step` |
| `st.file_uploader(label, type=None)` | `UploadedFile` or list | `accept_multiple_files`, `on_change` |
| `st.color_picker(label, value="#000000")` | `str` (hex) | `disabled` |

**Widget `key` best practice:**
- Always assign a unique `key` when a widget appears inside a loop, conditional, or callback-driven flow. Keys must be strings and unique across the entire app.

## 5. Forms

Group widgets so they trigger only one rerun on submit.

```python
with st.form("my_form"):
    name = st.text_input("Name")
    age = st.number_input("Age", 0, 120, 25)
    submitted = st.form_submit_button("Submit")
    if submitted:
        st.write(f"Hello {name}, age {age}")
```

**Rules:**
- Widgets inside a form cannot have individual `on_change` callbacks.
- `st.form_submit_button` must be called inside the `with st.form(...)` block.
- Only one submit button per form.

## 6. Data Display & Visualization

### Data tables

| API | Purpose | Key Params |
|-----|---------|------------|
| `st.dataframe(data, use_container_width=False)` | Interactive scrollable table. | `column_config`, `hide_index`, `on_select` |
| `st.data_editor(data, num_rows="fixed")` | Editable table (returns modified data). | `column_config`, `disabled`, `on_change` |
| `st.table(data)` | Static full-width table (no interactivity). | — |
| `st.json(body)` | Syntax-highlighted JSON tree. | `expanded` |
| `st.metric(label, value, delta=None)` | KPI card with optional delta indicator. | `delta_color` ("normal"/"inverse"/"off") |

**DataFrame example:**
```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2, 3], "B": [4, 5, 6]})
st.dataframe(df, use_container_width=True, hide_index=True)
```

### Charts

| API | Backend | Notes |
|-----|---------|-------|
| `st.line_chart(data)` | Vega-Lite (simplified) | Quick EDA; accepts DataFrame or array. |
| `st.area_chart(data)` | Vega-Lite | Same as above. |
| `st.bar_chart(data)` | Vega-Lite | Same as above. |
| `st.scatter_chart(data)` | Vega-Lite | Same as above. |
| `st.map(data)` | Mapbox | DataFrame must have `lat`/`lon` or `latitude`/`longitude` columns. |
| `st.altair_chart(chart, use_container_width=True)` | Altair | Full Vega-Lite grammar. |
| `st.plotly_chart(fig, use_container_width=True)` | Plotly | Interactive webGL/JS charts. |
| `st.bokeh_chart(fig)` | Bokeh | Requires `bokeh` package. |
| `st.pydeck_chart(pydeck_obj)` | PyDeck | 3D/Geo layers. |
| `st.graphviz_chart(fig)` | Graphviz | DOT language diagrams. |
| `st.pyplot(fig=None)` | Matplotlib | Clears figure after render unless `clear_figure=False`. |

**Matplotlib best practice:**
```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 5, 6])
st.pyplot(fig)
plt.close(fig)  # Prevent memory leaks in long-running apps
```

## 7. Media, Downloads & Status

### Media

| API | Accepts | Notes |
|-----|---------|-------|
| `st.image(image, caption=None)` | PIL Image, numpy array, file path, URL, bytes | Use `use_container_width` (not deprecated `use_column_width`). |
| `st.audio(data, format="audio/wav")` | File path, bytes, file-like object | `start_time` param available. |
| `st.video(data, format="video/mp4")` | File path, bytes, URL | Supports subtitles via `subtitles` dict. |

### File download

```python
import streamlit as st

csv = df.to_csv(index=False).encode("utf-8")
st.download_button(
    label="Download CSV",
    data=csv,
    file_name="data.csv",
    mime="text/csv"
)
```

**Rules:**
- `st.download_button` requires `data` to be `str`, `bytes`, or file-like.
- For large files, wrap data generation in a cached function.

### Progress & status

| API | Purpose |
|-----|---------|
| `st.progress(value)` | 0–100 progress bar. Update by assigning to the returned object. |
| `st.spinner(text="In progress...")` | Context manager; shows spinner while block executes. |
| `st.status(label, expanded=False)` | Collapsible status container for multi-step workflows. |
| `st.toast(body)` | Brief non-blocking toast notification (auto-dismisses). |
| `st.balloons()` / `st.snow()` | Celebration effects. Use sparingly. |

**Progress example:**
```python
progress_bar = st.progress(0)
for i in range(100):
    time.sleep(0.1)
    progress_bar.progress(i + 1)
```

## 8. Session State

`st.session_state` is a dict-like persistent store that survives reruns.

```python
import streamlit as st

# Initialize safely
if "count" not in st.session_state:
    st.session_state.count = 0

# Read / write
st.write(f"Count: {st.session_state.count}")

if st.button("Increment"):
    st.session_state.count += 1
```

**Key patterns:**

| Pattern | Code |
|---------|------|
| Initialize once | `if "key" not in st.session_state: st.session_state.key = default` |
| Atomic update | `st.session_state.key += 1` |
| Delete | `del st.session_state.key` |
| Callback write | Callbacks can write to session state; widgets cannot read from it directly in the same run. |

**Callback pattern:**
```python
def on_button_click():
    st.session_state.clicked = True
    st.session_state.last_action = "submit"

st.button("Submit", on_click=on_button_click)
```

**Rules:**
- Session state keys are strings. Avoid keys that collide with widget keys unless intentional.
- A widget with `key="foo"` stores its value in session state as `st.session_state.foo`, but only after the first interaction.

## 9. Caching

Cache expensive functions to skip redundant execution across reruns and sessions.

| Decorator | Use case | Invalidation |
|-----------|----------|--------------|
| `@st.cache_data` | Data loading, DataFrame transforms, API calls (returns serializable data). | TTL, `clear()` call, or input change. |
| `@st.cache_resource` | ML models, DB connections, heavy singleton objects (not serializable). | App restart or `clear()` call. |

**Example:**
```python
@st.cache_data(ttl=3600, show_spinner="Loading data...")
def load_data(url):
    return pd.read_csv(url)

df = load_data("https://example.com/data.csv")
```

**Best practices:**
- Always use `ttl` on `@st.cache_data` to prevent stale data.
- Do **not** mutate cached return values in-place; it corrupts the cache. Return a copy if mutation is needed.
- Use `@st.cache_resource` for `transformers` pipelines, `torch` models, SQLAlchemy engines, etc.
- Pass explicit `hash_funcs` for unhashable argument types if needed.

## 10. Multi-Page Apps, Secrets & Deployment

### Multi-page structure

```
my_app/
├── streamlit_app.py      # or Home.py
├── pages/
│   ├── 1_Dashboard.py
│   ├── 2_Analytics.py
│   └── 3_Settings.py
└── .streamlit/
    └── config.toml
```

**Rules:**
- Files in `pages/` are auto-discovered. Prefix with numbers to control sidebar order.
- Each page is a standalone script with its own execution context; share state via `st.session_state`.
- `st.set_page_config` can be called once per page.

### Secrets

Store sensitive values in `.streamlit/secrets.toml` (never commit to Git):

```toml
# .streamlit/secrets.toml
db_password = "super-secret"
[api]
key = "sk-..."
```

Access in code:
```python
password = st.secrets["db_password"]
api_key = st.secrets["api"]["key"]
```

### Configuration

`.streamlit/config.toml`:
```toml
[theme]
primaryColor = "#F63366"
backgroundColor = "#FFFFFF"
secondaryBackgroundColor = "#F0F2F6"
textColor = "#262730"
font = "sans serif"

[runner]
fastReruns = true

[server]
maxUploadSize = 200
```

### Deployment checklist

1. **Requirements:** Pin `requirements.txt` with exact versions.
2. **Secrets:** Verify `secrets.toml` is in `.gitignore`; inject via hosting platform.
3. **State:** Ensure session state initialization handles cold starts.
4. **Cache:** Set appropriate `ttl` values; call `clear()` in admin/debug flows if needed.
5. **Resources:** Close DB connections and file handles properly; use context managers.
6. **Logging:** Use Python `logging` instead of `print`; Streamlit Cloud captures stderr.

## 11. Best Practices Summary

1. **Top-down execution:** Streamlit reruns the entire script on every widget interaction. Keep expensive computations outside the main flow or cache them.
2. **Unique keys:** Duplicate widget keys raise `DuplicateWidgetID`. Use `f"widget_{i}"` patterns in loops.
3. **Minimal sidebar:** Use the sidebar for global filters and navigation, not for primary content.
4. **Progressive disclosure:** Use `st.expander` and `st.tabs` to reduce cognitive load on complex apps.
5. **Chart memory:** Matplotlib figures must be explicitly closed after `st.pyplot` to prevent memory growth.
6. **Container width:** Set `use_container_width=True` on charts and dataframes so they adapt to `layout="wide"` mode.
7. **Lazy loading:** For heavy visualizations, gate them behind `if st.checkbox("Show advanced charts"):` to improve initial load time.
8. **Avoid global mutable state:** Wrap side effects in callbacks or cached functions.
9. **Rerun control:** Use `st.rerun()` to force a script restart after state mutations that should immediately reflect in the UI.
10. **Debugging state:** Add `st.write(st.session_state)` in a sidebar expander during development to inspect live state.
