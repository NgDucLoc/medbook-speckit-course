# Hiện trạng · 06 — Thành phần và luồng xử lý

> **Mã:** CTX-HT-06 · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · suy ngược từ code · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** `server.js`, `src/**`, `public/**`; đồ thị `require` đọc từ đầu mỗi file

---

## 1. Kiến trúc

```
Trình duyệt (HTML / CSS / JS thuần, ES module)
        │  HTTP + JSON, header X-Demo-User-Id
        ▼
   Express (server.js)
        │
   Routes ──► Services ──► Repositories ──► Pool ──► PostgreSQL 16
```

Ý định phân lớp (ADR-006): route nhận / trả HTTP, service chứa nghiệp vụ và giao dịch, repository chứa SQL. **Thực tế** có ngoại lệ ghi ở §4.

## 2. Thành phần backend

| ID | File | Trách nhiệm | Hàm / điểm chính |
| --- | --- | --- | --- |
| **HT-CMP-01** | `server.js` | Dựng Express, gắn middleware (JSON, file tĩnh), `/health`, ba router dưới `/api`, 404 cho `/api`, trang đơn cho GET còn lại, **bộ xử lý lỗi tập trung**, trình tự khởi động | `start()` = đợi DB → migrate → seed → lắng nghe. Xuất `app`, `pool`, `migrateAndSeed`, `waitForDatabase` cho test |
| **HT-CMP-02** | `db/pool.js` | Kết nối PostgreSQL | `query(sql, params)` → mảng dòng · `getClient()` cho giao dịch · `waitForDatabase()` thử 30 lần cách 1 giây. Địa chỉ mặc định `postgres://medbook:medbook@localhost:55432/medbook` |
| **HT-CMP-03** | `db/migrate.js` | Tạo bảng, chỉ mục nếu chưa có | `migrate()` — một câu lệnh nhiều phần, idempotent |
| **HT-CMP-04** | `db/seed.js` | Nạp dữ liệu mẫu | `seed({reset})`, `resetData()` — xem `04-du-lieu.md` §6 |
| **HT-CMP-05** | `errors.js` | Tạo lỗi có mã HTTP | `httpError(status, message)` |
| **HT-CMP-06** | `utils/validate.js` | Chuẩn hóa đầu vào | `toInt(v, fallback)` (dùng `parseInt`, nên `"12abc"` → 12) · `required(v, name)` → 400 |
| **HT-CMP-07** | `middleware/demoAuth.js` | Đọc header, gắn `req.user` | gọi `authService.authenticateByHeader` |
| **HT-CMP-08** | `middleware/requireRole.js` | Chặn theo vai trò | trả **trực tiếp** `403 { error }`, không qua `next(error)` |
| **HT-CMP-09** | `routes/auth.routes.js` | 3 endpoint đăng nhập | HT-API-02, 03, 04 |
| **HT-CMP-10** | `routes/doctors.routes.js` | 7 endpoint bác sĩ và slot | HT-API-05 → 11 |
| **HT-CMP-11** | `routes/appointments.routes.js` | 5 endpoint lịch hẹn | HT-API-12 → 16 |
| **HT-CMP-12** | `services/authService.js` | Đăng nhập, xác thực theo header | `listDemoUsers`, `login`, `authenticateByHeader` (HT-BR-01, 02, 04) |
| **HT-CMP-13** | `services/slotService.js` | Nghiệp vụ slot của staff | `listManageableSlots`, `createSlot`, `updateSlot` (HT-BR-14, 15, 16). **Không dùng giao dịch** |
| **HT-CMP-14** | `services/appointmentService.js` | Nghiệp vụ lịch hẹn | `bookAppointment`, `cancelAppointment` (giao dịch, có khóa dòng), `confirmAppointment`, `listMyAppointments`, `listAppointmentsForStaff` (HT-BR-05 → 11) |
| **HT-CMP-15** | `repositories/userRepository.js` | SQL bảng `users` | `listDemoUsers`, `findPublicById`, `findForLoginByEmail` |
| **HT-CMP-16** | `repositories/doctorRepository.js` | SQL bác sĩ, chuyên khoa | `listSpecializations`, `searchDoctors` |
| **HT-CMP-17** | `repositories/slotRepository.js` | SQL slot | `slotSelect(where)` (câu chọn dùng chung, đổi tên cột sang camelCase bằng alias), `listAvailable`, `listAllUpcoming`, `listAvailableByDoctor`, `createSlot`, `findDetailedById`, `findForUpdate(client, id)`, `updateStatus(client, id, status)`, `updateSlot` |
| **HT-CMP-18** | `repositories/appointmentRepository.js` | SQL lịch hẹn | `appointmentSelect(where)`, `findDetailedById`, `listByPatient`, `listForStaff`, `create(client, …)`, `findForUpdate(client, id)`, `updateStatus(client, …)`, `countActiveBySlot`, `confirmBooked` (câu lệnh có điều kiện) |

