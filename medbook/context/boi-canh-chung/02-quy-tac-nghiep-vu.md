# Hiện trạng · 02 — Quy tắc nghiệp vụ

> **Mã:** CTX-HT-02 · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · suy ngược từ code · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** `authService.js`, `appointmentService.js`, `slotService.js`, `*Repository.js`, `requireRole.js`, `migrate.js`, `tests/api.test.js`, `doc/prod.md` §6

Các quy tắc dưới đây là những gì hệ thống **đang thực thi có chủ ý**: có trong code **và** khớp `doc/prod.md`, có test, hoặc có chú thích trong code. Hành vi xảy ra ngẫu nhiên hoặc thiếu kiểm tra **không** ghi ở đây mà ở `08-khoang-trong.md` — để không biến lỗi thành luật.

Cột **Kiểm chứng**: `Test` = có test trong `tests/api.test.js` · `Probe` = tôi đã chạy thật (2026-09-28) · `—` = chưa có gì ngoài đọc code.

---

## Nhóm A — Đăng nhập và phân quyền

### HT-BR-01 — Đăng nhập demo bằng email và mật khẩu
Email được cắt khoảng trắng và không phân biệt hoa thường; mật khẩu so khớp đúng với mật khẩu demo của tài khoản. Thiếu email hoặc mật khẩu ⇒ từ chối *"Vui lòng nhập email và mật khẩu"* (400). Sai ⇒ *"Sai email hoặc mật khẩu demo"* (401). **Không đăng nhập được chỉ bằng mã người dùng.** Kết quả không chứa mật khẩu.
- **Bằng chứng:** `authService.js:16–29`, `userRepository.js:14–20`
- **Kiểm chứng:** `Test` :135–150 · `Probe` (email viết hoa vẫn vào được)

### HT-BR-02 — Mọi thao tác ngoài đăng nhập cần khai báo người dùng
Mọi endpoint, trừ `/health`, `GET /api/demo-users`, `POST /api/demo-login`, đòi header `X-Demo-User-Id`. Thiếu ⇒ *"Thiếu tài khoản demo"*; không đọc ra số ⇒ *"Tài khoản demo không hợp lệ"*; không có người dùng đó ⇒ *"Không tìm thấy tài khoản demo"* (đều 401). Người dùng tìm được sẽ được gắn vào request cho các bước sau.
- **Bằng chứng:** `authService.js:31–44`, `demoAuth.js`
- **Kiểm chứng:** `Test` :53–57 (chỉ trường hợp thiếu header)

### HT-BR-03 — Phân quyền theo vai trò
| Việc | Patient | Staff |
| --- | :---: | :---: |
| Xem chuyên khoa, bác sĩ, khung giờ còn trống | ✅ | ✅ |
| Đặt lịch · xem lịch của mình | ✅ | ❌ |
| Xem mọi lịch hẹn · xác nhận lịch | ❌ | ✅ |
| Xem slot quản lý · thêm slot · sửa slot | ❌ | ✅ |
| Hủy lịch | ✅ chỉ lịch của mình | ✅ mọi lịch |

Sai vai trò ⇒ *"Không đủ quyền"* (403).
- **Bằng chứng:** `requireRole.js`; route ở `appointments.routes.js`, `doctors.routes.js`. Riêng **hủy lịch không có `requireRole`** — kiểm quyền nằm trong service (`appointmentService.js:62–64`)
- **Kiểm chứng:** `Test` :59–65 (bệnh nhân gọi endpoint staff) · `Probe` (staff đặt lịch ⇒ 403; bệnh nhân hủy lịch người khác ⇒ 403)

### HT-BR-04 — Không lộ mật khẩu qua API
Danh sách tài khoản demo và thông tin người dùng hiện tại chỉ gồm: mã, tên, email, vai trò, mã hồ sơ bệnh nhân.
- **Bằng chứng:** `userRepository.js:3`, `authService.js:5–10`
- **Kiểm chứng:** `Probe` (khóa trả về: `id, name, email, role, patientId`)

---

## Nhóm B — Đặt lịch

### HT-BR-05 — Chỉ đặt được slot còn trống
Slot không tồn tại ⇒ *"Không tìm thấy khung giờ"* (404). Slot đã ở trạng thái đã đặt ⇒ *"Khung giờ đã được đặt"* (409). Thiếu `slotId` ⇒ *"Thiếu slotId"* (400).
- **Bằng chứng:** `appointmentService.js:8, 15–17`
- **Kiểm chứng:** `Test` :79–87 · `Probe` (thiếu `slotId`)

