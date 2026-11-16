# Project status

Intent discovery and the MVP specification are complete for the course-demo MVP. Implementation has not yet started.

`docs/spec.md` defines requirements, record and deletion rules, stage transitions, dashboard calculations, a 90-day history-based funnel, the brand-aligned desktop design, local architecture, sample data and acceptance checks. It uses `docs/intent.md` (the requested `docs/intend.md` does not exist), `docs/TECH_STACK.md` and the brand skill as sources. Technical choices in the specification are proposed implementation decisions, not installed dependencies.

The user interview clarified `docs/intent.md`:

- Prioritize a polished dashboard and interactive pipeline, with realistic sample data for a course demo.
- Document first-time installation of Node, Python and uv; allow internet during setup and require offline use afterward.
- Allow standalone contacts and deals, and preserve and unlink related records on deletion.
- Use stage probabilities of 10%, 30%, 60%, 80%, 100% and 0%; record today's actual close date when marking Won, and allow reopening Won and Lost deals.
- Group monthly won revenue by actual close date, and use stage history for the funnel. The designer will choose and explain the funnel calculation and time window.
- Keep record CRUD, search, sorting, status filters, timelines and follow-ups. Defer activity editing/deletion, creating deals directly in columns, and manual card ordering.

Documentation was reviewed for consistency with the interview answers and brand guidance. No application code was changed or tests run; this work only adds the specification and updates status.
