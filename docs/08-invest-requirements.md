# INVEST Requirements — Workflow & tự động hóa (V1)

> Trạng thái thiết kế: TARGET V1
>
> Phạm vi: F05 — US-WF-01 … US-WF-20

Mỗi user story đủ nhỏ để triển khai và kiểm thử độc lập. Xác thực, phân quyền, kiểm tra dữ liệu, ghi nhật ký và xử lý lỗi là **ràng buộc nghiệm thu dùng chung**, không phải tính năng ẩn.

## INVEST checklist

| Tiêu chí | Áp dụng trong backlog này |
|---|---|
| **Independent** | Mỗi story tạo ra một kết quả demo được; chỉ ghi phụ thuộc khi thật sự cần. |
| **Negotiable** | Giao diện và cách cài đặt có thể đổi, miễn mục tiêu, quyền và business rule giữ nguyên. |
| **Valuable** | Mỗi story nêu rõ lợi ích cho người dùng hoặc doanh nghiệp. |
| **Estimable** | Đầu vào, đầu ra, vai trò, phụ thuộc và kết quả lỗi đều được mô tả. |
| **Small** | Mỗi story một năng lực chính, tách được thành task API, application, persistence, UI và test. |
| **Testable** | Mỗi story liên kết tới tiêu chí nghiệm thu quan sát được trong [06-acceptance-criteria.md](06-acceptance-criteria.md). |

## Product backlog

Ưu tiên theo MoSCoW: **Must** (bắt buộc V1), **Should** (nên có), **Could** (làm nếu còn thời gian).

