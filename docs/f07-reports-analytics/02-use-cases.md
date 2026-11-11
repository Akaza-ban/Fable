# Use cases — Reporting & Analytics

Roles, ticket visibility scope, permission matrix and `BR-xx` rules are reused from [mvp.md](../mvp.md). Ticket statuses, SLA policy and AI decisions are reused from the Workflow module ([03-use-cases.md](../03-use-cases.md), `BR-WF-xx` in [business-rules.md](../requirements/business-rules.md)). This file only adds `UC-BC-xx`, `BR-BC-xx`, metrics `M-xx` and **proposes** new permissions (section 5). The proposed permissions take effect only once the owner of `mvp.md` accepts them.

Related module: [Activity log](../activity-log/02-use-cases.md) (`UC-NK-xx`, `BR-NK-xx`) — the audit log that several metrics read from.

This module only **reads** data owned by other modules; it never redefines or changes it:

| Data | Owner | Used for |
|---|---|---|
| Ticket statuses `NEW`, `NEEDS_INFO`, `TRIAGE`, `ASSIGNED`, `IN_PROGRESS`, `AUTOMATING`, `RESOLVED`, `CLOSED`, `CANCELLED` | Workflow / Ticket | Time metrics |
| SLA policy versions, `sla_breached`, `SLA_WARNING` / `SLA_BREACHED` events (BR-WF-07, BR-WF-08) | Workflow | SLA metrics |
| `ai_decisions` (outcome `AUTO_APPROVED` / `NEEDS_INFO` / `ROUTE_TO_HUMAN`, BR-WF-17) | Workflow | AI metrics |
| Escalation, satisfaction ratings | Ticket | M-07, M-09 |
| KB suggestions and "helpful" ratings | Knowledge Base | M-06 |
| `AuditLog` (BR-09) | Shared | M-04, M-05 |

## 1. Overview

| Code | Use case | Actor |
|---|---|---|
| UC-BC-01 | Ticket statistics by type and group | ADMIN; TECHNICIAN (within scope) |
| UC-BC-02 | Track handling time and SLA breaches | ADMIN; TECHNICIAN (within scope) |
| UC-BC-03 | Evaluate the AI self-resolution rate | ADMIN |
| UC-BC-04 | Summarize requester satisfaction | ADMIN; TECHNICIAN (within scope) |
| UC-BC-05 | Filter and export reports | ADMIN |
| UC-BC-06 | Configure report parameters | ADMIN |

---

## 2. Metric definitions

Reports, the Dashboard and the demo all use **the same calculation** below (BR-BC-01). Parameters marked "Admin-configured" are edited in UC-BC-06 unless another module owns them. Default values are **suggestions for the demo**, not commitments.

| Code | Metric | Calculation | Parameters |
|---|---|---|---|
| M-01 | First response time | From ticket creation to the **first public reply** to the requester by a TECHNICIAN, ADMIN or AI_AGENT. Automatic notifications (`actor_type = SYSTEM`) do not count. Human and AI first responses are reported separately. | Whether an AI reply meets the response SLA: Q-BC-04 |
| M-02 | Resolution time | From creation to the **last** transition to `RESOLVED`, **minus** time in `NEEDS_INFO` (SLA paused, BR-WF-07) and minus time spent `RESOLVED` before the requester reported "not fixed". Measured on the working calendar of the ticket's SLA policy version (BR-WF-08), so M-02 and M-03 agree. | Working calendar: owned by Workflow (UC-WF-02) |
| M-03 | SLA breach | **Read from Workflow, not recomputed:** a ticket is breached when Workflow set `sla_breached = true` (`SLA_BREACHED` event). "At risk" = an `SLA_WARNING` event without a breach yet. Breach rate = breached tickets / tickets with an SLA policy in the period. | Targets and warning threshold: owned by Workflow (UC-WF-02; demo P1–P4 defaults, warning 75%) |
| M-04 | AI self-resolution rate | Numerator: tickets `CLOSED` in the period that **(a)** had no handling action by a TECHNICIAN or ADMIN (reply, assignment, status change, reclassification, internal note, retry or manual resolution of a workflow run) **and (b)** were not reopened within N days of closing. Denominator: tickets `CLOSED` in the period. **Excluded from both:** `CANCELLED` tickets, tickets auto-closed for no response (`NEEDS_INFO` → `CLOSED`, UC-WF-06), and tickets closed less than N days ago (shown separately as "under observation"). The requester's confirmation is not a handling action. | N: Admin-configured (demo: 7 days, the same as the Workflow reopen window, Q-BC-10) |
| M-05 | AI classification correction rate | Tickets whose type or group was set by the AI (`actor_type = AI`) and **later** changed by a human (`actor_type = USER`) / tickets classified by the AI. Derived from the audit log (BR-09). | — |
| M-06 | Helpful KB suggestion rate | KB suggestions rated "helpful" / KB suggestions **with a rating**. Also: rating rate = suggestions with a rating / suggestions shown. Ratings come from the KB module. | Who can rate: Q-BC-08 |
| M-07 | Satisfaction | Average score (1–5) of rated tickets, with the count at each score from 1 to 5. Response rate = rated tickets / tickets `CLOSED` in the period. The response rate is always shown next to the average. | — |
| M-08 | AI routing outcome | Share of AI decisions per outcome: `AUTO_APPROVED` (route A), `NEEDS_INFO` (route B), `ROUTE_TO_HUMAN` (route C), plus the reasons for route C. Read from `ai_decisions`; this is the data Workflow provides to reporting (NFR-WF-OBS-001). | — |
| M-09 | Escalation rate | Tickets escalated at least once (manual escalation in the Ticket module, or SLA escalation by Workflow, UC-WF-06) / tickets created in the period. The two sources are reported separately. | — |

