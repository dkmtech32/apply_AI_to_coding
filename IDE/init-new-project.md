# Khởi tạo dự án mới: ChatGPT chốt plan & thiết kế → Cursor implement

Luồng **greenfield** gợi ý: làm **rõ và chốt** trên **ChatGPT** (hoặc công cụ tương đương **không** có quyền xem repo) — **tính năng, cấu trúc project, UI/UX, database** — rồi đưa **gói artifact** vào **Cursor** để code bám repo thật, rules và lệnh verify.

ChatGPT **không** thấy cây thư mục của bạn; mọi đường dẫn file và convention cuối cùng do **Cursor** đối chiếu khi implement. Tránh để một công cụ vừa “bịa cấu trúc” vừa sửa code — tách giai đoạn cho rõ.

Liên quan: [index.md](index.md), [implement-new-feature.md](implement-new-feature.md) (brief ngoài repo → kế hoạch gắn file), [debug.md](debug.md), [write-test-case.md](write-test-case.md).

---

## Khi nào nên dùng luồng này

- Bạn muốn **scope và thiết kế** (feature, folder mental model, màn hình, schema) **ổn định trước** khi Agent sửa hàng loạt file.
- Team cần **một nguồn brief** (từ ChatGPT) để Cursor chỉ việc **map vào stack** (Laravel, Next, v.v.) và `.cursor/rules` đã ghi.

---

## Giai đoạn 1 — ChatGPT: plan, cấu trúc, UI/UX, DB (chưa mở Cursor để code)

Làm **lần lượt hoặc lặp** cho đến khi bạn chấp nhận bản chốt. Mỗi lần lưu output vào một chỗ cố định (file `docs/brief-*.md`, Notion, v.v.) để sau này **`@`** trong Cursor.

### 1.1 Chốt tính năng và phạm vi

Yêu cầu ChatGPT **không** đoán đường dẫn repo. Gợi ý prompt (rút gọn từ [implement-new-feature.md](implement-new-feature.md)):

```text
Đóng vai senior PM + engineer. Tôi mô tả sản phẩm / ý tưởng dưới đây.

Trả lại:
1. Mục tiêu và phạm vi KHÔNG làm (non-goals)
2. User stories hoặc acceptance criteria theo từng mốc (MVP trước)
3. Câu hỏi mở tối đa 5 ý — những gì bạn cần tôi quyết định
4. Rủi ro (bảo mật, dữ liệu, pháp lý, hiệu năng) và cách giảm thiểu

Giả định bạn KHÔNG có quyền truy cập repo. Không ghi đường dẫn file hay tên framework cụ thể trừ khi tôi nêu stack.
```

**Bản tiếng Anh:**

```text
Act as a senior PM + engineer. I describe the product / idea below.

Return:
1. Goals and explicit non-goals
2. User stories or acceptance criteria by milestone (MVP first)
3. At most 5 open questions — what you need me to decide
4. Risks (security, data, compliance, performance) and mitigations

Assume you do NOT have access to my repository. Do not invent file paths or specific framework names unless I state the stack.
```

Sau khi trả lời câu hỏi mở, chốt **MVP** vs **sau MVP**.

### 1.2 Chốt cấu trúc project và kiến trúc (mức khái niệm)

Khi đã biết **stack đích** (ví dụ monolith Laravel + SPA, hay API + web riêng), nhắc stack trong prompt:

```text
Với stack: [MÔ TẢ NGẮN — ví dụ Laravel API + Inertia + MySQL], hãy đề xuất:

1. Cấu trúc thư mục / bounded context (tên module logic, không cần khớp 100% repo thật)
2. Ranh giới tầng: HTTP, domain/service, persistence, jobs, notification
3. Cách tổ chức config, env, feature flag (khái niệm)
4. Chiến lược test (unit vs feature) ở mức nguyên tắc

Không bịa đường dẫn cụ thể. Ghi rõ giả định nếu thiếu thông tin.
```

**Bản tiếng Anh:**

