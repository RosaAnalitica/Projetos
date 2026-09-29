# Resumo Principal (overview page)

First page of the multi-page Power BI redesign for the Diário de Bordo /
Safety Daily report. This folder is the template for how every future page
folder under `power-bi-redesign/` should be structured:

```
overview/
├── index.html   # static HTML/CSS/SVG preview of the page layout
├── measure.dax  # copyable DAX measure template that renders the same layout
└── README.md    # this file
```

## What's illustrative vs. what's real

- **`index.html`** contains only **sample/illustrative numbers** (KPI %, bar
  values, the temporal line). It exists purely to preview the visual layout
  and CSS in a browser. None of its numbers come from the Power BI model or
  from the Performance Analyzer trace — the trace only records query row
  counts/timings, not chart values, so no "real" figures were available to
  reproduce here, and none were invented.
- **`measure.dax`** contains the actual logic that must run inside the Power
  BI model. It has **no hard-coded numbers** — every value comes from
  `CALCULATE`/`ADDCOLUMNS`/`CONCATENATEX` over model tables and measures, so
  its output reflects live data and current slicer selections. It reuses the
  same CSS class names/structure as `index.html`, so once wired up in the
  model the two should look the same.

Do not copy numbers from `index.html` into any report, and do not present
`measure.dax`'s comments/placeholders as confirmed business results.

## Fields and measures used (from the Performance Analyzer trace)

| Model object | Used for |
|---|---|
| `D_Concessao[Concessão]` | "Rotinas respondidas por Concessão" chart, and native slicer |
| `D_Cargo[cargo]` | "Rotinas respondidas por Cargo" chart, and native slicer |
| `D_Rotinas[Rotinas]` | "Distribuição por tipo de Rotina" chart |
| `D_Diarios[Diarios]` | "Distribuição por tipo de Diário" chart |
| `D_Periodos[Período]` | Native slicer |
| `F_Diario_De_Bordo[DiaSemana]` | Temporal chart axis (placeholder — see below) |
| `F_Diario_De_Bordo[Ano]` | Native slicer (year) |
| `_Measures[_Rotinas_Respondidas]` | KPI card and all bar-chart values |
| `Aux_Quantidades_Engenharia[_Rotinas_Respondidas_Geral]` | KPI card secondary figure |

These field/measure names are confirmed to exist in the model (they appear
in the trace), but the trace does **not** reveal their exact business
semantics (e.g. whether `_Rotinas_Respondidas` is already a percentage, or a
raw count that needs a denominator). Confirm this in the model before
shipping the measure.

### Known placeholders to resolve before using `measure.dax`

1. **Temporal chart axis.** The trace exposes `F_Diario_De_Bordo[DiaSemana]`
   (weekday name) and `F_Diario_De_Bordo[Ano]` (year), but neither is a
   proper chronological month/date label for a trend chart. Replace the
   `VAR TabelaTemporal` source in `measure.dax` with your actual calendar
   table's date/month column (and make sure it has a numeric sort-by column
   so points are ordered correctly).
2. **KPI semantics.** Confirm whether `_Measures[_Rotinas_Respondidas]` is a
   ratio (0–1) formatted as `%`, or a count that should be divided by an
   "expected routines" measure.
3. **`Aux_Quantidades_Engenharia[_Rotinas_Respondidas_Geral]` measure name.**
   The trace shows the table/column pair; confirm the actual measure name
   that wraps it (a raw column reference cannot be used directly inside
   `CONCATENATEX`/`FORMAT` the way a measure can). Despite the
   "Aux"/"Engenharia" (engineering) naming, the trace associates this
   column with the same routines-answered KPI as `_Measures[_Rotinas_Respondidas]`
   — it is likely a helper/auxiliary table feeding that KPI (e.g. a
   pre-aggregated or engineering-calculated variant), not an unrelated
   metric. Confirm this relationship in the model before relying on it.

## How to wire this up in Power BI

1. Open the report in Power BI Desktop and go to the target page.
2. Add the measure from `measure.dax` to an appropriate table in your model
   (e.g. `_Measures`). Resolve the placeholders listed above first.
3. Add the **HTML 5 Content** custom visual to the page and bind the new
   measure (e.g. `Resumo_Principal_HTML`) as its data field.
4. Add **native Power BI slicers** to the canvas (not inside the HTML
   visual) for `D_Concessao[Concessão]`, `D_Cargo[cargo]`,
   `D_Periodos[Período]`, and `F_Diario_De_Bordo[Ano]`. Leave their
   interaction with the HTML visual set to "Filter" (default) so
   selections flow into the measure's filter context automatically.
5. Open `index.html` in a browser to compare the intended layout/spacing
   against the live HTML visual, and adjust the shared CSS block (kept
   identical between the two files) if you need to tweak styling — update
   both files together to keep them in sync.

## Adding the next page

Create a new sibling folder (e.g. `power-bi-redesign/detalhamento/`) with
the same three files, and add a nav pill for it in this page's
`.page-nav` element (see `index.html`) once the new page exists, so the
compact page navigation stays consistent across pages.