**Common calculation rules:**
- **Count** statistics assign tickets to a period by **creation date**. **Time, SLA, M-04 and M-07** assign tickets by **resolution / close date**. `CANCELLED` tickets are counted in volume only.
- Grouping by priority uses the ticket's **final** priority; tickets whose priority was raised by SLA escalation are flagged (Q-BC-06).
- A ticket moved across groups is attributed to the **last group** that handled it (Q-BC-07).
- Every metric is computed only on tickets **within the viewer's scope** (BR-BC-02).

---

## 3. Use cases

### UC-BC-01 Ticket statistics by type and group
- **Main flow:** choose a time range (default: last 30 days) → the backend loads the tickets within the viewer's scope → counts new, open, closed, cancelled, reopened and escalated tickets (M-09), grouped by ticket type and technician group → returns a table with a daily / weekly / monthly trend.
- **Alternative:** a TECHNICIAN only sees figures for tickets assigned to them and to their groups; the group filter only lists their groups. If their membership just changed, the new scope applies from the next request (BR-07). Tickets in `TRIAGE` are shown as their own group "Unassigned".
- **Exceptions:** no `report:read` → `403`. Filtering by a group outside scope → `403`. Invalid range (from > to, or longer than the maximum) → `422` `INVALID_DATE_RANGE`.
- **BR:** BR-07, BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-02 Track handling time and SLA breaches
- **Main flow:** choose a period and filters → show M-01, M-02 (mean, median, P90) and M-03 by priority, ticket type and group → list of breached tickets (code, type, group, priority, SLA policy version, overdue by how long) → clicking a ticket opens its detail, checked against `ticket:read`.
- **Alternative:** list tickets **at risk** (M-03). Tickets without an SLA policy are counted separately as "no SLA". Tickets whose SLA policy version changed after a priority change are evaluated by Workflow against their current version (BR-WF-08); the report does not re-evaluate.
- **Exceptions:** no permission or outside scope → `403`. SLA data unavailable from Workflow → M-01 and M-02 are still shown; M-03 shows "unavailable" rather than 0%.
- **BR:** BR-WF-07, BR-WF-08, BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-03 Evaluate the AI self-resolution rate
- **Main flow:** choose a period → show M-04 (with the "under observation" count), M-05, M-06 and M-08 by ticket type → lists of: tickets the AI closed that were reopened; tickets whose AI classification was corrected (AI value → human value, from the audit log `before` / `after`); route C reasons ranked by frequency.
- **Alternative:** filter by AI service account. Compare periods to see whether the AI improves after suggestions are approved (BR-15) or after Admin changes the AI scope (UC-WF-03).
- **Exceptions:** TECHNICIAN, REQUESTER, AI_AGENT → `403` (proposal: ADMIN only, Q-BC-03). No ticket in the period has been closed for N days → "not enough data" rather than 0%.
- **BR:** BR-09, BR-15, BR-WF-17, BR-BC-01, BR-BC-03.

### UC-BC-04 Summarize requester satisfaction
- **Main flow:** choose a period → show M-07 (average, distribution from 1 to 5, response rate) by ticket type, group and handler (AI only / human) → list of tickets rated 1–2.
- **Alternative:** a TECHNICIAN sees ratings for tickets within their scope, but not other technicians' individual figures (Q-BC-01, Q-BC-02). Comments attached to ratings are visible to ADMIN only.
- **Exceptions:** response rate below the threshold (Admin-configured, demo 20%) → the score is shown with the warning "few ratings, not representative". No permission → `403`.
- **BR:** BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-05 Filter and export reports
- **Main flow:** choose a report (UC-BC-01…04) and filters (period, ticket type, group, priority, handler AI or human, SLA status, AI routing outcome) → preview → Export as CSV or XLSX → the system **writes audit log entry `REPORT_EXPORTED`** → builds a file with only allowlisted columns → returns the file.
- **Alternative:** row count above the limit (Admin-configured, demo 50,000) → `422` `EXPORT_TOO_LARGE`, asking to narrow the filters. Filters are kept in the URL so the same report can be reopened.
- **Exceptions:** no `report:export` → `403`. Audit log write fails → **no file is returned**, `503` `AUDIT_UNAVAILABLE` (BR-BC-05).
- **BR:** BR-09, BR-BC-02, BR-BC-04, BR-BC-05.

