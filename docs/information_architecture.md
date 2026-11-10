# Information Architecture

Screen layouts, role-based navigation hierarchy, data models, and API specifications for foundational modules and the **Dashboard** subsystem. Business rules and permissions refer to [mvp.md](mvp.md); interaction flows refer to [use_cases.md](use_cases.md) and acceptance test scenarios to [user_stories.md](user_stories.md).

---

## 1. Sitemap & Route Hierarchy

```
/login                         Authentication entrypoint (Public)
/login/2fa                     Two-Factor TOTP challenge step (Public)
/login/callback?code=…         Microsoft Entra ID OAuth2 callback (Public)
/forgot-password               Password recovery request (Public)
/reset-password?token=…        Password reset completion (Public, token-guarded)
/                              Root route (Auto-redirects based on user role)

/dashboard                     Dashboard Subsystem (Role-based auto-routing)
  ├─ /admin/dashboard          Global Operations & Management Dashboard (ADMIN)
  └─ /technician/dashboard     Engineering Team & Personal Workspace (TECHNICIAN, TECHNICIAN_LEAD)

/profile                       User Profile & Settings (Authenticated users)
  ├─ Account Details
  ├─ Change Password           (Enforced when must_change_password; hidden for SSO-only accounts)
  ├─ Security (2FA)
  └─ Roles & Permissions Summary

/admin/users                   User Management (ADMIN)
  ├─ /admin/users/new          Create User
  └─ /admin/users/:id          Edit User & Role/Group Assignment
/admin/groups                  Engineering Teams Management (ADMIN)
  └─ /admin/groups/:id         Team Details & Member Management
/admin/microsoft-sync          Microsoft Entra ID Mapping & Sync (ADMIN)
/admin/service-accounts        AI Service Account & API Key Management (ADMIN)
/admin/ai-suggestions          AI Routing & Skill Improvement Suggestions (ADMIN)
```

---

## 2. Navigation Menu by Role

| Navigation Item | ADMIN | TECHNICIAN | REQUESTER |
|---|---|---|---|
| **Dashboard** | ✅ *(Global Enterprise View)* | ✅ *(Team & Personal View)* | ❌ |
| Tickets *(External Module)* | All Tickets | Assigned / Group Tickets | My Submitted Tickets |
| User Management | ✅ | ❌ | ❌ |
| Engineering Teams | ✅ | ❌ | ❌ |
| Microsoft Entra Sync | ✅ | ❌ | ❌ |
| AI Service Accounts | ✅ | ❌ | ❌ |
| AI Suggestions (with pending badge) | ✅ | ❌ | ❌ |
| User Profile | ✅ | ✅ | ✅ |
| Sign Out | ✅ | ✅ | ✅ |

*Default landing page upon authentication:*
* `ADMIN` $\rightarrow$ `/admin/dashboard`
* `TECHNICIAN` $\rightarrow$ `/technician/dashboard`
* `REQUESTER` $\rightarrow$ `/my-tickets` (My Tickets)

When `must_change_password = true`, all routes redirect to `/profile` (Change Password tab). `AI_AGENT` has no graphical interface (API-only).

---

## 3. Screen & Widget Specifications

| Screen / Widget | Core Components & Data Rendered |
|---|---|
| **Executive Dashboard** (`/admin/dashboard`) | • **Control Bar:** Date range picker (Today, Last 7 Days, Last 30 Days, Custom Range), Engineering Team filter dropdown, Instant Refresh button, WebSocket status indicator.<br>• **Summary KPI Cards:** Total Ingested Tickets, New (`OPEN`), In Progress (`IN_PROGRESS`), Waiting for Info (`NEEDS_INFO`), Resolved (`RESOLVED`), Autonomous AI Resolution Rate (%).<br>• **Distribution Charts:** Priority breakdown donut chart (P1–P4), incident category bar chart, hourly ticket volume intake trend line.<br>• **SLA & Bottleneck Watch:** Top 10 critical tickets (live countdown timers, overdue badges), quick-reassign actions.<br>• **Workload Capacity Matrix:** Engineer utilization bars grouped by team, capacity indicators (Green: Normal, Yellow: Busy, Red: Overloaded).<br>• **Live Incident Stream:** Flashing critical P1 banner, ticket surge/spike alert, AI workflow failure notifications.<br>• **Deep Drill-down:** All cards and charts link directly to pre-filtered queries at `/tickets`. |
| **Technician Workbench** (`/technician/dashboard`) | • **My Queue Today:** Assigned tickets requiring immediate attention, sorted by priority and nearest SLA deadline.<br>• **Unassigned Team Queue:** Group queue of unassigned tickets awaiting pickup with a 1-click "Claim Ticket" action.<br>• **Personal SLA Countdown:** Live countdown timers for the technician's active assignments.<br>• **Productivity Metrics:** Daily/weekly resolved ticket count, average CSAT satisfaction rating.<br>• **Personal Capacity Meter:** Individual workload gauge compared against team average. |
| **Sign In** | Microsoft SSO button; Email & Password inputs; Submit button; Forgot password link; Account lockout & security error prompts |
| **Microsoft OAuth Callback** | Loading state; code exchange for JWT session tokens; route redirection based on role / 2FA status |
| **2FA Verification** | 6-digit TOTP input, verify button, rate-limiting lockout message |
| **User Directory** | Search bar; filters (role, status, team, skills); data table (name, email, role, team, skills, auth provider, status); user creation trigger; toggle status |
| **User Detail / Edit** | Email (immutable after creation), full name, phone, role, team, skills (for technicians), temporary password; reset 2FA button; Microsoft sync lock indicators |
| **Microsoft Entra Mapping** | Mapping table (Entra Group ID $\rightarrow$ System Role / Engineering Team); Add mapping modal; "Sync Now" trigger; Last sync run telemetry |
| **Engineering Teams** | Team list, description, assigned categories, member count, open ticket count; Add/Edit team modal |
| **AI Service Accounts** | Service account list (name, status, last active, created date); Generate key button (one-time display modal); Rotate / Revoke key triggers |
| **AI Suggestions** | Filter by type/status; proposal cards (target team/user, suggested change, confidence score, evidence tickets); Approve / Reject triggers |
| **Profile Settings** | Profile info tab; Change password tab; 2FA configuration tab; Permissions and roles explainer tab |

