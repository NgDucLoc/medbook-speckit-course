# TOÀN BỘ PROMPT — MedBook × Spec-Kit

Làm **Bước 0 (cài Spec-Kit, git init, specify init)** trong `KICHBAN.md` trước, rồi dán lần lượt từng khối vào Claude Code (mở ở thư mục `medbook/`). Khối bắt đầu bằng `/speckit-…` là lệnh Spec-Kit.

Đầu vào: `context/yeu-cau-chuc-nang-moi.md` (mô tả chức năng mới) + `context/boi-canh-chung/` (hệ thống hiện có).


---

## Phần 1 — Yêu cầu & thiết kế (chưa code)


### `day2/2.1-constitution.txt`

```text
/speckit-constitution Tạo constitution cho MedBook từ context/boi-canh-chung/10-nguyen-tac-va-rang-buoc.md:
giữ đủ 8 nguyên tắc I–VIII (không phá hợp đồng 16 endpoint & 14 test cũ; không thêm dependency
runtime ngoài express/pg; kiến trúc phân lớp route→service→repository, service không gọi
service; SQL thuần tham số hóa; đổi trạng thái slot trong transaction + khóa dòng + unique index
làm lớp phòng thủ cuối; mọi business rule có mã + nguồn quyết định, thiếu thì gắn [CẦN XÁC NHẬN]
chứ không tự bịa; tài liệu trước code sau; kiểm thử suy từ acceptance criteria), cùng ràng buộc
kỹ thuật bổ sung (Node 20, Express 4, PostgreSQL 16, frontend thuần, lỗi tiếng Việt không lộ SQL,
seed chạy lại được, kiểm tra quyền sở hữu ở tầng service). Version 1.0.0. Bỏ qua mọi tham chiếu
tới thư mục WB-2 (không tồn tại ở đây).
```


### `day2/2.2-cau-hoi-lam-ro.txt`

```text
Đây là yêu cầu thô (từ context/yeu-cau-chuc-nang-moi.md, mục "Yêu cầu mới"
). Ghi bai-nop/day2/01-clarification-questions.md: (1) điều gì còn mơ hồ, (2) điều
gì còn thiếu, (3) edge case chưa được đề cập, (4) ÍT NHẤT 8 câu hỏi clarification cho
stakeholder, mỗi câu có: lý do quan trọng, ai nên trả lời, phương án mặc định đề xuất. Ví dụ
gợi ý chủ đề: X phút là bao nhiêu; "ưu tiên y tế" ai gán, mấy mức; slot mới tạo có kích hoạt
không; bệnh nhân có lịch trùng giờ; hai slot trống cùng lúc; bệnh nhân từ chối có được vào lại
hàng chờ; thông báo kênh nào. KHÔNG tự trả lời — chỉ hỏi.
```


### `day2/2.3a-specify.txt`

```text
/speckit-specify Quản lý linh hoạt danh sách chờ (waiting list) cho MedBook. Hiện hệ thống
không có cơ chế nào khi một khung giờ khám trống đột ngột; nhân viên phải gọi/nhắn từng bệnh
nhân. Tính năng mới: khi một slot trở nên khả dụng (do hủy lịch hoặc nhân viên mở lại slot),
hệ thống tự xác định bệnh nhân phù hợp nhất trong danh sách chờ theo mức ưu tiên y tế, thời
gian chờ và sự phù hợp chuyên khoa/bác sĩ; gửi đề xuất cho bệnh nhân; bệnh nhân phản hồi trong
X phút; nếu từ chối hoặc không phản hồi thì chuyển sang bệnh nhân kế tiếp; xử lý xung đột lịch;
thông báo đầy đủ cho các bên (bệnh nhân, nhân viên). Vai trò: bệnh nhân (patient), nhân viên
(staff). Cần ít nhất 3 user story chính + acceptance criteria Given-When-Then (happy path và
edge case: danh sách chờ trống, tất cả từ chối, hết hạn, xung đột lịch, hai người cùng nhận).
Giữ nguyên mọi hành vi hiện có của MedBook; tuân thủ constitution.
```


### `day2/2.3b-tra-loi-specify.txt`

```text
Q1: B, Q2: B, Q3: A
```


### `day2/2.4-clarify.txt`

