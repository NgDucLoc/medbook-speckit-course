# Hiện trạng · 07 — Phi chức năng và cấu hình

> **Mã:** CTX-HT-07 · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · suy ngược từ code · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** `server.js`, `pool.js`, `.env.example`, `docker-compose.yml`, `Dockerfile`, `ci.yml`, `.eslintrc.json`, `doc/adr/`; múi giờ và cấu hình kiểm trên container đang chạy

Mọi mục dưới đây là **thuộc tính đang có**, không phải yêu cầu đã đặt ra. Không có tài liệu nào của dự án nêu chỉ tiêu phi chức năng (thời gian phản hồi, số người dùng, độ sẵn sàng).

---

## 1. Cấu hình

| Biến | Nơi đặt | Giá trị mặc định trong code | **Code có đọc không** |
| --- | --- | --- | :---: |
| `PORT` | compose, CI, `.env.example` | `4300` (`server.js:10`) | ✅ |
| `DATABASE_URL` | compose, CI, `.env.example` | `postgres://medbook:medbook@localhost:55432/medbook` (`pool.js:3–4`) | ✅ |
| `DEMO_AUTH_ENABLED` | compose, CI, `.env.example` | — | ❌ **Không nơi nào trong code đọc** |
| `NODE_ENV` | CI, `.env.example` | — | ❌ **Không nơi nào trong code đọc** |

Hệ quả: đặt `DEMO_AUTH_ENABLED=false` **không tắt** được đăng nhập demo (HT-GAP-13). Địa chỉ DB mặc định trỏ tới **cơ sở dữ liệu phát triển** ở cổng 55432 — chạy `npm test` mà không đặt `DATABASE_URL` sẽ xóa dữ liệu ở đó (HT-GAP-14).

`.env` thật bị `.gitignore` loại; chỉ `.env.example` được commit.

## 2. Khởi động và vận hành

| Khía cạnh | Hiện trạng |
| --- | --- |
| Trình tự | Đợi DB tối đa 30 lần × 1 giây → migrate → seed (không reset) → lắng nghe. Lỗi ở bước nào cũng thoát tiến trình mã 1 |
| Kiểm tra sức khỏe | `GET /health` chạy `select 1`. Compose chỉ kiểm sức khỏe của **DB** (`pg_isready`), không kiểm `app` |
| Tự khởi động lại | Không có chính sách `restart` cho `app` trong compose |
| Tắt máy êm | Không xử lý tín hiệu tắt; pool kết nối không được đóng chủ động |
| Log | `console.log` một dòng khi lên; `console.error` cho lỗi ≥ 500 (`server.js:40`). **Không** ghi log từng request |
| Giám sát, số đo | Không có |
| Seed chạy mỗi lần khởi động | Có — và ghi đè dữ liệu mẫu (HT-GAP-01, 02) |

## 3. Bảo mật

| Khía cạnh | Hiện trạng | Nguồn |
| --- | --- | --- |
| Xác thực | Header `X-Demo-User-Id` chứa mã người dùng; **giả mạo được bằng một dòng `curl`**. Phiên phía trình duyệt là mã người dùng trong `localStorage` | ADR-002 (chủ ý), `api.js:39` |
| Mật khẩu | Lưu chữ thường (`demo_password`), so khớp trực tiếp; mặc định `demo123` điền sẵn ở ô đăng nhập | `authService.js:24`, `index.html:98` |
| Phân quyền | Kiểm vai trò bằng `requireRole`; kiểm quyền sở hữu lịch hẹn ở service (chỉ cho việc hủy) | HT-BR-03, 09 |
| SQL injection | Mọi truy vấn dùng tham số (`$1, $2 …`); phần `where` động chỉ ghép **chuỗi hằng** của code, không ghép dữ liệu người dùng | `*Repository.js` |
| XSS | Giao diện đi qua `escapeHtml` cho mọi giá trị đưa vào HTML, kể cả thuộc tính `data-*` | `api.js:12–19` |
| Lộ thông tin lỗi | Lỗi 500 trả **nguyên văn thông báo của PostgreSQL** | HT-GAP-06 |
| Header bảo mật, CORS, giới hạn tần suất, CSRF | Không cấu hình. Giao diện và API cùng nguồn; xác thực bằng header tùy biến nên không có cookie để bị CSRF | — |
| Bí mật trong repo | Thông tin đăng nhập DB (`medbook/medbook`) ghi thẳng trong compose, CI, `pool.js` — chấp nhận được cho bản demo | — |

## 4. Đồng thời và nhất quán dữ liệu

| Cơ chế | Dùng ở đâu | Nhận xét |
| --- | --- | --- |
| Khóa dòng `select … for update` | Đặt lịch (khóa slot), hủy lịch (khóa lịch hẹn) | Có |
| Chỉ mục duy nhất từng phần | `one_active_appointment_per_slot` | Lớp cuối cùng chống đặt trùng |
| Câu lệnh có điều kiện | Xác nhận lịch | Không khe hở giữa đọc và ghi |
| **Không có** giao dịch / khóa | Sửa slot (`updateSlot`): kiểm tra lịch hoạt động rồi ghi ở hai bước riêng | Có khe hở (HT-GAP-10) |
| Thứ tự khóa | Đặt lịch: slot → lịch hẹn. Hủy lịch: lịch hẹn → slot | Ngược nhau; chưa thấy kịch bản bế tắc vì đặt lịch dừng khi slot đã đặt, nhưng cần lưu ý khi thêm luồng mới |
| Dữ liệu dẫn xuất được lưu riêng | `slots.status` lưu riêng, không tính từ lịch hẹn (ADR-003) | Có thể lệch nếu có đường ghi bỏ qua giao dịch |

