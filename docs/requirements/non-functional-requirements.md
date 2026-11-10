# Non-Functional Requirements — Workflow & tự động hóa (V1)

> Trạng thái thiết kế: BASELINE V1 ĐỀ XUẤT
> Trạng thái cài đặt: TARGET; kiểm chứng là PLANNED trừ khi có bằng chứng

Các chỉ tiêu là baseline cho hệ thống nội bộ doanh nghiệp, không phải SLA thương mại. Yêu cầu chỉ được coi là hoàn thành khi phương pháp kiểm chứng đạt dưới tải tham chiếu.

## Tải tham chiếu

| Thông số | Giá trị | Ghi chú |
|---|---|---|
| Ticket mới | **8.000 / ngày** | Từ các phòng ban |
| Trung bình giờ làm việc (8 giờ) | ~1.000 / giờ ≈ 17 / phút | |
| Giờ cao điểm (giả định gấp 2 trung bình) | ~2.000 / giờ ≈ 35 / phút | Dùng để kiểm thử tải |
| Lượt hỏi lại của AI | tối đa 2 / ticket | Mỗi lượt là một lần phân luồng lại |
| Dữ liệu tích lũy | ~2,9 triệu ticket / năm | Kèm `ai_decisions`, `workflow_runs`, nhật ký |
| Số kỹ thuật viên | chưa xác định | Xem Câu hỏi mở trong README |

## Danh sách NFR

| ID | Phát biểu | Lý do | Đo lường | Phương pháp kiểm chứng | Trạng thái |
|---|---|---|---|---|---|
| NFR-WF-CAP-001 | Hệ thống xử lý được tải cao điểm **35 ticket/phút** liên tục trong 1 giờ mà vẫn đạt các chỉ tiêu PERF bên dưới. | Đáp ứng 8.000 ticket/ngày có dư. | Kiểm thử tải với 2.000 ticket/giờ. | `PERF-WF-LOAD-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-001 | Phân luồng (AI phân loại + quyết định luồng) hoàn tất ≤ 15 giây (P95). | Người gửi nhận phản hồi gần như ngay. | Thời điểm tạo ticket / trả lời và bản ghi `ai_decisions`. | `PERF-WF-ROUTE-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-002 | Kiểm tra chính sách và quyết định duyệt ≤ 5 giây (P95) sau phân luồng. | Duyệt tự động phải tức thì. | Chênh lệch thời điểm. | `PERF-WF-APPROVE-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-003 | Lần gọi connector đầu tiên bắt đầu ≤ 30 giây (P95) sau khi AI duyệt. | Người gửi nhận quyền nhanh. | Thời điểm duyệt và attempt đầu tiên. | `PERF-WF-EXEC-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-004 | Tự động phân công kỹ thuật viên hoàn tất ≤ 10 giây (P95) sau khi ticket vào luồng C. | Ticket cần người được nhận ngay. | Chênh lệch thời điểm. | `PERF-WF-ASSIGN-001` | PROPOSED / PLANNED |
| NFR-WF-PERF-005 | Cảnh báo SLA được gửi ≤ 2 phút sau khi đạt ngưỡng. | Nhắc phải kịp lúc mới có giá trị. | Chênh lệch ngưỡng lý thuyết và thời điểm gửi. | `INT-WF-SLA-002` | PROPOSED / PLANNED |
| NFR-WF-REL-001 | Ticket chờ phân luồng, bộ đếm SLA và run chờ thử lại được lưu bền vững (hàng đợi trong DB) và tiếp tục đúng sau khi khởi động lại dịch vụ. | Không mất ticket khi restart ở mức 8.000/ngày. | Kiểm thử khởi động lại giữa chừng. | `INT-WF-DURABLE-001` | PROPOSED / PLANNED |
| NFR-WF-REL-002 | Không có thay đổi trùng trên hệ thống cấp quyền bên ngoài khi gọi lặp, timeout hoặc restart. | Đảm bảo BR-WF-04. | Số thay đổi trên mock connector trong kiểm thử chèn lỗi. | `INT-WF-IDEM-001` | PROPOSED / PLANNED |
| NFR-WF-REL-003 | Thử lại lỗi tạm thời tối đa 3 lần với giãn cách 1 / 5 / 15 phút. | Chịu được sự cố ngắn của hệ thống bên ngoài. | Mốc thời gian các attempt. | `UNIT-APP-WF-RETRY-001` | PROPOSED / PLANNED |
| NFR-WF-REL-004 | Khi AI Service không khả dụng, ticket mới được chuyển luồng C (con người) trong ≤ 30 giây; không ticket nào bị kẹt. | Hệ thống vẫn vận hành khi AI lỗi. | Kiểm thử tắt AI Service. | `INT-WF-AIDOWN-001` | PROPOSED / PLANNED |
| NFR-WF-SAFE-001 | Tỷ lệ cấp quyền ngoài danh mục do AI duyệt = **0** trong bộ kiểm thử đối kháng (ticket cố tình mô tả sai, xin quyền phòng ban khác, prompt injection). | AI hiểu sai không được dẫn tới cấp sai. | Số `GRANT_ACCESS`/`CREATE_ACCOUNT` ngoài danh mục. | `SEC-WF-AIPOLICY-001` | PROPOSED / PLANNED |
| NFR-WF-SEC-001 | Credential connector chỉ nằm trong secret của môi trường triển khai, không trong DB, log, API response. | Giảm rủi ro lộ khóa. | Quét log/DB mẫu. | `SEC-WF-SECRET-001` | PROPOSED / PLANNED |
| NFR-WF-SEC-002 | Tài khoản dịch vụ của connector chỉ có quyền tối thiểu cho các thao tác trong whitelist. | Giới hạn thiệt hại nếu bị lạm dụng. | Rà soát quyền tài khoản dịch vụ. | `SEC-WF-LEASTPRIV-001` | PROPOSED / PLANNED |
| NFR-WF-SEC-003 | 100% endpoint workflow kiểm tra quyền ở server và khai báo Bearer security. | Trình duyệt không phải ranh giới bảo mật. | Độ phủ policy và kiểm tra contract. | `INT-WF-RBAC-*` | PROPOSED / PLANNED |
| NFR-WF-AUD-001 | 100% hành động có nguồn `AI`/`AUTOMATION` có bản ghi nhật ký kèm căn cứ. | Truy vết và giải trình được. | Đối chiếu sự kiện với nhật ký. | `INT-WF-AUDIT-001` | PROPOSED / PLANNED |
| NFR-WF-OBS-001 | Hệ thống đo được **tỷ lệ AI tự giải quyết**, tỷ lệ hỏi lại và tỷ lệ chuyển người theo ngày / loại yêu cầu. | Đánh giá hiệu quả AI (cung cấp dữ liệu cho F07). | Truy vấn trên `ai_decisions`. | `INT-WF-METRIC-001` | PROPOSED / PLANNED |
| NFR-WF-MAINT-001 | Độ phủ unit test cho kiểm tra chính sách, quy tắc phân công và tính SLA ≥ 90%. | Logic rủi ro cao. | Báo cáo coverage theo namespace. | CI coverage | PROPOSED / PLANNED |
| NFR-WF-EXT-001 | Thêm connector mới chỉ cần cài một interface `IAutomationConnector` (`Execute`, `Verify`), không sửa phần điều phối workflow. | Dễ mở rộng sang hệ thống cấp quyền khác. | Review kiến trúc. | `ARCH-WF-001` | PROPOSED / PLANNED |
