# Intent: the Accountable MVP

## Problem

A small business keeps track of its customers and sales in spreadsheets, inboxes and people's heads. Nobody can see at a glance which companies we deal with, who our contacts are, what's in the sales pipeline, what happened last with a customer or what needs doing next.

## Proposed outcome

Accountable is a business management system: a sales CRM that will grow into a full Business OS, including ERP. This first version is a course demo of a sales CRM for one person, running on their own computer, with realistic sample data. Think of it as a private Salesforce that demonstrates the everyday CRM essentials. It should feel sharp, modern and professional, and be a pleasure to use. The polished dashboard and interactive pipeline are the priorities, while the supporting sections retain the core workflows described below.

### Five sections

- **Dashboard**, the landing page: how sales are going at a glance. Deals won and revenue won month by month, the open pipeline and its expected revenue (each deal's value times its probability of close), a historical sales funnel showing how deals have progressed through stages over time, recent activity, and the follow-ups that are coming up or overdue. Revenue won is the full value of deals marked Won, grouped by actual close month. The designer chooses the historical funnel calculation and time window, and explains them clearly in the UI. It must use recorded stage history, rather than just the current stage of each deal. It should be visually stunning, with at least a line chart, a bar chart and a visual of the pipeline. Which figures each one shows, and how, is the designer's call.
- **Organizations**: the companies we do business with. Open one to see its contacts and deals.
- **Contacts**: the people we deal with, including leads and customers, with a filter by status. Open one to see their organization and a timeline of everything that has happened with them.
- **Deals**: the potential sales, with each one's stage, value and close date. Open one to see its details and its timeline. A deal still open after its expected close date stands out, in the table and on its pipeline card.
- **Pipeline**: the deals as cards on a board, one column per stage. Drag a card to another column to change the deal's stage. Manual ordering within columns and creating deals directly in a column are deferred; new deals are added from the deals table. The board shows the total value and expected revenue of the deals still open, each column shows its own, and the figures change as soon as a card moves.

Every organization, contact and deal table has a search box, columns you can sort by clicking their headings, a way to add a new record, and edit and delete on each row. That's how we add a new deal, a new lead or a new customer. Deleting asks first, and says what will happen to the records linked to the one being deleted. Linked records are kept and unlinked, rather than deleted along with it; deletion is not blocked by existing links.

From any contact or deal you can log an activity (a note, call or email) and give it a follow-up date. A follow-up shows on the dashboard as a task until it's marked done. Activity editing and deletion are deferred. When a deal moves to another stage, including Won or Lost, an entry such as "Moved from Proposal to Negotiation" appears in its timeline and in the dashboard's recent activity by itself. Stage changes are timestamped and persisted so they also support the historical funnel.

### Four kinds of record

- **Organization**: a company we do business with. Name, website, industry and notes. An organization has many contacts and many deals.
- **Contact**: a person we deal with. Name, email, phone, job title, an optional organization, and a status (lead, qualified or customer). A contact can exist without an organization. A lead is a contact with the lead status, so there's no separate list of leads.
- **Deal**: a potential sale. Name, an optional organization and optional main contact, its stage (New, Qualified, Proposal, Negotiation, Won or Lost), its value in US dollars, its probability of close, and its expected or actual close date. A deal can exist without an organization or main contact. Default probabilities are New 10%, Qualified 30%, Proposal 60%, Negotiation 80%, Won 100% and Lost 0%. A deal takes its new stage's probability when it moves; a probability set by hand is kept until the stage changes. Marking a deal Won records today's date as its actual close date. Won and Lost deals can be reopened by moving them to an open stage.
- **Activity**: something that happened, or needs to happen, with a contact or deal. Type (note, call or email), the contact or deal it relates to, a description and when it happened, and optionally a follow-up date and whether it's done.

### Sample data

Accountable comes with realistic sample data, so every screen looks alive the first time it starts: organizations across several industries, contacts at every status, deals at every stage (including deals won and lost over the past months, deals expected to close in the months ahead and a few running late), and activities with some follow-ups due soon and some overdue. Sample deals include historical stage transitions and appropriate actual close dates, so the historical funnel and monthly won figures are populated on first launch.

### Look and feel

The look follows Accountable's brand guidelines, which take priority over any general design guidance. The MVP is light mode, designed for a laptop or desktop screen.

## Affected users and systems

- The immediate audience is a course learner or reviewer exploring a realistic solo-salesperson demo on their own computer. Production use and larger teams are later concerns.
- No other systems are involved yet.

## Constraints

- The technology in `docs/TECH_STACK.md`.
- First-time setup instructions cover installing Node, Python and uv, as well as project dependencies, on Mac, Linux and Windows. Internet may be required during setup. After setup, it runs locally with one documented command and needs no internet connection to use.
- Every change is kept: it's still there after a browser refresh, and every time the app starts it shows every change made before.
- One documented command resets the data to the sample data, to start again from scratch.
- Not in the MVP: sign-in, user accounts or several users; a dark mode or a layout for phones; AI features; email, calendar or phone integrations; currencies other than US dollars; reports beyond the dashboard; tags or custom fields; changing the pipeline stages; importing or exporting data.
- Also deferred to keep the dashboard and pipeline in focus: activity editing and deletion, creating deals directly in pipeline columns, and manual card ordering within columns. Record CRUD, search, sorting, contact status filters, timelines and follow-ups remain in scope.

## Success criteria

1. A course learner can follow the first-time setup instructions, including installing Node, Python and uv, then start Accountable with the one documented command, open it in a browser and land on the dashboard, with realistic sample data on every screen. After setup, the demo works offline.
2. Organizations, contacts (leads, qualified contacts and customers) and deals can each be added, found by searching, sorted, edited and deleted from their tables, and every change is still there after a refresh and after a restart. Deleting asks first. The reset command brings back the sample data.
3. Contacts can be filtered by status, and each record's page shows the records it's linked to. Standalone contacts and deals can be created. Deleting a linked record preserves the others and clears their link to the deleted record, as explained in the confirmation.
4. Dragging a deal to another column on the pipeline changes its stage and its probability of close, and the deals list shows both. Stage changes persist after refresh and restart. The board's and the columns' totals and expected revenue change as soon as the card moves, and late deals stand out. Stage defaults are 10%, 30%, 60%, 80%, 100% and 0% in stage order. Marking a deal Won records today's actual close date; Won and Lost deals can be reopened.
5. An activity or follow-up logged on a contact or deal appears in its timeline, newest first, and so does an entry for each change of stage. Follow-ups show on the dashboard as upcoming or overdue until they're marked done.
6. The dashboard's figures match the data, and they reflect a deal marked won or a new follow-up. Monthly won revenue sums full won deal values by actual close month. The funnel uses persisted historical stage transitions, with its calculation and time window explained in the UI, and sample history makes it meaningful on first launch.
7. Every screen is sharp, modern and professional, and follows the brand. The dashboard is visually stunning, with a line chart, a bar chart and a visual of the pipeline.
8. Before it's called done, someone has used it end to end in a real browser, the way a salesperson would: every section and every action above, each screen checked by eye, with no errors in the browser's console. Passing tests is necessary, but it isn't enough.

## Deferred decisions

- Sign-in, several people sharing the same data, and a record of who did what are for a later intent.
- Making Accountable available to people beyond the computer it runs on is for later too.