```text
/speckit-clarify

Bạn đang chạy ở chế độ không tương tác; Human không thể trả lời từng câu. Đây là các QUYẾT ĐỊNH CỦA HUMAN, hãy dùng làm câu trả lời cho mọi câu hỏi liên quan và ghi vào mục Clarifications (không hỏi lại những mục đã có đáp án):
- Thời gian phản hồi X = 15 phút, cấu hình bằng biến môi trường OFFER_TTL_MINUTES.
- Mức ưu tiên y tế: 3 mức urgent > high > normal; nhân viên gán khi thêm bệnh nhân vào danh sách chờ.
- Thứ tự chọn: ưu tiên y tế giảm dần → thời gian vào danh sách chờ tăng dần → id tăng dần. Không random, không FIFO thuần.
- Phù hợp: cùng chuyên khoa với slot; nếu bệnh nhân chọn bác sĩ cụ thể thì phải đúng bác sĩ đó.
- Kích hoạt: chỉ khi slot về available do hủy lịch hoặc nhân viên mở lại slot. Slot mới tạo / seed không kích hoạt.
- Lịch trùng giờ: bệnh nhân đã có lịch trùng giờ thì bỏ qua, xét người kế tiếp.
- Kênh thông báo: trong ứng dụng (bệnh nhân polling GET /api/my-offers); không SMS/email.
- Từ chối / hết hạn: chuyển người kế tiếp; bệnh nhân từ chối không được đề xuất lại đúng slot đó; hết hạn xử lý bằng sweeper quét định kỳ.
- Lead time tối thiểu: không tạo đề xuất nếu slot bắt đầu trong dưới 30 phút.
- Hết hàng chờ: slot giữ available bình thường.
Hãy hoàn tất toàn bộ vòng clarify trong một lượt.
```


### `day2/2.5a-review-user-story.txt`

```text
Đọc specs/001-*/spec.md. Review như một Requirement Reviewer: (1) mỗi user story có đủ
role/action/benefit và AC Given-When-Then không; (2) happy path vs edge case có đủ không; (3)
liệt kê những chỗ AI (spec bạn vừa sinh) viết SAI hoặc THIẾU so với context/boi-canh-chung
(đặc biệt luật sẵn có HT-BR và khoảng trống HT-GAP-03: hệ thống chưa chống đặt trùng giờ).
Ghi bai-nop/day2/02-ai-errors-log.md: mỗi dòng = lỗi/thiếu, vì sao, mức độ, đã sửa chưa.
File này sẽ dùng làm bằng chứng ở Day 4. Chưa sửa spec.md.
```


### `day2/2.5b-cap-nhat-spec.txt`

```text
/speckit-specify cập nhật spec.md hiện có: áp dụng các mục tôi đã đồng ý trong
bai-nop/day2/02-ai-errors-log.md (đánh dấu mục nào là "Accepted"). Không thêm business rule mới
ngoài những gì đã chốt ở phần Clarifications; nếu thiếu thì để [NEEDS CLARIFICATION].
```


### `day2/2.6a-plan.txt`

```text
/speckit-plan Ngăn xếp cố định theo constitution: Node 20, Express 4, PostgreSQL 16, SQL thuần,
frontend thuần, KHÔNG thêm dependency runtime. Trong research.md hãy đánh giá ít nhất 3 phương
án kiến trúc cho Offer Engine: (A) xử lý đồng bộ trong appointmentService/slotService,
(B) Offer Engine in-process, DB là nguồn sự thật, sweeper setInterval quét offer hết hạn,
(C) message broker + worker riêng. Với mỗi phương án: cách tiếp cận, component thêm/sửa trên
codebase hiện có, trade-off (performance, complexity, maintainability, scalability, độ tin cậy,
rollback) trong ma trận có trọng số. CHỌN phương án B và giải thích (A không xử lý được timeout;
C vi phạm ràng buộc dependency và "một lệnh chạy"). Ghi điều kiện khi nào nên chuyển sang C.
Thiết kế: AppointmentOffer là entity độc lập (không phải trạng thái của appointments); Offer
Engine gọi SAU commit và NGOÀI transaction của service gọi nó; không giữ state trong bộ nhớ;
chỉ gọi xuống repository; thông báo in-app qua GET /api/my-offers. Sinh data-model.md,
contracts/ (endpoint mới, KHÔNG sửa endpoint cũ), quickstart.md.
```


### `day2/2.6b-architecture-blueprint.txt`