---

## 4. Core Domain Entities & Data Models

| Entity / Model | Core Fields |
|---|---|
| `DashboardMetricCache` | cache_key (hash of range + scope + group_id), total_tickets, open_count, in_progress_count, needs_info_count, resolved_count, ai_resolved_count, ai_resolution_rate, priority_breakdown (json), category_breakdown (json), hourly_trend (json), computed_at, expires_at |
| `TechnicianWorkload` | technician_id, group_id, active_ticket_count, p1_p2_count, workload_index, capacity_status (`NORMAL` / `BUSY` / `OVERLOADED`), last_assigned_at, updated_at |
| `SystemAlert` | id, type (`CRITICAL_P1` / `SPIKE_DETECTED` / `WORKFLOW_ERROR` / `SLA_BREACH`), severity (`HIGH` / `CRITICAL`), title, summary, related_ticket_ids[], status (`ACTIVE` / `ACKNOWLEDGED` / `RESOLVED`), created_at, resolved_at |
| `User` | id, email (unique), password_hash, microsoft_oid, full_name, phone, avatar_url, role, role_source (`MANUAL` / `MICROSOFT`), status (`ACTIVE` / `DISABLED`), disabled_reason, skills[], last_synced_at, must_change_password, totp_secret_enc, totp_enabled, failed_login_count, locked_until, last_login_at, created_at, updated_at |
| `TechnicianGroup` | id, name (unique), description, categories[], created_at, updated_at |
| `GroupMember` | group_id, user_id, source (`MANUAL` / `MICROSOFT`), joined_at |
| `MicrosoftGroupMapping` | id, entra_group_id, entra_group_name, target_type (`ROLE` / `TECHNICIAN_GROUP`), target_role, technician_group_id, created_at, updated_at |
| `MicrosoftSyncRun` | id, trigger (`LOGIN` / `SCHEDULED` / `MANUAL`), started_at, finished_at, users_processed, changes_count, removed_users[], errors[] |
| `RefreshToken` | id, user_id, token_hash, expires_at, revoked_at |
| `PasswordResetToken` | id, user_id, token_hash, expires_at, used_at |
| `ServiceAccount` | id, name, api_key_hash, role = `AI_AGENT`, status (`ACTIVE` / `DISABLED`), last_used_at, created_at |
| `AiSuggestion` | id, type (`GROUP_CATEGORY` / `USER_SKILL`), target_id, operation (`ADD` / `REMOVE`), value, reason, confidence (0–1), evidence_ticket_ids[], status (`PENDING` / `APPROVED` / `REJECTED`), created_by, reviewed_by, review_note, reviewed_at, created_at |
| `AuditLog` | id, actor_id, actor_type (`USER` / `AI` / `SYSTEM`), action, target_type, target_id, before, after, suggestion_id, created_at |

---

## 5. API Endpoints Catalog

Base path: `/api/v1`. Users authenticate via `Authorization: Bearer <access_token>`; AI agents authenticate via `X-API-Key`. Standard error format: `{ "code": "...", "message": "..." }`.

