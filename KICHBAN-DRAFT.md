# KỊCH BẢN THỰC HÀNH — MedBook × Spec-Kit (5 ngày)

> **Trạng thái: DRAFT v0.1 — chờ bạn duyệt nội dung, CHƯA chạy thử.**
> Sau khi bạn duyệt, tôi sẽ chạy thử toàn bộ bằng Claude Code (headless), sửa tới khi chạy đúng, rồi chốt thành `KICHBAN.md`.

---

## 0. Cách đọc kịch bản này

**Mục tiêu:** đi hết 5 ngày của case study MedBook (waiting list) nhưng **dùng Spec-Kit làm xương sống**: mọi artifact quan trọng nằm trong một chuỗi có truy vết, thay vì mỗi phase một cuộc hội thoại AI riêng (đúng vấn đề "context drift" của Day 1).

**Quy ước:**

| Ký hiệu | Nghĩa |
| --- | --- |
| 🖥️ `Terminal` | Gõ trong terminal bình thường |
| 🤖 `Claude` | Gõ trong phiên `claude` (đã mở ở thư mục `medbook/`) |
| 👤 **Human Checkpoint** | Bạn đọc, quyết định, sửa — không được bỏ qua |
| 📄 **Kết quả** | File phải xuất hiện sau bước đó (để tự kiểm tra) |

**Phân vai:** Spec-Kit lo chuỗi *đặc tả → kế hoạch → task → code*. Những hoạt động Spec-Kit **không** có lệnh (Artifact Map, Responsibility Matrix, Failure Analysis, Governance, Metrics) dùng prompt thường nhưng **ghi vào `bai-nop/`** và **tham chiếu ngược về artifact Spec-Kit** (`specs/001-.../`).

### Bản đồ Ngày ↔ Spec-Kit

| Ngày | Chủ đề | Lệnh Spec-Kit dùng | Sản phẩm (đề bài) |
| --- | --- | --- | --- |
| 0 | Chuẩn bị | `specify init` | Môi trường chạy được, 14 test xanh |
| 1 | Từ SDLC truyền thống sang AI-Driven | *(chưa có lệnh — chỉ đọc cấu trúc Spec-Kit để đưa vào Future-State)* | Artifact Map · AI Opportunity Matrix · Responsibility Matrix · Future-State Workflow |
| 2 | Requirement & System Design | `/speckit-constitution` → `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-checklist` | Requirement Package · Architecture Blueprint |
| 3 | Development & Quality | `/speckit-tasks` → `/speckit-analyze` → `/speckit-implement` → `/speckit-converge` | Development Package · Testing Package · Code Review Report |
| 4 | Governance & Failure Analysis | `/speckit-analyze` (audit) · `/speckit-constitution` (nâng v1.1) · `/speckit-checklist` | Failure Analysis Report · Review Workflow · Governance Checklist |
| 5 | Capstone | `/speckit-analyze` (truy vết cuối) | Opportunity Map (updated) · Responsibility Matrix (final) · Validation Workflow · Success Metrics |

### Cấu trúc thư mục

```
MedBook-SpecKit-Course/
├── KICHBAN.md                  ← file này (bản chốt)
└── medbook/                    ← GỐC DỰ ÁN (mở Claude ở đây)
    ├── context/                ← bối cảnh khởi đầu khóa học (chỉ đọc)
    │   ├── boi-canh-chung/     ← hệ thống MedBook "đầu Day 1", có dẫn chứng dòng code
    │   └── de-bai/             ← MedBook_CaseStudy_5Days (pdf + txt)
    ├── bai-nop/day1 … day5/    ← sản phẩm nộp từng ngày
    ├── src/ public/ tests/ …   ← code MedBook gốc (chưa có waiting list)
    ├── .specify/  .claude/skills/   ← Spec-Kit sinh ra ở Bước 0
    └── specs/001-…/            ← Spec-Kit sinh ra từ Day 2 (spec, plan, tasks…)
```

> ⚠️ Thư mục **không** chứa bất kỳ đáp án nào (không có spec package Day 2, không có code waiting list). Mọi thứ do bạn + AI + Spec-Kit tạo ra.

---

## BƯỚC 0 — Chuẩn bị môi trường (~15 phút)

**Yêu cầu:** `specify` ≥ 0.16, `claude` (Claude Code), Node 20+, Docker Desktop đang chạy, `git`.

🖥️ **0.1 Kiểm tra công cụ**

```bash
specify version
claude --version
node -v && docker info --format '{{.ServerVersion}}'
```

🖥️ **0.2 Vào gốc dự án, khởi tạo git (Spec-Kit dùng git để đặt tên nhánh/feature)**

```bash
cd MedBook-SpecKit-Course/medbook
git init -b main
git add -A && git commit -m "chore: baseline MedBook đầu Day 1"
```

🖥️ **0.3 Khởi tạo Spec-Kit trong thư mục hiện tại**

```bash
specify init --here --integration claude --script sh --force
git add -A && git commit -m "chore: specify init"
```

📄 Kết quả: `.specify/` và `.claude/skills/speckit-*/` (constitution, specify, clarify, plan, checklist, tasks, analyze, implement, converge, taskstoissues).

🖥️ **0.4 Chạy baseline MedBook (xác nhận hệ thống nền hoạt động)**

```bash
docker compose up db -d
npm ci
export DATABASE_URL="postgres://medbook:medbook@localhost:55432/medbook"
npm run db:migrate && npm run db:seed
npm test            # kỳ vọng: 14 test pass
npm run lint        # kỳ vọng: không lỗi
```

📄 Kết quả: **14 test xanh** — đây là "hợp đồng" Nguyên tắc I sẽ bảo vệ suốt khóa học.

🖥️ **0.5 Mở Claude Code**

```bash
claude
```

Trong phiên Claude gõ `/` và kiểm tra thấy các lệnh `speckit-*`.

---

## DAY 1 — From Traditional SDLC to AI-Driven SDLC (không code)

**Key message:** nhiều AI tool rời rạc → context rơi rụng giữa các phase. **Spec-Kit = một dòng artifact duy nhất** mà mọi phase cùng đọc/ghi.