```text
Tổng hợp từ plan.md, research.md, data-model.md, contracts/ vào bai-nop/day2/04-architecture-blueprint.md:
sơ đồ kiến trúc high-level (Mermaid), danh sách component THÊM / SỬA trên codebase MedBook hiện
có (đường dẫn file cụ thể), quyết định công nghệ, trade-off analysis, rủi ro chính (đặc biệt
đồng thời: hai người cùng accept, sweeper chạy song song) và cách giảm. Chỉ tổng hợp, không bịa
thêm quyết định mới.
```


### `day2/2.7a-checklist.txt`

```text
/speckit-checklist Chất lượng yêu cầu của waiting list: đầy đủ, rõ ràng, nhất quán, đo được; tập
trung vào luật chọn ứng viên, vòng đời offer (pending/accepted/declined/expired), đồng thời,
thông báo, quyền sở hữu dữ liệu.
```


### `day2/2.7b-requirement-review.txt`

```text
Review toàn bộ spec.md + plan.md + data-model.md + contracts/ + checklists/. Ghi
bai-nop/day2/03-requirement-review-notes.md: (1) user story/AC nào mâu thuẫn nhau; (2) gap;
(3) đối chiếu kiến trúc đã chọn: yêu cầu nào khó/không khả thi; (4) kiểm tra từng nguyên tắc
I–VIII của constitution có bị vi phạm không. Với mỗi phát hiện: mức độ và đề xuất sửa. Sau đó
sửa spec.md/plan.md cho những phát hiện mức Critical/High, ghi rõ đã sửa gì.
```


### `day2/2.8-tong-ket.txt`

```text
Tạo bai-nop/day2/README.md: checklist cuối ngày (7 mục đề Day 2) ✅/❌ + dẫn chứng; liệt kê các
câu hỏi mở còn lại (nếu có) — không tự trả lời.
```


---

## Phần 2 — Hiện thực & kiểm thử


### `day3/3.1-tasks.txt`

```text
/speckit-tasks Chia theo user story để mỗi story có thể kiểm thử độc lập. Thứ tự: migration +
repository → Offer Engine (matching, tạo offer) → hook sau commit ở hủy lịch / mở lại slot →
accept/decline + xử lý conflict → sweeper timeout → API my-offers/waiting-list → giao diện.
Mỗi task có đường dẫn file và trỏ về AC tương ứng. Có task test cho từng AC, gồm test đồng thời.
```


### `day3/3.2a-analyze.txt`

```text
/speckit-analyze
```


### `day3/3.2b-sua-finding.txt`

```text
Sửa tất cả finding CRITICAL và HIGH trong báo cáo /speckit-analyze vừa rồi, CHỈ bằng cách cập nhật spec.md / plan.md / tasks.md (Nguyên tắc VII: tài liệu trước, code sau; KHÔNG sửa code ở bước này). Với mỗi finding đã sửa, nói rõ file và mục đã đổi; finding MEDIUM/LOW chỉ liệt kê, chưa sửa. Không thêm business rule mới ngoài những gì đã chốt ở Clarifications.
```


### `day3/3.3a-duyet-checklist.txt`

```text
Duyệt checklist specs/001-waiting-list/checklists/offer-engine.md: với từng mục, đối chiếu spec.md/plan.md/data-model.md/contracts và đánh dấu [x] nếu yêu cầu ĐÃ đạt (ghi chú ngắn nơi có bằng chứng). Mục nào chưa đạt: giữ [ ] và sửa spec/plan nếu là thiếu sót nhỏ rồi mới tick; nếu cần quyết định của Human thì để [ ] và liệt kê cho tôi. Không tick bừa.
```


### `day3/3.3b-quyet-dinh-human.txt`

```text
Quyết định của Human cho MỌI mục checklist còn mở (áp theo chủ đề, không theo mã mục):
- SLA/độ trễ: không cam kết SLA cứng; bệnh nhân thấy đề xuất ở lần polling kế tiếp, sweeper xử lý hết hạn trong tối đa 60 giây.
- Lịch sử từ chối/hết hạn cho nhân viên: KHÔNG làm (ngoài phạm vi).
- Trạng thái "đã đọc/chưa đọc" của đề xuất: KHÔNG làm; danh sách đề xuất sắp theo thời gian mới nhất trước là đủ.
- Bệnh nhân đang giữ một đề xuất chờ phản hồi trùng giờ với slot mới: KHÔNG tạo đề xuất mới cho họ.
- Chỉ số đo theo thời gian (ví dụ tỉ lệ lấp slot): đo theo tuần (7 ngày gần nhất), là chỉ số vận hành, không build trong phạm vi này.
- Mọi mục khác còn mở: chọn phương án đơn giản nhất, không thêm business rule mới ngoài mục Clarifications.
Ghi các quyết định vào mục Clarifications của spec.md (thêm FR + task + test nếu cần), rồi tick các mục checklist tương ứng.
```


