# Accountable MVP specification

## Purpose and scope

Accountable is a local sales CRM for one person exploring a course demo. It brings companies, people, deals and follow-ups into one place. The dashboard and interactive pipeline are the visual priorities; the supporting screens must provide complete everyday workflows.

This specification implements [the intent](intent.md), follows [the technical stack](TECH_STACK.md), and applies `.agents/skills/brand/SKILL.md`. Calculation and interaction choices below resolve decisions left open by the intent. Implementation has not started.

The MVP runs on a laptop or desktop in light mode. It includes five sections: Dashboard, Organizations, Contacts, Deals and Pipeline. It excludes authentication, multiple users, hosting, mobile layouts, dark mode, AI, integrations, additional currencies, custom stages, tags, custom fields, import/export and reports outside the dashboard. Activity editing/deletion, manual pipeline ordering and creating deals inside a column are deferred.

## Product requirements

### Records and relationships

All records have stable IDs and creation/update timestamps. Required strings are trimmed and cannot be blank. Optional blank fields are stored as null. Money is stored as integer US cents, never floating-point dollars; accept nonnegative amounts with at most two decimal places. Probability is an integer percentage from 0 through 100. Dates must be valid calendar dates. Email and website fields, when supplied, must have valid formats.

| Record | Fields | Relationships |
| --- | --- | --- |
| Organization | Required name; optional website, industry, notes | Many contacts and deals |
| Contact | Required name and status; optional email, phone, job title | Optional organization; may be a deal's main contact |
| Deal | Required name, stage, value, probability; optional expected close date and derived actual close date | Optional organization and main contact |
| Activity | Required type, description, happened-at timestamp; optional follow-up date; done flag defaults false | Exactly one contact or deal |

Contact statuses are `lead`, `qualified` and `customer`; a new contact defaults to lead. A deal defaults to New and USD 0.00. A main contact need not belong to the deal's organization; changing either relationship never silently changes the other. Contact status changes are manual, including after a deal is won.

Maintain separate expected and actual close dates. Expected dates can remain empty, including when reopening a deal. Actual close date is system-managed and applies only while the deal is Won. The detail screen exposes the expected date and, for Won deals, the actual date. Lists and cards label the displayed date clearly.

### Tables and record pages

Each organization, contact and deal table provides:

- A labeled search input, sortable data column headings, an Add action, and Edit/Delete actions on every row.
- Case-insensitive substring search across the displayed text fields, including linked record names; search combines with the contact status filter.
- Ascending/descending sorting with an indicated direction, nulls last and ID as a stable tie-breaker. Default order is most recently updated first.
- A record name link to its detail page. Row actions must not trigger navigation.
- A useful empty state, a separate no-results state with Clear filters, and visible loading/error states with retry.

Organization columns: name, website, industry, contact count and deal count. Contact columns: name, status, organization, email, phone and job title. Deal columns: name, organization, main contact, stage, value, probability and close date. Deal dates are labeled Expected or Actual; sorting uses the displayed date. Search covers names, organization/contact names, industry, website, email, phone and job title where applicable, plus deal stage. Notes and activity descriptions are not table search fields.

Add and Edit open a focused form dialog with inline validation, Cancel and Save. Save closes only after successful persistence. Failed saves retain entered values. Relationship selectors allow no selection and search by record name. Successful creation opens the detail page; successful edits refresh visible values.

Organization details show its fields and linked contacts/deals with links to their pages. Contact details show its fields, organization, deals for which it is the main contact, and activity timeline. Deal details show its fields, linked organization/main contact, stage selector and timeline. Contact and deal pages provide Log activity. Timelines are newest first with timestamp and stable tie-break ordering.

### Deletion

Every Delete action opens a confirmation naming the record, showing affected link counts and explaining that related records are kept. Cancel changes nothing. Deleting an organization clears its contacts' and deals' organization links. Deleting a contact clears main-contact links on deals. Deleting a deal does not delete its organization or contact.