**Chung cho cả ngày** — 🤖 nạp bối cảnh một lần:

```text
Đọc theo thứ tự: context/de-bai/MedBook_CaseStudy_5Days.txt (phần Hệ thống sẵn có + Day 1),
context/boi-canh-chung/README.md, 01-pham-vi-va-boi-canh.md, 06-thanh-phan-va-luong.md,
08-khoang-trong.md. Chưa viết gì vào file. Tóm tắt cho tôi trong 10 dòng: MedBook là gì,
đang có gì, thiếu gì, team 5 người dùng AI thế nào và đau ở đâu.
```

### 1.1 Activity 1 — SDLC Artifact Map & điểm đứt gãy

🤖
```text
Tôi đang phân tích quy trình phát triển tính năng "Quản lý danh sách chờ" cho MedBook.
Team: 1 BA (ChatGPT), 2 Backend Dev (GitHub Copilot), 1 QA (Gemini), 1 Tech Lead (ChatGPT
conversation riêng), Scrum 2 tuần/sprint. Hãy ghi vào bai-nop/day1/01-artifact-map.md:
(1) Bảng artifact theo phase Requirements → Design → Dev → Test → Ops: tên artifact, ai tạo,
ai nhận, định dạng. (2) Thông tin nào phải đi từ phase này sang phase sau. (3) Ít nhất 3 điểm
đứt gãy (context drift), mỗi điểm có: phase nào → phase nào, thông tin bị mất, vì sao mỗi
phase dùng AI tool riêng làm nó tệ hơn, hậu quả đo được (ví dụ "BA→Dev hỏi lại 3–4 lần").
(4) Sơ đồ dòng chảy dạng Mermaid. Dẫn chứng điểm đứt gãy bằng tài liệu trong context/ khi có
(ví dụ HT-GAP-21: đề bài mô tả hiện trạng khác code thật).
```

📄 `bai-nop/day1/01-artifact-map.md` · 👤 **Checkpoint:** ≥ 3 điểm đứt gãy, mỗi điểm có phase nguồn/đích.

### 1.2 Activity 2 — AI Opportunity Matrix

🤖
```text
Ghi bai-nop/day1/02-ai-opportunity-matrix.md. Bảng phase (Requirements, Design, Dev, Test,
Ops) × cột: vai trò AI cụ thể (Assistant / Copilot / Collaborator), giá trị tạo ra, điều kiện
để AI chạy tốt, rủi ro nếu context không truyền đúng, mức tác động (H/M/L). Thêm mục: pain
point nào AI KHÔNG giải được nếu context không được truyền đúng. Không được ghi chung chung
kiểu "dùng AI để tăng tốc". Cột "kỳ vọng" để Day 5 đối chiếu với thực tế.
```

📄 `02-ai-opportunity-matrix.md` · 👤 5 phase đều được điền, vai trò cụ thể.

### 1.3 Activity 3 — Human-AI Responsibility Matrix

🤖
```text
Ghi bai-nop/day1/03-responsibility-matrix.md: mỗi phase có 3 cột — AI làm gì / AI đề xuất,
Human duyệt / Chỉ Human quyết — và checkpoint review ở đâu (ít nhất 2 phase có checkpoint cụ
thể). Cuối file: nếu để AI tự chạy hoàn toàn qua một phase (chọn Requirements), rủi ro lớn
nhất là gì? Dùng ví dụ từ context/boi-canh-chung/08-khoang-trong.md.
```

### 1.4 Activity 4 — Future-State Workflow (đưa Spec-Kit vào)

Trước hết 🤖 để AI hiểu Spec-Kit đang có gì:

```text
Đọc .specify/templates/ (constitution, spec, plan, tasks, checklist) và danh sách skill trong
.claude/skills/speckit-*/SKILL.md (chỉ đọc phần mô tả và Outline). Tóm tắt: mỗi lệnh đọc gì,
ghi ra file nào, ai (Human/AI) duyệt ở đâu. Chưa chạy lệnh nào.
```

Sau đó 🤖:

```text
Ghi bai-nop/day1/04-future-state-workflow.md: quy trình mới để phát triển waiting list, dùng
chuỗi Spec-Kit làm xương sống: constitution → specify → clarify → plan → checklist → tasks →
analyze → implement → converge. Với mỗi bước: AI vai trò gì, Human duyệt gì, artifact sinh ra
(đường dẫn), thông tin handoff sang bước sau. Có sơ đồ Mermaid. Cuối file so sánh với quy trình
cũ (5 người, 4 AI tool): cải thiện ở đâu, rủi ro mới nào cần chú ý (ví dụ over-reliance,
spec trôi so với code).
```

### 1.5 Hoàn tất Day 1

🤖
```text
Tạo bai-nop/day1/README.md: checklist cuối ngày (6 mục trong đề Day 1) với trạng thái ✅/❌ và
dẫn tới file bằng chứng. Thêm 1 đoạn 5 dòng: vì sao AI giúp nhanh hơn nhưng không đồng nghĩa
SDLC hiệu quả hơn.
```

🖥️ `git add -A && git commit -m "day1: artifact map, opportunity matrix, responsibility matrix, future-state"`

---

## DAY 2 — AI-Driven Requirement Engineering & System Design (không code)

### 2.1 Constitution — luật chung của dự án

🤖 **Lệnh Spec-Kit #1**
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

📄 `.specify/memory/constitution.md` · 👤 **Checkpoint:** đủ 8 nguyên tắc, không còn `[PLACEHOLDER]`.

### 2.2 Activity 1 — Phân tích yêu cầu thô, tìm mơ hồ

Trước khi chạy `/speckit-specify`, làm Activity 1 của đề: 🤖
```text
Đây là yêu cầu thô (từ context/de-bai/MedBook_CaseStudy_5Days.txt, mục "Yêu cầu mới — xuyên
suốt 5 ngày"). Ghi bai-nop/day2/01-clarification-questions.md: (1) điều gì còn mơ hồ, (2) điều
gì còn thiếu, (3) edge case chưa được đề cập, (4) ÍT NHẤT 8 câu hỏi clarification cho
stakeholder, mỗi câu có: lý do quan trọng, ai nên trả lời, phương án mặc định đề xuất. Ví dụ
gợi ý chủ đề: X phút là bao nhiêu; "ưu tiên y tế" ai gán, mấy mức; slot mới tạo có kích hoạt
không; bệnh nhân có lịch trùng giờ; hai slot trống cùng lúc; bệnh nhân từ chối có được vào lại
hàng chờ; thông báo kênh nào. KHÔNG tự trả lời — chỉ hỏi.
```

