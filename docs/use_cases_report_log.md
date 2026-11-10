# Use cases — Báo cáo & phân tích, Nhật ký hoạt động

Vai trò, phạm vi xem ticket, ma trận quyền và quy tắc `BR-xx` dùng lại từ [mvp.md](mvp.md). File này chỉ thêm `UC-BC-xx`, `UC-NK-xx`, `BR-BC-xx`, `BR-NK-xx` và **đề xuất** quyền mới (mục 6). Các quyền mới chưa có trong `mvp.md` nên chưa có hiệu lực cho tới khi chủ file `mvp.md` chấp nhận.

Ghi chú về phụ thuộc: trạng thái ticket, mức ưu tiên, ghi chú nội bộ, yêu cầu phê duyệt, lần chạy workflow, gợi ý KB là dữ liệu của các module Ticket, Xử lý ticket bằng AI, Workflow và Kho kiến thức. Hai module này chỉ **đọc** dữ liệu đó, không định nghĩa lại.

## 1. Tổng quan

| Mã | Use case | Actor |
|---|---|---|
| UC-BC-01 | Thống kê ticket theo loại và nhóm | ADMIN; TECHNICIAN (trong phạm vi) |
| UC-BC-02 | Theo dõi thời gian xử lý và vi phạm SLA | ADMIN; TECHNICIAN (trong phạm vi) |
| UC-BC-03 | Đánh giá tỷ lệ AI tự giải quyết | ADMIN |
| UC-BC-04 | Tổng hợp mức hài lòng của người gửi | ADMIN; TECHNICIAN (trong phạm vi) |
| UC-BC-05 | Lọc và xuất báo cáo | ADMIN |
| UC-BC-06 | Cấu hình tham số chỉ số | ADMIN |
| UC-NK-01 | Xem lịch sử thay đổi ticket | ADMIN; TECHNICIAN, REQUESTER (trong phạm vi) |
| UC-NK-02 | Theo dõi thao tác của người dùng và AI | ADMIN |
| UC-NK-03 | Xem yêu cầu phê duyệt và kết quả | ADMIN; TECHNICIAN, REQUESTER (trong phạm vi) |
| UC-NK-04 | Tra cứu lỗi và lịch sử chạy workflow | ADMIN; TECHNICIAN (tóm tắt, trong phạm vi) |
| UC-NK-05 | Ghi nhật ký (chỉ ghi thêm, che dữ liệu nhạy cảm) | Hệ thống |

---

## 2. Định nghĩa chỉ số

Báo cáo, Dashboard và demo đều dùng **cùng một cách tính** dưới đây (BR-BC-01). Tham số ghi "Admin cấu hình" được chỉnh ở UC-BC-06. Giá trị mặc định là **đề xuất cho demo**, chưa phải cam kết SLA.