```text
Given target stack: [SHORT DESCRIPTION — e.g. Laravel API + Inertia + MySQL], propose:

1. Folder layout / bounded contexts (logical module names — need not match a real repo exactly)
2. Layer boundaries: HTTP, domain/service, persistence, jobs, notifications
3. How to organize config, env, and feature flags (conceptual)
4. Testing strategy (unit vs integration/feature) at a principles level

Do not invent concrete paths. State assumptions clearly if information is missing.
```

Output dùng làm **kim chỉ nam**; khi vào Cursor, yêu cầu Agent **đối chiếu với convention repo** và sửa cho khớp.

### 1.3 UI/UX trước khi code

Yêu cầu ChatGPT:

```text
Dựa trên MVP đã chốt, hãy:

1. Danh sách màn hình / route người dùng (tên + mục đích)
2. Luồng chính (happy path) dạng bước 1→2→3
3. Trạng thái cần thiết: loading, empty, error, permission denied
4. Component / pattern lặp lại (shell, list+filter, form, modal, toast)
5. Gợi ý a11y: focus, nhãn form, contrast — mức checklist
```

**Bản tiếng Anh:**

```text
Based on the agreed MVP:

1. List of screens / user-facing routes (name + purpose)
2. Primary happy-path flow as numbered steps 1→2→3
3. Required UI states: loading, empty, error, permission denied
4. Recurring components / patterns (shell, list+filter, form, modal, toast)
5. Accessibility checklist: focus, form labels, contrast — keep it practical
```

**Tuỳ chọn trực quan:** sau khi có bản trên, dùng **Stitch**, **Lovable**, hoặc Figma để có pixel / prototype — xem [lovable.md](../lovable/lovable.md) và phần “Công cụ UI tùy chọn” bên dưới. Cursor vẫn implement theo **repo + rules**, coi export prototype là **tham chiếu**, không merge mù generated code.

### 1.4 Thiết kế database

```text
Từ MVP và các màn hình đã liệt kê, thiết kế mức ERD / logical schema:

1. Bảng (hoặc aggregate), khóa chính, quan hệ 1-n / n-n
2. Cột quan trọng và kiểu dữ liệu (không cần SQL chi tiết nếu chưa chốt engine)
3. Index / ràng buộc unique có ý nghĩa nghiệp vụ
4. Soft delete, audit trail, multi-tenant (nếu có) — ghi rõ có/không
5. Migration / seed ở mức ý tưởng (thứ tự phụ thuộc FK)

Không đặt tên cột bịa nếu có chuẩn domain — dùng thuật ngữ tôi đã dùng trong mô tả.
```

**Bản tiếng Anh:**

```text
From the MVP and listed screens, design a logical schema / ERD-level model:

1. Tables (or aggregates), primary keys, 1-n / n-n relationships
2. Important columns and data types (detailed SQL optional if the engine is not fixed)
3. Indexes / unique constraints that matter for the business rules
4. Soft deletes, audit trail, multi-tenant (if any) — state yes/no
5. Migration / seed intent only (dependency order for FKs)

Use domain terms from my description; do not invent column names that contradict the domain language I used.
```

### Checklist xong giai đoạn ChatGPT

- [ ] MVP và non-goals đã **chốt bằng văn bản**.
- [ ] Có **danh mục màn hình + luồng** (và prototype tuỳ chọn).
- [ ] Có **mô tả schema / ERD** đủ để viết migration.
- [ ] Có **một file hoặc một chỗ** tổng hợp để dán vào Cursor (hoặc commit vào `docs/`).

---

## Giai đoạn 2 — Repo, Cursor rules, artifact

1. Tạo repo (hoặc `laravel new` / starter đã chọn), mở **workspace root** trong Cursor.
2. Thêm **`.cursor/rules`** (và tuỳ chọn **Agent Skill**) khóa stack thật: framework, DB, formatter, test runner, ranh giới module.
3. **`.cursorignore`**: secret, `vendor`, `node_modules`, export nặng, dump DB.
4. Đặt output ChatGPT vào repo (ví dụ `docs/00-brief.md`) hoặc dán trực tiếp vào chat kèm **`@docs/...`**.

Mẫu rule tối thiểu (chỉnh thành stack bạn):

```markdown
---
description: Stack repo này (bắt buộc khi implement)
globs: "**/*"
alwaysApply: true
---

- Runtime / framework: …
- Database + migration tool: …
- API / frontend (Inertia, SPA, Blade…): …
- Auth: …
- Test: … (lệnh chạy)
- Không commit secret; dùng .env.example
```

