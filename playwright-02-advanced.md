# 📘 TỔNG HỢP KIẾN THỨC PLAYWRIGHT (Advanced)

---

## 📑 Mục lục

1. [npm, npx, package.json và node_modules](#1-npm-npx-packagejson-và-node_modules)
2. [Quản lý phiên bản với Semantic Versioning](#2-quản-lý-phiên-bản-với-semantic-versioning)
3. [Nâng cấp Playwright & cài trình duyệt](#3-nâng-cấp-playwright--cài-trình-duyệt)
4. [Cấu trúc test: test, describe, hooks](#4-cấu-trúc-test-test-describe-hooks)
5. [Quản lý test bằng Tag](#5-quản-lý-test-bằng-tag)
6. [Locator – cách tìm phần tử](#6-locator--cách-tìm-phần-tử)
7. [Action – thao tác với phần tử](#7-action--thao-tác-với-phần-tử)
8. [Assertion – kiểm tra kết quả](#8-assertion--kiểm-tra-kết-quả)
9. [Chạy test (VSCode & Command line)](#9-chạy-test-vscode--command-line)
10. [Debug test](#10-debug-test)
11. [API Testing với `request`](#11-api-testing-với-request)
12. [Spread operator & merge object](#12-spread-operator--merge-object)
13. [Page Object Model (POM) & Component](#13-page-object-model-pom--component)
14. [Data Driven Testing](#14-data-driven-testing)
15. [Quản lý môi trường (dev / staging / production)](#15-quản-lý-môi-trường-dev--staging--production)
16. [Cheat sheet – bảng lệnh tổng hợp](#16-cheat-sheet--bảng-lệnh-tổng-hợp)
17. [Các lỗi thường gặp](#17-các-lỗi-thường-gặp)

---

## 1. npm, npx, package.json và node_modules

### 1.1. Bức tranh tổng thể

```
                 ┌──────────────────────┐
   tải về  ───►  │ node_modules (~200MB)│   ← thư mục chứa code thư viện thật
                 └──────────────────────┘
 ┌──────────┐                                 ┌───────────────────────┐
 │   Code   │  ──── npm install ──────────►   │ Kho package tập trung │
 │          │  ◄──── tải thư viện về ──────   │  (npm registry)       │
 └──────────┘                                 └───────────────────────┘
 package.json  ← "danh sách đi chợ": ghi tên + version thư viện cần dùng
```

- **`package.json`**: giống như *danh sách đi chợ* – chỉ ghi tên thư viện và version, không chứa code thư viện.
- **`node_modules`**: giống như *giỏ hàng đã mua về* – chứa code thật của thư viện, rất nặng (có thể ~200MB) → **không commit lên git**, ai cần thì tự chạy `npm install` để tải về.
- **`package-lock.json`**: ghi lại **chính xác** version của toàn bộ cây thư viện (kể cả thư viện con) → **nên commit**.

### 1.2. `npm install` – cài thư viện

| Lệnh | Ý nghĩa |
|---|---|
| `npm install` | Cài **tất cả** thư viện đang được khai báo trong `package.json` |
| `npm install <ten_thu_vien>` | Cài thêm 1 thư viện mới (tự ghi vào `dependencies`) |
| `npm install -D <ten_thu_vien>` | Cài thư viện vào `devDependencies` (chỉ dùng khi phát triển/test) |
| `npm ci` | Cài **đúng y** theo `package-lock.json` (dùng trong CI) |

**Ví dụ:**

```bash
npm install -D dotenv     # cài thư viện dotenv để đọc file .env
npm install -D fs         # sau lệnh này package.json sẽ có thêm dòng "fs": "^0.0.1-security"
```

> 💡 Mỗi lần clone project về máy mới → việc đầu tiên là chạy `npm install`.

### 1.3. `npx` – chạy (execute) một công cụ

`npx` = **n**ode **p**ackage e**x**ecute → chữ **x = execute (thực thi)**.
Dùng để *chạy* một công cụ đã cài (hoặc chưa cài) mà không cần gõ đường dẫn dài.

```bash
npx ts-node <file.ts>          # chạy trực tiếp 1 file TypeScript
npx tsx <file.ts>              # tương tự ts-node, nhanh hơn
npx playwright test ...        # chạy test Playwright
npx playwright --version       # xem version Playwright đang dùng
```

**Ví dụ:** chạy file đọc JSON mà không cần Playwright test:

```bash
npx ts-node tests/lesson-01/02-readjson.ts
```

> 🧠 Dễ nhớ: **npm** = *mua đồ* (cài), **npx** = *dùng đồ* (chạy).

---

## 2. Quản lý phiên bản với Semantic Versioning

### 2.1. Cấu trúc `MAJOR.MINOR.PATCH`

Mọi package npm (kể cả Playwright) đều theo chuẩn **SemVer**:

```
1 . 50 . 2
│    │    └── PATCH: sửa bug, KHÔNG có breaking change
│    └─────── MINOR: thêm tính năng mới, KHÔNG có breaking change
└──────────── MAJOR: CÓ breaking change, cần review kỹ
```

| Thay đổi version | Ý nghĩa |
|---|---|
| `1.49.0 → 1.49.1` | Patch: sửa bug nhỏ – yên tâm nâng |
| `1.49.0 → 1.50.0` | Minor: thêm API mới, locator mới… – vẫn tương thích |
| `1.x.x → 2.0.0` | Major: có thể thay đổi API, làm **hỏng test cũ** |

> 🧠 Ví dụ đời thường: điện thoại cập nhật
> - Patch = vá lỗi pin hao,
> - Minor = thêm tính năng chụp ảnh mới,
> - Major = đổi hẳn giao diện → app cũ có thể không chạy.

### 2.2. Ý nghĩa `^`, `~` và exact version trong `package.json`

```json
{
  "devDependencies": {
    "@playwright/test": "^1.50.0",   // caret
    "@playwright/test": "~1.50.0",   // tilde
    "@playwright/test": "1.50.0"     // exact
  }
}
```

| Ký hiệu | Cho phép nâng cấp | Ví dụ `1.50.0` chấp nhận |
|---|---|---|
| `^` (caret) | Minor + Patch, giữ nguyên Major | `1.50.x`, `1.51.x`… nhưng **không** `2.x.x` |
| `~` (tilde) | Chỉ Patch, giữ nguyên Minor | `1.50.x` nhưng **không** `1.51.x` |
| Exact | Không nâng cấp tự động | Chỉ đúng `1.50.0` |

> ⚠️ **Mặc định** khi chạy `npm install <pkg>`, npm dùng `^` → cho phép tự nâng Patch và Minor.

### 2.3. Khi nào nên lock (khóa) version?

**Dùng exact version khi:**
- Project có CI/CD chạy trên nhiều môi trường → cần mọi máy dùng **đúng một** version.
- Team đông người → tránh cảnh *"chạy được trên máy mình"* do lệch version.
- Playwright đang ổn định, không cần update liên tục.

**Nên commit `package-lock.json`**: file này ghi lại version chính xác của toàn bộ dependency tree → cả team (và CI) cài đúng version khi chạy `npm ci`.

```bash
# Cài theo đúng package-lock.json (dùng trong CI)
npm ci

# Cài theo package.json (có thể nâng version nhỏ)
npm install
```

---

## 3. Nâng cấp Playwright & cài trình duyệt

### Cách 1: Sửa trực tiếp `package.json`

```json
"@playwright/test": "^1.63.0"   →   "@playwright/test": "^1.65.0"
```

Sau khi sửa **bắt buộc** chạy lại:

```bash
npm install              # cài lại thư viện theo version mới
npx playwright install   # cài các trình duyệt tương ứng với version Playwright mới
npx playwright --version # kiểm tra lại version
```

> ❓ Vì sao phải `npx playwright install`? Mỗi version Playwright đi kèm một bản Chromium/Firefox/WebKit riêng. Nâng Playwright mà không cài lại browser → test báo lỗi *"Executable doesn't exist"*.

### Cách 2: Dùng câu lệnh (nâng lên bản mới nhất)

```bash
npm install -D @playwright/test@latest
npx playwright install
```

### Cài một (hoặc nhiều) trình duyệt cụ thể

```bash
npx playwright install chromium                 # chỉ Chromium
npx playwright install chromium firefox webkit  # cả 3
```

---

## 4. Cấu trúc test: test, describe, hooks

### 4.1. `test` và `test.describe`

- `test(...)`: một test case.
- `test.describe(...)`: một **nhóm** test case (giống thư mục chứa nhiều file).

```ts
import { test, expect } from '@playwright/test';

test.describe('Login with password @login', () => {
  test('Test 1', async ({ page }) => { /* ... */ });
  test('Test 2', async ({ page }) => { /* ... */ });
});

test.describe('Register @login', () => {
  test('Test 1', async ({ page }) => { /* ... */ });
  test('Test 2', async ({ page }) => { /* ... */ });
});
```

### 4.2. Hooks – code chạy trước/sau test

| Hook | Chạy khi nào | Số lần |
|---|---|---|
| `test.beforeAll` | Trước **tất cả** test trong nhóm | 1 lần / worker |
| `test.beforeEach` | Trước **mỗi** test | N lần |
| `test.afterEach` | Sau **mỗi** test | N lần |
| `test.afterAll` | Sau **tất cả** test trong nhóm | 1 lần / worker |

```ts
test.describe('Login with password @login', () => {
  test.beforeAll(async () => { console.log('Before all is running'); });
  test.beforeEach(async ({ page }) => { console.log('Before Each is running'); });
  test.afterEach(async ({ page }) => { console.log('After Each is running'); });
  test.afterAll(async () => { console.log('afterAll is running'); });

  test('Test 1', async ({ page }) => { console.log('test 1 is running'); });
});
```

Output:

```
Before all is running
Before Each is running
test 1 is running
After Each is running
afterAll is running
```

> 🧠 Ví dụ đời thường – đi học:
> - `beforeAll` = mở cửa lớp (1 lần)
> - `beforeEach` = điểm danh trước mỗi tiết
> - `afterEach` = lau bảng sau mỗi tiết
> - `afterAll` = khóa cửa lớp (1 lần)

> ⚠️ **Lưu ý quan trọng:**
> - `page` và `context` **không dùng được** trong `beforeAll`/`afterAll`. Lỗi gặp phải:
>   `Error: "context" and "page" fixtures are not supported in "afterAll" since they are created on a per-test basis.`
>   → Nếu muốn dùng chung 1 page cho nhiều test, tự tạo bằng `browser.newContext()` / `browser.newPage()`.
> - Vì test chạy **song song nhiều worker**, mỗi worker sẽ chạy `beforeAll` riêng → bạn có thể thấy `Before all is running` in ra nhiều lần.

### 4.3. Tên test không được trùng nhau

```ts
test('Test 2', ...);
test('Test 2', ...);   // ❌ Error: duplicate test title "Login with password @login › Test 2"
```

→ Đặt tên khác nhau: `Test 2-1`, `Test 2-2`.

### 4.4. `test.step` – chia test thành từng bước

Giúp report dễ đọc hơn, biết test fail ở bước nào.

```ts
test('PUT – cập nhật todo', async ({ request }) => {
  let createdId = 0;

  await test.step('Step 1: tạo todo', async () => { /* ... */ });
  await test.step('Step 2: update todo', async () => { /* ... */ });
});
```

---

## 5. Quản lý test bằng Tag

### 5.1. Gắn tag

```ts
// Cách 1: viết tag trong tên test
test('đăng nhập thành công @smoke @regression', async ({ page }) => { ... });

// Cách 2: dùng annotation `tag` (khuyên dùng – tên test gọn hơn)
test('đăng nhập thành công', { tag: ['@smoke', '@regression'] }, async ({ page }) => { ... });

// Có thể gắn tag cho cả describe
test.describe('Login with password @login', () => { ... });
```

### 5.2. Chạy test theo tag

```bash
npx playwright test --grep @smoke                 # chỉ chạy test có @smoke
npx playwright test -g "@smoke"                   # -g là viết tắt của --grep
npx playwright test --grep "@smoke|@regression"   # @smoke HOẶC @regression (dấu | = OR)
npx playwright test --grep-invert @slow           # BỎ QUA test có @slow
```

**Ví dụ minh họa:**

```ts
test.describe('Login @login', () => {
  test('Test 1', { tag: ['@smoke'] }, async () => {});
  test('Test 2', { tag: ['@ui', '@smoke'] }, async () => {});
  test('Test 3', { tag: ['@ui'] }, async () => {});
});
```

| Lệnh | Test được chạy |
|---|---|
| `-g "@smoke"` | Test 1, Test 2 |
| `-g "@ui"` | Test 2, Test 3 |
| `-g "@smoke\|@ui"` | Test 1, 2, 3 |
| `--grep-invert "@smoke"` | Test 3 |
| `-g "@login"` | Cả 3 (vì tag nằm ở describe) |

**Convention phổ biến:**
- `@smoke` – test nhanh, kiểm tra luồng chính
- `@regression` – bộ test đầy đủ
- `@slow` – test nặng, chạy ít hơn

---

## 6. Locator – cách tìm phần tử

### 6.1. Các loại locator (thứ tự ưu tiên từ trên xuống)

```ts
// 1. Theo role (KHUYẾN NGHỊ – gần với cách user tương tác)
page.getByRole('button', { name: 'Đăng nhập' });
page.getByRole('textbox', { name: 'Email' });
page.getByRole('link', { name: 'Quên mật khẩu' });

// 2. Theo text hiển thị
page.getByText('Chào mừng trở lại');
page.getByText('Chào mừng', { exact: false }); // khớp một phần

// 3. Theo test ID (thuộc tính data-testid)
page.getByTestId('login-button');

// 4. Theo label của input
page.getByLabel('Email');
page.getByLabel('Mật khẩu');

// 5. Theo placeholder
page.getByPlaceholder('Nhập email của bạn');

// 6. CSS selector
page.locator('.btn-primary');
page.locator('#submit-btn');

// 7. XPath (dùng khi không còn cách nào khác)
page.locator('//button[@type="submit"]');
```

**So sánh nhanh:**

| Loại | Đặc điểm |
|---|---|
| Playwright selector (`getByRole`, `getByLabel`…) | Có **ngữ nghĩa**, dễ đọc, bền vững khi UI đổi style |
| CSS | **Performance cao nhất** |
| XPath | Mạnh nhưng dài, dễ gãy – dùng cuối cùng |

**Ví dụ HTML → locator:**

```html
<label for="email">Email</label>
<input id="email" placeholder="Nhập email của bạn" data-testid="email-input" />
<button type="submit" class="btn-primary">Đăng nhập</button>
```

```ts
page.getByLabel('Email');                       // tìm input qua label
page.getByPlaceholder('Nhập email của bạn');    // qua placeholder
page.getByTestId('email-input');                // qua data-testid
page.getByRole('button', { name: 'Đăng nhập' });// qua role + tên nút
page.locator('.btn-primary');                   // qua CSS class
```

### 6.2. Chaining (nối chuỗi) & Filtering (lọc)

```ts
// Tìm trong một phần tử cha
const form = page.locator('form#login');
await form.getByLabel('Email').fill('user@test.com');

// Lọc theo text
page.getByRole('listitem').filter({ hasText: 'Sản phẩm A' });

// Lọc theo locator con (item nào có chứa ảnh)
page.getByRole('listitem').filter({ has: page.getByRole('img') });

// Lấy phần tử thứ n (đánh số từ 0)
page.getByRole('row').nth(1);   // dòng thứ 2

// Phần tử đầu / cuối
page.getByRole('row').first();
page.getByRole('row').last();
```

> 🧠 Chaining giống như chỉ đường: *"Vào **tòa nhà A** → lên **tầng 3** → tìm **phòng 301**"* thay vì tìm "phòng 301" trên cả thành phố.

```ts
const body  = page.locator('body');
const child = body.locator('#list-items');   // chỉ tìm #list-items bên trong body
```

---

## 7. Action – thao tác với phần tử

```ts
// Click
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByRole('button').dblclick();                 // double click
await page.getByRole('button').click({ button: 'right' }); // click chuột phải

// Nhập text
await page.getByLabel('Email').fill('user@test.com'); // xóa rồi gõ (nhanh)
await page.getByLabel('Search').type('playwright');   // gõ từng ký tự (giả lập bàn phím)
await page.getByLabel('Code').clear();                // xóa nội dung

// Checkbox / Radio
await page.getByLabel('Đồng ý điều khoản').check();
await page.getByLabel('Đồng ý điều khoản').uncheck();
await page.getByLabel('Nam').check();                 // radio button

// Dropdown (select)
await page.getByLabel('Quốc gia').selectOption('VN');                  // theo value
await page.getByLabel('Quốc gia').selectOption({ label: 'Việt Nam' }); // theo chữ hiển thị

// Hover
await page.getByRole('button', { name: 'Menu' }).hover();

// Keyboard
await page.getByLabel('Search').press('Enter');
await page.keyboard.press('Tab');
await page.keyboard.press('Control+A');

// Upload file
await page.getByLabel('Upload').setInputFiles('path/to/file.pdf');
```

### Bảng tra nhanh theo loại control

| Control | Action |
|---|---|
| Button / Link | `click()` |
| Text input | `fill()` |
| Checkbox | `check()` / `uncheck()` |
| Radio | `check()` |
| Select / Dropdown | `selectOption()` |
| File input | `setInputFiles()` |
| Date picker | `fill()` với format ngày, hoặc tương tác bàn phím |

**Ví dụ hoàn chỉnh – điền form đăng ký:**

```ts
test('Đăng ký tài khoản', async ({ page }) => {
  await page.goto('https://material.playwrightvn.com/01-xpath-register-page.html');
  await page.getByLabel('Username').fill('tester01');
  await page.getByLabel('Email').fill('tester01@test.com');
  await page.getByLabel('Male').check();
  await page.getByLabel('Country').selectOption({ label: 'Vietnam' });
  await page.getByRole('button', { name: 'Register' }).click();
});
```

---

## 8. Assertion – kiểm tra kết quả

### 8.1. Assertion thường (so sánh giá trị, KHÔNG tự chờ)

```ts
expect(response.status()).toBe(200);        // bằng chính xác
expect(items).toHaveLength(3);              // độ dài mảng
expect(title).toContain('Dashboard');       // chứa chuỗi
expect(value).toBeTruthy();                 // giá trị "đúng" (khác null, 0, '', false...)
expect(obj).toEqual({ a: 1 });              // so sánh sâu (deep equality) object
expect(body.todos).toBeInstanceOf(Array);   // kiểu dữ liệu
```

### 8.2. Web-first assertion (TỰ ĐỘNG CHỜ – mặc định 5 giây)

Có `await` phía trước và truyền vào **locator** hoặc **page**:

```ts
// Trạng thái hiển thị
await expect(page.getByText('Đăng nhập thành công')).toBeVisible();
await expect(page.getByRole('dialog')).toBeHidden();

// URL
await expect(page).toHaveURL('/dashboard');
await expect(page).toHaveURL(/.*dashboard/);   // dùng regex

// Tiêu đề trang
await expect(page).toHaveTitle('Dashboard - App');

// Text
await expect(page.getByRole('heading')).toHaveText('Chào mừng');     // khớp toàn bộ
await expect(page.getByRole('heading')).toContainText('Chào');       // khớp một phần

// Attribute
await expect(page.getByRole('button')).toHaveAttribute('disabled');
await expect(page.getByRole('img')).toHaveAttribute('src', /avatar/);

// Giá trị input
await expect(page.getByLabel('Email')).toHaveValue('user@test.com');

// Class
await expect(page.getByRole('button')).toHaveClass(/btn-active/);

// Đảo ngược
await expect(page.getByText('Lỗi')).not.toBeVisible();

// Tăng timeout riêng cho 1 assertion
await expect(page.getByRole('heading')).toHaveText('Chào mừng', { timeout: 10_000 });
```

### 8.3. Auto-wait – lợi thế lớn nhất

```ts
// ❌ KHÔNG cần làm thế này
await page.waitForSelector('.success-message');
const text = await page.textContent('.success-message');
expect(text).toBe('Thành công');

// ✅ Chỉ cần
await expect(page.locator('.success-message')).toHaveText('Thành công');
// Playwright tự chờ element xuất hiện VÀ có text đúng (retry đến khi pass hoặc hết timeout)
```

> 🧠 Phân biệt nhanh:
> | | Assertion thường | Web-first assertion |
> |---|---|---|
> | Dùng cho | Biến, số, object, response API | Locator, page |
> | `await` | Không cần | **Bắt buộc** |
> | Tự chờ/retry | ❌ | ✅ (mặc định 5s) |

---

## 9. Chạy test (VSCode & Command line)

### 9.1. Qua VSCode (extension Playwright Test for VSCode)

Cài extension **Playwright Test for VSCode** (id: `ms-playwright.playwright`).

**Tính năng chính:**
- **Testing panel** (icon ống nghiệm ở sidebar): xem toàn bộ test theo cây thư mục.
- **Run / Debug** từng test, describe hoặc cả file bằng nút ▶ cạnh dòng code.
- **Pick locator**: click vào phần tử trên browser → tự sinh locator.
- **Record new test**: thao tác trên browser → tự sinh code test.

**Workflow thường dùng:**
1. Mở Testing panel (`Ctrl+Shift+P` → "Testing: Focus on Test Explorer View").
2. Click ▶ cạnh tên test để chạy đơn lẻ.
3. Click icon 🐞 để chạy debug, đặt breakpoint trực tiếp trong code.

### 9.2. Qua Command line

**Cú pháp chung:**

```bash
npx playwright test [tùy-chọn] [bộ-lọc-test...]
```
Không có tùy chọn, không có bộ lọc = chạy **toàn bộ** test.

**Chạy all / file / folder:**

```bash
npx playwright test                                           # tất cả test
npx playwright test tests/login.spec.ts                       # 1 file
npx playwright test tests/auth/                               # cả folder
npx playwright test tests/login.spec.ts tests/checkout.spec.ts # nhiều file
```

**Chạy theo tên (grep):**

```bash
npx playwright test --grep "login"            # tên chứa "login"
npx playwright test --grep @smoke             # theo tag
npx playwright test --grep "@smoke|@sanity"   # kết hợp OR
npx playwright test --grep-invert @slow       # loại trừ
npx playwright test tests/example.spec.ts -g "get started link"  # file + tên test
```

**Headed / Headless:**

```bash
npx playwright test            # Headless (mặc định) – KHÔNG mở cửa sổ browser
npx playwright test --headed   # Headed – mở browser, thấy test chạy trực tiếp
```
> Dùng `--headed` khi debug để quan sát; dùng headless trong CI để tiết kiệm tài nguyên.

**Liệt kê test (không chạy):**

```bash
npx playwright test --list
```
```
Listing tests:
  [chromium] › lesson-01/hooks.spec.ts:20:9 › Login with password @login › Test 1
  [chromium] › lesson-01/hooks.spec.ts:27:9 › Login with password @login › Test 2
  ...
Total: 13 tests in 3 files
```
> 💡 `--list` rất hữu ích để kiểm tra code có lỗi cú pháp không: nếu sai sẽ báo lỗi và `Total: 0 tests`.

### 9.3. Bảng tóm tắt lệnh chạy test

| Mục đích | Lệnh |
|---|---|
| Chạy tất cả test | `npx playwright test` |
| Chạy 1 file | `npx playwright test tests/login.spec.ts` |
| Chạy theo tên test | `npx playwright test -g "tên test"` |
| Chạy có giao diện browser | `npx playwright test --headed` |
| Chạy 1 project (browser) | `npx playwright test --project=chromium` |
| Debug | `npx playwright test --debug` |
| UI Mode | `npx playwright test --ui` |
| Chạy tuần tự | `npx playwright test --workers=1` |
| Dừng khi gặp lỗi | `npx playwright test -x` |
| Retry khi fail | `npx playwright test --retries=2` |
| Chạy lại test fail lần trước | `npx playwright test --last-failed` |
| Bật trace | `npx playwright test --trace=on` |
| Xem báo cáo | `npx playwright show-report` |
| Codegen (ghi lại thao tác) | `npx playwright codegen https://example.com` |
| Xem trace | `npx playwright show-trace trace.zip` |
| Cài trình duyệt | `npx playwright install` |
| Xóa cache | `npx playwright clear-cache` |

---

## 10. Debug test

| Cách | Mô tả |
|---|---|
| **Cách 1** | "Cắm" breakpoint (chấm đỏ) vào dòng code trong VSCode → bấm 🐞 Debug Test bên cạnh test |
| **Cách 2** | Đặt breakpoint → mở **JavaScript Debug Terminal** (Terminal → dấu `+` ▾ → JavaScript Debug Terminal) → chạy lệnh test như bình thường → test sẽ dừng ở breakpoint |
| **Cách 3** | `npx playwright test --debug` → mở Playwright Inspector, chạy từng bước |
| **Cách 4 (tạm thời)** | Thêm `await page.waitForTimeout(3_000);` để dừng 3 giây quan sát (chỉ dùng khi debug, **xóa** sau đó) |

**Ví dụ Cách 2:**

```ts
test('get started link', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await page.getByRole('link', { name: 'Get started' }).click(); // 🔴 đặt breakpoint ở đây
  await expect(page.getByRole('heading', { name: 'Installation' })).toBeVisible();
});
```

Trong JavaScript Debug Terminal:

```bash
npx playwright test tests/example.spec.ts -g "get started link" --headed
# → "Debugger attached." → test dừng tại dòng có breakpoint
```

---

## 11. API Testing với `request`

### 11.1. Cấu trúc một Response

```
┌──────── Response ────────┐
│  Status code (200, 201…) │  → response.status()
│  Header                  │  → response.headers()
│  Body                    │  → response.json() / response.text()
└──────────────────────────┘
```

- Dùng fixture `request` thay vì `page` → **không khởi động trình duyệt**, chạy rất nhanh.
- Header thường gặp: `content-type: application/json`, `server`, `date`…

### 11.2. GET – lấy toàn bộ danh sách

```ts
test('GET /todos.php – lấy toàn bộ danh sách todo', async ({ request }) => {
  const response = await request.get('https://material.playwrightvn.com/api/todo-app/v1/todos.php');

  expect(response.status()).toBe(200);
  const body = await response.json();     // body dạng object JS
  // const text = await response.text();  // body dạng chuỗi
  // console.log(response.headers());     // xem header

  expect(body.success).toBe(true);
  expect(body.todos).toBeInstanceOf(Array);
});
```

> 💡 `toBeInstanceOf(Object)` cũng pass với mảng, vì **Array là con của Object** trong JavaScript.

### 11.3. GET – lấy 1 item với query params

```ts
test('GET /todo.php – lấy một todo cụ thể', async ({ request }) => {
  const response = await request.get('https://material.playwrightvn.com/api/todo-app/v1/todo.php', {
    params: { id: 91 },   // → .../todo.php?id=91
  });

  const body = await response.json();
  console.log(body);
  // { success: true, todo: { id: 91, title: 'Don dep nha cua', status: 'pending', ... } }
});
```

### 11.4. POST – tạo mới

```ts
test('POST /todo.php – tạo todo', async ({ request }) => {
  const testData = {
    title: 'Học Playwright API Testing',
    status: 'pending',
    priority: 'high',
  };

  const response = await request.post('https://material.playwrightvn.com/api/todo-app/v1/todo.php', {
    data: testData,   // body gửi lên (tự chuyển thành JSON)
  });

  expect(response.status()).toBe(201);   // 201 = Created
  const body = await response.json();

  expect(body.success).toBe(true);
  expect(body.todo).toMatchObject({
    id: expect.any(Number),       // chỉ cần là số, không quan tâm giá trị
    title: testData.title,
    priority: testData.priority,
  });
});
```

**Giải thích:**
- `toMatchObject`: chỉ kiểm tra **các field liệt kê**, các field khác trong object bị bỏ qua (khác `toEqual` – phải khớp toàn bộ).
- `expect.any(Number)`: khớp với bất kỳ giá trị kiểu Number → dùng cho id tự sinh.
- Tách `testData` ra biến riêng → tái sử dụng khi gửi và khi kiểm tra, tránh gõ lặp.

### 11.5. PUT – cập nhật toàn bộ (kết hợp `test.step`)

```ts
test('PUT /todo.php – cập nhật toàn bộ todo', async ({ request }) => {
  const url = 'https://material.playwrightvn.com/api/todo-app/v1/todo.php';
  const testData = {
    bodyCreate: { title: 'Todo cũ', status: 'pending', priority: 'high' },
    bodyUpdate: { title: 'New title - from PUT API', status: 'completed' },
  };
  let createdId: number = 0;

  await test.step('Step 1: tạo todo', async () => {
    const response = await request.post(url, { data: testData.bodyCreate });
    expect(response.status()).toBe(201);
    const body = await response.json();
    createdId = body.todo.id;            // lưu id để dùng ở bước sau
    console.log('Created id: ', createdId);
  });

  await test.step('Step 2: update todo', async () => {
    const response = await request.put(url, {
      data: {
        ...testData.bodyUpdate,          // spread operator: trải các field của bodyUpdate ra
        id: createdId,
      },
    });
    expect(response.status()).toBe(200);
  });
});
```

### 11.6. Chuyển đổi JSON ↔ text

```ts
const body = await response.json();          // object

const rawString  = JSON.stringify(body);     // object → chuỗi text
const parsedJSON = JSON.parse(rawString);    // chuỗi text → object
```

| Hàm | Đầu vào | Đầu ra |
|---|---|---|
| `JSON.stringify(obj)` | Object | String |
| `JSON.parse(str)` | String | Object |

---

## 12. Spread operator & merge object

Spread operator `...` = **"trải"** toàn bộ thuộc tính của object ra.

```ts
const obj1 = { username: 'Phong', age: 18 };
const obj2 = { username: 'Tran',  address: 'HCMC' };

const object3 = {
  ...obj1,
  ...obj2,
};
// Kết quả:
// { username: 'Tran', age: 18, address: 'HCMC' }
```

> ⚠️ **Override (ghi đè)**: khi trùng key (`username`), object nào **đứng sau** sẽ thắng → `username = 'Tran'`.

**Ứng dụng thực tế:**

```ts
// 1. Thêm field vào từng phần tử mảng
const modified = pages.map(item => ({ ...item, currentDate: new Date() }));

// 2. Gộp body update với id
const payload = { ...bodyUpdate, id: createdId };

// 3. Data mặc định của môi trường + data riêng của test case (xem mục 15)
return { ...envJsonData, ...testDataJson };

// 4. Trong playwright.config.ts
use: { ...devices['Desktop Chrome'] }
```

---

## 13. Page Object Model (POM) & Component

### 13.1. POM là gì?

Thay vì viết locator + thao tác trực tiếp trong file test, ta gom chúng vào một **class đại diện cho 1 trang**.

```
POM (class của 1 trang)
 ├─ properties
 │    ├─ locator     (các phần tử trên trang)
 │    └─ component   (các khối UI dùng chung)
 └─ methods          (các hành động: login(), search()...)
```

**Lợi ích:** UI thay đổi → chỉ sửa 1 chỗ trong class, không phải sửa hàng chục file test.

### 13.2. Chia trang thành Component

Một trang thường có các khối lặp lại ở nhiều trang khác:

```
┌──────────────────────────────────────┐
│ [Home] [Product] [Media] [Contact]   │ → NavBarComponent   (#navbar)
└──────────────────────────────────────┘
              ┌──────────────────┐
              │ [Add][Edit][Del] │       → ActionBarComponent
              └──────────────────┘
┌──────────────────────────────────────┐
│            #list-items               │ → TableComponent
└──────────────────────────────────────┘
```

**Code minh họa:**

```ts
// pages/components/navbar.component.ts
import { Locator, Page } from '@playwright/test';

export class NavBarComponent {
  readonly root: Locator;

  constructor(private readonly page: Page) {
    this.root = page.locator('#navbar');             // phần tử gốc của component
  }

  async clickMenuHome()    { await this.root.getByRole('link', { name: 'Home' }).click(); }
  async clickMenuProduct() { await this.root.getByRole('link', { name: 'Product' }).click(); }
  async clickMenuMedia()   { await this.root.getByRole('link', { name: 'Media' }).click(); }
}
```

```ts
// pages/demo.page.ts
import { Page } from '@playwright/test';
import { NavBarComponent } from './components/navbar.component';
import { ActionBarComponent } from './components/action-bar.component';

export class LoginPage {
  navBar: NavBarComponent;          // property là component
  actionBar: ActionBarComponent;

  constructor(private readonly page: Page) {
    this.navBar = new NavBarComponent(page);
    this.actionBar = new ActionBarComponent(page);
  }

  async clickMenuProduct() {
    await this.navBar.clickMenuProduct();
  }
}
```

```ts
// tests/xxx.spec.ts
test('Mở trang Product', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.clickMenuProduct();
});
```

> 💡 Component dùng **chaining locator**: tìm phần tử gốc (`#navbar`) trước, rồi tìm phần tử con bên trong → tránh nhầm với phần tử giống tên ở chỗ khác trên trang.

### 13.3. Thiết kế theo hướng component (UI Kit)

Nhiều trang dùng chung một kiểu bảng (`products`, `categories` đều có cột `id | name | action`) → viết **1 `TableComponent`** dùng cho tất cả.

```ts
export class TableComponent {
  constructor(private readonly root: Locator) {}

  row(n: number)        { return this.root.getByRole('row').nth(n); }
  firstColumn(n: number){ return this.row(n).getByRole('cell').first(); }
}

// Dùng cho nhiều trang
const productTable  = new TableComponent(page.locator('#products'));
const categoryTable = new TableComponent(page.locator('#categories'));
```

> ⚠️ Tránh viết kiểu `if (page === 'product') return 'locator A'; else return 'locator B';` trong component – đây là dấu hiệu UI không đồng nhất. Tốt nhất đề xuất dev dùng chung **UI Kit** (cùng class, ví dụ `class='sortable'`) để component tái sử dụng được.

---

## 14. Data Driven Testing

**Ý tưởng:** 1 logic test + nhiều bộ dữ liệu = nhiều test case. Không phải copy-paste code.

### 14.1. Data driven với array JavaScript

```ts
import { test, expect } from '@playwright/test';

const loginData = [
  { username: 'admin', password: 'admin123', expectSuccess: true  },
  { username: 'user1', password: 'wrong',    expectSuccess: false },
  { username: '',      password: '',         expectSuccess: false },
];

for (const data of loginData) {
  test(`Login với username="${data.username}" → ${data.expectSuccess ? 'thành công' : 'thất bại'}`,
    async ({ page }) => {
      await page.goto('https://material.playwrightvn.com/01-login.html');
      await page.getByPlaceholder('Username').fill(data.username);
      await page.getByPlaceholder('Password').fill(data.password);
      await page.getByRole('button', { name: 'Login' }).click();

      if (data.expectSuccess) {
        await expect(page.getByText('Login successful')).toBeVisible();
      } else {
        await expect(page.getByText('Login failed')).toBeVisible();
      }
    });
}
```

→ Sinh ra **3 test**. Có thể dùng `forEach` thay cho `for...of`:

```ts
testData.forEach(item => {
  test(`Test ${item.id}: Test page ${item.name}`, async ({ page }) => {
    await page.goto(item.url);
    await expect(page.getByRole('heading')).toHaveText(item.expect);
  });
});
```

> ⚠️ Tên test phải **khác nhau** → luôn đưa dữ liệu vào tên (dùng template string `` `...${}...` ``).

### 14.2. Data driven với file JSON

Tách dữ liệu ra file riêng → file test chỉ chứa logic.

```json
// data/login-data.json
[
  { "username": "admin", "password": "admin123", "expectSuccess": true },
  { "username": "user1", "password": "wrong",    "expectSuccess": false }
]
```

**Cách 1 – import trực tiếp** (cần `"resolveJsonModule": true` trong `tsconfig.json`):

```ts
import loginData from '../data/login-data.json';

for (const data of loginData) {
  test(`Login: ${data.username}`, async ({ page }) => { /* ... */ });
}
```

**Cách 2 – đọc file bằng `fs.readFileSync` + `JSON.parse`** (linh hoạt, đọc được file theo tên động):

```ts
import * as fs from 'node:fs';
import * as path from 'node:path';

const raw   = fs.readFileSync(path.resolve('src/data/00-data.json'), 'utf-8'); // đọc ra chuỗi
const pages = JSON.parse(raw);                                                 // chuỗi → object
console.log(pages);
```

- `path.resolve(...)`: tạo đường dẫn tuyệt đối tính từ thư mục chạy lệnh (root project).
- `'utf-8'`: đọc file dưới dạng text (nếu bỏ sẽ nhận về Buffer).

### 14.3. Xử lý dữ liệu sau khi đọc: map, filter, find, reduce

```ts
// map – biến đổi từng phần tử (thêm field currentDate)
const modifiedData = pages.map(item => ({ ...item, currentDate: new Date() }));

// filter – lọc phần tử thỏa điều kiện
const onlyTodo = pages.filter(p => p.name.includes('Todo'));

// find – lấy phần tử ĐẦU TIÊN thỏa điều kiện
const productPage = pages.find(p => p.name === 'Product page');

// reduce – gộp mảng thành 1 giá trị
const totalNameLength = pages.reduce((sum, p) => sum + p.name.length, 0);
```

### 14.4. Đọc dữ liệu từ file CSV

```ts
import fs from 'fs';
import path from 'path';

export function readCSV(filePath: string) {
  const content = fs.readFileSync(path.resolve(filePath), 'utf-8');
  const [header, ...rows] = content.trim().split('\n'); // dòng đầu là header
  const keys = header.split(',');

  return rows.map(row => {
    const values = row.split(',');
    return Object.fromEntries(keys.map((k, i) => [k.trim(), values[i].trim()]));
  });
}

// users.csv:
// username,password
// admin,admin123
// user1,wrong
// → [{ username: 'admin', password: 'admin123' }, { username: 'user1', password: 'wrong' }]
```

---

## 15. Quản lý môi trường (dev / staging / production)

**Vấn đề:** Cùng một test nhưng chạy trên dev, staging, production cần **dữ liệu khác nhau** (user, password, URL…).

**Giải pháp:** dùng biến môi trường `ENV` để chọn bộ dữ liệu.

### 15.1. Truyền biến môi trường khi chạy lệnh

```bash
# macOS / Linux / Git Bash
ENV=staging npx playwright test
ENV=staging CLASS=pwa103 npx playwright test          # nhiều biến
ENV=production npx playwright test tests/lesson-01/06-env-management-utils.spec.ts           # không mở browser
ENV=production npx playwright test tests/lesson-01/06-env-management-utils.spec.ts --headed  # có mở browser
```

```powershell
# Windows PowerShell (cú pháp khác!)
$env:ENV="staging"; npx playwright test
```

Trong code đọc bằng: `process.env.ENV`.

### 15.2. Dùng file `.env` với thư viện dotenv

```bash
npm install -D dotenv
```

```ini
# .env
ENV=dev
```

```ts
// playwright.config.ts
import dotenv from 'dotenv';
import path from 'path';
dotenv.config({ path: path.resolve(__dirname, '.env') });

console.log(process.env.ENV); // dev
```

> 💡 Biến truyền trên dòng lệnh (`ENV=staging ...`) sẽ **ưu tiên hơn** giá trị trong `.env` (dotenv mặc định không ghi đè biến đã có).
> ⚠️ File `.env` thường chứa thông tin nhạy cảm → nên thêm vào `.gitignore`.

### 15.3. Tiến hóa cách tổ chức data theo môi trường

**Bước 1 – Để data trong object, chọn theo `process.env.ENV`:**

```ts
const testData = {
  user: {
    dev: 'dev-user@gmail.com',
    staging: 'staging-user@gmail.com',
    production: 'production-user@gmail.com',
  },
};
const username = testData.user[process.env.ENV];
```

**Bước 2 – Tách ra file JSON theo môi trường:**

```
src/data/
 ├─ 01-test-data-dev.json         { "user": "dev-user@gmail.com" }
 ├─ 01-test-data-staging.json     { "user": "staging-user@gmail.com" }
 └─ 01-test-data-production.json  { "user": "production-user@gmail.com" }
```

```ts
const env = process.env.ENV || 'dev';   // fallback: không truyền ENV thì dùng dev
const raw = fs.readFileSync(path.resolve(`src/data/01-test-data-${env}.json`), 'utf-8');
const testDataJson = JSON.parse(raw);
```

**Bước 3 – Gom vào hàm tiện ích (utils) để tái sử dụng:**

```ts
// src/utils/config.ts
export function getConfig(configFile: string) {
  const env = process.env.ENV || 'dev';
  const raw = fs.readFileSync(path.resolve(`src/data/${configFile}-${env}.json`), 'utf-8');
  return JSON.parse(raw);
}

// trong test
const testDataJson = getConfig('01-test-data');
await page.getByLabel('Username').fill(testDataJson.user);
```

**Bước 4 – Data theo từng test case:**

```json
// src/data/02-test-data-dev.json
{
  "case_1": { "user": "devcase1-user@gmail.com" },
  "case_2": { "user": "devcase2-user@gmail.com" },
  "case_3": {}
}
```

```ts
// src/utils/config-2.ts
export function getConfig(configFile: string, caseID: string) {
  const env = process.env.ENV || 'dev';
  const raw = fs.readFileSync(path.resolve(`src/data/${configFile}-${env}.json`), 'utf-8');
  return JSON.parse(raw)[caseID];
}
```

**Bước 5 – Merge data chung của môi trường + data riêng của case (dùng spread):**

```json
// src/data/dev.json  (data chung cho môi trường dev)
{ "user": "dev-user@gmail.com", "password": "abc@gmail.com" }
```

```ts
// src/utils/config-3.ts
export function getConfig(configFile: string, caseId: string) {
  const env = process.env.ENV || 'dev';

  const envJsonData  = JSON.parse(fs.readFileSync(path.resolve(`src/data/${env}.json`), 'utf-8'));
  const testDataJson = JSON.parse(fs.readFileSync(path.resolve(`src/data/${configFile}-${env}.json`), 'utf-8'))[caseId];

  return { ...envJsonData, ...testDataJson };  // data của case GHI ĐÈ data chung
}
```

| Gọi | Kết quả (ENV=dev) |
|---|---|
| `getConfig('02-test-data', 'case_1')` | `{ user: 'devcase1-user@gmail.com', password: 'abc@gmail.com' }` |
| `getConfig('02-test-data', 'case_3')` | `{ user: 'dev-user@gmail.com', password: 'abc@gmail.com' }` (case rỗng → dùng mặc định) |

### 15.4. Path alias – import ngắn gọn

Thay vì `import { getConfig } from '../../src/utils/config'`, cấu hình trong `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "resolveJsonModule": true,
    "paths": {
      "@data/*":     ["src/data/*"],
      "@pages/*":    ["src/pages/*"],
      "@fixtures/*": ["src/fixtures/*"],
      "@utils/*":    ["src/utils/*"]
    }
  }
}
```

```ts
import { getConfig } from '@utils/config';
import { LoginPage } from '@pages/login.page';
```

---

## 16. Cheat sheet – bảng lệnh tổng hợp

| Nhóm | Lệnh | Ý nghĩa |
|---|---|---|
| **npm** | `npm install` | Cài thư viện theo package.json |
| | `npm install -D <pkg>` | Cài thêm thư viện (devDependencies) |
| | `npm ci` | Cài đúng theo package-lock.json |
| | `npm install -D @playwright/test@latest` | Nâng Playwright lên mới nhất |
| **Playwright** | `npx playwright --version` | Xem version |
| | `npx playwright install` | Cài browser |
| | `npx playwright install chromium firefox webkit` | Cài browser cụ thể |
| **Chạy test** | `npx playwright test` | Chạy tất cả |
| | `npx playwright test <file>` | Chạy 1 file |
| | `npx playwright test -g "@smoke"` | Theo tên/tag |
| | `npx playwright test --grep-invert "@slow"` | Loại trừ |
| | `npx playwright test --list` | Liệt kê test |
| | `npx playwright test --headed` | Mở browser |
| | `npx playwright test --project=chromium` | Chỉ 1 browser |
| | `npx playwright test --ui` / `--debug` | UI mode / Debug |
| | `npx playwright test --workers=1` | Chạy tuần tự |
| | `npx playwright test --retries=2` | Retry |
| | `npx playwright test --last-failed` | Chạy lại test fail |
| **Môi trường** | `ENV=staging npx playwright test` | Truyền ENV (bash) |
| | `$env:ENV="staging"; npx playwright test` | Truyền ENV (PowerShell) |
| **Công cụ** | `npx playwright show-report` | Xem report |
| | `npx playwright codegen <url>` | Ghi lại thao tác thành code |
| | `npx playwright show-trace trace.zip` | Xem trace |
| | `npx ts-node <file>` / `npx tsx <file>` | Chạy file TS thường |

---

## 17. Các lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| `duplicate test title "..."` | 2 test cùng tên trong cùng describe | Đổi tên test cho khác nhau |
| `"context" and "page" fixtures are not supported in "afterAll"` | Dùng `{ page }` trong `beforeAll`/`afterAll` | Bỏ `page` hoặc tự tạo bằng `browser.newPage()` |
| `Executable doesn't exist ...` | Nâng Playwright nhưng chưa cài browser | `npx playwright install` |
| `Total: 0 tests` khi `--list` | Code có lỗi cú pháp (thiếu `)`, `}`…) | Đọc dòng báo lỗi phía trên và sửa |
| `expect(received).toEqual(expected)` | Giá trị thực tế khác mong đợi | Kiểm tra lại dữ liệu/logic |
| `Cannot find module '@utils/...'` | Chưa cấu hình `paths` trong tsconfig | Thêm `baseUrl` + `paths` |
| `ENV=staging` không chạy trên Windows | Cú pháp bash không dùng được ở PowerShell/CMD | Dùng `$env:ENV="staging"; ...` hoặc Git Bash |
| `process.env.ENV` là `undefined` | Không truyền ENV, chưa load `.env` | Dùng fallback `process.env.ENV \|\| 'dev'` và `dotenv.config()` |

---

> ✅ **Lộ trình ôn tập gợi ý:** npm/npx → SemVer → Cấu trúc test & hooks → Tag → Locator → Action → Assertion → Chạy/Debug → API testing → Spread → POM → Data driven → Quản lý môi trường.
