# Hiện trạng · 03 — User story và tiêu chí chấp nhận

> **Mã:** CTX-HT-03 · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 ·
> **Nguồn:** `02-quy-tac-nghiep-vu.md`, `public/js/`, `public/index.html`, `tests/api.test.js`

Các story dưới đây **mô tả chức năng đang có**, viết ngược từ hành vi quan sát được — không phải yêu cầu do ai đặt ra. Tiêu chí (Given – When – Then) chỉ ghi những gì hệ thống **đang làm thật** và đã được xác nhận qua test, lần chạy thử, hoặc đọc code.

**Cột Kiểm chứng:** `Test` (có trong `tests/api.test.js`, kèm dòng) · `Probe` (chạy thật 2026-09-28) · `UI` (đọc code giao diện; chưa mở trình duyệt để bấm) · `—` (chỉ đọc code).

---

## Backlog

| ID | Actor | User story | BR |
| --- | --- | --- | --- |
| **HT-US-01** | Patient, Staff | Tôi muốn đăng nhập bằng tài khoản demo và giữ phiên khi tải lại trang | HT-BR-01, 02, 04 |
| **HT-US-02** | Patient | Tôi muốn tìm bác sĩ theo chuyên khoa hoặc tên để chọn người phù hợp | HT-BR-13 |
| **HT-US-03** | Patient | Tôi muốn xem khung giờ còn trống của một bác sĩ theo ngày để chọn giờ | HT-BR-12 |
| **HT-US-04** | Patient | Tôi muốn đặt lịch (trực tiếp hoặc online) vào một khung giờ trống để giữ chỗ | HT-BR-05, 06, 07, 08 |
| **HT-US-05** | Patient | Tôi muốn xem các lịch hẹn của mình và trạng thái từng lịch | HT-BR-11 |
| **HT-US-06** | Patient | Tôi muốn hủy lịch hẹn của mình khi không đi được | HT-BR-09 |
| **HT-US-07** | Staff | Tôi muốn xem lịch hẹn theo ngày để điều phối | HT-BR-11 |
| **HT-US-08** | Staff | Tôi muốn xác nhận lịch đang chờ để bệnh nhân biết lịch được nhận | HT-BR-10 |
| **HT-US-09** | Staff | Tôi muốn hủy lịch của bất kỳ bệnh nhân khi cần | HT-BR-09 |
| **HT-US-10** | Staff | Tôi muốn xem khung giờ của mọi bác sĩ theo ngày để nắm lịch làm việc | HT-BR-03 |
| **HT-US-11** | Staff | Tôi muốn thêm khung giờ mới cho bác sĩ | HT-BR-14 |
| **HT-US-12** | Staff | Tôi muốn chặn giờ hoặc mở lại một khung giờ để phản ánh lịch làm việc thực tế | HT-BR-15, 16 |
| **HT-US-13** | Patient, Staff | Tôi muốn thấy các chỉ số tổng quan ngay khi vào | — |
| **HT-US-14** | Patient, Staff | Tôi muốn đổi giao diện sáng / tối | — |

---

## HT-US-01 — Đăng nhập và giữ phiên

| AC | Loại | Given | When | Then | BR | Kiểm chứng |
| --- | --- | --- | --- | --- | --- | --- |
| **HT-AC-01.1** | Happy | Tài khoản `an@medbook.local` mật khẩu `demo123` | Gửi đăng nhập | Thành công, nhận thông tin người dùng (không có mật khẩu); giao diện hiện thông báo "Xin chào <tên>" | HT-BR-01, 04 | `Probe`, `UI` |
| **HT-AC-01.2** | Alternative | Email viết hoa `AN@medbook.local` | Gửi đăng nhập | Vẫn vào được | HT-BR-01 | `Probe` |
| **HT-AC-01.3** | Exception | Mật khẩu sai | Gửi đăng nhập | Từ chối 401 "Sai email hoặc mật khẩu demo" | HT-BR-01 | `Test` :135–142 |
| **HT-AC-01.4** | Exception | Chỉ gửi mã người dùng, không có email/mật khẩu | Gửi đăng nhập | Từ chối 400 | HT-BR-01 | `Test` :144–150 |
| **HT-AC-01.5** | Exception | Không có header người dùng | Gọi `GET /api/me` | Từ chối 401 "Thiếu tài khoản demo" | HT-BR-02 | `Test` :53–57 |
| **HT-AC-01.6** | Alternative | Đã đăng nhập, người dùng tải lại trang | Trang khởi động | Đọc mã người dùng đã lưu trong trình duyệt, gọi `/api/me`; thành công thì vào thẳng ứng dụng, thất bại thì về màn hình đăng nhập | HT-BR-02 | `UI` (`login.js:52–66`) |
| **HT-AC-01.7** | Alternative | Đang đăng nhập | Bấm đăng xuất | Xóa mã người dùng đã lưu, về màn hình đăng nhập | — | `UI` (`login.js:4–9`) |

