# Viết test case trong Cursor khi implement thực tế

Tài liệu này dùng cho trường hợp dev đang làm task thật trong repo và muốn dùng **Cursor Agent** để thêm test, sửa test hoặc viết test trước khi implement.

Mục tiêu không phải là “viết thật nhiều test”, mà là:

- Test đúng phần đang thay đổi.
- Bám convention hiện có của repo.
- Chạy được bằng command thực tế trong project.
- Giúp dev verify task trước khi merge.
- Tránh để Cursor tự đoán framework, folder, naming hoặc mock sai.

Các ví dụ đường dẫn và lệnh dưới đây bám **Laravel**: PHPUnit là mặc định; nếu team dùng **Pest**, file vẫn trong `tests/` và lệnh vẫn là `php artisan test`.

---

## Khi nào nên dùng Cursor để viết test

Nên dùng Cursor viết test trong các trường hợp sau:

| Trường hợp                             | Nên làm gì                                                |
| -------------------------------------- | --------------------------------------------------------- |
| Implement feature mới                  | Viết test cho behavior chính trước hoặc ngay sau khi code |
| Fix bug                                | Viết regression test để bug không quay lại                |
| Refactor code cũ                       | Viết characterization test trước khi sửa logic            |
| Thêm API mới                           | Viết integration / contract test cho request-response     |
| Thêm logic tính toán                   | Viết unit test cho các case bình thường và case biên      |
| Sửa UI flow quan trọng                 | Viết UI test hoặc smoke test                              |
| Đụng payment, wallet, permission, auth | Bắt buộc có test cho case lỗi và edge case                |

Không nên dùng Cursor viết test kiểu chung chung như:

```text
Write tests for the whole project.
```

Prompt này quá rộng, Cursor dễ tạo test vô dụng, sai convention hoặc không chạy được.

---

## Nguyên tắc thực tế

### 1. Luôn đưa file thật vào context

Khi prompt Cursor, nên `@` ít nhất 2 loại file:

```text
@app/Services/CampaignService.php
@tests/Unit/CampaignServiceTest.php
```

Một file là **code cần test**, một file là **test mẫu cùng style**.

Nếu chưa có test mẫu gần đó, chọn test ở module khác nhưng cùng framework.

Ví dụ:

```text
@tests/Unit/UserServiceTest.php
@app/Services/CampaignService.php
```

---

### 2. Bắt Cursor đọc trước, không sửa ngay

Trước khi cho Agent viết test, nên yêu cầu nó phân tích:

```text
First inspect the implementation and existing test style. Do not edit files yet.
Tell me:
1. Which behavior should be tested
2. Which test file should be created or updated
3. Which command should be used to run the related test
```

Làm vậy giúp tránh tình trạng Agent tạo file test sai chỗ hoặc dùng nhầm framework.

---

### 3. Test theo behavior, không test implementation detail

Nên test:

```text
When campaign daily budget is exceeded, campaign should not be eligible for serving.
```

Không nên test kiểu:

```text
Should call private method calculateBudgetStatus.
```

Test nên khóa behavior mà user/business quan tâm, không khóa chi tiết code nội bộ.

---

### 4. Một test case chỉ nên kiểm tra một behavior chính

Ví dụ tốt:

```text
it should reject campaign when daily budget is exceeded
```

Ví dụ không tốt:

```text
it should create campaign, update budget, calculate report, send notification and write logs
```

Nếu test quá nhiều thứ cùng lúc, khi fail sẽ khó biết lỗi nằm ở đâu.

---

### 5. Unit test không gọi network thật

Với unit test, phải mock các phần sau:

- HTTP API bên ngoài
- Database thật
- Queue thật
- Payment gateway
- AI provider
- Email/SMS service
- Time/random nếu ảnh hưởng kết quả

Integration test có thể dùng DB test, nhưng phải rõ setup/teardown.

---

### 6. Luôn yêu cầu Cursor đưa command chạy test

Sau khi viết test, Cursor phải trả về command cụ thể:

```bash
php artisan test --filter=CampaignServiceTest
```

Hoặc chạy theo file:

```bash
php artisan test tests/Unit/CampaignServiceTest.php
```

