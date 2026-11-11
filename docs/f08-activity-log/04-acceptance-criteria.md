# Acceptance Criteria — F08 Activity Log

> Design status: TARGET V1
>
> Implementation status: NOT STARTED. Every criterion is an acceptance target.

Format: *Given* (context) · *When* (action) · *Then* (observable result). Criteria are grouped by use case (`AC-UC-NK-xx`, as in F04 and F05) and numbered by story: `AC-NK-02.3` is the third criterion of US-NK-02. The UI may satisfy a criterion visually, but permissions and rules **must** be enforced by the API.

Sample people (same as F04): requester **Lan** (Accounting, Real Estate), requester **Minh** (Sales, EV Charging), technician **Hai** (IT Helpdesk), technician **Quang** (SAP, lead of SAP), Admin **Thu**.

## AC-UC-NK-01 — Ticket history

*(US-NK-01, US-NK-02)*

- **AC-NK-01.1** *Given* Hai opens a ticket assigned to him, *when* he opens `#history`, *then* every entry shows the actor (name, **AI** with the domain agent, or **Automation**), time and before / after values.
- **AC-NK-01.2** *Given* an entry written by F05, *then* it links to its evidence (`decision_id`, `rule_id`, `run_id`).
- **AC-NK-01.3** *Given* a ticket outside Hai's scope, *when* he calls its history, *then* `403`.
- **AC-NK-02.1** *Given* Lan opens her own ticket, *then* she sees only events visible to the requester: created, status changes, public replies, AI questions and her answers, KB answer, approval / execution summary, resolved, closed, reopened, cancelled, rated.
- **AC-NK-02.2** *Given* the ticket has an internal note, an AI confidence, a policy condition, workflow attempts and an SLA warning, *then* none of them is in Lan's UI **or in the API response**.
- **AC-NK-02.3** *Given* a new event type not yet classified, *then* it is hidden from Lan (BR-NK-04).
- **AC-NK-02.4** *Given* Lan calls the history of Minh's ticket, *then* `403`.

## AC-UC-NK-02 — Track actions of people and AI

*(US-NK-04, US-NK-10)*

- **AC-NK-04.1** *Given* Thu filters `actor_type = AI`, *then* only AI entries are returned; `AUTOMATION` returns assignment rules, SLA timers, API execution, verification and Microsoft sync.
- **AC-NK-04.2** *Given* filters on actor, action, target, evidence id and period, *then* they combine and results are paginated.
- **AC-NK-04.3** *Given* Thu opens an entry, *then* `before` / `after` are shown as a diff.
- **AC-NK-04.4** *Given* a user disabled last month, *then* old entries still show their name.
- **AC-NK-04.5** *Given* a Technician or Requester calls `/audit-logs`, *then* `403`.
- **AC-NK-10.1** *Given* the `HELPDESK` agent answered Minh with KB-205, *then* the log has `AI_KB_ANSWER_SENT` with `agent = HELPDESK`, the article, `model_version` and `prompt_version`.
- **AC-NK-10.2** *Given* Hai corrected the AI category, *then* the log has `AI_FIELDS_CORRECTED` with AI value → human value and the `analysis_id`.
- **AC-NK-10.3** *Given* an analysis stored as `INVALID`, *then* the log has `AI_ANALYSIS_INVALID` with the failed check, visible to Admin only.

## AC-UC-NK-03 — Approval chain

*(US-NK-05)*

- **AC-NK-05.1** *Given* Lan's `GRANT_ACCESS` request was approved and executed, *then* SEC-NK-02 shows request → AI understanding → each policy condition ✓ / ✗ → decision → attempts → verification → confirmation, each with actor and time.
- **AC-NK-05.2** *Given* an auto-approved request, *then* the decision step shows actor `AI` and its `decision_id`; Thu's *View AI decision* opens SCR-WF-31.
- **AC-NK-05.3** *Given* a request approved but failed at execution, *then* it is highlighted "Approved — execution failed" with the assigned technician.
- **AC-NK-05.4** *Given* a retry, *then* it appears as new steps and earlier steps are unchanged.
- **AC-NK-05.5** *Given* Lan opens the chain, *then* she sees one status line and the short reason, no confidence, conditions, payloads or attempts.

## AC-UC-NK-04 — Workflow runs and errors

*(US-NK-06, US-NK-07)*

- **AC-NK-06.1** *Given* Thu, *then* she can filter runs by status (`QUEUED` … `MANUALLY_RESOLVED`), error type, operation, connector and period.
- **AC-NK-06.2** *Given* a run, *then* each attempt shows time, duration, result code and masked request / response.
- **AC-NK-06.3** *Given* Hai is assigned, *then* he sees the full attempts; another technician of his group sees a summary.
- **AC-NK-06.4** *Given* a Requester calls `/workflow-runs`, *then* `403`.
- **AC-NK-07.1** *Given* the AI Service times out or returns an `INVALID` analysis, *then* the AI result is not applied and the ticket is routed to a person (rule or `TRIAGE`).
- **AC-NK-07.2** *Given* that routing, *then* the history has `TICKET_ROUTED_TO_HUMAN` with `AI_UNAVAILABLE` or `AI_INVALID_OUTPUT`.

## AC-UC-NK-05 — Write the log

*(US-NK-03, US-NK-08, US-NK-09)*

- **AC-NK-03.1** *Given* any role including Admin, *when* it sends `PUT`, `PATCH` or `DELETE` to a log endpoint, *then* `405`.
- **AC-NK-03.2** *Given* the application's database account, *when* it runs `UPDATE` or `DELETE` on the audit table, *then* the database refuses.
- **AC-NK-03.3** *Given* a correction, *then* it is a new entry with `corrects_id`; the old entry is unchanged.
- **AC-NK-08.1** *Given* a field named like `password`, `token`, `secret`, `api_key`, `authorization`, `cookie` or `otp`, *then* it is stored as `"***"`.
- **AC-NK-08.2** *Given* a `Bearer …` string, a JWT or an API key inside free text, *then* it is stored as `"***"`.
- **AC-NK-08.3** *Given* any entry, *then* `password_hash`, `totp_secret_enc`, `api_key_hash` and connector credentials never appear in `before` / `after`.
- **AC-NK-09.1** *Given* the "execution started" entry cannot be written, *then* the connector is not called and the run ends `FAILED` with `AUDIT_UNAVAILABLE`.
- **AC-NK-09.2** *Given* AC-NK-09.1, *then* the ticket is routed to a technician.
- **AC-NK-09.3** *Given* the connector call succeeded but the result entry failed, *then* the entry is retried, the Admin is alerted, and the connector is not called a second time.

**Related:** [User stories](03-user-stories-invest.md) · [Use cases](02-use-cases.md) · [Business rules](05-business-rules.md)
