# Acceptance Criteria — Workflow & tự động hóa (V1)

> Trạng thái thiết kế: TARGET V1
>
> Trạng thái cài đặt: CHƯA TRIỂN KHAI — mọi tiêu chí là mục tiêu nghiệm thu

Các tiêu chí mô tả hành vi sản phẩm mục tiêu. Giao diện có thể thỏa tiêu chí về mặt hiển thị, nhưng phân quyền và business rule **phải** được thực thi ở API.

## AC-UC-WF-01 — Cấu hình quy tắc phân công

*(US-WF-01, US-WF-02)*

- Chỉ Admin tạo, sửa, bật/tắt, sắp thứ tự, lưu trữ quy tắc; vai trò khác nhận `403`.
- Quy tắc mới được lưu ở trạng thái Tắt và nằm cuối danh sách thứ tự.
- Nhóm đích không có thành viên hoạt động → `400`; tên trống hoặc không có điều kiện nào → `400`.
- Khi nhiều quy tắc cùng khớp, quy tắc có thứ tự nhỏ nhất được áp dụng.
- Chạy thử trả về quy tắc khớp, nhóm và kỹ thuật viên dự kiến; số bản ghi phân công và nhật ký không đổi sau khi chạy thử.
- Mỗi thao tác thay đổi tạo một bản ghi nhật ký `WORKFLOW_RULE_*` chứa giá trị trước và sau.

## AC-UC-WF-02 — Cấu hình SLA

*(US-WF-03, US-WF-04)*

- Chỉ Admin cập nhật chính sách SLA và lịch làm việc.
- Thời hạn giải quyết nhỏ hơn thời hạn phản hồi, hoặc ngưỡng cảnh báo ngoài 1–99% → `400`.
- Ticket tạo sau khi lưu dùng phiên bản mới; ticket đang mở giữ phiên bản đã gắn trừ khi đổi mức ưu tiên.
- Hạn chót bỏ qua ngoài giờ làm việc và ngày nghỉ: ticket P3 tạo lúc 16:30 thứ Sáu (giờ làm 08:00–17:00) có hạn phản hồi 11:30 thứ Hai.
- Thời gian ticket ở `NEEDS_INFO` không tính vào SLA; mỗi lần tạm dừng/tiếp tục được lưu với thời điểm.

## AC-UC-WF-03 — Cấu hình phạm vi xử lý của AI

*(US-WF-05, US-WF-06, US-WF-07)*

- Chỉ Admin thay đổi cấu hình; ngưỡng ngoài 0.00–1.00 hoặc số lượt hỏi ngoài 0–5 → `400`.
- Tắt `AUTO_APPROVE` → mọi ticket mới thuộc luồng A chuyển sang luồng C (con người) cho đến khi bật lại.
- Hành động AI bị tắt không được gọi cho ticket mới tạo sau thời điểm tắt.
- Mục whitelist tham chiếu connector không tồn tại → `400`.
- Danh mục quyền tham chiếu hệ thống không có connector → `400`.
- Thay đổi danh mục quyền có hiệu lực ngay với các quyết định và lần thực thi sau thời điểm lưu.
- API response và giao diện không bao giờ chứa thông tin bí mật của connector.

## AC-UC-WF-04 — Phân luồng ticket

*(US-WF-08, US-WF-09)*

- Mỗi ticket mới có đúng một bản ghi `ai_decisions` hiện hành với `outcome` ∈ {`AUTO_APPROVED`, `NEEDS_INFO`, `ROUTE_TO_HUMAN`}, độ tin cậy và lý do.
- Ticket *"Xin cấp quyền đọc thư mục Kế toán chung"* từ nhân viên phòng Kế toán, quyền có trong danh mục, độ tin cậy ≥ ngưỡng → luồng A.
- Ticket *"Chuột bị hỏng, cần cấp chuột mới"* → luồng C (cần con người), không tạo `workflow_run`.
- Độ tin cậy < ngưỡng hoặc thiếu trường bắt buộc → ticket `NEEDS_INFO`, người gửi nhận một câu hỏi cụ thể nêu rõ thông tin còn thiếu.
- Sau khi người gửi trả lời, ticket được phân luồng lại; sau 2 lượt hỏi vẫn chưa đạt → luồng C với lý do `MAX_CLARIFICATION_REACHED`.
- AI Service không phản hồi trong 30 giây → luồng C với lý do `AI_UNAVAILABLE`.
- Phân luồng hoàn tất ≤ 15 giây (P95) sau khi ticket được tạo hoặc người gửi trả lời.

## AC-UC-WF-05 — Tự động phân công kỹ thuật viên

*(US-WF-10, US-WF-11)*

- Ticket luồng C khớp quy tắc được giao cho một kỹ thuật viên đang hoạt động, không vắng mặt, thuộc nhóm đích, trong ≤ 10 giây (P95).
- Với `ROUND_ROBIN`, N ticket liên tiếp chia đều cho các thành viên khả dụng (chênh lệch ≤ 1); với `LEAST_LOADED`, chọn người có ít ticket mở nhất.
- Thông báo cho kỹ thuật viên có bản tóm tắt AI **và lý do chuyển người** (ví dụ *"Yêu cầu phần cứng"*, *"Quyền X không thuộc danh mục phòng ban"*).
- Không khớp quy tắc → ticket `TRIAGE` và trưởng nhóm tiếp nhận nhận thông báo.
- Phân công lại thiếu lý do → `400`; trưởng nhóm phân công ra ngoài nhóm mình → `403`.
- Nhật ký phân công tự động có nguồn `AUTOMATION` và `ruleId`.

