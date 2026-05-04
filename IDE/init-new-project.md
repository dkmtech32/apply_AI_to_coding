# Khởi tạo dự án mới: frontend thiết kế trước + backend bám chặt tech stack

Luồng **greenfield** đầy đủ: **dựng UI/UX trước** ngoài Cursor, rồi đưa artifact vào Cursor để code. Với **backend**, **khóa tech stack** bằng **Agent Skills** và/hoặc **project rules** của Cursor, sau đó bám **kế hoạch triển khai** trước khi viết code ồ ạt.

Liên quan: [index.md](index.md) (Cursor cơ bản), [implement-new-feature.md](implement-new-feature.md) (brief ChatGPT → kế hoạch Cursor).

---

## Khi nào nên dùng luồng này

- Bạn muốn **hướng UI nhìn thấy được** trước khi viết component (ít phải sửa layout lặp lại).
- Backend cần **stack có chủ đích** (ngôn ngữ, framework, DB, auth, môi trường triển khai)—không phải “model đoán bừa”.

---

## Phase 0 — Repo và rào chắn

1. Tạo thư mục trống và mở làm **workspace root** trong Cursor.
2. Sớm thêm **`.cursor/rules`** hoặc **Agent Skill** cho những thứ ít đổi:
   - Package manager, formatter, test runner
   - **Stack backend** (xem dưới) sau khi đã chốt
3. Dùng **`.cursorignore`** cho secret và artifact cồng kềnh (export, `node_modules`, dump).