### `day3/3.4a-implement-dot1.txt`

```text
/speckit-implement Chỉ làm các task migration, repository và Offer Engine (matching + tạo offer)
trong tasks.md. Viết code cùng style/convention codebase hiện có (xem doc/backend-flows.md và
src/services/appointmentService.js). Tuân thủ constitution: SQL tham số hóa, transaction + khóa
dòng, không thêm dependency. Đánh dấu [X] task đã xong. Dừng sau đợt này để tôi review.
```


### `day3/3.4b-sua-test.txt`

```text
Chạy `npm test` (biến môi trường DATABASE_URL đã được đặt sẵn — dùng đúng nó, không đổi cổng, không đụng DB khác). Hiện có test fail (deadlock, transaction aborted…). Tìm nguyên nhân gốc (đọc kỹ lỗi, đừng đoán), sửa tận gốc mà KHÔNG làm yếu hay xóa test nào và không sửa hành vi của 14 test cũ. Chạy lại tới khi toàn bộ xanh và `npm run lint` sạch, rồi giải thích nguyên nhân trong 3 dòng.
```


### `day3/3.4c-implement-dot2.txt`

```text
/speckit-implement Làm tiếp các task: hook sau commit ở hủy lịch và mở lại slot; accept/decline;
xử lý conflict lịch và race condition khi hai bệnh nhân cùng accept. Dừng sau đợt này. Chạy test bằng biến DATABASE_URL đã đặt sẵn.
```


### `day3/3.4d-implement-dot3.txt`

```text
/speckit-implement Làm tiếp: sweeper timeout (setInterval, đọc DB mỗi lượt, cấu hình bằng env),
endpoint waiting-list và my-offers, giao diện tối thiểu (staff thêm bệnh nhân vào danh sách chờ;
bệnh nhân thấy thẻ đề xuất có đếm ngược, nút chấp nhận/từ chối). Chạy test bằng biến DATABASE_URL đã đặt sẵn.
```


### `day3/3.5-unit-test.txt`

