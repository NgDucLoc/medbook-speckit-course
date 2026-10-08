# Bối cảnh chung — MedBook đầu Day 1, suy ngược từ code

> **Mã:** CTX-HT · **Phiên bản:** v0.1 · **Ngày:** 2026-09-28
> **Trạng thái:** Bối cảnh chung cho WB-1 → WB-5 · suy ngược từ code · không phải sản phẩm nộp, không cần duyệt
> **Nguồn:** `WB-1-…/source-code/` (bản Day 1: chưa có danh sách chờ; code giống hệt ở cả 5 WB) · ứng dụng đang chạy · các lần chạy thử

---

## 1. Đây là gì

Bộ tài liệu mô tả MedBook **đúng như nó là ở đầu Day 1**, viết ngược từ code: hệ thống làm gì, theo quy tắc nào, dữ liệu và API ra sao, thành phần nào lo việc gì, đâu là chỗ còn thiếu hoặc sai. Nó là **bối cảnh chung của cả 5 WB**: mọi WB đọc cùng một mô tả hệ thống có dẫn chứng, thay cho việc dựa vào tài liệu tham chiếu do khóa học cấp (vốn có cả nội dung của Day 2).

Nó **không** phải yêu cầu do ai đặt ra. Code cho biết hệ thống **đang làm gì**, không cho biết nó **nên** làm gì. Vì vậy bộ này tách hai loại:

| Loại | Ở đâu | Nghĩa |
| --- | --- | --- |
| **Luật có chủ ý** | `02-quy-tac-nghiep-vu.md` | Có trong code **và** được xác nhận bằng tài liệu dự án, test hoặc chú thích của tác giả |
| **Hành vi ngẫu nhiên, thiếu sót, lỗi** | `08-khoang-trong.md` | Xảy ra thật nhưng không có căn cứ là ý định. **Không** được coi là luật |

Chỗ nào ý định không rõ đều ghi `[CẦN XÁC NHẬN]` hoặc nằm ở `08-`. Bộ này **không phân xử** và không đòi ai quyết định: nó chỉ cho người dùng sau biết hệ thống thật đang như thế nào.

## 2. Cách suy ra và độ tin cậy

1. Đọc toàn bộ mã nguồn (`server.js`, `src/`, `public/`, `tests/`, cấu hình), ghi dòng làm bằng chứng.
2. Chạy ứng dụng (`docker compose up --build`).
3. **Chạy thử hơn 30 hành vi** trên một bản ứng dụng tách biệt với cơ sở dữ liệu tạm (không đụng ứng dụng đang chạy).
4. Chạy thử seed trên cơ sở dữ liệu tạm khác để kiểm chứng hành vi khởi động lại.
5. Đối chiếu với tài liệu trong repo (`doc/prod.md`, `data-model.md`, ADR) và với đề bài WB-2.

| Ký hiệu | Nghĩa |
| --- | --- |
| `Code` / `tệp:dòng` | Đọc thấy trong mã nguồn tại dòng đó (tính từ `source-code/`) |
| `Test` | Có test trong `tests/api.test.js` (kèm dòng) |
| `Probe` | Đã chạy thật ngày 2026-09-28 và quan sát kết quả |
| `UI` | Đọc từ mã giao diện; **chưa mở trình duyệt để bấm** |
| `Suy ra` | Suy luận, chưa kiểm chứng |

## 3. Danh mục

| File | Nội dung |
| --- | --- |
| `01-pham-vi-va-boi-canh.md` | Hệ thống làm gì, cho ai; chức năng **có** và **không có**; công nghệ; màn hình; dữ liệu mẫu |
| `02-quy-tac-nghiep-vu.md` | **HT-BR-01 … 16** — luật đang được thực thi có chủ ý |
| `03-user-story-va-tieu-chi.md` | **HT-US-01 … 14**, **HT-AC** (54 tiêu chí Given–When–Then) |
| `04-du-lieu.md` | **HT-ENT-01 … 06** — sáu bảng, chỉ mục, vòng đời, dữ liệu mẫu, cách seed chạy |
| `05-api.md` | **HT-API-01 … 16** — 16 endpoint, hình dạng đối tượng, bảng mã lỗi thật |
| `06-thanh-phan-va-luong.md` | **HT-CMP-01 … 28** — thành phần, đồ thị phụ thuộc, luồng xử lý |
| `07-phi-chuc-nang-va-cau-hinh.md` | Cấu hình, bảo mật, đồng thời, múi giờ, quy ước, CI |
| `08-khoang-trong.md` | **HT-GAP-01 … 23** — lỗi, thiếu sót, rủi ro, tài liệu lệch code |
| `09-truy-vet.md` | Quy tắc ↔ code ↔ test; **quy tắc nào chưa có test bảo vệ** |
| `10-nguyen-tac-va-rang-buoc.md` | 8 nguyên tắc và ràng buộc bất biến, ràng buộc **mọi** yêu cầu, thiết kế, task viết thêm — không riêng tính năng nào |