Hoặc gọi trực tiếp PHPUnit (khi team không dùng wrapper):

```bash
./vendor/bin/phpunit tests/Unit/CampaignServiceTest.php
```

Không chấp nhận câu chung chung như:

```text
Run the tests.
```

---

## Workflow thực tế khi implement feature mới

Dùng workflow này khi bạn đang thêm feature thật.

### Bước 1: Cho Cursor hiểu task

Prompt:

```text
We are implementing this feature:

[FEATURE DESCRIPTION]

Before writing code, inspect the existing implementation and tests.

Context:
@path/to/production/file
@path/to/sample/test/file

Do not edit files yet.

Return:
1. Current behavior
2. Missing behavior
3. Test cases we should add
4. Suggested test file path
5. Test command to run
```

---

### Bước 2: Yêu cầu viết test trước

Prompt:

```text
Now add tests for the agreed behavior.

Rules:
- Follow the style of the existing test file.
- Do not refactor production code yet.
- Mock external dependencies.
- Keep fixtures minimal.
- Make the tests fail if the feature is not implemented.

After editing, summarize:
1. Files changed
2. Test cases added
3. Command to run this test only
```

---

### Bước 3: Chạy test đỏ

Dev chạy command mà Cursor đưa.

Kết quả mong muốn:

- Test fail vì behavior chưa implement.
- Không fail vì import sai, mock sai, setup sai.

Nếu test fail vì setup sai, yêu cầu Cursor sửa test trước, chưa implement logic.

Prompt:

```text
The test fails because of setup/import/mock issue, not because of missing business behavior.

Error:
[PASTE ERROR]

Fix the test setup only. Do not implement production logic yet.
```

---

### Bước 4: Implement tối thiểu cho test xanh

Prompt:

```text
Now implement the smallest production code change to make the tests pass.

Rules:
- Do not change test expectations unless they are clearly wrong.
- Keep the change minimal.
- Do not introduce new dependencies.
- Follow existing project patterns.

After editing, summarize:
1. Production files changed
2. Why the implementation satisfies the tests
3. Any risk or edge case not covered
```

---

### Bước 5: Chạy test lại và review diff

Sau khi test xanh, yêu cầu Cursor tự review:

```text
Review the diff you made.

Check:
1. Is the test meaningful?
2. Is the production change minimal?
3. Are there missing edge cases?
4. Are there flaky parts?
5. Are there any security or permission concerns?

Do not edit yet. Just report findings.
```

---

## Workflow thực tế khi fix bug

Dùng khi có bug production hoặc bug QA report.

### Bước 1: Mô tả bug bằng input/output

Prompt:

```text
We need to fix a bug.

Bug:
[WHAT IS WRONG]

Current wrong behavior:
[ACTUAL RESULT]

Expected behavior:
[EXPECTED RESULT]

Repro input:
[INPUT / REQUEST / LOG / DATA]

Context:
@path/to/buggy/file
@path/to/related/test/file

Task:
1. Inspect the code
2. Identify where the bug likely happens
3. Propose the smallest regression test
4. Do not edit files yet
```

---

### Bước 2: Viết regression test

Prompt:

```text
Add a regression test for this bug.

Rules:
- The test must fail on the current wrong behavior.
- The test must pass after the correct fix.
- Keep the test focused only on this bug.
- Do not fix production code yet.

Return:
1. Test file changed
2. Test name added
3. Command to run only this test
```

---

### Bước 3: Fix bug

Prompt:

```text
Now fix the bug with the smallest safe change.

Rules:
- Do not broaden the scope.
- Do not change unrelated behavior.
- Keep existing public API unchanged.
- If the fix affects other cases, mention them.

After editing, summarize:
1. Root cause
2. Fix
3. Tests to run
```

---

## Workflow thực tế khi refactor legacy code

Với legacy code, đừng bảo Cursor refactor ngay.

Sai:

```text
Refactor this whole module.
```

Đúng:

```text
We need to refactor this module, but first protect current behavior with characterization tests.

Context:
@path/to/legacy/module
@path/to/sample/test

Task:
1. Identify important current behaviors
2. Add characterization tests for those behaviors
3. Do not change production code yet
4. Mark the tests with comments that they capture current behavior before refactor
```

