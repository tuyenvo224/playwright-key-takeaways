# Playwright Lesson 8 - Test Suite, Hooks & Assertion

## Mục lục
1. [Test Group / Suite (test.describe)](#1-test-group--suite)
2. [Hooks](#2-hooks)
3. [Assertion & Web-first Assertion](#3-assertion--web-first-assertion)

---

## 1. Test Group / Suite

### Test suite là gì?
- **Test suite** = tập hợp nhiều test cases lại với nhau
- Mục đích: **nhóm các test** lại cho dễ quản lý, dễ đọc, dễ bảo trì

### Cú pháp `test.describe()`

```typescript
import { test, expect } from '@playwright/test';

test.describe('<tên suite>', async () => {
  test('test1', async ({ page }) => {
    // code ...
  });

  test('test 2', async ({ page }) => {
    // code ...
  });
})
```

### Ví dụ thực tế

```typescript
import { test } from '@playwright/test';

test.describe("Material Site", async () => {
  test("User Registration Page", async ({ page }) => {
    console.log("Test User Registration Page");
  });

  test("Product Page", async ({ page }) => {
    console.log("Test Product Page");
  });
})
```

> Khi chạy, Playwright sẽ hiển thị tên suite > tên test, giúp dễ xác định test nào đang chạy.

---

## 2. Hooks

### Hooks là gì?
Hooks là các **hàm đặc biệt** chạy tự động tại các thời điểm nhất định trong vòng đời của test:
- **Trước khi chạy** test/suite
- **Trong khi chạy** test/suite
- **Sau khi chạy** test/suite

### 4 loại Hooks trong Playwright

| Hook | Thời điểm chạy | Chạy bao nhiêu lần |
|---|---|---|
| `test.beforeAll` | Trước khi toàn bộ suite bắt đầu | 1 lần duy nhất |
| `test.beforeEach` | Trước MỖI test | Mỗi test 1 lần |
| `test.afterEach` | Sau MỖI test | Mỗi test 1 lần |
| `test.afterAll` | Sau khi toàn bộ suite kết thúc | 1 lần duy nhất |

### Sơ đồ thứ tự thực thi

```
beforeAll (1 lần)
│
├── beforeEach → Test 1 → afterEach
│
└── beforeEach → Test 2 → afterEach
│
afterAll (1 lần)
```

### Ví dụ đầy đủ 4 hooks

```typescript
import { test } from '@playwright/test';

test.describe("Material site", async () => {

  // Chạy 1 lần duy nhất trước toàn bộ suite
  test.beforeAll(async ({ page }) => {
    await test.step("Go to Playwright Page", async () => {
      await page.goto("https://playwrightvn.com");
    });
  });

  // Chạy trước MỖI test
  test.beforeEach(async ({ page }) => {
    await test.step("Go to Material Page", async () => {
      await page.goto("https://material.playwrightvn.com/");
    });
  });

  // Chạy sau MỖI test
  test.afterEach(async ({ page }) => {
    await test.step("Go to Google.com", async () => {
      await page.goto("https://google.com");
    });
  });

  // Chạy 1 lần duy nhất sau toàn bộ suite
  test.afterAll(async () => {
    await test.step("Kết thúc test suite", async () => {
      console.log("Tất cả đã xong");
    });
  });

  test("User Registration Page", async ({ page }) => {
    await test.step("Click to User Registration page", async () => {
      await page.locator('//a[@href="01-xpath-register-page.html"]').click();
    });
  });

  test("Product Page", async ({ page }) => {
    await test.step("Click Product Page", async () => {
      await page.locator('//a[@href="02-xpath-product-page.html"]').click();
    });
  });
});
```

### Ứng dụng thực tế của Hooks

```
beforeAll  → Setup: Tạo tài khoản 1 lần duy nhất
beforeEach → Precondition: Đăng nhập trước mỗi test
  test 1: Test tạo danh mục thành công
  test 2: Test sửa danh mục thành công
  test 3: Test sửa handle của danh mục
afterEach  → Teardown nhỏ: Dọn dẹp sau mỗi test
afterAll   → Teardown: Xoá tài khoản 1 lần duy nhất
```

---

## 3. Assertion & Web-first Assertion

### Assertion là gì?
- **Assertion** (khẳng định/xác nhận) = câu lệnh để **kiểm tra** xem một điều gì đó có **đúng như mong đợi** hay không
- **Không có assertion** = không biết test có thành công hay thất bại

### Tại sao cần Assertion?

| Không có Assertion | Có Assertion |
|---|---|
| Chỉ thực hiện hành động | Kiểm tra kết quả có đúng không |
| `await page.click('button')` | `await expect(page.locator('button')).toBeVisible()` |
| "Tôi click nút" | "Tôi kiểm tra xem nút có hiển thị không" |

### Playwright assert thông qua hàm `expect`

```typescript
import { test, expect } from '@playwright/test';

test("Test 01", async ({ page }) => {
  // Khẳng định rằng: title trang phải là "Homepage"
  await expect(page).toHaveTitle('Homepage');

  // Khẳng định rằng: button phải visible (nhìn thấy được)
  await expect(page.locator('button')).toBeVisible();

  // Khẳng định rằng: giá trị phải bằng 5
  expect(2 + 3).toBe(5);
});
```

---

### Các loại Assertion

#### A. Generic Assertions (Assertion thông thường)
- Cú pháp: `expect(giá trị).<matcher>(giá trị mong đợi)`
- Dùng cho **giá trị thông thường** (số, chuỗi, mảng, boolean...)
- **Không có auto-waiting** (không tự động chờ)

```typescript
// Kiểm tra số
expect(1 + 2).toEqual(3);

// Kiểm tra độ dài mảng
const arr = [1, 2, 3];
expect(arr).toHaveLength(3);

// Kiểm tra chuỗi có chứa text
const str = "Hello Viet Nam";
expect(str).toContain("Nam");

// Kiểm tra title trang (lấy giá trị trước rồi assert)
const title1 = await page.title();
expect(title1).toContain("Playwright Việt Nam");
```

#### B. Web-first Assertions (Assertion ưu tiên web)
- Cú pháp: `await expect(phần tử).<matcher>()`
- Dùng cho **elements trên web**
- **Tự động chờ** (auto-waiting) đến khi điều kiện được thỏa mãn
- Timeout mặc định: **5 giây**

```typescript
// Ví dụ web-first assertion
await expect(page.locator('button')).toBeVisible();
await expect(page).toHaveTitle(/Homepage/);
```

---

### Web-first Assertions phổ biến

#### Element State (Trạng thái phần tử)

```typescript
// Kiểm tra visibility (hiển thị/ẩn)
await expect(locator).toBeVisible();
await expect(locator).toBeHidden();

// Kiểm tra enabled/disabled
await expect(locator).toBeEnabled();
await expect(locator).toBeDisabled();

// Kiểm tra checked (checkbox/radio)
await expect(locator).toBeChecked();

// Kiểm tra focus
await expect(locator).toBeFocused();
```

#### Text & Content (Nội dung văn bản)

```typescript
// Có chứa text
await expect(locator).toContainText('Hello');

// Text chính xác
await expect(locator).toHaveText('Welcome');

// Text khớp regex
await expect(locator).toHaveText(/welcome/i);

// Kiểm tra nhiều elements
await expect(locator).toHaveText(['Item 1', 'Item 2']);
```

#### Attributes & Properties (Thuộc tính)

```typescript
// Kiểm tra attribute
await expect(locator).toHaveAttribute('href', '/about');

// Kiểm tra class
await expect(locator).toHaveClass('active');
await expect(locator).toHaveClass(/btn-primary/);

// Kiểm tra value (input fields)
await expect(locator).toHaveValue('john@example.com');

// Kiểm tra count (số lượng phần tử)
await expect(locator).toHaveCount(5);
```

#### Page Assertions (Kiểm tra trang)

```typescript
// Kiểm tra URL
await expect(page).toHaveURL('https://example.com/');
await expect(page).toHaveURL(/.*checkout/);

// Kiểm tra Title
await expect(page).toHaveTitle('My App');
```

---

### Điểm khác biệt quan trọng: Auto-waiting

#### Vấn đề với Generic Assertion (không có auto-wait)
```typescript
// Button xuất hiện ngẫu nhiên sau 1-5 giây
// Nếu dùng generic assertion:
const isEnable = await page.locator("//button[@type='submit']").isEnabled();
expect(isEnable).toEqual(true); // Có thể FAIL nếu button chưa enable

// Workaround tệ: chờ cứng 10 giây
await page.waitForTimeout(10_000); // Luôn chờ đủ 10s dù button enable sớm hơn
const isEnable2 = await page.locator("//button[@id='submitButton']").isEnabled();
expect(isEnable2).toEqual(true);
```

#### Giải pháp với Web-first Assertion (có auto-wait)
```typescript
// Web-first assertion tự động chờ linh hoạt (tối đa timeout)
const submitButton = page.locator("//button[@id='submitButton']");
await expect(submitButton).toBeEnabled({ timeout: 10_000 });
// Nếu 1s button enable → thoát luôn
// Nếu 3s button enable → thoát luôn
// Tối đa chờ 10s, nếu vẫn chưa enable → FAIL
```

#### So sánh trực quan

```
Phần tử xuất hiện ngẫu nhiên 1-5 giây:

Generic Assertion (không nên dùng):
  Chờ cứng thời gian 5s ──────────────────────→ [luôn mất đủ 5s]

Web-first Assertion (nên dùng):
  Chờ flexible, tối đa 5s
    ↑ 1s → nếu đạt điều kiện, thoát luôn ✓
    ↑ 3s → nếu đạt điều kiện, thoát luôn ✓
    ↑ 5s → timeout → FAIL ✗
```

> **Tip:** Web-first assertion vừa **nhanh hơn** (thoát sớm khi điều kiện đạt) vừa **chính xác hơn** (không cần biết trước thời gian chờ). Luôn ưu tiên dùng web-first assertion cho elements trên web.

---

### Tùy chỉnh timeout cho Web-first Assertion

```typescript
// Timeout mặc định là 5000ms (5 giây)
await expect(locator).toBeEnabled();

// Tùy chỉnh timeout (ví dụ 10 giây)
await expect(locator).toBeEnabled({ timeout: 10_000 });

// Cú pháp ngắn hơn (không cần gán biến)
await expect(page.locator("//button[@id='submitButton']")).toBeEnabled({ timeout: 10_000 });
```

---

## Tóm tắt nhanh

| Khái niệm | Mô tả | Cú pháp |
|---|---|---|
| `test.describe` | Nhóm các test case lại thành 1 suite | `test.describe('tên', async () => {...})` |
| `test.beforeAll` | Chạy 1 lần trước toàn bộ suite | `test.beforeAll(async ({ page }) => {...})` |
| `test.beforeEach` | Chạy trước mỗi test | `test.beforeEach(async ({ page }) => {...})` |
| `test.afterEach` | Chạy sau mỗi test | `test.afterEach(async ({ page }) => {...})` |
| `test.afterAll` | Chạy 1 lần sau toàn bộ suite | `test.afterAll(async () => {...})` |
| Generic Assertion | Kiểm tra giá trị thông thường, không auto-wait | `expect(value).toBe(expected)` |
| Web-first Assertion | Kiểm tra element web, có auto-wait | `await expect(locator).toBeVisible()` |
