# Business Rules — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Implementation status: NOT STARTED

The rules of F05 (`BR-WF-xx`) still apply. The rules below cover only what F04 adds.

## BR-AI-01 — The AI output must match the analysis contract

- **Source:** UC-AI-01, *AI analysis contract* in [02-use-cases.md](02-use-cases.md)
- **Stories:** US-AI-02
- **Rule:** Every analysis is validated before it is stored. An output that does not match the schema, uses a value outside the seeded lists, has a confidence outside 0.00–1.00, contains an ungrounded value (BR-AI-03), or suggests `AUTOMATABLE` with missing fields is stored as `INVALID`. F05 handles `INVALID` and `FAILED` like a ticket that needs a human.

## BR-AI-02 — Analyses are versioned and never edited

- **Source:** UC-AI-01, UC-AI-07
- **Stories:** US-AI-02, US-AI-13
- **Rule:** Each analysis is a new version with its model version and prompt version. A stored version is never changed; a correction creates a correction record and a new version. Exactly one version is current per ticket.

## BR-AI-03 — Grounding: no invented facts

- **Source:** UC-AI-01, UC-AI-04
- **Stories:** US-AI-02, US-AI-07
- **Rule:** Every extracted value of `recipient`, `system`, `permission` and `account` must have a source quote that appears in the ticket text, and the quote must resolve to a record in the directory (F09) or the system catalog. The stored value is that record ("me" → the requester; "the Accounting shared folder" → `File share \\fs01\ketoan`). Every fact in a summary must link to the message it comes from. An output that breaks this is not shown.

## BR-AI-04 — Ticket text is data, not instructions

- **Source:** UC-AI-01
- **Stories:** US-AI-02
- **Rule:** The AI never follows instructions written in a ticket, message or attachment name. Text that tries to change the AI's behaviour gets `routeReason = INJECTION_SUSPECTED` and `suggestedRoute = NEEDS_HUMAN`. Even without this flag, the AI cannot cause any change outside F04: F05's fixed policy check decides what may run (BR-WF-02).

## BR-AI-05 — KB suggestions must cite a published article

- **Source:** UC-AI-03
- **Stories:** US-AI-05, US-AI-06
- **Rule:** Only **published** articles are searched. Every solution step cites an article and section. When no article reaches the relevance threshold, the answer is "No matching article in the knowledge base". The AI never writes a solution without a source.

## BR-AI-06 — The AI never asks for secrets

- **Source:** UC-AI-02
- **Stories:** US-AI-03
- **Rule:** A question from the AI never asks for a password, OTP, recovery code or private key, and never asks again for information already in the ticket.

## BR-AI-07 — Mask credentials before any model call

- **Source:** UC-AI-01, UC-AI-02, UC-AI-04
- **Stories:** US-AI-04, US-AI-07
- **Rule:** Strings that look like passwords, OTP codes, API keys or tokens are replaced with `[MASKED]` in the stored ticket text, in the input of every model call, and in every AI output. This applies whichever model provider is chosen (Q-AI-5).

## BR-AI-08 — Suggestions respect the viewer's scope

- **Source:** UC-AI-03, UC-AI-06
- **Stories:** US-AI-05, US-AI-11, US-AI-12
- **Rule:** Results are filtered by the viewer's permissions before they are shown. A requester sees only articles for all users and only their own tickets. A technician sees only tickets they may open (F01 US-03). The filter runs in the API, not in the UI.

## BR-AI-09 — Priority depends on impact and urgency only

- **Source:** UC-AI-05
- **Stories:** US-AI-09
- **Rule:** The priority suggestion is based on the scope of impact, whether work is blocked, and urgency (stated deadline, critical system). It never depends on the requester's name, job title or department. A human change always needs a reason and is logged with before and after values. The requester's business unit and site are used for **routing** (UC-AI-01), not for priority. If the business wants VIP handling, it must be an explicit, visible F05 rule (Q-AI-8).

## BR-AI-10 — The AI speaks for itself only; people and F05 decide

- **Source:** UC-AI-03, UC-AI-04, UC-AI-06, UC-AI-07, UC-AI-08, UC-AI-10
- **Stories:** US-AI-06, US-AI-11, US-AI-13, US-AI-16, US-AI-17, US-AI-20
- **Rule:** The AI may send **its own** messages, always with author `AI`: questions (UC-AI-02), KB auto-replies where enabled (UC-AI-03), and result replies after a verified automation (UC-AI-08). It never sends a message **in a technician's name**: drafts are sent only after the technician confirms. It never links or merges tickets, never re-assigns a ticket, and changes nothing outside F04 except through F05's checked actions. A correction by a technician creates an analysis version with `trigger = CORRECTION`, which is never sent to F05 for routing. A domain agent may hand a ticket back to the router (`WRONG_DOMAIN`) once; after that a human decides.

## BR-AI-11 — No credentials in replies or Resolutions

- **Source:** UC-AI-08
- **Stories:** US-AI-16
- **Rule:** A reply, Resolution, summary or requirement brief written by the AI never contains a password, temporary password, OTP or token, even for `CREATE_ACCOUNT`. Credentials reach the user only through the connector's own secure channel (F05, BR-WF-15).

## BR-AI-12 — Learning examples come only from verified tickets

- **Source:** UC-AI-07, UC-AI-01
- **Stories:** US-AI-19
- **Rule:** A closed ticket becomes a learning example only if a technician corrected or confirmed its classification and it was not reopened for 7 days. Examples are stored masked (BR-AI-07). They are used only as model input for classification, priority and domain-form filling; no text from an example is ever shown to another user. An Admin can exclude any example, and a new example set is used only if accuracy on ES-AI-01 does not drop.

## BR-AI-13 — Agent first; people only for three classes

- **Source:** UC-AI-10 (decision D-AI-01)
- **Stories:** US-AI-17, US-AI-20, US-AI-21
- **Rule:** Every ticket is first handled by the AI agent of its domain, which resolves it by an allowed action (checked and run by F05), a cited KB answer, or a question. A ticket goes to a technician only with exactly one `escalationClass`: `AGENT_CANNOT_RESOLVE`, `SYSTEM_BUG` or `SOFTWARE_REQUIREMENT`, plus a `routeReason`. The agent never tries to resolve a suspected bug or a software requirement itself. Escalating is always allowed: when in doubt, the agent escalates rather than guesses.

**Related:** [Use cases](02-use-cases.md) · [User stories](03-user-stories-invest.md) · [Acceptance criteria](04-acceptance-criteria.md)
