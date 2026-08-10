# Lesson 5 - DOM Terminology & Playwright Basic

> Nguồn: PlaywrightVN - Học automation test từ chưa biết gì

---

## Mục lục

1. [JavaScript - Functions Nâng Cao](#1-javascript---functions-nâng-cao)
2. [DOM (Document Object Model)](#2-dom-document-object-model)
3. [Selector](#3-selector)
4. [Playwright Basic Syntax](#4-playwright-basic-syntax)

---

## 1. JavaScript - Functions Nâng Cao

### 1.1 Function Expression (Biểu thức hàm)

Thay vì khai báo hàm theo kiểu truyền thống, ta có thể **gán hàm vào một biến**:

```js
// Cách truyền thống (Function Declaration)
function add(a, b) {
    return a + b;
}

// Function Expression (gán cho biến)
const add = function(a, b) {
    return a + b;
};

// Cách gọi giống nhau
console.log(add(2, 3)); // 5
```

**Thực hành:**
```js
// Hàm có tham số name, trả về "Hello <name>"
const hello = function(name) {
    return `Hello ${name}`;
};
console.log(hello('Tuyền')); // Hello Tuyền

// Hàm tính giá trị = price * quantity - discount
const cal = function(price, quantity, discount) {
    return price * quantity - discount;
};
console.log(`Tổng giá trị là: ${cal(50000, 2, 10000)}`); // 90000
```

---

### 1.2 Lambda Function (Arrow Function)

Arrow function xuất hiện từ **ES6 (ES2015)**, là cách viết **ngắn gọn hơn** cho function, dùng dấu `=>`.

**So sánh 3 cách viết cùng 1 hàm:**

```js
// 1. Function Declaration (truyền thống)
function add(a, b) {
    return a + b;
}

// 2. Function Expression
const add = function(a, b) {
    return a + b;
};

// 3. Arrow Function (Lambda)
const add = (a, b) => {
    return a + b;
};

// 4. Rút gọn nhất (implicit return - chỉ 1 dòng code)
const add = (a, b) => a + b;
```

**Các trường hợp đặc biệt:**

```js
// Không có tham số: bắt buộc phải có () rỗng
const greet = () => console.log("Hello!");
const getRandom = () => Math.random();

// Một tham số: có thể bỏ dấu ngoặc tròn
const double = x => x * 2;
const square = x => x * x;

// Hoặc giữ dấu ngoặc (tùy style)
const triple = (x) => x * 3;
```

**Thực hành:**
```js
// Lambda function
const hello2 = (name) => {
    return `Hello ${name}`;
};
console.log(hello2('Tuyền')); // Hello Tuyền

const cal2 = (price, quantity, discount) => {
    return price * quantity - discount;
};
console.log(`Tổng giá trị là: ${cal2(20_000, 10, 10_000)}`); // 190000
```

---

### 1.3 Anonymous Function (Hàm ẩn danh)

**Anonymous function** là hàm **không có tên**. Dùng khi hàm chỉ cần dùng **một lần** hoặc làm **callback**.

```js
// Named function (có tên)
function namedFunction() {
    console.log("I have a name!");
}

// Anonymous function không thể đứng một mình (SyntaxError)
// function() { ... } // LỖI!

// Phải được sử dụng ngay bằng 2 cách:

// 1. Gán cho biến
const anonymousFunc = function() {
    console.log("I'm anonymous but stored in a variable!");
};

// 2. Dùng làm callback
setTimeout(function() {
    console.log("Anonymous callback!");
}, 1000);
```

---

## 2. DOM (Document Object Model)

### 2.1 DOM là gì?

Khi vào một website, ta nhìn thấy: các khối text, hình ảnh, liên kết, ô input...

Máy tính "nhìn" website dưới dạng **cây có cấu trúc**. Cấu trúc cây đó gọi là **DOM (Document Object Model)**.

**Cách xem DOM:**
- Bấm phím **F12** (hoặc chuột phải → "Inspect") → chọn tab **Element**

```
document
└── <html>  (Root element)
    ├── <head>
    │   └── <title> "My title"
    └── <body>
        ├── <h1> "A heading"
        └── <a href="..."> "Link text"
```

---

### 2.2 Cấu trúc một Element HTML

Lấy ví dụ: `<option value="usa">United States</option>`

| Thành phần | Ví dụ | Ý nghĩa |
|---|---|---|
| **Thẻ mở** | `<option` | Bắt đầu element |
| **Thuộc tính** | `value` | Tên thuộc tính |
| **Giá trị thuộc tính** | `"usa"` | Giá trị của thuộc tính |
| **Text** | `United States` | Nội dung hiển thị |
| **Thẻ đóng** | `</option>` | Kết thúc element |

Một thẻ có thể có **nhiều thuộc tính**:
```html
<option value="usa" school="HN">United States</option>
```

**Thẻ tự đóng** (không có nội dung bên trong):
```html
<img src="image.jpg" alt="Image description"/>
<br/>
<hr/>
```

---

### 2.3 Các thẻ HTML thường gặp

#### Thẻ Cấu Trúc Cơ Bản

| Thẻ | Ý nghĩa |
|---|---|
| `<html>` | Thẻ gốc của trang |
| `<head>` | Chứa metadata: tiêu đề website, hiển thị Google |
| `<body>` | Nội dung của cả website hiển thị |
| `<div>` | Khối/container chung |
| `<span>` | Inline container |
| `<header>`, `<footer>`, `<nav>`, `<section>` | Thẻ ngữ nghĩa |

#### Thẻ Nội Dung

| Thẻ | Ý nghĩa |
|---|---|
| `<h1>` đến `<h6>` | Tiêu đề (heading) |
| `<p>` | Đoạn văn |
| `<a>` | Liên kết |
| `<img>` | Hình ảnh |
| `<ul>`, `<ol>`, `<li>` | Danh sách |

#### Thẻ Form (Quan trọng cho Testing!)

| Thẻ | Ý nghĩa |
|---|---|
| `<form>` | Biểu mẫu |
| `<input>` | Ô nhập liệu (text, password, checkbox, radio, etc.) |
| `<button>` | Nút bấm |
| `<select>` và `<option>` | Dropdown |
| `<textarea>` | Vùng văn bản nhiều dòng |

> **Lưu ý:** Có 2 loại thẻ:
> - **Thẻ tiêu chuẩn**: do tổ chức uy tín mozilla định nghĩa
> - **Thẻ tự định nghĩa**: do lập trình viên/website tự định nghĩa

---

## 3. Selector

### 3.1 Selector là gì?

**Automation = tương tác** với các phần tử trên trang web (input, fill, click, ...).

Để tương tác được, ta cần **tìm** được phần tử đó. **Selector là công cụ giúp ta tìm** phần tử trên DOM.

### 3.2 Ba loại Selector thường dùng

| Loại | Ưu điểm | Nhược điểm | Ví dụ |
|---|---|---|---|
| **XPath** | Đa dạng, dùng được trong ~99.99% trường hợp | Hơi dài | `//button[normalize-space()='Add to cart']` |
| **CSS Selector** | Ngắn gọn, performance cao | Không linh hoạt bằng XPath | `.add-to-cart` |
| **Playwright Selector** | Cú pháp ngắn, không phụ thuộc DOM | Chỉ dùng trong Playwright | `page.getByText("Add to cart")` |

**Thứ tự ưu tiên sử dụng:**
> **Playwright selector > CSS Selector > XPath**

Tuy nhiên, vẫn cần học cả 3 loại vì có dự án "thích" dùng CSS, có dự án "thích" dùng XPath.

---

### 3.3 XPath Selector

**XPath = XML Path** — dùng để điều hướng qua các element trong tài liệu XML/HTML.

#### Hai loại XPath:

**1. XPath Tuyệt đối (Absolute):**
- Đi dọc theo toàn bộ cây DOM từ gốc
- Bắt đầu bằng `/` (1 dấu gạch chéo)
- Ví dụ: `/html/body/div/form/input`
- **Nhược điểm:** Dễ bị hỏng khi dev thay đổi cấu trúc trang

**2. XPath Tương đối (Relative) — Nên dùng:**
- Tìm dựa vào đặc tính của element, không cần đường dẫn đầy đủ
- Bắt đầu bằng `//` (2 dấu gạch chéo)
- Cú pháp: `//tenthe[@thuoctinh="giatri"]`

#### Cú pháp XPath tương đối:

```
//tên_thẻ[@tên_thuộc_tính="giá_trị"]
```

**Ví dụ thực tế (xem trong DevTools với F12 → tab Elements):**

```xpath
//h1[@id="self"]
//form[@id="registrationForm"]
//input[@id="email"]
//input[@type="email"]
```

> **Mẹo:** Trong DevTools, gõ XPath vào thanh tìm kiếm (Ctrl+F trong tab Elements) để kiểm tra selector có đúng không. Kết quả "1 of 1" nghĩa là tìm đúng 1 element duy nhất.

**Nhược điểm của XPath tuyệt đối:**
- Nếu có nhiều đối tượng giống nhau, không thể bắt được
- Bảng sinh ra động không thể dùng XPath tuyệt đối
- Nếu phần tử nằm sâu trong nhiều thẻ, mất thời gian tìm và không ổn định

---

## 4. Playwright Basic Syntax

### 4.1 Cấu trúc một Test

**Automation = tương tác + verify**

Trong bài này học cách **tương tác** với các phần tử:
- Navigation (điều hướng trang)
- Click
- Fill (nhập liệu)

#### Khai báo một test:

```ts
import { test } from '@playwright/test';

test('<tên test>', async ({ page }) => {
    // Code của test
});
```

#### Khai báo các step bên trong test:

```ts
test('<tên test>', async ({ page }) => {
    await test.step('Tên step', async () => {
        // Code here
    });
});
```

> **Lưu ý quan trọng:** Mỗi step nên được **map 1-1 với test case** để dễ maintain.

---

### 4.2 Navigate — Điều hướng trang

```ts
await page.goto('https://pw-practice.playwrightvn.com/');
```

---

### 4.3 Locate — Chọn phần tử

Dùng `page.locator("<selector>")` để chọn phần tử trên trang:

```ts
// Dùng XPath
page.locator("//input[@id='email']")

// Dùng CSS selector
page.locator(".email-input")
```

---

### 4.4 Click — Nhấp chuột

```ts
// Click thường
await page.locator("//button").click();

// Double click
await page.locator("//button").dblclick();

// Click chuột phải
page.locator("//button").click({
    button: 'right'
});

// Click kèm bấm phím khác (ví dụ Shift+Click)
page.locator("").click({
    modifiers: ['Shift'],
});
```

---

### 4.5 Input — Nhập liệu

```ts
// fill: giống việc paste content vào ô input (nhanh)
page.locator("//input").fill('Playwright Viet Nam');

// pressSequentially: giống việc gõ từng chữ cái (chậm hơn, có delay)
page.locator("//input").pressSequentially('Playwright Viet Nam', {
    delay: 100,
});
```

---

### 4.6 Radio / Checkbox

```ts
// Lấy trạng thái hiện tại (đang checked hay không)
const isChecked = await page.locator("//input").isChecked();

// Check (tích vào)
await page.locator("//input").check();

// Uncheck (bỏ tích)
await page.locator("//input").setChecked(false);
```

---

### 4.7 Select — Chọn dropdown

```ts
await page.locator('//select[@id="country"]')
    .selectOption({ label: 'USA' });
```

---

### 4.8 Upload File

```ts
await page.locator("//input[@id='profile']")
    .setInputFiles("<file-path>");
```

---

### 4.9 Xử lý Dialog (Confirm/Alert)

Khi có popup confirm xuất hiện, cần **setup sự kiện TRƯỚC** khi trigger dialog:

```ts
// Nhấn OK (accept)
page.on('dialog', async dialog => dialog.accept());
await page.click("//button[text()='Delete']");

// Nhấn Cancel (dismiss)
// page.on('dialog', async dialog => dialog.dismiss());
// await page.click("//button[text()='Delete']");
```

> **Quan trọng:** Sự kiện `page.on('dialog', ...)` phải được viết **TRƯỚC** hành động trigger dialog.

---

### 4.10 Ví dụ Test Hoàn Chỉnh

```ts
import { test } from '@playwright/test';

test('Test case 1', async ({ page }) => {

    await test.step('Step Navigate Material website', async () => {
        await page.goto("https://material.playwrightvn.com/");
    });

    await test.step('Step Click Lesson 1', async () => {
        await page.locator("//a[text()='Bài học 1: Register Page (có đủ các element)']").click();
    });

    await test.step('Step Input', async () => {
        await page.locator("//input[@id='username']").fill("Tuyen Vo");
        await page.locator("//input[@id='email']").pressSequentially("tuyen@gmail.com", { delay: 100 });
    });

    await test.step('Step Radio button/ Checkbox', async () => {
        let isCheckedMale = await page.locator("//input[@id='male']").isChecked();
        console.log(isCheckedMale); // false

        await page.locator("//input[@id='male']").check();
        isCheckedMale = await page.locator("//input[@id='male']").isChecked();
        console.log(isCheckedMale); // true
    });

    await test.step('Step Select option', async () => {
        await page.locator("//select[@id='country']").selectOption('Canada');
    });

    await test.step('Step Upload file', async () => {
        await page.locator("//input[@id='profile']").setInputFiles("tests/data-test/data-te...");
    });

});
```

---

## Tóm tắt nhanh

| Chủ đề | Điểm chính |
|---|---|
| **Function Expression** | Gán hàm vào biến — `const fn = function() {}` |
| **Arrow Function** | Cú pháp ngắn — `const fn = () => {}` hoặc `const fn = () => value` |
| **Anonymous Function** | Hàm không tên, dùng làm callback hoặc gán cho biến |
| **DOM** | Cây cấu trúc HTML của trang web, xem bằng F12 → Elements |
| **HTML Element** | Gồm: thẻ mở, thuộc tính, giá trị, text, thẻ đóng |
| **XPath** | Selector mạnh nhất, dùng `//tag[@attr="value"]` |
| **CSS Selector** | Ngắn gọn, dùng `.class` hoặc `#id` |
| **Playwright Selector** | Thân thiện nhất, `page.getByText()`, `page.getByRole()` |
| **page.goto()** | Điều hướng đến URL |
| **page.locator()** | Chọn element theo selector |
| **click() / dblclick()** | Click đơn / click đôi |
| **fill() / pressSequentially()** | Nhập text nhanh / nhập từng chữ |
| **check() / setChecked()** | Tích / bỏ tích checkbox |
| **selectOption()** | Chọn giá trị trong dropdown |
| **setInputFiles()** | Upload file |
| **page.on('dialog', ...)** | Xử lý popup confirm/alert |
