# Business Rules — Workflow & tự động hóa (V1)

> Trạng thái thiết kế: TARGET V1
> Trạng thái cài đặt: CHƯA TRIỂN KHAI

Chỉ các quy tắc đã được xác nhận trong tài liệu dự án được liệt kê ở đây.

## BR-WF-01 — Chỉ tự động thực thi hành động trong whitelist

- **Nguồn:** `03-use-cases.md` UC-WF-03, UC-WF-08
- **US liên quan:** US-WF-06, US-WF-17
- **API liên quan:** `createAutomationAction`, `updateAutomationAction`, thực thi nội bộ
- **Quy tắc:** Hệ thống chỉ tự động gọi connector cho các mục whitelist đang bật. V1 gồm `CHECK_ACCOUNT`, `CREATE_ACCOUNT`, `GRANT_ACCESS`, `REVOKE_ACCESS`. Mọi yêu cầu khác được chuyển cho kỹ thuật viên.

## BR-WF-02 — AI chỉ duyệt khi đạt kiểm tra chính sách cố định

- **Nguồn:** UC-WF-07, AC-UC-WF-07
- **US liên quan:** US-WF-07, US-WF-15
- **API liên quan:** nội bộ (AI decision), `updateDepartmentEntitlements`
- **Quy tắc:** AI Service được tự duyệt yêu cầu cơ bản, nhưng chỉ khi **tất cả** điều kiện chính sách (không dùng AI) đạt: thao tác trong whitelist đang bật; quyền / mẫu tài khoản thuộc danh mục của phòng ban người nhận; thời hạn ≤ tối đa; người nhận hợp lệ (BR-WF-16). AI không được duyệt ngoài phạm vi này dù độ tin cậy cao đến đâu.

## BR-WF-03 — Yêu cầu cần con người luôn chuyển kỹ thuật viên

- **Nguồn:** UC-WF-04, UC-WF-05
- **US liên quan:** US-WF-10
- **API liên quan:** nội bộ (routing)
- **Quy tắc:** Yêu cầu phần cứng (cấp chuột, bàn phím, sửa máy…), yêu cầu ngoài whitelist, yêu cầu không đạt chính sách, và yêu cầu vẫn chưa rõ sau số lượt hỏi tối đa luôn được chuyển cho kỹ thuật viên. AI không thực hiện, chỉ tóm tắt và nêu lý do chuyển.

## BR-WF-04 — Thực thi idempotent

- **Nguồn:** AC-UC-WF-08, AC-UC-WF-09
- **US liên quan:** US-WF-17, US-WF-20
- **API liên quan:** `retryWorkflowRun`, connector
- **Quy tắc:** Khóa idempotency = `decisionId + action`. Một quyết định duyệt dẫn tới tối đa một thay đổi trên hệ thống cấp quyền bên ngoài, kể cả khi thử lại.

## BR-WF-05 — Nguồn hành động trong nhật ký

- **Nguồn:** `08-invest-requirements.md` ràng buộc dùng chung
- **US liên quan:** US-WF-01 … US-WF-20
- **API liên quan:** mọi thao tác thay đổi
- **Quy tắc:** Mỗi bản ghi nhật ký có `source` ∈ {`USER`, `AI`, `AUTOMATION`}. `USER` gắn với tài khoản người dùng; `AI` là actor AI Service; `AUTOMATION` là tác vụ nội bộ của hệ thống (quy tắc, bộ hẹn giờ, thực thi API) — **không phải actor**. Bản ghi `AI`/`AUTOMATION` phải có căn cứ (`decisionId`, `ruleId`, `slaPolicyVersion` hoặc `runId`).

## BR-WF-06 — Ngưỡng tin cậy và hỏi lại

- **Nguồn:** UC-WF-04, AC-UC-WF-04
- **US liên quan:** US-WF-05, US-WF-09
- **API liên quan:** `updateAiActionSetting`
- **Quy tắc:** Độ tin cậy dưới ngưỡng hoặc thiếu thông tin bắt buộc → AI hỏi lại người gửi. Tối đa **2 lượt** (giá trị đề xuất, Admin cấu hình được); vẫn chưa đạt → chuyển kỹ thuật viên.

## BR-WF-07 — Tạm dừng SLA

- **Nguồn:** AC-UC-WF-02
- **US liên quan:** US-WF-04
- **API liên quan:** nội bộ (SLA engine)
- **Quy tắc:** Đồng hồ SLA tạm dừng khi ticket ở `NEEDS_INFO` và chạy tiếp khi người gửi trả lời. Ở `TRIAGE` đồng hồ vẫn chạy.

