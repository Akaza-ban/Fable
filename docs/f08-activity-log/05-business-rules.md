# Business Rules — F08 Activity Log

> Design status: TARGET V1
>
> Implementation status: NOT STARTED

The rules of F05 (`BR-WF-xx`), F04 (`BR-AI-xx`) and F01 (`BR-05 … BR-20`, `docs/mvp.md` on `feature/author`) still apply. The rules below cover only what F08 adds.

## BR-NK-01 — Logging scope

- **Source:** UC-NK-05, *Event catalog* in [02-use-cases.md](02-use-cases.md)
- **Stories:** US-NK-01, US-NK-04, US-NK-10
- **Rule:** The record structure of F01 BR-09 is kept; its scope is extended to: ticket changes; AI analyses, corrections, questions, KB answers and escalations (F04); AI decisions, workflow run steps and SLA events (F05); routing to a person after AI failure; report exports, digests and report settings (F07). AI entries use `actor_type = AI` with `agent`, `model_version`, `prompt_version`; automated entries use `actor_type = AUTOMATION` with their evidence id (BR-WF-05). F01 entries written as `SYSTEM` are read as `AUTOMATION` (Q-NK-03).

## BR-NK-02 — The log is append-only

- **Source:** UC-NK-01, UC-NK-02, UC-NK-05
- **Stories:** US-NK-03
- **Rule:** Nobody can edit or delete an entry, **including Admin**: there is no edit or delete API (`405`), and the application's database account has only `INSERT` and `SELECT` on the audit table. A correction is a new entry that references the old one. Retention (Q-NK-07) is handled by a scheduled archive job outside the application account.

## BR-NK-03 — Secrets are masked before writing

- **Source:** UC-NK-04, UC-NK-05
- **Stories:** US-NK-08
- **Rule:** All `before` / `after` values, payloads and API responses are masked before writing: fields whose name contains `password`, `token`, `secret`, `api_key`, `authorization`, `cookie`, `otp` → `"***"`; strings matching `Bearer …`, JWT or API key patterns → `"***"`. `password_hash`, `totp_secret_enc`, `api_key_hash` and connector credentials are never written (BR-05, BR-13, BR-WF-15, BR-AI-07).

## BR-NK-04 — Requesters see an allowlist

- **Source:** UC-NK-01, UC-NK-03
- **Stories:** US-NK-02, US-NK-05
- **Rule:** A Requester sees only event types marked visible in the event catalog, and routing / workflow events only as a summary. The filter is applied in the API. An event type that has not been classified is hidden from the Requester by default.

## BR-NK-05 — AI failure is recorded and goes to a person

- **Source:** UC-NK-04
- **Stories:** US-NK-07
- **Rule:** When the AI Service does not respond or its analysis is `INVALID` / `FAILED`, the AI result is not applied and F05 routes the ticket to a person (UC-WF-04); the log records `TICKET_ROUTED_TO_HUMAN` with `AI_UNAVAILABLE` or `AI_INVALID_OUTPUT`.

## BR-NK-06 — No external change without a log entry

- **Source:** UC-NK-05
- **Stories:** US-NK-09
- **Rule:** For operations that change an external system (`CREATE_ACCOUNT`, `GRANT_ACCESS`, `REVOKE_ACCESS`), writing the "execution started" entry is a precondition. If it fails, the connector is not called and the access is not granted.

## BR-NK-07 — Same transaction for internal changes

- **Source:** UC-NK-05
- **Stories:** US-NK-03, US-NK-09
- **Rule:** A change to the system database and its log entry are written in the same transaction (NFR-WF-AUD-001). If the entry cannot be written, the change is rolled back.

**Related:** [Use cases](02-use-cases.md) · [Acceptance criteria](04-acceptance-criteria.md) · [Shared business rules](../requirements/business-rules.md)