| Mã | Chỉ số | Cách tính | Tham số |
|---|---|---|---|
| M-01 | Thời gian phản hồi đầu | Từ lúc tạo ticket đến **phản hồi công khai đầu tiên** cho người gửi, do TECHNICIAN, ADMIN hoặc AI_AGENT gửi. Không tính thông báo tự động "đã nhận ticket" (`actor_type = SYSTEM`). Báo cáo tách riêng phản hồi đầu của người và của AI. | — |
| M-02 | Thời gian xử lý | Từ lúc tạo đến lần chuyển sang trạng thái đã giải quyết **cuối cùng**, **trừ** tổng thời gian ticket ở trạng thái chờ người gửi bổ sung, và trừ thời gian ở trạng thái đã giải quyết trước khi bị mở lại. | Tính theo giờ lịch hay giờ làm việc: Admin cấu hình (demo: giờ lịch, múi giờ `Asia/Ho_Chi_Minh`) |
| M-03 | Vi phạm SLA | Ticket vi phạm nếu M-01 > mục tiêu phản hồi **hoặc** M-02 > mục tiêu xử lý của mức ưu tiên của ticket. Ticket đang mở mà đã vượt mục tiêu cũng tính là vi phạm. Tỷ lệ vi phạm = số ticket vi phạm / số ticket có áp SLA trong kỳ. Hai loại vi phạm (phản hồi, xử lý) được đếm riêng. | Mục tiêu theo mức ưu tiên: Admin cấu hình. Demo (nếu module Ticket dùng 4 mức Khẩn / Cao / Trung bình / Thấp): phản hồi 1h / 4h / 8h / 24h; xử lý 4h / 24h / 72h / 120h |
| M-04 | Tỷ lệ AI tự giải quyết | Tử số: số ticket đã đóng trong kỳ mà **(a)** không có thao tác xử lý nào của TECHNICIAN hoặc ADMIN (trả lời, đổi trạng thái, giao việc, sửa phân loại, ghi chú) **và (b)** không bị mở lại trong N ngày sau khi đóng. Mẫu số: số ticket đã đóng trong kỳ. Ticket đóng chưa đủ N ngày bị **loại khỏi cả tử số lẫn mẫu số**, hiển thị riêng là "đang theo dõi". Có thêm tỷ lệ phụ: tử số / số ticket AI đã nhận xử lý. | N: Admin cấu hình (demo: 7 ngày) |
| M-05 | Tỷ lệ phân loại AI bị sửa | Số ticket có loại hoặc nhóm do AI gán (`actor_type = AI`) mà **sau đó** bị người (`actor_type = USER`) đổi loại hoặc chuyển nhóm / số ticket có loại do AI gán. Xác định bằng audit log (BR-09). | — |
| M-06 | Tỷ lệ gợi ý KB hữu ích | Số gợi ý KB được đánh dấu "hữu ích" / số gợi ý KB **có phản hồi**. Có thêm tỷ lệ phản hồi = số gợi ý có phản hồi / số gợi ý đã hiển thị. | Ai được đánh dấu: xem câu hỏi mở |
| M-07 | Mức hài lòng | Điểm trung bình trên thang 1–5 của các ticket đã được chấm, kèm phân bố số ticket theo từng điểm từ 1 đến 5. Tỷ lệ người chấm = số ticket đã chấm / số ticket đã đóng trong kỳ. Luôn hiện tỷ lệ người chấm cạnh điểm trung bình. | Thời hạn chấm sau khi đóng: Admin cấu hình (demo: 7 ngày) |

**Quy tắc chung khi tính:**
- Thống kê **số lượng** gán ticket vào kỳ theo **ngày tạo**. Chỉ số **thời gian, SLA, M-04, M-07** gán ticket vào kỳ theo **ngày đóng hoặc ngày giải quyết**.
- Ticket chuyển qua nhiều nhóm được tính cho **nhóm cuối cùng** xử lý (xem câu hỏi mở Q-07).
- Mọi chỉ số chỉ tính trên tập ticket **trong phạm vi** của người xem (BR-BC-02).

---

## 3. Báo cáo & phân tích

