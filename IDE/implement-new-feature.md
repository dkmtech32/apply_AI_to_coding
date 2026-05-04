# Triển khai tính năng mới: ChatGPT (chiến lược) → Cursor (kế hoạch theo repo)

Dùng **ChatGPT** khi bạn cần **bản kế hoạch mức sản phẩm / kiến trúc** mà chưa cần mở codebase. Dùng **Cursor** khi bạn cần **kế hoạch gắn với file thật**, quy ước dự án và lệnh kiểm thử. Trang này mô tả cách bàn giao rõ ràng giữa hai bước.

Hướng dẫn Cursor chung (rules, MCP, quy trình hằng ngày): xem [index.md](index.md). Thêm **use case khi code** (upgrade, CI, review PR, observability, i18n/a11y, perf…): cùng file [index.md](index.md), mục _Use cases_. Playbook **debug**: [debug.md](debug.md).

---

## Vì sao chia hai bước

| Giai đoạn                        | Công cụ phù hợp         | Bạn nhận được                                                                                  |
| -------------------------------- | ----------------------- | ---------------------------------------------------------------------------------------------- |
| **Làm rõ / định hình tính năng** | ChatGPT (hoặc tương tự) | User story, ràng buộc, phương án, rủi ro, mốc triển khai—**không hallucinate đường dẫn file**. |
| **Kế hoạch triển khai**          | Cursor                  | Các bước theo file, chỗ tái sử dụng, cách kiểm tra / chạy thử—**bám `@` context thật**.        |

ChatGPT **không** thấy cây thư mục của bạn; Cursor thì **có**. Để ChatGPT quyết định _làm gì_; để Cursor quyết định _làm ở đâu_ và _làm thế nào_ trong **đúng** repo này.

---

## Bước 1 — Trên ChatGPT: lấy “bản tóm tắt tính năng” để dán sang Cursor

Yêu cầu **bản brief có cấu trúc**, không cần code. Dùng hoặc chỉnh prompt mẫu sau:

```text
Hãy đóng vai kỹ sư senior. Tôi sẽ mô tả một tính năng. Hãy trả lại:

1. Mục tiêu và phạm vi KHÔNG làm (gạch đầu dòng)
2. Hành vi người dùng thấy được / tiêu chí chấp nhận (acceptance criteria)
3. Câu hỏi mở (thứ bạn cần tôi làm rõ—tối đa 5 ý)
4. Phương án kỹ thuật (A/B) kèm đánh đổi—KHÔNG ghi đường dẫn file
5. Rủi ro (bảo mật, dữ liệu, hiệu năng, migration) và cách giảm thiểu
6. Đề xuất mốc (MVP → hoàn thiện) theo thứ tự giai đoạn

Giả định bạn KHÔNG có quyền truy cập repo của tôi. Không bịa thư viện hay đường dẫn.
```

**Bản tiếng Anh:**

```text
Act as a senior engineer. I will describe a feature. Produce:

1. Goal and non-goals (bullets)
2. User-visible behavior / acceptance criteria
3. Open questions (what you need from me—max 5)
4. Technical approach options (A/B) with tradeoffs—no file paths
5. Risks (security, data, performance, migration) and mitigations
6. Suggested milestones (MVP → polish) as ordered phases

Assume you do NOT have access to my repository. Do not invent libraries or paths.
```

Chọn **một** bản prompt (Việt hoặc Anh); điền mô tả tính năng **phía trên** khối đó. Khi ChatGPT hỏi làm rõ, trả lời cho đến khi bạn chấp nhận bản brief.

**Tuỳ chọn:** Nếu tính năng liên quan **bí mật, PII, hoặc tuân thủ**, nói rõ trong ChatGPT để brief nêu ràng buộc—sau này bạn lặp lại trong Cursor (không dán secret vào cả hai công cụ).

---

## Bước 2 — Trên Cursor: biến brief thành kế hoạch triển khai

### Mở đúng phiên

- Dùng **Plan mode** nếu muốn **duyệt kế hoạch triển khai trước** khi sửa nhiều file; không thì dùng **Chat** với cùng cấu trúc yêu cầu.
- Bắt **thread mới** để tránh model bị nhiễu ngữ cảnh cũ.

