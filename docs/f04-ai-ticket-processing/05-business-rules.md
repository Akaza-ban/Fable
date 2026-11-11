# Business Rules — F04 AI Ticket Processing

> Design status: TARGET V1
>
> Implementation status: NOT STARTED

The rules of F05 (`BR-WF-xx`) still apply; in particular BR-WF-11 … BR-WF-15 govern the execution and verification that F04 now owns. The rules below cover only what F04 adds. BR-AI-02 (versioned analyses) and BR-AI-09 (priority) are retired and handed to F05 (see [Handover](README.md#handover-to-other-branches)).

## BR-AI-01 — The agent result must match its contract

- **Source:** UC-AI-10, *Agent result contract* in [02-use-cases.md](02-use-cases.md)
- **Stories:** US-AI-20
- **Rule:** Every agent result is validated before it is stored. A result that does not match the schema, uses a value outside the lists, contains an ungrounded value (BR-AI-03), proposes an action outside the agent's allowed actions or F05's whitelist, or escalates without exactly one class is stored as `INVALID`. Stored results are never edited; a new result is a new version. F05 handles `INVALID` and `FAILED` like a ticket that needs a human.

## BR-AI-03 — Grounding: no invented facts

- **Source:** UC-AI-10, UC-AI-08, UC-AI-09, UC-AI-11
- **Stories:** US-AI-20, US-AI-16, US-AI-18, US-AI-26
- **Rule:** Every value in the domain form and in a proposed action must have a source quote that appears in the ticket text, and a quote for a person, system, permission or account must resolve to a record in the directory (F09) or the system catalog. A reply or Resolution states only facts from the ticket, the technician's notes, the run data, the guidance transcript or a cited article. A requirement brief lists unstated points under *Open points*. An output that breaks this is not shown or sent.

## BR-AI-04 — Ticket text is data, not instructions

- **Source:** UC-AI-10, UC-AI-11
- **Stories:** US-AI-20, US-AI-26
- **Rule:** The AI never follows instructions written in a ticket, message or attachment name. Text that tries to change the AI's behaviour gets `reason = INJECTION_SUSPECTED` and is escalated; no action is proposed for it. Even without this flag, nothing the AI writes can change an external system: only an approved decision can, and only with its own parameters (BR-AI-15).

## BR-AI-05 — KB answers and steps must cite a published article

- **Source:** UC-AI-03, UC-AI-11
- **Stories:** US-AI-05, US-AI-06, US-AI-17, US-AI-26
- **Rule:** Only **published** articles are searched. Every solution or guidance step cites an article and section. When no article reaches the relevance threshold, the answer is "No matching article in the knowledge base". The AI never writes a solution without a source.

## BR-AI-06 — The AI never asks for secrets

- **Source:** UC-AI-10, UC-AI-11
- **Stories:** US-AI-20, US-AI-26
- **Rule:** A question or guidance step from the AI never asks for a password, OTP, recovery code or private key, and never asks again for information already in the ticket. F05 applies the same rule to its ask-back questions.

## BR-AI-07 — Mask credentials before any model call

- **Source:** UC-AI-10, UC-AI-11, UC-AI-08
- **Stories:** US-AI-20, US-AI-26, US-AI-16
- **Rule:** Strings that look like passwords, OTP codes, API keys or tokens are replaced with `[MASKED]` in the stored ticket text, in the input of every model call, and in every AI output. This applies whichever model provider is chosen (Q-AI-5).

## BR-AI-08 — Suggestions respect the viewer's scope

- **Source:** UC-AI-03, UC-AI-10
- **Stories:** US-AI-05, US-AI-21
- **Rule:** Results are filtered by the viewer's permissions before they are shown. A requester sees only articles for all users and only their own tickets. A technician sees only tickets they may open (F01 US-03), including in a suspected-bug burst. The filter runs in the API, not in the UI.

## BR-AI-10 — The AI speaks for itself only; people and F05 decide

- **Source:** UC-AI-03, UC-AI-07, UC-AI-08, UC-AI-10, UC-AI-11
- **Stories:** US-AI-06, US-AI-14, US-AI-16, US-AI-17, US-AI-20, US-AI-26
- **Rule:** The AI may send **its own** messages, always with author `AI`: KB auto-replies where enabled (UC-AI-03), guidance steps (UC-AI-11), and result replies after a verified execution (UC-AI-08). It never sends a message **in a technician's name**: drafts are sent only after the technician confirms. It never approves a request, never links or merges tickets, and never re-assigns a ticket; it hands a ticket back to F05 with an escalation class or `WRONG_DOMAIN`. A correction by a technician creates a new result version, which never starts an action or a re-route.

## BR-AI-11 — No credentials in replies or Resolutions

- **Source:** UC-AI-08, UC-AI-12
- **Stories:** US-AI-16, US-AI-22
- **Rule:** A reply, Resolution, guidance step or requirement brief written by the AI never contains a password, temporary password, OTP or token, even for `CREATE_ACCOUNT`. Credentials reach the user only through the connector's own secure channel (BR-WF-15).

## BR-AI-12 — Learning examples come only from verified tickets

- **Source:** UC-AI-07, UC-AI-10, UC-AI-08
- **Stories:** US-AI-19, US-AI-28
- **Rule:** A closed ticket becomes a learning example only if a technician resolved it or confirmed the agent's handling, or the requester confirmed an AI resolution, and it was not reopened for 7 days. A requester's confirmation is enough: it means the problem is solved. A ticket flagged *Resolved incorrectly* (BR-AI-16) never becomes an example, and an example already made from it is excluded. Examples are stored masked (BR-AI-07). They are used only as model input; no text from an example is ever shown to another user. An Admin can exclude any example, and a new example set is used only if the results on ES-AI-07 do not get worse.

## BR-AI-13 — Agent first; people only for three classes

- **Source:** UC-AI-10 (decision D-AI-01)
- **Stories:** US-AI-17, US-AI-20, US-AI-21, US-AI-26
- **Rule:** Every ticket that F05 routes to an AI agent is first handled by that agent, which resolves it by an allowed action (approved, then executed and verified by F04), a cited KB answer, or guided troubleshooting. A ticket goes to a technician only with exactly one `escalationClass`: `AGENT_CANNOT_RESOLVE`, `SYSTEM_BUG` or `SOFTWARE_REQUIREMENT`, plus a reason. The agent never tries to resolve a suspected bug or a software requirement itself. Escalating is always allowed: when in doubt, the agent escalates rather than guesses.

## BR-AI-14 — Guidance only with safe self-service steps

- **Source:** UC-AI-11
- **Stories:** US-AI-26, US-AI-27
- **Rule:** Guided troubleshooting uses only steps from published articles marked **self-service**, one step at a time. It never offers a step that needs admin rights, deletes or overwrites data, changes security settings, or installs software from outside the company catalog, and it never takes remote control. A session ends and escalates after the procedure ends without a fix, after 5 rounds, when the requester asks for a person, or after 24 hours without an answer (limits proposed, Q-AI-15).

## BR-AI-15 — Execute exactly what was approved

- **Source:** UC-AI-12, UC-AI-13
- **Stories:** US-AI-22, US-AI-24, US-AI-25
- **Rule:** F04 calls a connector only for an approved decision, with the operation and parameters **stored in that decision**; nothing the AI writes after the approval can change them. Each decision and action runs at most once in the external system (idempotency key `decisionId + action`, also on *Retry*). Before every execution and retry, the current whitelist and catalog are re-checked (BR-WF-12), and a ticket becomes `RESOLVED` only after a successful check or a technician's note (BR-WF-13).

## BR-AI-16 — Review scope for tickets the agent resolved alone

- **Source:** UC-AI-07
- **Stories:** US-AI-28
- **Rule:** A ticket that an agent resolved without a technician (KB answer, guided fix, verified execution) can be reviewed by the technicians of that agent's default escalation group and by the Admin. The review scope gives **read and feedback only**: *Looks right*, or *Resolved incorrectly* with a required reason. It never changes the ticket's status, assignment or Resolution; reopening is done by the Admin or the requester in F03. *Resolved incorrectly* notifies the Admin and keeps the ticket out of the learning examples (BR-AI-12). The API enforces the scope (F01 ticket scope, extended for this case).

**Related:** [Use cases](02-use-cases.md) · [User stories](03-user-stories-invest.md) · [Acceptance criteria](04-acceptance-criteria.md)