### UC-BC-01 Thống kê ticket theo loại và nhóm
- **Luồng chính:** chọn khoảng thời gian (mặc định 30 ngày gần nhất) → backend lấy tập ticket trong phạm vi người xem → đếm số ticket mới, đang mở, đã đóng, bị mở lại, nhóm theo loại ticket và nhóm kỹ thuật → trả bảng, kèm xu hướng theo ngày / tuần / tháng.
- **Nhánh:** TECHNICIAN chỉ thấy số liệu của ticket giao cho mình và ticket của nhóm mình. Bộ lọc nhóm chỉ hiện những nhóm người đó đang thuộc. Người đó vừa đổi nhóm → phạm vi mới có hiệu lực từ request kế tiếp (BR-07).
- **Ngoại lệ:** không có `report:read` → `403`. Lọc theo nhóm ngoài phạm vi → `403`. Khoảng thời gian sai (từ > đến, hoặc dài hơn mức tối đa) → `422` `INVALID_DATE_RANGE`.
- **BR:** BR-07, BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-02 Theo dõi thời gian xử lý và vi phạm SLA
- **Luồng chính:** chọn kỳ và bộ lọc → hiển thị M-01, M-02 (trung bình, trung vị, P90) và M-03 theo mức ưu tiên, loại ticket, nhóm → danh sách ticket vi phạm (mã, loại, nhóm, ưu tiên, loại vi phạm, vượt bao lâu) → bấm vào ticket để mở chi tiết, kiểm tra theo `ticket:read`.
- **Nhánh:** xem danh sách ticket **sắp vi phạm** (đang mở, đã dùng ≥ X% thời gian mục tiêu; X do Admin cấu hình, demo 80%). Ticket chưa có mức ưu tiên → không áp SLA, đếm riêng là "chưa áp SLA".
- **Ngoại lệ:** chưa cấu hình mục tiêu SLA → vẫn hiện M-01 và M-02; M-03 hiện "chưa cấu hình", không trả 0%. Không có quyền hoặc ngoài phạm vi → `403`.
- **BR:** BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-03 Đánh giá tỷ lệ AI tự giải quyết
- **Luồng chính:** chọn kỳ → hiển thị M-04 (kèm số ticket "đang theo dõi"), M-05, M-06 theo loại ticket → danh sách ticket AI đã đóng nhưng bị mở lại, và danh sách ticket có phân loại AI bị sửa (giá trị AI gán → giá trị người sửa, lấy từ `before` / `after` của audit log).
- **Nhánh:** lọc theo service account AI (khi có nhiều service account). So sánh giữa các kỳ để thấy AI có tốt dần lên sau những đề xuất được duyệt không (BR-15).
- **Ngoại lệ:** TECHNICIAN, REQUESTER, AI_AGENT → `403` (đề xuất: chỉ ADMIN xem, xem Q-03). Kỳ chưa có ticket nào đóng đủ N ngày → hiện "chưa đủ dữ liệu", không trả 0%.
- **BR:** BR-09, BR-15, BR-BC-01, BR-BC-03.

### UC-BC-04 Tổng hợp mức hài lòng của người gửi
- **Luồng chính:** chọn kỳ → hiển thị M-07 (điểm trung bình, phân bố từ 1 đến 5, tỷ lệ người chấm) theo loại ticket, nhóm, và theo người xử lý (AI hay người) → danh sách ticket bị chấm 1–2 điểm.
- **Nhánh:** TECHNICIAN xem điểm của ticket trong phạm vi mình. Không xem được bảng xếp hạng từng kỹ thuật viên khác (xem Q-02). Bình luận kèm điểm chỉ ADMIN xem.
- **Ngoại lệ:** tỷ lệ người chấm dưới ngưỡng (Admin cấu hình, demo 20%) → vẫn hiện điểm nhưng gắn cảnh báo "ít người chấm, chưa đại diện". Không có quyền → `403`.
- **BR:** BR-BC-01, BR-BC-02, BR-BC-03.

### UC-BC-05 Lọc và xuất báo cáo
- **Luồng chính:** chọn loại báo cáo (UC-BC-01…04), bộ lọc (kỳ, loại ticket, nhóm, mức ưu tiên, người xử lý AI hay người, trạng thái SLA) → xem trước → bấm Xuất, chọn định dạng CSV hoặc XLSX → hệ thống **ghi audit log `REPORT_EXPORTED`** → tạo file chỉ gồm các cột trong danh sách cho phép → trả file.
- **Nhánh:** số dòng vượt giới hạn (Admin cấu hình, demo 50.000) → `422` `EXPORT_TOO_LARGE`, yêu cầu thu hẹp bộ lọc. Bộ lọc được lưu trong URL để mở lại cùng báo cáo.
- **Ngoại lệ:** không có `report:export` → `403`. Ghi audit log thất bại → **không trả file**, lỗi `503` `AUDIT_UNAVAILABLE` (BR-BC-05).
- **BR:** BR-09, BR-BC-02, BR-BC-04, BR-BC-05.

