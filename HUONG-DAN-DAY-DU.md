# HƯỚNG DẪN ĐẦY ĐỦ — Thêm chức năng mới cho MedBook bằng Spec-Kit

Tài liệu một file: cài đặt → từng bước (chức năng, prompt, kết quả, điểm kiểm tra). Làm từ trên xuống dưới.

## 1. Tổng quan

**Mục tiêu:** thêm chức năng **Quản lý linh hoạt danh sách chờ (waiting list)** vào MedBook đang chạy, theo quy trình Spec-Driven Development của Spec-Kit.

**Chuỗi Spec-Kit:**

```
constitution → specify → clarify → plan → checklist → tasks → analyze → implement → converge
 (luật chung)  (đặc tả)  (làm rõ)  (kế hoạch) (soát yêu cầu) (chia việc) (soát nhất quán) (code) (đối chiếu)
```

| Lệnh | Chức năng | Ghi ra |
| --- | --- | --- |
| `/speckit-constitution` | Nguyên tắc bất biến của dự án | `.specify/memory/constitution.md` |
| `/speckit-specify` | Đặc tả: user story, yêu cầu, AC Given-When-Then | `specs/001-…/spec.md` |
| `/speckit-clarify` | Hỏi ≤ 5 câu để hết mơ hồ, ghi lại vào spec | `spec.md` (mục Clarifications) |
| `/speckit-plan` | Kế hoạch kỹ thuật, so sánh kiến trúc, dữ liệu, API | `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` |
| `/speckit-checklist` | Kiểm tra CHẤT LƯỢNG yêu cầu | `checklists/*.md` |
| `/speckit-tasks` | Chia việc theo user story, có ID và đường dẫn file | `tasks.md` |
| `/speckit-analyze` | Chỉ đọc: tìm bất nhất giữa spec/plan/tasks/constitution | báo cáo trong chat |
| `/speckit-implement` | Thực thi task, tick `[X]` | code + `tasks.md` |
| `/speckit-converge` | So code với spec, thêm task cho phần còn thiếu | `tasks.md` |

**Prompt thường** (không có `/speckit-…`) dùng Claude để tạo tài liệu phụ, lưu vào `bai-nop/`.

**Nguyên tắc làm việc:** AI đề xuất, **bạn quyết định** (👤). Kết quả AI mỗi lần khác nhau — đọc câu hỏi/báo cáo thật rồi trả lời, đừng dán máy móc. Thời gian AI sinh mỗi lệnh Spec-Kit khoảng 3–15 phút (implement lâu hơn).

## 2. Cấu trúc thư mục

```
MedBook-SpecKit-Course/
├── HUONG-DAN-DAY-DU.md        ← file này
└── medbook/                   ← GỐC DỰ ÁN (mở terminal và Claude ở đây)
    ├── context/
    │   ├── yeu-cau-chuc-nang-moi.md   ← ĐẦU VÀO: mô tả chức năng mới
    │   └── boi-canh-chung/            ← hệ thống MedBook hiện có (chỉ đọc)
    ├── bai-nop/day2 day3 day4/        ← tài liệu phụ do prompt thường tạo
    ├── src/ public/ tests/ …          ← code MedBook (chưa có waiting list)
    ├── .specify/ .claude/skills/      ← Spec-Kit tạo ra ở bước 0.2
    └── specs/001-waiting-list/        ← Spec-Kit tạo ra khi chạy specify
```

## 3. Đầu vào: mô tả chức năng mới

File `medbook/context/yeu-cau-chuc-nang-moi.md` (đầu vào thô, còn mơ hồ có chủ ý):

> # Yêu cầu chức năng mới — Quản lý linh hoạt danh sách chờ (waiting list)
>
> > Đây là **đầu vào thô** của buổi đầu tiên. Chưa phải đặc tả: còn mơ hồ và thiếu có chủ ý, việc của bạn (cùng Spec-Kit) là làm rõ.
>
> ## Bối cảnh
>
> MedBook là nền tảng web quản lý lịch khám bệnh viện, đã chạy được (xem `boi-canh-chung/`).
>
> Phía bệnh nhân đã có: tìm bác sĩ theo tên/chuyên khoa; xem lịch làm việc và slot còn trống; đặt lịch khám (trực tiếp hoặc online); hủy lịch hẹn.
>
> Phía nhân viên đã có: quản lý lịch làm việc (thêm, điều chỉnh slot); xem lịch hẹn theo ngày; xác nhận hoặc hủy lịch hẹn.
>
> ## Vấn đề
>
> Hệ thống chưa có cơ chế nào xử lý khi một slot khám bị trống đột ngột (ví dụ bệnh nhân hủy lịch). Nhân viên đang xử lý thủ công hoàn toàn: gọi điện hoặc nhắn tin cho từng bệnh nhân trong danh sách chờ.
>
> ## Yêu cầu mới
>
> Bệnh viện muốn phát triển tính năng **Quản lý linh hoạt danh sách chờ**:
>
> - Khi một slot khám trở nên khả dụng, hệ thống tự động xác định bệnh nhân phù hợp nhất trong danh sách chờ.
> - Tiêu chí chọn: mức độ ưu tiên y tế, thời gian chờ, sự phù hợp chuyên khoa / bác sĩ.
> - Hệ thống gửi đề xuất cho bệnh nhân → bệnh nhân phản hồi trong X phút.
> - Nếu từ chối hoặc không phản hồi → chuyển sang bệnh nhân tiếp theo.
> - Xử lý xung đột lịch, thông báo đầy đủ cho tất cả các bên.