📄 `01-clarification-questions.md` · 👤 Đây là danh sách câu hỏi của *bạn*; giữ lại để so với kết quả `/speckit-clarify`.

### 2.3 `/speckit-specify` — user story + acceptance criteria

🤖 **Lệnh Spec-Kit #2**
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

> Spec-Kit sẽ tạo nhánh/thư mục `specs/001-…/` (tên có thể khác chút), `spec.md` và `checklists/requirements.md`.

📄 `specs/001-*/spec.md` · 👤 **Checkpoint:** ≥ 3 user story, mỗi story có AC Given-When-Then; các điểm chưa chắc được gắn `[NEEDS CLARIFICATION]` (không phải AI tự quyết).

### 2.4 `/speckit-clarify` — chốt quyết định bằng Human

🤖 **Lệnh Spec-Kit #3**
```text
/speckit-clarify
```

Spec-Kit hỏi tối đa 5 câu một. **Bạn trả lời** (đây là quyết định của Human — bảng bên dưới là đáp án mặc định của kịch bản, bạn có thể đổi):

| Chủ đề khi Spec-Kit hỏi | Trả lời mặc định |
| --- | --- |
| Thời gian phản hồi X | **15 phút**, cấu hình bằng biến môi trường `OFFER_TTL_MINUTES` |
| Mức ưu tiên y tế & ai gán | 3 mức `urgent > high > normal`; **nhân viên** gán khi thêm bệnh nhân vào danh sách chờ |
| Thứ tự chọn ứng viên | Ưu tiên y tế giảm dần → thời gian vào danh sách chờ tăng dần → id tăng dần (phá hòa). Không random, không FIFO thuần |
| Điều kiện phù hợp | Cùng chuyên khoa với slot; nếu bệnh nhân chọn bác sĩ cụ thể thì phải đúng bác sĩ đó |
| Khi nào kích hoạt | Chỉ khi slot về `available` do **hủy lịch** hoặc **nhân viên mở lại slot**. Slot mới tạo / seed **không** kích hoạt |
| Lịch trùng giờ | Bệnh nhân đã có lịch trùng giờ → bỏ qua (không phạt), xét người kế tiếp |
| Kênh thông báo | Trong ứng dụng (in-app, bệnh nhân polling `GET /api/my-offers`); **không** SMS/email |
| Từ chối / hết hạn | Chuyển người kế tiếp; bệnh nhân từ chối **không** được đề xuất lại đúng slot đó; hết hạn xử lý bằng sweeper quét định kỳ |
| Lead time tối thiểu | Không tạo đề xuất nếu slot bắt đầu trong < 30 phút |
| Hết hàng chờ | Slot giữ `available` bình thường, ghi nhận để nhân viên thấy |

👤 **Checkpoint:** mỗi câu trả lời được Spec-Kit ghi vào mục `## Clarifications` trong `spec.md`.

### 2.5 Activity 2 — Review user story & acceptance criteria

🤖
```text
Đọc specs/001-*/spec.md. Review như một Requirement Reviewer: (1) mỗi user story có đủ
role/action/benefit và AC Given-When-Then không; (2) happy path vs edge case có đủ không; (3)
liệt kê những chỗ AI (spec bạn vừa sinh) viết SAI hoặc THIẾU so với context/boi-canh-chung
(đặc biệt luật sẵn có HT-BR và khoảng trống HT-GAP-03: hệ thống chưa chống đặt trùng giờ).
Ghi bai-nop/day2/02-ai-errors-log.md: mỗi dòng = lỗi/thiếu, vì sao, mức độ, đã sửa chưa.
File này sẽ dùng làm bằng chứng ở Day 4. Chưa sửa spec.md.
```

👤 **Checkpoint:** bạn đọc `02-ai-errors-log.md`, đồng ý/không đồng ý từng dòng, rồi 🤖:

```text
/speckit-specify cập nhật spec.md hiện có: áp dụng các mục tôi đã đồng ý trong
bai-nop/day2/02-ai-errors-log.md (đánh dấu mục nào là "Accepted"). Không thêm business rule mới
ngoài những gì đã chốt ở phần Clarifications; nếu thiếu thì để [NEEDS CLARIFICATION].
```

### 2.6 Activity 3 — `/speckit-plan`: 2+ phương án kiến trúc rồi chọn

🤖 **Lệnh Spec-Kit #4**
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

📄 `specs/001-*/{plan.md, research.md, data-model.md, contracts/, quickstart.md}` · 👤 **Checkpoint:** research.md có ≥ 2 phương án + trade-off; plan.md có mục *Constitution Check* đạt.

🤖 Đóng gói **Architecture Blueprint** (deliverable của đề):
```text
Tổng hợp từ plan.md, research.md, data-model.md, contracts/ vào bai-nop/day2/04-architecture-blueprint.md:
sơ đồ kiến trúc high-level (Mermaid), danh sách component THÊM / SỬA trên codebase MedBook hiện
có (đường dẫn file cụ thể), quyết định công nghệ, trade-off analysis, rủi ro chính (đặc biệt
đồng thời: hai người cùng accept, sweeper chạy song song) và cách giảm. Chỉ tổng hợp, không bịa
thêm quyết định mới.
```

### 2.7 Activity 4 — Requirement Review & Gap Analysis

🤖 **Lệnh Spec-Kit #5**
```text
/speckit-checklist Chất lượng yêu cầu của waiting list: đầy đủ, rõ ràng, nhất quán, đo được; tập
trung vào luật chọn ứng viên, vòng đời offer (pending/accepted/declined/expired), đồng thời,
thông báo, quyền sở hữu dữ liệu.
```

