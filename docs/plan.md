# MVP implementation plan

## Files that change

Sources: [intent](intent.md), [specification](spec.md), [technical stack](TECH_STACK.md) and the selected [mockup 1](mockups/01-overview.html). This is a proposed file structure, not an implementation. Use mockup 1's layout and brand treatment; replace its session-only behavior and illustrative figures with persisted API data. Application implementation has not started.

| Proposed files | Responsibility |
| --- | --- |
| `package.json`, `package-lock.json` | Root npm workspace, pinned JavaScript dependencies and cross-platform development, test and seed commands. |
| `pyproject.toml`, `uv.lock` | Python 3.12, FastAPI, SQLAlchemy Core, validation, timezone support and pytest dependencies/configuration. |
| `.gitignore` | Exclude local databases, environments, dependency directories, generated builds and test artifacts. |
| `scripts/dev.mjs`, `scripts/test.mjs`, `scripts/seed.mjs` | Coordinate processes, propagate failures, stop child processes and route test/seed commands to the correct database. Avoid shell-specific syntax. |
| `backend/__init__.py`, `backend/app.py`, `backend/config.py`, `backend/clock.py` | One backend package. FastAPI entry/lifecycle, loopback configuration, stable local data path and small clock functions accepting a fixed time in tests. |
| `backend/db.py`, `backend/migrations/` | Keep SQLite connection setup and SQLAlchemy Core table/index definitions together initially; retain versioned schema upgrades and nullable relationships. No ORM or raw SQL. |
| `backend/schemas.py`, `backend/routes.py` | Validated request/response contracts and one initial routes module covering records, activities, completion, stage moves, timelines, dashboard and pipeline. |
| `backend/records.py`, `backend/deals.py`, `backend/activities.py`, `backend/reporting.py` | Focused business functions using SQLAlchemy Core directly: record/deletion rules, deal lifecycle/history, activities/follow-ups and authoritative aggregate calculations. No intermediate services or repositories package. |
| `backend/seed.py` | Deterministic relative-date fixtures, first-database initialization and explicit transactional reset. |
| `backend/tests/conftest.py`, `backend/tests/test_*.py` | Fixed clocks, disposable databases, domain tests and API/database integration tests. |
| `frontend/package.json`, `frontend/index.html`, `frontend/vite.config.ts`, `frontend/tsconfig*.json` | React/TypeScript/Vite workspace, local backend proxy, build and Vitest configuration. |
| `frontend/src/main.tsx`, `frontend/src/App.tsx`, `frontend/src/styles/` | Application entry, five-section routing, selected mockup's sidebar/layout, brand tokens and shared styles. |
| `frontend/public/fonts/` | Bundled Instrument Sans, Instrument Serif and JetBrains Mono, with font source/license attribution. Reuse the available mockup fonts after checking their families and weights. |
| `frontend/src/api/client.ts`, `frontend/src/api/types.ts` | Small typed JSON client, consistent HTTP errors and shared API contracts. Query hooks live with the feature that owns the data. |
| `frontend/src/components/` | Components with actual use across features, such as an accessible dialog, TanStack Table presentation and money/date display. Keep entity forms and feature-specific components in their features. |
| `frontend/src/features/dashboard/` | Dashboard page, local components and query hooks: summary cards, Recharts line/bar charts, open-stage visual, historical reach bars, pending follow-ups and recent activity. |
| `frontend/src/features/pipeline/` | Pipeline page, local components and query/mutation hooks: six-stage dnd-kit board, cards, totals, saving state and accessible stage menu. |
| `frontend/src/features/organizations/`, `frontend/src/features/contacts/`, `frontend/src/features/deals/` | Each feature owns its list/detail pages, entity forms, components and query/mutation hooks. Searchable/sortable tables, status filter, optional links and CRUD stay close to their entity. |
| `frontend/src/features/activities/` | Activity logging form, timeline components and completion hooks reused by contact/deal details and the dashboard where needed. |
| `frontend/src/**/*.test.ts`, `frontend/src/**/*.test.tsx`, `frontend/src/test/setup.ts` | Vitest/Testing Library tests beside the feature or shared component they verify, plus common test setup. |
| `playwright.config.ts`, `e2e/fixtures.ts`, `e2e/*.spec.ts` | Installed Google Chrome, app processes owned by the tests, isolated databases, browser workflows and failure traces/screenshots. |
| `docs/STATUS.md` | Phase progress, actual setup/run/reset instructions, validation results and manual browser review evidence. |

Keep the course `README.md` and the three mockup options unchanged. Changes to the specification are only needed if implementation reveals a real conflict; record any resolution explicitly rather than silently changing product behavior. Do not add deferred MVP features.