Activities attached to a deleted contact or deal remain stored with a null target and a saved target-name label. Their follow-ups remain actionable on the dashboard, with “Contact deleted” or “Deal deleted” in place of a link. Stage events for deleted deals remain as historical records with a saved deal name; they appear in recent activity and the historical funnel. Current revenue and pipeline metrics exclude deleted deals. The funnel help text explains this historical-retention distinction. Deletion confirmation explicitly mentions retained history and follow-ups. Organization deletion does not affect activities attached to its contacts or deals.

### Deal lifecycle

| Stage | Default probability | Open? |
| --- | --- | --- |
| New | 10% | Yes |
| Qualified | 30% | Yes |
| Proposal | 60% | Yes |
| Negotiation | 80% | Yes |
| Won | 100% | No |
| Lost | 0% | No |

A manual probability override persists until a stage change. Moving to another stage always applies that stage's default, even when editing through a form. A form must make that reset clear and disable probability overrides during that stage-changing save; the user can edit probability afterward. Saving unchanged stage produces no stage event and does not reset probability.

Creating a deal records an initial stage-entry event, including when it starts Won or Lost. Every subsequent change records previous stage, new stage and timestamp, and automatically appears in the deal timeline and dashboard recent activity. Stage changes from the table form, detail selector and board use the same service operation.

Entering Won sets actual close date to today in the local business timezone. Leaving Won clears that date; entering Won again sets a new date. Lost has no actual close date. Reopening either closed stage sets the new open-stage probability and retains the expected close date, which can therefore be overdue. Historical events are never removed when a deal is reopened.

An open deal is late when its expected close date is before today. Today's dates are not late; undated deals are not late. Tables and cards show a Raspberry indicator and the text “Overdue,” so color is not the only signal.

### Activities and follow-ups

Log activity accepts note, call or email; required description; happened-at timestamp defaulting to now; and optional follow-up date. A contact activity belongs only to that contact, and a deal activity only to that deal; do not duplicate entries across linked records. Stage changes are separate system events, not editable user activities.

Every activity with a follow-up date and done=false is a task. The dashboard groups all pending tasks as Overdue (before today), Today, and Upcoming (after today), sorted by due date then creation time. Each task shows its description, due date, type and target link or retained deletion label. Mark done persists completion and removes it from pending lists. Activity detail in the timeline still shows the completed follow-up. No activity edit/delete or task reopening is required.

## Dashboard and pipeline calculations

All date boundaries use the computer's local timezone captured as the app's business timezone at startup; timestamps are stored in UTC and dates as ISO calendar dates. The backend supplies this timezone and the reporting boundaries to the UI. Expected revenue is calculated in cents per deal, rounded to the nearest cent, then summed. All monetary displays explicitly include USD, for example `USD 12,450.00`; chart ticks may use `USD 12k`, with exact amounts in tooltips.

### Dashboard

The landing page includes:

1. Four summary cards: revenue won this calendar month, deals won this month, total open pipeline value, and expected open revenue. Month totals use currently Won deals and their actual close dates. Open metrics use only the four open stages.
2. A line chart of full won revenue for each of the last six calendar months, including the current partial month. Include zero months, point tooltips and the label “Current month to date.”
3. A bar chart of won deal count over those same months. Do not probability-weight won revenue or count historical wins on deals that are now reopened or Lost.
4. An open-pipeline visual with four stage segments, counts, summed value and expected revenue. Stage colors and ordering match the board. Show zero-value stages and a useful all-empty state.
5. A historical funnel, recent activity and follow-ups. Recent activity combines user activities and stage changes, newest first; show the latest ten with a timestamp, target and description.

Historical funnel definition: choose the cohort of deals first created during the trailing 90 local calendar days, including today. For each stage, count distinct cohort deal IDs with a recorded entry into that stage between their creation and now. Display New, Qualified, Proposal, Negotiation, Won and Lost, with Lost as a separate outcome beside Won. Count each deal at most once per stage, including after reopening. Do not infer skipped stages. Directly creating a Proposal deal counts Proposal but not New or Qualified, so counts need not decrease. Render aligned horizontal reach bars, not a tapered graphic implying guaranteed attrition. Include date range and this explanation beside the chart: “Deals created in the last 90 days. Each bar counts deals recorded entering that stage. Skipped stages are not counted; reopened deals count once per stage. History includes deleted deals.” The historical cohort uses retained initial-entry events, so deletion does not erase it.