Rồi 🤖:
```text
Review toàn bộ spec.md + plan.md + data-model.md + contracts/ + checklists/. Ghi
bai-nop/day2/03-requirement-review-notes.md: (1) user story/AC nào mâu thuẫn nhau; (2) gap;
(3) đối chiếu kiến trúc đã chọn: yêu cầu nào khó/không khả thi; (4) kiểm tra từng nguyên tắc
I–VIII của constitution có bị vi phạm không. Với mỗi phát hiện: mức độ và đề xuất sửa. Sau đó
sửa spec.md/plan.md cho những phát hiện mức Critical/High, ghi rõ đã sửa gì.
```

### 2.8 Gate Day 2 & chốt phiên bản

🤖
```text
Tạo bai-nop/day2/README.md: checklist cuối ngày (7 mục đề Day 2) ✅/❌ + dẫn chứng; liệt kê các
câu hỏi mở còn lại (nếu có) — không tự trả lời.
```

🖥️
```bash
git add -A && git commit -m "day2: constitution, spec, clarifications, plan, checklist, blueprint"
git tag day2-frozen
```

> `day2-frozen` là **đầu vào bắt buộc** của Day 3. Sau mốc này sửa spec phải theo Nguyên tắc VII (sửa tài liệu trước, ghi lý do).

---

## DAY 3 — AI-Driven Development & Quality Engineering (ngày duy nhất chạm code)

### 3.1 `/speckit-tasks`

🤖 **Lệnh Spec-Kit #6**
```text
/speckit-tasks Chia theo user story để mỗi story có thể kiểm thử độc lập. Thứ tự: migration +
repository → Offer Engine (matching, tạo offer) → hook sau commit ở hủy lịch / mở lại slot →
accept/decline + xử lý conflict → sweeper timeout → API my-offers/waiting-list → giao diện.
Mỗi task có đường dẫn file và trỏ về AC tương ứng. Có task test cho từng AC, gồm test đồng thời.
```

📄 `specs/001-*/tasks.md`

### 3.2 `/speckit-analyze` — cổng trước khi code

🤖 **Lệnh Spec-Kit #7**
```text
/speckit-analyze
```

👤 **Checkpoint (hard gate):** không còn finding CRITICAL. Có thì sửa spec/plan/tasks trước (không sửa code ngầm), rồi chạy `/speckit-analyze` lại.

### 3.3 Chuẩn bị môi trường code

🖥️
```bash
git checkout -b feature/waitlist
docker compose up db -d
export DATABASE_URL="postgres://medbook:medbook@localhost:55432/medbook"
npm test    # vẫn 14 pass trước khi động vào code
```

### 3.4 Activity 1 — Service Implementation (`/speckit-implement`, chia đợt)

Chia theo đợt nhỏ để Human review từng đợt (Human-in-the-loop):

🤖 **Đợt 1 — nền dữ liệu & matching**
```text
/speckit-implement Chỉ làm các task migration, repository và Offer Engine (matching + tạo offer)
trong tasks.md. Viết code cùng style/convention codebase hiện có (xem doc/backend-flows.md và
src/services/appointmentService.js). Tuân thủ constitution: SQL tham số hóa, transaction + khóa
dòng, không thêm dependency. Đánh dấu [X] task đã xong. Dừng sau đợt này để tôi review.
```

👤 Review diff (`git diff`), chạy `npm test` + `npm run lint`. Rồi:

🤖 **Đợt 2 — hook & phản hồi**
```text
/speckit-implement Làm tiếp các task: hook sau commit ở hủy lịch và mở lại slot; accept/decline;
xử lý conflict lịch và race condition khi hai bệnh nhân cùng accept. Dừng sau đợt này.
```

🤖 **Đợt 3 — timeout, API, giao diện**
```text
/speckit-implement Làm tiếp: sweeper timeout (setInterval, đọc DB mỗi lượt, cấu hình bằng env),
endpoint waiting-list và my-offers, giao diện tối thiểu (staff thêm bệnh nhân vào danh sách chờ;
bệnh nhân thấy thẻ đề xuất có đếm ngược, nút chấp nhận/từ chối).
```

👤 **Checkpoint:** `npm test` xanh, `npm run lint` sạch, đủ ≥ 2/3 component chính (matching, notification, timeout) — mục tiêu là cả 3.

### 3.5 Activity 2 — Unit Test từ Acceptance Criteria

