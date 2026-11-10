# 02. DANH SÁCH USER STORIES & ACCEPTANCE CRITERIA
> **Phân hệ:** Kho kiến thức — KB (Knowledge Base)  
> **Phương pháp luận:** Agile / Scrum Framework  
> **Cấu trúc User Story:** `As a [Role], I want to [Action], So that [Benefit]`  
> **Tiêu chí chấp nhận (AC):** Chuẩn Gherkin (`Given - When - Then`)

---

## 1. Chân dung người dùng (User Personas)

| Persona | Vai trò | Đặc điểm & Nhu cầu chính |
| :--- | :--- | :--- |
| **Trần Văn Minh** (30 tuổi) | **End-User (Người gửi ticket)** | Nhân viên văn phòng, không rành kỹ thuật sâu. Muốn tìm nhanh cách giải quyết lỗi (mạng, máy in, phần mềm) để tiếp tục công việc ngay mà không phải chờ IT hỗ trợ. |
| **Nguyễn Văn Tuấn** (27 tuổi) | **Kỹ thuật viên (IT Support)** | Thường xuyên xử lý các sự cố lặp lại. Cần tra cứu nhanh quy trình chuẩn và muốn lưu lại kinh nghiệm xử lý ca khó thành bài KB một cách tiện lợi, không mất công gõ lại từ đầu. |
| **Lê Thị Lan** (35 tuổi) | **Admin / IT Support Lead** | Phụ trách chất lượng hỗ trợ và kho tri thức. Cần kiểm soát tính chính xác, an toàn thông tin (bảo mật, không lộ tài khoản/mật khẩu), quản lý chất lượng và độ cập nhật của các bài viết KB. |
| **AI Knowledge Engine** | **Trợ lý AI hệ thống** | Tự động đọc dữ liệu các ticket đã giải quyết (Resolved), phân tích ngữ nghĩa, làm sạch thông tin nhạy cảm và sinh bản thảo bài viết cho KTV. |

---

## 2. Bảng tổng hợp User Stories (Ước lượng & Độ ưu tiên)

| Mã Story | Tên User Story | Vai trò | Mức ưu tiên (MoSCoW) | Story Points |
| :--- | :--- | :--- | :---: | :---: |
| **US-KB-01** | Tra cứu giải pháp sự cố thông minh | End-User / KTV | **Must Have** | 5 |
| **US-KB-02** | Soạn thảo bài hướng dẫn xử lý mới | Kỹ thuật viên | **Must Have** | 5 |
| **US-KB-03** | Chỉnh sửa bài viết và quản lý phiên bản | KTV / Admin | **Must Have** | 5 |
| **US-KB-04** | AI tự động tạo bài KB từ Ticket Resolved | KTV / AI Engine | **Must Have** | 8 |
| **US-KB-05** | Phê duyệt và xuất bản bài viết | Admin / Lead | **Must Have** | 5 |
| **US-KB-06** | Đánh giá hữu ích và báo cáo bài viết sai lệch | End-User / KTV | **Should Have** | 3 |
| **US-KB-07** | Quản lý và xử lý bài viết bị gắn cờ báo lỗi | Admin / KTV | **Should Have** | 5 |
| **US-KB-08** | Tự động gợi ý KB khi người dùng nhập Ticket | End-User | **Could Have** | 5 |

---

## 3. Đặc tả chi tiết từng User Story

### US-KB-01: Tra cứu giải pháp sự cố thông minh
*   **User Story:**
    > *Là một* **Người dùng hoặc Kỹ thuật viên**,  
    > *Tôi muốn* tìm kiếm tài liệu giải pháp bằng từ khóa tự nhiên, mã lỗi hoặc câu hỏi thông thường,  
    > *Để* nhanh chóng tự khắc phục sự cố mà không tốn thời gian chờ đợi.
