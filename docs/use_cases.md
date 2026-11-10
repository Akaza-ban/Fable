# Use cases

Vai trò, ma trận quyền và quy tắc `BR-xx` xem ở [MVP.md](MVP.md).

## Tổng quan

| Mã | Use case | Actor |
|---|---|---|
| UC-01 | Đăng nhập (kèm 2FA nếu bật) | Mọi người dùng |
| UC-02 | Đăng xuất | Mọi người dùng |
| UC-03 | Kiểm tra quyền & phạm vi trên mỗi request | Hệ thống |
| UC-04 | Xem / tìm danh sách người dùng | Admin |
| UC-05 | Thêm người dùng | Admin |
| UC-06 | Sửa người dùng, gán vai trò & kỹ năng | Admin |
| UC-07 | Bật / tắt tài khoản | Admin |
| UC-08 | Quản lý nhóm kỹ thuật & thành viên | Admin |
| UC-09 | Xem & cập nhật hồ sơ cá nhân | Mọi người dùng |
| UC-10 | Đổi mật khẩu | Mọi người dùng |
| UC-11 | Xem vai trò & quyền của mình | Mọi người dùng |
| UC-12 | Quên mật khẩu | Mọi người dùng (chưa đăng nhập) |
| UC-13 | Bật / tắt 2FA | Mọi người dùng; Admin reset cho người khác |
| UC-14 | Quản lý service account AI | Admin |
| UC-15 | AI gửi đề xuất cải thiện phân luồng | AI_AGENT |
| UC-16 | Duyệt / từ chối đề xuất của AI | Admin |
| UC-17 | Đăng nhập bằng Microsoft | Người dùng đã được Admin tạo tài khoản |
| UC-18 | Ánh xạ & đồng bộ nhóm Microsoft | Admin; Hệ thống (đồng bộ định kỳ) |

---

## Đăng nhập & phân quyền

### UC-01 Đăng nhập
- **Luồng chính:** nhập email + mật khẩu → hệ thống kiểm tra → nếu bật 2FA: trả `challenge_token`, người dùng nhập mã 6 số → trả access token (15 phút) + refresh token (7 ngày) → chuyển tới trang chủ theo vai trò.
- **Nhánh:** `must_change_password = true` → chuyển tới màn hình đổi mật khẩu, chưa dùng được chức năng khác (BR-10). ADMIN chưa cài 2FA → bắt buộc cài trước khi vào hệ thống (BR-12).
- **Ngoại lệ:** sai thông tin → báo lỗi chung "Email hoặc mật khẩu không đúng"; tài khoản bị tắt → báo "Tài khoản đã bị vô hiệu hóa"; sai mật khẩu hoặc mã 2FA 5 lần → khóa 15 phút (BR-04).
- **BR:** BR-01, BR-04, BR-10, BR-12.

### UC-17 Đăng nhập bằng Microsoft
- **Luồng chính:** bấm "Đăng nhập bằng Microsoft" → chuyển sang trang đăng nhập Microsoft (OIDC, Authorization Code + PKCE) → Microsoft trả về `/auth/microsoft/callback` → backend kiểm tra `state`, tenant, lấy `oid`, email, họ tên → tìm user theo `microsoft_oid`, lần đầu thì theo email rồi liên kết (BR-16) → **đồng bộ vai trò & nhóm kỹ thuật** (BR-18) → chuyển về `/login/callback?code=…` (mã một lần, 60 giây) → frontend đổi mã lấy access + refresh token → trang chủ theo vai trò **sau khi đồng bộ**.
- **Nhánh:** ADMIN (sau đồng bộ) → vẫn phải nhập mã 2FA (BR-17). Lần đầu liên kết → ghi audit log `MICROSOFT_LINKED`.
- **Ngoại lệ:** tài khoản ngoài tenant → "Tài khoản Microsoft không thuộc tổ chức"; email chưa được Admin tạo → "Tài khoản chưa được cấp quyền truy cập", không tạo user; tài khoản bị tắt → "Tài khoản đã bị vô hiệu hóa"; người dùng hủy ở Microsoft hoặc `state` sai → về `/login` kèm thông báo lỗi; Microsoft Graph lỗi → vẫn cho đăng nhập với vai trò / nhóm hiện tại, ghi lỗi vào kết quả đồng bộ.
- **BR:** BR-03, BR-09, BR-12, BR-16, BR-17, BR-18.

### UC-02 Đăng xuất
- Thu hồi refresh token hiện tại, xóa token phía client, về trang đăng nhập.