🤖
```text
Đọc AC trong spec.md và các task test trong tasks.md. Viết test (node --test, cùng style
tests/api.test.js) trong tests/waitlist.test.js: mỗi test có comment "AC-x.y" trỏ về tiêu chí.
Phủ: happy path (có ứng viên và accept), edge case (hàng chờ trống, tất cả từ chối, lịch trùng,
slot < 30 phút, hết hạn), timeout, và test ĐỒNG THỜI (hai bệnh nhân accept cùng lúc → đúng 1
thành công). Chạy `npm test`. Ghi bai-nop/day3/03-testing-package.md: bảng AC → test (còn AC
nào chưa có test?), test pass/fail và nguyên nhân, test nào AI viết đúng, test nào phải sửa và
vì sao, báo cáo coverage (`node --test --experimental-test-coverage tests/*.test.js`).
```

### 3.6 Activity 3 — Code Review & Security Check

🤖
```text
Review diff của nhánh feature/waitlist so với main (git diff main). Chỉ ra: (1) code quality
(naming, độ phức tạp, trùng lặp); (2) bug/logic error tiềm ẩn; (3) lỗ hổng bảo mật — đặc biệt
SQL injection, kiểm tra QUYỀN SỞ HỮU offer (bệnh nhân A có accept được offer của B không?),
race condition khi nhiều bệnh nhân nhận cùng slot, sweeper chạy chồng; (4) điểm có thể thành vấn
đề hiệu năng khi lớn. Đối chiếu từng nguyên tắc I–VIII. Ghi bai-nop/day3/04-code-review-report.md:
≥ 3 issue, mỗi issue có severity (Critical/Major/Minor), file:dòng, đề xuất, và phân loại FIX NOW
hay TECH DEBT kèm lý do. Nêu rõ cả những gì bạn KHÔNG chắc hoặc không kiểm tra được.
```

👤 **Checkpoint:** bạn quyết định issue nào fix ngay. Rồi 🤖: `Sửa các issue tôi chọn FIX NOW: <liệt kê>. Thêm test cho từng issue đã sửa.`

### 3.7 `/speckit-converge` — kiểm tra còn thiếu gì so với spec

🤖 **Lệnh Spec-Kit #8**
```text
/speckit-converge
```
Nếu nó thêm task mới → `/speckit-implement` để hoàn tất, chạy lại `/speckit-analyze` (không còn CRITICAL).

### 3.8 Nghiệm thu bằng tay & đóng gói

🖥️
```bash
npm test && npm run lint
docker compose up --build -d
# mở http://localhost:4300 — luồng: staff mai.staff@medbook.local thêm an@ vào hàng chờ (urgent);
# linh@ đặt một slot rồi hủy → an@ thấy thẻ đề xuất → Chấp nhận → thấy lịch của mình.
```

🤖
```text
Tạo bai-nop/day3/01-development-package.md (API mới, quyết định implementation chính,
file đã thêm/sửa, cách chạy) và bai-nop/day3/README.md (checklist cuối ngày đề Day 3 ✅/❌ +
dẫn chứng; ghi lại: AI sinh đúng gì, cần sửa gì, vì sao).
```

🖥️ `git add -A && git commit -m "day3: waiting list implemented + tests + review" && git tag day3-done`

---

## DAY 4 — Human-AI Collaboration & Governance (không code)

**Kịch bản sự cố (đề Day 4):** sau 2 ngày production, có bệnh nhân nhận đề xuất từ bác sĩ sai chuyên khoa, và nhận thông báo liên tục dù đã có lịch; nguyên nhân ban đầu: matching logic sai ở một edge case AC mà test không phủ.

### 4.1 Tái hiện sự cố trên code thật (bằng chứng, không suy diễn)

🤖
```text
Đóng vai AI-Assisted Auditor, KHÔNG sửa code. Với 2 triệu chứng: (a) bệnh nhân nhận đề xuất từ
bác sĩ/chuyên khoa sai; (b) bệnh nhân nhận đề xuất lặp dù đã có lịch trùng giờ — viết thử các
test thăm dò trong tests/probe-day4.test.js (không commit vào bộ test chính) để xem code hiện
tại có tái hiện không. Báo cáo trung thực: tái hiện được / không tái hiện được / không kết luận
được, kèm file:dòng và đầu ra test. Không kết luận "an toàn" nếu không có bằng chứng.
```

> Nếu code không tái hiện lỗi, đó là kết quả hợp lệ: phần phân tích tiếp tục dưới dạng *"nếu lỗi này xảy ra thì vì sao test hiện tại không bắt được"* (khoảng trống phủ AC).

### 4.2 Activity 1 — AI Failure Analysis

🤖
```text
Dùng bai-nop/day2/02-ai-errors-log.md, spec.md (AC), tests/waitlist.test.js, bai-nop/day3/04-code-review-report.md
và kết quả 4.1. Ghi bai-nop/day4/01-failure-analysis-report.md: timeline sự cố; truy từng bước
Day 2→3: requirement mơ hồ ở đâu, test thiếu ở đâu, review bước nào đáng lẽ phải bắt được (dẫn
file:dòng/ID AC); phân loại nguyên nhân gốc vào 5 nhãn (Spec sai/thiếu · AI sinh sai · Test thiếu ·
Review bỏ sót · Process không có checkpoint) với tỉ trọng; kết luận đây là lỗi AI hay lỗi
Human-AI process và vì sao; risk assessment nếu không phát hiện sớm; bài học. Mọi kết luận phải
có bằng chứng; chỗ không chắc ghi [CẦN XÁC NHẬN].
```

### 4.3 Activity 2 — Human-AI Review Workflow

🤖
```text
Ghi bai-nop/day4/02-human-ai-review-workflow.md: checkpoint còn thiếu trong Day 1–3; workflow
review mới có ≥ 3 validation checkpoint (ai review gì, khi nào, tiêu chí); phân biệt rõ chỗ
nào Human-in-the-loop (review từng output) và chỗ nào Human-on-the-loop (batch/trigger); chỉ rõ
checkpoint nào là HARD GATE chặn merge. Ánh xạ mỗi checkpoint sang lệnh Spec-Kit tương ứng
(/speckit-analyze, /speckit-checklist, /speckit-converge…). Sơ đồ Mermaid.
```

### 4.4 Activity 3 — AI Governance Checklist

🤖
```text
Ghi bai-nop/day4/03-ai-governance-checklist.md: ≥ 8 mục, phân theo loại risk (hallucination,
security, incomplete output, over-reliance); mỗi mục có: câu kiểm chứng được (Yes/No), mitigation,
người chịu trách nhiệm, cách kiểm chứng (lệnh/file). Phải đủ cụ thể để dùng trước khi merge
code do AI sinh vào main. Ví dụ mục: "mọi AC đều có test trỏ về", "không có dependency runtime
mới", "không business rule nào thiếu mã nguồn quyết định".
```

### 4.5 Đưa governance vào Spec-Kit (không để thành tài liệu nằm chết)

🤖 **Lệnh Spec-Kit #9**
```text
/speckit-constitution Nâng lên v1.1.0: thêm mục "Governance cho mã do AI sinh" từ 3 hard-gate
trong bai-nop/day4/02-human-ai-review-workflow.md và 5 mục quan trọng nhất của
bai-nop/day4/03-ai-governance-checklist.md. Ghi Sync Impact Report. Không đổi nguyên tắc I–VIII.
```

🤖 **Lệnh Spec-Kit #10**
```text
/speckit-checklist Pre-merge cho mã do AI sinh của waiting list, dựa trên constitution v1.1.0.
```

🤖 **Lệnh Spec-Kit #11** — audit trôi lệch spec ↔ code:
```text
/speckit-analyze Tập trung: AC nào chưa có test; chỗ nào code làm khác spec; business rule nào
thiếu nguồn quyết định.
```

🤖
```text
Tạo bai-nop/day4/README.md: checklist cuối ngày (7 mục đề Day 4) ✅/❌ + dẫn chứng; trả lời
trong 5 dòng: lỗi này là lỗi AI hay lỗi process, vì sao.
```

🖥️ `git add -A && git commit -m "day4: failure analysis, review workflow, governance (constitution v1.1.0)" && git tag day4-done`

---

## DAY 5 — Capstone: Designing AI-Driven SE Workflows

### 5.1 Activity 1 — Review toàn bộ artifact

🤖
```text
Lập bai-nop/day5/01-artifact-review.md: bảng mọi artifact của Day 1–4 (đường dẫn, phase, AI đóng
vai gì thực tế, chất lượng đánh giá theo bằng chứng). Artifact nào AI support tốt, artifact nào
AI tạo ra vấn đề (dẫn từ Day 4). So sánh với kế hoạch Day 1 (bai-nop/day1/02-ai-opportunity-matrix.md): khác ở đâu.
Chạy `git log --oneline` và `ls specs/001-*` để dùng số liệu thật, không ước chừng.
```

### 5.2 AI Opportunity Map (cập nhật)

🤖
```text
Ghi bai-nop/day5/02-ai-opportunity-map-updated.md: lấy bảng Day 1 làm nền, thêm cột "Thực tế sau
4 ngày" và "Bằng chứng" (đường dẫn). Phân biệt rõ kỳ vọng vs thực tế; đánh dấu vai trò AI nào có
giá trị, vai trò nào gây overhead.
```

### 5.3 Responsibility Matrix (bản chốt)

🤖
```text
Ghi bai-nop/day5/03-responsibility-matrix-final.md: cập nhật từ Day 1 bằng bài học thực tế
(dùng bai-nop/day4/01-failure-analysis-report.md và các quyết định ở Clarify). Với mỗi phase:
AI làm / AI đề xuất–Human duyệt / chỉ Human quyết / checkpoint. Đề xuất mô hình collaboration tối
ưu cho team MedBook.
```

### 5.4 AI Validation Workflow & Quality Gate

🤖
```text
Ghi bai-nop/day5/04-ai-validation-workflow.md: sơ đồ Mermaid đầy đủ từ requirement đến deployment
với quality gate ở cả 4 chuyển phase (Requirements→Design, Design→Dev, Dev→Test, Test→Deploy).
Mỗi gate: tiêu chí, người approve, AI hỗ trợ kiểm tra được không (và bằng lệnh Spec-Kit nào, ví
dụ /speckit-checklist, /speckit-analyze, /speckit-converge, `npm test`, `npm run lint`).
Dùng specs/001-*/ làm ví dụ truy vết 2 chiều requirement ↔ design ↔ task ↔ test.
```

### 5.5 Success Metrics Proposal

🤖
```text
Ghi bai-nop/day5/05-success-metrics-proposal.md: ≥ 4 metric (lead time, defect rate, review
effort, documentation quality…), mỗi metric có baseline (trước AI), target (sau AI), cách đo cụ
thể và nguồn dữ liệu. Baseline dùng số liệu có trong đề (ví dụ review tăng gấp đôi, BA hỏi lại
3–4 lần); chỗ không có số liệu thì ghi rõ "giả định" chứ không bịa. Thêm mục: đo "chất lượng
artifact do AI sinh" khách quan bằng cách nào (ví dụ % AC có test, số finding của analyze).
```

### 5.6 Retrospective & đóng gói

🤖
```text
Tôi vừa hoàn thành dự án nhỏ dùng AI xuyên SDLC (Requirements → Design → Development → Testing
→ Governance) với Spec-Kit. Dựa trên toàn bộ bai-nop/ và specs/: (1) phase nào AI support hiệu
quả nhất, vì sao; (2) phase nào AI gây thêm vấn đề, vì sao; (3) Spec-Kit giải quyết được điểm đứt
gãy nào của Day 1 và chưa giải quyết được điểm nào; (4) nếu làm lại nên đổi gì. Ghi
bai-nop/day5/06-retrospective.md và bai-nop/day5/README.md (checklist cuối ngày 7 mục ✅/❌ + mục
lục toàn bộ artifact 5 ngày).
```

🖥️ `git add -A && git commit -m "day5: capstone" && git tag day5-done`

**Kiểm tra cuối:** `npm test` xanh · `npm run lint` sạch · mỗi `bai-nop/dayN/README.md` có checklist · `specs/001-*/` có spec/plan/tasks · `.specify/memory/constitution.md` là v1.1.0.

---

## PHỤ LỤC — Kế hoạch chạy thử (sau khi bạn duyệt)

| Việc | Cách làm |
| --- | --- |
| Chạy từng bước | Một bản sao sạch của thư mục; mỗi prompt chạy bằng `claude -p "<prompt>" --permission-mode acceptEdits` (headless, cùng phiên bằng `--continue`) |
| Chỗ Human trả lời | `/speckit-clarify` là tương tác: khi chạy thử tôi sẽ nạp sẵn đáp án ở bảng 2.4 vào prompt |
| Kiểm tra mỗi bước | Có đúng file 📄 sinh ra; `npm test` / `npm run lint`; không còn placeholder; không đụng 14 test cũ |
| Sửa kịch bản | Bước nào lỗi (tên lệnh sai, prompt dài, AI không ghi file…) → sửa prompt/lệnh trong kịch bản → chạy lại bước đó |
| Báo cáo | Cuối cùng: danh sách bước đã chạy, thay đổi so với draft, các chỗ không thể tự động hóa |

---

## PHỤ LỤC B — Lớp quay VIDEO (một video xuyên suốt)

**Mục tiêu:** 1 video liên tục. **Bản dùng thật là Phụ lục C (~18 phút).** Phụ lục B bên dưới là bản đầy đủ tham khảo, **ước tính 35–45 phút** sau khi dựng (thời gian thật khoảng 3–5 giờ; các số dưới đây là ƯỚC LƯỢNG, sẽ thay bằng số đo thật sau khi chạy thử).

### Quy tắc quay/dựng

| Loại cảnh | Xử lý |
| --- | --- |
| AI đang sinh file / chạy `/speckit-*` | **Tua 8–16x**, giữ lại 3 giây đầu (thấy lệnh chạy) và khung cuối (thấy kết quả) |
| `npm ci`, `docker compose up --build`, `npm test` dài | Tua 16x, dừng ở dòng kết quả (`14 pass`, `MedBook đang chạy…`) |
| Prompt dài | Dán sẵn từ file; trên video **chỉ hiện 2–3 dòng đầu** + chú thích chữ tên prompt; prompt đầy đủ đặt ở mô tả video/KICHBAN.md |
| Mở file kết quả (spec.md, tasks.md, báo cáo…) | Giữ **thời gian thật 5–10 giây**, cuộn nhanh, zoom vào phần quan trọng (ID AC, Clarifications, Constitution Check) |
| `/speckit-clarify` | Giữ thời gian thật (có hỏi đáp), cắt đoạn chờ giữa các câu |
| Human Checkpoint (👤) | Hiện chữ **"👤 Human duyệt"** trên màn hình 2–3 giây, cho thấy bạn đọc/chấp nhận; không cần quay cả quá trình đọc |
| Day 1, Day 5 (prompt thường) | Mỗi prompt: dán → tua → lướt file kết quả 5 giây |

### Phân bổ thời lượng dự kiến trên video

| Đoạn | Nội dung | Số prompt | Thời lượng video |
| --- | --- | ---: | ---: |
| Mở đầu | Vấn đề context drift + chuỗi Spec-Kit (slide/giọng nói) | 0 | 2:00 |
| Bước 0 | init, baseline 14 test xanh, mở Claude | 0 | 2:30 |
| Day 1 | 4 activity + hiểu cấu trúc Spec-Kit | ~6 | 4:00 |
| Day 2 | constitution, specify, clarify, review, plan, checklist | ~12 | 10:00 |
| Day 3 | tasks, analyze, implement ×3, test, review, converge, demo UI | ~10 | 12:00 |
| Day 4 | tái hiện sự cố, failure analysis, workflow, governance, constitution v1.1 | ~8 | 6:00 |
| Day 5 | tổng hợp, matrix, workflow, metrics, retro | ~7 | 4:00 |
| Kết | Bài học + Spec-Kit giải/chưa giải được gì | 0 | 1:30 |
| **Tổng** | | **~43** | **≈ 42:00** |

> Nếu muốn ngắn hơn (~30 phút): rút Day 1/Day 5 còn "dán → hiện file kết quả 3 giây", chỉ giữ thời gian thật ở Day 2 (clarify, plan) và Day 3 (analyze, test xanh, demo UI).

### Mẹo quay để dựng ít phải cắt

1. **Quay màn hình liên tục**, không dừng giữa các bước; đánh dấu mốc bằng cách gõ `# --- DAY 2 ---` trong terminal để dễ tìm điểm cắt.
2. **Chuẩn bị sẵn file `prompts/`** (mỗi prompt một file `.txt`) và dán bằng một phím để không gõ sai trên video.
3. **Phóng chữ terminal** (≥ 18pt), mở sẵn 2 pane: terminal + `git diff`/file kết quả.
4. **Đồng hồ**: bật bộ đếm trên màn hình để người xem thấy thời gian thật trôi qua trong lúc tua.
5. **Quay thử toàn bộ một lần** (chính là lần chạy thử kịch bản) để lấy thời gian thật trước khi quay chính thức.
6. Khi AI sinh khác lần chạy thử, **không cắt bỏ** kết quả lạ; nếu sai thì sửa prompt rồi quay lại bước đó.