## 4. Bước 0 — Cài đặt và chuẩn bị

### 0.1 Cài công cụ

| Công cụ | Cài / kiểm tra |
| --- | --- |
| `uv` | macOS: `brew install uv` hoặc `curl -LsSf https://astral.sh/uv/install.sh \| sh`; kiểm tra `uv --version` |
| **Spec-Kit** | `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v0.16.2` ; kiểm tra `specify version` |
| Claude Code | `npm install -g @anthropic-ai/claude-code` ; kiểm tra `claude --version`, đăng nhập một lần bằng `claude` |
| Node 20+, git | `node -v`, `git --version` |
| PostgreSQL | Docker Desktop **hoặc** PostgreSQL cài sẵn |

> Đã kiểm chứng với `specify` v0.16.2 (lệnh dạng `/speckit-*`); bản khác có thể đặt tên lệnh khác. Lệnh cài `uv`/`specify` chưa được chạy lại trên máy sạch.

### 0.2 Khởi tạo git và Spec-Kit

```bash
cd MedBook-SpecKit-Course/medbook
git init -b main
git add -A && git commit -m "baseline: MedBook đầu khóa học"

specify check
specify init --here --integration claude --script sh --force
git add -A && git commit -m "chore: specify init"
```

Kết quả: `.specify/` và `.claude/skills/speckit-*` (constitution, specify, clarify, plan, checklist, tasks, analyze, implement, converge, taskstoissues).

### 0.3 Chạy baseline MedBook

```bash
# DB riêng cho bài tập — test sẽ XÓA dữ liệu, đừng trỏ vào DB có dữ liệu thật
docker compose up db -d                      # hoặc dùng PostgreSQL cài sẵn
export DATABASE_URL="postgres://medbook:medbook@localhost:55432/medbook"
npm ci && npm run db:migrate && npm run db:seed
npm test && npm run lint                     # kỳ vọng: 14 test pass, lint sạch
```

Nếu Docker không chạy: dùng PostgreSQL cài sẵn, tạo DB `medbook` và đặt `DATABASE_URL` tương ứng. Cổng 55432 bị chiếm → đổi cổng.

### 0.4 Mở Claude Code

```bash
claude       # gõ "/" → thấy các lệnh speckit-*
```

Không thấy `speckit-*`: thoát, kiểm tra `.claude/skills/` có tồn tại, mở lại `claude` đúng trong `medbook/`.

## 5. Các bước và prompt


### Phần A — Yêu cầu & thiết kế (chưa code)


#### Bước 1. Constitution — nguyên tắc của dự án

- **Lệnh:** `/speckit-constitution`
- **Chức năng:** Ghi các nguyên tắc bất biến của dự án (không phá API cũ, không thêm dependency, kiến trúc phân lớp, SQL tham số hóa, an toàn đồng thời…) vào một file mà MỌI bước sau đều phải tuân theo. Đây là "luật chung" giữ cho AI không đi lệch.
- **Kết quả:** `.specify/memory/constitution.md` (v1.0.0, 8 nguyên tắc)
- **👤 Kiểm tra:** Đủ 8 nguyên tắc I–VIII, không còn `[PLACEHOLDER]`.

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


#### Bước 2. Phân tích yêu cầu thô, tìm chỗ mơ hồ

- **Lệnh:** Prompt thường
- **Chức năng:** Bạn tự để AI soi file yêu cầu thô và liệt kê điều mơ hồ, thiếu, edge case, cùng ≥ 8 câu hỏi cho stakeholder — AI chỉ HỎI, không tự trả lời. Dùng để so với kết quả `/speckit-clarify` ở bước sau.
- **Kết quả:** `bai-nop/day2/01-clarification-questions.md`
- **👤 Kiểm tra:** Có ≥ 8 câu hỏi, mỗi câu có lý do, người trả lời, phương án mặc định.

```text
Đây là yêu cầu thô (từ context/yeu-cau-chuc-nang-moi.md, mục "Yêu cầu mới"
). Ghi bai-nop/day2/01-clarification-questions.md: (1) điều gì còn mơ hồ, (2) điều
gì còn thiếu, (3) edge case chưa được đề cập, (4) ÍT NHẤT 8 câu hỏi clarification cho
stakeholder, mỗi câu có: lý do quan trọng, ai nên trả lời, phương án mặc định đề xuất. Ví dụ
gợi ý chủ đề: X phút là bao nhiêu; "ưu tiên y tế" ai gán, mấy mức; slot mới tạo có kích hoạt
không; bệnh nhân có lịch trùng giờ; hai slot trống cùng lúc; bệnh nhân từ chối có được vào lại
hàng chờ; thông báo kênh nào. KHÔNG tự trả lời — chỉ hỏi.
```


