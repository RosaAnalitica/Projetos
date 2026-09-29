# Power BI redesign

Workspace for the multi-page Power BI dashboard redesign, based on the supplied Safety Daily / Diário de Bordo reference.

## Planned pages

- `overview/` — consolidated overview page
- Add one folder per additional report page as the redesign grows.

## Implementation notes

- Keep interactive filters as native Power BI slicers.
- Use the HTML Content visual to render the consolidated page layout and charts.
- Build visual values from DAX/model measures so native slicer selections flow through the model's filter context.
- The HTML/DAX artifacts in this folder are design and implementation references; the final measure must be added to the Power BI model.