### Packaging and extension rules

Start with the modules above and create files when their work begins; do not scaffold empty layers for future ERP, authentication or a database migration. The backend is a single `backend` package, with `app.py` as the entry point; there is no additional nested `app/` package. Keep HTTP validation/response handling in routes and schemas, business operations in the focused modules, and table/connection definitions in `db.py`.

A route calls a business function that uses SQLAlchemy Core directly. Business operations own explicit transaction boundaries: a stage change and its event commit together, as do deletion and unlinking. Pass an existing connection when one operation calls another within the same transaction. Avoid repository classes, base services, generic CRUD engines, dependency containers and wrappers that merely forward library calls. Use ordinary functions for configuration and time; FastAPI's normal dependency mechanism is sufficient for supplying runtime dependencies.

Start with one `routes.py` and table definitions in `db.py`. Split routes by feature when unrelated endpoint groups become difficult to navigate; extract table definitions only when schema growth makes that separation useful. Keep helpers beside their callers. Extract a helper or module when repeated use or a clear independent responsibility warrants it, rather than to meet a file-size rule. The named business modules have real boundaries already: deal transitions, activity handling, record relationships/deletion and reporting.

The frontend uses feature folders without a parallel `pages/` tree. Each feature owns its pages, components, query keys/hooks, mutations and tests. `App.tsx` composes routes and the application shell. Use explicit exports/imports for query keys needed by other features' invalidation; keep optimistic pipeline behavior with the relevant mutation rather than in a generic cache framework. TanStack Query owns server state; React's local state is sufficient for forms and transient UI. Introduce no custom state-management layer.

Share concrete UI behavior when it is used across features: dialog accessibility, table rendering, relationship selection or money formatting. Do not build a universal entity configuration system, schema-driven form engine or speculative design-system package. A feature may start with a page file and a query file; it does not need separate subdirectories for every component, hook and type. Future extension should follow new business capabilities while preserving API contracts, transactional rules and migrations, rather than adding empty layers now.

## Order of work

### Phase 1 — Workspace and isolated local runtime

Create the root npm workspace and uv project, React/Vite frontend and FastAPI backend. Establish typed API conventions, local proxying, loopback binding and a health endpoint. Add the cross-platform launch/test/seed entry points. Select a data path resolved from the repository/configuration rather than the caller's working directory. Test configuration must explicitly supply a separate database path and fail if it targets the ordinary workspace database.

Use the single backend package and feature-based frontend structure above. Create only the files needed for the working shell and test harness in this phase; introduce business modules during their implementation phases.

Introduce the injectable UTC clock and startup business timezone, including timezone data on Windows. Establish versioned schema initialization and a durable initialization marker that distinguishes a new database from an existing empty one. Set up Vitest, pytest and Playwright's Chrome/process configuration before application features depend on them.

**Completion check:** one command starts both services; shutdown cleans them up; test services use disposable storage; a minimal browser smoke test reaches the frontend and API without touching user data. Add initial installation instructions to status.

### Phase 2 — Persistent domain model and sample data

Implement organizations, contacts, deals, activities and immutable stage events with SQLAlchemy Core, initially keeping connection setup and table definitions in `backend/db.py`. Define validation, integer-cent money, nullable links and saved labels/cohort timestamps for retained history. Record an initial stage-entry event on creation. Centralize stage changes in `backend/deals.py` so probability, actual close date and history commit together, including stage changes submitted through record editing. Keep business functions directly responsible for their queries and transactions, without adding a repository or generic CRUD layer.

Implement deletion/unlinking and follow-up completion transactions. Keep expected close dates when reopening; clear actual close dates when leaving Won. A same-stage save keeps a manual probability and adds no event. Preserve history and actionable follow-ups when their live record is deleted.

Seed roughly 15 organizations, 40 contacts, 45 deals and 80 activities. Include all required statuses/stages, six months of wins, 90-day history, skipped stages, reopening, standalone records, empty optional fields, late deals and completed/pending tasks. Seed only during initial database creation; implement the explicit reset separately.

**Completion check:** backend domain tests prove lifecycle/deletion behavior and persistence across connection reopen; repeated startup does not rewrite dates or data; reset restores coherent fixtures. An intentionally emptied database remains empty.

### Phase 3 — Read APIs and reporting

Expose record list/detail and mutation endpoints, combined timelines, follow-ups, pipeline data and dashboard aggregates in `backend/routes.py`. Keep handlers focused on HTTP concerns and calls to business functions; avoid adding pass-through functions or classes. Implement API search/sort/filter semantics, nulls-last sorting and stable tie-breaks. Return link counts for deletion confirmation and field-specific validation errors. Establish typed frontend contracts against the response schemas.

