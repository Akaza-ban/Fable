# Danh mục tính năng (Feature Catalog)

> Trạng thái thiết kế: TARGET V1
>
> Phạm vi tài liệu hiện tại: chỉ **F05 Workflow & tự động hóa** được đặc tả chi tiết; các tính năng còn lại sẽ bổ sung sau.

Tên hệ thống: **Hệ thống phân luồng & xử lý ticket IT tự động ứng dụng AI**.

| ID | Tên | V1 | Trạng thái đặc tả |
|---|---|---|---|
| F01 | Đăng nhập & phân quyền | Có | Chưa đặc tả |
| F02 | Dashboard | Có | Chưa đặc tả |
| F03 | Quản lý ticket | Có | Chưa đặc tả |
| F04 | Xử lý ticket bằng AI | Có | Chưa đặc tả |
| **F05** | **Workflow & tự động hóa** | Có | **Đã đặc tả (tài liệu này)** |
| F06 | Kho kiến thức — KB | Có | Chưa đặc tả |
| F07 | Báo cáo & phân tích | Có | Chưa đặc tả |
| F08 | Nhật ký hoạt động | Có | Chưa đặc tả |
| F09 | Quản lý người dùng | Có | Chưa đặc tả |
| F10 | Hồ sơ cá nhân | Có | Chưa đặc tả |

## F05 Workflow & tự động hóa

**Bối cảnh:** doanh nghiệp nhận khoảng **8.000 ticket/ngày** từ các phòng ban. Phần lớn là yêu cầu cơ bản, lặp lại (kiểm tra tài khoản, cấp tài khoản, cấp thêm quyền) — nếu kỹ thuật viên xử lý tay sẽ quá tải.

**Mục tiêu V1:** AI Service tự duyệt và hệ thống tự thực thi các yêu cầu cơ bản **trong phạm vi được cấu hình**; các yêu cầu cần con người (phần cứng, ngoài phạm vi) được chuyển cho kỹ thuật viên; khi AI không chắc chắn thì hỏi lại người gửi.

| Loại yêu cầu | Ví dụ | Ai xử lý |
|---|---|---|
| Cơ bản, trong phạm vi AI | Kiểm tra tài khoản, cấp tài khoản, cấp thêm quyền | **AI duyệt → hệ thống gọi API thực thi** |
| Cần con người | Cấp chuột, bàn phím, sửa máy, lỗi phức tạp, quyền ngoài danh mục | **Kỹ thuật viên** |
| AI chưa chắc chắn | Ticket mô tả mơ hồ, thiếu thông tin | **AI hỏi lại người gửi**, rồi phân luồng lại |

- F05.1 Cấu hình quy tắc phân công kỹ thuật viên (điều kiện, nhóm đích, chiến lược chia việc, thứ tự, chạy thử)
- F05.2 Cấu hình chính sách SLA theo mức ưu tiên và lịch làm việc
- F05.3 Cấu hình **phạm vi xử lý của AI**: hành động AI và ngưỡng tin cậy, danh mục hành động tự động (whitelist), danh mục quyền theo phòng ban
- F05.4 **Phân luồng ticket**: AI tự xử lý / hỏi lại người gửi / chuyển con người
- F05.5 **Tự động phân công kỹ thuật viên theo quy tắc** + hàng chờ thủ công
- F05.6 Nhắc phản hồi, cảnh báo và chuyển cấp khi vi phạm SLA
- F05.7 **AI duyệt yêu cầu trong phạm vi** (AI hiểu yêu cầu + kiểm tra chính sách cố định)
- F05.8 **Gọi API thực thi sau khi được duyệt** (idempotent, có thử lại, thu hồi quyền khi hết hạn)
- F05.9 Xác minh kết quả và chuyển kỹ thuật viên khi lỗi

**Ngoài phạm vi F05 (thuộc tính năng khác):**

- Mô hình phân loại, trích xuất thông tin, tóm tắt của AI → F04 (F05 chỉ *cấu hình* và *sử dụng* kết quả).
- Tra cứu lịch sử chạy workflow và lỗi → F08 Nhật ký hoạt động (F05 chỉ *sinh* dữ liệu).
- Thống kê tỷ lệ AI tự giải quyết, vi phạm SLA → F07 Báo cáo & phân tích.
- Quản lý nhóm kỹ thuật, phòng ban, thành viên → F09 Quản lý người dùng.

**Related:** [Use cases](03-use-cases.md) · [INVEST backlog](08-invest-requirements.md) · [Acceptance criteria](06-acceptance-criteria.md)