## HT-US-02 — Tìm bác sĩ

| AC | Loại | Given | When | Then | BR | Kiểm chứng |
| --- | --- | --- | --- | --- | --- | --- |
| **HT-AC-02.1** | Happy | Có 6 bác sĩ | Không đặt bộ lọc | Thấy cả 6, sắp theo chuyên khoa rồi tên; mỗi bác sĩ kèm chuyên khoa, phòng, số giờ trống | HT-BR-13 | `Probe` |
| **HT-AC-02.2** | Alternative | Chuyên khoa Tim mạch (mã 1) | Lọc theo chuyên khoa | Chỉ thấy bác sĩ thuộc Tim mạch (2 người) | HT-BR-13 | `Probe` |
| **HT-AC-02.3** | Alternative | Tên "Hùng" | Tìm theo tên | Thấy bác sĩ có tên chứa chuỗi đó, không phân biệt hoa thường | HT-BR-13 | `Probe` |
| **HT-AC-02.4** | Exception | Tên không khớp ai | Tìm theo tên | Danh sách rỗng, giao diện hiện "Không có bác sĩ nào khớp bộ lọc hiện tại." | HT-BR-13 | `Probe`, `UI` |

## HT-US-03 — Xem khung giờ trống

| AC | Loại | Given | When | Then | BR | Kiểm chứng |
| --- | --- | --- | --- | --- | --- | --- |
| **HT-AC-03.1** | Happy | Bác sĩ có slot còn trống từ hôm nay | Bấm "Xem giờ" | Thấy các slot còn trống, sắp theo ngày và giờ, mỗi slot có nút "Giữ chỗ" | HT-BR-12 | `Test` :98–102, `UI` |
| **HT-AC-03.2** | Alternative | Chọn một ngày | Đổi ô ngày | Chỉ thấy slot còn trống của ngày đó | HT-BR-12 | `Test` :98 |
| **HT-AC-03.3** | Exception | Bác sĩ hết giờ trống | Xem giờ | Giao diện hiện "Bác sĩ này không còn giờ trống cho lựa chọn hiện tại." | HT-BR-12 | `UI` |
| **HT-AC-03.4** | Alternative | Xem slot của ngày hôm nay | Giao diện vẽ danh sách | Slot nhóm theo ngày ("Hôm nay · …"), có vạch "Bây giờ" xen vào đúng vị trí giờ hiện tại | — | `UI` (`ui.js:109–135`) |
| **HT-AC-03.5** | Alternative | Slot đã đặt hoặc bị chặn | Xem giờ | Không xuất hiện trong danh sách của bệnh nhân | HT-BR-12 | `Test` :98–102 (ngược lại: slot xuất hiện khi trống) |

## HT-US-04 — Đặt lịch

| AC | Loại | Given | When | Then | BR | Kiểm chứng |
| --- | --- | --- | --- | --- | --- | --- |
| **HT-AC-04.1** | Happy | Slot 1 còn trống; bệnh nhân An | Đặt lịch online | Nhận 201; lịch hẹn ở trạng thái đã đặt, đúng slot, hình thức online; slot chuyển đã đặt. Giao diện hiện "Đã giữ chỗ, chờ nhân viên xác nhận" và làm mới danh sách | HT-BR-05, 06 | `Test` :67–77, `UI` |
| **HT-AC-04.2** | Alternative | Không chọn hình thức khác | Đặt lịch | Hình thức mặc định là trực tiếp | HT-BR-06 | `Probe` |
| **HT-AC-04.3** | Exception | Slot 2 đã có lịch của An | Bệnh nhân đặt slot 2 | Từ chối 409 "Khung giờ đã được đặt" | HT-BR-05 | `Test` :79–87 |
| **HT-AC-04.4** | Exception | Không gửi `slotId` | Đặt lịch | Từ chối 400 "Thiếu slotId" | HT-BR-05 | `Probe` |
| **HT-AC-04.5** | Exception | Hai yêu cầu đặt cùng một slot cùng lúc | Chạy song song | Đúng một 201, một 409 | HT-BR-07, 08 | `Test` :178–189 |
| **HT-AC-04.6** | Exception | Người dùng là staff | Gọi đặt lịch | Từ chối 403 "Không đủ quyền" | HT-BR-03 | `Probe` |
| **HT-AC-04.7** | Exception | Hình thức khám là giá trị lạ (`"weird"`) | Đặt lịch | **Vẫn thành công**, hình thức thành trực tiếp *(hành vi hiện có, ý định chưa rõ: HT-GAP-09)* | HT-BR-06 | `Probe` |