Compute all money and reporting boundaries on the backend. Supply six calendar months, including zero months, and actual-close-based won revenue/counts. Compute open values and per-deal-rounded expected revenue. Build the 90-local-day creation cohort from persisted events, count distinct stage entries without inferring skipped stages, and retain deleted-deal history. Return the reporting timezone/date ranges and latest ten combined events.

**Completion check:** hand-calculated fixture totals match the API, including boundary dates, reopening and deleted records. Lists, details, aggregates and timelines agree after a transaction.

### Phase 4 — Selected design and record workflows

Translate mockup 1 into React with the Midnight sidebar, light canvas, rounded white panels, brand fonts, spacing and stage colors. Add five-section navigation, a shared HTTP client/error format and accessible dialogs. Keep pages, forms and query hooks in their owning feature folders. Extract shared components for actual repeated needs, without a universal entity/form framework. Use the mockup's visual hierarchy rather than copying its hard-coded data or custom chart/drag code.

Implement the three record tables and detail pages with add/edit/delete, case-insensitive search, sortable headings and combined contact status filtering. Add optional searchable relationship selectors, inline validation and explicit deletion consequences. Link related records on detail pages. Add note/call/email logging, newest-first timelines and follow-up completion.

**Completion check:** browser CRUD journeys work with the API and survive reload; no placeholder navigation or session-only data remains. Dialogs support keyboard focus/return, failed saves retain input, and empty/no-result/error states are usable.

### Phase 5 — Dashboard and interactive pipeline

Build the dashboard in mockup 1's order: summary cards; revenue line chart beside won-count bars; open pipeline and historical stage reach; follow-ups and recent activity. Include exact USD tooltips, chart text/table equivalents, current-month labeling, distinct reporting windows and the specified funnel explanation. Group pending tasks as Overdue, Today and Upcoming.

Build the six-column pipeline with consistent stage colors, quiet closed columns, stable newest-created-first ordering and explicit detail links. Wire dnd-kit and the keyboard-accessible stage menu to the same mutation. Optimistically update card state, probability, relevant dates and totals, show saving, block a second move of that card, and roll back on failure with retry. Same-column drops do nothing. Link Add deal to the Deals table.

Invalidate affected details, tables, dashboard and board queries after every mutation; a new activity, completion, stage move, edit or deletion must update all relevant views. Keep the server authoritative after optimistic changes settle.

**Completion check:** stage changes update all affected views and persist after restart. Dashboard figures match known data and adapt to mutations. Real-browser visual review at 1280×800 and 1440×900 matches mockup 1's direction without clipping controls.

### Phase 6 — Release verification and learner instructions

Complete the Proof suite below, including process restart, explicit reset, failed saves and offline operation. Run the entire test command from a clean checkout/setup with no application already running. Inspect every screen/action in real Chrome, capture screenshots and inspect console/page errors. Fix issues found before recording completion.

Finish setup documentation in `docs/STATUS.md` for Mac, Linux and Windows: Node 26 (24+ supported), Python 3.12, uv, dependencies, installed Chrome and any test prerequisites; local URL and data path; development/test/reset commands; and reset's destructive effect. Record which platforms were actually exercised and any unverified platform steps. Verify that after setup neither normal use nor fonts depend on external network requests.

**Completion check:** all automated checks and manual browser journeys pass; actual results and any remaining limitations are recorded. Only then mark the MVP complete.

## Risk

