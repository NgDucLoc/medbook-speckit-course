# KỊCH BẢN — Thêm chức năng mới cho MedBook bằng Spec-Kit

Làm theo thứ tự. Mỗi bước = **dán nguyên một file** trong `prompts/dayN/` vào Claude Code (đã mở ở thư mục `medbook/`).
Prompt bắt đầu bằng `/speckit-…` là lệnh Spec-Kit. Prompt còn lại là prompt thường, ghi kết quả vào `bai-nop/dayN/`.

**Trạng thái:** ✅ đã chạy thật và hoạt động (bước cài đặt 0.1: chưa chạy lại trên máy sạch) · ⏳ chưa chạy thử (cùng kiểu với bước ✅ gần nhất).

## Bước 0 — Cài đặt và chuẩn bị (từ đầu)

Thư mục `medbook/` chứa **code MedBook đầu khóa** (chưa có waiting list) và `context/` (hệ thống hiện có + `yeu-cau-chuc-nang-moi.md` là đầu vào). **Chưa** cài Spec-Kit, **chưa** có git. Toàn bộ prompt ở [PROMPTS.md](PROMPTS.md) (một file) hoặc `prompts/dayN/`.

### 0.1 Cài công cụ

| Công cụ | Cài / kiểm tra |
| --- | --- |
| `uv` (trình cài Python tool) | macOS: `brew install uv` — hoặc `curl -LsSf https://astral.sh/uv/install.sh \| sh`; kiểm tra `uv --version` |
| **Spec-Kit (`specify`)** | `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v0.16.2` ; kiểm tra `specify version` |
| Claude Code (`claude`) | `npm install -g @anthropic-ai/claude-code` ; kiểm tra `claude --version`, đăng nhập một lần bằng `claude` |
| Node 20+ , git | `node -v`, `git --version` |
| PostgreSQL | Docker Desktop **hoặc** PostgreSQL cài sẵn |

> Đã kiểm chứng với `specify` v0.16.2 (lệnh dạng `/speckit-*`). Bản khác có thể đặt tên lệnh khác.

### 0.2 Khởi tạo git và Spec-Kit

```bash
cd MedBook-SpecKit-Course/medbook
git init -b main
git add -A && git commit -m "baseline: MedBook đầu khóa học"

specify check                                   # xem công cụ nào còn thiếu
specify init --here --integration claude --script sh --force
git add -A && git commit -m "chore: specify init"
```

Kết quả: `.specify/` (template, script, constitution rỗng) và `.claude/skills/speckit-*` (constitution, specify, clarify, plan, checklist, tasks, analyze, implement, converge, taskstoissues).

### 0.3 Chạy baseline MedBook

```bash
# DB riêng cho bài tập (test sẽ xóa dữ liệu — đừng trỏ vào DB có dữ liệu thật)
docker compose up db -d                         # hoặc dùng PostgreSQL cài sẵn
export DATABASE_URL="postgres://medbook:medbook@localhost:55432/medbook"
npm ci && npm run db:migrate && npm run db:seed
npm test && npm run lint                        # kỳ vọng: 14 test pass, lint sạch
```

### 0.4 Mở Claude Code

```bash
claude        # gõ "/" → thấy các lệnh speckit-*
```

Nếu không thấy `speckit-*`: thoát `claude`, kiểm tra `.claude/skills/` có tồn tại, mở lại `claude` trong đúng thư mục `medbook/`.

---

## Day 2 — Requirement & System Design (không code)

| Prompt | Lệnh | Kết quả |  |
| --- | --- | --- | :-: |
| `day2/2.1-constitution` | `/speckit-constitution` | `.specify/memory/constitution.md` (8 nguyên tắc) | ✅ |
| `day2/2.2-cau-hoi-lam-ro` | — | `bai-nop/day2/01-clarification-questions.md` | ⏳ |
| `day2/2.3a-specify` | `/speckit-specify` | `specs/001-waiting-list/spec.md` | ✅ |
| `day2/2.3b-tra-loi-specify` | trả lời | Spec-Kit tự hỏi ≤ 3 câu `[NEEDS CLARIFICATION]` → trả lời `Q1: B, Q2: B, Q3: A` (đọc câu hỏi thật rồi chọn theo ý bạn) | ✅ |
| `day2/2.4-clarify` | `/speckit-clarify` | mục `## Clarifications` trong `spec.md` (prompt kèm sẵn 10 quyết định của Human — sửa theo ý bạn) | ✅ |
| `day2/2.5a-review-user-story` → `2.5b-cap-nhat-spec` | — | `bai-nop/day2/02-ai-errors-log.md`, cập nhật `spec.md` | ⏳ |
| `day2/2.6a-plan` | `/speckit-plan` | `plan.md`, `research.md` (3 phương án, chọn B), `data-model.md`, `contracts/`, `quickstart.md` | ✅ |
| `day2/2.6b-architecture-blueprint` | — | `bai-nop/day2/04-architecture-blueprint.md` | ⏳ |
| `day2/2.7a-checklist` | `/speckit-checklist` | `checklists/offer-engine.md` | ✅ |
| `day2/2.7b-requirement-review` | — | `bai-nop/day2/03-requirement-review-notes.md` + sửa spec/plan | ⏳ |
| `day2/2.8-tong-ket` | — | `bai-nop/day2/README.md` | ⏳ |

