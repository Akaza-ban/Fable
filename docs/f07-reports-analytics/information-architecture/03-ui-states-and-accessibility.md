# UI States and Accessibility — F07 Reports & Analytics

> Design status: TARGET V1
>
> Implementation status: NOT STARTED
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [User flows](02-user-flows.md)

## 1. Data state contract

Every tile, chart and list shows exactly one of these states. Tiles load independently; the filter bar never waits for them.

| State | When | What the user sees |
|---|---|---|
| `LOADING` | Query running | Skeleton of the tile or chart; the previous figures stay greyed while filters change |
| `READY` | Result available | The figure, its period and "Parameters in use" |
| `NOT_ENOUGH_DATA` | No ticket matches, or none closed for N days (M-04) | "Not enough data for this period" — never `0%` (BR-BC-01) |
| `UNDER_OBSERVATION` | Tickets closed less than N days ago | Count shown next to M-04: "312 under observation (closed < 7 days)" |
| `UNAVAILABLE` | Source branch down (F05 SLA, F04 analyses) | "SLA data unavailable" on the affected tiles only; other tiles stay `READY` |
| `LOW_SAMPLE` | Satisfaction response rate below the threshold | Score shown with the warning "Few ratings, not representative" |
| `FORBIDDEN` | Role or scope not allowed | Tab not shown; direct URL → SCR-90 |
| `ERROR` | `422 INVALID_DATE_RANGE` | Message under the period field; previous figures kept |

## 2. Who produced the figure

| Marker | Meaning | Where |
|---|---|---|
| **From Workflow (F05)** | Figure read from F05 data, never recomputed | M-03 tiles, Breached / At risk lists |
| **AI** | Part of a split that the AI handled alone | M-04 path split, M-01 AI first response, M-07 handler "AI only" |
| **Target** | Line set in Report settings | M-04 weekly chart |

## 3. Chart rules

- One metric per chart; the axis label states the unit and the period.
- Rates are always shown with their count ("41% · 3,280 of 8,000").
- Colors come from one categorical palette; breached / at risk also use an icon and a label, never color alone.
- Every chart has a *View as table* toggle with the same numbers.

## 4. Responsive layout

| Width | Layout |
|---|---|
| ≥ 1200 px | Filter bar on top; tiles in a row; charts two per row |
| 768–1199 px | Tiles two per row; charts one per row |
| < 768 px | One column; filter bar collapses into a *Filters* button; tables scroll inside their card, the page never scrolls sideways |

## 5. Accessibility (WCAG 2.1 AA)

- Tabs use `role="tablist"`; the active tab and the filters are reflected in the URL.
- Tiles updated after a filter change are `aria-live="polite"`.
- Charts have a text summary ("Breach rate rose from 6% to 9% over 4 weeks") and the table view.
- Contrast ≥ 4.5:1; focus is visible on every bar and row that drills down.
- SCR-BC-10 traps focus, closes with Esc, and returns focus to *Export*.
