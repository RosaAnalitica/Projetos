# Power BI redesign

Workspace for the multi-page Power BI dashboard redesign, based on the supplied Safety Daily / Diário de Bordo reference.

## Pages

- `overview/` — "Resumo Principal" page: `index.html` (static preview), `measure.dax` (copyable DAX template), and a `README.md` with setup notes. **Implemented.**
- Add one folder per additional report page as the redesign grows, following the same `index.html` / `measure.dax` / `README.md` structure.

## Implementation notes

- Keep interactive filters as native Power BI slicers.
- Use the HTML Content visual to render the consolidated page layout and charts.
- Build visual values from DAX/model measures so native slicer selections flow through the model's filter context.
- The HTML/DAX artifacts in this folder are design and implementation references; the final measure must be added to the Power BI model.
