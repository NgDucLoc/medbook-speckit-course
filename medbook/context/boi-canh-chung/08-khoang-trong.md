# Hiện trạng · 08 — Khoảng trống, lỗi và rủi ro

> **Mã:** CTX-HT-08 · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · suy ngược từ code · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** đọc toàn bộ code; **các mục ghi `Probe` đã được tái hiện thật** trên bản ứng dụng tách biệt và cơ sở dữ liệu tạm (2026-09-28), không đụng dữ liệu của ứng dụng đang chạy

Đây là nơi đặt mọi hành vi **không phải luật có chủ ý** (`02-quy-tac-nghiep-vu.md`): lỗi, kiểm tra còn thiếu, tài liệu lệch code. Bộ tài liệu hiện trạng **không sửa** những điều này — chỉ ghi nhận để người quyết định biết, và để các WB sau không giẫm lên.

**Cột Loại:** `Lỗi` (hành vi sai so với ý định rõ ràng) · `Thiếu` (thiếu một kiểm tra hoặc quy tắc) · `Rủi ro` (chưa tái hiện, suy ra từ code) · `Đã biết` (tài liệu dự án đã nêu, chủ ý) · `Lệch tài liệu`.

---

## 1. Danh sách

| ID | Mức | Loại | Vấn đề | Bằng chứng | Tác động lên tính năng danh sách chờ |
| --- | :---: | :---: | --- | --- | --- |
| **HT-GAP-01** | **High** | Lỗi | **Khởi động lại có thể làm ứng dụng không lên được.** Seed ghi đè lịch hẹn mẫu mã 1 và 2 về trạng thái ban đầu mỗi lần khởi động. Nếu lịch 1 đã bị hủy và slot 2 đã được đặt bởi lịch mới, seed đặt lại lịch 1 thành đang hoạt động ⇒ hai lịch hoạt động trên một slot ⇒ vi phạm chỉ mục duy nhất ⇒ `process.exit(1)` | `Probe` · `seed.js:96–104`, `migrate.js:53–55`, `server.js:57–62` | Dữ liệu mẫu và bất kỳ seed mới nào **không được ghi đè dữ liệu người dùng** (TASK-09) |
| **HT-GAP-02** | Medium | Lỗi | **Khởi động lại hoàn tác thao tác của người dùng trên dữ liệu mẫu.** Lịch mẫu đã hủy sống lại (và slot về đã đặt); slot 1–12 bị đặt lại giờ bắt đầu / kết thúc; slot bị staff chặn giờ trở lại còn trống | `Probe` (3 kịch bản) · `seed.js:68–104` | Demo và kiểm thử thủ công của tính năng mới không lặp lại được nếu ứng dụng khởi động lại giữa chừng |
| **HT-GAP-03** | **High** | Thiếu | **Không có luật "một bệnh nhân không đặt hai lịch trùng giờ".** Đề bài WB-2 liệt kê luật này trong "Business Rules", nhưng không có trong code lẫn `doc/prod.md`. Đã đặt được hai lịch cùng giờ ở hai bác sĩ khác nhau | `Probe` (đặt slot 1, rồi slot trùng giờ của bác sĩ khác ⇒ cả hai 201) · `appointmentService.js:7–35` | Luật lọc ứng viên BR-03 (d) là **luật mới**, không thừa hưởng từ hệ thống; luồng đặt chủ động vẫn cho phép trùng giờ |
| **HT-GAP-04** | Medium | Thiếu | **Đặt được slot ở quá khứ.** Chỉ danh sách hiển thị lọc `date >= current_date`; đặt theo mã slot không kiểm ngày | `Probe` (slot 2020-01-01 đặt được) · `appointmentService.js:15–17` | Không có khái niệm "quá sát giờ" nào để tái sử dụng cho BR-16 |
| **HT-GAP-05** | Medium | Thiếu | **Tạo / sửa slot không kiểm tra giá trị.** Ngày hoặc giờ sai định dạng ⇒ 500; `doctorId` không tồn tại ⇒ 500; giờ kết thúc trước giờ bắt đầu vẫn tạo được; hai slot chồng giờ của cùng bác sĩ được phép | `Probe` (ngày `abc`, giờ `25:99`, `doctorId` 999, 10:00→09:00) · `slotService.js:18–49`, `migrate.js:35–42` | Xác nhận R-08 của WB-2: BR-03 (d) so khoảng giờ, kết quả có thể lạ nếu dữ liệu slot đã chồng nhau |
| **HT-GAP-06** | Medium | Lỗi | **Lỗi 500 lộ nguyên văn thông báo của PostgreSQL** (tên ràng buộc, kiểu dữ liệu) cho client, thay vì "Lỗi máy chủ" | `Probe` · `server.js:38–42` (`error.message`) | Mọi endpoint mới kế thừa cách xử lý lỗi này nếu không tự bọc |
| **HT-GAP-07** | Medium | Lỗi | **`id` trong body ghi đè `id` trên đường dẫn** ở `PUT /api/slots/:id` do ghép `{ id: req.params.id, ...req.body }`. `PUT /api/slots/3` với body `{"id":4,…}` sửa nhầm slot 4 | `Probe` · `doctors.routes.js:68` | Mẫu ghép `...req.body` không nên nhân rộng sang endpoint mới |
| **HT-GAP-08** | Low | Lỗi | **Xác nhận lịch không tồn tại trả 409**, cùng thông báo với "không xác nhận được", thay vì 404 | `Probe` · `appointmentService.js:45–52` | — |
| **HT-GAP-09** | Low | Thiếu | **`type` không hợp lệ bị đổi âm thầm thành `in_person`** thay vì báo 400 | `Probe` · `appointmentService.js:9` | Đề xuất sẽ cần hình thức khám (R-16); đừng bắt chước kiểu đổi âm thầm |
| **HT-GAP-10** | Medium | Rủi ro | **Sửa slot kiểm tra rồi ghi không có giao dịch.** `countActiveBySlot` và `update slots` là hai lệnh riêng. Nếu bệnh nhân đặt xen giữa trong lúc staff gửi `{status:"available", …}` cho một slot vừa trống, slot bị đặt về *còn trống* dù đã có lịch hoạt động ⇒ người kế tiếp đặt vào sẽ va chỉ mục duy nhất và nhận lỗi 500 | Suy ra từ `slotService.js:34–46`; **chưa tái hiện** | Chính là vùng tính năng mới sẽ chạm (`slots.status` khi slot trống / bị chiếm — BR-10). Mọi đường mới đổi `slots.status` phải nằm trong giao dịch |
| **HT-GAP-11** | Low | Thiếu | **Header người dùng đọc bằng `parseInt`**: `"1abc"` được chấp nhận là người dùng 1 | `Probe` · `authService.js:35`, `validate.js:3–6` | — |
| **HT-GAP-12** | *Cao nếu lên thật* | Đã biết | **Xác thực demo giả mạo được** (đổi header hoặc `localStorage`); mật khẩu lưu chữ thường | ADR-002 · `authService.js:24` | Kiểm quyền sở hữu dữ liệu phải ở tầng service (BR-14 của WB-2) |
| **HT-GAP-13** | Medium | Lỗi | **Biến `DEMO_AUTH_ENABLED` và `NODE_ENV` được đặt nhưng không nơi nào đọc** — không tắt được đăng nhập demo, không phân biệt môi trường | grep toàn bộ `server.js`, `src`, `public`, `tests` không thấy | Không có công tắc môi trường sẵn có để tắt Offer Engine khi test; phải tự thêm |
| **HT-GAP-14** | Medium | Rủi ro | **`npm test` xóa sạch mọi bảng** (`truncate … restart identity cascade`) trên `DATABASE_URL` hiện hành, không có rào chắn; địa chỉ mặc định trỏ tới **CSDL phát triển** (`localhost:55432/medbook`) | `tests/api.test.js:27–36`, `seed.js:3–14`, `pool.js:3–4` | Test mới cũng dùng khuôn này ⇒ cùng rủi ro |
| **HT-GAP-15** | Low | Thiếu | **Không có chỉ mục** cho `appointments.patient_id`, `slots.doctor_id`, `slots.date` | Danh sách chỉ mục thật trên CSDL | Truy vấn chọn ứng viên sẽ cần chỉ mục riêng |
| **HT-GAP-16** | Low | Lỗi | **`availableSlots` trả về chuỗi** (`"3"`), không phải số | `Probe` · `doctorRepository.js:22` | — |
| **HT-GAP-17** | Low | Lỗi | **Ký tự `%`, `_` trong ô tìm tên không được thoát** ⇒ `q=%` khớp mọi bác sĩ | `Probe` · `doctorRepository.js:15` | — |
| **HT-GAP-18** | Medium | Thiếu | **Bộ 14 test bỏ trống nhiều quy tắc:** hủy lịch của người khác (403) · hủy hai lần (409) · xác nhận lịch đã hủy hoặc không tồn tại · đặt thiếu `slotId` · đăng nhập thành công · tìm bác sĩ · danh sách lịch hẹn (cả hai vai) · danh sách slot của staff · trạng thái slot sai · đặt đồng thời bởi **hai người khác nhau** | `09-truy-vet.md` §2–§3 | Điều kiện "14 test cũ vẫn xanh" chỉ bảo vệ được phần đã có test |
| **HT-GAP-19** | Low | Thiếu | **Ý định chưa rõ:** bệnh nhân hủy được cả lịch *đã xác nhận*; giao diện còn hiện nút Hủy cho lịch đó. Không tài liệu nào nói được hay không | `appointmentService.js:65–67` (chỉ chặn lịch đã hủy) · `patient.js:135–137` | Nếu hủy lịch đã xác nhận cũng kích hoạt đề xuất (BR-01) thì phạm vi kích hoạt rộng hơn dự tính |
| **HT-GAP-20** | Low | Lệch tài liệu | **Bốn endpoint chỉ đọc gọi repository thẳng từ route**, bỏ qua service, trái mô tả "route → service → repository" | `doctors.routes.js:13, 22, 35, 44` · `doc/prod.md` §8 | Người viết đơn vị mới dễ hiểu sai "quy ước phân lớp" |
| **HT-GAP-21** | **High** | Lệch tài liệu | **Đề bài WB-2 mô tả hiện trạng có những thứ hệ thống không có** — xem bảng ở §2 | so sánh đề bài với code | Xem R-17 ở `WB-2/bai-nop/01-yeu-cau/06-ra-soat-yeu-cau.md` |
| **HT-GAP-22** | Low | Lỗi | **Lỗi cú pháp JSON trả thông báo tiếng Anh** của bộ phân tích | `Probe` | — |
| **HT-GAP-23** | **Medium** | Rủi ro | **Múi giờ.** PostgreSQL và ứng dụng chạy **UTC**; `slots.date` và giờ không kèm múi giờ. Giờ do staff nhập (giờ địa phương) sẽ bị so với `now()` UTC ⇒ mọi phép tính "còn bao lâu tới giờ khám" lệch bằng độ lệch múi giờ (UTC+7 ⇒ 7 giờ). Lọc "từ hôm nay" theo ngày UTC lệch một ngày so với ô ngày mặc định của giao diện (giờ máy người dùng) trong 00:00–07:00 giờ địa phương | Kiểm `show timezone` = UTC; `pool.js`; `api.js:6–10`; `slotRepository.js:26, 41` | **Ảnh hưởng trực tiếp BR-06 (hạn trả lời) và BR-16 (thời gian tối thiểu trước giờ khám):** hai luật này so sánh `date + start_time` với thời điểm hiện tại. Cần quyết định múi giờ chuẩn của dữ liệu trước khi hiện thực |

