# Lesson 10 — Page Object Model: import, type Page vs value page, và baseURL

## 1. Vì sao có file chỉ `import { expect }`, không import `page`

```ts
import { expect } from '@playwright/test';
```

- `expect` là một **hàm** được `@playwright/test` export sẵn (static export) → dùng ở đâu cũng phải `import`.
- `page` (chữ thường) **không phải** thứ để import. Nó là một **fixture** — một đối tượng do Playwright Test runner tự tạo ra và "bơm" (inject) vào khi một test đang chạy, thường xuất hiện như tham số của `test()`:

```ts
import { test, expect } from '@playwright/test';

test('login test', async ({ page }) => {   // page được Playwright cấp phát ở đây
  const loginPage = new LoginPage(page);   // rồi truyền tay vào Page Object
  await loginPage.goto();
});
```

- Trong class `LoginPage`, `page` không tự sinh ra mà được **nhận qua constructor** từ file test:

```ts
export class LoginPage {
  constructor(page) {
    this.page = page;   // chỉ gán lại, không tạo mới
  }
}
```

→ Đây là mô hình **Page Object Model (POM)**: Page Object không phải là file test nên không cần `test`/fixture `page`, chỉ cần `expect` nếu có viết assertion bên trong (ví dụ hàm `expectError()`).

**Quy tắc chung:** chỉ import những gì thực sự dùng trong code.

## 2. Phân biệt `Page` (Type) và `page` (value) — lỗi dễ nhầm

| | `page` (chữ thường) | `Page` (viết hoa) |
|---|---|---|
| Là gì | **Giá trị runtime** — đối tượng thật đại diện cho tab trình duyệt | **Type/interface** của TypeScript, mô tả "một page có những method gì" |
| Nguồn gốc | Fixture do Playwright test runner cấp phát lúc chạy (`test('...', async ({ page }) => {...})`) | Export tĩnh từ `@playwright/test`, chỉ dùng để khai báo kiểu |
| Khi nào cần import | Không import — nó tự đến qua tham số của `test()` hoặc qua constructor Page Object | Chỉ cần import **nếu code có viết type annotation** kiểu `: Page` ở đâu đó |

Ví dụ có dùng type `Page` (bắt buộc phải import):

```ts
import { expect, Page } from '@playwright/test';

export class MyLoginPage {
  page: Page;                    // property có kiểu Page
  constructor(page: Page) {      // tham số có kiểu Page
    this.page = page;
  }
}
```

Ví dụ không dùng type annotation (không cần import `Page`):

```ts
import { expect } from '@playwright/test';

export class LoginPage {
  constructor(page) {   // không có ": Page" → TypeScript hiểu ngầm là any
    this.page = page;
  }
}
```

**Nếu bỏ `Page` khỏi import mà code vẫn còn viết `: Page` ở đâu đó** → lỗi compile:
```
Cannot find name 'Page'.
```

Muốn bỏ, phải xóa luôn phần khai báo kiểu (`page: Page` → `page`), chấp nhận mất type-checking/autocomplete — không khuyến khích nếu đang code TypeScript nghiêm túc.

## 3. Vì sao `page.goto('/login')` biết vào trang nào dù không ghi URL đầy đủ

Nhờ cấu hình **`baseURL`** trong `playwright.config.ts` (project `playwright-campaign` đã có sẵn file này ở gốc thư mục).

```ts
// playwright.config.ts
export default defineConfig({
  use: {
    baseURL: 'https://your-domain.com',
  },
});
```

Khi gọi:
```ts
await this.page.goto('/login');
```
Playwright tự ghép `baseURL` + `/login` → điều hướng tới `https://your-domain.com/login`.

- Nếu **không cấu hình `baseURL`**, gọi `goto('/login')` sẽ lỗi vì không biết domain nào để ghép:
  ```
  Error: page.goto: Cannot navigate to invalid URL
  ```
- Muốn biết chính xác `/login` trỏ đi đâu trong một project, luôn mở `playwright.config.ts` → tìm `baseURL` trong khối `use`.

## Lưu ý bổ sung

- **`getByLabel`, `getByRole`, `getByText`** (thấy trong `LoginPage`) là các *locator ưu tiên* của Playwright — nên dùng trước `page.locator(xpath)` vì bám theo accessibility tree, ít bị vỡ khi UI đổi class/id. `MyLoginPage` dùng toàn XPath (`//input[@id='...']`) — vẫn chạy được nhưng dễ gãy hơn khi DOM thay đổi.
- **`exact: false`** trong `getByText(text, { exact: false })` nghĩa là so khớp một phần (substring), hữu ích khi thông báo lỗi có thể kèm thêm text động.
- Khi một `getByRole` có thể khớp nhiều phần tử (ví dụ nút show/hide password cũng chứa chữ "Mật khẩu" trong aria-label), cần thêm `{ name: '...' }` để tránh **strict mode violation** (Playwright báo lỗi khi 1 locator khớp >1 phần tử mà bạn gọi action/assertion trực tiếp trên nó).
- Đặt tên: dùng `page` (thường) cho biến/tham số/property chứa giá trị; dùng `Page` (hoa) chỉ khi cần chú thích kiểu — tên trùng chữ nhưng khác hoa/thường là nguồn nhầm lẫn phổ biến nhất khi mới học TypeScript + Playwright.