### UC-03 Kiểm tra quyền & phạm vi
- Mỗi request: xác thực token → lấy vai trò, trạng thái hiện tại từ DB / cache ngắn (BR-07) → kiểm tra quyền của endpoint → lọc dữ liệu theo phạm vi.
- Không có quyền hoặc dữ liệu ngoài phạm vi → `403`. Token hết hạn → `401`, client dùng refresh token để lấy token mới. Tài khoản bị tắt → `401` (BR-03).
- AI_AGENT xác thực bằng API key (header `X-API-Key`), chỉ dùng được các quyền ghi trong ma trận. Mọi request của AI ghi log với `actor_type = AI`.
- **BR:** BR-03, BR-07, BR-09.

### UC-12 Quên mật khẩu
- **Luồng chính:** nhập email → hệ thống luôn báo "Nếu email tồn tại, bạn sẽ nhận được link đặt lại" → gửi email chứa link (token 1 lần, 30 phút) → người dùng mở link, nhập mật khẩu mới → kiểm tra BR-05 → đặt lại thành công, thu hồi mọi refresh token, gỡ khóa.
- **Ngoại lệ:** token hết hạn / đã dùng → báo "Link không còn hiệu lực"; tài khoản bị tắt hoặc không có mật khẩu (chỉ dùng Microsoft) → không gửi email.
- **BR:** BR-05, BR-11, BR-17.

### UC-13 Bật / tắt 2FA
- **Bật:** hệ thống sinh secret + mã QR → người dùng quét bằng ứng dụng Authenticator → nhập mã 6 số để xác nhận → bật.
- **Tắt:** nhập mật khẩu + mã 2FA hiện tại → tắt. ADMIN không tự tắt được (BR-12).
- **Admin reset 2FA người khác** (người dùng mất điện thoại): xóa secret, đăng xuất mọi phiên của người đó, ghi audit log `2FA_RESET`.
- **BR:** BR-09, BR-12.

## Quản lý người dùng

### UC-04 Xem / tìm danh sách người dùng
- Danh sách có phân trang; tìm theo tên, email; lọc theo vai trò, trạng thái, nhóm, kỹ năng.
- **Ngoại lệ:** không có `user:read` → `403`.

### UC-05 Thêm người dùng
- Nhập email, họ tên, vai trò, nhóm và kỹ năng (nếu là TECHNICIAN), mật khẩu tạm (tùy chọn). Có mật khẩu tạm → `must_change_password = true`. Bỏ trống → người dùng chỉ đăng nhập được bằng Microsoft, tự liên kết theo email ở lần đầu (BR-16). Vai trò chọn khi tạo có thể bị thay bởi đồng bộ Microsoft sau lần đăng nhập đầu (BR-18).
- **Ngoại lệ:** email đã tồn tại → `409`; mật khẩu tạm không đạt BR-05 → `422`.
- Ghi audit log `USER_CREATED`.
- **BR:** BR-01, BR-05, BR-09, BR-10.

### UC-06 Sửa người dùng, gán vai trò & kỹ năng
- Sửa họ tên, SĐT, kỹ năng (`skills`), vai trò, nhóm. Không cho tự hạ vai trò, không hạ Admin cuối cùng (BR-02).
- Vai trò / nhóm đang do Microsoft quản lý → `409` `MANAGED_BY_MICROSOFT` (BR-19); giao diện khóa ô đó và ghi "Đồng bộ từ Microsoft".
- Ghi audit log `USER_UPDATED` / `ROLE_CHANGED` kèm giá trị trước và sau.
- **BR:** BR-02, BR-07, BR-09.

### UC-07 Bật / tắt tài khoản
- Tắt → thu hồi token (BR-03). Không cho tắt chính mình hoặc Admin cuối cùng (BR-02).
- Tài khoản bị hệ thống tự tắt do rời Microsoft (BR-20) hiện lý do "Đã bị gỡ bên Microsoft"; Admin vẫn bật lại tay được.
- Nếu tắt TECHNICIAN còn ticket đang xử lý: hiển thị cảnh báo số ticket; việc chuyển ticket thuộc module Ticket / Workflow.
- Ghi audit log `USER_DISABLED` / `USER_ENABLED`.
- **BR:** BR-02, BR-03, BR-09.

### UC-08 Quản lý nhóm kỹ thuật & thành viên
- Tạo / sửa / xóa nhóm (tên, mô tả, danh mục ticket phụ trách). Thêm / bớt TECHNICIAN vào nhóm (thành viên thêm tay có `source = MANUAL`; thành viên `source = MICROSOFT` không bớt tay được — BR-19).
- Danh mục phụ trách của nhóm được AI dùng để phân luồng.
- **Ngoại lệ:** xóa nhóm còn ticket đang mở → `409` (BR-08); thêm người không phải TECHNICIAN → `422`.
- **BR:** BR-08, BR-09.

