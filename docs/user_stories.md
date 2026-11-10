# User stories

Mỗi story kèm acceptance criteria (AC). Use case liên quan: [use_cases.md](use_cases.md). Quy tắc `BR-xx`: [MVP.md](MVP.md).

## Đăng nhập & phân quyền

**US-01 — Đăng nhập** (UC-01)
Là người dùng, tôi muốn đăng nhập bằng email và mật khẩu để dùng hệ thống theo đúng vai trò của mình.
- AC1: Đúng thông tin → vào trang chủ đúng vai trò (Admin → Dashboard, Technician → Ticket được giao, Requester → Ticket của tôi).
- AC2: Sai thông tin → thông báo lỗi chung, không tiết lộ email có tồn tại hay không.
- AC3: Sai 5 lần liên tiếp → bị khóa 15 phút.
- AC4: Tài khoản bị tắt → không đăng nhập được.
- AC5: Đăng nhập lần đầu bằng mật khẩu tạm → bắt buộc đổi mật khẩu; gọi API khác trước khi đổi → `403` với `code = PASSWORD_CHANGE_REQUIRED`.

**US-18 — Đăng nhập bằng Microsoft** (UC-17)
Là nhân viên, tôi muốn đăng nhập bằng tài khoản Microsoft công ty để không phải nhớ thêm mật khẩu.
- AC1: Tài khoản Microsoft thuộc tenant, email khớp user Admin đã tạo → vào đúng user đó, user được liên kết `microsoft_oid`, audit log `MICROSOFT_LINKED`.
- AC2: Email chưa được Admin tạo → bị từ chối "Tài khoản chưa được cấp quyền truy cập", không có user nào được tạo.
- AC3: Tài khoản Microsoft ngoài tenant → bị từ chối, không tạo user.
- AC4: User bị tắt đăng nhập bằng Microsoft → bị từ chối.
- AC5: ADMIN đăng nhập bằng Microsoft → vẫn phải nhập mã 2FA.
- AC6: User chỉ dùng Microsoft gọi `POST /auth/login` bằng mật khẩu hoặc `POST /me/password` → bị chặn (lỗi đăng nhập chung / `422`).

- AC7: Đăng nhập xong → vai trò và nhóm kỹ thuật đã được đồng bộ theo nhóm Microsoft (xem US-19), trang chủ theo vai trò mới.

**US-19 — Đồng bộ vai trò & nhóm từ Microsoft** (UC-18)
Là Admin, tôi muốn vai trò và nhóm kỹ thuật của mỗi người tự theo nhóm Microsoft họ tham gia, để không phải cập nhật tay hai nơi.
- AC1: Ánh xạ nhóm Microsoft "IT-Helpdesk" → vai trò TECHNICIAN và → nhóm kỹ thuật "Hỗ trợ IT". Người trong "IT-Helpdesk" đăng nhập → thành TECHNICIAN, có trong "Hỗ trợ IT" với `source = MICROSOFT`.
- AC2: Người thuộc 2 nhóm ánh xạ tới TECHNICIAN và ADMIN → vai trò ADMIN.
- AC3: Người bị gỡ khỏi nhóm Microsoft → sau lần đồng bộ kế tiếp (tối đa 1 giờ hoặc bấm "Đồng bộ ngay") mất vai trò / nhóm tương ứng; thành viên `MANUAL` vẫn giữ.
- AC4: Admin sửa tay vai trò hoặc bớt thành viên đang do Microsoft quản lý → `409` `MANAGED_BY_MICROSOFT`.
- AC5: Đồng bộ làm mất Admin đang hoạt động cuối cùng → không đổi, báo lỗi trong kết quả đồng bộ.
- AC6: Mọi thay đổi do đồng bộ có audit log `actor_type = SYSTEM` với giá trị trước và sau.
- AC7: TECHNICIAN hoặc AI_AGENT gọi API ánh xạ / đồng bộ → `403`.

