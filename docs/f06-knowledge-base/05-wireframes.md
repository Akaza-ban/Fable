# 05. Wireframes & UX/UI Layouts — F06 Knowledge Base

> Design status: TARGET V1
>
> System name: **AI-Powered IT Ticket Routing and Automated Resolution System**
>
> Scope: **F06 Knowledge Base — KB** (Core UI Wireframes)

---

## 1. Screen 1: Search Portal & Article Reader (Requester & Technician View)

```text
+------------------------------------------------------------------------------------------------------------------------+
| [LOGO] IT SERVICE DESK          Home     My Requests     [Knowledge Base]     Support          (🔔) [Avatar v]         |
+------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                        |
|      ================================= IT RESOLUTION & KNOWLEDGE BASE PORTAL =================================         |
|      |                                                                                                       |         |
|      |  [ 🔍 Describe your issue, error code, or software name...                             ] [ Search ]   |         |
|      =========================================================================================================         |
|      Trending topics:  [VPN Setup]   [Printer Error 0x0000011b]   [Password Reset]   [Docker on macOS]                 |
|                                                                                                                        |
|------------------------------------------------------------------------------------------------------------------------|
| Home > Network & Connectivity > VPN > KB-1042                                                                          |
|------------------------------------------------------------------------------------------------------------------------|
|                                                                                        | RELATED ARTICLES              |
| KB-1042: Fix VPN Connection Failure After Active Directory Password Reset             |                               |
| [PUBLIC - Company Wide]  •  Updated: Nov 10, 2026  •  Author: Tuan IT  •  Ver: v1.2    | • KB-1011: Cisco VPN Client   |
| Tags: [vpn] [active-directory] [password-reset] [auth-failed]                          | • KB-1089: Gitlab Permissions |
|----------------------------------------------------------------------------------------| • KB-2033: 2FA Authenticator  |
|                                                                                        |                               |
| 1. PROBLEM DESCRIPTION:                                                                |-------------------------------|
| After employees update their corporate Active Directory credentials, GlobalProtect or | HELPFULNESS RATING            |
| Cisco AnyConnect fails with "Authentication Failed" or "Invalid username or password"  | 96% found this helpful        |
| even when the new password is typed correctly.                                         | Views: 1,420                  |
|                                                                                        |                               |
| 2. ROOT CAUSE:                                                                         |                               |
| The client VPN software caches legacy authentication tokens inside the Windows         |                               |
| Credential Manager (or macOS Keychain).                                                |                               |
|                                                                                        |                               |
| 3. STEP-BY-STEP RESOLUTION:                                                            |                               |
|                                                                                        |                               |
|  [Step 1] Sign Out and Disconnect Current VPN Session                                  |                               |
|           - Right-click the VPN tray icon in the taskbar.                              |                               |
|           - Navigate to [Settings] -> [Account] -> Click [Sign Out].                   |                               |
|                                                                                        |                               |
|  [Step 2] Purge Stale Credentials in Windows Credential Manager                        |                               |
|           - Press [Win + R], enter the following command, and press Enter:             |                               |
|             +-----------------------------------------------------------+ [ Copy Code ]|                               |
|             | control keymgr.dll                                        |              |                               |
|             +-----------------------------------------------------------+              |                               |
|           - Under Windows Credentials, find entries matching "GlobalProtect"           |                               |
|             and click [Remove].                                                        |                               |
|                                                                                        |                               |
|  [Step 3] Re-authenticate and Complete MFA Challenge                                   |                               |
|           - Relaunch the VPN client, enter your new password.                          |                               |
|           - Approve the Microsoft Authenticator push challenge on your mobile phone.  |                               |
|                                                                                        |                               |
| 4. SCREENSHOT ATTACHMENTS:                                                             |                               |
|    +-----------------------------------------------------------------------------+     |                               |
|    | [ SCREENSHOT: GLOBALPROTECT SIGN OUT BUTTON & CLEAR CREDENTIALS INTERFACE ] |     |                               |
|    +-----------------------------------------------------------------------------+     |                               |
|                                                                                        |                               |
|----------------------------------------------------------------------------------------+-------------------------------|
|                                                                                                                        |
|  [ ARTICLE QUALITY & FEEDBACK ]                                                                                        |
|  Did this article solve your problem?                                                                                  |
|  [ 👍 Yes, Resolved (138) ]                  [ 👎 Not Helpful / Report Issue (4) ]                                     |
|                                                                                                                        |
|  ====================================================================================================================  |
|  Still unable to resolve your issue? Our IT technicians are ready to assist.                                           |
|  [ + Create Support Ticket ]  (Context from this article will be automatically attached to your request)              |
|  ====================================================================================================================  |
+------------------------------------------------------------------------------------------------------------------------+
```

