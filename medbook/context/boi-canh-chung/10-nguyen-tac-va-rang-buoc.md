# Bối cảnh chung · 10 — Nguyên tắc và ràng buộc cho mọi thay đổi

> **Mã:** CTX-NT · **Phiên bản:** v0.1 · **Ngày:** 2026-09-29
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** `08-khoang-trong.md` (mã `HT-GAP-…`) của thư mục này

Tám nguyên tắc dưới đây ràng buộc **mọi** đặc tả, thiết kế, task và code viết thêm cho MedBook kể
từ đây — không riêng tính năng danh sách chờ. Mỗi nguyên tắc có lý do và bằng chứng cụ thể, không
phải quy ước áp từ ngoài vào.

---

## Nguyên tắc

### I. Không phá hợp đồng đang chạy
16 endpoint hiện có PHẢI giữ nguyên request, response và mã lỗi. Thay đổi chỉ được **thêm**
endpoint mới, không sửa endpoint cũ. 14 test tích hợp hiện có (`tests/api.test.js`) PHẢI luôn
xanh; một phần việc không giữ được 14 test này xanh thì coi như chưa xong, bất kể phần mới đã
đúng đến đâu. *Lý do:* đây là bản demo dạy học, không có môi trường thử riêng; hỏng hệ thống đang
chạy là hỏng buổi học. Nguồn: `K-10`, `HT-API-01…16`.

### II. Không thêm dependency runtime
Chỉ dùng `express` và `pg`. Thêm bất kỳ thư viện runtime nào — kể cả để tiện — là thay đổi một
quyết định kiến trúc, phải được ghi lại và giải thích rõ, không được tự quyết ở mức một phần việc
nhỏ. *Lý do:* dự án cam kết chạy bằng một lệnh `docker compose up`, không có bước build, không có
ma trận tương thích để bảo trì. Nguồn: `K-08`, `K-09`.

### III. Kiến trúc phân lớp, không nhảy tầng
Route chỉ nhận và trả HTTP rồi gọi service; route KHÔNG viết SQL hay chứa luật nghiệp vụ. Service
chứa luật nghiệp vụ và giao dịch; service KHÔNG đụng đối tượng request/response. Repository chỉ
chứa SQL tham số hóa; repository KHÔNG quyết định mã HTTP hay chứa luật nghiệp vụ. Service KHÔNG
được gọi service khác — chỉ gọi xuống repository, để tránh vòng phụ thuộc. *Lý do:* bốn endpoint
hiện có gọi thẳng repository từ route là một sai lệch đã biết, không phải mẫu để nhân rộng. Nguồn:
`K-06`, `HT-GAP-20`.

### IV. SQL thuần, tham số hóa, không ORM
Mọi câu lệnh SQL PHẢI dùng tham số (`$1, $2, …`). Nối chuỗi giá trị người dùng vào SQL bị từ chối
ngay khi rà soát, không có ngoại lệ. Không đưa ORM hay query builder vào dự án. *Lý do:* đơn giản,
minh bạch cho mục đích giảng dạy, và tránh lớp trừu tượng che giấu lỗi tiêm SQL. Nguồn: `K-01`.

### V. Đổi trạng thái đồng thời phải an toàn
Mọi thao tác đổi `slots.status` PHẢI nằm trong một giao dịch, dùng khóa dòng trước khi đọc và
ghi. Mọi phép chuyển trạng thái đơn lẻ khác PHẢI dùng câu lệnh cập nhật có điều kiện (đổi khi
đúng trạng thái nguồn, trả về dòng đã đổi), không đọc-rồi-ghi. Ràng buộc nghiệp vụ sống còn PHẢI
có một chỉ mục duy nhất ở cơ sở dữ liệu làm lớp phòng thủ cuối — không chỉ dựa vào kiểm tra ở tầng
service. *Lý do:* việc sửa slot hiện tại kiểm tra rồi ghi ở hai câu lệnh riêng, không giao dịch,
để lại một khe hở đồng thời thật; không được lặp lại lỗi này ở phần việc mới. Nguồn: `K-04`,
`HT-GAP-10`.

### VI. Không quy tắc nghiệp vụ nào thiếu mã và nguồn quyết định
Mọi quy tắc nghiệp vụ PHẢI có một mã ổn định, một nguồn quyết định (ai, khi nào, dựa trên gì), và
một thành phần chịu trách nhiệm duy nhất. Mã chỉ cấp một lần; loại bỏ thì đánh dấu bỏ, không xóa
mã. Khi thiếu thông tin để quyết một quy tắc, PHẢI dừng lại, gắn nhãn "cần xác nhận", và nêu rõ:
thiếu gì, vì sao quan trọng, ai nên trả lời — không được tự đặt ra quy tắc rồi trình bày như đã
chốt. *Lý do:* đây là điểm dễ tự bịa nhất khi đặc tả còn mơ hồ (ví dụ thứ tự ưu tiên chọn bệnh
nhân), và một quy tắc bịa ra mà không ai chịu trách nhiệm là quy tắc không tin được. Nguồn:
`NT-02`, `NT-03`.