### HT-BR-06 — Kết quả của một lần đặt lịch thành công
Tạo một lịch hẹn ở trạng thái *đã đặt (chờ xác nhận)* và chuyển slot sang *đã đặt*, **cùng một giao dịch**. Hình thức khám là *online* nếu bệnh nhân chọn online; **mọi giá trị khác** thành *trực tiếp*. Lịch hẹn thuộc về bệnh nhân đang đăng nhập.
- **Bằng chứng:** `appointmentService.js:9, 14–25`, `appointmentRepository.js:51–59`
- **Kiểm chứng:** `Test` :67–77 · `Probe` (`type: "weird"` ⇒ `in_person`). *Việc đổi âm thầm là hành vi hay ý định? Xem HT-GAP-09.*

### HT-BR-07 — Một slot chỉ có một lịch hẹn đang hoạt động
"Đang hoạt động" là *đã đặt* hoặc *đã xác nhận*. Được bảo đảm hai lớp: khóa dòng slot trong lúc đặt, và chỉ mục duy nhất từng phần ở cơ sở dữ liệu. Lịch đã hủy không tính, nên slot được đặt lại sau khi hủy.
- **Bằng chứng:** `slotRepository.js:64–70` (`for update`), `migrate.js:53–55`
- **Kiểm chứng:** `Test` :178–189 (xem HT-BR-08)

### HT-BR-08 — Hai bệnh nhân đặt đồng thời một slot: đúng một người thành công
Hai yêu cầu cùng lúc cho cùng một slot ⇒ một lần thành công (201), một lần bị từ chối (409); không lỗi hệ thống.
- **Bằng chứng:** HT-BR-05, HT-BR-07
- **Kiểm chứng:** `Test` :178–189 *(cả hai request cùng một người dùng — không phủ trường hợp hai người khác nhau, nhưng cơ chế khóa như nhau)*

---

## Nhóm C — Hủy và xác nhận

### HT-BR-09 — Hủy lịch
- Lịch không tồn tại ⇒ *"Không tìm thấy lịch hẹn"* (404).
- Bệnh nhân chỉ hủy lịch **của mình**; lịch người khác ⇒ *"Không đủ quyền"* (403). Staff hủy được lịch của bất kỳ ai.
- Lịch đã hủy ⇒ *"Lịch hẹn đã bị hủy"* (409). *Đã hủy là trạng thái cuối.*
- Thành công: lịch → *đã hủy* và slot → *còn trống*, **cùng một giao dịch** (có khóa dòng lịch hẹn).
- Cả lịch *đã đặt* lẫn *đã xác nhận* đều hủy được. *(Ý định chưa rõ — xem HT-GAP-19.)*
- **Bằng chứng:** `appointmentService.js:54–80`
- **Kiểm chứng:** `Test` :89–103 (bệnh nhân hủy lịch của mình, slot về còn trống), :163–176 (staff hủy) · `Probe` (hủy lịch người khác, hủy hai lần, hủy lịch không tồn tại)

### HT-BR-10 — Xác nhận lịch
Chỉ staff. Chỉ chuyển được từ *đã đặt* sang *đã xác nhận*; trường hợp khác (đã xác nhận, đã hủy, **không tồn tại**) đều ⇒ *"Chỉ xác nhận được lịch đang chờ xác nhận"* (409). Dùng một câu lệnh có điều kiện nên không có khe hở thời gian.
- **Bằng chứng:** `appointmentRepository.js:82–88`, `appointmentService.js:45–52`
- **Kiểm chứng:** `Test` :105–114 (thành công) · `Probe` (lịch đã hủy ⇒ 409; lịch không tồn tại ⇒ 409, xem HT-GAP-08)

---

## Nhóm D — Xem thông tin

### HT-BR-11 — Ai thấy lịch hẹn nào
Bệnh nhân chỉ thấy lịch của mình, mới nhất trước. Staff thấy mọi lịch, lọc được theo ngày; kèm tên và số điện thoại bệnh nhân.
- **Bằng chứng:** `appointmentRepository.js:3–29, 36–49`
- **Kiểm chứng:** — *(không có test cho hai endpoint danh sách)*