### UC-BC-06 Cấu hình tham số chỉ số
- **Luồng chính:** Admin sửa các tham số: mục tiêu SLA theo mức ưu tiên, giờ lịch hay giờ làm việc, N ngày của M-04, ngưỡng "sắp vi phạm", thời hạn chấm điểm, ngưỡng tỷ lệ người chấm, giới hạn số dòng xuất, khoảng thời gian tối đa → lưu → ghi audit log `METRIC_CONFIG_UPDATED` kèm giá trị trước và sau.
- **Nhánh:** đổi tham số chỉ ảnh hưởng **cách tính khi xem**; báo cáo đã xuất trước đó không đổi. Mỗi báo cáo hiển thị tham số đang dùng (ví dụ "N = 7 ngày").
- **Ngoại lệ:** không phải ADMIN → `403`. Giá trị không hợp lệ (âm, N = 0, mục tiêu phản hồi > mục tiêu xử lý) → `422`.
- **BR:** BR-09, BR-BC-01.

---

## 4. Nhật ký hoạt động

Nhật ký dùng đúng audit log của **BR-09**: ai (`actor_type` = `USER` / `AI` / `SYSTEM`), làm gì (`action`), lúc nào, trên đối tượng nào, **giá trị trước và sau**, kèm `suggestion_id` khi có (BR-15). Hai module này **mở rộng phạm vi ghi** của BR-09 sang ticket, phê duyệt và workflow (BR-NK-01), không đổi cấu trúc bản ghi.

### UC-NK-01 Xem lịch sử thay đổi ticket
- **Luồng chính:** mở ticket (kiểm tra `ticket:read`) → tab Lịch sử → danh sách sự kiện theo thời gian, mới nhất ở trên: tạo ticket, AI phân loại hoặc phân luồng, đổi loại, nhóm, người xử lý, ưu tiên, trạng thái, phản hồi, ghi chú nội bộ, yêu cầu phê duyệt, chạy workflow, đóng, mở lại, chấm điểm → mỗi dòng có người thực hiện (tên người, "AI", hoặc "Hệ thống"), thời điểm, giá trị trước và sau.
- **Nhánh — REQUESTER:** chỉ thấy các sự kiện thuộc **danh sách cho phép** (BR-NK-04): tạo, đổi trạng thái, phản hồi công khai, yêu cầu bổ sung thông tin, kết quả phê duyệt (dạng tóm tắt), đóng, mở lại, chấm điểm. **Không** thấy ghi chú nội bộ, chi tiết workflow, độ tin cậy của AI, chuyển nhóm nội bộ. Người thực hiện là AI hiện "Trợ lý AI".
- **Nhánh — TECHNICIAN:** thấy đầy đủ với ticket trong phạm vi; chi tiết workflow chỉ ở dạng tóm tắt (UC-NK-04).
- **Ngoại lệ:** ticket ngoài phạm vi → `403` (giống `ticket:read`, không trả `404` để lộ ticket có tồn tại hay không). Gọi API sửa / xóa bản ghi lịch sử → không có endpoint, trả `405` (BR-NK-02).
- **BR:** BR-09, BR-NK-01, BR-NK-02, BR-NK-04.

### UC-NK-02 Theo dõi thao tác của người dùng và AI
- **Luồng chính:** Admin mở trang Nhật ký → lọc theo `actor_type`, người thực hiện (user hoặc service account), `action`, loại đối tượng, mã đối tượng, `suggestion_id`, khoảng thời gian → bảng có phân trang → mở một dòng để xem `before` / `after` dạng so sánh.
- **Nhánh:** từ trang người dùng hoặc service account bấm "Xem nhật ký" → mở sẵn bộ lọc theo người đó. Lọc `actor_type = AI` để rà soát mọi việc AI đã làm. Lọc `actor_type = SYSTEM` để xem thay đổi do đồng bộ Microsoft (BR-18, BR-20). User đã bị tắt hoặc gỡ vẫn hiện đúng tên trong nhật ký (BR-20 không xóa bản ghi `User`).
- **Ngoại lệ:** không có `audit:read` → `403`. Khoảng thời gian dài hơn mức tối đa → `422` `INVALID_DATE_RANGE`.
- **BR:** BR-09, BR-15, BR-18, BR-20, BR-NK-02, BR-NK-03.