**Mẫu dùng lại được** cho code mới: giao dịch `begin → khóa dòng → kiểm tra → ghi → commit / rollback → release` (`appointmentService.js:14–33`); câu lệnh có điều kiện `update … where status = … returning id` (`appointmentRepository.js:82–88`); khai báo cột trả về bằng alias camelCase trong một hàm dựng câu (`slotSelect`, `appointmentSelect`).

## 3. Thành phần frontend

| ID | File | Trách nhiệm |
| --- | --- | --- |
| **HT-CMP-19** | `public/index.html`, `styles.css` | Khung trang: màn hình đăng nhập, khung ứng dụng, khu vực bệnh nhân và staff, toast; giao diện sáng / tối bằng thuộc tính `data-theme` |
| **HT-CMP-20** | `js/main.js` | Khởi động, gắn sự kiện, điều hướng theo vai trò, `loadAll`, đổi theme; mọi handler qua `guard()` để lỗi API thành toast |
| **HT-CMP-21** | `js/state.js` | Trạng thái dùng chung (`user`, `users`, `doctors`, `selectedDoctorId`) và bảng nhãn tiếng Việt |
| **HT-CMP-22** | `js/api.js` | `api(path, options)` — tự gắn `X-Demo-User-Id`, ném `Error(payload.error)`; `today()` theo giờ máy người dùng; `escapeHtml`; `toast` |
| **HT-CMP-23** | `js/ui.js` | "Time rail": vẽ mọi danh sách theo ngày, cột giờ thẳng hàng, vạch "Bây giờ"; nhãn trạng thái slot / lịch hẹn; `onClick` bọc lỗi |
| **HT-CMP-24** | `js/views/login.js` | Đăng nhập, khôi phục phiên, đăng xuất |
| **HT-CMP-25** | `js/views/patient.js` | Tìm bác sĩ, xem giờ, đặt lịch, lịch của tôi, hủy |
| **HT-CMP-26** | `js/views/staff.js` | Lịch hẹn theo ngày, xác nhận, hủy, danh sách slot, thêm slot, chặn / mở lại |

**Lưu ở trình duyệt:** `medbookUserId` (mã người dùng để giữ phiên), `medbookTheme` (sáng / tối). **Chống chèn HTML:** mọi giá trị đưa vào giao diện đi qua `escapeHtml`.

**Điểm kết nối cho tính năng mới:** sau mỗi thao tác đặt / hủy / xác nhận, giao diện phát sự kiện `medbook:reload` để làm mới tất cả danh sách (`main.js:139`).

## 4. Đồ thị phụ thuộc

```
server.js ─► routes/auth ────► services/authService ──► repositories/userRepository
          │                └─► middleware/demoAuth ──► services/authService
          ├► routes/doctors ─► repositories/doctorRepository        ← route gọi repository trực tiếp
          │                 ├► repositories/slotRepository          ← route gọi repository trực tiếp
          │                 └► services/slotService ─► slotRepository, appointmentRepository
          └► routes/appointments ─► services/appointmentService ─► appointmentRepository, slotRepository, db/pool.getClient
```

