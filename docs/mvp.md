# MVP — Đăng nhập & phân quyền, Quản lý người dùng, Hồ sơ cá nhân

## 1. Bối cảnh
Hệ thống **phân luồng & xử lý ticket tự động ứng dụng AI**: người dùng gửi ticket → AI phân loại, phân luồng → điều phối tới kỹ thuật viên phù hợp, hoặc AI tự xử lý ticket đơn giản.

**Định hướng sản phẩm:** MVP làm đủ toàn bộ chức năng của 3 module nền tảng. Hướng phát triển duy nhất sau MVP là **tự động hóa ngày càng sâu**: AI học dần từ ticket thực tế của người dùng để hiểu được cả ticket phức tạp (xem mục 8).

Tài liệu này chỉ mô tả 3 module nền tảng. Các module khác (Dashboard, Quản lý ticket, Xử lý ticket bằng AI, Workflow & tự động hóa, Kho kiến thức, Báo cáo, Nhật ký hoạt động) dùng lại vai trò, phạm vi quyền và dữ liệu định nghĩa ở đây.

## 2. Vai trò

| Mã | Vai trò | Mô tả |
|---|---|---|
| `ADMIN` | Quản trị viên | Quản lý người dùng, nhóm, vai trò, service account AI; duyệt đề xuất của AI; xem mọi ticket; cấu hình hệ thống |
| `TECHNICIAN` | Kỹ thuật viên | Xử lý ticket được giao cho mình hoặc nhóm mình |
| `REQUESTER` | Người gửi ticket | Tạo ticket và xem ticket của mình |
| `AI_AGENT` | AI (service account) | Phân luồng, tóm tắt, gợi ý, xử lý ticket đơn giản; gửi đề xuất cải thiện phân luồng. Không đăng nhập bằng mật khẩu, dùng API key. **Không được tự cấp quyền** |

Mỗi người dùng có **một vai trò**.

## 3. Phạm vi MVP

**Có trong MVP**
- Đăng nhập / đăng xuất bằng email + mật khẩu (JWT access + refresh token).
- Đăng nhập bằng tài khoản Microsoft (Microsoft Entra ID, chuẩn OpenID Connect) của tổ chức. Chỉ người đã được Admin tạo tài khoản mới đăng nhập được.
- Đồng bộ vai trò và nhóm kỹ thuật từ nhóm Microsoft (Entra group) mà người đó tham gia, theo bảng ánh xạ Admin cấu hình.
- Người bị tắt / xóa bên Microsoft → hệ thống tự gỡ: tắt tài khoản, thu hồi phiên, gỡ khỏi mọi nhóm kỹ thuật.
- Xác thực 2 lớp (2FA) bằng TOTP (ứng dụng Authenticator).
- Quên mật khẩu: đặt lại qua link gửi email.
- Bắt buộc đổi mật khẩu tạm ở lần đăng nhập đầu.
- Phân quyền theo vai trò + phạm vi dữ liệu ticket (`own`, `assigned`, `group`, `all`).
- Admin: xem, tìm, thêm, sửa người dùng; bật / tắt tài khoản; gán vai trò; gán kỹ năng (`skills`) cho kỹ thuật viên; reset 2FA cho người khác.
- Admin: tạo / sửa / xóa nhóm kỹ thuật, danh mục phụ trách; thêm / bớt thành viên.
- Admin: quản lý service account AI (tạo, xoay API key, bật / tắt).
- Admin: cấu hình ánh xạ nhóm Microsoft → vai trò / nhóm kỹ thuật; xem kết quả đồng bộ; bấm đồng bộ ngay.
- Admin: duyệt / từ chối **đề xuất của AI** (thêm danh mục cho nhóm, thêm kỹ năng cho kỹ thuật viên) — đây là điểm AI "học" tác động vào dữ liệu phân luồng.
- Người dùng: xem và sửa hồ sơ (họ tên, SĐT, avatar), đổi mật khẩu, bật / tắt 2FA, xem vai trò, nhóm, kỹ năng và quyền của mình.
- Ghi audit log cho mọi thao tác trên người dùng, vai trò, nhóm, service account, đề xuất AI.

**Không làm** (xem mục "Giả định")
- AI tự cấp quyền hoặc tự áp dụng thay đổi lên người dùng / nhóm — luôn cần Admin duyệt.

## 4. Phạm vi xem ticket