## HT-US-05 — Xem lịch của tôi

| AC | Loại | Given | When | Then | BR | Kiểm chứng |
| --- | --- | --- | --- | --- | --- | --- |
| **HT-AC-05.1** | Happy | An có lịch | Mở "Lịch của tôi" | Thấy lịch của mình, mới nhất trước, mỗi lịch kèm bác sĩ, chuyên khoa, phòng, hình thức, nhãn trạng thái (Chờ xác nhận / Đã xác nhận / Đã hủy) | HT-BR-11 | `Probe`, `UI` |
| **HT-AC-05.2** | Exception | Bệnh nhân chưa có lịch | Mở "Lịch của tôi" | Giao diện hiện "Bạn chưa có lịch hẹn nào. Chọn một bác sĩ để bắt đầu." | HT-BR-11 | `UI` |
| **HT-AC-05.3** | Alternative | Có lịch đã hủy | Xem chỉ số "Lịch của tôi" | Số hiển thị **không** tính lịch đã hủy | — | `UI` (`patient.js:110–113`) |

## HT-US-06 — Hủy lịch của tôi

| AC | Loại | Given | When | Then | BR | Kiểm chứng |
| --- | --- | --- | --- | --- | --- | --- |
| **HT-AC-06.1** | Happy | Lịch 1 của An đang chờ | An hủy | 200; lịch chuyển đã hủy; slot 2 xuất hiện lại trong danh sách trống | HT-BR-09 | `Test` :89–103 |
| **HT-AC-06.2** | Exception | Lịch 2 thuộc Linh | An cố hủy | Từ chối 403 "Không đủ quyền" | HT-BR-09 | `Probe` |
| **HT-AC-06.3** | Exception | Lịch đã hủy | Hủy lần nữa | Từ chối 409 "Lịch hẹn đã bị hủy" | HT-BR-09 | `Probe` |
| **HT-AC-06.4** | Exception | Mã lịch không tồn tại | Hủy | Từ chối 404 "Không tìm thấy lịch hẹn" | HT-BR-09 | `Probe` |
| **HT-AC-06.5** | Alternative | Lịch đã hủy | Xem "Lịch của tôi" | Không có nút Hủy ở lịch đó | HT-BR-09 | `UI` |
| **HT-AC-06.6** | Alternative | Lịch **đã xác nhận** | Bệnh nhân hủy | Vẫn hủy được | HT-BR-09 | `Test` :163–176 *(bằng staff)*, `UI` *(bệnh nhân có nút Hủy)* — ý định: HT-GAP-19 |

## HT-US-07 / 08 / 09 — Staff xem, xác nhận, hủy lịch hẹn

| AC | Loại | Given | When | Then | BR | Kiểm chứng |
| --- | --- | --- | --- | --- | --- | --- |
| **HT-AC-07.1** | Happy | Staff đăng nhập | Xem lịch hẹn (mặc định ngày hôm nay) | Thấy lịch của ngày đó kèm tên, số điện thoại bệnh nhân, bác sĩ, phòng, hình thức, trạng thái | HT-BR-11 | `Probe`, `UI` |
| **HT-AC-07.2** | Alternative | Không chọn ngày | Gọi danh sách | Trả mọi lịch, mới nhất trước | HT-BR-11 | `Probe` |
| **HT-AC-07.3** | Exception | Người dùng là bệnh nhân | Gọi danh sách của staff | Từ chối 403 "Không đủ quyền" | HT-BR-03 | `Test` :59–65 |
| **HT-AC-08.1** | Happy | Lịch 1 đang chờ | Staff xác nhận | 200; lịch thành đã xác nhận | HT-BR-10 | `Test` :105–114 |
| **HT-AC-08.2** | Exception | Lịch đã hủy | Xác nhận | 409 "Chỉ xác nhận được lịch đang chờ xác nhận" | HT-BR-10 | `Probe` |
| **HT-AC-08.3** | Exception | Mã lịch không tồn tại | Xác nhận | **409** cùng thông báo trên, không phải 404 *(HT-GAP-08)* | HT-BR-10 | `Probe` |
| **HT-AC-08.4** | Alternative | Lịch đã xác nhận | Xem danh sách | Không hiện nút Xác nhận | HT-BR-10 | `UI` |
| **HT-AC-09.1** | Happy | Lịch của bệnh nhân bất kỳ | Staff hủy | 200; lịch đã hủy; slot trống lại | HT-BR-09 | `Test` :163–176 |