### Việc tôi sẽ làm thêm khi chạy thử (phục vụ quay video)

- Đo **thời gian thật** từng bước và điền vào cột `⏱` của từng bước.
- Gắn nhãn `▶ tua` / `⏸ giữ thật` cho từng bước dựa trên số đo.
- Xuất thư mục `prompts/` (mỗi prompt một file, đánh số theo thứ tự quay) để dán khi quay.
- Xuất `KICHBAN-VIDEO.md`: bảng cảnh quay theo thứ tự, mỗi dòng gồm bước · lệnh/prompt · màn hình lên hình · tua/giữ · lời dẫn gợi ý.

---

## PHỤ LỤC C — BẢN QUAY RÚT GỌN (~18 phút, mục tiêu: học viên hiểu LUỒNG sử dụng)

**Nguyên tắc:** video chỉ quay **chuỗi lệnh Spec-Kit cốt lõi**, mỗi lệnh đúng **một lần**. Mọi bước lặp lại (cùng kiểu prompt, nhiều deliverable, nhiều đợt implement) **không quay**: chúng đã được chạy sẵn trước giờ quay (hoặc học viên tự làm theo kịch bản đầy đủ), trên video chỉ lướt qua danh sách file kết quả. Phụ lục B (quy tắc tua/giữ) vẫn áp dụng cho các cảnh quay.