---

## 2. Screen 2: Article Authoring & Editing Workspace (Technician View)

```text
+------------------------------------------------------------------------------------------------------------------------+
| [<- Back to List]           KB ARTICLE AUTHORING WORKSPACE               (Auto-saved: 10:42)   [ Preview Mode ]        |
|                                                                              [ Save Draft ]   [ Submit for Review ]    |
+------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                        |
|  Article Title (* Required):                                                                                           |
|  [ Fix VPN Connection Failure After Active Directory Password Reset                                                  ] |
|                                                                                                                        |
|  +--------------------------------------------+  +------------------------------------+  +---------------------------+ |
|  | Category (*):                              |  | Target Audience Scope (*):         |  | Associated Error Code:    | |
|  | [ Network & VPN                           v]  |  (*) Public (Company-Wide)         |  | [ AUTH_ERR_04             ] | |
|  |                                            |  |  ( ) Internal Only (IT Team)       |  |                           | |
|  +--------------------------------------------+  +------------------------------------+  +---------------------------+ |
|                                                                                                                        |
|  Keywords / Tags: [ vpn x ] [ active-directory x ] [ credentials x ] [ + Add Tag... ]                                  |
|                                                                                                                        |
|  Rich Text & Markdown Editor:                                                                                          |
|  +-------------------------------------------------------------------------------------------------------------------+ |
|  | [B] [I] [U] [S] | [H1] [H2] [H3] | [• List] [1. List] | [<> Code] [🔗 Link] [📷 Image] [📊 Table] | [🤖 AI Rewrite] | |
|  +-------------------------------------------------------------------------------------------------------------------+ |
|  | ### 1. Problem Description                                                                                        | |
|  | After employees update their corporate Active Directory credentials...                                            | |
|  |                                                                                                                   | |
|  | ### 2. Root Cause                                                                                                 | |
|  | The client VPN software caches legacy tokens in Windows Credential Manager...                                     | |
|  |                                                                                                                   | |
|  | ### 3. Step-by-Step Resolution                                                                                    | |
|  | - **Step 1**: Disconnect and sign out of VPN...                                                                   | |
|  | - **Step 2**: Purge stale credentials with `control keymgr.dll`...                                                | |
|  |                                                                                                                   | |
|  +-------------------------------------------------------------------------------------------------------------------+ |
|                                                                                                                        |
|  Attachments & Scripts (Max 25MB):                                                                                     |
|  [ 📎 clear_credentials_fix.bat (12 KB) [x] ]   [ 📎 vpn_setup_guide.pdf (2.4 MB) [x] ]   [ + Upload File ]            |
|                                                                                                                        |
|  Revision Changelog Summary (Visible to Reviewer):                                                                     |
|  [ Added shortcut command for Windows 11 Credential Manager and updated MFA challenge screenshots                    ] |
+------------------------------------------------------------------------------------------------------------------------+
```

---

## 3. Screen 3: AI Drawer — Synthesis from Resolved Ticket

```text
+------------------------------------------------------------------------------------------------------------------------+
|                                    TICKET #TK-4921 DETAILS (STATUS: RESOLVED)                                          |
+------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                        |
|  ====================================================================================================================  |
|  ||  🤖 AI ASSISTANT: RESOLVED TICKET TO KB CONVERSION                                           [Close / Dismiss ✕] ||  |
|  ====================================================================================================================  |
|  ||  AI has analyzed resolution thread between Technician Nguyen Van Tuan and Requester Hoang Nam.                  ||  |
|  ||  [✔ PII Sanitized]: Automatically masked email (h***@company.com) and client host IP (192.168.1.50 -> <Client_IP>).||  |
|  ||                                                                                                                ||  |
|  ||  Suggested Title:                                                                                              ||  |
|  ||  [ Fix Docker Desktop Daemon Startup Crash on macOS Sequoia                                                  ] ||  |
|  ||                                                                                                                ||  |
|  ||  Recommended Category: [ Development Tools & Environment v ]    Tags: [docker, macos, sequoia, permission]     ||  |
|  ||                                                                                                                ||  |
|  ||  Extracted Troubleshooting Structure:                                                                          ||  |
|  ||  +-----------------------------------------------------------------------------------------------------------+ ||  |
|  ||  | • PROBLEM: Docker Desktop crashes immediately with "Docker daemon is not responding" after macOS upgrade.  | ||  |
|  ||  | • ROOT CAUSE: macOS Sequoia security policy resets socket ownership on /var/run/docker.sock to root:wheel.  | ||  |
|  ||  | • STEP-BY-STEP RESOLUTION:                                                                                | ||  |
|  ||  |   Step 1: Launch macOS Terminal.                                                                          | ||  |
|  ||  |   Step 2: Run symlink fix: `sudo ln -s -f ~/.docker/run/docker.sock /var/run/docker.sock`                  | ||  |
|  ||  |   Step 3: Enter administrative credentials and restart Docker Desktop.                                    | ||  |
|  ||  +-----------------------------------------------------------------------------------------------------------+ ||  |
|  ||                                                                                                                ||  |
|  ||  Extraction Confidence: 94% (Verified solution from Ticket #TK-4921)                                          ||  |
|  ||                                                                                                                ||  |
|  ||  [ Dismiss ]                            [ Author Manually ]                  [ 👉 Transfer to KB Editor Form ] ||  |
|  ====================================================================================================================  |
|                                                                                                                        |
+------------------------------------------------------------------------------------------------------------------------+
```

