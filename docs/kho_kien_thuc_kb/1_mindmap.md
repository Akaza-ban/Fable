# 01. MINDMAP PHÂN HỆ: KHO KIẾN THỨC — KB (KNOWLEDGE BASE)
> **Hệ thống:** Phân luồng & Xử lý Ticket tự động ứng dụng AI  
> **Phân hệ phụ trách:** Kho kiến thức — KB  
> **Mục tiêu:** Quản trị vòng đời tri thức kỹ thuật, hỗ trợ người dùng tự xử lý sự cố (Self-service), trợ giúp kỹ thuật viên giải quyết nhanh ticket và tự động hóa trích xuất tri thức từ ticket đã xử lý thành công thông qua AI.

---

## 1. Sơ đồ Mindmap hệ thống (Mermaid Diagram)

```mermaid
mindmap
  root((KHO KIẾN THỨC - KB))
    (1) Tạo / sửa bài hướng dẫn xử lý
      Trình soạn thảo linh hoạt
        Soạn thảo Rich Text trực quan
        Hỗ trợ cú pháp Markdown & Code block có highlight
        Chèn ảnh chụp màn hình, video minh họa, đính kèm file log
      Gắn siêu dữ liệu - Metadata
        Danh mục phân cấp (Category đa tầng: Phần mềm, Phần cứng, Mạng, Quyền truy cập...)
        Hệ thống Tags từ khóa và Mã lỗi hệ thống (Error Code)
        Phạm vi hiển thị (Toàn công ty - Public vs Nội bộ IT - Internal Only)
      Quản lý trạng thái soạn thảo
        Tự động lưu bản nháp định kỳ (Auto-save)
        Quản lý phiên bản chỉnh sửa (Draft versioning)
        Xem trước giao diện hiển thị (Preview Mode)
    (2) Duyệt và xuất bản nội dung KB
      Quy trình phê duyệt (Approval Workflow)
        Gửi yêu cầu duyệt kèm ghi chú thay đổi (Change note)
        Thông báo tự động đến người kiểm duyệt (Reviewer notification)
      Công cụ hỗ trợ kiểm duyệt
        So sánh phiên bản trực quan (Diff View giữa bản cũ và mới)
        Kiểm tra độ an toàn thông tin & bảo mật PII
      Hành động phê duyệt
        Phê duyệt và xuất bản ngay (Publish)
        Yêu cầu chỉnh sửa kèm lý do chi tiết (Request Changes)
        Từ chối bài viết (Reject)
      Vòng đời & Lưu trữ
        Quản lý số hiệu phiên bản (v1.0, v1.1, v2.0...)
        Lưu trữ bài hết hiệu lực (Archive / Deprecate)
    (3) Tra cứu giải pháp từ KB đã duyệt
      Công cụ tìm kiếm thông minh
        Tìm kiếm ngữ nghĩa (Semantic Search dựa trên AI Embedding / RAG)
        Tìm kiếm theo từ khóa chính xác, mã lỗi hoặc mã bài viết (KB-ID)
        Tự động gợi ý từ khóa khi nhập (Auto-complete / Suggestion)
      Bộ lọc đa chiều
        Lọc theo Danh mục chuyên môn
        Lọc theo Thiết bị / Hệ điều hành (Windows, macOS, Linux, Mobile)
        Lọc theo Thời gian cập nhật và Độ phổ biến (Helpful rate)
      Trải nghiệm tra cứu tích hợp
        Cổng thông tin tự phục vụ cho End-user (Self-Service Portal)
        Gợi ý bài viết tự động trong form tạo ticket (Ticket Deflection)
        Tích hợp panel tra cứu nhanh trong giao diện Kỹ thuật viên xử lý ticket
    (4) Đề xuất bài viết từ ticket đã resolved
      Quét và phân tích tự động bằng AI
        Theo dõi sự kiện Ticket chuyển sang trạng thái Resolved / Closed
        AI quét lịch sử hội thoại, mô tả lỗi và trao đổi giữa KTV và User
      Làm sạch và bảo vệ dữ liệu nhạy cảm
        Tự động nhận diện và ẩn danh (PII Redaction: Tên, Email, SĐT, IP, Token)
      Trích xuất cấu trúc chuẩn hóa KB
        Đề xuất Tiêu đề bài viết chuẩn SEO/Search
        Tóm tắt hiện tượng sự cố (Problem Description)
        Xác định nguyên nhân gốc rễ (Root Cause)
        Chuẩn hóa các bước giải quyết tuần tự (Step-by-step Solution)
      Quy trình cộng tác Kỹ thuật viên
        Gợi ý KTV: "Tạo bài KB từ ticket này chỉ với 1 click"
        Chuyển nội dung AI trích xuất vào giao diện soạn thảo để KTV tinh chỉnh
    (5) Phản hồi tài liệu sai / không hữu ích
      Đánh giá nhanh (Helpfulness Rating)
        Nút bình chọn hữu ích: Thumbs Up / Thumbs Down
        Thống kê điểm số hài lòng theo từng bài viết
      Báo cáo lỗi chi tiết (Feedback Reporting)
        Lý do: Lỗi thời không còn đúng, Bước làm sai dẫn đến lỗi nặng hơn, Thiếu hình ảnh/ngữ cảnh, Link hỏng
        Kèm bình luận góp ý và ảnh minh chứng
      Quy trình xử lý phản hồi (Feedback Loop)
        Gắn nhãn cảnh báo (Flagged / Needs Review) trên bài viết
        Đẩy thông báo tới Tác giả bài viết hoặc Tech Lead
        Ghi nhận lịch sử chỉnh sửa sau phản hồi
```