Ký hiệu: ★ = quay lên hình · ○ = không quay, chạy sẵn / nói bằng lời.

### Luồng chính được quay

| # | Đoạn | Việc trên màn hình | Bước kịch bản | Video |
| ---: | --- | --- | --- | ---: |
| 1 | Mở đầu | Slide 1 tấm: vấn đề context drift → chuỗi Spec-Kit `constitution → specify → clarify → plan → checklist → tasks → analyze → implement → converge` | — | 1:30 |
| 2 | Bước 0 | ★ `specify init --here --integration claude`; ★ mở cây thư mục `.specify/` + `.claude/skills/`; ★ gõ `/` thấy các lệnh `speckit-*`. ○ baseline 14 test (chạy sẵn, chỉ chụp kết quả xanh 3 giây) | 0.3–0.5 | 1:30 |
| 3 | Day 1 | Nói bằng lời: 4 artifact (Artifact Map, Opportunity Matrix, Responsibility Matrix, Future-State). ★ Chỉ **một** prompt: Future-State Workflow (1.4) → hiện sơ đồ Mermaid. ○ 3 activity còn lại: lướt 4 file trong `bai-nop/day1/` | 1.4 | 1:15 |
| 4 | Constitution | ★ `/speckit-constitution` → mở `constitution.md`, zoom 8 nguyên tắc | 2.1 | 1:00 |
| 5 | Specify | ★ `/speckit-specify` → mở `spec.md`: user story, AC Given-When-Then, chỗ `[NEEDS CLARIFICATION]` | 2.3 | 1:30 |
| 6 | Clarify | ★ `/speckit-clarify` → trả lời 2–3 câu **thật**, cắt phần còn lại; zoom mục `## Clarifications` | 2.4 | 1:45 |
| 7 | Plan | ★ `/speckit-plan` → mở `research.md` (3 phương án + chọn B), lướt `data-model.md`, `contracts/` | 2.6 | 1:30 |
| 8 | Checklist | ★ `/speckit-checklist` → mở checklist, chỉ ra 1–2 mục fail rồi sửa spec (nói: "đây là bước review requirement") | 2.7 | 1:00 |
| 9 | Tasks | ★ `/speckit-tasks` → mở `tasks.md`: phase theo story, ID task, đường dẫn file, AC tham chiếu | 3.1 | 1:00 |
| 10 | Analyze (cổng) | ★ `/speckit-analyze` → bảng finding; sửa 1 CRITICAL nếu có; nhấn mạnh "hard gate trước khi code" | 3.2 | 1:15 |
| 11 | Implement | ★ `/speckit-implement` **đợt 1** (tua) → `git diff --stat`, task chuyển `[X]`. ○ đợt 2, 3: nói "làm tương tự", hiện sẵn diff cuối | 3.4 | 1:30 |
| 12 | Test & Review | ★ `npm test` → toàn bộ xanh (14 cũ + test mới); ★ lướt bảng AC → test. ○ viết test, code review: mở sẵn `code-review-report.md` 5 giây | 3.5–3.6 | 1:15 |
| 13 | Converge | ★ `/speckit-converge` → kết quả "không còn gap" hoặc task bổ sung | 3.7 | 0:45 |
| 14 | Demo UI | ★ Staff thêm bệnh nhân vào hàng chờ → hủy lịch → bệnh nhân thấy thẻ đề xuất → chấp nhận (cắt gọn, ~45 giây) | 3.8 | 0:45 |
| 15 | Day 4 | Nói bằng lời sự cố (sai chuyên khoa). ★ **Một** prompt: Failure Analysis (4.2) → mở bảng nguyên nhân gốc; ★ `/speckit-constitution` nâng v1.1.0 (4.5) → chỉ Sync Impact Report. ○ Review Workflow + Governance Checklist: lướt tên file | 4.2, 4.5 | 1:30 |
| 16 | Day 5 + kết | ★ Hiện sơ đồ AI Validation Workflow (đã sinh sẵn) + bảng Success Metrics; ○ 4 deliverable còn lại không quay. Tổng kết: Spec-Kit giải/chưa giải được gì | 5.4 | 1:15 |
| | | | **Tổng** | **≈ 19:00** |