**Rule template (English):**

```markdown
---
description: Mandatory stack for this repository
globs: "**/*"
alwaysApply: true
---

- Runtime / framework: …
- Database + migration tool: …
- API / frontend (Inertia, SPA, Blade…): …
- Auth: …
- Tests: … (how to run)
- No secrets in git; use .env.example
```

Tài liệu: [Cursor Docs](https://cursor.com/docs) · [Rules](https://cursor.com/docs/rules).

---

## Giai đoạn 3 — Trong Cursor: kế hoạch gắn file rồi implement

**Không** nhảy vào code hàng loạt trước khi Agent đã **đối chiếu brief với repo**.

### Bước A — Handoff prompt (Plan mode khuyến nghị)

```text
Dưới đây là brief & thiết kế từ ChatGPT (không có context repo lúc soạn).

Nhiệm vụ:
1. Đối chiếu với repository hiện tại — chỉnh lại đường dẫn / module cho khớp convention.
2. Đề xuất thứ tự slice nhỏ (deploy/khả dụng được trước): migration → model → API → UI.
3. Mỗi slice: file dự kiến, test cần có, lệnh verify local.
4. Nêu phần "ngoài phạm vi" PR đầu.

Context:
@docs/00-brief.md (hoặc file bạn đặt)
@.cursor/rules/…

Chưa sửa code cho đến khi tôi duyệt kế hoạch.
```

**Bản tiếng Anh:**

```text
Below is the product brief and design from ChatGPT (no repo access when it was written).

Your tasks:
1. Reconcile with this repository — adjust paths and modules to match real conventions.
2. Propose an order of small vertical slices (shippable early): migrations → models → API → UI.
3. For each slice: likely files, tests to add, and local verify commands.
4. Call out what is "out of scope" for the first PR.

Context:
@docs/00-brief.md (or your brief file)
@.cursor/rules/…

Do not edit application code until I approve the plan.
```

### Bước B — Thực thi

Chuyển **Agent**, làm **từng slice**: migration + test + API/UI tương ứng. Giữ **hợp đồng API** (OpenAPI hoặc type dùng chung) khớp brief nếu có tách frontend/backend.

---

## Công cụ UI tùy chọn (sau bản ChatGPT)

Khi cần **hình ảnh / click được** trước khi code chi tiết:

### Stitch (MCP trong Cursor khi implement)

- Ngoài Cursor: tạo screen trong [Stitch](https://stitch.google), lưu token & luồng.
- Trong Cursor: bật MCP Stitch (`user-stitch`), **`@`** export hoặc lấy screen qua MCP khi implement.
- Mẹo: tắt MCP thì **PNG/SVG + spec** vẫn **`@`** được.

### Lovable

- Chi tiết prompt & handoff: [lovable.md](../lovable/lovable.md).
- Coi code export là **reference**; viết lại trong repo theo stack và lint.

---

## Nối frontend và backend

1. **Contract:** một nguồn sự thật (OpenAPI, DTO PHP/TS dùng chung, v.v.).
2. **Môi trường:** `README` có cách chạy web, API, test; biến env quan trọng trong `.env.example`.
3. **E2E:** khi luồng chính ổn định local, cân nhắc smoke E2E (Dusk/Pest/Playwright tuỳ stack).

---

## Checklist một trang

**Trước khi code nhiều trong Cursor**

- [ ] Brief ChatGPT (feature + UI + DB) đã **chốt** và nằm trong repo hoặc sẵn sàng dán.
- [ ] `.cursor/rules` đã phản **stack thật**.
- [ ] Kế hoạch **slice** trong Cursor đã duyệt.

**Mốc merge đầu tiên**

- [ ] DB: migration cốt lõi + seed tối thiểu (nếu cần).
- [ ] Backend/API hoặc domain + test happy path.
- [ ] Frontend: shell + **một** luồng MVP với API mock hoặc thật.
- [ ] `README` cách chạy và verify.

---

_Stitch MCP: `list_projects`, `list_screens`, `get_screen` — [MCP docs](https://cursor.com/docs/mcp)._