---

## 2. Bảng phân rã tính năng chi tiết (Functional Decomposition)

| STT | Nhánh chức năng | Module con | Mô tả nghiệp vụ chi tiết | Đối tượng sử dụng |
| :---: | :--- | :--- | :--- | :--- |
| **1** | **Tạo / Sửa bài hướng dẫn** | 1.1 Trình soạn thảo đa năng | Soạn thảo định dạng giàu (Rich Text), chèn Heading, Code block, Table, đính kèm ảnh chụp màn hình, file log mẫu. | Kỹ thuật viên, Admin |
| | | 1.2 Quản lý Metadata | Phân loại Category phân cấp cha-con, gắn Tags từ khóa và Error code, xác định quyền truy cập (Public / Internal). | Kỹ thuật viên, Admin |
| | | 1.3 Auto-save & Bản nháp | Hệ thống lưu tự động mỗi 15-30 giây, cho phép tạo bản nháp độc lập mà không ảnh hưởng bài đang xuất bản. | Kỹ thuật viên, Admin |
| **2** | **Duyệt và xuất bản KB** | 2.1 Workflow kiểm duyệt | Bài viết từ KTV chuyển sang trạng thái `Pending Review`, kích hoạt thông báo đến Admin / Tech Lead. | Kỹ thuật viên, Admin |
| | | 2.2 Công cụ Diff View | Xem sự khác biệt giữa phiên bản hiện tại và phiên bản chỉnh sửa mới (High-light xanh/đỏ). | Admin, Tech Lead |
| | | 2.3 Phê duyệt / Từ chối | Quyết định xuất bản bài viết lên hệ thống, hoặc từ chối kèm lý do chỉnh sửa cụ thể. | Admin, Tech Lead |
| | | 2.4 Quản lý Version | Lưu vết lịch sử thay đổi (Changelog), cho phép rollback (khôi phục) về phiên bản cũ khi cần. | Admin |
| **3** | **Tra cứu giải pháp KB** | 3.1 AI Semantic Search | Tìm kiếm theo ngôn ngữ tự nhiên (Ví dụ: "quên pass máy in phòng họp"), RAG trích xuất đoạn trả lời trực tiếp. | Toàn bộ User, KTV |
| | | 3.2 Bộ lọc chuyên sâu | Lọc theo danh mục, phạm vi, độ phổ biến, ngày cập nhật mới nhất. | Toàn bộ User, KTV |
| | | 3.3 Tích hợp Self-Service | Gợi ý giải pháp tức thì khi người dùng đang nhập form tạo ticket nhằm giảm tải số lượng ticket tạo ra. | Người gửi ticket |
| | | 3.4 Tích hợp KTV Console | Cho phép KTV tra cứu và nhúng nhanh link bài KB hoặc các bước khắc phục vào nội dung phản hồi ticket. | Kỹ thuật viên |
| **4** | **AI Đề xuất từ Ticket Resolved** | 4.1 Lọc Ticket đủ điều kiện | Quét các ticket được giải quyết thành công và có phản hồi đánh giá tích cực từ người dùng. | Hệ thống AI |
| | | 4.2 Tẩy rửa dữ liệu nhạy cảm | AI tự động lọc bỏ họ tên, email, token, IP riêng tư để bảo vệ bí mật nội bộ trước khi tạo bài. | Hệ thống AI |
| | | 4.3 Trích xuất cấu trúc KB | Tự động điền Form: Title, Category, Problem, Root Cause, Solution Steps. | Hệ thống AI |
| | | 4.4 KTV Review & Tạo Draft | Kỹ thuật viên kiểm tra bản AI nháp, điều chỉnh văn phong và nhấn gửi duyệt chỉ trong 1 phút. | Kỹ thuật viên |
| **5** | **Phản hồi tài liệu sai / lỗi** | 5.1 Đánh giá Helpfulness | Vote Có / Không (Helpful / Unhelpful) để đo lường mức độ tin cậy của bài viết. | Người gửi ticket, KTV |
| | | 5.2 Form báo cáo lỗi | Thu thập lý do chi tiết: Nội dung lỗi thời, hướng dẫn sai kỹ thuật, link tải phần mềm bị hỏng. | Người gửi ticket, KTV |
| | | 5.3 Cảnh báo & Vòng lặp cải tiến | Khi nhận nhiều vote tiêu cực hoặc báo cáo sai, bài viết tự động chuyển cờ `Flagged` và giao việc sửa cho Tác giả. | Admin, Kỹ thuật viên |

---

## 3. Mối liên kết giữa Module KB với các phân hệ khác

```text
[Người gửi ticket]  <---(Tra cứu tự xử lý, Đánh giá bài viết)---+
                                                              |
[Kỹ thuật viên]     <---(Tra cứu giải pháp, Viết bài, Gửi duyệt)-+--->  [KHO KIẾN THỨC (KB)]
                                                              |            |
[Admin / Điều phối] <---(Kiểm duyệt nội dung, Quản lý danh mục)-+            |
                                                                           |
[AI Engine]         <---(Đọc Ticket Resolved -> Sinh bản nháp KB)-----------+
                    <---(Truy vấn KB -> Gợi ý giải pháp cho Ticket mới)----+
```