### UC-NK-03 Xem yêu cầu phê duyệt và kết quả
- **Luồng chính:** mở yêu cầu phê duyệt (từ ticket hoặc từ danh sách) → hiển thị chuỗi bước theo thời gian, mỗi bước có actor, thời điểm và kết quả:
  1. **Yêu cầu:** ai yêu cầu, cho ai, quyền gì, trên hệ thống nào, lý do, ticket gốc.
  2. **Kiểm tra chính sách:** kết quả *đạt / không đạt / cần người duyệt*, kèm tên chính sách được áp dụng.
  3. **Quyết định:** *duyệt / từ chối*, do người duyệt (`actor_type = USER`, kèm lý do) hoặc **"duyệt theo chính sách"** (`actor_type = SYSTEM`, kèm tên chính sách). AI không bao giờ là người quyết định (mvp.md mục 2: AI không được tự cấp quyền).
  4. **Gọi API thực thi:** thời điểm, hệ thống đích, kết quả (mã phản hồi đã che thông tin nhạy cảm).
  5. **Xác minh kết quả:** hệ thống kiểm tra lại quyền đã thực sự được cấp chưa → *đã xác minh / xác minh thất bại*.
- **Nhánh — đã duyệt nhưng thực thi lỗi:** bước 4 lỗi hoặc bước 5 thất bại → yêu cầu ở trạng thái "Đã duyệt — thực thi lỗi", hiển thị nổi bật, ticket chuyển người xử lý tay (như UC-NK-04). Lần thử lại được thêm thành **bước mới**; bước cũ không bị ghi đè.
- **Nhánh — từ chối hoặc không đạt chính sách:** chuỗi dừng ở bước 2 hoặc 3, hiện lý do.
- **Nhánh — REQUESTER:** chỉ thấy yêu cầu thuộc ticket của mình, dạng tóm tắt (đang chờ / đã duyệt / bị từ chối / đã cấp xong / đang xử lý lỗi). Không thấy tên chính sách, phản hồi API, tên người duyệt (xem Q-09).
- **Ngoại lệ:** yêu cầu thuộc ticket ngoài phạm vi → `403`. Không có `approval:read` → `403`.
- **BR:** BR-09, BR-NK-01, BR-NK-03, BR-NK-04, BR-NK-06.

### UC-NK-04 Tra cứu lỗi và lịch sử chạy workflow
- **Luồng chính:** Admin mở danh sách lần chạy workflow → lọc theo workflow, trạng thái (thành công / lỗi / đã chuyển xử lý tay), loại lỗi, ticket, khoảng thời gian → mở một lần chạy: các bước, thời gian từng bước, đầu vào và đầu ra **đã được che** (BR-NK-03), thông báo lỗi.
- **Loại lỗi cần phân biệt:** `AI_TIMEOUT` (AI không phản hồi trong thời gian chờ, Admin cấu hình, demo 30 giây); `AI_INVALID_OUTPUT` (sai định dạng, nhãn loại hoặc nhóm không tồn tại, độ tin cậy ngoài 0–1); `EXTERNAL_API_ERROR` (gọi hệ thống ngoài lỗi); `AUDIT_UNAVAILABLE` (BR-NK-06).
- **Nhánh — AI lỗi → chuyển xử lý tay:** khi gặp `AI_TIMEOUT` hoặc `AI_INVALID_OUTPUT` → hệ thống **không áp dụng** kết quả của AI lên ticket → chuyển ticket sang hàng chờ xử lý tay (Q-08) → ghi audit log `TICKET_FALLBACK_TO_MANUAL` (`actor_type = SYSTEM`, kèm mã lần chạy và loại lỗi) → sự kiện này hiện trong UC-NK-01.
- **Nhánh — TECHNICIAN:** với ticket trong phạm vi, chỉ thấy tóm tắt: workflow nào, lỗi loại gì, lúc nào, đã chuyển cho ai. Không thấy đầu vào, đầu ra chi tiết.
- **Ngoại lệ:** không có `workflow_log:read` → `403`. Lần chạy của ticket ngoài phạm vi → `403`.
- **BR:** BR-09, BR-NK-01, BR-NK-03, BR-NK-05, BR-NK-06.