Sau khi test xanh mới refactor:

```text
Now refactor the module while keeping all characterization tests passing.

Rules:
- No behavior change.
- No API change.
- Keep commits small.
- After refactor, explain what changed structurally.
```

---

## Các loại test nên dùng

### Unit test

Dùng cho:

- Function tính toán
- Service logic
- Parser
- Validator
- Mapper
- Normalizer
- State machine

Ví dụ thực tế:

```text
Test variant ETA calculator:
- 1 layer variant
- 2 layer variant
- no variant
- invalid quantity
```

---

### Integration test

Dùng cho:

- API endpoint
- Database query
- Repository
- Service dùng DB test
- Auth middleware
- Permission flow

Ví dụ:

```text
POST /campaigns should create campaign with valid targeting rules.
```

---

### Contract test

Dùng cho:

- API response shape
- Client SDK
- Webhook payload
- External integration mock
- Schema không được đổi âm thầm

Ví dụ:

```text
Delivery report API must always return:
- telco_name
- total_dn
- success_count
- error_count
- success_rate_percent
```

---

### UI test

Dùng cho:

- Form quan trọng
- Flow tạo campaign
- Login
- Checkout
- Upload/import
- Dashboard filter

Nên assert theo nội dung hoặc nhãn form mà user thấy, không khóa vào class CSS.

Tốt (Feature test HTTP / Blade):

```text
$response->assertOk();
$response->assertSee('Create Campaign');
```

Hoặc với **Laravel Dusk** (E2E trình duyệt): assert theo text nút/link user đọc được, không theo selector class họa tiết.

Không tốt:

```text
Chỉ assert theo class CSS (ví dụ .btn-primary.mt-4) thay vì text/label người dùng thấy
```

---

## Prompt mẫu dùng cho Cursor

### Prompt 1: Thêm test cho module

```text
Add automated tests for this module.

Context:
@path/to/production/file
@path/to/sample/test/file

Test type:
[UNIT | INTEGRATION]

Behavior to cover:
1. [Happy path]
2. [Edge case]
3. [Error case]

Rules:
- Follow the existing test style from the sample file.
- Use the existing test framework only.
- Do not call external network services.
- Mock dependencies at the boundary.
- Keep fixtures small.
- Do not make large production changes for testability.
- If the code is hard to test, stop and propose the smallest refactor seam first.

Deliverables:
1. Test files created or updated
2. Short explanation of each test case
3. Exact command to run this test only
4. Any wider test command that should be run before merge
```

---

### Prompt 2: Viết regression test cho bug

```text
Add a regression test for this bug.

Bug summary:
[DESCRIBE BUG]

Current wrong behavior:
[ACTUAL]

Expected behavior:
[EXPECTED]

Repro:
[STEPS OR INPUT DATA]

Context:
@path/to/buggy/file
@path/to/related/test/file

Rules:
- The test must fail before the fix.
- The test must pass after the fix.
- Keep the scope small.
- Do not fix production code yet.
- Do not change unrelated tests.

Deliverables:
1. Test case name
2. File changed
3. Why this test catches the bug
4. Command to run only this test
```

---

### Prompt 3: Fix bug sau khi đã có test đỏ

```text
The regression test is now failing for the expected reason.

Error/output:
[PASTE TEST ERROR]

Now fix the production code.

Rules:
- Make the smallest safe change.
- Do not change the test unless the expectation is incorrect.
- Keep existing public API unchanged.
- Do not refactor unrelated code.

After editing, explain:
1. Root cause
2. What changed
3. Why the test now passes
4. Any edge case still not covered
```

---

### Prompt 4: Thêm API contract test

```text
Add contract tests for this API.

Endpoint:
[METHOD] [PATH]

Expected response shape:
[DESCRIBE REQUIRED FIELDS]

Context:
@path/to/router-or-controller
@path/to/service
@path/to/sample/api/test

Rules:
- Assert required fields and types.
- Avoid full response snapshot unless the repo already uses snapshots.
- Include at least one validation error case.
- Use the existing HTTP test helper.
- Keep test data minimal.

Deliverables:
1. Test cases added
2. Test data used
3. Command to run the API test
```