#### Bước 3. Specify — biến yêu cầu thành đặc tả

- **Lệnh:** `/speckit-specify`
- **Chức năng:** Tạo đặc tả tính năng: user story có ưu tiên, yêu cầu chức năng (FR-xxx), tiêu chí chấp nhận Given-When-Then, edge case, tiêu chí thành công. Chỉ mô tả "cái gì/vì sao", chưa nói công nghệ. Lệnh tự tạo thư mục tính năng; không tạo nhánh git.
- **Kết quả:** `specs/001-waiting-list/spec.md`, `checklists/requirements.md`
- **👤 Kiểm tra:** ≥ 3 user story, mỗi story có AC Given-When-Then. Lệnh có thể dừng để hỏi ≤ 3 câu → làm bước kế tiếp.

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


#### Bước 4. Trả lời các câu hỏi then chốt của Specify

- **Lệnh:** Trả lời (một dòng)
- **Chức năng:** `/speckit-specify` chỉ giữ tối đa 3 điểm `[NEEDS CLARIFICATION]` không có mặc định an toàn (ví dụ: ai gán mức ưu tiên, bị từ chối có bị gỡ khỏi hàng chờ không, phạm vi xử lý xung đột). Đây là quyết định của Human. **Đọc câu hỏi thật rồi chọn chữ cái theo ý bạn** — dòng dưới chỉ là ví dụ.
- **Kết quả:** `spec.md` được cập nhật, hết `[NEEDS CLARIFICATION]`
- **👤 Kiểm tra:** Không còn `[NEEDS CLARIFICATION]` trong `spec.md`.

```text
Q1: B, Q2: B, Q3: A
```


#### Bước 5. Clarify — hỏi lại các điểm còn mơ hồ

- **Lệnh:** `/speckit-clarify`
- **Chức năng:** Quét đặc tả theo danh mục (phạm vi, dữ liệu, UX, phi chức năng, edge case…) và hỏi tối đa 5 câu trọng yếu, mỗi câu mã hóa câu trả lời trở lại vào `spec.md`. Prompt dưới kèm sẵn 10 quyết định của Human vì chạy không tương tác; **khi tự làm hãy sửa các quyết định này theo ý bạn** (X phút, mức ưu tiên, kênh thông báo…).
- **Kết quả:** mục `## Clarifications` trong `spec.md`
- **👤 Kiểm tra:** Có đủ câu trả lời trong Clarifications; không còn mơ hồ chặn việc lập kế hoạch.

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


#### Bước 6. Review user story & tiêu chí chấp nhận

- **Lệnh:** Prompt thường
- **Chức năng:** AI đóng vai người review yêu cầu: kiểm tra story/AC, chỉ ra chỗ spec do AI sinh ra SAI hoặc THIẾU so với hệ thống thật (đối chiếu `context/boi-canh-chung`). Ghi lại để biết AI hay sai ở đâu.
- **Kết quả:** `bai-nop/day2/02-ai-errors-log.md`
- **👤 Kiểm tra:** Bạn đọc từng dòng, đánh dấu đồng ý/không đồng ý.

```text
Đọc specs/001-*/spec.md. Review như một Requirement Reviewer: (1) mỗi user story có đủ
role/action/benefit và AC Given-When-Then không; (2) happy path vs edge case có đủ không; (3)
liệt kê những chỗ AI (spec bạn vừa sinh) viết SAI hoặc THIẾU so với context/boi-canh-chung
(đặc biệt luật sẵn có HT-BR và khoảng trống HT-GAP-03: hệ thống chưa chống đặt trùng giờ).
Ghi bai-nop/day2/02-ai-errors-log.md: mỗi dòng = lỗi/thiếu, vì sao, mức độ, đã sửa chưa.
File này sẽ dùng làm bằng chứng ở Day 4. Chưa sửa spec.md.
```


#### Bước 7. Cập nhật spec theo review

- **Lệnh:** `/speckit-specify` (cập nhật)
- **Chức năng:** Áp các mục bạn đã đồng ý ở bước trước vào `spec.md`, không bịa thêm luật nghiệp vụ.
- **Kết quả:** `spec.md` cập nhật
- **👤 Kiểm tra:** Các mục "Accepted" đã vào spec.

```text
/speckit-specify cập nhật spec.md hiện có: áp dụng các mục tôi đã đồng ý trong
bai-nop/day2/02-ai-errors-log.md (đánh dấu mục nào là "Accepted"). Không thêm business rule mới
ngoài những gì đã chốt ở phần Clarifications; nếu thiếu thì để [NEEDS CLARIFICATION].
```


#### Bước 8. Plan — kế hoạch kỹ thuật & chọn kiến trúc