| Risk | Mitigation and evidence |
| --- | --- |
| Packaging accumulates speculative abstractions or scatters related work. | Keep focused backend functions using SQLAlchemy Core directly and feature-owned frontend pages/components/hooks/tests. Start with one routes module and one database module; split for demonstrated complexity. Source review checks for unnecessary forwarding layers, generic entity/form engines and unused helpers. |
| Mockups look finished but omit persistence and full workflows. | Treat them as visual references. Replace session state and sample metrics with API data; keep CRUD, timelines and linked-record deletion in the plan. Prove changes survive refresh and process restart. |
| Stage changes partly commit or produce duplicate history. | One backend transaction/service for board, detail and form changes. Serialize a pending card move; treat unchanged stage as a no-op. Test rollback and history counts, including won/reopen cycles. |
| Deletion erases the funnel or strands follow-ups. | Separate retained historical identity/name/cohort fields from nullable live links. Confirm affected counts and consequences before deletion. Test retained events/tasks and cleared relationships. |
| Dashboard and board figures disagree or suffer rounding errors. | Store cents and perform per-deal expected-value rounding centrally using exact arithmetic. Specify nearest-cent half-up handling for ties in the calculation contract. Compare independent fixture totals across API and UI. |
| UTC/local dates shift wins, overdue status or cohorts. | Capture an actual local timezone at startup, not just a fixed UTC offset; provide timezone data on Windows. Store UTC timestamps separately from calendar dates and use one injected clock. Test midnight, month/year changes, daylight-saving transitions and the inclusive 90-day window. |
| A new initialization/reset path overwrites existing work. | Persist schema/initialization state independent of row counts; reset only through the documented explicit command. Exercise deleted-all-records, repeated startup and reset scenarios in temporary databases. |
| Optimistic cache changes leave stale totals or lose user input. | Define shared query keys/invalidation, snapshot affected state for rollback and reconcile from the server. Retain form values on failure and provide retry. Simulate server/network failures in browser tests. |
| Tests touch real data or require an already running app. | Require dedicated test database paths, allocate isolated run/worker databases and let Playwright start/stop its services. Do not reuse a normal development server. Check a sentinel user database before/after the suite. |
| One-command setup differs across operating systems. | Use Node process orchestration with explicit arguments and portable paths; avoid Unix-only commands. Document installed prerequisites, clear startup failures and platform validation evidence. |
| Styling, inaccessible drag controls or external assets weaken the chosen design. | Reuse local brand fonts, mature libraries and the selected composition. Provide a stage menu, chart equivalents and visible keyboard focus; inspect both target sizes and block external requests during verification. |

## Proof

These are planned checks, not results. `npm test` must fail if any frontend, backend or browser test fails. Use installed Google Chrome for Playwright Test, fixed clocks for date-sensitive fixtures and isolated temporary databases. Also run the frontend TypeScript check and production build before completion. Domain assertions belong primarily on the backend; frontend tests verify presentation and interaction rather than reimplementing reporting rules.

### Requirement traceability

Each coverage group below links a source requirement to implementation phases and planned evidence. IDs are stable references for implementation reviews and test descriptions. Every row currently has status **Planned — not implemented or verified**. Listing a test is coverage planning, not proof that the requirement passes. Record actual results against these IDs in `docs/STATUS.md` during implementation; do not mark a group verified until every clause in its row has evidence.

