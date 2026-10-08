# Hiện trạng · 01 — Phạm vi và bối cảnh

> **Mã:** CTX-HT-01 · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · suy ngược từ code · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** `source-code/` (toàn bộ), `doc/prod.md`, ứng dụng đang chạy · **Cách đọc bằng chứng:** `README.md` §2

Tài liệu này mô tả **MedBook đúng như nó là ở đầu Day 1**, trước khi có bất kỳ tài liệu hay code nào của tính năng danh sách chờ. Mọi câu đều suy ra từ code hoặc từ việc chạy thật; chỗ nào chỉ là suy đoán được ghi rõ.

---

## 1. Hệ thống làm gì

MedBook là ứng dụng web đặt lịch khám. Bệnh nhân tìm bác sĩ, xem khung giờ còn trống và đặt lịch; nhân viên điều phối xem lịch hẹn, xác nhận hoặc hủy, và quản lý khung giờ làm việc của bác sĩ. *(Nguồn: `Code` các route; `UI` `public/index.html`; `Doc` `doc/prod.md` §1–§3.)*

## 2. Người dùng

| Actor | Trong hệ thống | Đăng nhập | Việc chính |
| --- | --- | :---: | --- |
| **Patient** (bệnh nhân) | `users.role = 'patient'`, gắn với một hồ sơ `patients` | Có | Tìm bác sĩ · xem giờ trống · đặt lịch · xem và hủy lịch của mình |
| **Staff** (nhân viên) | `users.role = 'staff'`, không gắn hồ sơ bệnh nhân | Có | Xem lịch hẹn theo ngày · xác nhận · hủy · thêm slot · chặn / mở lại slot |
| **Doctor** (bác sĩ) | Chỉ là **dữ liệu** trong bảng `doctors` | **Không** | Không thao tác gì trong hệ thống |

Hệ thống chỉ có **hai vai trò**. Không có quản trị viên, không có vai trò bác sĩ. *(`Code` `migrate.js:16`, CHECK `role in ('patient','staff')`.)*

## 3. Chức năng có thật

**Bệnh nhân**

| # | Chức năng | Nơi thực hiện |
| ---: | --- | --- |
| 1 | Đăng nhập demo bằng email và mật khẩu | `authService.login` |
| 2 | Xem danh sách chuyên khoa | `GET /api/specializations` |
| 3 | Tìm bác sĩ theo chuyên khoa và/hoặc tên, thấy số giờ trống | `GET /api/doctors` |
| 4 | Xem khung giờ còn trống của một bác sĩ, lọc theo ngày | `GET /api/doctors/:id/slots` |
| 5 | Đặt lịch ("Giữ chỗ") với hình thức trực tiếp hoặc online | `POST /api/appointments` |
| 6 | Xem "Lịch của tôi" và trạng thái từng lịch | `GET /api/my-appointments` |
| 7 | Hủy lịch của chính mình | `POST /api/appointments/:id/cancel` |
| 8 | Xem tổng quan: số bác sĩ hiển thị · số lịch của tôi (không tính lịch đã hủy) · số khung giờ trống | `UI` `patient.js:29–34, 110–113` |

**Nhân viên**

| # | Chức năng | Nơi thực hiện |
| ---: | --- | --- |
| 9 | Đăng nhập demo | như trên |
| 10 | Xem mọi lịch hẹn, lọc theo ngày | `GET /api/appointments` |
| 11 | Xác nhận lịch đang chờ | `POST /api/appointments/:id/confirm` |
| 12 | Hủy lịch của bất kỳ bệnh nhân | `POST /api/appointments/:id/cancel` |
| 13 | Xem slot sắp tới của mọi bác sĩ, lọc theo ngày | `GET /api/slots` |
| 14 | Thêm slot mới | `POST /api/slots` |
| 15 | Chặn giờ (đặt `booked`) hoặc mở lại (đặt `available`) một slot | `PUT /api/slots/:id` |
| 16 | Xem tổng quan: số lịch trong ngày · số slot trống · số bác sĩ | `UI` `staff.js` |

**Dùng chung:** đổi giao diện sáng / tối (lưu ở trình duyệt) · giữ phiên qua F5 (lưu mã người dùng ở trình duyệt).

## 4. Những gì hệ thống **không** có

Quan trọng cho mọi tính năng xây thêm. Mỗi mục đã kiểm bằng cách tìm trong code (không có route, service, bảng hay màn hình tương ứng).

