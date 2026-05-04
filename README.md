## Ghi chú chung (áp dụng mọi phiên)

### 1. Chuẩn bị workspace

- **Mở đúng root project**: Luôn mở đúng thư mục gốc của project, không mở nhầm parent folder chứa nhiều repo.
- **Chờ indexing xong**: Với Cursor hoặc IDE có AI indexing, nên chờ project index xong rồi mới hỏi AI về codebase.
- **Giữ repo sạch trước khi dùng Agent**: Commit/stash thay đổi hiện tại trước khi cho AI sửa nhiều file để dễ rollback.
- **Đọc README / docs nội bộ trước**: Nếu repo có `README.md`, `AGENTS.md`, `.cursor/rules`, `CONTRIBUTING.md`, hãy cho AI đọc trước.
- **Tách task nhỏ**: Không yêu cầu “build toàn bộ feature lớn” ngay từ đầu. Chia thành plan → implement từng bước → review → test.

---

### 2. Quản lý ngữ cảnh khi prompt

- **Dùng `@` có chọn lọc**: Chỉ tag vài file/folder liên quan trực tiếp, tránh kéo cả monorepo nếu không cần.
- **Nói rõ scope**: Ví dụ: “chỉ sửa backend”, “không đụng UI”, “không đổi schema”, “không thêm package”.
- **Nêu rõ trạng thái hiện tại**: Code đang lỗi gì, muốn đạt kết quả gì, đã thử cách nào rồi.
- **Đưa input/output mong muốn**: Với API, command, scraper, tool xử lý data, nên mô tả sample input và expected output.
- **Nhắc constraint kỹ thuật**: Framework, version, DB, queue, cache, hosting, coding style, naming convention.
- **Không nhồi quá nhiều yêu cầu trong một prompt**: Nếu prompt có quá nhiều ý, AI dễ sửa lan hoặc bỏ sót.

---

### 3. Chọn đúng mode: Chat, Ask, Agent, Composer

- **Chat / Ask**: Dùng để hỏi kiến trúc, đọc hiểu code, giải thích lỗi, review logic, tìm hướng xử lý.
- **Agent / Composer**: Dùng khi cần sửa nhiều file, tạo feature, refactor, viết test, chạy command.
- **Manual first với task rủi ro cao**: Với migration DB, auth, payment, production config, nên để AI đề xuất plan trước, chưa cho sửa ngay.
- **Một phiên một mục tiêu**: Khi chuyển từ feature sang debug hoặc refactor, mở chat mới để tránh nhiễu context.
- **Dùng AI như pair programmer, không phải autopilot**: AI có thể viết nhanh, nhưng mình vẫn phải quyết định kiến trúc và review.

---

### 4. Cách ra lệnh để AI sửa code tốt hơn

- **Bắt đầu bằng plan**: Yêu cầu AI đọc code rồi đưa kế hoạch trước khi implement.
- **Yêu cầu diff nhỏ**: “Make the smallest safe change”, “avoid unrelated refactor”.
- **Yêu cầu giải thích sau khi sửa**: AI nên tóm tắt file nào đã đổi, đổi gì, vì sao.
- **Yêu cầu giữ backward compatibility**: Đặc biệt với API, DB schema, config, data import/export.
- **Yêu cầu tự kiểm tra**: Sau khi sửa, bảo AI chạy test/lint/build hoặc ít nhất nêu command cần chạy.
- **Không cho AI tự ý thêm dependency**: Trừ khi thật sự cần và phải giải thích lý do.
- **Không cho AI tự ý đổi format toàn repo**: Tránh thay đổi noise như prettier toàn bộ file không liên quan.

---

### 5. Review diff trước khi merge

