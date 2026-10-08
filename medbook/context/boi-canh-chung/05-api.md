# Hiện trạng · 05 — API

> **Mã:** CTX-HT-05 · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · suy ngược từ code · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** `server.js`, `src/routes/*.js`, `src/services/*.js`, `src/repositories/*.js`; hành vi lỗi đã chạy thật trên bản tách biệt (2026-09-28)

**16 endpoint** (15 dưới `/api` và `/health`). JSON cả hai chiều.

---

## 1. Quy ước chung

**Phản hồi:** thành công `{ "data": … }`; lỗi `{ "error": "<thông báo>" }`. Tạo mới `201`, còn lại `200`.

**Khai báo người dùng:** header `X-Demo-User-Id: <mã người dùng>`. Không có token hay phiên phía máy chủ (HT-BR-02).

**Bảng mã lỗi đang dùng**

| HTTP | Khi nào | Thông báo |
| :---: | --- | --- |
| 400 | Thiếu trường bắt buộc | `Thiếu <tên trường>` (vd. `Thiếu slotId`) |
| 400 | Đăng nhập thiếu email hoặc mật khẩu | `Vui lòng nhập email và mật khẩu` |
| 400 | Trạng thái slot ngoài `available`/`booked` | `Trạng thái slot không hợp lệ` |
| 400 | JSON gửi lên sai cú pháp | Thông báo **tiếng Anh** của bộ phân tích, vd. `Expected property name or '}' in JSON at position 1` |
| 401 | Đăng nhập sai | `Sai email hoặc mật khẩu demo` |
| 401 | Thiếu / sai / không tìm thấy header người dùng | `Thiếu tài khoản demo` · `Tài khoản demo không hợp lệ` · `Không tìm thấy tài khoản demo` |
| 403 | Sai vai trò, hoặc hủy lịch của người khác | `Không đủ quyền` |
| 404 | Không tìm thấy | `Không tìm thấy khung giờ` · `Không tìm thấy lịch hẹn` · `Không tìm thấy API` |
| 409 | Xung đột trạng thái | `Khung giờ đã được đặt` · `Lịch hẹn đã bị hủy` · `Chỉ xác nhận được lịch đang chờ xác nhận` · `Không thể mở lại khung giờ đang có lịch hẹn` |
| 500 | Lỗi không lường trước, **kể cả dữ liệu đầu vào sai định dạng** | **Nguyên văn thông báo lỗi của PostgreSQL** (vd. `invalid input syntax for type date: "abc"`) — *không phải* "Lỗi máy chủ". Xem HT-GAP-06 |

**Ngoài `/api`:** `GET` bất kỳ đường dẫn nào không khớp trả `public/index.html` (trang đơn) — `server.js:34–36`. `/api/<không tồn tại>` ⇒ 404 JSON.

## 2. Hình dạng đối tượng

**Người dùng** `{ id, name, email, role, patientId }` — `patientId` rỗng với staff.

**Bác sĩ** `{ id, name, title, room, specializationId, specialization, availableSlots }` — `availableSlots` là **chuỗi** (vd. `"3"`) vì `count` của PostgreSQL trả kiểu số lớn; giao diện ép bằng `Number()` (HT-GAP-16).

**Slot** `{ id, doctorId, doctorName, title, room, specializationId, specialization, date, startTime, endTime, status }` — `date` dạng `YYYY-MM-DD`, giờ dạng `HH:MM`, `status` là `available` hoặc `booked`.

**Lịch hẹn** `{ id, patientId, patientName, patientPhone, slotId, status, type, createdAt, doctorId, doctorName, specialization, room, date, startTime, endTime }` — `createdAt` dạng `YYYY-MM-DD HH:MM`.

---

## 3. Đăng nhập demo

### HT-API-01 · `GET /health`
Không cần đăng nhập. Chạy `select 1`; thành công ⇒ `{ "ok": true, "service": "medbook" }`. DB lỗi ⇒ 500. *(`server.js:17–24`; `Test` :47–51.)*

### HT-API-02 · `GET /api/demo-users`
Không cần đăng nhập. Trả **mọi** tài khoản (8), sắp theo vai trò rồi tên, không có mật khẩu. Dùng cho ô "chọn nhanh tài khoản" ở màn hình đăng nhập. *(`auth.routes.js:7–13`; `Probe`.)*

### HT-API-03 · `POST /api/demo-login`
Không cần đăng nhập. Body `{ email, password }`. Thành công ⇒ `{ data: { user } }`. Lỗi: 400 thiếu trường; 401 sai. *(HT-BR-01; `Test` :135–150.)*

### HT-API-04 · `GET /api/me`
Cần header. Trả người dùng ứng với header. *(HT-BR-02; `Test` :53–57.)*

## 4. Bác sĩ và khung giờ

### HT-API-05 · `GET /api/specializations`
Cần đăng nhập (mọi vai trò). Trả `[{ id, name }]` sắp theo tên. *(`doctorRepository.js:3–5`.)*

### HT-API-06 · `GET /api/doctors?specializationId=&q=`
Cần đăng nhập. Cả hai tham số không bắt buộc. `specializationId` không đọc ra số thì bị bỏ qua. `q` được cắt khoảng trắng, khớp `ilike '%q%'` trên **tên** bác sĩ; ký tự `%` và `_` **không được thoát** nên `q=%` khớp mọi bác sĩ (HT-GAP-17). *(HT-BR-13; `Probe`.)*