### UC-NK-05 Ghi nhật ký (Hệ thống)
- **Luồng chính:** khi có hành động thuộc phạm vi BR-09 hoặc BR-NK-01 → che dữ liệu nhạy cảm (BR-NK-03) → ghi một bản ghi mới, **chỉ thêm** (BR-NK-02). Với thay đổi trong DB của hệ thống, bản ghi được ghi **trong cùng giao dịch** với thay đổi: ghi log lỗi thì thay đổi cũng bị hủy.
- **Nhánh — hành động cấp quyền (gọi hệ thống ngoài):** ghi bản ghi "bắt đầu thực thi" **trước** khi gọi API. Ghi không được → **không gọi API**, yêu cầu ở trạng thái "Đã duyệt — thực thi lỗi" với lỗi `AUDIT_UNAVAILABLE`, ticket chuyển xử lý tay (BR-NK-06). Gọi API xong mà ghi kết quả thất bại → đưa vào hàng đợi ghi lại, cảnh báo Admin; không gọi API lần nữa.
- **Ngoại lệ:** đầu vào chứa trường nhạy cảm không nhận diện được theo tên → vẫn che theo mẫu (BR-NK-03). Có request sửa / xóa → `405`.
- **BR:** BR-09, BR-13, BR-NK-02, BR-NK-03, BR-NK-06.

---

## 5. Quy tắc nghiệp vụ mới

**Báo cáo**
- **BR-BC-01**: Mọi báo cáo và Dashboard tính theo định nghĩa M-01…M-07 ở mục 2. Mỗi báo cáo ghi rõ kỳ, múi giờ và tham số đang dùng.
- **BR-BC-02**: Số liệu chỉ tính trên tập ticket người xem được đọc theo `ticket:read` (mvp.md mục 4). Phạm vi được lọc **trong truy vấn ở backend**, trước khi tổng hợp. Không được tổng hợp trên toàn bộ ticket rồi ẩn bớt ở giao diện.
- **BR-BC-03**: Báo cáo chỉ **đọc**. Không endpoint báo cáo nào được tạo, sửa hoặc xóa ticket, audit log hay dữ liệu nghiệp vụ. Nên truy vấn bằng tài khoản DB chỉ có quyền đọc.
- **BR-BC-04**: File xuất chỉ chứa các cột trong danh sách cho phép: mã ticket, loại, nhóm, ưu tiên, trạng thái, các mốc thời gian, chỉ số M-01…M-07. **Không** chứa ghi chú nội bộ, mật khẩu hoặc hash, token, API key, secret 2FA, đầu vào / đầu ra workflow. Ô bắt đầu bằng `=`, `+`, `-`, `@` được thêm tiền tố `'` để chống chèn công thức (CSV injection).
- **BR-BC-05**: Mỗi lần xuất ghi audit log `REPORT_EXPORTED` (`actor_type = USER`; `after` = loại báo cáo, bộ lọc, định dạng, số dòng). Ghi log thất bại → không trả file.