### Gắn ngữ cảnh thật

Tối thiểu nên **`@`**:

- **`@`** **service hoặc package** nơi thay đổi có khả năng nằm (thư mục—tránh cả monorepo nếu không cần).
- **`@`** **tài liệu có sẵn** (`README`, ADR, tài liệu API).
- Nếu có **ticket hoặc thiết kế**, dán link hoặc tóm tắt vào tin nhắn (và dùng MCP nếu đã cấu hình).

### Mẫu prompt (dán output ChatGPT vào sau dòng chỉ dẫn)

```text
Dưới đây là bản brief tính năng từ bên ngoài (không có quyền xem repo).

Nhiệm vụ của bạn:
1. Đối chiếu brief với repository này—chỉ ra chỗ sai hoặc thiếu.
2. Đề xuất kế hoạch triển khai có trích dẫn đường dẫn, module và pattern đã có trong codebase.
3. Liệt kê task theo thứ tự: slice nhỏ giao được trước. Mỗi task gồm: file có thể đụng, test cần thêm/sửa, cách verify local.
4. Ghi rõ “ngoài phạm vi PR này” nếu nên hoãn việc gì.

Feature brief:
[dán output ChatGPT tại đây]
```

**Bản tiếng Anh:**

```text
Below is a feature brief from an external planner (no repo access).

Your job:
1. Validate the brief against this repository—call out anything wrong or missing.
2. Propose an implementation plan that references concrete paths, modules, and patterns already in the codebase.
3. List tasks in order: smallest shippable slice first. Each task should include: files likely touched, tests to add/update, and how to verify locally.
4. Explicitly state “out of scope for this PR” if we should defer work.

Feature brief:
[paste ChatGPT output here]
```

Nếu dùng **Plan mode**, yêu cầu rõ **“một kế hoạch để tôi duyệt trước khi sửa file”** để Cursor không nhảy bước.

---

## Bước 3 — Sau khi Cursor đưa ra kế hoạch

1. **Đọc lọc “path ảo”** — mọi khẳng định quan trọng phải tương ứng file bạn mở được trong editor.
2. **Khớp rule nhóm** — nếu có `.cursor/rules`, lướt xem kế hoạch có trái quy ước không; bổ sung rule nếu lặp lại lỗ hổng.
3. **Thu hẹp scope** — chuyển mục “nice to have” sang mốc sau.
4. **Thực thi** — chuyển sang **Agent** (hoặc chấp nhận kế hoạch) và làm **từng mốc một**, có test xen giữa các bước.

---

## Checklist copy-paste

**Trước khi rời ChatGPT**

- [ ] Brief có **tiêu chí chấp nhận rõ** và **phạm vi không làm**.
- [ ] **Câu hỏi mở** đã trả lời hoặc ghi rõ hoãn lại.
- [ ] Không phụ thuộc chi tiết stack **bịa** mà bạn chưa xác nhận.

**Trước khi chạy code trong Cursor**

- [ ] Ngữ cảnh **`@`** bao phủ khu vực thay đổi chính (không phải “cả ổ đĩa”).
- [ ] Kế hoạch ghi **thư mục/file thật** hoặc nói rõ “bước khám phá trước”.
- [ ] Lệnh **verify** đúng với repo (`pnpm test`, `pytest`, v.v.).

---

## Tuỳ chọn: một dòng cầu nối

Nếu câu trả lời ChatGPT quá dài, bạn tự tóm lại rồi dán:

```text
Tóm bản brief này tối đa 10 gạch đầu dòng (mục tiêu, AC, mốc, rủi ro). Sau đó trong Cursor tôi sẽ @<thư_mục> và xin kế hoạch triển khai theo repo.
```

**Bản tiếng Anh:**

```text
Summarize this feature brief into at most 10 bullets (goal, AC, milestones, risks). Then in Cursor I will attach @<folder> and ask for a repo-specific implementation plan.
```

Cách này giữ tin nhắn đầu trong Cursor ngắn mà vẫn giữ đúng ý.

---

_Tài liệu chính thức Cursor: [cursor.com/docs](https://cursor.com/docs)._