- **Lệnh:** `/speckit-plan`
- **Chức năng:** Dựa trên spec + constitution, sinh kế hoạch kỹ thuật: so sánh ≥ 2–3 phương án kiến trúc (ma trận trọng số), chọn một phương án và giải thích, rồi thiết kế mô hình dữ liệu, hợp đồng API mới, kịch bản kiểm thử nhanh. Có mục "Constitution Check" kiểm tra vi phạm nguyên tắc.
- **Kết quả:** `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` trong `specs/001-waiting-list/`
- **👤 Kiểm tra:** `research.md` có ≥ 2 phương án + lý do chọn; `plan.md` Constitution Check đạt.

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


#### Bước 9. Đóng gói Architecture Blueprint

- **Lệnh:** Prompt thường
- **Chức năng:** Tổng hợp các file của `/speckit-plan` thành một tài liệu kiến trúc duy nhất: sơ đồ, component thêm/sửa, quyết định công nghệ, trade-off, rủi ro.
- **Kết quả:** `bai-nop/day2/04-architecture-blueprint.md`
- **👤 Kiểm tra:** Sơ đồ Mermaid + danh sách file thêm/sửa cụ thể.

```text
Tổng hợp từ plan.md, research.md, data-model.md, contracts/ vào bai-nop/day2/04-architecture-blueprint.md:
sơ đồ kiến trúc high-level (Mermaid), danh sách component THÊM / SỬA trên codebase MedBook hiện
có (đường dẫn file cụ thể), quyết định công nghệ, trade-off analysis, rủi ro chính (đặc biệt
đồng thời: hai người cùng accept, sweeper chạy song song) và cách giảm. Chỉ tổng hợp, không bịa
thêm quyết định mới.
```


#### Bước 10. Checklist — kiểm tra CHẤT LƯỢNG yêu cầu

- **Lệnh:** `/speckit-checklist`
- **Chức năng:** Sinh checklist kiểu "unit test cho tiếng Việt/yêu cầu": yêu cầu có đầy đủ, rõ ràng, nhất quán, đo được không (KHÔNG kiểm tra code chạy đúng). Tập trung vào vùng rủi ro bạn nêu.
- **Kết quả:** `specs/001-waiting-list/checklists/offer-engine.md`
- **👤 Kiểm tra:** Danh sách mục kiểm tra (~20–30 mục). Sẽ cần tick hết ở bước 3.3.

```text
/speckit-checklist Chất lượng yêu cầu của waiting list: đầy đủ, rõ ràng, nhất quán, đo được; tập
trung vào luật chọn ứng viên, vòng đời offer (pending/accepted/declined/expired), đồng thời,
thông báo, quyền sở hữu dữ liệu.
```


#### Bước 11. Rà soát toàn bộ yêu cầu + kiến trúc

- **Lệnh:** Prompt thường
- **Chức năng:** Rà mâu thuẫn, khoảng trống, tính khả thi so với kiến trúc đã chọn, và vi phạm constitution; sửa các phát hiện mức nặng.
- **Kết quả:** `bai-nop/day2/03-requirement-review-notes.md` + `spec.md`/`plan.md` đã sửa
- **👤 Kiểm tra:** Không còn finding Critical/High chưa xử lý.

```text
Review toàn bộ spec.md + plan.md + data-model.md + contracts/ + checklists/. Ghi
bai-nop/day2/03-requirement-review-notes.md: (1) user story/AC nào mâu thuẫn nhau; (2) gap;
(3) đối chiếu kiến trúc đã chọn: yêu cầu nào khó/không khả thi; (4) kiểm tra từng nguyên tắc
I–VIII của constitution có bị vi phạm không. Với mỗi phát hiện: mức độ và đề xuất sửa. Sau đó
sửa spec.md/plan.md cho những phát hiện mức Critical/High, ghi rõ đã sửa gì.
```


#### Bước 12. Tổng kết phần 1

- **Lệnh:** Prompt thường
- **Chức năng:** Lập checklist hoàn thành và liệt kê câu hỏi còn mở (không tự trả lời).
- **Kết quả:** `bai-nop/day2/README.md`
- **👤 Kiểm tra:** Sau bước này: `git add -A && git commit -m "yêu cầu & thiết kế" && git tag day2-frozen`.

```text
Tạo bai-nop/day2/README.md: checklist cuối ngày (7 mục đề Day 2) ✅/❌ + dẫn chứng; liệt kê các
câu hỏi mở còn lại (nếu có) — không tự trả lời.
```


### Phần B — Hiện thực & kiểm thử


#### Bước 13. Tasks — chia việc hiện thực

- **Lệnh:** `/speckit-tasks`
- **Chức năng:** Biến plan + spec thành danh sách task có thứ tự: nhóm theo user story (mỗi story kiểm thử độc lập được), có ID (T001…), đường dẫn file, cờ `[P]` làm song song, task test cho từng tiêu chí.
- **Kết quả:** `specs/001-waiting-list/tasks.md`
- **👤 Kiểm tra:** Mỗi story có nhóm task; mỗi task có đường dẫn file.