### HT-BR-12 — Chỉ hiển thị khung giờ còn trống từ hôm nay
Danh sách dành cho bệnh nhân (`/slots/available`, `/doctors/:id/slots`) chỉ gồm slot còn trống có ngày **từ hôm nay trở đi**, sắp xếp theo ngày, giờ, tên bác sĩ.
- **Bằng chứng:** `slotRepository.js:25–27, 39–47`
- **Kiểm chứng:** `Test` :98–102 (slot xuất hiện lại sau khi hủy) · `Probe`

### HT-BR-13 — Tìm bác sĩ
Lọc theo chuyên khoa (khớp chính xác) và/hoặc theo tên (chứa chuỗi, không phân biệt hoa thường); kết hợp bằng "và". Mỗi bác sĩ kèm chuyên khoa, phòng và **số khung giờ còn trống từ hôm nay**. Sắp theo tên chuyên khoa rồi tên bác sĩ. Không có kết quả ⇒ danh sách rỗng, không lỗi.
- **Bằng chứng:** `doctorRepository.js:7–32`
- **Kiểm chứng:** `Probe` (lọc chuyên khoa 1 ⇒ 2 bác sĩ; tên "Hùng" ⇒ 1) · *không có test*

---

## Nhóm E — Quản lý khung giờ (staff)

### HT-BR-14 — Thêm slot
Bắt buộc có `doctorId`, `date`, `startTime`, `endTime`; thiếu ⇒ *"Thiếu <tên trường>"* (400). Slot mới **luôn** ở trạng thái còn trống.
- **Bằng chứng:** `slotService.js:18–26`
- **Kiểm chứng:** `Test` :116–133 · `Probe` (thiếu trường) · *hệ thống **không** kiểm tra giá trị hợp lệ của các trường — xem HT-GAP-05*

### HT-BR-15 — Sửa slot
Staff đổi được giờ bắt đầu, giờ kết thúc, trạng thái (chỉ giá trị *còn trống* hoặc *đã đặt*; giá trị khác ⇒ *"Trạng thái slot không hợp lệ"* 400). Trường không gửi thì giữ nguyên. **Không được mở lại (đặt còn trống) slot đang có lịch hẹn đang hoạt động** ⇒ *"Không thể mở lại khung giờ đang có lịch hẹn"* (409); muốn giải phóng phải hủy lịch. Slot không tồn tại ⇒ *"Không tìm thấy khung giờ"* (404).
- **Bằng chứng:** `slotService.js:6–12, 28–49` — kèm chú thích của tác giả: *"Mở lại slot đang có lịch hẹn sẽ khiến slots.status lệch với appointments…"*
- **Kiểm chứng:** `Test` :152–161, :163–176 · `Probe` (trạng thái sai)

### HT-BR-16 — Staff có thể chặn giờ
Staff đặt slot còn trống sang *đã đặt* mà không có lịch hẹn nào, để mô phỏng bác sĩ bận. Bệnh nhân không đặt được slot này nữa.
- **Bằng chứng:** `slotService.js:28–49`, `staff.js:101–104` (nút "Chặn giờ"), `doc/prod.md` §6 ("mô phỏng điều chỉnh lịch làm việc")
- **Kiểm chứng:** `Test` :191–200

---

## Ràng buộc chỉ nằm ở giao diện

Những điều dưới đây giao diện làm, **backend cũng chặn độc lập** nên không phải lỗ hổng, nhưng cần biết khi đổi giao diện.

| Ràng buộc ở giao diện | Nơi | Backend có chặn không |
| --- | --- | --- |
| Ẩn nút Hủy với lịch đã hủy | `patient.js:135–137`, `staff.js:51–53` | Có — HT-BR-09 |
| Chỉ hiện nút Xác nhận khi lịch đang chờ | `staff.js:46–48` | Có — HT-BR-10 |
| Bệnh nhân chỉ chọn được slot từ danh sách còn trống | `patient.js:63–95` | Có — HT-BR-05 |
| Hình thức khám chỉ có hai lựa chọn | `index.html:182` | **Không** — HT-BR-06 nhận mọi giá trị và đổi thành trực tiếp |

## Quy ước phản hồi (áp dụng cho mọi quy tắc)

Thành công `{ "data": … }`, tạo mới `201`, còn lại `200`. Lỗi `{ "error": "<tiếng Việt>" }`. Xem `05-api.md` §1 cho bảng mã lỗi.