- **Đọc diff như code review thật**: Không accept toàn bộ chỉ vì code “trông đúng”.
- **Kiểm tra file ngoài scope**: Nếu AI sửa file không liên quan, cần revert hoặc hỏi lý do.
- **Chú ý logic ngầm**: AI có thể sửa cho pass case hiện tại nhưng làm hỏng edge case.
- **Kiểm tra naming và convention**: AI hay tạo tên function/class không khớp style của project.
- **Kiểm tra error handling**: Đảm bảo có xử lý lỗi, log, fallback, validation.
- **Kiểm tra security**: Không expose secret, token, internal ID, SQL injection, auth bypass.
- **Chạy test/build trước khi commit**: Không merge chỉ dựa trên giải thích của AI.

---

### 6. Bảo mật và dữ liệu nhạy cảm

- **Không dán secret/PII**: Không paste API key, token, password, private key, số điện thoại thật, email khách hàng thật.
- **Dùng tên biến thay vì giá trị thật**: Ví dụ: `SHOPEE_API_KEY`, `DATABASE_URL`, `JWT_SECRET`.
- **Ẩn file nhạy cảm khỏi AI context**: Dùng `.gitignore`, `.cursorignore`, hoặc không tag file đó.
- **Không paste production dump**: Nếu cần debug data, tạo sample đã anonymize.
- **Cẩn thận với log**: Log có thể chứa token, phone, email, transaction id, cookie.
- **Không để AI viết config production khi chưa review**: Đặc biệt Nginx, Docker, CI/CD, Cloudflare, payment, auth.

---

### 7. Rules và coding convention

- **Ưu tiên rule trong repo**: `.cursor/rules`, `AGENTS.md`, `README.md` quan trọng hơn note chung.
- **Viết rule ngắn, rõ, có ví dụ**: Rule quá dài dễ bị AI bỏ qua.
- **Tách rule theo stack**: Laravel, Next.js, scraper, SQL, test, UI có thể có rule riêng.
- **Ghi rõ “không được làm”**: Ví dụ: “không dùng raw SQL nếu đã có repository pattern”.
- **Cập nhật rule sau mỗi lỗi lặp lại**: Nếu AI hay sai một kiểu, biến nó thành rule.
- **Đừng rule hóa mọi thứ**: Rule quá nhiều làm AI chậm và dễ nhiễu.

---

### 8. Debug với AI

- **Đưa lỗi đầy đủ**: Bao gồm stack trace, command đã chạy, environment, file liên quan.
- **Nói rõ lỗi xảy ra khi nào**: Khi build, runtime, request API, queue job, cron, browser extension, crawler step.
- **Yêu cầu phân tích nguyên nhân trước**: Không nên để AI sửa ngay khi chưa hiểu root cause.
- **Cho AI tạo checklist debug**: Giúp tránh đoán mò.
- **Tách reproduction case nhỏ**: Với bug phức tạp, yêu cầu AI tạo minimal reproduction hoặc test case.
- **So sánh expected vs actual**: Đây là phần AI cần nhất để debug chính xác.
- **Không fix bằng workaround mù**: Nếu AI thêm delay/retry/random fallback, cần hỏi vì sao.

---

### 9. Viết feature mới

- **Bắt đầu từ spec nhỏ**: User story, input/output, edge cases, permission, UI state, API contract.
- **Thiết kế data trước khi code**: Với feature có DB, cần schema/model/migration trước.
- **Chốt API contract trước UI**: Tránh UI làm xong rồi backend trả format khác.
- **Yêu cầu AI tìm điểm tích hợp trong repo**: File route, controller, service, component, store, test hiện có.
- **Implement từng lớp**: DB → backend service → API → UI → test.
- **Có checklist nghiệm thu**: Ví dụ: create/update/delete, validation, permission, empty state, loading, error.
- **Không để AI refactor lớn trong lúc thêm feature nhỏ**.

---

### 10. Refactor

- **Refactor phải có mục tiêu rõ**: Dễ đọc hơn, tách service, giảm duplicate, tăng testability, tối ưu performance.
- **Không refactor khi chưa có test hoặc checklist manual test**.
- **Giữ behavior cũ**: Refactor không nên đổi logic nghiệp vụ nếu không yêu cầu.
- **Refactor theo bước nhỏ**: Đổi tên → tách hàm → tách class → update test.
- **Yêu cầu AI nêu rủi ro**: Những chỗ có thể bị ảnh hưởng sau refactor.
- **Tránh “big bang refactor”**: AI rất dễ đổi quá nhiều file khiến review khó.