```text
/speckit-tasks Chia theo user story để mỗi story có thể kiểm thử độc lập. Thứ tự: migration +
repository → Offer Engine (matching, tạo offer) → hook sau commit ở hủy lịch / mở lại slot →
accept/decline + xử lý conflict → sweeper timeout → API my-offers/waiting-list → giao diện.
Mỗi task có đường dẫn file và trỏ về AC tương ứng. Có task test cho từng AC, gồm test đồng thời.
```


#### Bước 14. Analyze — cổng kiểm tra trước khi code

- **Lệnh:** `/speckit-analyze`
- **Chức năng:** CHỈ ĐỌC. Đối chiếu `spec.md`, `plan.md`, `tasks.md` và constitution để tìm mâu thuẫn, trùng lặp, yêu cầu chưa có task, vi phạm nguyên tắc; xếp theo CRITICAL → LOW.
- **Kết quả:** Báo cáo trong khung chat (không ghi file)
- **👤 Kiểm tra:** Đọc báo cáo; CRITICAL phải sửa trước khi implement.

```text
/speckit-analyze
```


#### Bước 15. Sửa finding của Analyze (chỉ ở tài liệu)

- **Lệnh:** Prompt thường
- **Chức năng:** Sửa CRITICAL/HIGH bằng cách cập nhật spec/plan/tasks — KHÔNG sửa code (tài liệu trước, code sau).
- **Kết quả:** `spec.md`/`plan.md`/`tasks.md` cập nhật
- **👤 Kiểm tra:** Hết CRITICAL; có thể chạy lại `/speckit-analyze` để xác nhận.

```text
Sửa tất cả finding CRITICAL và HIGH trong báo cáo /speckit-analyze vừa rồi, CHỈ bằng cách cập nhật spec.md / plan.md / tasks.md (Nguyên tắc VII: tài liệu trước, code sau; KHÔNG sửa code ở bước này). Với mỗi finding đã sửa, nói rõ file và mục đã đổi; finding MEDIUM/LOW chỉ liệt kê, chưa sửa. Không thêm business rule mới ngoài những gì đã chốt ở Clarifications.
```


#### Bước 16. Duyệt checklist chất lượng yêu cầu

- **Lệnh:** Prompt thường
- **Chức năng:** `/speckit-implement` TỪ CHỐI chạy khi checklist còn mục chưa tick. AI đối chiếu từng mục, tick mục đã đạt (kèm bằng chứng), sửa thiếu sót nhỏ, và liệt kê mục cần Human quyết định.
- **Kết quả:** `checklists/offer-engine.md` (tick) + danh sách câu hỏi
- **👤 Kiểm tra:** Biết chính xác mục nào cần bạn quyết.

```text
Duyệt checklist specs/001-waiting-list/checklists/offer-engine.md: với từng mục, đối chiếu spec.md/plan.md/data-model.md/contracts và đánh dấu [x] nếu yêu cầu ĐÃ đạt (ghi chú ngắn nơi có bằng chứng). Mục nào chưa đạt: giữ [ ] và sửa spec/plan nếu là thiếu sót nhỏ rồi mới tick; nếu cần quyết định của Human thì để [ ] và liệt kê cho tôi. Không tick bừa.
```


#### Bước 17. Quyết định của Human cho mục còn mở

- **Lệnh:** Prompt thường
- **Chức năng:** Bạn trả lời các mục AI không được tự quyết. Prompt viết theo chủ đề (không theo mã CHK vì mã đổi mỗi lần) — **đọc danh sách thật của bạn và sửa quyết định cho khớp**.
- **Kết quả:** `spec.md` (Clarifications/FR) + checklist tick hết
- **👤 Kiểm tra:** Checklist đều `[x]`.

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


#### Bước 18. Implement đợt 1 — nền dữ liệu & luật chọn ứng viên

- **Lệnh:** `/speckit-implement`
- **Chức năng:** Thực thi task theo `tasks.md`, tick `[X]` khi xong. Chia đợt để bạn review từng phần. Đợt 1: migration, repository, Offer Engine (chọn ứng viên, tạo đề xuất). Trước khi chạy test hãy đặt `DATABASE_URL` trỏ DB bài tập.
- **Kết quả:** Code trong `src/`, `tests/`, `tasks.md` được tick
- **👤 Kiểm tra:** Đọc `git diff`; `npm test`, `npm run lint`.

```text
/speckit-implement Chỉ làm các task migration, repository và Offer Engine (matching + tạo offer)
trong tasks.md. Viết code cùng style/convention codebase hiện có (xem doc/backend-flows.md và
src/services/appointmentService.js). Tuân thủ constitution: SQL tham số hóa, transaction + khóa
dòng, không thêm dependency. Đánh dấu [X] task đã xong. Dừng sau đợt này để tôi review.
```


#### Bước 19. Chạy test & tìm nguyên nhân gốc nếu fail