Every metric is derived from persisted data. After a mutation, refresh all affected lists, details, dashboard metrics and board totals without requiring a browser reload. Explain the different six-month and 90-day chart windows in their subtitles.

### Pipeline

Display six columns in stage order, with closed stages visually quieter. The board header shows total open value and expected revenue. Each column shows count, summed value and summed expected revenue for its own deals; label Won/Lost totals as closed and exclude them from the board header totals.

Cards show name, organization or “No organization,” value, probability and labeled close date or “No expected date.” Include an overdue text marker where relevant. Cards open deal details through an explicit link. Use stable newest-created-first ordering; moving a card does not create a manually stored position.

Drag a card into another column to change stage. Update stage, probability, date and totals immediately, indicate saving, and persist the operation atomically. On failure restore previous values and totals and show a retryable error. Prevent another move of that card until its save finishes. Dropping in the same column does nothing. Provide a keyboard-accessible Move to stage menu that invokes the same operation. The Deals page provides the Add deal action; the board links there.

## Visual and interaction design

Use a persistent Midnight sidebar with Accountable branding, five labeled navigation links and a clear active state. The main area uses a light `#F0F4F5` canvas, white rounded panels without outlines, generous spacing and a strong page heading. Prioritize layouts at 1280×800 and 1440×900. At narrower desktop widths, tables and the board may scroll horizontally without clipping controls.

Use Instrument Sans for headings, labels and body; Instrument Serif sparingly for one dashboard accent line; JetBrains Mono for amounts, counts and identifiers. Bundle fonts locally so offline operation does not depend on Google Fonts. Use a consistent spacing scale of 4, 8, 12, 16, 24 and 32px, 16px panel corners, and restrained shadows. Avoid edge-only panel borders, a top-of-page rule and excessive gradients.

| Role | Color and treatment |
| --- | --- |
| Primary action and first chart series | Sea Green `#06D6A0` with Midnight text |
| Secondary emphasis and second series | Marigold `#FFC145` with Midnight text |
| Third series | Coral `#F78C6B` with Midnight text |
| Information and fourth series | Cerulean `#118AB2` with white text |
| Alerts and fifth series | Raspberry `#EF476F` with white text |
| Light-surface text | Midnight `#073B4C` |
| Dark-surface text | Sea Mist `#CDD8DB` |

Use Sea Green, Marigold, Coral and Cerulean for the four open stages across dashboard and pipeline. Won uses a labeled Sea Green treatment; Lost uses a labeled muted treatment. Darken bright colors when used as small text on light surfaces and verify contrast. Use short labels and plain business English, with no jargon or exclamation marks.

Dashboard composition: summary cards across the top; a large revenue line chart alongside the monthly wins bars; open pipeline and historical funnel below; follow-ups and recent activity in the lower row. Give charts clear axes, restrained grid lines and accessible text summaries. Pair visual emphasis with useful information instead of decorative widgets.

All inputs have visible labels, errors explain how to correct the value, and dialogs trap focus and return it to their trigger. Support keyboard navigation, visible focus, semantic tables with sort state, sufficient contrast and reduced motion. Charts offer text/table equivalents; stage and overdue status never rely on color alone. Destructive confirmations state consequences and use explicit labels such as “Delete organization.”

## Technical design

Use Node.js 26 (24 or later supported), a React/TypeScript frontend built with Vite, and a Python FastAPI backend managed with uv. Persist locally in SQLite, using SQLAlchemy and versioned migrations. This selects concrete tools within the prescribed stack; later Supabase and Render work is outside this MVP.

Use mature libraries: TanStack Table for sorting/filtering tables, TanStack Query for server state, Recharts for charts and dnd-kit for accessible drag/drop. Keep domain calculations on the backend and share API types through an explicit typed client. Serve assets locally and make no runtime calls to external services.

