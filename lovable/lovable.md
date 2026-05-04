# Lovable: hướng dẫn dựng frontend (kèm prompt tiếng Anh)

[Lovable](https://lovable.dev) giúp tạo **UI web tương tác** nhanh (thường **React + Vite + TypeScript + Tailwind**, thường kèm **shadcn/ui**). Bạn dùng Lovable để **khóa IA, layout, luồng và cảm giác UX**; sau đó đưa code sang **repo thật** (GitHub hoặc copy) và tinh chỉnh trong **Cursor** với lint, kiến trúc và API thật.

Tài liệu chính: [Lovable Docs](https://docs.lovable.dev/) · [GitHub integration](https://docs.lovable.dev/integrations/github) · [Export / FAQ](https://lovable.dev/faq/code-export).

---

## Khi nào nên dùng Lovable

- Cần **demo hoặc MVP UI** trước khi gắn backend phức tạp.
- Muốn **thống nhất navigation + key screens** với PM/design mà chưa cần repo production.
- Chấp nhận “Lovable sinh code”—bạn sẽ **review, refactor và tách data layer** ở Cursor.

---

## Quy trình đề xuất

1. **Mở project mới** trên Lovable; dán **Prompt A** (bootstrap) ở dưới, điền các placeholder dạng `[APP_NAME]`, `[ROUTES]`, v.v.
2. **Bổ sung từng màn / luồng** bằng **Prompt B**; lặp cho đến khi đủ MVP.
3. **Dọn UX** bằng **Prompt C** (responsive, a11y, empty/error, copy).
4. **Chốt dữ liệu giả** bằng **Prompt D** để sang Cursor không bị “field bịa”.
5. **Xuất code**: kết nối **GitHub** hoặc tải/copy theo tài liệu Lovable; **không** coi `.env` / secret là đã xong—cấu hình lại ngoài Lovable.
6. Trong Cursor: clone hoặc copy vào repo; **`@`** thư mục đó và dùng **Prompt E** (handoff) nếu cần viết lại theo chuẩn team.

Chi tiết tích hợp với Cursor và backend: [init-new-project.md](init-new-project.md).

---

## Nguyên tắc khi prompt trong Lovable

- **Một mục tiêu mỗi lần**: “Thêm trang X” tốt hơn “Làm hết app”.
- **Nêu rõ vai trò người dùng** và **đường đi chính** (happy path).
- **Liệt kê trạng thái UI**: loading, empty, error, success; không để Lovable chỉ làm happy path.
- **Constraints**: ví dụ “no external paid API”, “desktop-first”, “keyboard navigable”.
- **Không nhét secret** vào prompt; dùng placeholder `YOUR_API_BASE_URL`.

---

## Prompt mẫu (tiếng Anh — copy vào Lovable)

Placeholders bạn thay: `[APP_NAME]`, `[AUDIENCE]`, `[PRIMARY_GOAL]`, `[ROUTES]`, v.v.

### Prompt A — Bootstrap project (shell + IA)

```text
You are building a web app in this Lovable project.

## Product
- Name: [APP_NAME]
- Audience: [AUDIENCE]
- Primary user goal: [PRIMARY_GOAL]
- Non-goals (explicit): [NON_GOALS]

## Information architecture
- Routes / pages (names + one-line purpose): [ROUTES]
- Global navigation: [sidebar | top nav | tabs] (pick one and justify briefly)

## UX / UI requirements
- Visual style: [minimal | dense dashboard | playful | corporate] + brand colors if any: [COLORS_OR_NONE]
- Must be responsive: mobile + desktop
- Accessibility: keyboard navigable, visible focus states, semantic headings, form labels
- Include: app shell (header/nav), routing between listed pages, consistent spacing/typography

## Engineering constraints (frontend only for now)
- Prefer clean component boundaries: presentational components separate from data fetching hooks (use mock data in /mocks or inline fixtures with clear TODO comments)
- No real backend integration; use realistic mock JSON matching these domain entities: [ENTITIES]
- No third-party auth providers unless I explicitly ask later

## Deliverables now
1) Implement the shell + empty pages wired to routes (placeholder content OK)
2) Add a short README-style comment block at the top of the main layout describing routes (as code comments)
3) List any open questions as // TODO comments in one central file (e.g., src/open-questions.ts)

Do NOT invent features outside the listed routes and goals.
```

### Prompt B — One screen / flow (chi tiết)

```text
Implement the following page/flow in the existing app:

## Page
- Route: [ROUTE_PATH]
- User story: As a [PERSONA], I want [ACTION] so that [OUTCOME].

## UI sections (top → bottom)
[SECTION_1]
[SECTION_2]
[SECTION_3]

## States to implement (all required)
- Loading
- Empty (helpful CTA copy)
- Error (retry + friendly message)
- Success / populated

## Interactions
- Primary CTA: [CTA_NAME] → [WHAT_HAPPENS_NEXT]
- Secondary actions: [ACTIONS]
- Form validation rules: [RULES]

## Data (mock only)
Use TypeScript types and mock data shaped like:
[PASTE_TYPE_OR_JSON_SHAPE]

## Constraints
- Keep styling consistent with the existing layout
- Avoid adding new routes unless strictly required
- Prefer reusable components if the same pattern appears twice

After changes, summarize what you changed in 5 bullets (for my changelog).
```

### Prompt C — Polish pass (responsive, a11y, copy)

```text
Polish pass on the whole app (no new features):

## Responsive
- Fix breakpoints for small screens for: [LIST_PAGES_OR_COMPONENTS]
- Ensure tables/lists collapse gracefully (card layout on mobile if needed)

## Accessibility
- Audit focus order on modals/drawers
- Ensure icon-only buttons have aria-labels
- Color contrast for text vs background (WCAG AA target)

## Copy
- Replace vague button labels like “Submit” with action-specific text
- Improve empty states with 1 sentence guidance + one CTA

## Performance / hygiene
- Remove dead code and unused imports introduced during iteration
- Keep bundle reasonable: avoid new heavy dependencies unless necessary

Do not change the route structure unless there is a clear broken UX bug.
```

### Prompt D — Mock API contract (cho handoff Cursor/backend)

```text
Define a stable mock “API contract” for the frontend (still mock data, but structured as if real):

## Endpoints (REST shape)
[List endpoints like:]
- GET /api/[resource] → returns [SHAPE]
- POST /api/[resource] → body [SHAPE] → returns [SHAPE]

## Types
Generate TypeScript interfaces/types in src/types/[domain].ts matching the shapes exactly.

## Mock layer
- Centralize mocks in src/mocks/[domain].ts
- Add a small fetch wrapper (e.g., src/lib/api.ts) that currently returns mocks but is written to swap to real fetch later (single place to change base URL)

## Rules
- No secrets in code; base URL from import.meta.env.VITE_API_BASE_URL with a safe default for local dev
- Include error shape: { code: string; message: string; details?: unknown }

Do not implement a real server—only client-side mocks and types.
```

### Prompt E — Handoff tóm tắt cho Cursor (dán sau khi đã có repo/code)

Dùng trong Cursor (Chat/Agent), kèm `@` thư mục code Lovable:

```text
This folder contains UI generated from Lovable (React/Vite/TS/Tailwind). Treat it as a reference implementation.

Goals:
1) Match the UX and layout closely, but refactor to our repo conventions: [CONVENTIONS]
2) Replace mock data layer with our real API client: [API_STYLE e.g., OpenAPI fetch / tRPC]
3) Split “view components” vs “data hooks”; no business logic hidden in huge page files
4) Preserve routes: [LIST_ROUTES] unless you find a clear naming conflict—if so, propose a migration map
5) Add tests for critical UI states if our stack supports it: [TEST_TOOLS]

Out of scope: redesign / new features. Fix TypeScript/lint issues introduced by refactor only as needed.

Start with a short repo-specific implementation plan, then execute in small commits/steps.
```

---

## Checklist trước khi rời Lovable

- [ ] Đủ **trạng thái UI**: loading / empty / error cho màn quan trọng.
- [ ] **Mobile** dùng được cho luồng chính.
- [ ] **Mock type** thống nhất; không copy field “tạm bợ” khó thay bằng API thật.
- [ ] Đã **export / sync GitHub** (hoặc có bản zip/copy) và bạn biết cách chạy `npm`/`pnpm` local.
- [ ] Đã **ghi chú** việc phải tạo lại `.env` và secret — không có trong export.

---

## Lưu ý licensing và sở hữu code

Theo tài liệu Lovable, bạn sở hữu code tạo ra; vẫn nên đọc điều khoản hiện tại trên [lovable.dev](https://lovable.dev) và FAQ export trước khi ship sản phẩm thương mại.