- **Lệnh:** Prompt thường
- **Chức năng:** Lỗi hay gặp: `node --test` chạy các file test SONG SONG trên cùng một DB gây deadlock. Prompt yêu cầu tìm nguyên nhân gốc, không xóa/làm yếu test.
- **Kết quả:** Toàn bộ test xanh, lint sạch
- **👤 Kiểm tra:** Test cũ (14) + test mới đều pass.

```text
Chạy `npm test` (biến môi trường DATABASE_URL đã được đặt sẵn — dùng đúng nó, không đổi cổng, không đụng DB khác). Hiện có test fail (deadlock, transaction aborted…). Tìm nguyên nhân gốc (đọc kỹ lỗi, đừng đoán), sửa tận gốc mà KHÔNG làm yếu hay xóa test nào và không sửa hành vi của 14 test cũ. Chạy lại tới khi toàn bộ xanh và `npm run lint` sạch, rồi giải thích nguyên nhân trong 3 dòng.
```


#### Bước 20. Implement đợt 2 — luồng chấp nhận/từ chối & đồng thời

- **Lệnh:** `/speckit-implement`
- **Chức năng:** Hook sau khi hủy lịch/mở lại slot, chấp nhận/từ chối đề xuất, xử lý xung đột lịch và race condition khi hai người cùng chấp nhận.
- **Kết quả:** Code + test
- **👤 Kiểm tra:** `npm test`, `npm run lint` xanh.

```text
/speckit-implement Làm tiếp các task: hook sau commit ở hủy lịch và mở lại slot; accept/decline;
xử lý conflict lịch và race condition khi hai bệnh nhân cùng accept. Dừng sau đợt này. Chạy test bằng biến DATABASE_URL đã đặt sẵn.
```


#### Bước 21. Implement đợt 3 — hết hạn, API, giao diện

- **Lệnh:** `/speckit-implement`
- **Chức năng:** Sweeper xử lý đề xuất hết hạn, endpoint mới, giao diện tối thiểu (nhân viên thêm vào hàng chờ; bệnh nhân thấy thẻ đề xuất có đếm ngược).
- **Kết quả:** Code + test
- **👤 Kiểm tra:** Toàn bộ task trong `tasks.md` được tick.

```text
/speckit-implement Làm tiếp: sweeper timeout (setInterval, đọc DB mỗi lượt, cấu hình bằng env),
endpoint waiting-list và my-offers, giao diện tối thiểu (staff thêm bệnh nhân vào danh sách chờ;
bệnh nhân thấy thẻ đề xuất có đếm ngược, nút chấp nhận/từ chối). Chạy test bằng biến DATABASE_URL đã đặt sẵn.
```


#### Bước 22. Bổ sung kiểm thử từ tiêu chí chấp nhận

- **Lệnh:** Prompt thường
- **Chức năng:** Viết test trỏ ngược về từng AC (happy path, edge case, hết hạn, đồng thời) và báo cáo AC nào chưa có test + coverage.
- **Kết quả:** `tests/waitlist.test.js`, `bai-nop/day3/03-testing-package.md`
- **👤 Kiểm tra:** Bảng AC → test, không còn AC trống.