### 5.1. Dashboard API
| Method | Path | Permission / Role | Description |
|---|---|---|---|
| `GET` | `/dashboard/summary` | `dashboard:read` (`ADMIN`, `TECHNICIAN`) | Retrieve aggregated KPIs, AI resolution stats, and distribution charts (scoped to role). |
| `GET` | `/dashboard/sla-breached` | `dashboard:read` (`ADMIN`, `TECHNICIAN`) | List critical tickets approaching deadline (warning) or currently overdue (breached). |
| `GET` | `/dashboard/workload` | `dashboard:workload:read` (`ADMIN`, `TECHNICIAN_LEAD`) | Fetch technician workload matrix and capacity indicators grouped by team. |
| `GET` | `/dashboard/alerts` | `dashboard:read` (`ADMIN`, `TECHNICIAN`) | Retrieve all currently active system incidents and alerts (`ACTIVE`). |
| `POST` | `/dashboard/alerts/:id/ack` | `dashboard:write` (`ADMIN`, `TECHNICIAN`) | Acknowledge receipt of a critical incident alert. |
| `WS` | `/dashboard/live-feed` | `dashboard:read` (`ADMIN`, `TECHNICIAN`) | WebSocket channel streaming real-time ticket creation and high-priority incident events. |

### 5.2. Authentication & Administration API
| Method | Path | Permission | Description |
|---|---|---|---|
| `POST` | `/auth/login` | Public | Authenticate via email + password (returns tokens or 2FA challenge). |
| `POST` | `/auth/login/2fa` | Challenge token | Verify 6-digit TOTP code. |
| `GET` | `/auth/microsoft/login` | Public | Initiate Microsoft Entra OAuth2 flow. |
| `GET` | `/auth/microsoft/callback` | Public | Microsoft OAuth2 authorization callback. |
| `POST` | `/auth/microsoft/exchange` | Public | Exchange one-time auth code for session tokens. |
| `POST` | `/auth/refresh` | Public | Issue a new access token via refresh token. |
| `POST` | `/auth/logout` | Authenticated | Revoke active refresh token and sign out. |
| `POST` | `/auth/forgot-password` | Public | Request password recovery email. |
| `POST` | `/auth/reset-password` | Public | Complete password reset via verification token. |
| `GET` | `/users` | `user:read` | List/search users with pagination and filters. |
| `POST` | `/users` | `user:create` | Provision a new user account. |
| `GET / PATCH` | `/users/:id` | `user:read` / `user:update` | Inspect or update user details. |
| `PATCH` | `/users/:id/status` | `user:disable` | Enable or disable a user account. |
| `PATCH` | `/users/:id/role` | `role:assign` | Reassign user roles. |
| `POST` | `/users/:id/2fa/reset` | `user:update` | Admin reset of user 2FA configuration. |
| `GET / POST` | `/groups` | `group:read` / `group:manage` | List or create engineering teams. |
| `GET / PATCH / DELETE` | `/groups/:id` | `group:read` / `group:manage` | Read, update, or delete engineering team. |
| `POST / DELETE` | `/groups/:id/members[/:userId]` | `group:manage` | Add or remove members from team. |
| `GET / POST` | `/service-accounts` | `service_account:manage` | Manage AI service accounts. |
| `PATCH` | `/service-accounts/:id/status` | `service_account:manage` | Activate or suspend an AI service account. |
| `POST` | `/service-accounts/:id/rotate-key` | `service_account:manage` | Rotate service account API key. |
| `GET / POST` | `/microsoft-sync/mappings` | `microsoft_sync:manage` | Manage Microsoft group mapping rules. |
| `DELETE` | `/microsoft-sync/mappings/:id` | `microsoft_sync:manage` | Delete a group mapping rule. |
| `GET` | `/microsoft-sync/entra-groups?search=…` | `microsoft_sync:manage` | Search groups via Microsoft Graph. |
| `POST` | `/microsoft-sync/run` | `microsoft_sync:manage` | Trigger an immediate Microsoft directory sync. |
| `GET` | `/microsoft-sync/runs` | `microsoft_sync:manage` | View directory sync execution history. |
| `POST` | `/ai-suggestions` | `ai_suggestion:create` | AI submits operational tuning proposals. |
| `GET` | `/ai-suggestions` | `ai_suggestion:review` | Review pending AI suggestions. |
| `POST` | `/ai-suggestions/:id/approve` | `ai_suggestion:review` | Approve and apply an AI suggestion. |
| `POST` | `/ai-suggestions/:id/reject` | `ai_suggestion:review` | Reject an AI suggestion with feedback. |
| `GET / PATCH` | `/me` | `profile:read:self` / `profile:update:self` | Retrieve or edit personal profile details. |
| `POST` | `/me/password` | `profile:update:self` | Change account password. |
| `POST` | `/me/2fa/setup` / `enable` / `disable` | `profile:update:self` | Manage personal 2FA setup. |
| `GET` | `/me/permissions` | `profile:read:self` | Inspect active permissions and granted scopes. |