---

## 4. Screen 4: Administrative Governance & Visual Diff View (Admin Console)

```text
+------------------------------------------------------------------------------------------------------------------------+
| [ADMIN CONSOLE]     Overview     Ticket Management     AI Automation     [Knowledge Base Management]    (Admin: Le Lan)|
+------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                        |
|  OPERATIONAL HEALTH METRICS:                                                                                           |
|  +-----------------------+  +-----------------------+  +-----------------------+  +----------------------------------+ |
|  | PUBLISHED ARTICLES    |  | PENDING REVIEWS       |  | FLAGGED FOR REVISION  |  | SELF-SERVICE DEFLECTION RATE     | |
|  | 148 active articles   |  | 03 awaiting approval  |  | 02 reported outdated  |  | 34.8% (Saved ~420 tickets/month) | |
|  +-----------------------+  +-----------------------+  +-----------------------+  +----------------------------------+ |
|                                                                                                                        |
|  ARTICLE MANAGEMENT QUEUE:                                                                                             |
|  [ Tab 1: Pending Review (3) ]   [ Tab 2: Published (148) ]   [ Tab 3: Flagged (2) ]   [ Tab 4: Archived Repository ]   |
|                                                                                                                        |
|  Filters: Category [ All v ]   Author [ All v ]   Type [ All v ]   Search: [ Search by Title or KB Code...           ] |
|                                                                                                                        |
|  +----------+-------------------------------------------------------+---------------+------------+-----------+-------+ |
|  | CODE     | ARTICLE TITLE                                         | AUTHOR        | SUBMITTED  | VERSION   | ACTION| |
|  +----------+-------------------------------------------------------+---------------+------------+-----------+-------+ |
|  | KB-1042  | Fix VPN Connection Failure After AD Password Reset    | Tuan Nguyen   | 10/11/2026 | v1.2 (Rev)| [Diff]|[Approve]|[Reject]
|  | KB-1090  | Fix Docker Desktop Daemon Crash on macOS Sequoia      | AI Service    | 10/11/2026 | v1.0 (New)| [Diff]|[Approve]|[Reject]
|  | KB-1088  | Requesting Gitlab Enterprise Repository Permissions   | Hoang Anh Le  | 09/11/2026 | v1.0 (New)| [Diff]|[Approve]|[Reject]
|  +----------+-------------------------------------------------------+---------------+------------+-----------+-------+ |
|                                                                                                                        |
|------------------------------------------------------------------------------------------------------------------------|
|  VISUAL DIFF INSPECTION MODAL: KB-1042 (v1.1 vs v1.2 Proposed)                                                         |
|------------------------------------------------------------------------------------------------------------------------|
|  [ Current Active: v1.1 (Public) ]                     |  [ Proposed Revision: v1.2 (Pending Approval) ]               |
|  - Step 2: Open Start Menu and search for Credential.  |  + Step 2: Press [Win + R] and execute "control keymgr.dll".  |
|  - Find legacy passwords and remove them.              |  + (Direct command saves 3 manual navigation steps for user)  |
|                                                        |  + [Updated Windows 11 Credential Manager screenshot added]   |
|                                                                                                                        |
|  ADMIN AUDIT NOTE: [ Validated technical procedure. Verified clear command syntax. Approved for production.          ] |
|                                                                                                                        |
|  [ Cancel ]                         [ ✍ Request Additional Revisions ]                      [ ✔ APPROVE & PUBLISH v1.2 ]|
+------------------------------------------------------------------------------------------------------------------------+
```