**Quan sát:**
- Service **không** gọi service khác. Hai service dùng chung repository (`slotService` dùng cả `appointmentRepository`).
- **Route gọi repository trực tiếp** cho 4 endpoint chỉ đọc (`doctors.routes.js:13, 22, 35, 44`) — lệch khỏi mô tả phân lớp "route → service → repository" ở `doc/prod.md` §8; xem HT-GAP-20.
- Chỉ `appointmentService` dùng giao dịch. `slotService.updateSlot` kiểm tra rồi ghi **không** giao dịch (HT-GAP-10).

## 5. Luồng xử lý chính

### F-1 · Đặt lịch (`POST /api/appointments`)
| Bước | Thành phần | Việc |
| ---: | --- | --- |
| 1 | `demoAuth`, `requireRole('patient')` | Xác thực, kiểm vai trò |
| 2 | Route | Gom `slotId`, `type` từ body và `patientId` từ người dùng |
| 3 | `bookAppointment` | `toInt(required(slotId))`; `type` → online hoặc in_person |
| 4 | `getClient` + `begin` | Mở giao dịch |
| 5 | `slotRepository.findForUpdate` | **Khóa dòng slot** (`select … for update`) |
| 6 | | Không có ⇒ 404; `status ≠ available` ⇒ 409 |
| 7 | `slotRepository.updateStatus` → `booked` | |
| 8 | `appointmentRepository.create` | Thêm lịch `booked` |
| 9 | `commit` (lỗi ⇒ `rollback`); `release` | |
| 10 | `findDetailedById` **sau** giao dịch | Đọc chi tiết để trả về |

Thứ tự khóa: **slot** trước, rồi ghi lịch hẹn.

### F-2 · Hủy lịch (`POST /api/appointments/:id/cancel`)
Xác thực (mọi vai trò) → `begin` → **khóa dòng lịch hẹn** → không có ⇒ 404 → bệnh nhân mà không phải chủ ⇒ 403 → đã hủy ⇒ 409 → lịch → `cancelled` → **slot → `available`** → `commit` → đọc chi tiết. Thứ tự khóa: **lịch hẹn** trước, rồi slot (ngược thứ tự với F-1).

### F-3 · Xác nhận lịch
Xác thực → `requireRole('staff')` → `update appointments set status='confirmed' where id=$1 and status='booked' returning id` → không dòng ⇒ 409 → đọc chi tiết.

### F-4 · Sửa slot (`PUT /api/slots/:id`)
Xác thực → staff → chuẩn hóa trạng thái (sai ⇒ 400) → **nếu đặt `available`**: đếm lịch hoạt động của slot, `> 0` ⇒ 409 → `update slots set … coalesce(…)` → không dòng ⇒ 404. Hai bước kiểm tra và ghi **không** nằm trong một giao dịch.

### F-5 · Đăng nhập và khôi phục phiên
Đăng nhập: `POST /api/demo-login` → lưu `medbookUserId`. Tải lại trang: đọc `medbookUserId` → `GET /api/me` → thành công thì vào ứng dụng, thất bại thì xóa và về màn hình đăng nhập.

### F-6 · Khởi động
`waitForDatabase` (tối đa 30 giây) → `migrate` → `seed` (**không** reset) → `listen`. Bất kỳ bước nào ném lỗi ⇒ ghi log và `process.exit(1)` (`server.js:57–62`). Docker Compose **không** đặt chính sách tự khởi động lại cho `app`.

## 6. Kiểm thử, CI, đóng gói

| ID | File | Nội dung |
| --- | --- | --- |
| **HT-CMP-27** | `tests/api.test.js` | 14 test tích hợp. Trước mỗi test chạy `migrateAndSeed({reset: true})` — **xóa sạch và nạp lại** dữ liệu. Server lắng nghe cổng ngẫu nhiên. Mẫu tham chiếu: test đặt đồng thời (:178–189) |
| **HT-CMP-28** | `.github/workflows/ci.yml`, `Dockerfile`, `docker-compose.yml` | CI: PostgreSQL 16 dịch vụ → `npm ci` → lint → migrate + seed → test, trên DB `medbook_test`. Docker: `node:20-alpine`, `npm install --omit=dev`, `npm start`; compose thêm PostgreSQL 16 có kiểm tra sức khỏe, app chờ DB khỏe mới chạy |