**Nhật ký**
- **BR-NK-01**: Mở rộng phạm vi ghi của BR-09, giữ nguyên cấu trúc bản ghi, cho: thay đổi ticket (tạo, loại, nhóm, người xử lý, ưu tiên, trạng thái, phản hồi, ghi chú nội bộ, đóng, mở lại, chấm điểm); mọi bước của yêu cầu phê duyệt; kết quả chạy workflow và việc chuyển xử lý tay; xuất báo cáo; đổi tham số chỉ số. Thao tác của AI dùng `actor_type = AI`. Thao tác do workflow hoặc chính sách tự động dùng `actor_type = SYSTEM`.
- **BR-NK-02**: Nhật ký **chỉ ghi thêm**. Không ai sửa hoặc xóa được, **kể cả ADMIN**: không có API sửa hoặc xóa (`405`), và tài khoản DB của ứng dụng chỉ có quyền `INSERT` và `SELECT` trên bảng audit log. Muốn đính chính thì ghi một bản ghi mới tham chiếu bản ghi cũ.
- **BR-NK-03**: Trước khi ghi, mọi giá trị `before`, `after`, đầu vào và đầu ra workflow, phản hồi API phải được **che**: trường có tên chứa `password`, `token`, `secret`, `api_key`, `authorization`, `cookie`, `otp` → `"***"`. Chuỗi khớp mẫu `Bearer …`, JWT, API key → `"***"`. Không bao giờ ghi `password_hash`, `totp_secret_enc`, `api_key_hash` vào `before` / `after` (cùng tinh thần BR-05, BR-13).
- **BR-NK-04**: REQUESTER chỉ thấy các sự kiện thuộc **danh sách cho phép** (UC-NK-01). Danh sách được lọc ở backend. Sự kiện mới chưa được phân loại mặc định là **ẩn** với REQUESTER.
- **BR-NK-05**: AI không phản hồi trong thời gian chờ hoặc trả kết quả không hợp lệ → kết quả AI không được áp dụng; ticket chuyển xử lý tay; ghi `TICKET_FALLBACK_TO_MANUAL`.
- **BR-NK-06**: Với hành động **cấp quyền**, ghi được audit log là điều kiện để thực hiện. Ghi log thất bại → **không cấp quyền**, yêu cầu ghi nhận lỗi `AUDIT_UNAVAILABLE` và ticket chuyển xử lý tay.

---

## 6. Đề xuất bổ sung ma trận quyền

Đề xuất để chủ file [mvp.md](mvp.md) quyết định. File này **không** sửa `mvp.md`.

| Quyền đề xuất | ADMIN | TECHNICIAN | REQUESTER | AI_AGENT | Dùng ở |
|---|---|---|---|---|---|
| `report:read` | all | assigned, group (không xem số liệu của từng kỹ thuật viên khác) | ❌ | ❌ | UC-BC-01, 02, 04 (UC-BC-03 chỉ ADMIN) |
| `report:export` | ✅ | ❌ | ❌ | ❌ | UC-BC-05 |
| `report:configure` | ✅ | ❌ | ❌ | ❌ | UC-BC-06 |
| `ticket_history:read` | all | assigned, group | own (lọc theo BR-NK-04) | all (chỉ đọc, để học từ các lần bị sửa, mvp.md mục 8) | UC-NK-01 |
| `audit:read` | ✅ (toàn hệ thống) | ❌ | ❌ | ❌ | UC-NK-02 |
| `approval:read` | all | assigned, group | own (tóm tắt) | ❌ (xem Q-10) | UC-NK-03 |
| `workflow_log:read` | ✅ (chi tiết) | assigned, group (tóm tắt) | ❌ | ❌ | UC-NK-04 |

Không đề xuất quyền `audit:write`, `audit:update` hay `audit:delete` cho vai trò nào: chỉ backend được ghi (UC-NK-05), và không ai được sửa hoặc xóa (BR-NK-02).

---

## 7. Câu hỏi mở (cần nhóm chốt)

**Vai trò**
- **Q-01**: Có cần vai trò **quản lý / trưởng nhóm** để xem báo cáo cả nhóm, gồm số liệu của từng kỹ thuật viên? Hiện chỉ có 4 vai trò, mỗi người một vai trò. File này không thêm vai trò mới.
- **Q-02**: TECHNICIAN có được thấy số liệu (SLA, điểm hài lòng) của từng đồng nghiệp trong nhóm không, hay chỉ thấy số liệu tổng của nhóm và của chính mình?
- **Q-03**: UC-BC-03 (hiệu quả AI) chỉ dành cho ADMIN, hay TECHNICIAN cũng được xem trong phạm vi của mình?
- **Q-04**: **Ai duyệt yêu cầu cấp quyền** trong ticket? BR-15 chỉ nói ADMIN duyệt *đề xuất của AI* (`GROUP_CATEGORY` / `USER_SKILL`), chưa nói về cấp quyền trên hệ thống ngoài (thư mục, phần mềm…). Cần vai trò "người duyệt" (quản lý trực tiếp, chủ tài nguyên) hay ADMIN duyệt? Có cấm người yêu cầu tự duyệt yêu cầu của chính mình không?