### UC-18 Ánh xạ & đồng bộ nhóm Microsoft
- **Cấu hình ánh xạ:** Admin thêm dòng ánh xạ: chọn nhóm Microsoft (nhập Object ID hoặc tìm theo tên qua Microsoft Graph) → đích là **một vai trò** (`ADMIN` / `TECHNICIAN` / `REQUESTER`) hoặc **một nhóm kỹ thuật**. Xóa dòng ánh xạ được.
- **Đồng bộ:** chạy khi đăng nhập Microsoft, mỗi 1 giờ, hoặc Admin bấm "Đồng bộ ngay" → với mỗi user đã liên kết: lấy nhóm Microsoft → tính vai trò và nhóm kỹ thuật theo BR-18 → ghi thay đổi + audit log `ROLE_SYNCED` / `GROUP_MEMBER_SYNCED` (`actor_type = SYSTEM`).
- **Tự gỡ người rời tổ chức:** tài khoản Microsoft bị tắt hoặc bị xóa → tắt tài khoản (`disabled_reason = MICROSOFT_REMOVED`), thu hồi phiên, gỡ khỏi mọi nhóm kỹ thuật, ghi audit log `USER_REMOVED_BY_MICROSOFT`; kết quả đồng bộ liệt kê người bị gỡ kèm số ticket đang mở (BR-20).
- **Kết quả đồng bộ:** thời điểm chạy, số user đã xử lý, số thay đổi, danh sách lỗi / bỏ qua (user không phải TECHNICIAN nằm trong nhóm Microsoft ánh xạ tới nhóm kỹ thuật; đụng Admin cuối cùng; Graph lỗi).
- **Ngoại lệ:** Object ID không tồn tại trên Microsoft → `422`; ánh xạ trùng → `409`; nhóm kỹ thuật đích bị xóa → dòng ánh xạ bị xóa theo; hạ hoặc gỡ Admin cuối cùng → bỏ qua (BR-19, BR-20).
- **BR:** BR-02, BR-03, BR-07, BR-09, BR-18, BR-19, BR-20.

### UC-14 Quản lý service account AI
- Tạo service account (tên) → hệ thống sinh API key, hiển thị **một lần** (BR-13).
- Xoay key: sinh key mới, key cũ vô hiệu ngay. Bật / tắt: tắt → mọi request bằng key đó trả `401`.
- Danh sách hiển thị tên, trạng thái, lần dùng cuối; không bao giờ hiển thị lại key.
- Ghi audit log `SERVICE_ACCOUNT_CREATED` / `API_KEY_ROTATED` / `SERVICE_ACCOUNT_DISABLED`.
- **BR:** BR-09, BR-13.

### UC-15 AI gửi đề xuất cải thiện phân luồng
- AI phát hiện mẫu từ ticket đã xử lý (ticket bị chuyển nhóm, kỹ thuật viên giải quyết tốt một loại vấn đề) → gửi đề xuất loại `GROUP_CATEGORY` hoặc `USER_SKILL`, kèm lý do, độ tin cậy, danh sách ticket bằng chứng.
- Đề xuất ở trạng thái `PENDING`, không thay đổi dữ liệu nào.
- **Ngoại lệ:** loại đề xuất khác 2 loại trên → `422`; trùng đề xuất đã bị từ chối trong 30 ngày → `409`; thiếu bằng chứng → `422`.
- **BR:** BR-14, BR-15.

### UC-16 Duyệt / từ chối đề xuất của AI
- Admin xem danh sách đề xuất `PENDING`: nội dung thay đổi, lý do, độ tin cậy, link tới ticket bằng chứng.
- **Duyệt:** hệ thống áp dụng thay đổi (thêm / bớt danh mục nhóm hoặc kỹ năng), trạng thái `APPROVED`, audit log actor là Admin kèm `suggestion_id`.
- **Từ chối:** nhập lý do (tùy chọn), trạng thái `REJECTED`; lý do được lưu để AI học.
- **Ngoại lệ:** đề xuất đã xử lý → `409`; đối tượng đích đã bị xóa / tắt → `422`, đề xuất chuyển `REJECTED`.
- **BR:** BR-09, BR-14, BR-15.

## Hồ sơ cá nhân

### UC-09 Xem & cập nhật hồ sơ
- Sửa họ tên, SĐT, avatar. Email, vai trò, trạng thái, kỹ năng chỉ xem (BR-01).

### UC-10 Đổi mật khẩu
- Nhập mật khẩu cũ, mật khẩu mới, xác nhận → kiểm tra BR-05 → đổi thành công → đăng xuất các phiên khác (BR-06), bỏ cờ `must_change_password` (BR-10).
- **Ngoại lệ:** sai mật khẩu cũ → `400`; mật khẩu mới không đạt → `422`; tài khoản không có mật khẩu (chỉ dùng Microsoft) → `422` (BR-17).

### UC-11 Xem vai trò & quyền
- Hiển thị vai trò, nhóm đang thuộc, kỹ năng, trạng thái 2FA, đã liên kết Microsoft hay chưa và danh sách quyền dạng dễ đọc (ví dụ "Xem ticket được giao cho bạn và nhóm của bạn").
