# Lesson 9 - Git & CSS Selector & Playwright Selector

---

## Mục lục

1. [Git - Merge](#1-git---merge)
2. [Git - Conflict](#2-git---conflict)
3. [Git - Rebase](#3-git---rebase)
4. [Git - Squash](#4-git---squash)
5. [CSS Selector](#5-css-selector)
6. [Playwright Selector (Locator)](#6-playwright-selector-locator)

---

## 1. Git - Merge

### Khái niệm

> Làm việc nhóm = làm **việc riêng** rồi **gộp** công việc vào nhau.
> Trong git, gộp vào gọi là **"merge"** — tức là gộp code từ nhánh A vào nhánh B.

```
feature/A  ──────────────►
                           \
                            merge──►  main
main       ──────────────►
```

### Merge strategy — 2 chiến lược chính

#### a) Fast-forward merge

- Khi merge **không tạo ra commit merge mới**.
- Xảy ra khi **nhánh chính (main) không có thay đổi nào** kể từ lúc tạo nhánh feature.
- Git chỉ đơn giản "tua nhanh" con trỏ main lên đến commit cuối của feature.

```
Trước merge:          Sau merge:
feature/A  main       feature/A  main
  1  ──►  1             1  ──►  1
  2  ──►  2             2  ──►  2
  3                     3  ──►  3
  4                     4  ──►  4
  5                     5  ──►  5
```

**Lệnh:**
```bash
git checkout main
git merge feature/A
```

#### b) Three-way merge

- Khi merge **có tạo ra 1 commit merge mới**.
- Xảy ra khi **cả 2 nhánh đều có commit mới** sau khi tách ra — lịch sử của 2 nhánh đã khác nhau.
- Git tạo ra 1 commit đặc biệt (merge commit) để ghi lại việc gộp.

```
Trước merge:              Sau merge (có merge commit):
feature/A  main           feature/A  main
  1──►1                     1──►1
  2──►2                     2──►2
  3  ─►6                    3  ─►6
  4    7                    4    7
  5                         5   ─►  8 (merge commit)
```

---

## 2. Git - Conflict

### Khái niệm

- **Conflict (xung đột)** xảy ra khi **2 người cùng sửa cùng 1 vị trí** trong file, sau đó merge vào nhau.
- Git không biết giữ phần nào → đánh dấu để con người tự quyết định.

### Conflict trông như thế nào?

Khi có conflict, git chèn các ký hiệu đặc biệt vào file:

```
<<<<<<< HEAD
nội dung của nhánh mình (current)
=======
nội dung của nhánh muốn merge vào (incoming)
>>>>>>> feature/2
```

| Phần | Ý nghĩa |
|------|---------|
| `<<<<<<< HEAD` đến `=======` | Nội dung đang ở **nhánh của mình** |
| `=======` đến `>>>>>>> branch` | Nội dung của **nhánh muốn merge vào** |

### Cách xử lý conflict

**Bước 1:** Xóa hoặc giữ các phần code tương ứng theo ý muốn.

**Bước 2:** Xóa các dòng đánh dấu conflict (`<<<<<<`, `=======`, `>>>>>>>`).

**Bước 3:** Commit lại sau khi resolve xong.

### Quy trình xử lý conflict chuyên nghiệp

1. **Đọc code**, xác định conflict ở đâu.
2. Giải quyết các conflict **"dễ"** trước — không cần trao đổi.
3. Với conflict **"khó"** → cần **trao đổi với author** trước khi merge, tránh làm mất code của người khác.

---

## 3. Git - Rebase

### Khái niệm

> **Git rebase** là lệnh để **thay đổi base (gốc)** của một branch, giúp tạo ra lịch sử commit **tuyến tính** và **sạch sẽ** hơn.

Thay vì tạo merge commit như three-way merge, rebase "cắt" các commit của feature branch và "dán lại" lên đầu branch đích.

```
Trước rebase:             Sau rebase (rồi merge):
feature/A  main           feature/A  main
  1──►1                     1──►1
  2──►2                     2──►2
  3  ─►6                    3  ─►6
  4    7                    4    7  ──►3'
  5                                    4'
                                       5'
```

Các commit 3, 4, 5 của feature được "chép lại" thành 3', 4', 5' với base mới là commit 7 của main.

### Lệnh

```bash
# Đứng trên nhánh feature, rebase lên nhánh main
git rebase main
```

### Ưu điểm

- Lịch sử commit **tuyến tính**, dễ đọc hơn.
- Không có merge commit thừa.

---

## 4. Git - Squash

### Khái niệm

> **Git squash** là kỹ thuật **gộp nhiều commit thành một commit duy nhất**, giúp lịch sử commit **gọn gàng** và **dễ đọc** hơn.

**Ví dụ:**
```
Trước squash (7 commit nhỏ lẻ):        Sau squash (3 commit có nghĩa):
- Add Playwright login test             - test: Add complete Playwright login test suite
- Fix selector typo                     - Implement login form tests with validation
- Add form validation tests             - Add error handling and screenshots
- Fix flaky test timeout                  and Configure proper wait conditions
- Update test CSS selectors
- Add screenshot on failure
- Fix wait condition
```

### Cú pháp

```bash
git rebase -i HEAD~{số_lượng_commit_muốn_gộp}

# Ví dụ: gộp 3 commit gần nhất
git rebase -i HEAD~3
```

### Các bước thực hiện Squash trong VIM

1. Lệnh trên sẽ mở editor VIM với danh sách các commits.

2. Gõ **`i`** để vào chế độ INSERT (edit).

3. **Commit đầu tiên** luôn giữ nguyên `pick`.
   Các commit muốn gộp vào → đổi `pick` thành **`s`** hoặc **`squash`**.

   ```
   pick e24bc61 feat: add example for lesson 09   ← giữ nguyên
   s 15aeb5a feat: add file1                      ← đổi thành "s"
   ```

4. Gõ **`ESC`** để thoát chế độ INSERT.

5. Gõ **`:wq`** rồi **`Enter`** để lưu và thoát VIM.

6. Màn hình tiếp theo: chỉnh sửa **commit message** cuối cùng.
   - Dòng có `#` = comment, bị bỏ qua.
   - Thường chỉ giữ lại message đầu tiên, comment các message còn lại bằng `#`.

7. Gõ **`:wq`** → **`Enter`** lần nữa để hoàn thành.

---

## 5. CSS Selector

### Khái niệm

> **CSS Selector** là cú pháp dùng để **chọn (select) các phần tử HTML** trong DOM.
> Được sử dụng rộng rãi trong CSS styling và automation testing.

### CSS vs XPath — So sánh nhanh

| Mục tiêu | CSS Selector | XPath |
|----------|-------------|-------|
| Tất cả elements | `*` | `//*` |
| Tất cả `<div>` | `div` | `//div` |
| Tất cả `<p>` con | `p *` | `//p/*` |
| Element theo ID | `#title` | `//*[@id="title"]` |
| Element theo Class | `.title` | `//*[contains(@class,'title')]` |
| Element theo Attribute | `[name="title"]` | `//*[contains(@name,'title')]` |
| Con đầu tiên của `<div>` | `div:first-child` | `//div/*[0]` |
| Next element | `p + *` | `//p/following-sibling::*[0]` |

> **CSS cú pháp ngắn gọn, hiệu năng cao hơn XPath.**

### Bảng tham chiếu CSS Selector thực hành

| Loại | CSS Selector | XPath |
|------|-------------|-------|
| Tag | `div` | `//div` |
| ID | `#registrationForm` | `//form[@id="registrationForm"]` |
| Class | `.form-group` | `//div[@class='form-group']` |
| Child (con trực tiếp) | `#parent > input` | `//div[@id='parent']/input` |
| Descendant (con cháu) | `#ancestor div` | `//div[@id='ancestor']/descendant::div` |
| Combine | `div, input` | `//div \| //input` |

### Hạn chế của CSS Selector

CSS Selector **không thể** chọn trong một số trường hợp:
- Chọn phần tử **cha (parent)**
- Chọn **anh em đứng TRƯỚC** (preceding sibling)
- Chọn theo **text một phần**

→ Trong các trường hợp này, bắt buộc dùng **XPath**.

---

## 6. Playwright Selector (Locator)

### Khái niệm

> **Playwright Selector** là hệ thống locator **mạnh mẽ và linh hoạt** của Playwright để tìm và tương tác với các phần tử trên trang web.

### Tổng quan các locator

```typescript
// CSS selector
await page.locator('button');

// XPath
await page.locator('//input[@id="username-input"]');

// Playwright locators (khuyên dùng)
await page.getByRole('textbox', { name: 'Username' });
await page.getByPlaceholder('Enter your username');
await page.getByLabel('Username');
await page.getByTitle('Username Field');
await page.getByTestId('usernameField');
await page.getByText('Welcome back');
await page.getByAltText('playwright logo');
```

### Các locator thường dùng

---

#### `page.getByRole()`

Tìm element theo **ARIA role** — vai trò ngữ nghĩa của element.

**Các role phổ biến:**

| Role | Loại element |
|------|-------------|
| `button` | Nút bấm |
| `link` | Liên kết (`<a>`) |
| `textbox` | Ô nhập text (`<input>`) |
| `checkbox` | Hộp kiểm |
| `radio` | Nút radio |
| `heading` | Tiêu đề (`<h1>...<h6>`) |
| `listitem` | Mục trong danh sách |

**Ví dụ:**
```typescript
// Tìm button có text "Submit"
await page.getByRole('button', { name: 'Submit' }).click();

// Tìm link có text "Learn more"
await page.getByRole('link', { name: 'Learn more' }).click();

// Tìm heading level 1
await page.getByRole('heading', { level: 1 }).textContent();

// Tìm checkbox đã được check
await page.getByRole('checkbox', { checked: true });
```

---

#### `page.getByText()`

Tìm element theo **text hiển thị** trên trang.

```typescript
// Tìm chính xác text
await page.getByText('Welcome back').click();

// Tìm text có chứa (substring)
await page.getByText('Welcome', { exact: false });

// Dùng regex (case insensitive)
await page.getByText(/welcome/i);

// Kết hợp với locator khác (chaining)
await page.locator('div').getByText('Hello');
```

---

#### `page.getByLabel()`

Tìm **input element** thông qua text của `<label>` liên kết với nó.

```html
<!-- HTML -->
<label for="email">Email address</label>
<input id="email" type="email">
```

```typescript
// Tìm input thông qua label
await page.getByLabel('Email address').fill('test@example.com');

// Tìm không chính xác (có chứa text)
await page.getByLabel('Email', { exact: false }).fill('test@example.com');

// Label bao quanh input
// <label>Password <input type="password" /></label>
await page.getByLabel('Password').fill('secret');
```

---

#### `page.getByPlaceholder()`

Tìm input theo thuộc tính **`placeholder`**.

```html
<input type="email" placeholder="name@example.com"/>
```

```typescript
await page.getByPlaceholder('name@example.com').fill('playwright@microsoft.com');
```

---

#### `page.getByTitle()`

Tìm element theo thuộc tính **`title`**.

```html
<span title='Issues count'>25 issues</span>
```

```typescript
await expect(page.getByTitle('Issues count')).toHaveText('25 issues');
```

---

#### `page.getByAltText()`

Tìm element (thường là `<img>`) theo thuộc tính **`alt`**.

```html
<img alt="playwright logo" src="/img/playwright-logo.svg" width="100" />
```

```typescript
await page.getByAltText('playwright logo').click();
```

---

#### `page.getByTestId()`

Tìm element theo **thuộc tính test ID** (mặc định là `data-testid`).

```html
<button data-testid="directions">Itinéraire</button>
```

```typescript
await page.getByTestId('directions').click();

// Nếu dùng thuộc tính khác (vd: id)
await page.setTestIdAttribute('id');
await page.getByTestId('directions').click();
```

---

### Tóm tắt — Khi nào dùng locator nào?

| Tình huống | Locator nên dùng |
|------------|-----------------|
| Element có role rõ ràng (button, link...) | `getByRole()` |
| Element có text hiển thị | `getByText()` |
| Form input có label | `getByLabel()` |
| Form input có placeholder | `getByPlaceholder()` |
| Element có attribute `title` | `getByTitle()` |
| Hình ảnh có alt text | `getByAltText()` |
| Element được đánh dấu test-id | `getByTestId()` |
| Phần tử phức tạp, cần CSS/XPath | `page.locator('css hoặc xpath')` |

---

### Ví dụ code thực tế (từ demo)

```typescript
import { test } from '@playwright/test';

test("Demo Playwright Selector", async ({ page }) => {
    await page.goto("https://material.playwrightvn.com/01-xpath-register-page.html");

    // Dùng XPath locator
    const title1 = await page.locator("//h1[@id='self']").textContent();
    // Dùng getByRole
    const title2 = await page.getByRole("heading", { name: "User Registration" }).textContent();

    // Tương tác với các element
    await page.getByRole("checkbox", { name: "Traveling" }).check();
    await page.getByRole("checkbox", { name: "Cooking" }).check();
    await page.getByRole("radio", { name: "Male", exact: true }).click();
    await page.getByLabel("Username").fill("hihi");

    console.log(title1);
    console.log(title2);
});
```

---

## Tổng kết Lesson 9

| Chủ đề | Điểm chính |
|--------|-----------|
| **Git Merge** | Gộp code từ nhánh A vào B; 2 kiểu: Fast-forward (không tạo commit merge) và Three-way (tạo merge commit) |
| **Git Conflict** | Xảy ra khi 2 người sửa cùng chỗ; giải quyết bằng cách chọn giữ phần nào, xóa markers |
| **Git Rebase** | Thay đổi base của branch, tạo lịch sử tuyến tính sạch hơn; dùng `git rebase <tên_nhánh>` |
| **Git Squash** | Gộp nhiều commit nhỏ thành 1 commit lớn có nghĩa; dùng `git rebase -i HEAD~N` |
| **CSS Selector** | Cú pháp ngắn gọn, hiệu năng cao hơn XPath; hạn chế với parent/preceding-sibling/partial text |
| **Playwright Locator** | Bộ locators mạnh mẽ: `getByRole`, `getByText`, `getByLabel`, `getByPlaceholder`, `getByTitle`, `getByAltText`, `getByTestId` |
