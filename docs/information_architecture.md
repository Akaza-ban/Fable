# Information architecture

Cấu trúc màn hình, điều hướng theo vai trò và dữ liệu chính của 3 module. Quyền và `BR-xx` xem ở [MVP.md](MVP.md); luồng xử lý ở [use_cases.md](use_cases.md).

## 1. Sơ đồ trang (sitemap)

```
/login                         Đăng nhập (công khai)
/login/2fa                     Nhập mã 2FA (sau bước mật khẩu hoặc Microsoft)
/login/callback?code=…         Nhận kết quả đăng nhập Microsoft (công khai)
/forgot-password               Quên mật khẩu (công khai)
/reset-password?token=…        Đặt lại mật khẩu (công khai, có token)
/                              Trang chủ, chuyển hướng theo vai trò
/profile                       Hồ sơ cá nhân             (mọi người dùng)
  ├─ Thông tin tài khoản
  ├─ Đổi mật khẩu              (bắt buộc khi must_change_password; ẩn nếu chỉ dùng Microsoft)
  ├─ Bảo mật (2FA)
  └─ Vai trò & quyền hạn
/admin/users                   Danh sách người dùng       (ADMIN)
  ├─ /admin/users/new          Thêm người dùng
  └─ /admin/users/:id          Chi tiết / sửa người dùng
/admin/groups                  Danh sách nhóm kỹ thuật    (ADMIN)
  └─ /admin/groups/:id         Chi tiết nhóm & thành viên
/admin/microsoft-sync          Ánh xạ & đồng bộ Microsoft (ADMIN)
/admin/service-accounts        Service account AI         (ADMIN)
/admin/ai-suggestions          Đề xuất của AI             (ADMIN)
```

## 2. Menu theo vai trò

| Mục menu | ADMIN | TECHNICIAN | REQUESTER |
|---|---|---|---|
| Dashboard *(module khác)* | ✅ | ❌ | ❌ |
| Ticket *(module khác)* | Tất cả ticket | Ticket được giao | Ticket của tôi |
| Người dùng | ✅ | ❌ | ❌ |
| Nhóm kỹ thuật | ✅ | ❌ | ❌ |
| Đồng bộ Microsoft | ✅ | ❌ | ❌ |
| Service account AI | ✅ | ❌ | ❌ |
| Đề xuất của AI (kèm số đang chờ) | ✅ | ❌ | ❌ |
| Hồ sơ cá nhân | ✅ | ✅ | ✅ |
| Đăng xuất | ✅ | ✅ | ✅ |

Trang chủ sau đăng nhập: ADMIN → Dashboard, TECHNICIAN → Ticket được giao, REQUESTER → Ticket của tôi. Khi `must_change_password = true` mọi trang chuyển về `/profile` tab Đổi mật khẩu. AI_AGENT không có giao diện. Menu chỉ để tiện dùng; backend vẫn phải chặn quyền.

## 3. Nội dung màn hình

