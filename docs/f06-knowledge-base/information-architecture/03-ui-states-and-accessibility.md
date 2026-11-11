# UI States and Accessibility — F06 Knowledge Base

> Design status: TARGET V1
>
> Related: [IA overview](README.md) · [Screens and navigation](01-screens-and-navigation.md) · [User flows](02-user-flows.md)

---

## 1. Article Lifecycle Status Chips

Every article card, detail header, and table row displays its lifecycle state using standardized status chips with explicit text labels and icons (never color alone):

| State Code | Display Label | Visual Token | Where Visible | Permitted Actions |
|---|---|---|---|---|
| `DRAFT` | Draft | Neutral Slate (`#64748B`) + Pen icon | Author, Reviewers | Edit, Delete, Submit for Review |
| `PENDING_REVIEW` | Pending Review | Amber (`#D97706`) + Clock icon | Reviewers, Author | View Diff, Approve, Request Changes, Reject |
| `CHANGES_REQUESTED` | Changes Requested | Orange (`#EA580C`) + Alert icon | Author, Reviewers | Amend draft, Re-submit |
| `PUBLISHED` | Published | Emerald (`#059669`) + Check icon | Public (All Roles) | Read, Search, Fork Revision (`v1.1 Draft`), Rate |
| `FLAGGED` | Under Revision | Rose (`#E11D48`) + Flag icon | All Roles (Warning Banner) | Read (with warning), Assign to Author, Deprecate |
| `ARCHIVED` | Archived | Dim Slate (`#94A3B8`) + Archive icon | Admins Only | Read-only audit historical reference |

---

## 2. Component UI State Contract

Every screen and widget in F06 implements an explicit state machine:

| State | Trigger / Scenario | Visual UX Representation |
|---|---|---|
| `LOADING` | Executing semantic vector search or fetching article revision | Skeleton wireframes matching exact line shapes with subtle pulsing shimmer (max duration budget: 1.5s for search, 5.0s for AI synthesis). |
| `READY` | Content successfully loaded | High-contrast rendered Markdown, code blocks with syntax highlighting, and responsive media. |
| `EMPTY` | Search returns zero matches above similarity threshold (0.50) | Friendly illustration with copy: *"No validated solutions match your query."* Prompts primary action button: **[+ Create Support Ticket]**. |
| `SAVING` (Auto-save) | Background debounce trigger (every 20s during editing) | Subtle inline pulse: *"Auto-saving draft..."* transitions to *"Draft saved at HH:mm:ss"*. Does not interrupt active typing. |
| `SUCCESS` | Successful publication or report submission | Auto-dismissing toast notification (5 seconds) with checkmark icon in `#057A55` (Green-600). |
| `ERROR` / `TIMEOUT` | AI extraction service timeout or network error | Non-blocking inline banner with retry action: *"AI synthesis service is currently busy. [Retry] or [Author Manually]"*. |

---

## 3. Visual Diff Inspection Contract (`SCR-KB-05`)

The Visual Diff component allows reviewers to inspect exact character and line-level changes between the current active version (e.g. `v1.1`) and the proposed revision (`v1.2`):

```text
DIFF INSPECTION VISUAL TOKENS
├── Additions:       Background: #DEF7EC (Green-100), Text: #03543F (Green-900), Prefix: "+"
├── Deletions:       Background: #FDE8E8 (Red-100), Text: #9B1C1C (Red-900), Strikethrough, Prefix: "-"
└── Unchanged:       Standard body text (#374151)
```

*   **Diff Navigation:** Sticky navigation bar enables reviewers to jump between modified sections using `[Previous Change]` and `[Next Change]` hotkeys (`[J]` / `[K]`).
*   **Media Diffing:** For screenshot replacements, the viewer provides a 50/50 slider overlay allowing reviewers to drag across images to inspect interface changes.

---

## 4. Accessibility (WCAG 2.1 AA Compliance)

1.  **Color Contrast:** All body text maintains a contrast ratio of at least `4.5:1` against white and dark slate backgrounds; large headers maintain at least `3:1`.
2.  **Keyboard Accessibility:** 
    *   Search portal (`/kb`) opens with global hotkey `Cmd+K` / `Ctrl+K`.
    *   Diff modal traps focus inside the dialog; pressing `Escape` closes the modal with confirmation prompt.
    *   All code block copy buttons are navigable and operable via `Tab` + `Enter`.
3.  **Screen Reader Optimization (ARIA):**
    *   Status badges announce state with `aria-label="Status: Pending Review"`.
    *   Search results list uses `aria-live="polite"` to announce result counts upon query completion without interrupting reader narration.
    *   Step-by-step fix procedures use semantic `<ol>` ordered lists with discrete `<section>` containers.