```text
Đọc AC trong spec.md và các task test trong tasks.md. Viết test (node --test, cùng style
tests/api.test.js) trong tests/waitlist.test.js: mỗi test có comment "AC-x.y" trỏ về tiêu chí.
Phủ: happy path (có ứng viên và accept), edge case (hàng chờ trống, tất cả từ chối, lịch trùng,
slot < 30 phút, hết hạn), timeout, và test ĐỒNG THỜI (hai bệnh nhân accept cùng lúc → đúng 1
thành công). Chạy `npm test`. Ghi bai-nop/day3/03-testing-package.md: bảng AC → test (còn AC
nào chưa có test?), test pass/fail và nguyên nhân, test nào AI viết đúng, test nào phải sửa và
vì sao, báo cáo coverage (`node --test --experimental-test-coverage tests/*.test.js`).
```


#### Bước 23. Code review & kiểm tra bảo mật

- **Lệnh:** Prompt thường
- **Chức năng:** Review diff: chất lượng, bug, SQL injection, quyền sở hữu offer, race condition, hiệu năng; phân loại FIX NOW / TECH DEBT. Bạn quyết định mục nào sửa ngay.
- **Kết quả:** `bai-nop/day3/04-code-review-report.md`
- **👤 Kiểm tra:** ≥ 3 issue có severity và file:dòng.

```text
Review diff của nhánh feature/waitlist so với main (git diff main). Chỉ ra: (1) code quality
(naming, độ phức tạp, trùng lặp); (2) bug/logic error tiềm ẩn; (3) lỗ hổng bảo mật — đặc biệt
SQL injection, kiểm tra QUYỀN SỞ HỮU offer (bệnh nhân A có accept được offer của B không?),
race condition khi nhiều bệnh nhân nhận cùng slot, sweeper chạy chồng; (4) điểm có thể thành vấn
đề hiệu năng khi lớn. Đối chiếu từng nguyên tắc I–VIII. Ghi bai-nop/day3/04-code-review-report.md:
≥ 3 issue, mỗi issue có severity (Critical/Major/Minor), file:dòng, đề xuất, và phân loại FIX NOW
hay TECH DEBT kèm lý do. Nêu rõ cả những gì bạn KHÔNG chắc hoặc không kiểm tra được.
```


#### Bước 24. Converge — đối chiếu còn thiếu gì

- **Lệnh:** `/speckit-converge`
- **Chức năng:** So code hiện có với spec/plan/tasks và THÊM task cho phần chưa làm vào `tasks.md`. Có task mới → chạy `/speckit-implement` rồi `/speckit-analyze` lại.
- **Kết quả:** `tasks.md` (task bổ sung nếu có)
- **👤 Kiểm tra:** Không còn gap hoặc đã có task cho gap.

```text
/speckit-converge
```


#### Bước 25. Đóng gói tài liệu phát triển

- **Lệnh:** Prompt thường
- **Chức năng:** Tổng hợp API mới, quyết định hiện thực, file đã thêm/sửa, cách chạy; ghi AI sinh đúng gì, phải sửa gì.
- **Kết quả:** `bai-nop/day3/01-development-package.md`, `README.md`
- **👤 Kiểm tra:** Sau đó: `npm test && npm run lint`, thử tay trên giao diện, rồi commit + tag `day3-done`.

```text
Tạo bai-nop/day3/01-development-package.md (API mới, quyết định implementation chính,
file đã thêm/sửa, cách chạy) và bai-nop/day3/README.md (checklist cuối ngày đề Day 3 ✅/❌ +
dẫn chứng; ghi lại: AI sinh đúng gì, cần sửa gì, vì sao).
```


### Phần C — Governance & phân tích lỗi (TÙY CHỌN)


#### Bước 26. (Tùy chọn) Thử tái hiện sự cố

- **Lệnh:** Prompt thường
- **Chức năng:** AI đóng vai kiểm toán, viết test thăm dò xem lỗi "sai chuyên khoa / thông báo lặp" có tái hiện trên code thật không; báo trung thực.
- **Kết quả:** `tests/probe-day4.test.js` (không commit)
- **👤 Kiểm tra:** Kết luận có bằng chứng.

```text
Đóng vai AI-Assisted Auditor, KHÔNG sửa code. Với 2 triệu chứng: (a) bệnh nhân nhận đề xuất từ
bác sĩ/chuyên khoa sai; (b) bệnh nhân nhận đề xuất lặp dù đã có lịch trùng giờ — viết thử các
test thăm dò trong tests/probe-day4.test.js (không commit vào bộ test chính) để xem code hiện
tại có tái hiện không. Báo cáo trung thực: tái hiện được / không tái hiện được / không kết luận
được, kèm file:dòng và đầu ra test. Không kết luận "an toàn" nếu không có bằng chứng.
```


#### Bước 27. (Tùy chọn) Phân tích nguyên nhân gốc

- **Lệnh:** Prompt thường
- **Chức năng:** Truy từ yêu cầu → test → review để tìm nguyên nhân gốc; phân loại và đánh giá rủi ro.
- **Kết quả:** `bai-nop/day4/01-failure-analysis-report.md`
- **👤 Kiểm tra:** Mỗi kết luận có bằng chứng.

```text
Dùng bai-nop/day2/02-ai-errors-log.md, spec.md (AC), tests/waitlist.test.js, bai-nop/day3/04-code-review-report.md
và kết quả 4.1. Ghi bai-nop/day4/01-failure-analysis-report.md: timeline sự cố; truy từng bước
Day 2→3: requirement mơ hồ ở đâu, test thiếu ở đâu, review bước nào đáng lẽ phải bắt được (dẫn
file:dòng/ID AC); phân loại nguyên nhân gốc vào 5 nhãn (Spec sai/thiếu · AI sinh sai · Test thiếu ·
Review bỏ sót · Process không có checkpoint) với tỉ trọng; kết luận đây là lỗi AI hay lỗi
Human-AI process và vì sao; risk assessment nếu không phát hiện sớm; bài học. Mọi kết luận phải
có bằng chứng; chỗ không chắc ghi [CẦN XÁC NHẬN].
```


#### Bước 28. (Tùy chọn) Thiết kế quy trình review Human-AI

- **Lệnh:** Prompt thường
- **Chức năng:** Xác định checkpoint còn thiếu, hard gate, Human-in-the-loop vs on-the-loop, ánh xạ sang lệnh Spec-Kit.
- **Kết quả:** `bai-nop/day4/02-human-ai-review-workflow.md`
- **👤 Kiểm tra:** ≥ 3 checkpoint.

```text
Ghi bai-nop/day4/02-human-ai-review-workflow.md: checkpoint còn thiếu trong các bước trước; workflow
review mới có ≥ 3 validation checkpoint (ai review gì, khi nào, tiêu chí); phân biệt rõ chỗ
nào Human-in-the-loop (review từng output) và chỗ nào Human-on-the-loop (batch/trigger); chỉ rõ
checkpoint nào là HARD GATE chặn merge. Ánh xạ mỗi checkpoint sang lệnh Spec-Kit tương ứng
(/speckit-analyze, /speckit-checklist, /speckit-converge…). Sơ đồ Mermaid.
```


#### Bước 29. (Tùy chọn) Governance checklist

- **Lệnh:** Prompt thường
- **Chức năng:** Checklist ≥ 8 mục theo loại rủi ro trước khi merge code do AI sinh.
- **Kết quả:** `bai-nop/day4/03-ai-governance-checklist.md`
- **👤 Kiểm tra:** Mục kiểm chứng được Yes/No.

```text
Ghi bai-nop/day4/03-ai-governance-checklist.md: ≥ 8 mục, phân theo loại risk (hallucination,
security, incomplete output, over-reliance); mỗi mục có: câu kiểm chứng được (Yes/No), mitigation,
người chịu trách nhiệm, cách kiểm chứng (lệnh/file). Phải đủ cụ thể để dùng trước khi merge
code do AI sinh vào main. Ví dụ mục: "mọi AC đều có test trỏ về", "không có dependency runtime
mới", "không business rule nào thiếu mã nguồn quyết định".
```


#### Bước 30. (Tùy chọn) Đưa governance vào constitution

- **Lệnh:** `/speckit-constitution`
- **Chức năng:** Nâng constitution lên v1.1.0 với các hard gate và mục governance quan trọng nhất.
- **Kết quả:** `.specify/memory/constitution.md` v1.1.0
- **👤 Kiểm tra:** Có Sync Impact Report.

```text
/speckit-constitution Nâng lên v1.1.0: thêm mục "Governance cho mã do AI sinh" từ 3 hard-gate
trong bai-nop/day4/02-human-ai-review-workflow.md và 5 mục quan trọng nhất của
bai-nop/day4/03-ai-governance-checklist.md. Ghi Sync Impact Report. Không đổi nguyên tắc I–VIII.
```


#### Bước 31. (Tùy chọn) Checklist trước khi merge

- **Lệnh:** `/speckit-checklist`
- **Chức năng:** Checklist pre-merge cho code do AI sinh, dựa trên constitution mới.
- **Kết quả:** `checklists/…`
- **👤 Kiểm tra:** Dùng được trước mỗi lần merge.

```text
/speckit-checklist Pre-merge cho mã do AI sinh của waiting list, dựa trên constitution v1.1.0.
```


#### Bước 32. (Tùy chọn) Audit lệch spec ↔ code

- **Lệnh:** `/speckit-analyze`
- **Chức năng:** Kiểm tra AC nào chưa có test, code nào làm khác spec, luật nào thiếu nguồn quyết định.
- **Kết quả:** Báo cáo trong chat
- **👤 Kiểm tra:** Xử lý các finding.

```text
/speckit-analyze Tập trung: AC nào chưa có test; chỗ nào code làm khác spec; business rule nào
thiếu nguồn quyết định.
```


#### Bước 33. (Tùy chọn) Tổng kết governance

- **Lệnh:** Prompt thường
- **Chức năng:** Checklist hoàn thành và kết luận lỗi AI hay lỗi quy trình.
- **Kết quả:** `bai-nop/day4/README.md`
- **👤 Kiểm tra:** —

```text
Tạo bai-nop/day4/README.md: checklist cuối ngày (7 mục đề Day 4) ✅/❌ + dẫn chứng; trả lời
trong 5 dòng: lỗi này là lỗi AI hay lỗi process, vì sao.
```


## 6. Nghiệm thu cuối

```bash
npm test && npm run lint     # 14 test cũ + test mới đều xanh
npm start                    # http://localhost:4300 (mật khẩu mọi tài khoản demo: demo123)
```

Thử tay: nhân viên `mai.staff@medbook.local` thêm `an@medbook.local` vào danh sách chờ (mức `urgent`) → bệnh nhân khác (ví dụ `linh@medbook.local`) đặt rồi hủy một slot → `an@` thấy thẻ đề xuất → Chấp nhận → thấy lịch của mình.

## 7. Lưu ý đã gặp khi chạy thử

- `/speckit-specify` tự hỏi ≤ 3 câu then chốt — phải trả lời (bước 4).
- `/speckit-implement` từ chối chạy nếu `checklists/*.md` còn mục chưa tick — cần bước duyệt checklist và quyết định (bước 16–17).
- `node --test` chạy file test song song trên cùng DB → có thể deadlock; bước 19 yêu cầu tìm nguyên nhân gốc.
- `/speckit-analyze` thực tế bắt được lỗi nghiêm trọng thật (từng gặp 1 CRITICAL, 2 HIGH) — đừng bỏ qua.
- Spec-Kit không tự tạo nhánh git; bạn tự `git checkout -b feature/waitlist` nếu muốn.
- Đã chạy thật: bước 0 → 19 (đợt implement 1). Chưa chạy thử: implement đợt 2–3 và `/speckit-converge`, các prompt thường ở phần C và một số prompt phụ. Gặp lỗi thì sửa prompt tương ứng.