| ID | Source and required behavior | Phase / implementation owner | Planned evidence |
| --- | --- | --- | --- |
| R01 | [Scope](spec.md#purpose-and-scope): local solo CRM, five sections, Dashboard landing; no deferred features. | 1, 4–6 / app shell, routing, scope review | `startup.spec.ts`; manual scope review, including no extra integrations/accounts/settings. |
| R02 | [Records](spec.md#records-and-relationships): all organization/contact/deal/activity fields, stable IDs, creation/update timestamps, required trimming, optional nulls, valid enums/email/website/dates. | 2–4 / schema, validation, forms | `test_validation.py`, `test_records_api.py`; form component tests and `records.spec.ts`. |
| R03 | [Records](spec.md#records-and-relationships): lead/New/zero-value defaults; optional organization/main contact; no separate leads entity; independent relationships and manual contact statuses. | 2, 4 / records service and forms | `test_records_api.py`; `records.spec.ts` creates standalone and differently linked records and confirms winning a deal does not change contact status. |
| R04 | [Money](spec.md#dashboard-and-pipeline-calculations): integer cents, nonnegative two-decimal USD values, integer probabilities 0–100, per-deal expected-value rounding, explicit USD everywhere. | 2–5 / validation, reporting, display components | `test_money.py`, `test_validation.py`; money component tests; `dashboard.spec.ts`; manual tooltips/labels review. |
| R05 | [Tables](spec.md#tables-and-record-pages): exact columns for each entity, add/edit/delete per row, name links, row actions do not navigate. | 4 / record tables | Table component tests; `records.spec.ts` inspects column sets and row actions. |
| R06 | [Tables](spec.md#tables-and-record-pages): case-insensitive specified-field search including linked names, excluding notes/activity descriptions; contact status combines with search. | 3–4 / list API, table controls | `test_records_api.py`; table tests and `records.spec.ts`, including excluded-field matches. |
| R07 | [Tables](spec.md#tables-and-record-pages): ascending/descending heading sorts, indicated direction, nulls last, ID tie-break, default updated-first, displayed-date sorting. | 3–4 / list API and TanStack Table | `test_records_api.py`; table component tests; `records.spec.ts` covers numeric counts/value/probability and expected/actual date sorting. |
| R08 | [Forms](spec.md#tables-and-record-pages): add/edit dialogs, inline errors, cancel/save, searchable optional links, retain input on failure, close only after persistence; creation opens details. | 4 / shared dialogs and forms | Form component tests; `records.spec.ts`, `failures.spec.ts`, `accessibility.spec.ts`. |
| R09 | [Detail pages](spec.md#tables-and-record-pages): organization contacts/deals; contact organization/main-contact deals/timeline; deal organization/contact/stage/timeline; linked navigation. | 3–4 / detail API and pages | `test_records_api.py`, `test_timelines.py`; `records.spec.ts`. |
| R10 | [Deletion](spec.md#deletion): name/counts/consequences in confirmation; Cancel is harmless; related records survive and their appropriate live links clear; no blocking on existing links. | 2–4 / deletion transactions and dialog | `test_deletion.py`; dialog tests; `deletion.spec.ts`. |
| R11 | [Deletion](spec.md#deletion): retained labels, stage events and actionable follow-ups after deletion; no dead links; current metrics exclude deleted deals, historical funnel includes them; organization deletion leaves activities alone. | 2–5 / retained history, reporting, timelines/tasks | `test_deletion.py`, `test_followups.py`, `test_reporting.py`; `deletion.spec.ts` and `dashboard.spec.ts`. |
| R12 | [Lifecycle](spec.md#deal-lifecycle): six exact stages/default probabilities, manual override until a real stage change; unchanged stage adds no event; stage-changing form disables override and explains reset. | 2, 4–5 / shared deal service and forms | `test_deal_lifecycle.py`; form/mutation tests; `pipeline.spec.ts` tests board, detail and edit-form paths. |
| R13 | [Lifecycle](spec.md#deal-lifecycle): creation records initial entry; every transition atomically stores old/new stage and timestamp, appears once in timeline/recent activity and supports history. | 2–5 / deal service and stage events | `test_deal_lifecycle.py`, `test_storage.py`, `test_timelines.py`; `pipeline.spec.ts`. |
| R14 | [Close dates](spec.md#deal-lifecycle): expected and actual are separate; Won sets local today, leaving Won clears actual, re-win sets new date; Lost has no actual date; reopening retains expected date. | 2, 4–5 / deal service and date components | `test_deal_lifecycle.py`, `test_dates.py`; `pipeline.spec.ts` and `persistence.spec.ts`. |
| R15 | [Overdue](spec.md#deal-lifecycle): open and expected before today only; no-date/today/closed exclusions; visible textual and Raspberry indicators in table and card. | 2–5 / clock, presentation | `test_dates.py`; date/status component tests; `pipeline.spec.ts`; manual color/text check. |
| R16 | [Activities](spec.md#activities-and-follow-ups): note/call/email with description, happened-at default now, optional follow-up and done=false; exactly one target on creation, no duplicated linked-record entries; activity edit/delete deferred. | 2–4 / activity service/form/timeline | `test_validation.py`, `test_timelines.py`; activity component tests and `activities.spec.ts`. |
| R17 | [Follow-ups](spec.md#activities-and-follow-ups): all pending dated activities, three date groups, due/creation ordering, target link or deleted label; completion persists and remains in timeline; no task reopening. | 2–5 / activity service and dashboard | `test_followups.py`; activity tests; `activities.spec.ts`, `deletion.spec.ts`, `persistence.spec.ts`. |
| R18 | [Clock](spec.md#dashboard-and-pipeline-calculations): startup business timezone, UTC timestamps/ISO dates, shared local boundaries returned by backend. | 1–3 / clock and response contracts | `test_dates.py`, API contract checks; month/year/midnight/daylight-saving fixtures. |
| R19 | [Dashboard](spec.md#dashboard): four correct cards; full won value/count by actual close month; six months including zeros/current partial month, line/bar charts and exact tooltips; open-only value/expected totals. | 3, 5 / reporting and charts | `test_reporting.py`; dashboard component tests; `dashboard.spec.ts`, including a manually overridden Won probability that must not reduce won revenue. |
| R20 | [Open-stage visual](spec.md#dashboard): four ordered colored segments, counts/value/expected figures, zero stages and all-empty handling, consistent with board. | 3, 5 / pipeline aggregates and dashboard | `test_reporting.py`; dashboard tests; `dashboard.spec.ts` compares all stage totals. |
| R21 | [Historical funnel](spec.md#dashboard): inclusive 90-local-day creation cohort, persisted entries only, distinct deal per stage, initial/skipped/reopened/deleted behavior; six labeled stages with Lost separate, aligned reach bars and exact explanation/range. | 2–3, 5 / history and reporting | `test_dates.py`, `test_reporting.py`; dashboard component tests; `dashboard.spec.ts`; manual explanation/chart check. |
| R22 | [Recent activity](spec.md#dashboard): latest ten combined user/stage events, newest-first stable ordering, timestamp/target/description; contact/deal timelines also stable newest-first. | 3–5 / timeline/reporting APIs and components | `test_timelines.py`, `test_reporting.py`; `activities.spec.ts` and `pipeline.spec.ts`. |
| R23 | [Refresh behavior](spec.md#dashboard): every mutation refreshes all affected detail/list/dashboard/board figures without reload. | 4–5 / query keys and invalidation | Mutation component tests; `dashboard.spec.ts`, `records.spec.ts`, `deletion.spec.ts`, `activities.spec.ts`. |
| R24 | [Pipeline](spec.md#pipeline): six ordered columns, quiet closed stages, column counts/value/expected totals, open-only header totals; card identity/organization/value/probability/labeled dates; explicit detail link and newest-created-first ordering. | 3, 5 / board API/cards | `test_reporting.py`, list/API ordering checks; `pipeline.spec.ts`; manual design review. |
| R25 | [Pipeline moves](spec.md#pipeline): drag and accessible stage menu, immediate totals/state, saving lock, same-column no-op, atomic persistence, rollback/error/retry; no stored manual ordering or column creation; Add goes to Deals. | 2, 5 / deal service and board mutations | `test_storage.py`; mutation tests; `pipeline.spec.ts`, `failures.spec.ts`, `accessibility.spec.ts`. |
| R26 | [Visual design](spec.md#visual-and-interaction-design) and selected mockup 1: Midnight sidebar, light canvas, rounded white panels, fonts/spacing, chart/stage colors, plain business labels, restrained styling and desktop layouts. | 4–6 / styles and all screens | Manual screenshot comparison to mockup 1 and brand skill; `accessibility.spec.ts`; local-font check in `offline.spec.ts`. |
| R27 | [Accessibility/states](spec.md#visual-and-interaction-design): labels/errors, focus trap/return, keyboard focus/navigation, semantic sort state, contrast, reduced motion, chart equivalents and non-color status; useful loading/empty/no-match/retry states. | 4–6 / shared components and pages | Component tests; `accessibility.spec.ts`, `failures.spec.ts`; manual contrast, reduced-motion and chart review. |
| R28 | [Architecture](spec.md#technical-design) and [stack](TECH_STACK.md): Node/React/TypeScript/Vite workspace; Python 3.12/uv/FastAPI JSON API; SQLite/SQLAlchemy Core, no raw SQL; mature table/query/chart/drag libraries; typed client. Apply this plan's lean packaging rules. | 1–5 / workspace, backend, frontend | Dependency/configuration and source review, including direct business-function transactions, feature ownership and absence of speculative layers; TypeScript/build checks; API integration tests and `startup.spec.ts`. |
| R29 | [Storage/API](spec.md#technical-design): versioned migrations, required indexes/nullable integrity, saved historical identity/cohort, atomic writes, useful errors, no premature success and loopback-only service. | 1–4 / schema/services/routes | `test_storage.py`, `test_deletion.py`, `test_records_api.py`; failure component/browser tests; bind configuration/source review. |
| R30 | [Persistence/reset](spec.md#technical-design): seed only new database, preserve refresh/restart edits and intentionally empty data; stable configured path; one explicit transactional reset. | 1–2, 6 / configuration, initialization, seed | `test_storage.py`, `test_seed.py`; `persistence.spec.ts`; launch from another working directory and verify the same data path. |
| R31 | [Sample data](spec.md#sample-data-and-setup): approximate counts, five industries, all statuses/stages, standalone/optional fields, varied probabilities, past wins/losses, future/late deals, relative deterministic dates, coherent histories and all task groups. | 2, 6 / seed | `test_seed.py`, `startup.spec.ts`; manual first-launch review of every screen. |
| R32 | [Commands/tests](TECH_STACK.md) and [technical design](spec.md#technical-design): portable root dev/test/seed scripts, clean shutdown, failure propagation, Vitest/pytest/Playwright installed Chrome, self-starting isolated browser tests and no real-data changes. | 1, 6 / scripts and test configuration | Test-runner/process smoke checks; sentinel-database check; complete `npm test` with no app running; platform setup evidence. |
| R33 | [Setup/offline](spec.md#sample-data-and-setup): Mac/Linux/Windows installation of Node/Python/uv/dependencies and Chrome prerequisites; documented URL/data path/commands/reset effect in STATUS, course README unchanged; offline after setup. | 1, 6 / scripts, assets, documentation | Setup checklist and changed-file review; `offline.spec.ts`; record actual platform validation rather than assuming parity. |
| R34 | [Acceptance](spec.md#acceptance-and-verification): all workflows, real-browser visual review and console checks beyond automated tests; record progress/results and do not declare completion early. | 1–6 / acceptance review and status | All suites plus manual acceptance evidence below; every R01–R34 row requires recorded evidence before MVP completion. |

Intent success criteria map to these groups: **1 → R01, R26, R28, R30–R33; 2 → R05–R08, R10, R30; 3 → R03, R06, R09–R11; 4 → R12–R15, R23–R25, R30; 5 → R13, R16–R17, R22; 6 → R04, R18–R23, R31; 7 → R19–R21, R26–R27; 8 → R34.** The specification's more detailed requirements are covered by the remaining clauses in those same rows.

Audit findings:

- The implementation plan covers the intent's eight success criteria. The review added explicit proof for column sets/date sorting, independent links/contact status, excluded search fields, full won revenue despite a manual probability, and stable storage paths. These details must appear in the tests named above, not be assumed from generic CRUD coverage.
- The stack's SQLAlchemy Core/no-raw-SQL and Python 3.12 requirements are more specific than the specification's general SQLAlchemy/Python wording. Follow the stack; there is no need to introduce an ORM.
- “Exactly one contact or deal” applies when creating an activity. The specification's explicit deletion rule permits a retained activity with no live target afterward. Validation and database tests must distinguish these cases.
- The six-month chart window, 90-day stage-reach cohort and retained-deletion history are specification design decisions authorized by the intent's deferred funnel choice; they are not facts established by the mockup's sample numbers. Backend fixtures must prove them independently.
- Half-up handling of exact rounding ties is a proposed technical refinement in this plan, since the specification says nearest cent without defining ties. Record that convention in the implementation contract and test it; do not describe it as an original intent requirement.

No blocking contradiction was identified in this document review. This does not establish implementation correctness, browser quality or platform compatibility; those remain unverified until the planned evidence exists.

### Backend unit tests — pytest

Test filenames below identify coverage responsibilities. Related cases can share a test module while they remain easy to navigate; the list does not require a matching production module, helper or abstraction for each test group. Frontend tests stay beside their owning feature/component. Keep all R01–R34 coverage as packaging evolves.

- `test_validation.py`: required trimmed strings; optional blanks/nulls; valid contact statuses, activity types and stages; invalid email/website/calendar dates; nonnegative cents, excess decimal precision and probability bounds. Reject dangling relationships and activity creation without exactly one valid contact/deal target.
- `test_money.py`: USD conversion, zero values, manual probabilities, nearest-cent ties and per-deal rounding before summation. Use fixtures where rounding each deal differs from rounding only the total.
- `test_deal_lifecycle.py`: all six defaults; creation in an open/closed stage; initial entry; manual override retention; same-stage no-op; default reset on any actual stage change; stage-changing form behavior; entering Won sets local today; leaving Won clears actual date; reopening Won/Lost preserves expected date; re-winning records the new local date. Each real transition yields exactly one event.
- `test_dates.py`: overdue before today only; no-date/today/closed exclusions; local-vs-UTC midnight; six-month range across a year boundary; current partial month; daylight-saving boundaries; cohort days 1 and 90 included and day 91 excluded; future event exclusion.
- `test_reporting.py`: full won values grouped by actual close month, zero months, won counts, closed-stage exclusion from open totals, stage counts/values/expected totals, recent-event ordering and ten-entry limit. Reopened/deleted deals leave current won totals. Funnel counts persisted stage entries, initial direct-to-stage creation, skips and repeated/reopened transitions once per deal/stage; deleted history remains.
- `test_followups.py`: tasks grouped Overdue/Today/Upcoming and sorted by due date/creation time; completed and undated activities excluded; completion remains visible in the timeline; deleted targets keep tasks with retained labels.

### Backend API and database integration tests — pytest

- `test_records_api.py`: CRUD for all three record types; list search across specified fields and linked names; notes excluded from search; status/search combinations; ascending/descending sorts, nulls-last and stable tie-breaks; missing records and invalid inputs return useful errors. Organization and main-contact links are independently optional.
- `test_deletion.py`: correct affected counts, organization unlinking, contact main-link clearing, deal deletion preserving related records, retained activities/stage history, no orphan live foreign keys, and transactional rollback on an injected failure.
- `test_timelines.py`: note/call/email stay on their exact target without duplication; stage events appear only on the deal timeline and recent activity; newest-first stable ordering and saved deletion labels.
- `test_storage.py`: migrations on new/existing databases; initialized empty databases do not reseed; mutations/completion/history survive closing and reopening the database; failed stage writes leave all related rows unchanged.
- `test_seed.py`: deterministic contents for a fixed date, approximate required record counts, all stages/statuses, six months of wins, populated funnel, standalone records, varied probabilities, late/future dates and all follow-up groups. Events are chronological and end at each current stage. Explicit reset restores samples without leaving partial data.

### Frontend unit/component tests — Vitest

- Money/date/status components show USD, labeled expected/actual dates and overdue text, including absent dates and zero values.
- Table controls combine search/status filters, expose sortable headings and direction, preserve stable record links, and distinguish empty data from no matches. Row actions do not open details.
- Forms allow standalone records and searchable optional links, surface field errors, disable duplicate saves, preserve values on failure and close only after success. A stage change explains/resets probability; an unchanged stage retains the override.
- Deletion dialogs name the record, show affected counts/history consequences and perform no mutation on Cancel. Dialog opening/closing and keyboard traversal manage focus correctly.
- Activity forms validate the required description/type and optional follow-up; completion updates pending tasks while retaining timeline content.
- Query/mutation handling refreshes affected views. Pipeline pending state blocks repeat moves, same-stage moves make no request, and a rejected move restores card values/totals with retry available.
- Dashboard components show backend-supplied ranges, zero months, historical explanation, all stage labels, accessible chart equivalents and loading/empty/error states. Avoid tests tied to Recharts' internal SVG markup or dnd-kit internals.

### End-to-end tests — Playwright Test in Google Chrome

| Proposed test | Journey and evidence |
| --- | --- |
| `e2e/startup.spec.ts` | Start both services automatically on a fresh isolated database; land on the populated dashboard; navigate all five sections; check no console/page errors or unexpected failed API requests. |
| `e2e/records.spec.ts` | Add, search, sort, edit and delete each record type; combine contact status/search; create standalone records; traverse linked detail pages; check form validation and cancel/save behavior. |
| `e2e/deletion.spec.ts` | Cancel then confirm linked organization/contact/deal deletion; verify confirmation consequences, surviving records, cleared links, retained historical activity and still-actionable follow-ups. |
| `e2e/pipeline.spec.ts` | Drag between columns and move through the stage menu; compare header/column totals and probability in board, table and detail; same-column drop creates no event. Mark Won/Lost, reopen and re-win; verify actual/expected dates, overdue text and stage history. |
| `e2e/activities.spec.ts` | Log note, call and email from both target types; verify correct newest-first timelines and recent activity; add tasks in all date groups; complete a task and confirm persisted removal from pending tasks. |
| `e2e/dashboard.spec.ts` | Load a small known fixture and assert summary totals, six-month data/table equivalents, zero months, open-stage values and historical reach counts. Change a deal/value/probability, mark Won and add/complete a follow-up; verify affected figures update without reload. |
| `e2e/failures.spec.ts` | Fail a stage request and form save deliberately; confirm rollback/retry, retained input and no false success. Verify pending card cannot be moved twice. Exercise loading/error/retry and empty/no-results states. |
| `e2e/persistence.spec.ts` | Mutate records, links, stages, activities and completion; reload, stop both services and restart against the same test database; verify all changes. Delete all records and restart without reseeding; invoke the reset command against that test database and verify sample restoration. |
| `e2e/accessibility.spec.ts` | Complete dialog/form and stage-menu journeys by keyboard; inspect focus return, visible labels/sort state, textual status and chart equivalents. Check 1280×800 and 1440×900 without page-level control clipping; horizontal board/table scrolling remains usable. |
| `e2e/offline.spec.ts` | Keep loopback available while rejecting external network requests, reload and navigate all sections, perform a persisted mutation and verify locally served fonts/charts. Do not use a browser-wide offline switch that would also block the local API. |

Keep fixture/reset helpers confined to test configuration; no production endpoint should reset user data for browser tests. Capture traces and screenshots on failures, and retain baseline screenshots for reviewing the selected design. Verify the normal workspace database is unchanged by the suite.

### Manual acceptance evidence

Use real Chrome to perform every record action, relationship/deletion case, activity/follow-up action and pipeline move described above, including one refresh and full app restart. Inspect all five sections, detail pages, dialogs, chart tooltips, historical explanation and failure/empty states at both desktop sizes. Compare dashboard and pipeline screenshots with mockup 1, check contrast and keyboard controls, and inspect the browser console. Record screenshots, commands, actual pass/fail results and platform coverage in `docs/STATUS.md`. Automated green tests alone do not establish completion.
