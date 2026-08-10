# Playwright Selectors - Tổng Hợp Kiến Thức Chi Tiết

> Nguồn: https://material.playwrightvn.com/03-playwright-selectors.html

---

## Mục lục

1. [Tổng quan về Locators](#1-tổng-quan-về-locators)
2. [Thứ tự ưu tiên Selector](#2-thứ-tự-ưu-tiên-selector)
3. [getByRole()](#3-getbyrole)
4. [getByText()](#4-getbytext)
5. [getByLabel()](#5-getbylabel)
6. [getByPlaceholder()](#6-getbyplaceholder)
7. [getByAltText()](#7-getbyalttext)
8. [getByTitle()](#8-getbytitle)
9. [getByTestId()](#9-getbytestid)
10. [Các method hỗ trợ: first(), last(), nth()](#10-các-method-hỗ-trợ-first-last-nth)
11. [Regex trong Selectors](#11-regex-trong-selectors)
12. [Tóm tắt nhanh](#12-tóm-tắt-nhanh)
13. [ARIA Role - Hiểu sâu hơn](#13-aria-role---hiểu-sâu-hơn)

---

## 1. Tổng quan về Locators

Locator trong Playwright là cách để **tìm và tương tác với các element trên trang web**. Thay vì dùng CSS selector hoặc XPath cứng nhắc, Playwright cung cấp các built-in locator methods giúp test **dễ đọc hơn**, **bền vững hơn**, và **thân thiện với accessibility**.

### Lợi ích của Playwright Locators:
- Tự động chờ element xuất hiện (auto-wait)
- Kiểm tra element có thể tương tác được không
- Chống lại các thay đổi nhỏ về DOM
- Thân thiện với ARIA/Accessibility

---

## 2. Thứ tự ưu tiên Selector

Playwright khuyến nghị sử dụng các selector theo thứ tự ưu tiên từ cao đến thấp:

| Ưu tiên | Method | Khi nào dùng |
|---------|--------|--------------|
| 1 | `getByRole()` | **Ưu tiên nhất** - Dựa trên ARIA role, thân thiện accessibility |
| 2 | `getByLabel()` | Form elements có label |
| 3 | `getByPlaceholder()` | Input có placeholder text |
| 4 | `getByText()` | Tìm theo text hiển thị |
| 5 | `getByAltText()` | Hình ảnh, icon có alt text |
| 6 | `getByTitle()` | Elements có title attribute (tooltip) |
| 7 | `getByTestId()` | **Fallback cuối cùng** - Dùng khi các cách trên không phù hợp |

> **Nguyên tắc vàng**: Luôn ưu tiên selector phản ánh cách người dùng thực sự tương tác với UI, không phải cách kỹ thuật implement.

---

## 3. getByRole()

### Khái niệm
Tìm element theo **ARIA role** - cách được khuyến khích nhất vì phản ánh đúng semantic HTML và accessibility.

### Cú pháp
```typescript
page.getByRole(role, options?)
// options: { name?, exact?, level?, checked?, disabled?, expanded?, pressed? }
```

### 3.1 Button

```html
<!-- HTML -->
<button>Submit Form</button>
<button>Save Changes</button>
<button>Delete Item</button>
<button disabled>Disabled Button</button>
```

```typescript
// Tìm và click button theo tên
await page.getByRole('button', { name: 'Submit Form' }).click();
await page.getByRole('button', { name: 'Save Changes' }).click();
await page.getByRole('button', { name: 'Delete Item' }).click();

// Lưu ý: Button bị disabled sẽ không click được
// await page.getByRole('button', { name: 'Disabled Button' }).click(); // ❌ Sẽ fail
```

### 3.2 Link

```html
<!-- HTML -->
<a href="/home">Home</a>
<a href="/about">About Us</a>
<a href="/services">Services</a>
<a href="/contact">Contact</a>
<a href="/home-page">Home Page</a>  <!-- Cũng chứa "Home" -->
```

```typescript
// exact: true - khớp CHÍNH XÁC tên (phân biệt "Home" với "Home Page")
await page.getByRole('link', { name: 'Home', exact: true }).click();

// Không có exact - khớp một phần (partial match)
await page.getByRole('link', { name: 'About Us' }).click();
await page.getByRole('link', { name: 'Services' }).click();
await page.getByRole('link', { name: 'Contact' }).click();
```

> **Lưu ý `exact: true`**: Nếu trang có cả link "Home" và "Home Page", dùng `{ name: 'Home', exact: true }` để chỉ khớp đúng "Home", tránh match nhầm.

### 3.3 Heading

```html
<!-- HTML -->
<h1>Main Title H1</h1>
<h2>Section Title H2</h2>
<h3>Subsection Title H3</h3>
```

```typescript
// level: xác định cấp độ heading (1-6)
const text1 = await page.getByRole('heading', { name: 'Main Title H1', level: 1 }).textContent();
console.log(text1); // "Main Title H1"

const text2 = await page.getByRole('heading', { name: 'Section Title H2' }).textContent();
const text3 = await page.getByRole('heading', { name: 'Subsection Title H3' }).textContent();
```

### 3.4 Checkbox & Radio

```html
<!-- HTML -->
<input type="checkbox" id="terms"> <label for="terms">I agree to Terms of Service</label>
<input type="radio" id="cc" name="payment"> <label for="cc">Credit Card</label>
<input type="radio" id="pp" name="payment"> <label for="pp">PayPal</label>
```

```typescript
// Checkbox
await page.getByRole('checkbox', { name: 'I agree to Terms of Service' }).click();

// Radio button
await page.getByRole('radio', { name: 'Credit Card' }).click();
```

### 3.5 Textbox

```html
<!-- HTML -->
<label for="search">Search products</label>
<input type="text" id="search" placeholder="Search products">
```

```typescript
// name match với label text HOẶC placeholder text
await page.getByRole('textbox', { name: 'Search products' }).fill("hello");

// exact: true - khớp chính xác (cả label lẫn placeholder đều được kiểm tra)
await page.getByRole('textbox', { name: 'Search products', exact: true }).fill("HocTest.Com");
```

### 3.6 Combobox (Select/Dropdown)

```html
<!-- HTML -->
<label for="country">Country selection</label>
<select id="country" aria-label="Country selection">
    <option value="vietnam">Vietnam</option>
    <option value="usa">USA</option>
</select>
```

```typescript
// Chọn option trong dropdown
await page.getByRole('combobox', { name: 'Country selection' }).selectOption("Vietnam");
```

### 3.7 Table

```html
<!-- HTML -->
<table aria-label="User data table">
    <thead>
        <tr><th>Name</th><th>Email</th><th>Age</th></tr>
    </thead>
    <tbody>
        <tr><td>John Doe</td><td>john@example.com</td><td>30</td></tr>
        <tr><td>Jane Smith</td><td>jane@example.com</td><td>25</td></tr>
    </tbody>
</table>
```

```typescript
// Lấy toàn bộ nội dung table
const table = await page.getByRole('table', { name: 'User data table' }).textContent();
console.log(table);

// Lấy một row cụ thể (dùng regex để tìm row chứa "John Doe")
const row = await page.getByRole('row', { name: /John Doe/ }).textContent();
console.log(row); // "John Doe  john@example.com  30"

// Lấy một cell cụ thể
const text = await page.getByRole('cell', { name: /john@/ }).textContent();
console.log(text); // "john@example.com"
```

### Danh sách ARIA Roles phổ biến

| Role | HTML Element tương ứng |
|------|----------------------|
| `button` | `<button>`, `<input type="button/submit/reset">` |
| `link` | `<a href="...">` |
| `heading` | `<h1>` đến `<h6>` |
| `checkbox` | `<input type="checkbox">` |
| `radio` | `<input type="radio">` |
| `textbox` | `<input type="text">`, `<textarea>` |
| `combobox` | `<select>`, custom dropdown |
| `table` | `<table>` |
| `row` | `<tr>` |
| `cell` | `<td>`, `<th>` |
| `dialog` | `<dialog>`, modal popup |
| `img` | `<img>` |
| `list` | `<ul>`, `<ol>` |
| `listitem` | `<li>` |

---

## 4. getByText()

### Khái niệm
Tìm element dựa trên **nội dung text hiển thị** trên UI.

### Cú pháp
```typescript
page.getByText(text, options?)
// options: { exact?: boolean }
// text có thể là: string | RegExp
```

### 4.1 Ví dụ cơ bản

```html
<!-- HTML -->
<div>HocTest<span>.Com</span></div>
<div>HocTest</div>
<div>PlaywrightVN <span>.Com</span></div>
<div>PlaywrightVN</div>
<input type="button" value="Log in">
<button>Log me in</button>
```

```typescript
// Tìm element chứa text "HocTest.Com" (partial match với nội dung đầy đủ của div đầu)
await page.getByText("HocTest.Com").click();

// exact: true - chỉ khớp CHÍNH XÁC text (div thứ 2 có đúng "HocTest")
await page.getByText("HocTest", { exact: true }).click();

// ❌ Sẽ FAIL - không có element nào có text chính xác là "HocTestCom"
// await page.getByText("HocTestCom").click();

// Partial match - tìm div chứa "PlaywrightVN .Com"
await page.getByText("PlaywrightVN .Com").click();

// exact: true với "PlaywrightVN"
await page.getByText("PlaywrightVN", { exact: true }).click();

// Input type="button" - tìm theo value attribute
await page.getByText("Log in").click();       // matches <input type=button value="Log in">
await page.getByText("Log me in").click();    // matches <button>Log me in</button>
```

### 4.2 Cách matching hoạt động

```typescript
// HTML: <div>HocTest<span>.Com</span></div>
//       <div>HocTest</div>

page.getByText('.Com');              // Khớp <span>.Com</span>
page.getByText('HocTest.Com');       // Khớp <div> đầu tiên (full text = "HocTest.Com")
page.getByText('HocTest', { exact: true }); // Khớp <div> thứ hai (chính xác "HocTest")
page.getByText(/HocTest/);           // Khớp CẢ HAI <div> (regex partial match)
page.getByText(/^hoctest$/i);        // Khớp <div> thứ hai (regex case-insensitive, exact)
```

### 4.3 Các lưu ý quan trọng

> **Normalize whitespace**: `getByText()` tự động normalize khoảng trắng trước khi tìm. Nếu text có nhiều hơn 2 khoảng trắng liên tiếp, chúng sẽ bị giảm xuống còn 1 khoảng trắng.

```typescript
// HTML: <div>Hello   World</div>  (3 khoảng trắng)
page.getByText("Hello World"); // ✅ Vẫn match vì whitespace được normalize
```

> **Input type button/submit**: Tìm theo `value` attribute, không phải text content.

```html
<input type="button" value="Log in">  <!-- Tìm bằng getByText("Log in") -->
<input type="submit" value="Submit">  <!-- Tìm bằng getByText("Submit") -->
```

---

## 5. getByLabel()

### Khái niệm
Tìm form control (input, textarea, select) dựa trên **text của `<label>` liên kết** hoặc `aria-label` attribute.

### Cú pháp
```typescript
page.getByLabel(text, options?)
// options: { exact?: boolean }
```

### Cách hoạt động
`getByLabel()` tìm input được liên kết với label theo các cách:
1. `<label for="id">` + `<input id="id">` - liên kết qua `for` attribute
2. `<label><input></label>` - input nằm trong label
3. `<input aria-label="...">` - aria-label attribute
4. `<input aria-labelledby="labelId">` - aria-labelledby attribute

### Ví dụ

```html
<!-- HTML -->
<label for="fullname">Full Name</label>
<input type="text" id="fullname">

<label for="pwd1">Password</label>
<input type="password" id="pwd1">

<label for="pwd2">Password (Confirm)</label>
<input type="password" id="pwd2">

<!-- aria-label -->
<input aria-label="Username">
```

```typescript
// Tìm input theo label text (case-insensitive by default)
await page.getByLabel('full Name').fill("Heyyy");       // ✅ "Full Name" match "full Name"

// Khi có nhiều elements khớp cùng label text
await page.getByLabel('password').first().fill("strongpassword");  // input Password đầu tiên
await page.getByLabel('password').last().fill("strongpassword");   // input Password cuối cùng

// Dùng nth() để chọn theo index (0-based)
await page.getByLabel('Password').nth(0).fill("strongpassword");  // index 0 = phần tử đầu tiên
await page.getByLabel('Password').nth(1).fill("strongpassword");  // index 1 = phần tử thứ hai

// aria-label
await page.getByLabel('Username').fill('john');
```

> **Lưu ý**: `getByLabel()` mặc định **case-insensitive** (không phân biệt hoa/thường). `'full Name'` sẽ match `<label>Full Name</label>`.

---

## 6. getByPlaceholder()

### Khái niệm
Tìm input/textarea dựa trên **placeholder text**.

### Cú pháp
```typescript
page.getByPlaceholder(text, options?)
// options: { exact?: boolean }
```

### Ví dụ

```html
<!-- HTML -->
<input type="text" placeholder="Search products...">
<input type="email" placeholder="Enter your email">
<textarea placeholder="Write your message here..."></textarea>
```

```typescript
// Tìm input theo placeholder text
await page.getByPlaceholder('Search products...').fill('Laptop');
await page.getByPlaceholder('Enter your email').fill('user@example.com');
await page.getByPlaceholder('Write your message here...').fill('Hello!');

// Partial match (mặc định)
await page.getByPlaceholder('Search').fill('Laptop');  // match "Search products..."

// exact: true - khớp chính xác
await page.getByPlaceholder('Search products...', { exact: true }).fill('Laptop');
```

> **Khi nào dùng**: Dùng khi input không có label nhưng có placeholder text rõ ràng. Thường gặp trong search boxes.

---

## 7. getByAltText()

### Khái niệm
Tìm element (thường là `<img>`) dựa trên **alt text** attribute.

### Cú pháp
```typescript
page.getByAltText(text, options?)
// options: { exact?: boolean }
// text có thể là: string | RegExp
```

### Ví dụ

```html
<!-- HTML -->
<img src="laptop.jpg" alt="Product 1 - Laptop">
<img src="phone.jpg" alt="Product 2 - Smartphone">
<img src="tablet.jpg" alt="Product 3 - Tablet">
```

```typescript
// Tìm image theo alt text chính xác
await page.getByAltText('Product 1 - Laptop').click();

// Tìm theo regex (partial match, case-insensitive)
await page.getByAltText(/Smartphone/).click();  // match "Product 2 - Smartphone"
await page.getByAltText(/product/i).click();    // match tất cả (case-insensitive)

// Lấy src attribute của image
const src = await page.getByAltText('Product 1 - Laptop').getAttribute('src');
console.log(src); // "laptop.jpg"
```

> **Best practice**: Luôn đặt alt text có ý nghĩa cho hình ảnh - vừa tốt cho accessibility, vừa dễ test.

---

## 8. getByTitle()

### Khái niệm
Tìm element dựa trên **title attribute** (thường hiển thị dạng tooltip khi hover).

### Cú pháp
```typescript
page.getByTitle(text, options?)
// options: { exact?: boolean }
// text có thể là: string | RegExp
```

### Ví dụ

```html
<!-- HTML -->
<button title="Click to save your changes">Save</button>
<span title="This provides additional information">ℹ️</span>
<a href="/help" title="Get help and support">Help</a>
```

```typescript
// Tìm theo title attribute chính xác
await page.getByTitle('Click to save your changes').click();

// Tìm theo regex (partial match)
await page.getByTitle(/additional information/).click();
await page.getByTitle(/help/i).click();   // case-insensitive

// Lấy thông tin element
const titleText = await page.getByTitle('Click to save your changes').getAttribute('title');
```

> **Lưu ý**: `title` attribute thường dùng cho tooltip, icon thông tin, và các element cần giải thích thêm.

---

## 9. getByTestId()

### Khái niệm
Tìm element dựa trên **data-testid** attribute - được thiết kế đặc biệt cho testing, không ảnh hưởng đến UI hay accessibility.

### Cú pháp
```typescript
page.getByTestId(testId)
// testId: string | RegExp
```

### Ví dụ

```html
<!-- HTML -->
<div data-testid="product-card">
    <h3>Laptop Pro</h3>
    <p>$999</p>
</div>
<button data-testid="login-submit-btn">Login</button>
<input data-testid="search-input" type="text">
```

```typescript
// Tìm theo data-testid
await page.getByTestId('product-card').click();
await page.getByTestId('login-submit-btn').click();
await page.getByTestId('search-input').fill('Laptop');

// Regex
await page.getByTestId(/product/).click();
```

### Cấu hình testId attribute trong Playwright config

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    testIdAttribute: 'data-testid',  // mặc định
    // Hoặc dùng attribute tùy chỉnh:
    // testIdAttribute: 'data-cy',    // cho Cypress-style
    // testIdAttribute: 'data-qa',    // cho QA-style
  },
});
```

> **Khi nào dùng `getByTestId()`**: Dùng khi không thể dùng các selector khác (element không có role, label, text rõ ràng). Cần thêm `data-testid` vào HTML bởi developer.

---

## 10. Các method hỗ trợ: first(), last(), nth()

Khi một selector match nhiều elements, dùng các method này để chọn element cụ thể:

```typescript
// Khi có nhiều elements match
const buttons = page.getByRole('button');

await buttons.first().click();      // Phần tử đầu tiên (index 0)
await buttons.last().click();       // Phần tử cuối cùng
await buttons.nth(0).click();       // Phần tử index 0 (= first())
await buttons.nth(2).click();       // Phần tử index 2 (phần tử thứ 3)

// Ví dụ thực tế với getByLabel
await page.getByLabel('password').first().fill("pass1");   // Password field 1
await page.getByLabel('password').last().fill("pass2");    // Password field 2
await page.getByLabel('Password').nth(0).fill("pass1");    // index 0
await page.getByLabel('Password').nth(1).fill("pass2");    // index 1
```

> **Cảnh báo**: Việc dùng `nth()`, `first()`, `last()` dựa trên vị trí DOM có thể khiến test bị flaky nếu thứ tự elements thay đổi. Hãy cố gắng dùng selector cụ thể hơn nếu có thể.

---

## 11. Regex trong Selectors

Playwright cho phép dùng **Regular Expression** với tất cả các `getBy*` methods:

```typescript
// Regex cơ bản
page.getByText(/Hello/);           // Chứa "Hello"
page.getByText(/^Hello$/);        // Chính xác "Hello"
page.getByText(/^Hello$/i);       // Chính xác "Hello" (case-insensitive)
page.getByText(/hello/i);         // Chứa "hello" (case-insensitive)

// Ví dụ với các methods khác
page.getByRole('row', { name: /John Doe/ });        // Row chứa "John Doe"
page.getByRole('cell', { name: /john@/ });          // Cell chứa "john@"
page.getByAltText(/Smartphone/);                    // Alt text chứa "Smartphone"
page.getByTitle(/additional information/);          // Title chứa "additional information"

// Regex flags
/pattern/i   // case-insensitive
/pattern/g   // global match
/^pattern$/  // exact match (start ^ và end $ anchors)
```

### So sánh String vs Regex

```typescript
// String (mặc định là partial match khi không có exact:true)
page.getByText("Hello");              // Tìm elements chứa "Hello"
page.getByText("Hello", {exact:true}); // Tìm elements CHÍNH XÁC là "Hello"

// Regex
page.getByText(/Hello/);              // Tương tự string partial match
page.getByText(/^Hello$/);            // Tương tự string exact match
page.getByText(/^hello$/i);           // Exact match, case-insensitive
```

---

## 12. Tóm tắt nhanh

### Bảng cheat sheet

```typescript
// Role-based (recommended)
page.getByRole('button', { name: 'Submit' })
page.getByRole('link', { name: 'Home', exact: true })
page.getByRole('heading', { name: 'Title', level: 1 })
page.getByRole('checkbox', { name: 'Accept' })
page.getByRole('radio', { name: 'Option A' })
page.getByRole('textbox', { name: 'Search' })
page.getByRole('combobox', { name: 'Country' })
page.getByRole('table', { name: 'Users' })
page.getByRole('row', { name: /John/ })
page.getByRole('cell', { name: /email@/ })

// Text-based
page.getByText('Hello World')
page.getByText('Hello', { exact: true })
page.getByText(/hello/i)

// Form
page.getByLabel('Full Name')
page.getByPlaceholder('Enter email...')

// Media
page.getByAltText('Product Image')
page.getByAltText(/product/i)

// Tooltip
page.getByTitle('Save changes')
page.getByTitle(/save/i)

// Test ID
page.getByTestId('submit-btn')

// Multiple match helpers
.first()    // Element đầu tiên
.last()     // Element cuối cùng
.nth(n)     // Element theo index (0-based)
```

### Decision tree chọn Selector

```
Câu hỏi: Element có ARIA role không?
├── Có → getByRole() ✅ (ưu tiên nhất)
│
├── Không → Element là form control có label không?
│   ├── Có → getByLabel()
│   │
│   ├── Không → Input có placeholder không?
│   │   ├── Có → getByPlaceholder()
│   │   │
│   │   ├── Không → Element có text hiển thị đặc trưng không?
│   │   │   ├── Có → getByText()
│   │   │   │
│   │   │   ├── Không → Element là image/có alt text không?
│   │   │   │   ├── Có → getByAltText()
│   │   │   │   │
│   │   │   │   ├── Không → Element có title attribute không?
│   │   │   │   │   ├── Có → getByTitle()
│   │   │   │   │   │
│   │   │   │   │   └── Không → getByTestId() (yêu cầu dev thêm data-testid)
```

---

## Phần thực hành

### Full example từ demo code

```typescript
import { test, expect } from '@playwright/test';

test.describe("Playwright selector demo tests", async () => {
    test.beforeEach(async ({ page }) => {
        await page.goto('https://material.playwrightvn.com/03-playwright-selectors.html');
    });

    test('getByRole - Button, Link, Heading', async ({ page }) => {
        // Buttons
        await page.getByRole('button', { name: 'Submit Form' }).click();
        await page.getByRole('button', { name: 'Save Changes' }).click();

        // Links (exact để tránh match "Home Page")
        await page.getByRole('link', { name: 'Home', exact: true }).click();
        await page.getByRole('link', { name: 'About Us' }).click();

        // Headings
        const h1 = await page.getByRole('heading', { name: 'Main Title H1', level: 1 }).textContent();
        console.log(h1); // "Main Title H1"
    });

    test('getByRole - Form elements', async ({ page }) => {
        // Checkbox & Radio
        await page.getByRole('checkbox', { name: 'I agree to Terms of Service' }).click();
        await page.getByRole('radio', { name: 'Credit Card' }).click();

        // Textbox
        await page.getByRole('textbox', { name: 'Search products', exact: true }).fill("HocTest.Com");

        // Combobox
        await page.getByRole('combobox', { name: 'Country selection' }).selectOption("Vietnam");
    });

    test('getByRole - Table', async ({ page }) => {
        const table = await page.getByRole('table', { name: 'User data table' }).textContent();
        const row = await page.getByRole('row', { name: /John Doe/ }).textContent();
        const cell = await page.getByRole('cell', { name: /john@/ }).textContent();
        console.log({ table, row, cell });
    });

    test('getByText', async ({ page }) => {
        await page.getByText("HocTest.Com").click();
        await page.getByText("HocTest", { exact: true }).click();
        await page.getByText(/PlaywrightVN/i).click();
        await page.getByText("Log in").click();        // matches input[type=button][value="Log in"]
    });

    test('getByLabel với first/last/nth', async ({ page }) => {
        await page.getByLabel('full Name').fill("John Doe");
        await page.getByLabel('password').first().fill("pass1");
        await page.getByLabel('password').last().fill("pass2");
        await page.getByLabel('Password').nth(0).fill("pass1");
        await page.getByLabel('Password').nth(1).fill("pass2");
    });

    test('getByAltText', async ({ page }) => {
        await page.getByAltText('Product 1 - Laptop').click();
        await page.getByAltText(/Smartphone/).click();
    });

    test('getByTitle', async ({ page }) => {
        await page.getByTitle('Click to save your changes').click();
        await page.getByTitle(/additional information/).click();
    });
});
```

---

*Tài liệu được tổng hợp từ: https://material.playwrightvn.com/03-playwright-selectors.html*

---

## 13. ARIA Role - Hiểu sâu hơn

### ARIA là gì?

**ARIA** = **Accessible Rich Internet Applications** — bộ tiêu chuẩn của W3C giúp trình đọc màn hình (screen reader) hiểu được **ý nghĩa và chức năng** của các element trên trang web.

**Role** = vai trò / loại của element đó là gì trong giao diện.

### Vấn đề ARIA giải quyết

HTML thuần có nhiều tag "không có ngữ nghĩa" — trình đọc màn hình không thể hiểu element đó dùng để làm gì:

```html
<!-- ❌ Screen reader chỉ đọc là "div" — không biết đây là button -->
<div class="btn" onclick="submit()">Submit</div>

<!-- ✅ Screen reader biết đây là "button, Submit" -->
<button>Submit</button>

<!-- ✅ Thêm role="button" thủ công để screen reader hiểu đúng -->
<div role="button" onclick="submit()">Submit</div>
```

### Tại sao Playwright dùng ARIA Role?

Playwright dùng ARIA role vì nó phản ánh **cách người dùng thực sự nhìn và tương tác** với trang — nhất quán với screen reader và semantic HTML:

```
Người dùng thấy:   [Submit Form]      → đây là một "button"
Screen reader đọc: "Submit Form, button"
Playwright tìm:    getByRole('button', { name: 'Submit Form' })
```

Cả 3 đều nhất quán → đó là lý do `getByRole()` được ưu tiên số 1.

---

### Implicit Role vs Explicit Role

#### Implicit Role (Role tự nhiên — có sẵn trong HTML)

Các element HTML đã có role mặc định mà không cần khai báo thêm:

| HTML Element | ARIA Role tự nhiên | Ghi chú |
|---|---|---|
| `<button>` | `button` | |
| `<a href="...">` | `link` | Phải có `href` mới có role `link` |
| `<a>` (không có href) | `generic` | Không có role link |
| `<h1>` đến `<h6>` | `heading` | Level tương ứng 1-6 |
| `<input type="checkbox">` | `checkbox` | |
| `<input type="radio">` | `radio` | |
| `<input type="text">` | `textbox` | |
| `<input type="email">` | `textbox` | |
| `<input type="password">` | nội bộ, không phải textbox | |
| `<input type="search">` | `searchbox` | |
| `<textarea>` | `textbox` | |
| `<select>` | `combobox` | |
| `<table>` | `table` | |
| `<tr>` | `row` | |
| `<td>` | `cell` | |
| `<th>` | `columnheader` / `rowheader` | |
| `<img alt="...">` | `img` | Phải có alt text |
| `<img alt="">` | `presentation` | Alt rỗng = decorative |
| `<ul>`, `<ol>` | `list` | |
| `<li>` | `listitem` | |
| `<nav>` | `navigation` | |
| `<main>` | `main` | |
| `<header>` | `banner` | |
| `<footer>` | `contentinfo` | |
| `<form>` | `form` | |
| `<dialog>` | `dialog` | |
| `<div>`, `<span>` | *(không có role)* | Vô nghĩa với screen reader |

#### Explicit Role (Role khai báo thủ công bằng `role="..."`)

Dùng khi element không có semantic HTML phù hợp (thường gặp trong component custom):

```html
<!-- Custom button dùng div -->
<div role="button" tabindex="0" onclick="handleClick()">
    Click me
</div>

<!-- Custom tab panel -->
<div role="tablist">
    <div role="tab" aria-selected="true">Tab 1</div>
    <div role="tab" aria-selected="false">Tab 2</div>
</div>

<!-- Custom alert -->
<div role="alert">Đã xảy ra lỗi!</div>

<!-- Custom dialog/modal -->
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
    <h2 id="dialog-title">Xác nhận xóa</h2>
</div>
```

```typescript
// Playwright tìm được nhờ role khai báo thủ công
await page.getByRole('button', { name: 'Click me' }).click();
await page.getByRole('tab', { name: 'Tab 1' }).click();
await page.getByRole('dialog', { name: 'Xác nhận xóa' }).waitFor();
```

---

### Các ARIA Attributes quan trọng

ARIA không chỉ có `role` — còn có các **attribute bổ sung** mô tả trạng thái và quan hệ:

| Attribute | Mục đích | Ví dụ |
|---|---|---|
| `aria-label` | Đặt tên cho element (dùng thay label) | `<button aria-label="Close dialog">✕</button>` |
| `aria-labelledby` | Tham chiếu đến element khác làm label | `<input aria-labelledby="title-id">` |
| `aria-describedby` | Mô tả thêm cho element | `<input aria-describedby="hint-id">` |
| `aria-hidden` | Ẩn element khỏi screen reader | `<span aria-hidden="true">🎉</span>` |
| `aria-disabled` | Đánh dấu element bị vô hiệu hóa | `<button aria-disabled="true">` |
| `aria-expanded` | Dropdown/accordion đang mở/đóng | `<button aria-expanded="true">` |
| `aria-selected` | Item đang được chọn | `<div role="tab" aria-selected="true">` |
| `aria-checked` | Checkbox/radio đang được check | `<div role="checkbox" aria-checked="true">` |
| `aria-required` | Field bắt buộc nhập | `<input aria-required="true">` |
| `aria-invalid` | Field đang có lỗi | `<input aria-invalid="true">` |
| `aria-live` | Thông báo thay đổi động | `<div aria-live="polite">` |

#### Playwright với ARIA attributes

```html
<!-- HTML -->
<button aria-label="Đóng cửa sổ">✕</button>
<button aria-disabled="true">Disabled</button>
<div role="tab" aria-selected="true">Tab đang active</div>
<input aria-required="true" aria-label="Email">
```

```typescript
// getByRole dùng aria-label để match "name"
await page.getByRole('button', { name: 'Đóng cửa sổ' }).click();

// Tìm tab đang được chọn
await page.getByRole('tab', { selected: true }).textContent();

// Tìm element bị disabled
await page.getByRole('button', { disabled: true }).isDisabled();

// Tìm input bắt buộc
await page.getByRole('textbox', { name: 'Email' }).fill('user@example.com');
```

---

### Cách "name" trong getByRole() được tính

Khi dùng `getByRole('button', { name: 'Submit' })`, Playwright tính **accessible name** theo thứ tự ưu tiên:

```
1. aria-labelledby  →  tham chiếu đến element khác
2. aria-label       →  label trực tiếp trên element
3. <label for="id"> →  label HTML liên kết
4. Text content     →  text bên trong element
5. title attribute  →  tooltip (ít dùng nhất)
```

```html
<!-- 1. aria-labelledby: name = text của #btn-label -->
<span id="btn-label">Lưu tài liệu</span>
<button aria-labelledby="btn-label">💾</button>
<!-- getByRole('button', { name: 'Lưu tài liệu' }) ✅ -->

<!-- 2. aria-label: name = giá trị aria-label -->
<button aria-label="Close dialog">✕</button>
<!-- getByRole('button', { name: 'Close dialog' }) ✅ -->

<!-- 3. Text content: name = text hiển thị -->
<button>Submit Form</button>
<!-- getByRole('button', { name: 'Submit Form' }) ✅ -->

<!-- 4. title: name = giá trị title (ít dùng) -->
<button title="Save changes">💾</button>
<!-- getByRole('button', { name: 'Save changes' }) ✅ -->
```

---

### Ví dụ thực tế: Element KHÔNG có ARIA role

`<div>` và `<span>` không có role → không dùng được `getByRole()`:

```html
<!-- Các element này KHÔNG có ARIA role -->
<div class="card">Nội dung card</div>
<span class="badge">New</span>
<div class="tooltip-content">Mô tả thêm</div>
```

```typescript
// ❌ Không thể dùng getByRole vì div/span không có role
// page.getByRole(???, { name: 'Nội dung card' })

// ✅ Phải dùng getByText hoặc getByTestId
await page.getByText('Nội dung card').click();
await page.getByTestId('card').click();

// ✅ Hoặc developer thêm role thủ công
// <div role="region" aria-label="Product card">...</div>
// page.getByRole('region', { name: 'Product card' })
```

---

### Tóm tắt ARIA Role

```
ARIA Role = "nhãn dán ngữ nghĩa" cho element

Có 2 loại:
├── Implicit Role: HTML tự có (button, link, heading, checkbox...)
│   └── Dùng đúng semantic HTML → tự nhiên có role
│
└── Explicit Role: Khai báo thủ công bằng role="..."
    └── Dùng khi custom component không có semantic HTML

Playwright dùng ARIA role vì:
✅ Phản ánh cách người dùng thực sự tương tác
✅ Bền vững — không phụ thuộc vào CSS class hay DOM structure
✅ Thân thiện accessibility — test tốt = app accessible tốt
✅ Dễ đọc — getByRole('button') rõ hơn $('.btn-primary')
```