## AC-UC-WF-06 — Nhắc và cảnh báo SLA

*(US-WF-12, US-WF-13, US-WF-14)*

- Khi đạt ngưỡng cảnh báo, người được giao nhận đúng một thông báo cho mỗi ngưỡng, trong ≤ 2 phút.
- Khi quá hạn, ticket được gắn `sla_breached = true`, trưởng nhóm được thông báo, và sự kiện `SLA_BREACHED` được ghi.
- Nếu bật nâng ưu tiên, ưu tiên tăng đúng một bậc và không vượt P1.
- Ticket `NEEDS_INFO` được nhắc ở ngày 1 và ngày N−1, tự chuyển `CLOSED` ở ngày N với lý do *không phản hồi*.
- Người gửi mở lại được ticket tự đóng trong 7 ngày; sau đó → `409`.
- Ticket `AUTOMATING` không bao giờ bị tự đóng.

## AC-UC-WF-07 — AI duyệt yêu cầu trong phạm vi

*(US-WF-15, US-WF-16)*

- Chỉ duyệt khi **tất cả** điều kiện chính sách đạt: thao tác trong whitelist đang bật; quyền / mẫu tài khoản trong danh mục phòng ban người nhận; thời hạn ≤ tối đa; người nhận là người gửi hoặc cùng phòng ban; người nhận chưa có quyền đó.
- Một điều kiện không đạt → `ROUTE_TO_HUMAN`, không có lời gọi connector thay đổi dữ liệu, lý do nêu đúng điều kiện bị trượt.
- Yêu cầu quyền cho người khác phòng ban → luồng C, kể cả khi độ tin cậy = 1.00.
- Người nhận đã có quyền → ticket `RESOLVED` với thông báo *"Bạn đã có quyền này"*, không gọi `GRANT_ACCESS`.
- Kết quả AI thiếu trường hoặc sai định dạng → luồng C; không bao giờ duyệt khi dữ liệu trích xuất không hợp lệ.
- Admin xem được mỗi quyết định với: độ tin cậy, thông tin trích xuất, kết quả từng điều kiện, phiên bản cấu hình đã dùng; người gửi chỉ thấy lý do tóm tắt.
- Quyết định duyệt hoàn tất ≤ 5 giây (P95) sau khi phân luồng xong.

## AC-UC-WF-08 — Thực thi qua API

*(US-WF-17, US-WF-18)*

- Connector chỉ được gọi khi quyết định `AUTO_APPROVED` và chính sách **tại thời điểm chạy** vẫn cho phép; nếu không → `FAILED` mã `PRECONDITION_FAILED`, không có lời gọi ra ngoài.
- `CHECK_ACCOUNT` không thay đổi dữ liệu bên ngoài; kết quả (tồn tại, bị khóa, quyền hiện có) được gửi cho người gửi.
- Lần thực thi đầu tiên bắt đầu ≤ 30 giây sau khi AI duyệt (P95).
- Gọi lại với cùng khóa idempotency không tạo thêm thay đổi trên hệ thống cấp quyền bên ngoài.
- Lỗi tạm thời được thử lại tối đa 3 lần (1 / 5 / 15 phút); lỗi vĩnh viễn không được thử lại tự động.
- Mỗi lần gọi lưu một `workflow_run_attempt` với mã kết quả, thời lượng và payload đã che dữ liệu nhạy cảm.
- Quyền có thời hạn được thu hồi trong ≤ 5 phút sau khi hết hạn; người dùng được báo trước 1 ngày.

## AC-UC-WF-09 — Xác minh và chuyển người khi lỗi

*(US-WF-19, US-WF-20)*

- Ticket chỉ chuyển `RESOLVED` khi xác minh thành công hoặc kỹ thuật viên đánh dấu xử lý thủ công.
- Người gửi xác nhận → `CLOSED`; báo chưa được → giao kỹ thuật viên nhóm phụ trách.
- Run `FAILED` → ticket được giao kỹ thuật viên kèm mã lỗi, bước lỗi và lịch sử các lần thử.
- Thử lại tạo run mới dùng **cùng** khóa idempotency và kiểm tra lại chính sách; chính sách không còn cho phép → `409`.
- Xử lý thủ công thiếu ghi chú → `400`.
- Chỉ kỹ thuật viên được giao hoặc Admin được thử lại / xử lý thủ công; người khác → `403`.

## Definition of Ready

Một work item backend chỉ sẵn sàng khi nêu rõ operationId trong OpenAPI, application service, repository phụ thuộc, quy tắc RBAC, kết quả kiểm tra/lỗi và danh sách unit test. Bảng [traceability](traceability/master-traceability.md) cung cấp ánh xạ này.