**Thứ tự đọc:** `01` → `02` → `08` → `09` → `10`. Muốn dùng làm bối cảnh cho một task: `05`, `06`, `04`, `10`.

## 4. Mã

Tiền tố `HT-` (hiện trạng) để **không trùng** với các mã của tính năng mới ở WB-2 (`BR-xx`, `US-xx`, `AC-xx.y`, `API-xx`, `ENT-xx`, `CMP-xx`). Mã chỉ cấp một lần; bỏ một mục thì đánh `[BỎ]`, giữ mã.

`HT-BR` quy tắc · `HT-US` story · `HT-AC` tiêu chí · `HT-ENT` thực thể · `HT-API` endpoint · `HT-CMP` thành phần · `HT-GAP` khoảng trống.

## 5. Điều đáng biết nhất (đọc trước khi làm gì tiếp)

| # | Phát hiện | Mục |
| ---: | --- | --- |
| 1 | **Khởi động lại có thể làm ứng dụng không lên được**, và hoàn tác thao tác của người dùng trên dữ liệu mẫu (đã tái hiện) | HT-GAP-01, 02 |
| 2 | **Không có luật "một bệnh nhân không đặt hai lịch trùng giờ"** — đề bài WB-2 nói có, code không có (đã tái hiện) | HT-GAP-03 |
| 3 | **Đề bài WB-2 mô tả hiện trạng có "đổi lịch", "thông báo", Admin — hệ thống không có** | HT-GAP-21 |
| 4 | **Múi giờ:** máy chủ chạy UTC, giờ slot không kèm múi giờ — mọi luật dựa trên "còn bao lâu tới giờ khám" sẽ lệch | HT-GAP-23 |
| 5 | **3 trong 16 quy tắc không có test nào bảo vệ, 9 chỉ được bảo vệ một phần** — "14 test cũ xanh" là bảo đảm yếu | `09-truy-vet.md` §1 |
| 6 | Lỗi 500 lộ nguyên văn thông báo PostgreSQL; `id` trong body ghi đè `id` trên đường dẫn ở `PUT /api/slots/:id` | HT-GAP-06, 07 |

## 6. Lưu ý về các tài liệu dễ nhầm

`source-code/doc/specs/` (11 file, gồm `waitlist-feature.md`) và các đoạn nhắc nhánh `day03` trong `HUONG-DAN-CHAY.md` mô tả tính năng **chưa có trong code này**. Bộ hiện trạng **không** dùng chúng làm nguồn. Các tài liệu hợp lệ của Day 1 là `doc/prod.md`, `doc/data-model.md`, `doc/backend-flows.md`, `doc/cicd.md` và `doc/adr/001–007` — đã đối chiếu, xem `07-phi-chuc-nang-va-cau-hinh.md` §9.

## 7. Dùng bộ này thế nào ở các WB sau

| WB | Dùng để |
| --- | --- |
| **WB-1** | Đầu vào **IN-09**: bằng chứng cho Artifact Map và điểm đứt gãy |
| **WB-2** | Thay `00-boi-canh/01-he-thong-hien-tai.md` bằng bản có dẫn chứng dòng; `08-khoang-trong.md` là nguồn cho các mục R-xx; `HT-BR` là "luật đã có" mà `BR-xx` mới không được mâu thuẫn |
| **WB-3** | `09-truy-vet.md` cho biết test nào bảo vệ gì; thêm test cho quy tắc chưa được bảo vệ **trước** khi đổi |
| **WB-4** | `08-khoang-trong.md` §3 và `09-truy-vet.md` §4 là mẫu cho phân tích "AI sai ở đâu mà test không thấy" |
| **Mọi WB** | `10-nguyen-tac-va-rang-buoc.md` áp dụng ngay, không cần đợi cổng duyệt riêng — vi phạm một nguyên tắc phải giải thích rõ lý do |