---

### 11. Test với AI

- **Yêu cầu test cho behavior, không chỉ coverage**.
- **Cho AI đọc test hiện có trước**: Để giữ style test của project.
- **Test edge cases**: Empty input, invalid input, permission denied, duplicate data, network fail, timeout.
- **Viết test trước khi fix bug**: Với bug quan trọng, yêu cầu tạo failing test rồi mới sửa.
- **Không tin test do AI viết 100%**: AI có thể mock sai hoặc assert quá yếu.
- **Sau khi test pass, kiểm tra test có thật sự bắt được lỗi không**.

---

### 12. Làm việc với database

- **Không cho AI chạy destructive command nếu chưa review**: `DROP`, `TRUNCATE`, mass `DELETE`, migration rollback production.
- **Luôn yêu cầu backup plan**: Với migration hoặc data update.
- **Preview query trước khi update/delete**: Dùng `SELECT` kiểm tra trước.
- **Với import dữ liệu, cần dry-run mode**.
- **Log số dòng affected**: Để dễ audit.
- **Tách logic mapping rõ ràng**: Input column → DB column → transform rule → validation.
- **Cẩn thận timezone**: Nói rõ UTC, UTC+7, server time, DB time.

---

### 13. Làm việc với scraper / crawler

- **Không bắt đầu bằng code ngay**: Đầu tiên cần xác định nguồn data: DOM, API response, HTML, JSON-LD, sitemap, network request.
- **Ưu tiên API-first nếu site là React/Next.js/Tailwind**.
- **Dùng Playwright khi cần login, scroll, click, pagination, captcha handoff**.
- **Tách crawler thành step rõ ràng**: Discover URL → fetch/listen API → parse → normalize → dedupe → save.
- **Yêu cầu AI tạo schema output trước**: Tránh crawler lấy data lung tung.
- **Có retry/rate limit/dedupe**: Không chỉ viết happy path.
- **Log đủ để debug**: URL, page, status, item count, error reason.
- **Không hardcode selector quá cụ thể nếu class dynamic**.
- **Với extension crawler, giữ extension mỏng**: Logic quan trọng nên đưa về backend nếu cần bảo vệ.

---

### 14. Làm việc với UI/UX

- **Yêu cầu AI mô tả user flow trước khi code UI**.
- **Thiết kế state rõ ràng**: Empty, loading, success, error, permission required, logged out.
- **Ưu tiên non-tech user**: Label dễ hiểu, ít field, CTA rõ.
- **Không expose technical config nếu user không cần biết**.
- **Có progress state khi task chạy lâu**: Ví dụ crawler, import, export, AI generation.
- **Có preview trước khi submit**: Với invoice, campaign, message, email, import data.
- **Yêu cầu responsive ngay từ đầu**.
- **Review copywriting**: AI hay viết UI text dài hoặc quá technical.

---

### 15. Làm việc với command / automation

- **Command nên có dry-run**.
- **Có option giới hạn số record**: `--limit`, `--from`, `--to`, `--service-id`.
- **Có log rõ ràng**: Started, processed, skipped, failed, completed.
- **Có resume/retry nếu xử lý data lớn**.
- **Không xử lý production data trực tiếp khi chưa test trên bản copy**.
- **Với cronjob, cần idempotent**: Chạy lại không làm duplicate hoặc sai data.
- **Có lock nếu command không được chạy song song**.

---

### 16. Làm việc với Git

- **Commit trước khi cho AI sửa lớn**.
- **Dùng branch riêng cho từng thử nghiệm AI**.
- **Không accept diff quá lớn trong một lần**.
- **Commit message nên ghi rõ phần AI hỗ trợ** nếu team cần audit.
- **Dễ rollback**: Nếu AI làm sai, revert nhanh thay vì cố sửa trên đống diff lớn.
- **Review generated files**: AI có thể tạo file thừa, file duplicate, hoặc đặt sai folder.