```bash
git add -A && git commit -m "day2" && git tag day2-frozen
```

## Day 3 — Development & Quality (ngày duy nhất chạm code)

| Prompt | Lệnh | Kết quả |  |
| --- | --- | --- | :-: |
| `day3/3.1-tasks` | `/speckit-tasks` | `tasks.md` (task theo user story, có ID, đường dẫn file) | ✅ |
| `day3/3.2a-analyze` | `/speckit-analyze` | báo cáo bất nhất (lần chạy thử: 1 CRITICAL, 2 HIGH) | ✅ |
| `day3/3.2b-sua-finding` | — | sửa CRITICAL/HIGH **ở tài liệu**, chưa sửa code | ✅ |
| `day3/3.3a-duyet-checklist` | — | tick mục checklist đã đạt, liệt kê mục cần Human quyết | ✅ |
| `day3/3.3b-quyet-dinh-human` | — | ghi quyết định vào Clarifications, tick hết checklist | ✅ |
| `day3/3.4a-implement-dot1` | `/speckit-implement` | migration + repository + Offer Engine | ✅ |
| `day3/3.4b-sua-test` | — | chạy `npm test`, tìm nguyên nhân gốc nếu fail | ✅ |
| `day3/3.4c-implement-dot2` | `/speckit-implement` | hook hủy lịch/mở slot, accept/decline, race | ⏳ |
| `day3/3.4d-implement-dot3` | `/speckit-implement` | sweeper, API, giao diện | ⏳ |
| `day3/3.5-unit-test` | — | `tests/waitlist.test.js`, `bai-nop/day3/03-testing-package.md` | ⏳ |
| `day3/3.6-code-review` | — | `bai-nop/day3/04-code-review-report.md` | ⏳ |
| `day3/3.7-converge` | `/speckit-converge` | thêm task cho phần còn thiếu | ⏳ |
| `day3/3.8-development-package` | — | `bai-nop/day3/01-development-package.md`, `README.md` | ⏳ |

Nghiệm thu: `npm test && npm run lint`, rồi `npm start` → http://localhost:4300 (mật khẩu `demo123`): staff `mai.staff@medbook.local` thêm `an@medbook.local` vào danh sách chờ → bệnh nhân khác đặt rồi hủy slot → `an@` thấy thẻ đề xuất → Chấp nhận.

**Lưu ý khi chạy:**
- `/speckit-implement` **từ chối chạy** nếu `checklists/*.md` còn mục chưa tick → cần 3.3a/3.3b trước.
- `node --test` chạy các file test song song trên cùng một DB nên có thể **deadlock**; 3.4b yêu cầu tìm nguyên nhân gốc, không xóa/làm yếu test.
- Mã mục checklist (CHK…) và số câu hỏi mỗi lần AI sinh có thể khác; 3.3b viết theo chủ đề, nhưng hãy đọc danh sách thật của bạn.

```bash
git add -A && git commit -m "day3" && git tag day3-done
```

## Day 4 — Governance & Failure Analysis (không code)

| Prompt | Lệnh | Kết quả |  |
| --- | --- | --- | :-: |
| `day4/4.1-tai-hien-su-co` | — | thử tái hiện sự cố sai chuyên khoa / thông báo lặp (báo cáo trung thực) | ⏳ |
| `day4/4.2-failure-analysis` | — | `bai-nop/day4/01-failure-analysis-report.md` | ⏳ |
| `day4/4.3-review-workflow` | — | `02-human-ai-review-workflow.md` | ⏳ |
| `day4/4.4-governance-checklist` | — | `03-ai-governance-checklist.md` (≥ 8 mục) | ⏳ |
| `day4/4.5a-constitution-v1.1` | `/speckit-constitution` | nâng constitution lên v1.1.0 | ⏳ |
| `day4/4.5b-checklist-pre-merge` | `/speckit-checklist` | checklist trước khi merge | ⏳ |
| `day4/4.5c-analyze-audit` | `/speckit-analyze` | audit lệch spec ↔ code | ⏳ |
| `day4/4.6-tong-ket` | — | `bai-nop/day4/README.md` | ⏳ |

---

## Ghi chú

- Thứ tự Spec-Kit: constitution → specify → clarify → plan → checklist → tasks → analyze → implement → converge.
- Thời gian thật khi chạy thử mỗi lệnh Spec-Kit: 3–15 phút (implement đợt 1 ~20 phút). Khi quay video nên tua các đoạn AI sinh nội dung.
- Kết quả AI mỗi lần một khác; khi gặp câu hỏi hoặc finding thật, đọc rồi quyết định, đừng dán máy móc.
- Bản draft cũ (có phụ lục quay video): `KICHBAN-DRAFT.md`.
