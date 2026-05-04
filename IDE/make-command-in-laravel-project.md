# Tạo Artisan command xử lý dữ liệu (Laravel) trong Cursor

Khi cần **job một lần** hoặc **chạy theo lịch** để: import/sync dữ liệu, sửa hàng loạt có kiểm soát, rebuild cache/cột phụ sinh, backfill migration logic—**Artisan command** thường gọn hơn route HTTP hoặc tinker ad-hoc.

Liên quan: [write-test-case.md](write-test-case.md), [debug.md](debug.md), [init-new-project.md](init-new-project.md).

---

## Khi nào nên dùng command thay vì chỗ khác

| Nên dùng Artisan command | Cân nhắc cách khác |
| ------------------------ | ------------------- |
| Chạy từ terminal / CI với flag rõ ràng | Logic chỉ phục vụ HTTP → controller/job theo luồng app |
| Khối lượng lớn, cần chunk, log, dry-run | Một dòng sửa DB → migration có `DB::statement` (có review kỹ) |
| Cần lặp lại sau deploy | Một lần tuyệt đối → có thể closure trong migration (team cho phép) |

Job queue (`ShouldQueue`) phù hợp khi chạy **lâu** hoặc cần **retry**; command vẫn có thể `dispatch` job hoặc gọi service dùng chung.

---

## Nguyên tắc thực tế

1. **Đặt tên rõ**: class + `$signature` phản ánh hành động (`data:sync-orders`, `fix:user-timezones`).
2. **Không nhồi toàn bộ domain vào `handle()`**: gọi **service/action** đã có hoặc tạo mới để dễ test.
3. **Tập dữ liệu lớn**: `chunkById` / lazy collections; tránh `Model::all()`.
4. **Giao dịch**: bọc batch trong `DB::transaction` khi nhiều bảng phải khớp nhau; cân nhắc kích thước transaction.
5. **An toàn production**: tùy chọn **`--dry-run`** (chỉ log/đếm), **`--limit`** cho thử nghiệm; xác nhận tương tác (`confirm()`) nếu lệnh nguy hiểm.
6. **Idempotent** khi có thể: chạy lại không làm hỏng dữ liệu (hoặc ghi rõ "chỉ chạy một lần" trong comment + log).
7. **Output**: `$this->info` / `error` / progress bar (`withProgressBar`) để operator biết tiến độ.

---

## Dùng Cursor đúng cách

Trước khi nhờ Agent viết command:

- **`@`** một **command mẫu** trong repo (cùng style: options, service injection).
- **`@`** **model**, **migration**, hoặc **service** liên quan dữ liệu cần sửa.
- **`@`** **`routes/console.php`** nếu cần gắn **schedule** (nêu rõ trong prompt).

Yêu cầu Agent **không** seed secret; chỉ đọc **tên biến môi trường** nếu cần.

---

## Prompt mẫu (tiếng Việt)

Dán sau khi đã `@` file liên quan. Điền các chỗ trong ngoặc vuông.

```text
Tạo Artisan command Laravel để xử lý dữ liệu sau đây.

Mục đích nghiệp vụ:
[MÔ TẢ NGẮN — ví dụ đồng bộ trạng thái đơn từ bảng X sang Y]

Hành vi mong muốn:
- [ ] Chỉ đọc / [ ] Đọc + ghi (ghi rõ bảng và cột)
- Scope: [điều kiện WHERE / tập user_id / khoảng ngày …]
- Xử lý tập lớn: chunk/lazy; không load toàn bộ bộ nhớ

Yêu cầu kỹ thuật:
- Signature artisan gợi ý: [ví dụ data:sync-order-status]
- Option: --dry-run (không ghi DB, chỉ đếm hoặc log mẫu), --limit= (tuỳ chọn)
- Logic nghiệp vụ đặt trong service/action nếu hợp lý; handle() điều phối
- Log rõ: bắt đầu, kết thúc, số bản ghi xử lý, lỗi
- Không đổi convention hiện có của repo; bám style command mẫu đã @

Chưa gắn schedule trừ khi tôi yêu cầu.

Sau khi sửa: cho biết lệnh php artisan … để chạy thử và (nếu có) cách chạy test liên quan.
```

---

## Prompt template (English)

Use after `@` the same context files. Fill in the bracketed parts.

```text
Create a Laravel Artisan command for the following data work.

Business goal:
[SHORT DESCRIPTION — e.g. backfill normalized slug from title on posts]

Expected behavior:
- [ ] Read-only / [ ] Read+write (name tables and columns)
- Scope: [WHERE clause / id range / date range / …]
- Large datasets: use chunkById or lazy collections; never load everything into memory

Technical requirements:
- Suggested signature: [e.g. data:backfill-post-slugs]
- Options: --dry-run (no writes, report counts or sample), --limit= (optional)
- Put domain logic in a service/action when reasonable; keep handle() thin
- Log start, end, rows processed, failures
- Match existing repo conventions and the sample command already attached

Do not register a schedule unless I ask.

After changes: give the exact `php artisan …` to run locally and any related test command.
```

---

## Đăng ký lịch (tuỳ chọn)

Nếu command cần chạy định kỳ, sau khi code ổn, mô tả cho Agent:

```text
Đăng ký command [TÊN_SIGNATURE] trong routes/console.php: [daily / hourly / cron …], timezone nếu cần. Không block request; nếu job dài thì tách queue.
```

**English:**

```text
Register command [SIGNATURE] in routes/console.php on [daily / hourly / cron expression], timezone if needed. Must not block HTTP; if work is heavy, dispatch a queued job instead.
```

---

## Kiểm tra trước khi merge / chạy production

- [ ] Chạy `php artisan [signature] --dry-run` (nếu có) trên DB dev/staging clone.
- [ ] Thử `--limit=10` (nếu có) trước full run.
- [ ] Có test tối thiểu (unit cho service, hoặc feature nếu command chỉ gọi service đã test) — xem [write-test-case.md](write-test-case.md).
- [ ] PR mô tả: mục đích, rủi ro, cách rollback (nếu có).
- [ ] Không log PII/secret; documentation trong README hoặc runbook nội bộ nếu command vận hành.

---

_Lệnh tạo class gợi ý: `php artisan make:command TênCommand` — giữ namespace và vị trí file theo chuẩn dự án (ví dụ modular packages)._