| Vai trò | Được xem ticket |
|---|---|
| ADMIN | Tất cả |
| TECHNICIAN | Ticket giao cho mình + ticket của nhóm mình |
| REQUESTER | Ticket do mình tạo |
| AI_AGENT | Tất cả (chỉ đọc để phân luồng, tóm tắt, học) |

Phạm vi phải được áp ở **tầng truy vấn** (backend lọc dữ liệu), không chỉ ẩn trên giao diện. Truy cập ngoài phạm vi trả `403`.

## 5. Ma trận quyền

| Quyền | ADMIN | TECHNICIAN | REQUESTER | AI_AGENT |
|---|---|---|---|---|
| `user:read` | ✅ | ❌ | ❌ | ✅ (chỉ đọc để phân công) |
| `user:create` / `user:update` / `user:disable` | ✅ | ❌ | ❌ | ❌ |
| `role:assign` | ✅ | ❌ | ❌ | ❌ |
| `group:manage` | ✅ | ❌ | ❌ | ❌ |
| `group:read` | ✅ | ✅ (nhóm mình) | ❌ | ✅ |
| `service_account:manage` | ✅ | ❌ | ❌ | ❌ |
| `microsoft_sync:manage` | ✅ | ❌ | ❌ | ❌ |
| `ai_suggestion:create` | ❌ | ❌ | ❌ | ✅ |
| `ai_suggestion:review` | ✅ | ❌ | ❌ | ❌ |
| `profile:read:self` / `profile:update:self` | ✅ | ✅ | ✅ | ❌ |
| `ticket:read` | all | assigned, group | own | all |

`user:update` gồm cả sửa `skills` và reset 2FA của người khác.

## 6. Quy tắc nghiệp vụ

