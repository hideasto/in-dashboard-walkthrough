# Power BI In-Dashboard Onboarding Walkthrough

**[▶ Live interactive preview](https://htmlpreview.github.io/?https://github.com/YOUR-USERNAME/YOUR-REPO-NAME/blob/main/walkthrough-demo.html)** — click through Next/Back the way it behaves in the real dashboard.

A bookmark-driven, step-by-step walkthrough overlay for Power BI dashboards — the "first time here? want a tour?" pattern, built entirely with native Power BI bookmarks and buttons. No custom visuals, no external tooling.

I originally saw this technique demonstrated in a video I came across a while back — I wasn't able to track down the original link to credit it directly. This repo documents my own implementation of it, the gotchas I hit building it, and the trade-offs of the approach I chose versus a more complex alternative — not a claim that the underlying pattern is original.

## Motivation

Before this, onboarding for the dashboard was a recorded video tutorial on YouTube. In practice, most users weren't watching it end to end <!-- TODO: replace with view-duration / retention stat from YouTube Studio if available -->, and I kept seeing the same signs of people getting lost inside the actual dashboard — unsure what a filter did, unsure where to start.

A video requires a context switch: leave the tool, watch someone else use it, try to remember what they did, come back and attempt it yourself. That gap is where most of the drop-off happens. Putting the guidance *inside* the dashboard, next to the control it's explaining, removes that gap entirely.

The other requirement that shaped this design: the walkthrough shouldn't just be a passive tour. Users needed to be able to make real filter/slicer selections *while* the walkthrough is guiding them, without the tour resetting or overwriting what they picked. That constraint is what led to the bookmark-scoping approach documented below — see "How it works."

## What it does

- On first load, a modal asks the user if they'd like a guided tour or want to jump straight to the dashboard.
- A "Start Walkthrough" button also sits in the top header bar at all times, so users who dismissed the tour (or just want to replay it) can restart it without reloading the page. A "Frequent Q&A" button sits next to it, for the same reason — both are real buttons in the header, not text links buried in the page.
- Metric Selection, Disaggregation, and the Academic Year/Term toggle are shared and apply to both tables at once. The Primary Table and Comparison Table each have their **own independent** set of drill-down filters (Region/Route/Gym/Battle ID in the demo; School/Department/Subject/Course in the real dashboard) — so you can, for example, filter the top table down to a single course while leaving the bottom table showing the full collegewide benchmark. The walkthrough calls this out explicitly in the steps covering each table, since it's easy to miss on first use.
- If they opt in, a sequence of teal callout boxes walks them through each control on the page — what it is, what it does, why they'd use it — with Back/Next navigation.
- Critically, the walkthrough **does not interfere with the user's own filter/slicer selections**. A user can start the tour, select a school or a course mid-walkthrough, and their selection persists exactly as if the tour weren't running.

## Demo

Screenshots of the pattern (mock institution, entirely fictional data — see `walkthrough-screenshots/`):

| Step | Preview |
|---|---|
| Welcome gate | `00-welcome-prompt.png` |
| Step 1 | `01-clear-slicers.png` |
| Step 2 | `02-alltime-or-region.png` |
| Step 3 | `03-metric-and-disaggregation.png` |
| Step 4 | `04-primary-table.png` |
| Step 5 | `05-comparison-table.png` |
| Step 6 | `06-frequent-qa.png` |
| Clean dashboard, no overlay | `07-clean-dashboard-no-overlay.png` |

An interactive HTML mockup of the same pattern is included as `walkthrough-demo.html` — open it locally to click through Next/Back the way it behaves in the real dashboard. It's a stand-in for the concept, not an export of an actual Power BI file.

## How it works

1. **One bookmark per step**, each capturing only the state of the single slicer/parameter that tracks "which step is the user on" — not a full-page snapshot.
2. Each bookmark's capture scope is set to **"Selected visuals"** rather than "All visuals" or the default "Data" capture. This is the detail that matters most and the easiest thing to get wrong: if a bookmark captures the whole page's filter state, clicking "Next" on the tour silently overwrites whatever the user just selected in their own filters. Scoping capture to only the step-tracking control keeps everything else on the page untouched.
3. Next/Back buttons are wired to the Bookmark action, targeting the appropriate step's bookmark.
4. Bookmarks are grouped in the Bookmarks pane (one group per page) and named consistently (`P1_Step1`, `P1_Step2`, …) for maintainability.

## Why bookmarks instead of a parameter-driven build

A more "scalable" version of this pattern exists — using a What-If parameter plus a lookup table to drive step visibility and content, so the same measures and bookmarks are reused across many pages instead of rebuilt per page. I prototyped that version and deliberately didn't use it here. It requires:

- Card visuals instead of plain text boxes (plain text boxes can't bind to a DAX measure's value)
- A working "Set a value" button action, which isn't available in every Power BI Desktop version
- Meaningfully more DAX to write and debug

For a small number of pages, the added complexity wasn't worth it. **Bookmark count scales linearly with pages × steps** in this implementation — fine at the scale I'm using it at, worth revisiting with a parameter-driven design if a project grows into many pages with identical tour structure.

## Setup

1. Build your dashboard's filters/slicers as normal.
2. Add a step-tracking control (a small numeric slicer or What-If parameter) somewhere on the canvas — it doesn't need to be visible to end users.
3. Create your teal callout boxes as text boxes or shapes, positioned near the control each one explains.
4. For each step, select **only the step-tracking control**, record a bookmark, and set its capture scope to "Selected visuals" in the bookmark's options menu.
5. Group bookmarks per page and rename them for clarity.
6. Wire Next/Back buttons to the Bookmark action, targeting the correct step.

## Limitations

- Bookmark count is not reusable across pages — each page's tour is built independently.
- No built-in "seen it, don't show again" persistence across sessions (Power BI bookmarks don't carry user-level state between sessions on their own).
- If a step's target control moves or is redesigned, its bookmark needs to be re-recorded.

## License

MIT — see `LICENSE`.