| ID | User story | Giá trị | Phụ thuộc | Ưu tiên | AC |
|---|---|---|---|---|---|
| US-WF-01 | Là **Admin**, tôi muốn tạo và sửa quy tắc phân công theo điều kiện (loại yêu cầu, danh mục, ưu tiên, phòng ban) trỏ tới một nhóm kỹ thuật và chiến lược chia việc, để ticket cần con người tự đến đúng nhóm. | Bỏ khâu điều phối tay | Nhóm kỹ thuật (F09) | Must | AC-UC-WF-01 |
| US-WF-02 | Là **Admin**, tôi muốn sắp thứ tự ưu tiên các quy tắc và chạy thử trên ticket mẫu, để biết trước kết quả phân công trước khi bật. | Giảm cấu hình sai | US-WF-01 | Should | AC-UC-WF-01 |
| US-WF-03 | Là **Admin**, tôi muốn định nghĩa SLA theo mức ưu tiên và lịch làm việc, để mỗi ticket có hạn xử lý rõ ràng. | Cam kết dịch vụ đo được | — | Must | AC-UC-WF-02 |
| US-WF-04 | Là **Kỹ thuật viên**, tôi muốn SLA tạm dừng khi AI đang chờ người gửi trả lời, để không bị tính vi phạm vì việc ngoài tầm kiểm soát. | SLA công bằng | US-WF-03 | Should | AC-UC-WF-02 |
| US-WF-05 | Là **Admin**, tôi muốn bật/tắt từng hành động AI (kể cả tự duyệt), đặt ngưỡng tin cậy và số lượt hỏi lại tối đa, để kiểm soát mức độ AI được tự quyết. | Kiểm soát rủi ro AI | AI Service (F04) | Must | AC-UC-WF-03 |
| US-WF-06 | Là **Admin**, tôi muốn khai báo danh mục hành động tự động (kiểm tra tài khoản, cấp tài khoản, cấp quyền) kèm connector và tham số, để chỉ những việc đã định nghĩa mới được AI tự xử lý. | Tự động hóa có giới hạn | Connector đã khai báo | Must | AC-UC-WF-03 |
| US-WF-07 | Là **Admin**, tôi muốn khai báo danh mục quyền và mẫu tài khoản mà AI được tự cấp cho từng phòng ban, kèm thời hạn tối đa, để AI không cấp quyền vượt phạm vi. | Ngăn cấp quyền sai | US-WF-06, phòng ban (F09) | Must | AC-UC-WF-03 |
| US-WF-08 | Là **Người gửi**, tôi muốn yêu cầu cơ bản của tôi (kiểm tra tài khoản, cấp tài khoản, cấp thêm quyền) được AI nhận diện và xử lý ngay, để không phải chờ kỹ thuật viên. | Giải quyết tức thì phần lớn ticket | US-WF-05, US-WF-06 | Must | AC-UC-WF-04 |
| US-WF-09 | Là **Người gửi**, tôi muốn khi AI chưa chắc chắn thì AI hỏi lại tôi câu hỏi cụ thể (tối đa 2 lượt), để yêu cầu được hiểu đúng thay vì bị xử lý sai. | Giảm xử lý sai | US-WF-08 | Must | AC-UC-WF-04 |
| US-WF-10 | Là **Kỹ thuật viên**, tôi muốn yêu cầu cần con người (phần cứng, ngoài phạm vi AI, không đạt chính sách) được tự động giao cho tôi kèm bản tóm tắt AI và lý do, để tôi xử lý ngay không cần đọc lại từ đầu. | Kỹ thuật viên chỉ nhận việc cần người | US-WF-01, US-WF-08 | Must | AC-UC-WF-05 |
| US-WF-11 | Là **Trưởng nhóm**, tôi muốn ticket không khớp quy tắc nào vào hàng chờ thủ công, và tôi có thể phân công lại có lý do, để không ticket nào bị bỏ sót hay giao sai. | Không bỏ sót ticket | US-WF-10 | Must | AC-UC-WF-05 |
| US-WF-12 | Là **Kỹ thuật viên**, tôi muốn được nhắc khi ticket đạt ngưỡng cảnh báo SLA, để xử lý trước khi quá hạn. | Giảm vi phạm SLA | US-WF-03 | Must | AC-UC-WF-06 |
| US-WF-13 | Là **Trưởng nhóm**, tôi muốn ticket vi phạm SLA được tự động chuyển cấp (thông báo, gắn cờ, nâng ưu tiên tùy cấu hình), để can thiệp kịp thời. | Phản ứng nhanh khi trễ | US-WF-03 | Must | AC-UC-WF-06 |
| US-WF-14 | Là **Người gửi**, tôi muốn được nhắc khi AI đang chờ tôi trả lời, và ticket tự đóng sau N ngày nếu tôi không trả lời (vẫn mở lại được), để ticket không treo vô thời hạn. | Hàng đợi sạch | US-WF-09 | Should | AC-UC-WF-06 |
| US-WF-15 | Là **Người gửi**, tôi muốn yêu cầu nằm trong danh mục cho phép của phòng ban được AI duyệt ngay sau khi kiểm tra chính sách, để tôi không phải chờ ai duyệt. | Duyệt trong vài giây | US-WF-07, US-WF-08 | Must | AC-UC-WF-07 |
| US-WF-16 | Là **Admin**, tôi muốn xem từng quyết định của AI kèm độ tin cậy, thông tin trích xuất và kết quả từng điều kiện chính sách, để kiểm tra lại và điều chỉnh cấu hình khi AI quyết định chưa đúng. | Giải trình được quyết định AI | US-WF-15 | Must | AC-UC-WF-07 |
| US-WF-17 | Là **Người gửi**, tôi muốn sau khi AI duyệt, hệ thống tự gọi API thực thi (không cấp trùng, tự thử lại khi lỗi tạm thời), để tôi có tài khoản / quyền ngay mà không cần ai thao tác tay. | Tự giải quyết không cần KTV | US-WF-15 | Must | AC-UC-WF-08 |
| US-WF-18 | Là **Admin**, tôi muốn quyền có thời hạn được tự động thu hồi khi hết hạn, để giảm rủi ro tồn đọng quyền. | Bảo mật | US-WF-17 | Could | AC-UC-WF-08 |
| US-WF-19 | Là **Người gửi**, tôi muốn hệ thống tự kiểm tra lại kết quả trên hệ thống cấp quyền bên ngoài rồi mời tôi xác nhận, để chắc chắn yêu cầu đã thực sự được xử lý. | Kết quả đáng tin | US-WF-17 | Must | AC-UC-WF-09 |
| US-WF-20 | Là **Kỹ thuật viên**, tôi muốn khi thực thi hoặc xác minh thất bại, ticket được chuyển cho tôi kèm log lỗi để tôi thử lại hoặc xử lý thủ công, để lỗi tự động hóa không bị bỏ quên. | Có người chịu trách nhiệm khi lỗi | US-WF-17, US-WF-10 | Must | AC-UC-WF-09 |

Nhóm theo use case:

```text
US-WF-01–02   UC-WF-01  Cấu hình quy tắc phân công
US-WF-03–04   UC-WF-02  Cấu hình SLA
US-WF-05–07   UC-WF-03  Cấu hình phạm vi xử lý của AI
US-WF-08–09   UC-WF-04  Phân luồng ticket (AI tự xử lý / hỏi lại)
US-WF-10–11   UC-WF-05  Tự động phân công kỹ thuật viên
US-WF-12–14   UC-WF-06  Nhắc & cảnh báo SLA
US-WF-15–16   UC-WF-07  AI duyệt yêu cầu trong phạm vi
US-WF-17–18   UC-WF-08  Thực thi qua API
US-WF-19–20   UC-WF-09  Xác minh / chuyển người khi lỗi
```

