# AI-Powered IT Ticket Routing & Automated Resolution System

# 1. Requester Self-Service (Người gửi ticket)
## 1.1 Create Ticket & Attach Evidence
### Describe symptoms & error messages
### Upload screenshots & client log files
### Select initial perceived category
## 1.2 View Ticket Status & Timeline
### Real-time status progression (New -> In Progress -> Resolved)
### Historical list of submitted tickets
## 1.3 Communication & Clarification
### Answer AI clarification questions (max 2 rounds)
### Direct messaging & comments with support technician
## 1.4 Self-Service Solution Verification
### View AI-suggested KB articles (Instant Deflection)
### Confirm resolution or request escalation
## 1.5 Confirm Resolution / Reopen Ticket
### Confirm satisfaction upon issue resolution
### Reopen ticket within 48 hours if issue recurs
## 1.6 Cancel Request & Rate Service
### Self-cancel ticket when assistance is no longer needed
### Submit 1–5 star rating and feedback comments

# 2. Technician Workspace (Kỹ thuật viên)
## 2.1 View & Claim Assigned Tickets
### Personal queue and team queue views
### Manual claiming (Claim ticket action)
## 2.2 Review AI Summary & Context
### Read 3-line AI issue synthesis
### Inspect pre-collected context and user answers
## 2.3 Clarify Context & Remote Support
### Request additional details if ticket remains unclear
### Coordinate remote desktop sessions (TeamViewer / AnyDesk)
## 2.4 Update Ticket State & Document Fix
### Transition states (In Progress, Needs Info, Resolved)
### Record root cause and step-by-step resolution
## 2.5 Reassign & Escalate (Chuyển nhóm / cấp)
### Transfer ticket to specialized tier (Network, DevOps, Hardware)
### Escalate priority on critical business impact
## 2.6 Internal Collaboration & AI Tuning
### Add internal work notes (Hidden from Requester)
### Override AI category & priority (Continuous model feedback)

# 3. Administration & Coordination (Admin & Điều phối)
## 3.1 Global Ticket Oversight & Search
### Filter across department, status, SLA clock, technician
### Administrative intervention & status overrides
## 3.2 Technician Dispatch & Load Balancing
### Manual reassignments during absences or surges
### Rebalance workload across active shifts
## 3.3 RBAC & User Directory Management
### 4-tier roles: Requester, Technician, Team Lead, Admin
### Configure support teams and tier rosters
## 3.4 Routing Rules & Category Hierarchy
### Maintain multi-level category taxonomy
### Configure deterministic rules (Condition -> Target Team -> Distribution)
## 3.5 SLA Policies & Working Calendars
### Define response and resolution timers by priority (P1–P4)
### Configure operating hours, shifts, and holiday calendars
### Multi-channel alerts (Email, Slack, MS Teams, In-App)
## 3.6 AI Processing Scope & Governance
### Safe action catalog whitelist for automated API execution
### Configure confidence thresholds (e.g. >= 85%)

# 4. Knowledge Base — KB (Kho kiến thức)
## 4.1 Author & Edit Troubleshooting Guides
### Rich Text & Markdown editor with syntax-highlighted code
### Metadata tagging: Category, Tags, Error Codes
### Audience segmentation: Public (Company) vs Internal (IT Only)
### Auto-save drafts & version revision branching
## 4.2 Review & Publish Workflow
### Approval queue (Draft -> Pending Review -> Published)
### Side-by-side Visual Diff inspection (Additions vs Deletions)
### Reviewer actions: Approve, Request Changes, Reject
### Version increments (v1.0 -> v1.1) and immutable audit log
## 4.3 Smart Search & Solution Retrieval
### AI Semantic / RAG natural language search
### Instant answer snippet extraction
### Faceted filtering by OS, category, and helpfulness
### Real-time ticket deflection during request submission
## 4.4 AI Synthesis from Resolved Tickets
### Automatic trigger upon ticket closure
### Automated PII scrubbing (Mask names, IPs, tokens)
### Structured extraction: Problem, Root Cause, Solution Steps
### 1-Click transfer to authoring editor
## 4.5 Content Quality & Feedback Loop
### Reader helpfulness voting (Thumbs Up / Down)
### Outdated procedure reporting modal
### Automated flagging for articles with Helpful < 60%
### Scheduled review alerts for authors

