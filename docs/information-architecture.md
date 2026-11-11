# Master Information Architecture (IA) — Enterprise System

> System: **AI-Powered IT Ticket Routing and Automated Resolution System**
>
> Scale: ~8,000 requests/day across enterprise departments
>
> Design Status: TARGET V1

---

## 1. Information Architecture & Navigation Hierarchy (Relume Sitemap)

```text
PUBLIC ACCESS LAYER
└── /login                                  - SSO Corporate Authentication Portal (Google / Azure AD / Demo Role Switcher)

AUTHENTICATED WORKSPACE SHELL (Layout: Fixed 260px Collapsible Sidebar + Sticky 64px Global Topbar)
│
├── [REQUESTER SELF-SERVICE PORTAL]
│   ├── /portal                             - Requester Home, announcements, and quick self-help search
│   ├── /requests/new                       - Smart ticket draft form with real-time AI solution deflection
│   ├── /requests/my-tickets                - Requester personal tickets, progress timeline & verification status
│   └── /requests/:id                       - Ticket conversation, clarifying question answers, feedback rating
│
├── [TICKET & TECHNICIAN WORKSPACE (F03)]
│   ├── /tickets/inbox                      - Team queues, assigned tickets, triage unassigned queue
│   ├── /tickets/:id                        - Unified technician console (AI summary, conversation, internal notes)
│   │   ├── /tickets/:id/history            - Full ticket lifecycle and audit events
│   │   └── /tickets/:id/diff               - Field modification diff comparison
│   └── /tickets/escalations                - Approaching SLA breach tickets & department reassignments
│
├── [AI PROCESSING & ROUTING (F04)]
│   ├── /ai/classifications                 - Real-time classification feed, intent detection, confidence telemetry
│   ├── /ai/clarification-queue             - Active clarification interactions with requesters
│   └── /ai/model-feedback                  - Technician category overrides & continuous learning dataset
│
├── [WORKFLOW & AUTOMATION ENGINE (F05)]
│   ├── /automation/rules                   - Assignment rules builder (Round-robin, workload balance, team matching)
│   ├── /automation/sla-policies            - Priority-based SLA policies (P1–P4) & business hour calendar
│   ├── /automation/scope-whitelist         - AI auto-approval action whitelist & department entitlement matrix
│   └── /automation/executions              - Automated API action runs, execution status & fallback alerts
│
├── [KNOWLEDGE MANAGEMENT — KB (F06)]
│   ├── /kb/explore                         - Semantic solution search portal & faceted category hierarchy
│   ├── /kb/articles/:id                    - Article view (Problem, root cause, step-by-step fix, attachments)
│   ├── /kb/editor                          - Rich Text/Markdown authoring tool, code blocks & screenshot uploads
│   ├── /kb/ai-inbox                        - AI-synthesized drafts extracted from resolved tickets (1-click draft)
│   ├── /kb/approvals                       - Side-by-side Visual Diff inspection for pending article updates
│   └── /kb/flagged                         - Problematic & outdated article maintenance queue
│
├── [GOVERNANCE, ANALYTICS & AUDIT (F07 / F08)]
│   ├── /analytics/executive-dashboard      - Executive KPI telemetry, AI deflection rate, MTTA, MTTR, SLA %
│   ├── /analytics/team-workload            - Support team capacity, ticket queue depth & technician metrics
│   ├── /audit/activity-log                 - Immutable security and workflow audit trail (SOC-2 compliant)
│   └── /admin/users-rbac                   - Enterprise user directory, team assignments & role matrix
│
└── [SYSTEM EXCEPTION & FALLBACK]
    ├── /403                                - Access Forbidden (Role-Based Access Control fallback guard)
    └── /404                                - Page Not Found (Routing error fallback)
```

---

## 2. Standard Screen Hierarchy & Component States

To ensure UI consistency across all modules, every authenticated view implements standardized states:

| Component State | UX Behavior & Visual Indicator |
| :--- | :--- |
| **Loading State** | Skeleton loaders mirroring exact layout shapes; subtle pulsing shimmer; no disruptive full-screen spinners. |
| **Empty State** | Contextual icon, clear explanatory copy ("No tickets awaiting assignment"), and primary call-to-action button. |
| **Validation State** | Real-time inline field validation on blur; error text in `#E02424` (Red-600); input border highlighted. |
| **Saving / Processing** | Button state shows localized spinner, disabled interaction, and label changes (e.g., *"Publishing v1.2..."*). |
| **Success State** | Auto-dismissing toast notification (5 seconds) with checkmark icon in `#057A55` (Green-600). |
| **Error / Network Fail** | Non-blocking alert banner with explicit error description and a **[Retry Action]** button. |
| **403 Access Denied** | Dedicated view showing required permission role and button to navigate back to `/portal`. |

---

## 3. Global Role-Based Navigation Matrix

The navigation bar dynamically renders navigation items based on the user's authenticated role token:

| Module / Route Group | `REQUESTER` | `TECHNICIAN` | `TEAM_LEAD` | `ADMIN` |
| :--- | :---: | :---: | :---: | :---: |
| **Self-Service Portal** (`/portal`, `/requests`) | Full Access | Full Access | Full Access | Full Access |
| **Ticket Management** (`/tickets/*`) | Hidden / 403 | Assigned / Team | Full Team Queue | Full Access |
| **AI Routing & Classification** (`/ai/*`) | Hidden / 403 | View & Suggest | View & Review | Full Config |
| **Workflow & Automation Config** (`/automation/*`) | Hidden / 403 | Hidden / 403 | View Rules / Dry-run | Full Config |
| **Knowledge Base Authoring** (`/kb/editor`, `/kb/ai-inbox`) | Read-only | Full Authoring | Author & Review | Full Admin |
| **Knowledge Base Approvals** (`/kb/approvals`) | Hidden / 403 | Hidden / 403 | Approve / Reject | Full Admin |
| **Executive Analytics & Reports** (`/analytics/*`) | Hidden / 403 | Hidden / 403 | Team Metrics | Enterprise KPIs |
| **Audit Log & User Management** (`/audit`, `/admin`) | Hidden / 403 | Hidden / 403 | Hidden / 403 | Full Admin |