*   **Acceptance Criteria:**
    *   **AC1.1 (Semantic & Keyword Search):**  
        *Given* người dùng ở trang chủ Kho kiến thức,  
        *When* người dùng nhập cụm từ tìm kiếm (ví dụ: *"lỗi không in được qua mạng wifi"* hoặc mã lỗi *"0x0000011b"*),  
        *Then* hệ thống trả về danh sách các bài viết có liên quan theo độ chính xác giảm dần, tô sáng (highlight) đoạn trích trực tiếp chứa câu trả lời.
    *   **AC1.2 (Lọc theo phân quyền người xem):**  
        *Given* người dùng là End-User thông thường,  
        *When* thực hiện tra cứu,  
        *Then* hệ thống chỉ hiển thị các bài viết có phạm vi `Public` (Công khai), ẩn hoàn toàn các bài viết đánh dấu `Internal Only` (Nội bộ IT).
    *   **AC1.3 (Bộ lọc đa thuộc tính):**  
        *Given* kết quả tìm kiếm hiển thị,  
        *When* người dùng chọn bộ lọc theo Chuyên mục (Phần mềm, Mạng, Phần cứng) hoặc Hệ điều hành (Windows, macOS),  
        *Then* danh sách kết quả được cập nhật tương ứng tức thì.
    *   **AC1.4 (Xử lý khi không có kết quả):**  
        *Given* từ khóa tìm kiếm không khớp với bài viết nào,  
        *When* hoàn tất truy vấn,  
        *Then* hệ thống hiển thị thông báo thân thiện: *"Chưa có bài viết phù hợp. Bạn có muốn [Tạo ticket nhờ hỗ trợ kỹ thuật] không?"*.

---

### US-KB-02: Soạn thảo bài hướng dẫn xử lý mới
*   **User Story:**
    > *Là một* **Kỹ thuật viên**,  
    > *Tôi muốn* sử dụng công cụ soạn thảo trực quan để viết bài hướng dẫn xử lý sự cố kèm hình ảnh và thẻ phân loại,  
    > *Để* chuẩn hóa giải pháp kỹ thuật cho toàn đội ngũ và người dùng.
*   **Acceptance Criteria:**
    *   **AC2.1 (Trình soạn thảo phong phú):**  
        *Given* KTV ở màn hình "Tạo bài viết mới",  
        *When* soạn nội dung,  
        *Then* trình soạn thảo hỗ trợ đầy đủ các định dạng: Tiêu đề các cấp (H1, H2, H3), In đậm, Nghiêng, Danh sách số/chấm, Chèn bảng, Chèn ảnh (kéo thả hoặc dán từ clipboard), và Khối mã nguồn (Code block).
    *   **AC2.2 (Ràng buộc trường bắt buộc):**  
        *Given* KTV nhấn nút "Gửi phê duyệt",  
        *When* thiếu một trong các trường: Tiêu đề bài viết, Danh mục (Category), Phạm vi hiển thị (Public/Internal) hoặc Nội dung giải pháp,  
        *Then* hệ thống hiển thị thông báo lỗi tại trường tương ứng và chặn gửi yêu cầu.
    *   **AC2.3 (Cơ chế tự động lưu - Auto-save):**  
        *Given* KTV đang soạn thảo bài viết,  
        *When* sau mỗi 20 giây có thay đổi nội dung,  
        *Then* hệ thống tự động lưu vào bản nháp (`Draft`) và hiển thị thông báo nhỏ *"Đã lưu nháp lúc hh:mm:ss"*.

---

### US-KB-03: Chỉnh sửa bài viết và quản lý phiên bản (Version Control)
*   **User Story:**
    > *Là một* **Kỹ thuật viên hoặc Admin**,  
    > *Tôi muốn* cập nhật nội dung của một bài viết KB đã xuất bản mà không làm gián đoạn bài viết hiện tại,  
    > *Để* nội dung luôn bám sát các thay đổi về công nghệ và hệ thống.