| Không có | Ghi chú |
| --- | --- |
| **Danh sách chờ**, đề xuất khung giờ, hết hạn tự động | Tính năng cần xây (Day 3) |
| **Đổi lịch** (dời một lịch hẹn sang slot khác) | Không có endpoint; chỉ hủy rồi đặt lại |
| **Thông báo** dưới mọi hình thức (in-app, email, SMS) | Không có bảng, không có endpoint |
| Quản trị viên (Admin) | Chỉ `patient` và `staff` |
| Đăng nhập cho bác sĩ | Bác sĩ là dữ liệu |
| Quản lý bệnh nhân / bác sĩ (thêm, sửa, xóa) | Chỉ đọc; dữ liệu do seed nạp |
| Đăng ký tài khoản, đổi mật khẩu | Ngoài phạm vi (`doc/prod.md` §3) |
| Nhật ký thao tác (audit) | Không bảng nào ghi ai làm gì, lúc nào |
| Tác vụ nền / lập lịch | Không có `setInterval`, cron hay hàng đợi |
| Phân trang, giới hạn tần suất, CORS, bảo mật header | Không cấu hình |
| Múi giờ | `date` và giờ hiểu theo giờ máy chủ |

## 5. Công nghệ và cách chạy

| Thành phần | Giá trị | Nguồn |
| --- | --- | --- |
| Runtime | Node.js 20 (image `node:20-alpine`) | `Dockerfile:1` |
| Web framework | Express 4 (`^4.19.2`) | `package.json` |
| Cơ sở dữ liệu | PostgreSQL 16 (`pg ^8.12.0`), SQL thuần, không ORM | `package.json`, `docker-compose.yml` |
| Giao diện | HTML + CSS + JavaScript thuần (ES module), không build | `public/` |
| Kiểm thử | `node --test` + `node:assert`, test tích hợp thật, không mock | `tests/api.test.js` |
| Lint | ESLint 8 (`eslint:recommended`, `semi: always`) | `.eslintrc.json` |
| **Dependency runtime** | **Chỉ 2:** `express`, `pg` | `package.json` |

**Chạy:** `docker compose up --build` → ứng dụng `http://localhost:4300`, PostgreSQL `localhost:55432`. Ứng dụng đợi DB tối đa 30 giây, chạy migration, nạp dữ liệu mẫu rồi mới lắng nghe cổng. *(`Code` `server.js:49–55`, `pool.js:13–23`.)*

## 6. Màn hình

| Màn hình | Nội dung | Nguồn |
| --- | --- | --- |
| Đăng nhập | Ô chọn nhanh tài khoản demo (nhóm theo vai trò), email, mật khẩu (điền sẵn `demo123`) | `login.js`, `index.html:22–101` |
| Bệnh nhân — Tổng quan | Ba chỉ số | `index.html:152–154` |
| Bệnh nhân — Tìm bác sĩ | Lọc chuyên khoa, ô tìm tên, danh sách bác sĩ với số giờ trống và nút "Xem giờ" | `patient.js:21–61` |
| Bệnh nhân — Khung giờ | Chọn ngày, danh sách theo trục thời gian với vạch "Bây giờ", nút "Giữ chỗ", chọn hình thức khám | `patient.js:63–104`, `ui.js` |
| Bệnh nhân — Lịch của tôi | Danh sách lịch kèm nhãn trạng thái và nút Hủy (ẩn với lịch đã hủy) | `patient.js:106–151` |
| Nhân viên — Tổng quan | Ba chỉ số | `index.html:206–208` |
| Nhân viên — Lịch hẹn | Theo ngày; nút Xác nhận (chỉ khi `booked`), nút Hủy (ẩn khi đã hủy) | `staff.js:16–62` |
| Nhân viên — Lịch làm việc | Danh sách slot; nút "Chặn giờ" / "Mở lại"; biểu mẫu thêm slot | `staff.js:76–134` |

Thông báo lỗi và thành công hiện dưới dạng toast 3 giây. *(`api.js:26–32`.)*

## 7. Dữ liệu mẫu ở đầu Day 1

5 bệnh nhân · 8 tài khoản (5 bệnh nhân + 3 nhân viên, mật khẩu `demo123`) · 5 chuyên khoa · 6 bác sĩ (hai người cùng chuyên khoa Tim mạch) · 12 slot với ngày tương đối (hôm nay, ngày mai, ngày kia) · 2 lịch hẹn mẫu: lịch 1 của An ở slot 2 (`booked`, trực tiếp), lịch 2 của Linh ở slot 7 (`confirmed`, online). *(`Code` `seed.js`.)*
