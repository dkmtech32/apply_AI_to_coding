# Debug có hệ thống trong Cursor

Playbook cho **lỗi đã có triệu chứng** (local, staging, hoặc production): tái hiện → cô lập nguyên nhân → sửa tối thiểu → chống tái diễn. Khác với [implement-new-feature.md](implement-new-feature.md) (làm tính năng) và [init-new-project.md](init-new-project.md) (dự án mới). Viết test (TDD, hồi quy): [write-test-case.md](write-test-case.md).

Hướng dẫn tổng quan Cursor: [index.md](index.md) (mục *Use cases* có các mẫu ngắn, ví dụ *Fix a bug from a stack trace*).

---

## Khi nào dùng

- Hành vi sai, crash, 500, sai dữ liệu, flakey test—**đã có** log, trace, hoặc mô tả bước reproduce.
- Cần **rẽ nhánh giả thuyết** nhanh mà không refactor lan rộng.

---

## Nguyên tắc

1. **Một giả thuyết một vòng**: sửa ít, chạy lại, so kết quả.
2. **`@` phạm vi nhỏ nhất** chứa đỉnh stack hoặc module nghi ngờ; mở rộng dần nếu cần.
3. **Không đưa secret** vào chat (chỉ tên biến môi trường; log phải **che** token/PII).
4. **Chat** khi cần đọc luồng và giải thích; **Agent** khi đã rõ file cần sửa và lệnh test/build.
5. **Plan mode** nếu fix có thể chạm nhiều package hoặc migration DB.

---

## Luồng bốn bước

### 1. Ghi triệu chứng cụ thể

- Kỳ vọng vs thực tế (một câu).
- Môi trường: OS, branch, có cache không.
- **Steps reproduce** hoặc “không reproduce được ổn định—mô tả độ lệch”.

### 2. Bó hẹp (tái hiện tối thiểu)

- Một lệnh (`php artisan test tests/Feature/ExampleTest.php`, `curl`, UI click-path)—viết rõ trong prompt.
- Nếu không reproduce: nhờ Cursor đề xuất **log/instrumentation** tạm (xóa sau khi xong).

### 3. Phân tích và fix

- Dán **stack trace đầy đủ** hoặc log quanh exception.
- `@` file tại frame đầu; thêm `@` test liên quan nếu có.

### 4. Xác nhận và chống lặp

- Chạy lại cùng lệnh reproduce + suite liên quan.
- Thêm **test hồi quy** hoặc assertion log (nếu team cho phép).
- Ghi 2–4 dòng vào PR/commit message: *gốc lỗi / fix / cách verify*.

---

## Trường hợp log production / staging

- Đính kèm **request id / trace id**, thời gian (UTC), phiên bản deploy.
- Ưu tiên **một** đường dẫn code từ stack; nếu không có stack, dán đoạn log có **exception + vài dòng sau**.
- Dùng prompt **P2** bên dưới.

## Trường hợp CI / build đỏ

- Dán **log fail từ bước đầu tiên** (không chỉ dòng cuối).
- `@` workflow (`.github/workflows/…`), script, hoặc Dockerfile mà log nhắc tới.
- Dùng prompt **P3**.

---

## Prompt mẫu (copy vào Cursor)

Thay `[...]` trước khi gửi.

### P1 — Debug local (có reproduce)

```text
Bug report — help me debug in THIS repository.

## Expected vs actual
[EXPECTED] vs [ACTUAL]

## Repro steps (exact)
1. [STEP]
2. [STEP]
3. [STEP]

## Environment
- Command(s) run: [e.g. php artisan serve / php artisan test …]
- Branch / commit: [IF_KNOWN]

## Artifacts
Stack trace or error output:
[paste full trace/log]

## Task
1) Confirm repro from the steps (or suggest corrected steps).
2) Point to the failing code path with file:line references.
3) Propose the smallest fix; explain why it matches the symptoms.
4) List commands I should run to verify before/after.

Constraints: do not invent files; if uncertain, name the next folders to @. No secrets in output.
```

**Bản tiếng Việt:**

```text
Báo lỗi — hỗ trợ debug trong repo NÀY.

## Kỳ vọng vs thực tế
[KỲ_VỌNG] vs [THỰC_TẾ]

## Bước reproduce (chính xác)
1. [BƯỚC]
2. [BƯỚC]
3. [BƯỚC]

## Môi trường
- Lệnh đã chạy: [ví dụ php artisan serve / php artisan test …]
- Branch / commit: [NẾU_BIẾT]

## Đính kèm
Stack trace hoặc log lỗi đầy đủ:
[dán tại đây]

## Nhiệm vụ
1) Xác nhận reproduce từ các bước (hoặc sửa lại bước cho đúng).
2) Chỉ đường code lỗi kèm file:dòng.
3) Đề xuất fix nhỏ nhất; giải thích vì sao khớp triệu chứng.
4) Liệt kê lệnh cần chạy để verify trước/sau.

Ràng buộc: không bịa file; nếu chưa chắc, nêu thư mục cần @ tiếp. Không in secret.
```

### P2 — Debug từ log production / staging

```text
Production/staging incident — analyze using the repo.

Context:
- Env: [ENV]
- Time (UTC): [TIME]
- Release / commit: [SHA_OR_VERSION]
- Request/trace id: [ID_IF_ANY]

Logs / stack trace:
[paste]

Task:
1) Most likely root cause tied to specific code here.
2) Smallest safe mitigation and any monitoring to add.
3) How to reproduce locally or in staging.
4) Regression test or guardrail suggestion.

No secrets; redact if pasted accidentally.
```

### P3 — CI / build failure

```text
Build/CI failed. Full log:
[paste]

First failing command if known: [COMMAND]
Local repro: [YES/NO + command]

Task:
1) First root error vs noisy follow-ons.
2) Files to change in this repo.
3) Minimal patch strategy; call out breaking vs safe fixes.
4) What I should run locally to confirm green.

Keep the diff as small as possible.
```

---

## Checklist

- [ ] Có **bước reproduce** hoặc lý do rõ vì sao chưa reproduce.
- [ ] Đã **`@`** đúng vùng; diff **đã đọc** trước khi merge.
- [ ] **Không** lộ secret / PII trong prompt hoặc log thô.
- [ ] Đã chạy **verify** giống CI hoặc lệnh team quy ước.

---

*Tài liệu Cursor: [cursor.com/docs](https://cursor.com/docs).*