### UC-BC-06 Configure report parameters
- **Main flow:** the Admin edits report-only parameters: N days for M-04, satisfaction response-rate threshold, export row limit, maximum time range → save → audit log entry `REPORT_CONFIG_UPDATED` with before and after values.
- **Not here:** SLA targets, warning threshold and working calendar are configured in Workflow (UC-WF-02); this screen links to it.
- **Alternative:** a change only affects **how figures are computed when viewed**; files already exported do not change. Each report shows the parameters in use (for example "N = 7 days").
- **Exceptions:** not an ADMIN → `403`. Invalid value (negative, N = 0, limit = 0) → `422`.
- **BR:** BR-09, BR-BC-01.

---

## 4. New business rules

- **BR-BC-01**: All reports and the Dashboard use the definitions M-01…M-09 in section 2. Each report states its period, time zone (`Asia/Ho_Chi_Minh`) and the parameters in use.
- **BR-BC-02**: Figures are computed only on tickets the viewer can read under `ticket:read` (mvp.md section 4). Scope is filtered **in the backend query**, before aggregation. Aggregating over all tickets and hiding parts in the UI is not allowed.
- **BR-BC-03**: Reports are **read-only**. No report endpoint creates, edits or deletes tickets, audit logs or business data. SLA status is read from Workflow, not recomputed. Queries should use a read-only database account.
- **BR-BC-04**: Exported files contain only allowlisted columns: ticket code, type, group, priority, status, timestamps, metrics M-01…M-09. They **never** contain internal notes, passwords or hashes, tokens, API keys, 2FA secrets, workflow payloads or AI extracted fields. Cells starting with `=`, `+`, `-` or `@` are prefixed with `'` to prevent formula (CSV) injection.
- **BR-BC-05**: Every export writes audit log entry `REPORT_EXPORTED` (`actor_type = USER`; `after` = report, filters, format, row count). If the log write fails, no file is returned.

---

## 5. Proposed additions to the permission matrix

Proposals for the owner of [mvp.md](../mvp.md) to decide. This file does **not** edit `mvp.md`. Names follow the `resource:action` style of `mvp.md` (Q-BC-12). Activity log permissions are in [activity-log/02-use-cases.md](../activity-log/02-use-cases.md).

| Proposed permission | ADMIN | TECHNICIAN | REQUESTER | AI_AGENT | Used in |
|---|---|---|---|---|---|
| `report:read` | all | assigned, group (no other technicians' individual figures) | ❌ | ❌ | UC-BC-01, 02, 04 (UC-BC-03 ADMIN only) |
| `report:export` | ✅ | ❌ | ❌ | ❌ | UC-BC-05 |
| `report:configure` | ✅ | ❌ | ❌ | ❌ | UC-BC-06 |

---

## 6. Open questions (for the team to decide)

**Roles**
- **Q-BC-01**: Workflow already marks some technicians as **Team lead** (`is_lead`). Should a team lead see per-technician figures (SLA, satisfaction) for their own group? Proposal: yes, as a scope flag rather than a new role. This needs the owner of `mvp.md` to accept `is_lead`.
- **Q-BC-02**: Can a technician who is not a lead see colleagues' individual figures, or only the group total and their own?
- **Q-BC-03**: Is UC-BC-03 (AI effectiveness) for ADMIN only, or can team leads see it for their group?

**Metric calculation**
- **Q-BC-04**: Does an AI reply count as the first response for the SLA (Workflow's SLA engine decides; M-01 reports both)?
- **Q-BC-05**: Does the `SLA_BREACHED` event say whether the response or the resolution target was missed? Needed to split M-03.
- **Q-BC-06**: When SLA escalation raises a ticket's priority, should reports group it under its initial or final priority? Proposal: final, with a flag.
- **Q-BC-07**: A ticket moved across groups is attributed to the last group (proposal), the first, or each group that handled it?
- **Q-BC-08**: Who can rate a KB suggestion as helpful (requester, technician, both)? Owned by the KB module.
- **Q-BC-09**: At ~2.9 million tickets/year (Workflow NFR), live queries over a 12-month range will be slow. Proposal: live queries for the demo, daily pre-aggregated tables in production. Does the Dashboard share the same API?
- **Q-BC-10**: Workflow lets a requester reopen an auto-closed ticket within 7 days. Is the reopen window the same for every `CLOSED` ticket? If yes, N in M-04 should equal it.

**Export and conventions**
- **Q-BC-11**: Is PDF needed? Can satisfaction comments be exported (they may contain personal information)?
- **Q-BC-12**: Permission names differ between documents: `mvp.md` uses `user:read`, Workflow uses `workflowConfigManage`, `ticketReassign`. Which convention does the team keep?

See also: [01-user-stories.md](01-user-stories.md) · [03-information-architecture.md](03-information-architecture.md) · [Activity log](../activity-log/02-use-cases.md)