# 5. AI Intelligent Processing (Xử lý thông minh bằng AI)
## 5.1 Intent Classification & Routing Suggestion
### Natural language understanding (NLP) of incoming emails
### Predict team assignment with confidence score
## 5.2 Dynamic Clarification (Ask-Back)
### Detect ambiguous or underspecified descriptions
### Generate one targeted clarifying question in user's language
## 5.3 KB Solution Recommendations
### Query vector embeddings for semantic matches
### Cite verified KB solutions with source article links
## 5.4 Technician Executive Briefing
### Compress multi-turn discussions into concise summaries
### Highlight attempted troubleshooting steps
## 5.5 Business Impact & Priority Scoring
### Evaluate impact radius (Individual vs Entire department)
### Recommend priority P1 (Critical) through P4 (Low)
## 5.6 Deduplication & Incident Clustering
### Detect duplicate tickets from simultaneous users
### Suggest ticket merges or linkage to Major Incidents

# 6. Automation Workflow Engine (Workflow tự động)
## 6.1 Ingestion ➔ Triage ➔ Assignment
### Parse inbound channels and trigger rule engine
### Dispatch via Round-robin, Least Busy, or Skill matching
## 6.2 Self-Service Guidance ➔ Verification
### Dispatch automated self-help response
### Await requester resolution confirmation
## 6.3 Fallback to Human Handling
### Auto-escalate to technician if self-help is unsuccessful
### Ensure zero unhandled tickets
## 6.4 Context Reminders & SLA Escalation
### Auto-send follow-up reminders in Needs Info state
### Trigger escalation notifications before SLA breach
## 6.5 Real-Time Status Webhooks
### Broadcast status mutations to Webhooks, Slack, and Email
## 6.6 Auto-Close & Reopen Handlers
### Auto-close resolved tickets after 48h of inactivity
### Reopen ticket lifecycle on subsequent requester reply

# 7. Approval & Automated Execution (Phê duyệt & Thực thi)
## 7.1 Access & Entitlement Ingestion
### Capture specific requests (Account creation, shared drive access)
## 7.2 Deterministic Policy & Whitelist Check
### Validate against department entitlement catalog
### Enforce manager approval requirement if out of standard scope
## 7.3 Multi-Level Approval Workflow
### AI approval for pre-whitelisted standard actions
### Manager approval routing for elevated permissions
## 7.4 Post-Approval API Execution
### Invoke target connectors (Active Directory, Google Workspace, AWS, GitHub)
### Automate account provisioning and permission grants
## 7.5 Verification & Ticket Resolution
### Query target system to verify successful provisioning
### Append audit log and transition ticket to Resolved
## 7.6 Failure Recovery & Human Fallback
### Catch connector timeouts or API rejection errors
### Re-route failed automation tickets to human technician

# 8. Dashboard & Governance Telemetry (Dashboard & Giám sát)
## 8.1 Ticket Volume & Status Metrics
### Real-time incoming volume, backlog depth, and closure rates
### Priority distribution graphs (P1 to P4)
## 8.2 SLA Compliance & Resolution Times
### Mean Time to Acknowledge (MTTA)
### Mean Time to Resolve (MTTR)
### SLA compliance percentage tracking
## 8.3 AI Deflection & Automation Efficiency
### Percentage of tickets resolved without human intervention (25–35%)
### KB deflection success rate
## 8.4 Team Capacity & Workload Balance
### Active ticket burden per technician
### Shift capacity and utilization analytics
## 8.5 Security Audit Trail (SOC-2 Compliance)
### Immutable chronological action logging
### Actor tracking: User, Technician, Admin, and Automation
## 8.6 Operational Reporting & Export
### Export operational reports to Excel, CSV, and PDF
### Automated weekly executive digests
