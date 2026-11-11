# UI States and Accessibility — F02 Dashboard

> Design status: V1 — draft, awaiting user review  
> Implementation status: NOT STARTED  

## UI State Contract for Dashboard Surfaces

Dashboard components manage high-frequency data streams, asynchronous caching, and real-time state changes. Every widget implements the following state contract:

| UI State | Visual Representation & Interaction Behavior |
|---|---|
| **Initial Loading** | Keep the global navigation and top bar intact; render animated pulse skeletons for KPI cards, chart containers, and tables. Avoid full-page spinners. |
| **Empty State (Positive)** | When no SLA breaches or pending incidents exist: render a clean positive status card: *"All active tickets are currently meeting SLA targets."* with a green shield icon. |
| **Empty State (Queue)** | When the Unassigned Team Pool is empty: *"No unassigned tickets in queue. Great job!"*. |
| **Filter No Results** | Display: *"No tickets match the selected filters."* alongside an immediate **Clear Filters** button. |
| **Aggregation Error** | If an isolated widget fails to compute: render an inline alert box within that widget: *"Failed to load chart data. [Retry]"* while leaving adjacent widgets fully operational. |
| **WebSocket Disconnection** | Status badge in top bar transitions from `🟢 Live` to `🔴 Reconnecting` $\rightarrow$ automatically switches to `🟡 Polling` (every 30s) if disconnected $>15\text{s}$. Seamlessly restores live streaming upon reconnection. |
| **Data Conflict** | If an engineer attempts to claim a ticket that was just claimed by a colleague: render a non-blocking toast: *"Ticket #TK-1011 was already claimed by Dave."* and refresh the queue list. |

---

## Status Chips, Badges & Visual Telemetry

In compliance with accessibility standards, color is **never used as the sole conveyor of information**; badges always include text or icons.

### SLA Breach Telemetry

| Badge Label | Color Code | Visual Icon | Meaning |
|---|---|---|---|
| **On Track** | Green (`#16a34a`) | ⏱️ Timer | Remaining time $> 20\%$ of SLA target duration. |
| **Due Soon** | Yellow (`#ca8a04`) | ⚠️ Warning | Remaining time $\le 20\%$ of SLA target duration (approaching breach). |
| **Overdue** | Red (`#dc2626`) | 🚨 Flashing siren | Deadline has been breached; counter displays negative overdue duration. |

### Engineer Workload Capacity Badges

| Capacity Status | Color Code | Range / Threshold | Meaning |
|---|---|---|---|
| **Normal** | Green (`#16a34a`) | Workload Index $\le 4$ | Available for additional task assignment. |
| **Busy** | Yellow (`#ca8a04`) | Workload Index $5 - 7$ | High load; approaching operational ceiling. |
| **Overloaded** | Red (`#dc2626`) | Workload Index $> 7$ OR $\ge 2$ P1/P2 tickets | Over capacity; triggers AI rebalance recommendation. |

### Priority & AI Attribution

* **P1 Critical:** Solid Red badge (`#b91c1c`) + bold text `P1 Critical`.
* **P2 High:** Orange badge (`#c2410c`) + text `P2 High`.
* **P3 Medium:** Yellow badge (`#a16207`) + text `P3 Medium`.
* **P4 Low:** Slate Blue badge (`#475569`) + text `P4 Low`.
* **AI Attribution:** Purple pill (`#7b3fbf`) with white text and bold prefix: **AI** (e.g., `[AI] 38% Automated`, `[AI] Suggested Rebalance`).

---

## Responsive & Accessibility Standards (WCAG 2.1 AA)

### 1. Screen Reader Compatibility (`aria-live`)
* Asynchronous metric recalculations use polite announcements (`aria-live="polite"`), preventing speech synthesizer interruption.
* Critical P1 emergency alert banners use assertive announcements (`aria-live="assertive"` and `role="alert"`), ensuring immediate broadcast.
* Analytical charts provide hidden semantic tables (`sr-only`) detailing the exact dataset for visually impaired users.

### 2. Keyboard Navigation Protocol
* Users can traverse the dashboard layout using `Tab` and `Shift + Tab` in a predictable left-to-right, top-to-bottom order.
* Each KPI card and chart section is keyboard-focusable (`tabindex="0"`) with `Enter` / `Space` triggering the corresponding drill-down action.
* Modals (`SCR-DB-11`, `SCR-DB-12`) and Drawers (`SCR-DB-10`) trap keyboard focus while active and dismiss cleanly upon pressing `Esc`.

### 3. Responsive Adaptations
* **Technician Workbench (`SCR-DB-02`):** Fully mobile-responsive. On tablet and smartphone viewports, the layout collapses into a single-column card stack; "Claim Ticket" touch targets expand to $\ge 44 \times 44\text{ px}$.
* **Executive Dashboard (`SCR-DB-01`):** Desktop and tablet-first view. On narrow viewports ($< 1024\text{ px}$), the multi-column chart and workload grid stack vertically with horizontal scrollbars for data tables.

**Related:** [Screen Inventory](04-screen-inventory.md) · [User Flows](05-user-flows.md)