**Kích thước pool kết nối:** mặc định của thư viện `pg` (không cấu hình).

## 5. Thời gian và múi giờ

- `slots.date` và giờ lưu **tách rời**, không kèm múi giờ; hiểu theo "giờ máy chủ". `appointments.created_at` là `timestamptz`.
- PostgreSQL và ứng dụng trong Docker chạy **UTC** (kiểm bằng `show timezone`). `current_date` trong các truy vấn (lọc slot từ hôm nay, dữ liệu mẫu) là **ngày UTC**.
- Giao diện tính "hôm nay" theo **giờ máy người dùng** (`api.js:6–10`), có chủ ý để tránh lệch do `toISOString`.
- **Hệ quả:** với người dùng ở UTC+7, từ 00:00 đến 07:00 giờ địa phương ngày UTC còn là hôm qua — danh sách khung giờ "từ hôm nay" của máy chủ và ô ngày mặc định của giao diện lệch nhau một ngày (HT-GAP-23).
- Không có kiểm tra "slot đã qua giờ" ở bất kỳ đâu.

## 6. Hiệu năng và quy mô

Không có yêu cầu và không có số đo. Quan sát từ code: không phân trang cho danh sách lịch hẹn và slot; chỉ có chỉ mục khóa chính, email, tên chuyên khoa và chỉ mục lịch hoạt động (HT-GAP-15); truy vấn tìm bác sĩ nối và đếm slot của từng bác sĩ mỗi lần gọi. Với 12 slot mẫu không có vấn đề.

## 7. Ngôn ngữ

Giao diện và thông báo nghiệp vụ hoàn toàn **tiếng Việt**. Ngoại lệ: lỗi cú pháp JSON do bộ phân tích của Express trả về bằng tiếng Anh, và lỗi 500 mang nguyên văn tiếng Anh của PostgreSQL (HT-GAP-06, 22).

## 8. Chất lượng mã và kiểm thử

| Khía cạnh | Hiện trạng |
| --- | --- |
| Lint | ESLint 8, `eslint:recommended` + `no-unused-vars` (bỏ qua tham số bắt đầu `_`) + `no-undef` + `semi: always`. `npm run lint` |
| Kiểu dữ liệu | Không TypeScript, không chú thích kiểu |
| Test | 14 test tích hợp qua HTTP thật và PostgreSQL thật, không mock; **không** đo độ phủ |
| CI | GitHub Actions trên `push` vào `main` và mọi `pull_request`: `npm ci` → `lint` → `db:migrate` + `db:seed` → `npm test`, trên PostgreSQL 16 dịch vụ (`medbook_test`) |
| Đóng gói | `node:20-alpine`, `npm install --omit=dev`, chạy bằng quyền root (không đặt `USER`), không `HEALTHCHECK` trong Dockerfile |
| Quy ước | CommonJS · SQL viết chữ thường · đổi tên cột sang camelCase bằng alias trong SQL · `module.exports` ở cuối file · rất ít chú thích, chỉ chú thích **vì sao** |

## 9. Tài liệu đi kèm trong repo

`doc/prod.md` (phạm vi, luồng, quy tắc) · `doc/data-model.md` · `doc/backend-flows.md` · `doc/cicd.md` · `doc/adr/001–007`. Đã đối chiếu quy tắc nghiệp vụ trong `doc/prod.md` §6 với code: **khớp toàn bộ**.

**Tài liệu đã tự nêu các hạn chế sau** (nên coi là *đã biết, chấp nhận*): `slots.status` lưu trùng và có thể lệch với lịch hẹn (ADR-003; `data-model.md:209, 324–341, 464`) · slot bị staff chặn là `booked` mà không có lịch hẹn, **đúng ý đồ** (ADR-003) · migration và seed chạy mỗi lần khởi động, seed **ghi đè** tên và mật khẩu của người dùng mã 1–8, và chạy nhiều bản sao cùng lúc là hỏng (ADR-005 §Tiêu cực).

**Tài liệu nói mà thực tế không đúng:** `data-model.md:415` khẳng định chạy seed lần 2, 3, 10 "đều cho kết quả giống nhau, không lỗi trùng khóa" (minh họa bằng bảng `patients`, nhưng nêu như tính chất của cả seed). Đã tái hiện được hai trường hợp seed **không** idempotent với dữ liệu người dùng tạo ra: hồi sinh lịch mẫu đã hủy, và lỗi trùng khóa khiến ứng dụng không khởi động lại được (HT-GAP-01, 02).

**Lưu ý về `doc/specs/` và `HUONG-DAN-CHAY.md`:** thư mục `doc/specs/` (11 file, gồm `waitlist-feature.md`) và các đoạn nhắc nhánh `day03` trong `HUONG-DAN-CHAY.md` mô tả một tính năng **chưa có trong code này**. Chúng không thuộc trạng thái Day 1 và **không** được dùng làm nguồn cho bộ tài liệu hiện trạng.