**US-20 — Tự gỡ người rời tổ chức** (UC-18)
Là Admin, tôi muốn người bị tắt hoặc xóa tài khoản Microsoft tự bị gỡ khỏi hệ thống, để nhân viên đã nghỉ không còn truy cập được.
- AC1: Tài khoản Microsoft bị tắt → sau lần đồng bộ kế tiếp, user có `status = DISABLED`, `disabled_reason = MICROSOFT_REMOVED`, mọi request tiếp theo bằng token cũ bị từ chối.
- AC2: Tài khoản Microsoft bị xóa → kết quả như AC1.
- AC3: User bị gỡ khỏi mọi nhóm kỹ thuật (cả thành viên thêm tay); bản ghi user, ticket và audit log cũ vẫn còn.
- AC4: Kết quả đồng bộ liệt kê người bị gỡ và số ticket đang mở của họ; audit log `USER_REMOVED_BY_MICROSOFT` với `actor_type = SYSTEM`.
- AC5: Người bị gỡ là ADMIN đang hoạt động cuối cùng → không gỡ, báo lỗi trong kết quả đồng bộ.
- AC6: Tài khoản Microsoft được bật lại → user vẫn `DISABLED` cho tới khi Admin bật tay.

**US-02 — Đăng xuất** (UC-02)
Là người dùng, tôi muốn đăng xuất để bảo vệ tài khoản trên máy dùng chung.
- AC1: Sau khi đăng xuất, refresh token cũ không dùng được nữa.

**US-03 — Chỉ thấy ticket trong phạm vi** (UC-03)
Là người gửi ticket, tôi chỉ muốn thấy ticket của mình. Là kỹ thuật viên, tôi muốn thấy ticket giao cho mình và nhóm mình.
- AC1: REQUESTER gọi danh sách ticket → chỉ nhận ticket do mình tạo.
- AC2: TECHNICIAN mở ticket không thuộc mình / nhóm mình → `403`.
- AC3: ADMIN thấy tất cả ticket.

**US-04 — AI truy cập bằng service account** (UC-03)
Là hệ thống AI, tôi cần gọi API bằng API key để đọc ticket, nhóm và người dùng phục vụ phân luồng.
- AC1: API key hợp lệ → đọc được ticket, nhóm, người dùng (kèm `skills`).
- AC2: AI gọi API tạo / sửa người dùng, gán vai trò hoặc sửa nhóm → `403`.
- AC3: Mọi request của AI được ghi log với `actor_type = AI`.

**US-13 — Quên mật khẩu** (UC-12)
Là người dùng, tôi muốn đặt lại mật khẩu qua email khi quên để không phải nhờ Admin.
- AC1: Nhập email bất kỳ → luôn nhận cùng một thông báo.
- AC2: Link đặt lại dùng được 1 lần trong 30 phút; dùng lại hoặc quá hạn → báo không còn hiệu lực.
- AC3: Đặt lại thành công → mọi phiên cũ bị đăng xuất, tài khoản được gỡ khóa.
- AC4: Tài khoản bị tắt hoặc chỉ dùng Microsoft → không nhận được email đặt lại.

**US-14 — Xác thực 2 lớp** (UC-01, UC-13)
Là người dùng, tôi muốn bật 2FA để tài khoản an toàn hơn.
- AC1: Quét QR, nhập đúng mã → 2FA bật; lần đăng nhập sau phải nhập mã.
- AC2: Sai mã 2FA 5 lần → khóa 15 phút.
- AC3: ADMIN bắt buộc có 2FA; ADMIN gọi API tắt 2FA của mình → `403`.
- AC4: Admin reset 2FA cho người khác → người đó bị đăng xuất mọi phiên, có audit log `2FA_RESET`.

## Quản lý người dùng

**US-05 — Xem danh sách người dùng** (UC-04)
Là Admin, tôi muốn tìm và lọc người dùng để quản lý nhanh.
- AC1: Tìm theo tên / email, lọc theo vai trò, trạng thái, nhóm, kỹ năng; có phân trang.
- AC2: TECHNICIAN hoặc REQUESTER truy cập → `403`.

**US-06 — Thêm người dùng** (UC-05)
Là Admin, tôi muốn tạo tài khoản mới và gán vai trò.
- AC1: Tạo thành công → người dùng đăng nhập được bằng mật khẩu tạm và bị yêu cầu đổi ngay.
- AC2: Email trùng → báo lỗi `409`, không tạo.
- AC3: Có bản ghi audit log `USER_CREATED`.
- AC4: AI_AGENT gọi API tạo người dùng → `403`.
- AC5: Bỏ trống mật khẩu tạm → người đó đăng nhập được bằng Microsoft (email khớp), không đăng nhập được bằng mật khẩu.

**US-07 — Sửa người dùng, đổi vai trò & kỹ năng** (UC-06)
Là Admin, tôi muốn đổi thông tin, kỹ năng và vai trò của người dùng để AI phân việc đúng người.
- AC1: Đổi vai trò → quyền mới áp dụng ngay ở request tiếp theo của người đó.
- AC2: Không thể hạ vai trò của Admin cuối cùng hoặc của chính mình.
- AC3: Audit log có giá trị trước và sau.
- AC4: Kỹ năng mới hiện ngay khi AI đọc `GET /users/:id`.

