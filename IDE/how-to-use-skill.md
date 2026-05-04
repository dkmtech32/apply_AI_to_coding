# Hướng dẫn: Thiết lập Agent Skills Chuyên nghiệp

Tài liệu này cung cấp cái nhìn toàn diện về cách triển khai và sử dụng **Agent Skills** để tối ưu hóa hiệu suất và tính nhất quán của các AI coding agent trên nhiều nền tảng công nghệ khác nhau.

## 1. Agent Skills là gì?

Agent Skills là tập hợp các hướng dẫn, khuôn mẫu (patterns) và kiến thức chuyên môn mà bạn "dạy" cho AI agent của mình. Bằng cách thêm một skill, bạn đảm bảo agent sẽ tuân thủ kiến trúc cụ thể, tiêu chuẩn lập trình và các best practices của dự án mà không cần phải nhắc lại trong mỗi câu lệnh (prompt).

### Bắt đầu nhanh
Để thêm một bộ sưu tập skill vào dự án của bạn, hãy sử dụng lệnh sau:
```bash
npx skills add <owner/repo>
```
*Ví dụ (Bộ sưu tập từ Vercel Labs):*
```bash
npx skills add vercel-labs/agent-skills
```

---

## 2. Use Case 1: Frontend (React) - Tuân thủ Design System & FSD

Trong dự án React, skills được sử dụng để thực thi các mô hình kiến trúc như **Feature-Sliced Design (FSD)** và đảm bảo agent sử dụng thư viện component của bạn một cách chính xác.

### Triển khai: `skills/react-frontend.md`
```markdown
# Kỹ năng Phát triển Frontend React

## Mục tiêu
Hướng dẫn agent xây dựng các UI component bằng React, Tailwind CSS và Shadcn/UI, tuân theo kiến trúc FSD.

## Quy ước đặt tên
- Components: `PascalCase` (ví dụ: `UserAvatar.tsx`)
- Hooks: `camelCase` với tiền tố `use` (ví dụ: `useAuth.ts`)
- Thư mục: `kebab-case` (ví dụ: `feature-auth`)

## Quy tắc kiến trúc (FSD)
1. **Lớp (Layers)**: `shared` (ui, lib), `entities`, `features`, `widgets`, `pages`.
2. **Public API**: Mỗi module phải có một tệp `index.ts` chỉ xuất (export) những phần cần thiết.
3. **Không import chéo**: Một feature không được phép import trực tiếp từ một feature khác; phải sử dụng shared entity hoặc widget.

## Quy tắc thiết kế
- Sử dụng `lucide-react` cho icons.
- Luôn triển khai thiết kế đáp ứng (responsive) với các tiền tố Tailwind: `sm`, `md`, `lg`.
- Sử dụng tiện ích `cn()` để gộp class có điều kiện.
```

---

## 3. Use Case 2: Backend (Laravel) - Tiêu chuẩn & Hạ tầng Cloud

Đối với các nhà phát triển Laravel, skills giúp thu hẹp khoảng cách giữa logic ứng dụng và quản lý hạ tầng thông qua **Laravel Cloud**.

### Triển khai: `skills/laravel-backend.md`
```markdown
# Kỹ năng Backend Laravel & Cloud

## Mục tiêu
Quản lý Laravel models, migrations và triển khai trên Laravel Cloud theo các tiêu chuẩn tốt nhất.

## Tiêu chuẩn lập trình
- **Models**: Sử dụng Typed Properties và khai báo `$fillable`.
- **Migrations**: Luôn bao gồm `timestamps()` và sử dụng `foreignId()` với `constrained()`.
- **API**: Tuân thủ tiêu chuẩn RESTful; sử dụng `AnonymousResourceCollection` cho danh sách.

## Các lệnh Laravel Cloud
Khi được yêu cầu triển khai hoặc quản lý hạ tầng:
1. Kiểm tra môi trường hiện tại bằng `php artisan cloud:env`.
2. Chạy migration qua `php artisan cloud:migrate`.
3. Xóa cache bằng `php artisan cloud:clear`.

## Ví dụ
"Tạo model UserProfile có quan hệ 1-1 với User, sau đó triển khai lên production."
-> Agent sẽ tự động tạo Model, Migration và nhắc lệnh triển khai tương ứng.
```

---

## 4. Use Case 3: Custom Tool - Onboard Domain trên Cloudflare

Đây là một skill chuyên biệt cho các tác vụ DevOps. Nó dạy cho agent cách tương tác với hệ sinh thái Cloudflare bằng CLI `wrangler`.

### Triển khai: `skills/cloudflare-onboard.md`
```markdown
# Kỹ năng Onboard Domain Cloudflare

## Mục tiêu
Tự động hóa quy trình thêm, cấu hình và xác thực domain trên Cloudflare.

## Điều kiện tiên quyết
- Cloudflare API Token phải được thiết lập trong biến môi trường.
- `wrangler` CLI phải được cài đặt (global hoặc dev dependency).

## Quy trình thực hiện
1. **Thêm Domain**: Sử dụng lệnh `wrangler domains add <domain>`.
2. **Cấu hình DNS**:
   - Đối với Workers: Ánh xạ domain vào worker script.
   - Đối với Static: Cấu hình các bản ghi CNAME.
3. **Xác thực**: Kiểm tra trạng thái domain bằng `wrangler domains list`.

## Logic tương tác
- Nếu người dùng yêu cầu "onboard domain mới", hãy hỏi tên miền và tên worker mục tiêu.
- Kiểm tra xem domain đã tồn tại trong tài khoản chưa trước khi thực hiện thêm mới.
- Cung cấp các DNS Nameservers cho người dùng nếu domain nằm ở nhà đăng ký bên ngoài.

## Xử lý lỗi thường gặp
- Lỗi "Domain already exists": Bỏ qua bước thêm và chuyển sang bước cấu hình.
- Lỗi "Unauthorized": Nhắc người dùng chạy lệnh `npx wrangler login`.
```

---

## 5. Cách Quản lý Skills

- **Liệt kê Skills**: Xem những skill nào đang hoạt động trong phiên làm việc.
  ```bash
  npx skills list
  ```
- **Tìm kiếm Skills mới**: Khám phá thư viện từ cộng đồng.
  ```bash
  npx skills find
  ```
- **Cập nhật**: Đồng bộ hóa với phiên bản mới nhất từ repository.
  ```bash
  npx skills update
  ```

---

### Tài nguyên tham khảo
- [Danh mục Laravel Skills](https://skills.laravel.cloud/)
- [Vercel Agent Skills GitHub](https://github.com/vercel-labs/agent-skills)

> [!TIP]
> Các nhà phát triển chuyên nghiệp thường lưu trữ thư mục `.skills/` trong Git để đảm bảo toàn bộ đội ngũ (và các agent của họ) luôn đồng bộ với cùng một tiêu chuẩn lập trình.
