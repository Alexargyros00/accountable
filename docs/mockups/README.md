# Dashboard and pipeline design options

Open any HTML file directly in a desktop browser. Use Dashboard and Pipeline in the sidebar to switch views; `#pipeline` opens the board directly. No server, package installation or internet connection is needed. Keep the shared CSS, JavaScript and fonts alongside the HTML files.

| Direction | HTML | Dashboard screenshot | Pipeline screenshot |
| --- | --- | --- | --- |
| Overview: spacious panels and balanced reporting | [01-overview.html](01-overview.html) | [Dashboard](01-overview-dashboard.png) | [Pipeline](01-overview-pipeline.png) |
| Sales desk: compact grouped surfaces, green summary emphasis and dense cards | [02-sales-desk.html](02-sales-desk.html) | [Dashboard](02-sales-desk-dashboard.png) | [Pipeline](02-sales-desk-pipeline.png) |
| Workspace: larger charts, a serif accent and open board columns | [03-workspace.html](03-workspace.html) | [Dashboard](03-workspace-dashboard.png) | [Pipeline](03-workspace-pipeline.png) |

All three follow `../spec.md` and the brand skill: Midnight navigation, the light canvas, Instrument Sans, Instrument Serif accents, JetBrains Mono numbers and the shared stage palette. The fixed sample date is 16 November 2026. Initial open value is USD 188,000.00 and expected revenue is USD 98,800.00.

The prototypes support dragging deals between stages, keyboard-accessible stage selectors, immediate total/probability changes, Won and reopening behavior, recent stage events, follow-up completion, chart data disclosures and preview dialogs. Changes last for the current page session and reset on reload. Supporting record screens and backend persistence are outside these visual mockups. Historical funnel counts and older monthly figures are illustrative sample history rather than a database implementation.

Fonts are bundled locally from Google Fonts: Instrument Sans, Instrument Serif and JetBrains Mono. Charts are local SVG previews; the application specification calls for Recharts and dnd-kit during implementation.

## Verification

Screenshots were captured with Playwright at a 1440 × 900 viewport using full-page capture. Both views were visually reviewed for all three options. Playwright also checked 1280 × 800 layouts, six board columns, stage changes, updated expected revenue, updated won revenue and follow-up completion. No JavaScript or browser console errors were observed. Narrower desktop boards scroll internally.