Store organizations, contacts, deals, activities and immutable deal-stage events. Stage events retain deal ID/name and creation cohort timestamp even if the live deal is deleted. Activities retain target type/name after unlinking. Enforce relationship integrity with nullable links and deletion transactions. Index relationship IDs, event timestamps, follow-up dates and actual close dates. Stage updates, probability resets, close-date changes and event insertion commit in one transaction. Deletion and unlinking also commit atomically.

Expose local JSON APIs for record list/detail/create/update/delete; activity creation; follow-up completion; deal stage changes; combined timelines; dashboard aggregates; and pipeline data. Lists accept search/sort and contact status filters. Return field-specific validation errors and clear not-found/server errors. The client must not report success before persistence completes. Run on loopback only; no sign-in is needed.

The repository root owns npm scripts and `frontend/` is an npm workspace. `npm run dev` starts frontend and backend together with cross-platform process management and stops both cleanly. `npm test` runs frontend Vitest, backend pytest and Playwright Test in `e2e/` against installed Google Chrome; any failure fails the command. Browser tests start their own app processes. Each test run uses a separate temporary database and must never seed or modify the user's database.

On a fresh database, migrate and seed once. Subsequent startups preserve all data, including an intentionally emptied database. `npm run seed` explicitly resets the local database to the sample dataset in a transaction. Use a configured data path independent of the current working directory.

## Sample data and setup

Seed approximately 15 organizations across at least five industries, 40 contacts covering every status, 45 deals covering all stages and at least 80 activities. Include standalone contacts/deals, optional fields left empty, varied probabilities, overdue open deals, future close dates, recent wins/losses, reopened deals and skipped stages. Provide stage histories and actual win dates consistent with the rules. Spread wins across six months and include enough recently created deals to populate every funnel stage. Include overdue, today, upcoming and completed follow-ups.

Use a deterministic seed with dates relative to the reset date so the dashboard is useful whenever the demo is installed. Seed histories must be chronological, begin at deal creation and end in the current stage. Ordinary startup must never regenerate sample dates or overwrite user edits.

Document setup and operation in `docs/STATUS.md`, leaving the course README unchanged. Include Mac, Linux and Windows instructions for installing Node, Python and uv, dependencies and Chrome/test prerequisites; supported versions; local data location; browser URL; `npm run dev`; `npm test`; and the destructive effect of `npm run seed`. Internet is allowed during installation only; verify normal use offline after dependencies and fonts are installed.

## Acceptance and verification

Implementation is complete only when the following checks pass:

- Fresh setup opens a populated dashboard with line, bar and pipeline visuals; all five sections look complete at the target desktop sizes and work offline.
- Organization/contact/deal create, search, sort, edit and delete work; contact status filtering combines with search. Standalone records and relationship links work. Confirmation cancellation preserves records; confirmed deletion keeps related records and retained history as specified.
- Every mutation survives browser refresh and process restart. An emptied database stays empty until reset. Reset restores coherent sample data.
- Board moves and the alternative stage menu apply defaults, update all totals and record exactly one event per actual change. Same-stage drops do nothing. Save failures roll back the UI. Won, Lost, reopening, manual probabilities and overdue boundaries obey the lifecycle rules.
- Activities appear once in the correct timeline; stage changes appear in deal timelines and recent activity. Follow-ups appear in the correct date group, can be completed and remain actionable after their target is deleted.
- Unit tests verify monetary rounding, month boundaries, local dates, stage defaults, actual close dates, reopening, deletion, history-based funnel counting, skipped stages and duplicate entries. Use a fixed clock for reproducible date checks. Verify aggregates against a small known dataset, including zero months and deleted history.
- Browser tests cover the main workflows and persistence using their own database. A person also exercises every section/action end to end in real Chrome, checks layouts, charts, keyboard controls, dialogs and error states by eye, and confirms no browser console errors. Passing automated tests alone is insufficient.

Record implementation progress, actual setup instructions, test results and the manual browser review in `docs/STATUS.md` as the work is completed.