Tài liệu chính thức: [Cursor Docs](https://cursor.com/docs) · [Rules](https://cursor.com/docs/rules).

---

## Frontend — Stitch hoặc Lovable trước, rồi Cursor

### Phương án A: Google **Stitch**

**Mục tiêu:** Màn hình, luồng, và tư duy design system trước khi code.

**Ngoài Cursor**

1. Trong Stitch, tạo **project** và các screen cho luồng chính (happy path + empty/error nếu được).
2. Ghi lại **tên screen**, thứ tự luồng, và **design token** (màu, type scale, khoảng cách) mà Stitch cung cấp.

**Trong Cursor**

1. Bật MCP **Stitch** trong Cursor nếu bạn dùng (`stitch` / `user-stitch` trong cài đặt MCP). Xác thực nếu được hỏi.
2. Mở **Plan mode** (hoặc thread **Chat** mới). Nhờ model **lấy ngữ cảnh màn hình qua MCP** khi implement (ví dụ: list project/screen, rồi `get_screen` cho đúng màn).
3. **`@`** thư mục app đích (ví dụ `apps/web/` hoặc `frontend/`) và bản nháp **hợp đồng API** để layout khớp dữ liệu thật.
4. Làm **một lát cắt dọc (vertical slice)**: khung routing → một luồng hoàn chỉnh → component nối stub hoặc API thật.

**Mẹo:** Nếu tắt MCP, export **PNG/SVG** hoặc **spec** từ Stitch rồi **`@`** hoặc dán vào chat; chất lượng giảm nhẹ nhưng vẫn làm được.

### Phương án B: **Lovable**

Hướng dẫn chi tiết + bộ **prompt tiếng Anh** (bootstrap, từng màn, polish, mock API, handoff Cursor): [lovable.md](lovable.md).

**Mục tiêu:** Prototype tương tác nhanh và thường có cấu trúc UI **xuất ra** được.

**Ngoài Cursor**

1. Dựng trang cốt lõi và navigation trong Lovable cho đến khi các bên thống nhất **IA** (information architecture) và **component chính**.
2. Giữ **bản export** (zip code, đồng bộ repo, hoặc copy từng phần—tuỳ Lovable cho project bạn) và **URL** bản build tham chiếu. Chụp màn các trạng thái quan trọng để so regression.

**Trong Cursor**

1. Ưu tiên **target sạch** trong repo (ví dụ app Vite/Next) thay vì merge mù generated code: coi output Lovable là **bản tham chiếu**.
2. Dán hoặc import export vào thư mục **`reference/`** hoặc **`_lovable_source/`** (có thể gitignore nếu license lạ), rồi nhờ Cursor **viết lại** theo stack và lint của bạn.
3. **`@`** thư mục đó + **`@`** ghi chú thiết kế. Yêu cầu: bám layout và UX; refactor theo **thư viện component** và quy ước **routing** của team.

**Mẹo:** Lovable và repo cuối có thể khác về state và API—yêu cầu rõ Cursor **tách “presentation” và “data fetching”** để logic backend không bị nhét trùng trong component.

### Checklist frontend trước khi backend sâu

- [ ] **Bản đồ route** đã thống nhất (tên đồng bộ ngôn ngữ sản phẩm).
- [ ] **Danh mục component**: shell, list, form, modal, toast.
- [ ] **Responsive** và kỳ vọng **a11y** (phím, focus, độ tương phản).
- [ ] **Placeholder dữ liệu**: type giả hoặc nháp OpenAPI để màn không hard-code field bịa.

---

## Backend — áp dụng **skill tech stack**, rồi **plan → implement**

Ở đây “skill” = **Agent Skill** (`SKILL.md`) và/hoặc **project rules** **mã hóa stack** để mọi kế hoạch và PR đồng nhất.

### Bước 1 — Làm stack thành văn bản (skill + rules)

**Nếu team đã có skill** (ví dụ “Backend: FastAPI + Postgres + Alembic + Docker”):

- Đảm bảo file nằm đúng chỗ Cursor load skill (theo setup team) và **gọi / tuân thủ** khi mở đầu repo.

**Nếu chưa có**, tạo **project rule** tối thiểu trong `.cursor/rules/` trước (nhanh nhất), ví dụ:

```markdown
---
description: Stack backend repo này (mặc định bắt buộc)
globs: "**/*"
alwaysApply: true
---

- Ngôn ngữ/runtime: …
- Web framework: …
- Database: … (+ công cụ migration)
- Auth/session: …
- Kiểu API: REST / GraphQL / RPC
- Cấu hình: biến môi trường kiểu 12-factor; không commit secret
- Test: …
```

Sau có thể nâng thành **Agent Skill** tái dùng (`SKILL.md` + frontmatter `name` / `description`); xem mục Agent Skills trong [Cursor Docs](https://cursor.com/docs).

### Bước 2 — Prompt Cursor lấy **kế hoạch triển khai** (chưa code)

Dùng **Plan mode** nếu có. **`@`** thư mục `backend/` (hoặc `server/`) trống và mọi **OpenAPI / ERD / brief sản phẩm**. Dán prompt:

```text
Chúng ta đang khởi tạo backend từ đầu. Tuân thủ đúng rule/skill stack backend của project.

Hãy trả lại:
1. Cấu trúc repo (thư mục, module) và lý do
2. Danh sách dependency (thư viện) — ghi rõ version cố định hoặc chính sách version
3. Xử lý config & secret (.env.example), logging, dạng lỗi trả về
4. Tầng dữ liệu: cách đặt schema, migration, chiến lược seed
5. Ranh giới auth nếu cần (route public vs protected)
6. Chiến lược test (unit/integration) và lệnh chạy thân thiện CI
7. Danh sách task triển khai theo thứ tự (slice nhỏ deploy được trước)
8. Phần "ngoài phạm vi v1"

Chưa viết code ứng dụng cho đến khi tôi duyệt kế hoạch.
```

**Bản tiếng Anh:**

```text
We are initializing the backend from scratch. Follow the project's backend stack rule / skill exactly.

Deliver:
1. Repository layout (folders, modules) and why
2. Dependency list (libraries) with versions pinned or version policy
3. Config & secrets handling (.env.example), logging, error shape
4. Data layer: schema approach, migrations, seed strategy
5. Auth boundary if required (public vs protected routes)
6. Testing strategy (unit/integration) and CI-friendly commands
7. Ordered implementation tasks (smallest deployable slice first)
8. "Out of scope" for v1

Do not write application code until I approve the plan.
```

Điều chỉnh các mục nếu bạn dùng **serverless**, **BaaS**, hay **monolith**—khung vẫn giữ nguyên.

### Bước 3 — Thực thi sau khi duyệt

Chuyển sang **Agent**, làm **từng task**, chạy **migration và test** sau mỗi slice. Giữ type API đồng bộ với **hợp đồng phía frontend** (package type dùng chung hoặc OpenAPI).

---

## Nối frontend và backend

1. **Contract trước:** **OpenAPI** tối thiểu, **tRPC router**, hoặc **typed client**—một nguồn sự thật để hai phía import hoặc generate.
2. **Môi trường:** Ghi `API_BASE_URL`, cookie auth/CORS, port dev local trong `README`.
3. **E2E sau:** Khi một luồng chạy local ổn, thêm smoke E2E (Playwright/Cypress) nếu team cần.

---

## Checklist một trang

**Trước khi code “thật sự”**

- [ ] Đã có artifact từ Stitch **hoặc** Lovable (MCP, export, hoặc screenshot + ghi chú).
- [ ] **Stack backend** đã ghi trong `.cursor/rules` hoặc **Skill**; team đồng ý.
- [ ] **Plan** backend trong Cursor đã duyệt; frontend đã có phạm vi route/component.

**Mốc đủ merge đầu tiên**

- [ ] Frontend: shell + **một** luồng user đầy đủ với API **mock hoặc thật**.
- [ ] Backend: **health** + **một** resource + **migration** + **test** happy path.
- [ ] `README` mô tả cách chạy **web**, **API**, và **test**.

---

*Stitch MCP có các tool như `list_projects`, `list_screens`, `get_screen`—xem [tài liệu MCP](https://cursor.com/docs/mcp) khi cấu hình hoặc xử lý sự cố.*