> Cần xuống đúng 18 phút: bỏ cảnh 8 (checklist) và nói gộp vào cảnh 10, hoặc rút cảnh 3 còn 0:45.

### Bước KHÔNG quay (nhưng vẫn nằm trong kịch bản đầy đủ và được chạy thử)

| Bước | Lý do bỏ khỏi video |
| --- | --- |
| 1.1–1.3, 1.5 (3 activity Day 1 + README) | Cùng kiểu "prompt → file markdown"; một ví dụ (1.4) là đủ |
| 2.2 (câu hỏi clarification thủ công), 2.5 (review AC), blueprint, review notes, README Day 2 | Lặp lại mô hình "review rồi sửa"; `/speckit-checklist` đã đại diện |
| 3.3 (chuẩn bị nhánh/DB), 3.4 đợt 2–3, 3.5 viết test, 3.6 sửa review, 3.8 đóng gói | Lặp lại implement; chỉ quay kết quả (test xanh, UI) |
| 4.1 (probe), 4.3, 4.4, 4.5 checklist/analyze | Prompt thường ghi ra file; đại diện bằng 4.2 + constitution v1.1.0 |
| 5.1–5.3, 5.5, 5.6 | Tổng hợp từ artifact có sẵn; chỉ cho xem 5.4 |

### Chuẩn bị trước giờ quay

1. Chạy kịch bản đầy đủ **trước một lần** trên một bản sao; lưu lại artifact của các bước ○.
2. Với cảnh ★, quay trên một bản sao **sạch tới ngay trước bước đó** (dùng tag `day2-frozen`, `day3-done` và commit từng bước để quay lại đúng điểm).
3. Mỗi cảnh ★ có file `prompts/NN-ten.txt` để dán.

---

## CẦN BẠN DUYỆT / QUYẾT ĐỊNH

1. **Khung chung:** Day 1 chưa dùng lệnh Spec-Kit nào (đề Day 1 không có code/spec) — chỉ dùng để hiểu cấu trúc và đưa vào Future-State. Ổn chứ?
2. **Đáp án Clarify (bảng 2.4):** tôi lấy theo hướng của bộ spec mẫu trong repo (15 phút, 3 mức ưu tiên, in-app, kiến trúc in-process B). Bạn muốn giữ, hay đổi quyết định nào (ví dụ X phút, SMS)?
3. **Phương án kiến trúc 2.6:** prompt đang *dẫn dắt* chọn B. Bạn muốn để AI tự chọn (nội dung "thật" hơn nhưng kết quả khó đoán), hay giữ dẫn dắt (kịch bản lặp lại được)?
4. **Day 4:** nếu code đúng thì không tái hiện được sự cố. Giữ như thiết kế (báo cáo trung thực), hay muốn **cố ý chèn lỗi** vào code (ví dụ bỏ điều kiện chuyên khoa) để có sự cố thật?
5. **Dọn context:** trong `context/boi-canh-chung/` còn ~14 chỗ nhắc `WB-2/bai-nop` và nhánh `day03` (không tồn tại ở đây). Tôi sẽ xóa các tham chiếu này khi chốt — đồng ý?
6. **Chạy thử:** mất nhiều lượt gọi Claude (Day 3 `/speckit-implement` tốn nhất). OK chạy toàn bộ, hay chỉ chạy tới Day 3 trước?
7. **Video (Phụ lục C):** bản quay rút gọn ~18–19 phút gồm 16 cảnh, bỏ các bước lặp lại. Danh sách cảnh ★ và bước ○ đã đúng ý chưa?