## HT-US-10 / 11 / 12 — Staff quản lý khung giờ

| AC | Loại | Given | When | Then | BR | Kiểm chứng |
| --- | --- | --- | --- | --- | --- | --- |
| **HT-AC-10.1** | Happy | Staff đăng nhập | Xem lịch làm việc | Thấy slot của mọi bác sĩ từ hôm nay, cả trống lẫn đã đặt, lọc được theo ngày | HT-BR-03 | `Probe`, `UI` |
| **HT-AC-10.2** | Exception | Người dùng là bệnh nhân | Gọi danh sách slot quản lý | 403 | HT-BR-03 | `Test` :59–65 *(cùng cơ chế)* |
| **HT-AC-11.1** | Happy | Bác sĩ 2, ngày 2099-01-20, 11:00–11:30 | Staff thêm slot | 201; slot mới còn trống, đúng bác sĩ, ngày, giờ | HT-BR-14 | `Test` :116–133 |
| **HT-AC-11.2** | Exception | Thiếu một trường | Thêm slot | 400 "Thiếu <tên trường>" | HT-BR-14 | `Probe` |
| **HT-AC-11.3** | Alternative | Giá trị vô nghĩa (giờ kết thúc trước giờ bắt đầu) | Thêm slot | **Vẫn tạo được** *(HT-GAP-05)* | HT-BR-14 | `Probe` |
| **HT-AC-12.1** | Happy | Slot 3 còn trống | Staff chặn giờ | 200; slot đã đặt, không có lịch hẹn | HT-BR-16 | `Test` :191–200 |
| **HT-AC-12.2** | Exception | Slot 2 đang giữ lịch 1 | Staff mở lại | 409 "Không thể mở lại khung giờ đang có lịch hẹn" | HT-BR-15 | `Test` :152–161 |
| **HT-AC-12.3** | Alternative | Lịch 1 đã hủy | Staff mở lại slot 2 | 200; slot còn trống | HT-BR-15 | `Test` :163–176 |
| **HT-AC-12.4** | Exception | Trạng thái không phải "còn trống"/"đã đặt" | Sửa slot | 400 "Trạng thái slot không hợp lệ" | HT-BR-15 | `Probe` |
| **HT-AC-12.5** | Exception | Mã slot không tồn tại hoặc không đọc ra số | Sửa slot | 404 "Không tìm thấy khung giờ" | HT-BR-15 | `Probe` |

## HT-US-13 — Chỉ số tổng quan

| AC | Loại | Given | When | Then | Kiểm chứng |
| --- | --- | --- | --- | --- | --- |
| **HT-AC-13.1** | Happy | Bệnh nhân vào | Xem tổng quan | Ba chỉ số: số bác sĩ hiển thị (theo bộ lọc hiện tại) · số lịch của tôi không tính lịch đã hủy · tổng số giờ trống của các bác sĩ đang hiển thị | `UI` (`patient.js:29–34, 110–113`) |
| **HT-AC-13.2** | Happy | Staff vào | Xem tổng quan | Ba chỉ số: số lịch trong ngày đã chọn **(tính cả lịch đã hủy)** · số slot trống trong ngày · số bác sĩ | `UI` (`staff.js:23, 81–84`) |

## HT-US-14 — Giao diện sáng / tối

| AC | Loại | Given | When | Then | Kiểm chứng |
| --- | --- | --- | --- | --- | --- |
| **HT-AC-14.1** | Happy | Lần đầu vào | Mở trang | Giao diện **sáng** (không theo thiết lập máy) | `UI` (`index.html:6–7`, `main.js:32–35`) |
| **HT-AC-14.2** | Alternative | Đang ở giao diện sáng | Bấm nút đổi | Chuyển tối, lưu lựa chọn trong trình duyệt, nhãn nút đổi theo | `UI` (`main.js:37–42`) |

---

**Tổng:** 14 story · 54 tiêu chí — nguồn mạnh nhất là `Test`: 19 · `Probe`: 23 · `UI` (chỉ đọc code giao diện): 12. *Tiêu chí có nhiều nguồn được đếm theo nguồn mạnh nhất.*
**Chưa thử trong trình duyệt:** mọi tiêu chí `UI` — cần một lượt bấm thủ công để xác nhận.