---

### 17. Khi dùng ChatGPT trước Cursor

- **Dùng ChatGPT để lên plan/spec trước**.
- **Sau đó copy plan sang Cursor để plan theo repo thực tế**.
- **ChatGPT phù hợp để thiết kế kiến trúc, DB, flow, prompt, checklist**.
- **Cursor phù hợp để đọc repo, sửa file, chạy lệnh, viết test**.
- **Không nên vừa thiếu spec vừa yêu cầu Cursor code ngay**.
- **Sau khi Cursor implement, có thể paste diff hoặc lỗi lại ChatGPT để review độc lập**.

---

### 18. Khi nào nên mở chat mới

- Khi chuyển từ feature này sang feature khác.
- Khi debug xong và bắt đầu refactor.
- Khi context đã quá dài, AI bắt đầu trả lời sai scope.
- Khi đổi stack hoặc đổi repo.
- Khi muốn AI quên hướng suy luận cũ.
- Khi prompt trước đó có nhiều assumption sai.
- Khi cần tạo plan sạch từ đầu.

---

### 19. Prompt pattern nên dùng

- **Plan trước, code sau**:

  “Read these files and propose an implementation plan first. Do not edit code yet.”

- **Sửa nhỏ, không lan scope**:

  “Make the smallest possible change to fix this issue. Do not refactor unrelated code.”

- **Debug có hệ thống**:

  “Analyze the root cause first. List likely causes, how to verify each one, then suggest the safest fix.”

- **Review diff**:

  “Review this diff like a senior engineer. Look for bugs, security issues, edge cases, and unnecessary changes.”

- **Viết test**:

  “Add tests that cover the expected behavior and edge cases. Follow the existing test style in this repo.”

- **Tạo command xử lý data**:

  “Create a command with dry-run, limit, date range filter, logging, and safe error handling.”

---

### 20. Những lỗi AI thường gặp

- Sửa quá nhiều file ngoài scope.
- Tự thêm dependency không cần thiết.
- Đổi API contract mà không báo.
- Bỏ qua edge case hoặc permission.
- Viết code pass happy path nhưng thiếu error handling.
- Tạo abstraction quá sớm.
- Dùng selector DOM quá fragile.
- Viết test yếu, chỉ test mock thay vì behavior.
- Quên timezone.
- Quên backward compatibility.
- Tạo duplicate logic thay vì reuse service có sẵn.
- Viết UI đẹp nhưng workflow khó dùng.
- Dùng tên biến/function không theo convention repo.
- Xóa code “có vẻ thừa” nhưng thật ra đang được dùng ở nơi khác.

---

### 21. Nguyên tắc an toàn khi dùng AI để code

- AI được phép đề xuất, nhưng người dev chịu trách nhiệm merge.
- Càng gần production, càng cần review kỹ.
- Task càng mơ hồ, AI càng dễ tự bịa scope.
- Prompt tốt thường gồm: context + mục tiêu + constraint + file liên quan + expected output.
- Không dùng AI để che giấu việc mình không hiểu code. Dùng AI để hiểu code nhanh hơn.
---

### 22. Sử dụng Agent Skills

- **Tự động hóa tiêu chuẩn**: Sử dụng [Agent Skills](./IDE/how-to-use-skill.md) để dạy cho AI agent các quy trình chuyên biệt như deploy, quản lý hạ tầng, hoặc áp dụng kiến trúc phức tạp.
- **Tính nhất quán**: Đảm bảo AI luôn tuân thủ đúng "Anatomy" và "Workflow" của dự án mà không cần nhắc lại trong prompt.
- **Mở rộng khả năng**: Sử dụng lệnh `npx skills add` để tích hợp các bộ kỹ năng từ cộng đồng (như Vercel Labs, Laravel Cloud).
- **Tài liệu chi tiết**: Xem [Hướng dẫn thiết lập Agent Skills](./IDE/how-to-use-skill.md) để biết cách áp dụng cho React, Laravel và Cloudflare.