- **BR-01**: Email là duy nhất và dùng để đăng nhập. Người dùng không tự đổi email, vai trò, trạng thái, kỹ năng.
- **BR-02**: Luôn còn ít nhất 1 ADMIN đang hoạt động. Admin không tự tắt hoặc tự hạ vai trò của mình.
- **BR-03**: Tắt tài khoản → thu hồi toàn bộ refresh token ngay, không đăng nhập được nữa.
- **BR-04**: Sai mật khẩu (hoặc sai mã 2FA) 5 lần liên tiếp → khóa đăng nhập 15 phút. Áp dụng cho đăng nhập bằng mật khẩu và bước nhập mã 2FA; việc sai mật khẩu Microsoft do Microsoft tự xử lý.
- **BR-05**: Mật khẩu tối thiểu 8 ký tự, có chữ và số; lưu bằng bcrypt.
- **BR-06**: Đổi mật khẩu cần mật khẩu cũ; đổi xong thì đăng xuất các phiên khác.
- **BR-07**: Đổi vai trò hoặc nhóm có hiệu lực ở request kế tiếp (backend kiểm tra quyền từ DB hoặc cache ngắn hạn, không tin hoàn toàn vào token).
- **BR-08**: Một kỹ thuật viên có thể thuộc nhiều nhóm. Không xóa nhóm còn ticket đang mở.
- **BR-09**: Mọi thay đổi người dùng / vai trò / nhóm / service account / đề xuất AI / ánh xạ Microsoft ghi audit log: ai (`actor_type` = `USER`, `AI`, hoặc `SYSTEM` khi do đồng bộ Microsoft), làm gì, lúc nào, giá trị trước và sau.
- **BR-10**: Người dùng được tạo bằng mật khẩu tạm có `must_change_password = true`; phải đổi mật khẩu trước khi dùng chức năng khác.
- **BR-11**: Quên mật khẩu: luôn trả cùng một thông báo dù email có tồn tại hay không. Link đặt lại dùng 1 lần, hết hạn sau 30 phút. Đặt lại thành công → thu hồi mọi refresh token và gỡ khóa đăng nhập.
- **BR-12**: 2FA (TOTP) bắt buộc với ADMIN, tùy chọn với TECHNICIAN và REQUESTER. ADMIN không tự tắt được 2FA. Admin reset 2FA của người khác → người đó bị đăng xuất mọi phiên.
- **BR-13**: API key của service account chỉ hiển thị **một lần** khi tạo hoặc xoay; chỉ lưu hash. Xoay key hoặc tắt service account → key cũ vô hiệu ngay.
- **BR-14**: AI chỉ được **đề xuất**, không tự áp dụng. Đề xuất chỉ được thuộc 2 loại: thêm / bớt danh mục của nhóm (`GROUP_CATEGORY`) và thêm / bớt kỹ năng của kỹ thuật viên (`USER_SKILL`). AI không được đề xuất thay đổi vai trò, trạng thái hay quyền.
- **BR-15**: Mỗi đề xuất AI phải có lý do và danh sách ticket làm bằng chứng. Chỉ ADMIN duyệt; duyệt → hệ thống áp dụng thay đổi và ghi audit log với actor là Admin, kèm `suggestion_id`. Từ chối cũng được lưu lại để AI học (không đề xuất lại cùng nội dung trong 30 ngày).
- **BR-16**: Tài khoản do Admin tạo; hệ thống **không tự tạo** tài khoản khi đăng nhập Microsoft. Chỉ chấp nhận tài khoản Microsoft thuộc tenant của tổ chức (cấu hình `MICROSOFT_TENANT_ID`). Liên kết theo `microsoft_oid`; lần đầu thì khớp email (không phân biệt hoa thường) với `User` Admin đã tạo. Không khớp → từ chối "Tài khoản chưa được cấp quyền truy cập". Tài khoản `DISABLED` → từ chối (BR-03).
- **BR-17**: Đăng nhập Microsoft: ADMIN vẫn phải nhập mã 2FA của hệ thống (BR-12); vai trò khác không hỏi thêm mã. Người dùng không có mật khẩu (`password_hash` rỗng) chỉ đăng nhập được bằng Microsoft: không đăng nhập bằng mật khẩu, không đổi mật khẩu, không nhận email quên mật khẩu.
- **BR-18**: Đồng bộ nhóm Microsoft chạy cho mọi user đã liên kết `microsoft_oid`: mỗi lần đăng nhập Microsoft, định kỳ mỗi 1 giờ, và khi Admin bấm "Đồng bộ ngay". Danh sách nhóm lấy từ Microsoft Graph (kể cả nhóm lồng nhau).
  - **Vai trò:** user thuộc ≥ 1 nhóm Microsoft được ánh xạ tới vai trò → vai trò = vai trò cao nhất (ADMIN > TECHNICIAN > REQUESTER), `role_source = MICROSOFT`. Không thuộc nhóm nào được ánh xạ → giữ vai trò Admin gán, `role_source = MANUAL`.
  - **Nhóm kỹ thuật:** chỉ thêm / bớt các dòng `GroupMember` có `source = MICROSOFT`; dòng `MANUAL` do Admin thêm giữ nguyên. Chỉ thêm user có vai trò TECHNICIAN; user khác bị bỏ qua và ghi vào kết quả đồng bộ.
  - Thay đổi do đồng bộ có hiệu lực ở request kế tiếp (BR-07) và ghi audit log `actor_type = SYSTEM` (BR-09).
- **BR-19**: Vai trò `role_source = MICROSOFT` và thành viên nhóm `source = MICROSOFT` không sửa tay được (`409` `MANAGED_BY_MICROSOFT`); muốn đổi thì đổi nhóm bên Microsoft hoặc bỏ ánh xạ. Đồng bộ không được hạ vai trò ADMIN đang hoạt động cuối cùng (BR-02): bỏ qua thay đổi đó và báo lỗi trong kết quả đồng bộ.
- **BR-20**: Khi đồng bộ (BR-18) thấy tài khoản Microsoft của user đã liên kết bị **tắt** (`accountEnabled = false`) hoặc **bị xóa** (Graph trả không tồn tại) → hệ thống tự gỡ người đó:
  - `status = DISABLED`, `disabled_reason = MICROSOFT_REMOVED`, thu hồi mọi refresh token (BR-03);
  - xóa mọi dòng `GroupMember` của người đó (cả `MANUAL` và `MICROSOFT`);
  - **không xóa bản ghi `User`** để giữ lịch sử ticket và audit log;
  - ghi audit log `USER_REMOVED_BY_MICROSOFT` (`actor_type = SYSTEM`) và ghi số ticket đang mở của người đó vào kết quả đồng bộ (việc chuyển ticket thuộc module Ticket / Workflow).
  - Ngoại lệ: không gỡ ADMIN đang hoạt động cuối cùng (BR-02) — bỏ qua và báo lỗi nổi bật trong kết quả đồng bộ.
  - Tài khoản Microsoft được bật lại → **không** tự bật lại; Admin bật tay (`user:disable`).