---

### Prompt 5: Thêm UI test

```text
Add UI tests for this user flow.

Flow:
1. [Step 1]
2. [Step 2]
3. [Step 3]

Expected result:
[WHAT USER SHOULD SEE]

Context:
@resources/views/... hoặc @app/Livewire/...
@tests/Feature/... hoặc @tests/Browser/... (Dusk)

Rules:
- Dùng đúng stack UI test của repo (Feature + HTTP, Livewire::test, hoặc Dusk).
- Ưu tiên assert text hiển thị, nhãn form, HTTP status—tránh selector CSS mong manh.
- Mock HTTP client / facade ở một ranh giới nếu flow gọi API ngoài.
- Giữ test deterministic (thời gian, queue, mail nên fake).

Deliverables:
1. Test cases added
2. Mock strategy
3. Command to run the UI test
```

---

### Prompt 6: Review test chất lượng

```text
Review the tests added in this diff.

Focus on:
1. Do these tests verify business behavior?
2. Are they too coupled to implementation details?
3. Are there flaky async/time/random parts?
4. Are mocks too broad or unrealistic?
5. Are important edge cases missing?
6. Is the test data minimal?

Do not edit files yet. Return issues and suggested fixes.
```

---

## Checklist trước khi merge

Trước khi merge code có test do Cursor viết, kiểm tra:

- [ ] Test nằm đúng folder convention của repo.
- [ ] Tên test đọc lên hiểu behavior cần khóa.
- [ ] Test fail đúng lý do trước khi fix, nếu là TDD/regression.
- [ ] Không gọi network thật trong unit test.
- [ ] Không dùng secret thật.
- [ ] Không snapshot cả response lớn nếu không cần.
- [ ] Không test private method/internal detail.
- [ ] Có command chạy test riêng.
- [ ] Có chạy suite liên quan trước khi merge.
- [ ] Production code không bị sửa quá scope.
- [ ] Mock không che mất bug thật.
- [ ] Test data đủ nhỏ, dễ đọc.

---

## Anti-pattern cần tránh

### 1. Bảo Cursor viết test toàn repo

Không nên:

```text
Write tests for the whole project.
```

Nên:

```text
Add unit tests for App\Services\CampaignBudgetService only.
```

---

### 2. Không đưa test mẫu

Không nên:

```text
Write tests for this file.
```

Nên:

```text
Write tests for @app/Services/CampaignService.php and follow style from @tests/Unit/UserServiceTest.php.
```

---

### 3. Để Cursor tự chọn framework

Không nên:

```text
Add tests.
```

Nên:

```text
Use PHPUnit (Laravel default). Follow existing *Test.php style under tests/. Run with: php artisan test --filter=CampaignServiceTest
```

---

### 4. Test quá nhiều behavior cùng lúc

Không nên:

```text
should handle campaign lifecycle
```

Nên tách:

```text
should create campaign with valid input
should reject campaign when daily budget is exceeded
should pause campaign when status is inactive
```

---

### 5. Sửa production code lớn chỉ để test

Nếu code khó test, yêu cầu Cursor đề xuất seam nhỏ:

```text
If this is hard to test, propose the smallest dependency injection seam. Do not rewrite the module.
```

---

## Template workflow ngắn dùng hằng ngày

Dùng template này khi muốn làm nhanh trong Cursor:

```text
I need to add tests for this change.

Context:
@production-file
@sample-test-file

Task:
1. Inspect current code and test style.
2. Propose test cases first. Do not edit yet.
3. After I confirm, add focused tests only.
4. Give me the exact command to run them.
5. After tests are added, review for flakiness and implementation coupling.

Rules:
- Follow existing framework and style.
- Mock external services.
- Keep fixtures minimal.
- Do not make unrelated production changes.
```

---

## Kết luận

Khi dùng Cursor để viết test, cách làm tốt nhất là:

```text
Read existing code → read existing test style → propose cases → write focused test → run targeted test → fix setup → implement/fix → run related suite → review diff
```

Không nên để Cursor tự đoán quá nhiều. Test càng gần behavior thật, càng có giá trị khi implement, fix bug và refactor.