## BR-WF-08 — Phiên bản chính sách SLA

- **Nguồn:** UC-WF-02
- **US liên quan:** US-WF-03
- **API liên quan:** `updateSlaPolicy`
- **Quy tắc:** Mỗi lần lưu tạo phiên bản mới. Ticket gắn với phiên bản tại thời điểm tạo, và chỉ đổi phiên bản khi đổi mức ưu tiên.

## BR-WF-09 — Thứ tự quy tắc phân công

- **Nguồn:** AC-UC-WF-01, AC-UC-WF-05
- **US liên quan:** US-WF-01, US-WF-02, US-WF-10
- **API liên quan:** `reorderAssignmentRules`, `dryRunAssignmentRules`
- **Quy tắc:** Quy tắc đang bật được xét theo thứ tự tăng dần; quy tắc khớp đầu tiên thắng. Không khớp quy tắc nào → `TRIAGE`.

## BR-WF-10 — Kỹ thuật viên khả dụng

- **Nguồn:** AC-UC-WF-05
- **US liên quan:** US-WF-10
- **API liên quan:** nội bộ (assignment engine)
- **Quy tắc:** Chỉ giao cho thành viên của nhóm đích, tài khoản đang hoạt động và không đánh dấu vắng mặt.

## BR-WF-11 — Không tự đóng ticket đang tự động hóa

- **Nguồn:** AC-UC-WF-06
- **US liên quan:** US-WF-14
- **API liên quan:** nội bộ (scheduler)
- **Quy tắc:** Tác vụ tự đóng chỉ áp dụng cho `NEEDS_INFO` quá hạn; không bao giờ đóng ticket `AUTOMATING`.

## BR-WF-12 — Kiểm tra lại chính sách khi thực thi

- **Nguồn:** UC-WF-08, UC-WF-09
- **US liên quan:** US-WF-17, US-WF-20
- **API liên quan:** `retryWorkflowRun`
- **Quy tắc:** Mỗi lần thực thi hoặc thử lại đều kiểm tra lại whitelist và danh mục quyền **hiện hành**. Nếu Admin đã thu hẹp phạm vi sau khi AI duyệt, lần thực thi bị chặn.

## BR-WF-13 — Xác minh trước khi đánh dấu đã xử lý

- **Nguồn:** AC-UC-WF-09
- **US liên quan:** US-WF-19, US-WF-20
- **API liên quan:** `resolveWorkflowRunManually`, `confirmTicketResolution`
- **Quy tắc:** Ticket tự động hóa chỉ chuyển `RESOLVED` khi thao tác kiểm tra của connector thành công hoặc kỹ thuật viên ghi nhận xử lý thủ công có ghi chú.

## BR-WF-14 — Thu hồi quyền có thời hạn

- **Nguồn:** UC-WF-08
- **US liên quan:** US-WF-18
- **API liên quan:** nội bộ (scheduler)
- **Quy tắc:** Quyền cấp có `expires_at` được thu hồi tự động khi hết hạn bằng một `workflow_run` `REVOKE_ACCESS`; việc thu hồi phải ghi nhật ký.

## BR-WF-15 — Bảo vệ thông tin bí mật của connector

- **Nguồn:** AC-UC-WF-03, AC-UC-WF-08
- **US liên quan:** US-WF-06, US-WF-17
- **API liên quan:** mọi thao tác whitelist và workflow run
- **Quy tắc:** Credential của connector chỉ nằm trong cấu hình bí mật lúc triển khai. Payload lưu vào `workflow_run_attempts` phải che token, mật khẩu và khóa.

## BR-WF-16 — Người nhận hợp lệ khi AI tự duyệt

- **Nguồn:** UC-WF-07, AC-UC-WF-07
- **US liên quan:** US-WF-15
- **API liên quan:** nội bộ (AI decision)
- **Quy tắc:** AI chỉ tự duyệt khi người nhận quyền / tài khoản là chính người gửi, hoặc thuộc cùng phòng ban với người gửi. Yêu cầu cho người khác phòng ban luôn chuyển kỹ thuật viên.

## BR-WF-17 — Lưu căn cứ quyết định của AI

- **Nguồn:** UC-WF-04, UC-WF-07
- **US liên quan:** US-WF-16
- **API liên quan:** `listAiDecisions`, `getAiDecision`
- **Quy tắc:** Mỗi quyết định của AI lưu độ tin cậy, thông tin trích xuất, kết quả từng điều kiện chính sách, phiên bản cấu hình và lý do. Bản ghi không được sửa sau khi tạo.