**Thống kê:** 23 mục — High: 3 (01, 03, 21) + 1 "cao nếu lên thật" (12) · Medium: 10 · Low: 9. Đã tái hiện thật: 13 mục (`Probe`).

---

## 2. HT-GAP-21 — Đề bài WB-2 so với hệ thống thật

Đề bài WB-2 (`de-bai/WB-2.docx`, phần "Hiện trạng hệ thống") nêu như sau; cột phải là kết quả đối chiếu với code.

| Đề bài nói | Hệ thống thật |
| --- | :---: |
| Chức năng: Quản lý bệnh nhân | ❌ Chỉ có bảng `patients` do seed nạp; không endpoint thêm / sửa / xem |
| Chức năng: Quản lý bác sĩ | ⚠️ Chỉ đọc (tìm, xem); không thêm / sửa |
| Chức năng: Đặt lịch khám, Hủy lịch | ✅ |
| Chức năng: **Đổi lịch** | ❌ Không có |
| Chức năng: Quản lý lịch làm việc của bác sĩ | ✅ (staff thêm và điều chỉnh slot) |
| Chức năng: **Thông báo lịch khám** | ❌ Không có |
| Actors: Patient, Medical Staff, **Admin** | ⚠️ Chỉ Patient và Staff |
| Luật: Một lịch khám chỉ thuộc 1 bác sĩ | ✅ (qua slot) |
| Luật: **Một bệnh nhân không được đặt hai lịch trùng giờ** | ❌ HT-GAP-03 |
| Luật: Một slot tối đa một bệnh nhân | ✅ HT-BR-07 |
| Luật: **Lịch đã xác nhận phải thông báo cho bác sĩ** | ❌ Không có thông báo; bác sĩ không đăng nhập |
| Luật: **Nhân viên có thể đổi lịch** | ❌ |
| Luật: Bệnh nhân chỉ có thể huỷ lịch | ✅ (không có đổi lịch) |
| Luật: Chỉ đặt slot `available`; đã đặt ⇒ 409 | ✅ HT-BR-05 |
| Luật: Đặt ⇒ `booked`; hủy ⇒ `cancelled` và slot `available` | ✅ HT-BR-06, 09 |
| Luật: Bệnh nhân chỉ hủy lịch của mình; staff hủy bất kỳ | ✅ HT-BR-09 |
| Luật: Chỉ staff xác nhận; lịch `cancelled` là cuối | ✅ HT-BR-10 |
| Luật: Không mở lại slot đang có lịch hoạt động | ✅ HT-BR-15 |
| Luật: Chỉ hai vai trò; bác sĩ không đăng nhập | ✅ |
| Use case: xem metrics dashboard (bệnh nhân: số lịch của tôi, **lịch chờ xác nhận**, slot trống) | ⚠️ Có ba chỉ số, nhưng "lịch chờ xác nhận" **không** là chỉ số riêng (HT-AC-13.1) |

Kết luận: 11 ✅ · 3 ⚠️ · 6 ❌ trong số 20 dòng. Các dòng ❌ là chỗ dễ khiến người (hoặc AI) tin có sẵn một thứ để tái sử dụng mà thực tế không có.

---

## 3. Việc chưa kiểm chứng

- Mọi tiêu chí `UI` chưa được mở trình duyệt để bấm; chỉ đọc code giao diện.
- HT-GAP-10 (khe hở đồng thời khi sửa slot) suy ra từ code, chưa tái hiện.
- Hành vi dưới tải, và khi nhiều tiến trình ứng dụng chạy cùng lúc, chưa thử.