## Đánh giá INVEST từng story

`PASS` nghĩa là thiết kế hiện tại đủ thông tin để lập kế hoạch và nghiệm thu, **không** có nghĩa đã có code hay test.

| Story | I | N | V | E | S | T | Ghi chú |
|---|---|---|---|---|---|---|---|
| US-WF-01 | PASS | PASS | PASS | PASS | PASS | PASS | Điều kiện AND, quy tắc mới mặc định Tắt. |
| US-WF-02 | PASS | PASS | PASS | PASS | PASS | PASS | Chạy thử không ghi dữ liệu; khớp đầu tiên thắng. |
| US-WF-03 | PASS | PASS | PASS | PASS | PASS | PASS | Có bảng giá trị mặc định P1–P4; chính sách có phiên bản. |
| US-WF-04 | PASS | PASS | PASS | PASS | PASS | PASS | Trạng thái tạm dừng duy nhất: `NEEDS_INFO`. |
| US-WF-05 | PASS | PASS | PASS | PASS | PASS | PASS | 5 hành động AI, ngưỡng 0.00–1.00, số lượt hỏi mặc định 2. |
| US-WF-06 | PASS | PASS | PASS | PASS | PASS | PASS | 4 thao tác V1: `CHECK_ACCOUNT`, `CREATE_ACCOUNT`, `GRANT_ACCESS`, `REVOKE_ACCESS`. |
| US-WF-07 | PASS | PASS | PASS | PASS | PASS | PASS | Danh mục theo phòng ban; kiểm tra không dùng AI. |
| US-WF-08 | PASS | PASS | PASS | PASS | PASS | PASS | Ba luồng A/B/C có điều kiện rõ ràng. |
| US-WF-09 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Số lượt hỏi tối đa = 2 là giá trị đề xuất, chờ chốt. |
| US-WF-10 | PASS | PASS | PASS | PASS | PASS | PASS | Thông báo phải có lý do chuyển người. |
| US-WF-11 | PASS | PASS | PASS | PASS | PASS | PASS | Lý do phân công lại bắt buộc; SLA vẫn chạy ở `TRIAGE`. |
| US-WF-12 | PASS | PASS | PASS | PASS | PASS | PASS | Mỗi ngưỡng nhắc một lần. |
| US-WF-13 | PASS | PASS | PASS | PASS | PASS | PASS | Nâng ưu tiên là tùy chọn cấu hình. |
| US-WF-14 | PASS | PASS | PASS | PASS | PASS | PASS | N mặc định 5 ngày làm việc; mở lại trong 7 ngày. |
| US-WF-15 | PASS | NEEDS CONFIRM | PASS | PASS | PASS | PASS | Chưa chốt: quyền nhạy cảm có cần Admin duyệt hay không (xem Câu hỏi mở trong README). |
| US-WF-16 | PASS | PASS | PASS | PASS | PASS | PASS | Đọc từ `ai_decisions`, không cần AI chạy lại. |
| US-WF-17 | PASS | PASS | PASS | PASS | PASS | PASS | Khóa idempotency, 3 lần thử 1/5/15 phút; demo được với mock connector. |
| US-WF-18 | PASS | PASS | PASS | PASS | PASS | PASS | Thu hồi tái dùng cơ chế `workflow_run`. |
| US-WF-19 | PASS | PASS | PASS | PASS | PASS | PASS | Connector phải có thao tác kiểm tra. |
| US-WF-20 | PASS | PASS | PASS | PASS | PASS | PASS | Thử lại luôn kiểm tra lại chính sách hiện hành. |

## Ràng buộc nghiệm thu dùng chung

- Chỉ tồn tại 3 vai trò `ADMIN`, `TECHNICIAN`, `REQUESTER`; actor hệ thống duy nhất là **AI Service**.
- AI chỉ tự duyệt khi **kiểm tra chính sách cố định** đạt; API, không phải React client hay AI, là nơi thực thi RBAC, whitelist và danh mục quyền.
- Mọi hành động không do người dùng thực hiện đều ghi nhật ký với nguồn `AI` hoặc `AUTOMATION` và căn cứ (`decisionId`, `ruleId`, `runId`).
- Thông tin bí mật của connector không bao giờ xuất hiện trong UI, API response, log hay nhật ký.
- Lỗi API dùng chuẩn `ProblemDetails` (`400`, `403`, `404`, `409`).
- Thời gian lưu UTC; SLA tính theo giờ làm việc của lịch đã cấu hình.
- Hệ thống phải chịu được tải tham chiếu **8.000 ticket/ngày** (xem NFR).

**Related:** [Use cases](03-use-cases.md) · [Acceptance criteria](06-acceptance-criteria.md) · [Business rules](requirements/business-rules.md)
