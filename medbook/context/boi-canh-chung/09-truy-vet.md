# Hiện trạng · 09 — Truy vết: quy tắc ↔ code ↔ test

> **Mã:** CTX-HT-09 · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · suy ngược từ code · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** `02` → `08` của bộ này, `tests/api.test.js`

Mục đích: cho biết **quy tắc nào đang được test bảo vệ, quy tắc nào chưa**. Đây là thông tin cần trước khi thay đổi hệ thống: điều kiện "14 test cũ vẫn xanh" chỉ bảo vệ phần có test.

---

## 1. Quy tắc → code → test → tiêu chí

| BR | Code thực thi | Test | Tiêu chí | Mức bảo vệ |
| --- | --- | --- | --- | :---: |
| HT-BR-01 Đăng nhập | `authService.js:16–29` | :135, :144 | HT-AC-01.1 → 01.4 | ⚠️ Một phần *(chưa test đăng nhập thành công)* |
| HT-BR-02 Cần khai báo người dùng | `authService.js:31–44`, `demoAuth.js` | :53 | HT-AC-01.5 | ⚠️ Một phần *(chỉ trường hợp thiếu header)* |
| HT-BR-03 Phân quyền vai trò | `requireRole.js`, các route | :59 | HT-AC-04.6, 07.3, 10.2 | ⚠️ Một phần *(chỉ chiều bệnh nhân → endpoint staff)* |
| HT-BR-04 Không lộ mật khẩu | `userRepository.js:3`, `authService.js:5–10` | — | — | ❌ |
| HT-BR-05 Chỉ đặt slot còn trống | `appointmentService.js:15–17` | :67, :79 | HT-AC-04.1, 04.3, 04.4 | ✅ *(thiếu `slotId` và 404: chưa test)* |
| HT-BR-06 Kết quả đặt lịch | `appointmentService.js:9, 19–25` | :67 | HT-AC-04.1, 04.2 | ✅ |
| HT-BR-07 Một slot một lịch hoạt động | `slotRepository.js:64–70`, `migrate.js:53–55` | :178 | HT-AC-04.5 | ✅ |
| HT-BR-08 Đặt đồng thời | như trên | :178 | HT-AC-04.5 | ⚠️ *(hai yêu cầu cùng một người dùng)* |
| HT-BR-09 Hủy lịch | `appointmentService.js:54–80` | :89, :163 | HT-AC-06.1 → 06.6, 09.1 | ⚠️ Một phần *(chưa test: lịch người khác 403, hủy hai lần 409, không tìm thấy 404)* |
| HT-BR-10 Xác nhận lịch | `appointmentRepository.js:82–88` | :105 | HT-AC-08.1 → 08.3 | ⚠️ Một phần *(chỉ đường thành công)* |
| HT-BR-11 Ai thấy lịch nào | `appointmentRepository.js:36–49` | — | HT-AC-05.1, 07.1, 07.2 | ❌ |
| HT-BR-12 Chỉ hiện slot trống từ hôm nay | `slotRepository.js:25–27, 39–47` | :89 | HT-AC-03.1 → 03.5 | ⚠️ Một phần |
| HT-BR-13 Tìm bác sĩ | `doctorRepository.js:7–32` | — | HT-AC-02.1 → 02.4 | ❌ |
| HT-BR-14 Thêm slot | `slotService.js:18–26` | :116 | HT-AC-11.1 → 11.3 | ⚠️ Một phần *(chưa test thiếu trường)* |
| HT-BR-15 Sửa slot | `slotService.js:28–49` | :152, :163 | HT-AC-12.2 → 12.5 | ⚠️ Một phần |
| HT-BR-16 Chặn giờ | như trên | :191 | HT-AC-12.1 | ✅ |

**Tổng:** ✅ 4 · ⚠️ 9 · ❌ 3 trong 16 quy tắc.

## 2. Từng test bảo vệ điều gì

| # | Dòng | Tên test | Quy tắc | Endpoint | Tiêu chí |
| ---: | :---: | --- | --- | --- | --- |
| 1 | 47 | health endpoint is ready | — | HT-API-01 | — |
| 2 | 53 | demo auth rejects requests without X-Demo-User-Id | HT-BR-02 | HT-API-04 | HT-AC-01.5 |
| 3 | 59 | demo auth blocks patient from staff-only endpoint | HT-BR-03 | HT-API-14 | HT-AC-07.3 |
| 4 | 67 | patient can book an available slot | HT-BR-05, 06 | HT-API-12 | HT-AC-04.1 |
| 5 | 79 | patient cannot book an already booked slot | HT-BR-05 | HT-API-12 | HT-AC-04.3 |
| 6 | 89 | cancel appointment returns its slot to available | HT-BR-09, 12 | HT-API-16, 07 | HT-AC-06.1, 03.1 |
| 7 | 105 | staff confirms booked appointment | HT-BR-10 | HT-API-15 | HT-AC-08.1 |
| 8 | 116 | staff can create a new available slot | HT-BR-14 | HT-API-10 | HT-AC-11.1 |
| 9 | 135 | demo login requires a correct password | HT-BR-01 | HT-API-03 | HT-AC-01.3 |
| 10 | 144 | demo login cannot be bypassed with userId alone | HT-BR-01 | HT-API-03 | HT-AC-01.4 |
| 11 | 152 | staff cannot reopen a slot that still has an active appointment | HT-BR-15 | HT-API-11 | HT-AC-12.2 |
| 12 | 163 | staff can reopen a slot once its appointment is cancelled | HT-BR-09, 15 | HT-API-16, 11 | HT-AC-09.1, 12.3, 06.6 |
| 13 | 178 | concurrent booking of the same slot produces exactly one success | HT-BR-07, 08 | HT-API-12 | HT-AC-04.5 |
| 14 | 191 | staff can update slot status | HT-BR-16 | HT-API-11 | HT-AC-12.1 |