*   **Acceptance Criteria:**
    *   **AC3.1 (Tạo bản nháp chỉnh sửa độc lập):**  
        *Given* bài viết đang ở trạng thái `Published` (v1.0),  
        *When* KTV bấm nút "Chỉnh sửa bài viết",  
        *Then* hệ thống tạo một phiên bản nháp mới (v1.1 Draft) để KTV sửa đổi; bài viết gốc v1.0 vẫn hiển thị bình thường cho người đọc tra cứu.
    *   **AC3.2 (Ghi chú thay đổi bắt buộc):**  
        *Given* KTV hoàn tất chỉnh sửa trên phiên bản mới,  
        *When* bấm "Gửi phê duyệt bản cập nhật",  
        *Then* hệ thống yêu cầu nhập trường "Lý do / Tóm tắt thay đổi" (Changelog summary).
    *   **AC3.3 (Lịch sử phiên bản):**  
        *Given* người xem có quyền quản trị,  
        *When* mở tab "Lịch sử phiên bản",  
        *Then* hệ thống liệt kê danh sách toàn bộ các version trước đó, ngày cập nhật, người sửa và người duyệt.

---

### US-KB-04: AI tự động tạo bài KB từ Ticket đã Resolved
*   **User Story:**
    > *Là một* **Kỹ thuật viên**,  
    > *Tôi muốn* hệ thống AI tự động phân tích ticket vừa giải quyết xong và sinh ra khung bài viết KB hoàn chỉnh,  
    > *Để* nhanh chóng đóng góp tài liệu vào kho tri thức mà không tốn công biên soạn lại từ đầu.
*   **Acceptance Criteria:**
    *   **AC4.1 (Kích hoạt tạo bài bằng AI):**  
        *Given* một ticket đang ở trạng thái `Resolved` hoặc `Closed`,  
        *When* KTV bấm nút "Tạo bài KB với AI",  
        *Then* hệ thống gọi dịch vụ AI trích xuất và hiển thị Modal bản nháp bài viết chỉ trong vòng dưới 5 giây.
    *   **AC4.2 (Cấu trúc thông tin AI sinh ra):**  
        *Given* AI phân tích toàn bộ nội dung trao đổi và giải pháp trong ticket,  
        *When* tạo bản nháp,  
        *Then* bản nháp phải có đầy đủ các mục chuẩn hóa:
        - Tiêu đề khái quát (Loại bỏ các thông tin vụn vặt).
        - Mô tả sự cố (Hiện tượng người dùng gặp phải).
        - Nguyên nhân gốc rễ (Root Cause).
        - Các bước khắc phục tuần tự (Step-by-step resolution steps).
        - Gợi ý thẻ Tags và Danh mục phù hợp.
    *   **AC4.3 (Làm sạch thông tin nhạy cảm - PII Redaction):**  
        *Given* nội dung ticket gốc có chứa email cá nhân, mật khẩu, IP máy trạm hoặc tên người gửi,  
        *When* AI sinh nội dung bài viết,  
        *Then* các thông tin nhạy cảm đó phải được tự động làm mờ hoặc thay thế bằng biến mẫu (ví dụ: `user@company.com`, `<IP_Address>`).
    *   **AC4.4 (KTV tinh chỉnh và chuyển giao):**  
        *Given* bản nháp AI hiển thị,  
        *When* KTV bấm "Áp dụng vào Form soạn thảo",  
        *Then* dữ liệu được đưa trực tiếp vào màn hình soạn thảo để KTV đọc lại, sửa đổi và bấm "Gửi phê duyệt".

---

### US-KB-05: Phê duyệt và xuất bản bài viết
*   **User Story:**
    > *Là một* **Admin hoặc Tech Lead**,  
    > *Tôi muốn* kiểm duyệt các bài viết KB do KTV gửi lên bằng công cụ so sánh trực quan,  
    > *Để* đảm bảo thông tin phát hành ra kho kiến thức là chuẩn xác, an toàn và dễ hiểu.