## 7. Tiêu chí hoàn thành MVP
- Cả 4 vai trò đăng nhập (hoặc gọi API với AI) và chỉ thấy đúng dữ liệu trong phạm vi.
- Mọi API của 3 module kiểm tra quyền; gọi sai quyền trả `403`.
- AI_AGENT gửi được đề xuất; Admin duyệt thì dữ liệu phân luồng (danh mục nhóm, kỹ năng) thay đổi; AI không có đường nào tự sửa người dùng, nhóm hay vai trò.
- Tất cả user story trong [user_stories.md](user_stories.md) đạt acceptance criteria.

## 8. Định hướng: AI học dần từ ticket

Vòng lặp học (phần xử lý nằm ở module Ticket và Xử lý ticket bằng AI; 3 module này cung cấp nền tảng):

1. Ticket vào → AI phân loại, chọn nhóm / kỹ thuật viên dựa trên **danh mục nhóm** và **kỹ năng kỹ thuật viên**.
2. Kỹ thuật viên xác nhận hoặc sửa phân loại / chuyển nhóm *(module Ticket)* → đây là nhãn đúng để AI học.
3. AI tổng hợp các lần bị sửa, nhận ra mẫu mới (ví dụ ticket loại X luôn bị chuyển sang nhóm Y, kỹ thuật viên Z luôn giải quyết tốt loại W) và mẫu ticket phức tạp (nhiều vấn đề, cần nhiều nhóm).
4. AI gửi **đề xuất** (BR-14) → Admin duyệt (BR-15) → danh mục / kỹ năng cập nhật → lần phân luồng sau chính xác hơn.

3 module này đóng góp:

| Điểm | Vai trò trong việc học |
|---|---|
| `TechnicianGroup.categories` | Đầu vào phân luồng; AI đề xuất bổ sung |
| `User.skills` | Đầu vào chọn kỹ thuật viên; AI đề xuất bổ sung |
| `AiSuggestion` | Kênh duy nhất để AI thay đổi dữ liệu phân luồng, có người duyệt |
| `AuditLog.actor_type` | Tách hành động của AI và người để đo độ chính xác của AI |
| Ma trận quyền | Giới hạn AI: đọc rộng, ghi chỉ qua đề xuất |

## 9. Giả định (cần xác nhận)
- "MVP làm hết" được hiểu là: đưa 2FA, quên mật khẩu, đăng nhập Microsoft, quản lý service account, đề xuất AI vào MVP.
- Đăng nhập Microsoft dùng **một tenant** (một tổ chức). App đăng ký trên Entra cần quyền ứng dụng Microsoft Graph `GroupMember.Read.All` và `User.Read.All` (Admin tenant cấp quyền).
- Chỉ đồng bộ **vai trò và nhóm kỹ thuật**. Họ tên, SĐT, avatar không đồng bộ. "Tự gỡ" người bị tắt / xóa bên Microsoft được hiểu là tắt tài khoản + gỡ khỏi nhóm, không xóa hẳn dữ liệu (BR-20). Chậm nhất 1 giờ sau khi bị tắt bên Microsoft thì bị gỡ; trong thời gian đó người đó cũng không đăng nhập Microsoft mới được (Microsoft chặn), nhưng phiên đang mở vẫn dùng được tới lần đồng bộ.
- User chưa đăng nhập Microsoft lần nào (chưa có `microsoft_oid`) chưa được đồng bộ; vai trò ban đầu do Admin chọn khi tạo.
- Vẫn giữ **một người một vai trò** và **quyền cố định trong code**. Nhiều vai trò / vai trò tùy chỉnh làm ma trận quyền phức tạp hơn mà chưa có nhu cầu cụ thể.
- `skills` và `categories` là danh sách chuỗi tự do do Admin nhập (chưa có danh mục chuẩn). Nếu module Ticket có danh mục chuẩn thì dùng chung.
- Dữ liệu ticket được dùng để AI học trong nội bộ hệ thống; chưa có yêu cầu ẩn dữ liệu cá nhân (PII) trước khi học.
- Hệ thống có dịch vụ gửi email (SMTP) cho chức năng quên mật khẩu.

Xem thêm: [use_cases.md](use_cases.md) · [user_stories.md](user_stories.md) · [information_architecture.md](information_architecture.md)