### HT-API-07 · `GET /api/doctors/:id/slots?date=`
Cần đăng nhập. Slot **còn trống**, từ hôm nay, của bác sĩ; `date` không bắt buộc. `:id` không đọc ra số ⇒ trả mảng rỗng. `date` sai định dạng ⇒ 500. *(HT-BR-12; `Test` :98–102.)*

### HT-API-08 · `GET /api/slots/available`
Cần đăng nhập. Mọi slot còn trống từ hôm nay, của mọi bác sĩ. *(HT-BR-12; `Probe`. **Giao diện hiện tại không gọi endpoint này.**)*

### HT-API-09 · `GET /api/slots?date=` — **staff**
Slot từ hôm nay của mọi bác sĩ, cả trống lẫn đã đặt. `date` sai định dạng ⇒ 500 (`Probe`). *(HT-BR-03.)*

### HT-API-10 · `POST /api/slots` — **staff**, `201`
Body `{ doctorId, date, startTime, endTime }`, cả bốn bắt buộc. Slot luôn tạo ở trạng thái còn trống. **Không kiểm tra giá trị:** `date` sai ⇒ 500; `doctorId` không tồn tại ⇒ 500 (vi phạm khóa ngoại, lộ tên ràng buộc); giờ kết thúc trước giờ bắt đầu ⇒ vẫn tạo được (`Probe`). *(HT-BR-14; `Test` :116–133.)*

### HT-API-11 · `PUT /api/slots/:id` — **staff**
Body tùy chọn `{ startTime, endTime, status }`; trường thiếu thì giữ nguyên. Lỗi: 400 trạng thái sai; 409 mở lại slot đang có lịch; 404 không có slot (kể cả `:id` không đọc ra số); 500 giờ sai (vd. `25:99`). *(HT-BR-15, 16; `Test` :152–200.)*
⚠️ Route ghép `{ id: req.params.id, ...req.body }` (`doctors.routes.js:68`) nên **`id` trong body ghi đè `id` trên đường dẫn**: `PUT /api/slots/3` với body `{"id":4,…}` sửa slot 4 (`Probe`, HT-GAP-07).

## 5. Lịch hẹn

### HT-API-12 · `POST /api/appointments` — **patient**, `201`
Body `{ slotId, type }`; `type` không bắt buộc. `patientId` lấy từ người dùng đăng nhập, không nhận từ body. Trả lịch hẹn đầy đủ. Lỗi: 400 thiếu `slotId`; 404 slot không có; 409 slot đã đặt; 403 nếu là staff. `type` khác `"online"` (kể cả giá trị lạ) ⇒ `in_person`. *(HT-BR-05 → 08; `Test` :67–87, :178–189.)*

### HT-API-13 · `GET /api/my-appointments` — **patient**
Lịch của bệnh nhân đang đăng nhập, sắp theo ngày và giờ **giảm dần**. *(HT-BR-11.)*

### HT-API-14 · `GET /api/appointments?date=` — **staff**
Mọi lịch hẹn, lọc theo ngày (tùy chọn), sắp theo ngày và giờ giảm dần. Không phân trang. *(HT-BR-11; `Test` :59–65 cho 403.)*

### HT-API-15 · `POST /api/appointments/:id/confirm` — **staff**
Đổi `booked → confirmed`. Trả lịch hẹn đã cập nhật. Mọi trường hợp không đổi được (đã xác nhận, đã hủy, **không tồn tại**) ⇒ 409 `Chỉ xác nhận được lịch đang chờ xác nhận`. *(HT-BR-10; `Test` :105–114.)*

### HT-API-16 · `POST /api/appointments/:id/cancel` — **patient hoặc staff**
Route **không có** `requireRole`; kiểm quyền trong service. Lỗi: 404 không thấy; 403 bệnh nhân hủy lịch người khác; 409 đã hủy. Thành công trả lịch hẹn với `status: "cancelled"`. *(HT-BR-09; `Test` :89–103, :163–176.)*

---

## 6. Bảng tổng hợp

| ID | Method | Đường dẫn | Ai gọi được | Ghi |
| --- | --- | --- | --- | --- |
| HT-API-01 | GET | `/health` | mọi người | |
| HT-API-02 | GET | `/api/demo-users` | mọi người | |
| HT-API-03 | POST | `/api/demo-login` | mọi người | |
| HT-API-04 | GET | `/api/me` | đăng nhập | |
| HT-API-05 | GET | `/api/specializations` | đăng nhập | |
| HT-API-06 | GET | `/api/doctors` | đăng nhập | |
| HT-API-07 | GET | `/api/doctors/:id/slots` | đăng nhập | |
| HT-API-08 | GET | `/api/slots/available` | đăng nhập | giao diện không dùng |
| HT-API-09 | GET | `/api/slots` | staff | |
| HT-API-10 | POST | `/api/slots` | staff | 201 |
| HT-API-11 | PUT | `/api/slots/:id` | staff | ⚠️ id trong body ghi đè |
| HT-API-12 | POST | `/api/appointments` | patient | 201 |
| HT-API-13 | GET | `/api/my-appointments` | patient | |
| HT-API-14 | GET | `/api/appointments` | staff | |
| HT-API-15 | POST | `/api/appointments/:id/confirm` | staff | |
| HT-API-16 | POST | `/api/appointments/:id/cancel` | patient, staff | kiểm quyền trong service |

**Đã có test:** 10 trên 16 endpoint có ít nhất một test chạm tới — HT-API-01, 03, 04, 07, 10, 11, 12, 15, 16, và HT-API-14 (chỉ ca bị từ chối 403). **Chưa có test nào:** HT-API-02, 05, 06, 08, 09, 13. Chi tiết ở `09-truy-vet.md` §3.