| Màn hình | Thành phần chính |
|---|---|
| Đăng nhập | Nút "Đăng nhập bằng Microsoft"; email, mật khẩu, nút Đăng nhập, link Quên mật khẩu; thông báo lỗi / bị khóa / lỗi Microsoft |
| Nhận kết quả Microsoft | Màn hình chờ; đổi `code` lấy token rồi chuyển trang chủ hoặc `/login/2fa`; lỗi → về `/login` |
| Nhập mã 2FA | Ô mã 6 số, nút Xác nhận, thông báo sai mã / bị khóa |
| Quên mật khẩu | Email, nút Gửi, thông báo chung (BR-11) |
| Đặt lại mật khẩu | Mật khẩu mới, xác nhận, quy tắc mật khẩu; báo link hết hiệu lực |
| Danh sách người dùng | Ô tìm kiếm; bộ lọc vai trò, trạng thái, nhóm, kỹ năng; bảng (họ tên, email, vai trò, nhóm, kỹ năng, cách đăng nhập: mật khẩu / Microsoft, trạng thái kèm lý do tắt — "Đã bị gỡ bên Microsoft" khi `MICROSOFT_REMOVED`, lần đăng nhập cuối); nút Thêm; công tắc Bật / Tắt |
| Thêm / sửa người dùng | Email (chỉ nhập khi tạo), họ tên, SĐT, vai trò, nhóm và kỹ năng (khi là TECHNICIAN), mật khẩu tạm (khi tạo, tùy chọn — bỏ trống thì chỉ dùng Microsoft); nút Reset 2FA (khi sửa); trạng thái liên kết Microsoft (chỉ xem); vai trò / nhóm do Microsoft quản lý bị khóa, ghi "Đồng bộ từ Microsoft" |
| Ánh xạ & đồng bộ Microsoft | Bảng ánh xạ (nhóm Microsoft: tên + Object ID → đích: vai trò hoặc nhóm kỹ thuật); nút Thêm (tìm nhóm Microsoft theo tên) / Xóa; nút "Đồng bộ ngay"; kết quả lần chạy gần nhất (thời điểm, số user, số thay đổi, danh sách người bị gỡ kèm số ticket đang mở, danh sách lỗi / bỏ qua) |
| Danh sách nhóm | Tên nhóm, danh mục phụ trách, số thành viên, số ticket đang mở |
| Chi tiết nhóm | Thông tin nhóm; danh mục phụ trách; nhóm Microsoft đang ánh xạ tới; bảng thành viên (kèm kỹ năng, nguồn: Thủ công / Microsoft); thêm / bớt thành viên (không bớt được thành viên nguồn Microsoft) |
| Service account AI | Bảng (tên, trạng thái, lần dùng cuối, ngày tạo); nút Tạo, Xoay key, Bật / Tắt; hộp hiển thị key một lần |
| Đề xuất của AI | Bộ lọc trạng thái, loại; bảng (loại, đối tượng, thay đổi, lý do, độ tin cậy, số ticket bằng chứng, ngày tạo); xem chi tiết với link ticket; nút Duyệt / Từ chối (kèm lý do) |
| Hồ sơ: Thông tin | Avatar, họ tên, SĐT (sửa được); email, vai trò, trạng thái, kỹ năng, liên kết Microsoft (chỉ xem) |
| Hồ sơ: Đổi mật khẩu | Mật khẩu cũ, mật khẩu mới, xác nhận |
| Hồ sơ: Bảo mật | Trạng thái 2FA; bật (mã QR + ô xác nhận mã); tắt (mật khẩu + mã) — ẩn nút tắt với ADMIN |
| Hồ sơ: Vai trò & quyền | Vai trò, nhóm, kỹ năng, danh sách quyền dạng câu dễ hiểu |

## 4. Dữ liệu chính

| Entity | Trường chính |
|---|---|
| `User` | id, email (unique), password_hash (null nếu chỉ dùng Microsoft), microsoft_oid (unique, null nếu chưa liên kết), full_name, phone, avatar_url, role, role_source (`MANUAL` / `MICROSOFT`), status (`ACTIVE` / `DISABLED`), disabled_reason (`MANUAL` / `MICROSOFT_REMOVED`, null khi `ACTIVE`), skills[], last_synced_at, must_change_password, totp_secret_enc, totp_enabled, failed_login_count, locked_until, last_login_at, created_at, updated_at |
| `TechnicianGroup` | id, name (unique), description, categories[], created_at, updated_at |
| `GroupMember` | group_id, user_id, source (`MANUAL` / `MICROSOFT`), joined_at (PK kép group_id + user_id + source) |
| `MicrosoftGroupMapping` | id, entra_group_id, entra_group_name, target_type (`ROLE` / `TECHNICIAN_GROUP`), target_role (khi `ROLE`), technician_group_id (khi `TECHNICIAN_GROUP`), created_at, updated_at — unique (entra_group_id, target_type, target_role, technician_group_id) |
| `MicrosoftSyncRun` | id, trigger (`LOGIN` / `SCHEDULED` / `MANUAL`), started_at, finished_at, users_processed, changes_count, removed_users[] (user_id, open_ticket_count), errors[] (user_id, message) |
| `RefreshToken` | id, user_id, token_hash, expires_at, revoked_at |
| `PasswordResetToken` | id, user_id, token_hash, expires_at, used_at |
| `ServiceAccount` | id, name, api_key_hash, role = `AI_AGENT`, status (`ACTIVE` / `DISABLED`), last_used_at, created_at |
| `AiSuggestion` | id, type (`GROUP_CATEGORY` / `USER_SKILL`), target_id, operation (`ADD` / `REMOVE`), value, reason, confidence (0–1), evidence_ticket_ids[], status (`PENDING` / `APPROVED` / `REJECTED`), created_by (service account id), reviewed_by, review_note, reviewed_at, created_at |
| `AuditLog` | id, actor_id (null khi `SYSTEM`), actor_type (`USER` / `AI` / `SYSTEM`), action, target_type, target_id, before, after, suggestion_id, created_at |