**Khuôn của test:** trước mỗi test gọi `migrateAndSeed({ reset: true })` (xóa sạch và nạp lại); gọi HTTP thật vào Express ở cổng ngẫu nhiên; kiểm `response.status` và trường `error` / `data`. **Mẫu để sao chép cho test đồng thời** là test 13.

## 3. Endpoint nào có test

| Có test | Chưa có test nào |
| --- | --- |
| HT-API-01, 03, 04, 07, 10, 11, 12, 15, 16 · HT-API-14 (chỉ ca 403) | HT-API-02, 05, 06, 08, 09, 13 |

## 4. Quy tắc dễ bị phá mà không test nào báo

Những thay đổi sau sẽ **không làm hỏng test nào** trong 14 test hiện có:

| Nếu ai đó… | Quy tắc bị phá | Vì sao test không thấy |
| --- | --- | --- |
| Thêm mật khẩu vào kết quả `/api/demo-users` hoặc `/api/me` | HT-BR-04 | Không test kiểm khóa trả về |
| Bỏ điều kiện chủ sở hữu khi hủy lịch | HT-BR-09 | Test hủy chỉ dùng chủ lịch hoặc staff |
| Đổi `confirmBooked` thành xác nhận cả lịch đã hủy | HT-BR-10 | Chỉ test đường thành công |
| Đổi bộ lọc tìm bác sĩ (tên, chuyên khoa) | HT-BR-13 | Không có test |
| Đảo thứ tự sắp xếp hoặc bỏ lọc `date >= current_date` | HT-BR-11, 12 | Chỉ kiểm slot xuất hiện, không kiểm thứ tự / lọc ngày |
| Cho staff gọi endpoint đặt lịch | HT-BR-03 | Chỉ test một chiều phân quyền |

Khi xây tính năng mới chạm vào các quy tắc ở bảng này, **thêm test cho quy tắc trước khi đổi** — nếu không, "14 test xanh" không chứng minh điều gì về chúng.

## 5. Khoảng trống → ảnh hưởng tới tính năng danh sách chờ

Bảng này nối hiện trạng với các quy tắc và task của WB-2 (chưa hiện thực).

| Khoảng trống | Liên quan tới | Vì sao |
| --- | --- | --- |
| HT-GAP-01, 02 (seed) | TASK-09, dữ liệu mẫu mới | Seed mới không được ghi đè dữ liệu người dùng; nếu không, nó thừa hưởng lỗi khởi động lại |
| HT-GAP-03 (trùng giờ của bệnh nhân) | BR-03 (d), R-06 | Luật loại ứng viên trùng giờ là **mới**; đặt chủ động vẫn cho trùng, nên có thể tạo ra trùng giờ mà đề xuất phải né |
| HT-GAP-04, 23 (slot quá khứ, múi giờ) | BR-06, BR-16 | Hai luật này so sánh `date + start_time` với thời điểm hiện tại: cần một quy ước múi giờ trước khi hiện thực |
| HT-GAP-05 (slot không kiểm tra, chồng giờ) | R-08, BR-03 (d) | So khoảng giờ có thể cho kết quả lạ trên dữ liệu slot chồng nhau |
| HT-GAP-06, 22 (lỗi lộ nội dung) | Mọi endpoint mới | Lỗi 500 của endpoint mới cũng lộ nội dung PostgreSQL nếu không bọc |
| HT-GAP-10 (sửa slot không giao dịch) | BR-01, BR-10 | Tính năng mới thêm đường đổi `slots.status`; đường cũ này vẫn có khe hở |
| HT-GAP-12 (đăng nhập giả mạo được) | BR-14 | Kiểm chủ sở hữu ở service là bắt buộc |
| HT-GAP-13, 14 (cấu hình chết, test xóa CSDL) | công tắc `OFFER_ENGINE_ENABLED`, test mới | Không có sẵn cơ chế môi trường để dựa vào; test mới dùng cùng khuôn xóa sạch |
| HT-GAP-18 (độ phủ test) | K-10 "14 test cũ xanh" | Chỉ bảo vệ phần có test |
| HT-GAP-19 (hủy lịch đã xác nhận) | BR-01 | Xác định phạm vi "slot trở nên khả dụng" |
| HT-GAP-21 (đề bài lệch hệ thống) | R-17 | Xem bảng đối chiếu ở `08-khoang-trong.md` §2 |