### VII. Tài liệu trước, code sau
Phát hiện tài liệu (yêu cầu hoặc thiết kế) sai hoặc thiếu trong lúc viết code PHẢI dừng lại, sửa
tài liệu và đưa qua đúng cổng xác nhận của nó trước, rồi mới sửa code theo tài liệu đã sửa. KHÔNG
được sửa code cho "đúng ý" rồi để tài liệu lệch lại phía sau. *Lý do:* đây chính là điểm đứt gãy
thông tin mà buổi đầu tiên của case study chỉ ra là nguyên nhân khiến việc rà soát tốn gấp đôi
thời gian. Nguồn: `NT-05`.

### VIII. Kiểm thử từ tiêu chí chấp nhận, không từ code
Mỗi tiêu chí chấp nhận PHẢI có ít nhất một ca kiểm thử trỏ đúng về nó; ca kiểm thử được viết để
chứng minh tiêu chí, không được viết bằng cách quan sát code rồi mô tả lại hành vi của nó. Luồng
nào chạm tới ràng buộc duy nhất ở cơ sở dữ liệu PHẢI có kiểm thử đồng thời (nhiều yêu cầu cùng
lúc). *Lý do:* 3 trong 16 quy tắc của hệ thống hiện có không có kiểm thử nào bảo vệ, 9 quy tắc chỉ
được bảo vệ một phần — "kiểm thử cũ vẫn xanh" là một bảo đảm yếu nếu không chủ động lấp khoảng
trống này ở phần việc mới. Nguồn: `NT-08`, `09-truy-vet.md`.

---

## Ràng buộc kỹ thuật bổ sung

Ngăn xếp cố định: Node.js 20, Express 4, PostgreSQL 16, giao diện HTML/CSS/JavaScript thuần
(ES module, không framework, không bước build). Xác thực là **xác thực demo có chủ ý yếu** (một
header chứa mã người dùng, giả mạo được) — không thay bằng cơ chế xác thực thật trong phạm vi bài
tập; bù lại, mọi kiểm tra **quyền sở hữu dữ liệu** (ví dụ "mục này có phải của người đang gọi
không") PHẢI nằm ở tầng service, không được dựa vào vai trò hay vào header để đảm bảo an toàn.
Thông báo lỗi PHẢI bằng tiếng Việt, ngắn, không lộ câu lệnh SQL, tên bảng hay dấu vết kỹ thuật —
lỗi hệ thống hiện tại lộ nguyên văn thông báo của cơ sở dữ liệu là một sai lệch đã biết và không
được nhân rộng sang phần việc mới. Việc khởi tạo và nạp dữ liệu mẫu PHẢI chạy lại được nhiều lần
mà không lỗi, và KHÔNG BAO GIỜ được ghi đè dữ liệu do người dùng (hoặc một phiên chạy trước) đã
tạo ra — cách nạp dữ liệu mẫu hiện tại từng gây ra lỗi khởi động lại thật; phần việc mới không
được lặp lại kiểu ghi đè này.

## Cách áp dụng khi viết yêu cầu, thiết kế hoặc task mới

Thư mục này (`context/boi-canh-chung/`) là nguồn sự thật về hệ thống đang chạy, có dẫn chứng
dòng code, và nên được đọc trước khi thiết kế bất kỳ phần nào chạm vào luồng đặt lịch, hủy lịch,
hoặc trạng thái slot. Một phần việc chỉ coi là xong khi: không thêm
dependency, 14 test cũ vẫn xanh, mọi quy tắc được giao có vị trí hiện thực và kiểm thử tương ứng,
kiểm tra quyền sở hữu ở tầng service đã có kiểm thử, và mọi giả định trong lúc làm đã được xác
nhận hoặc ghi lại thành câu hỏi mở — không được để ngầm định.

Tài liệu này có hiệu lực cao hơn các tài liệu tham khảo khác trong repo khi có mâu thuẫn về
**ràng buộc bất biến** nêu trên; các tài liệu khác vẫn là nguồn đúng cho chi tiết nghiệp vụ và
thiết kế không được nêu ở đây. Vi phạm một nguyên tắc phải được giải thích rõ lý do và phương án
đơn giản hơn đã bị bỏ qua vì sao, không được vi phạm âm thầm.