Quan hệ: `User` n–n `TechnicianGroup` qua `GroupMember`; `User` 1–n `RefreshToken`, `PasswordResetToken`; `ServiceAccount` 1–n `AiSuggestion`; `TechnicianGroup` 1–n `MicrosoftGroupMapping` (xóa nhóm thì xóa ánh xạ theo). Một user có thể có 2 dòng `GroupMember` cùng nhóm (`MANUAL` và `MICROSOFT`); vẫn tính là 1 thành viên. `AiSuggestion.target_id` trỏ tới `TechnicianGroup` (loại `GROUP_CATEGORY`) hoặc `User` (loại `USER_SKILL`). Quyền của mỗi vai trò cố định trong code theo ma trận ở MVP.md (chưa lưu DB).

## 5. API chính

Prefix `/api/v1`. Người dùng gửi `Authorization: Bearer <access_token>`; AI gửi `X-API-Key`. Lỗi trả dạng `{ "code": "...", "message": "..." }`.

| Method | Path | Quyền |
|---|---|---|
| POST | `/auth/login` | công khai (trả token, hoặc `challenge_token` nếu bật 2FA) |
| POST | `/auth/login/2fa` | `challenge_token` |
| GET | `/auth/microsoft/login` | công khai (chuyển hướng sang Microsoft) |
| GET | `/auth/microsoft/callback` | công khai (Microsoft gọi về, kiểm tra `state`) |
| POST | `/auth/microsoft/exchange` | mã một lần từ callback (trả token, hoặc `challenge_token` với ADMIN) |
| POST | `/auth/refresh` | refresh token |
| POST | `/auth/logout` | đã đăng nhập |
| POST | `/auth/forgot-password` | công khai |
| POST | `/auth/reset-password` | công khai (token đặt lại) |
| GET | `/users` | `user:read` |
| POST | `/users` | `user:create` |
| GET / PATCH | `/users/:id` | `user:read` / `user:update` |
| PATCH | `/users/:id/status` | `user:disable` |
| PATCH | `/users/:id/role` | `role:assign` |
| POST | `/users/:id/2fa/reset` | `user:update` |
| GET / POST | `/groups` | `group:read` / `group:manage` |
| GET / PATCH / DELETE | `/groups/:id` | `group:read` / `group:manage` |
| POST / DELETE | `/groups/:id/members[/:userId]` | `group:manage` |
| GET / POST | `/service-accounts` | `service_account:manage` |
| PATCH | `/service-accounts/:id/status` | `service_account:manage` |
| POST | `/service-accounts/:id/rotate-key` | `service_account:manage` |
| GET / POST | `/microsoft-sync/mappings` | `microsoft_sync:manage` |
| DELETE | `/microsoft-sync/mappings/:id` | `microsoft_sync:manage` |
| GET | `/microsoft-sync/entra-groups?search=…` | `microsoft_sync:manage` (tìm nhóm qua Microsoft Graph) |
| POST | `/microsoft-sync/run` | `microsoft_sync:manage` |
| GET | `/microsoft-sync/runs` | `microsoft_sync:manage` |
| POST | `/ai-suggestions` | `ai_suggestion:create` |
| GET | `/ai-suggestions` | `ai_suggestion:review` |
| POST | `/ai-suggestions/:id/approve` | `ai_suggestion:review` |
| POST | `/ai-suggestions/:id/reject` | `ai_suggestion:review` |
| GET / PATCH | `/me` | `profile:read:self` / `profile:update:self` |
| POST | `/me/password` | `profile:update:self` |
| POST | `/me/2fa/setup` / `/me/2fa/enable` / `/me/2fa/disable` | `profile:update:self` |
| GET | `/me/permissions` | `profile:read:self` |

Khi `must_change_password = true`: chỉ cho gọi `GET /me`, `POST /me/password`, `POST /auth/logout`; còn lại trả `403` `PASSWORD_CHANGE_REQUIRED` (BR-10).

Cấu hình đăng nhập Microsoft (biến môi trường): `MICROSOFT_TENANT_ID`, `MICROSOFT_CLIENT_ID`, `MICROSOFT_CLIENT_SECRET`, `MICROSOFT_REDIRECT_URI`, `MICROSOFT_SYNC_INTERVAL_MINUTES` (mặc định `60`). App Entra cần quyền ứng dụng Graph `GroupMember.Read.All`, `User.Read.All`.

Khi ADMIN chưa bật 2FA: chỉ cho gọi `GET /me`, `POST /me/2fa/setup`, `POST /me/2fa/enable`, `POST /auth/logout`; còn lại trả `403` `TWO_FACTOR_REQUIRED` (BR-12).