**US-08 — Bật / tắt tài khoản** (UC-07)
Là Admin, tôi muốn tắt tài khoản người đã nghỉ để họ không truy cập được nữa.
- AC1: Tắt → người đó bị đăng xuất ngay ở request tiếp theo.
- AC2: Không tắt được chính mình hoặc Admin cuối cùng.
- AC3: Tắt TECHNICIAN còn ticket đang mở → hiện cảnh báo số ticket.

**US-09 — Quản lý nhóm kỹ thuật** (UC-08)
Là Admin, tôi muốn tạo nhóm kỹ thuật và thêm thành viên để AI phân luồng ticket đúng nhóm.
- AC1: Tạo / sửa nhóm với tên, mô tả, danh mục phụ trách.
- AC2: Thêm / bớt thành viên; một kỹ thuật viên có thể ở nhiều nhóm.
- AC3: Không xóa được nhóm còn ticket đang mở.
- AC4: TECHNICIAN gọi API sửa nhóm → `403`.

**US-15 — Quản lý service account AI** (UC-14)
Là Admin, tôi muốn tạo và xoay API key cho AI để kiểm soát việc AI truy cập hệ thống.
- AC1: Tạo / xoay → API key hiển thị đúng một lần; lần xem sau chỉ thấy tên, trạng thái, lần dùng cuối.
- AC2: Xoay key hoặc tắt service account → request bằng key cũ trả `401`.
- AC3: AI_AGENT gọi API quản lý service account → `403`.

**US-16 — AI đề xuất cải thiện phân luồng** (UC-15)
Là hệ thống AI, tôi muốn gửi đề xuất bổ sung danh mục cho nhóm hoặc kỹ năng cho kỹ thuật viên dựa trên ticket đã xử lý, để lần phân luồng sau chính xác hơn.
- AC1: Gửi đề xuất `GROUP_CATEGORY` / `USER_SKILL` có lý do và ticket bằng chứng → tạo ở trạng thái `PENDING`, dữ liệu nhóm / người dùng chưa đổi.
- AC2: Gửi đề xuất đổi vai trò hoặc trạng thái → `422`.
- AC3: Gửi lại nội dung đã bị từ chối trong 30 ngày → `409`.
- AC4: TECHNICIAN hoặc ADMIN gọi API tạo đề xuất → `403`.

**US-17 — Duyệt đề xuất của AI** (UC-16)
Là Admin, tôi muốn xem và duyệt đề xuất của AI để hệ thống thông minh dần mà vẫn trong tầm kiểm soát.
- AC1: Danh sách đề xuất `PENDING` hiện nội dung thay đổi, lý do, độ tin cậy, link ticket bằng chứng.
- AC2: Duyệt → danh mục nhóm / kỹ năng được cập nhật; audit log có actor là Admin và `suggestion_id`.
- AC3: Từ chối → lưu lý do; đề xuất không còn trong danh sách chờ.
- AC4: Duyệt đề xuất đã xử lý → `409`.
- AC5: AI_AGENT gọi API duyệt → `403`.

## Hồ sơ cá nhân

**US-10 — Cập nhật hồ sơ** (UC-09)
Là người dùng, tôi muốn sửa họ tên, SĐT, avatar.
- AC1: Lưu thành công và hiển thị ngay.
- AC2: Gửi kèm email, vai trò, trạng thái hoặc kỹ năng → trả `422`, không thay đổi gì.

**US-11 — Đổi mật khẩu** (UC-10)
Là người dùng, tôi muốn đổi mật khẩu để giữ tài khoản an toàn.
- AC1: Sai mật khẩu cũ → báo lỗi.
- AC2: Mật khẩu mới không đạt chính sách → báo lỗi cụ thể.
- AC3: Đổi thành công → các phiên khác bị đăng xuất.

**US-12 — Xem vai trò & quyền** (UC-11)
Là người dùng, tôi muốn biết mình có quyền gì để hiểu vì sao thấy hoặc không thấy một chức năng.
- AC1: Hiển thị vai trò, nhóm, kỹ năng, trạng thái 2FA, trạng thái liên kết Microsoft và danh sách quyền bằng câu dễ hiểu.
- AC2: AI_AGENT gọi `/me/permissions` → `403` (service account không có hồ sơ).
