# Business Rules — F07 Reports & Analytics

> Design status: TARGET V1
>
> Implementation status: NOT STARTED

The rules of F05 (`BR-WF-xx`), F04 (`BR-AI-xx`) and F01 (`BR-05 … BR-20`, `docs/mvp.md` on `feature/author`) still apply. The rules below cover only what F07 adds.

## BR-BC-01 — One definition per metric

- **Source:** *Metric definitions* in [02-use-cases.md](02-use-cases.md)
- **Stories:** all
- **Rule:** Reports, the F02 Dashboard, F05's SCR-WF-30 cards, exports and the digest use the definitions M-01 … M-11 and the same API. Each report states its period, time zone (`Asia/Ho_Chi_Minh`) and the parameters in use. A metric with no data is shown as "Not enough data", never `0%`. A metric is shown only at a granularity its definition allows: M-04 per week once the N-day window has passed, never per day (its latest days are always "under observation").

## BR-BC-02 — Scope is filtered in the query

- **Source:** UC-BC-01, UC-BC-02, UC-BC-04
- **Stories:** US-BC-02
- **Rule:** Figures are computed only on tickets the viewer may read (`ticket:read`, F01). The scope is applied in the backend query **before** aggregation. Aggregating everything and hiding parts in the UI is not allowed. A Technician never sees another technician's individual figures.

## BR-BC-03 — Reports are read-only

- **Source:** all use cases
- **Stories:** all
- **Rule:** No report endpoint creates, changes or deletes tickets, SLA data, AI data or log entries. SLA breach is read from F05, never recomputed. Report queries use a read-only database account. The only writes are F07's own parameters and its audit entries (through F08).

## BR-BC-04 — Exports contain allowlisted columns only

- **Source:** UC-BC-05
- **Stories:** US-BC-06
- **Rule:** Exported files contain only: ticket number, type, `domain/subcategory`, group, priority, status, resolution code, escalation class, timestamps, and metrics M-01 … M-11. They **never** contain internal notes, passwords or hashes, tokens, API keys, workflow payloads, AI extracted fields or rating comments. Cells starting with `=`, `+`, `-` or `@` are prefixed with `'`.

## BR-BC-05 — No export without a log entry

- **Source:** UC-BC-05, UC-BC-06
- **Stories:** US-BC-06, US-BC-07
- **Rule:** Every export writes `REPORT_EXPORTED` (source `USER`; report, filters, format, row count) **before** the file is returned; if the write fails, no file is returned (`503 AUDIT_UNAVAILABLE`). Every parameter change writes `REPORT_CONFIG_UPDATED` with before and after values.

## BR-BC-06 — A ticket counts as AI-resolved only after observation

- **Source:** UC-BC-03, metric M-04
- **Stories:** US-BC-04
- **Rule:** A closed ticket counts as resolved without a technician only when no Technician or Admin took a handling action on it and it was not reopened within N days of closing. N is the reopen window of F03 BR-TK-11, read by F07, not a parameter of its own. Until N days have passed it is "Under observation". The requester's confirmation and F05 automation are not handling actions.

## BR-BC-07 — The digest carries figures, not ticket content

- **Source:** UC-BC-08
- **Stories:** US-BC-11
- **Rule:** The weekly email contains aggregate figures and links only: no ticket titles, descriptions, names of requesters or comments. Links require sign-in and respect the recipient's role.

**Related:** [Use cases](02-use-cases.md) · [Acceptance criteria](04-acceptance-criteria.md) · [Shared business rules](../requirements/business-rules.md)