**Dữ liệu và phạm vi ghi log**
- **Q-05**: BR-09 hiện chỉ liệt kê thay đổi về người dùng, vai trò, nhóm, service account, đề xuất AI, ánh xạ Microsoft. Chủ `mvp.md` có đồng ý mở rộng sang ticket, phê duyệt, workflow (BR-NK-01) không? Có cần thêm trường (ví dụ `ticket_id`, `request_id` để nối các bước) vào `AuditLog` không?
- **Q-06**: UC-03 ghi "mọi request của AI ghi log với `actor_type = AI`", kể cả request chỉ đọc. Request đọc có vào bảng `AuditLog` không, hay vào một log truy cập riêng? Nếu ghi hết thì UC-NK-02 sẽ có rất nhiều dòng.
- **Q-07**: Ticket chuyển qua nhiều nhóm thì tính cho nhóm nào: nhóm cuối cùng (đang đề xuất), nhóm đầu tiên, hay chia cho mỗi nhóm đã xử lý?
- **Q-08**: Ticket chuyển xử lý tay thì vào đâu: hàng chờ của nhóm mà AI đã đoán, một nhóm mặc định, hay hàng chờ của ADMIN? Có thử gọi AI lại trước khi chuyển không (số lần thử do module Workflow cấu hình)?
- **Q-09**: REQUESTER có được thấy tên người duyệt và tên kỹ thuật viên xử lý trong lịch sử không?
- **Q-10**: AI_AGENT có cần đọc kết quả phê duyệt để tự xử lý tiếp, hay việc đó do workflow (`SYSTEM`) đảm nhận?
- **Q-11**: Thời gian lưu audit log là bao lâu? "Không ai xóa được" (BR-NK-02) có cần ngoại lệ cho việc tự động xóa sau hạn lưu trữ không?
- **Q-12**: Thực thể `AuditLog` có `actor_id` (null khi `SYSTEM`). Khi `actor_type = AI` thì `actor_id` trỏ tới `ServiceAccount` hay `User`? Cần chốt để UC-NK-02 lọc theo service account.

**Cách tính chỉ số**
- **Q-13**: Phản hồi đầu của AI có được tính để đạt SLA phản hồi (M-01, M-03) không, hay chỉ tính phản hồi của người?
- **Q-14**: SLA tính theo giờ lịch (mặc định cho demo) hay giờ làm việc? Nếu theo giờ làm việc thì lịch làm việc và ngày nghỉ do ai cấu hình?
- **Q-15**: Với M-04, có tính là "kỹ thuật viên thao tác" khi một người chỉ **duyệt** yêu cầu cấp quyền rồi AI xử lý phần còn lại không? Đang đề xuất: **không** tính.
- **Q-16**: Ai được đánh dấu gợi ý KB là hữu ích (người gửi, kỹ thuật viên, hay cả hai)? Việc đánh dấu thuộc module Kho kiến thức.
- **Q-17**: Mức ưu tiên và mục tiêu SLA do module Ticket hay module Báo cáo cấu hình? File này tạm đặt ở UC-BC-06.
- **Q-18**: Báo cáo tính trực tiếp khi xem (mặc định cho demo) hay dùng số liệu tổng hợp sẵn theo chu kỳ? Dashboard có dùng chung API với báo cáo không?

**Xuất file**
- **Q-19**: Có cần định dạng PDF không? Bình luận kèm điểm hài lòng có được xuất ra file không (có thể chứa thông tin cá nhân)?
- **Q-20**: Có cần xuất audit log (UC-NK-02) ra file không? Nếu có thì cần quyền riêng `audit:export` và cũng phải ghi log.

Xem thêm: [mvp.md](mvp.md) · [use_cases.md](use_cases.md) · [information_architecture.md](information_architecture.md)