*   **Acceptance Criteria:**
    *   **AC5.1 (Danh sách hàng đợi phê duyệt):**  
        *Given* Admin truy cập màn hình Quản trị KB,  
        *When* chọn tab "Chờ phê duyệt",  
        *Then* hệ thống hiển thị danh sách các bài viết có trạng thái `Pending Review` kèm tên tác giả, ngày gửi và loại yêu cầu (Tạo mới hoặc Cập nhật).
    *   **AC5.2 (So sánh trực quan - Diff View):**  
        *Given* bài viết gửi duyệt là bản cập nhật cho bài đã có,  
        *When* Admin xem chi tiết,  
        *Then* hệ thống cung cấp chế độ "Diff View" hiển thị nổi bật phần chữ được thêm mới (màu xanh lá) và phần bị xóa (màu đỏ gạch).
    *   **AC5.3 (Phê duyệt thành công):**  
        *Given* bài viết đạt yêu cầu,  
        *When* Admin bấm nút "Phê duyệt & Xuất bản",  
        *Then* trạng thái bài viết chuyển thành `Published`, xuất hiện ngay lập tức trên cổng tra cứu và gửi thông báo chúc mừng tới tác giả.
    *   **AC5.4 (Yêu cầu chỉnh sửa / Từ chối):**  
        *Given* bài viết chưa đạt chuẩn,  
        *When* Admin chọn "Yêu cầu chỉnh sửa" hoặc "Từ chối",  
        *Then* hệ thống bắt buộc nhập lý do vào ô phản hồi và gửi trả bài viết về trạng thái `Changes Requested` hoặc `Rejected` cho tác giả.

---

### US-KB-06: Đánh giá hữu ích và báo cáo tài liệu sai lệch / lỗi thời
*   **User Story:**
    > *Là một* **Người đọc (End-User hoặc KTV)**,  
    > *Tôi muốn* bình chọn tính hữu ích và báo cáo nếu nội dung bài viết bị sai hoặc không còn dùng được,  
    > *Để* giúp đội ngũ IT kịp thời cập nhật và duy trì chất lượng kho tri thức.
*   **Acceptance Criteria:**
    *   **AC6.1 (Vote nhanh Có/Không):**  
        *Given* người dùng đang đọc bài viết KB,  
        *When* người dùng bấm nút 👍 "Hữu ích",  
        *Then* hệ thống tăng số lượng vote hữu ích lên 1 đơn vị và vô hiệu hóa nút bấm để tránh spam.
    *   **AC6.2 (Form báo cáo chi tiết):**  
        *Given* người dùng bấm nút 👎 "Không hữu ích",  
        *When* hộp thoại phản hồi mở ra,  
        *Then* người dùng có thể chọn lý do: *"Nội dung lỗi thời"*, *"Các bước hướng dẫn sai/gây lỗi"*, *"Thiếu thông tin/ngữ cảnh"*, *"Khác"* và nhập ý kiến đóng góp.
    *   **AC6.3 (Tự động gắn cờ cảnh báo):**  
        *Given* một bài viết nhận được từ 3 báo cáo lỗi trở lên trong vòng 30 ngày,  
        *When* người dùng tiếp theo xem bài,  
        *Then* hệ thống hiển thị banner cảnh báo: *"⚠️ Bài viết này đang được xem xét cập nhật do có phản hồi chưa chính xác"*, đồng thời gửi thông báo tới Tác giả bài viết.

---

### US-KB-07: Quản lý và xử lý bài viết bị gắn cờ báo lỗi
*   **User Story:**
    > *Là một* **Admin hoặc Tác giả bài viết**,  
    > *Tôi muốn* theo dõi danh sách các bài viết bị người dùng báo cáo sai hoặc đánh giá thấp,  
    > *Để* có kế hoạch chỉnh sửa hoặc tạm ẩn tài liệu gây hiểu lầm.
*   **Acceptance Criteria:**
    *   **AC7.1 (Bảng điều khiển bài viết gắn cờ):**  
        *Given* Admin vào tab "Bài viết cần chú ý (Flagged Articles)",  
        *When* xem danh sách,  
        *Then* hệ thống hiển thị bài viết kèm tỷ lệ hữu ích (Helpful Rate %), số lượng báo cáo lỗi và trích đoạn ý kiến của người dùng.
    *   **AC7.2 (Hành động xử lý nhanh):**  
        *Given* Admin xem xét phản hồi của một bài viết bị gắn cờ,  
        *When* quyết định xử lý,  
        *Then* Admin có các tùy chọn:
        - "Giao việc sửa chữa" (Assign cập nhật cho KTV tác giả).
        - "Tạm ẩn bài viết" (Chuyển sang `Archived`/`Hidden` để tránh người dùng làm theo sai).
        - "Bỏ qua báo cáo" (Nếu báo cáo không hợp lệ).