```text
Đọc AC trong spec.md và các task test trong tasks.md. Viết test (node --test, cùng style
tests/api.test.js) trong tests/waitlist.test.js: mỗi test có comment "AC-x.y" trỏ về tiêu chí.
Phủ: happy path (có ứng viên và accept), edge case (hàng chờ trống, tất cả từ chối, lịch trùng,
slot < 30 phút, hết hạn), timeout, và test ĐỒNG THỜI (hai bệnh nhân accept cùng lúc → đúng 1
thành công). Chạy `npm test`. Ghi bai-nop/day3/03-testing-package.md: bảng AC → test (còn AC
nào chưa có test?), test pass/fail và nguyên nhân, test nào AI viết đúng, test nào phải sửa và
vì sao, báo cáo coverage (`node --test --experimental-test-coverage tests/*.test.js`).
```


### `day3/3.6-code-review.txt`

```text
Review diff của nhánh feature/waitlist so với main (git diff main). Chỉ ra: (1) code quality
(naming, độ phức tạp, trùng lặp); (2) bug/logic error tiềm ẩn; (3) lỗ hổng bảo mật — đặc biệt
SQL injection, kiểm tra QUYỀN SỞ HỮU offer (bệnh nhân A có accept được offer của B không?),
race condition khi nhiều bệnh nhân nhận cùng slot, sweeper chạy chồng; (4) điểm có thể thành vấn
đề hiệu năng khi lớn. Đối chiếu từng nguyên tắc I–VIII. Ghi bai-nop/day3/04-code-review-report.md:
≥ 3 issue, mỗi issue có severity (Critical/Major/Minor), file:dòng, đề xuất, và phân loại FIX NOW
hay TECH DEBT kèm lý do. Nêu rõ cả những gì bạn KHÔNG chắc hoặc không kiểm tra được.
```


### `day3/3.7-converge.txt`

```text
/speckit-converge
```


### `day3/3.8-development-package.txt`

```text
Tạo bai-nop/day3/01-development-package.md (API mới, quyết định implementation chính,
file đã thêm/sửa, cách chạy) và bai-nop/day3/README.md (checklist cuối ngày đề Day 3 ✅/❌ +
dẫn chứng; ghi lại: AI sinh đúng gì, cần sửa gì, vì sao).
```


---

## Phần 3 — Governance & phân tích lỗi (tùy chọn)


### `day4/4.1-tai-hien-su-co.txt`

```text
Đóng vai AI-Assisted Auditor, KHÔNG sửa code. Với 2 triệu chứng: (a) bệnh nhân nhận đề xuất từ
bác sĩ/chuyên khoa sai; (b) bệnh nhân nhận đề xuất lặp dù đã có lịch trùng giờ — viết thử các
test thăm dò trong tests/probe-day4.test.js (không commit vào bộ test chính) để xem code hiện
tại có tái hiện không. Báo cáo trung thực: tái hiện được / không tái hiện được / không kết luận
được, kèm file:dòng và đầu ra test. Không kết luận "an toàn" nếu không có bằng chứng.
```


### `day4/4.2-failure-analysis.txt`

```text
Dùng bai-nop/day2/02-ai-errors-log.md, spec.md (AC), tests/waitlist.test.js, bai-nop/day3/04-code-review-report.md
và kết quả 4.1. Ghi bai-nop/day4/01-failure-analysis-report.md: timeline sự cố; truy từng bước
Day 2→3: requirement mơ hồ ở đâu, test thiếu ở đâu, review bước nào đáng lẽ phải bắt được (dẫn
file:dòng/ID AC); phân loại nguyên nhân gốc vào 5 nhãn (Spec sai/thiếu · AI sinh sai · Test thiếu ·
Review bỏ sót · Process không có checkpoint) với tỉ trọng; kết luận đây là lỗi AI hay lỗi
Human-AI process và vì sao; risk assessment nếu không phát hiện sớm; bài học. Mọi kết luận phải
có bằng chứng; chỗ không chắc ghi [CẦN XÁC NHẬN].
```


### `day4/4.3-review-workflow.txt`

```text
Ghi bai-nop/day4/02-human-ai-review-workflow.md: checkpoint còn thiếu trong các bước trước; workflow
review mới có ≥ 3 validation checkpoint (ai review gì, khi nào, tiêu chí); phân biệt rõ chỗ
nào Human-in-the-loop (review từng output) và chỗ nào Human-on-the-loop (batch/trigger); chỉ rõ
checkpoint nào là HARD GATE chặn merge. Ánh xạ mỗi checkpoint sang lệnh Spec-Kit tương ứng
(/speckit-analyze, /speckit-checklist, /speckit-converge…). Sơ đồ Mermaid.
```


### `day4/4.4-governance-checklist.txt`

```text
Ghi bai-nop/day4/03-ai-governance-checklist.md: ≥ 8 mục, phân theo loại risk (hallucination,
security, incomplete output, over-reliance); mỗi mục có: câu kiểm chứng được (Yes/No), mitigation,
người chịu trách nhiệm, cách kiểm chứng (lệnh/file). Phải đủ cụ thể để dùng trước khi merge
code do AI sinh vào main. Ví dụ mục: "mọi AC đều có test trỏ về", "không có dependency runtime
mới", "không business rule nào thiếu mã nguồn quyết định".
```


### `day4/4.5a-constitution-v1.1.txt`

```text
/speckit-constitution Nâng lên v1.1.0: thêm mục "Governance cho mã do AI sinh" từ 3 hard-gate
trong bai-nop/day4/02-human-ai-review-workflow.md và 5 mục quan trọng nhất của
bai-nop/day4/03-ai-governance-checklist.md. Ghi Sync Impact Report. Không đổi nguyên tắc I–VIII.
```


### `day4/4.5b-checklist-pre-merge.txt`

```text
/speckit-checklist Pre-merge cho mã do AI sinh của waiting list, dựa trên constitution v1.1.0.
```


### `day4/4.5c-analyze-audit.txt`

```text
/speckit-analyze Tập trung: AC nào chưa có test; chỗ nào code làm khác spec; business rule nào
thiếu nguồn quyết định.
```


### `day4/4.6-tong-ket.txt`

```text
Tạo bai-nop/day4/README.md: checklist cuối ngày (7 mục đề Day 4) ✅/❌ + dẫn chứng; trả lời
trong 5 dòng: lỗi này là lỗi AI hay lỗi process, vì sao.
```
