# CSS Selectors - Tổng Hợp Kiến Thức Chi Tiết

> Nguồn:
> - CSS Selectors Cheatsheet by nana jeon (Feb 2019) — [PDF](https://appletree.or.kr/quick_reference_cards/CSS/CSS%20selectors%20cheatsheet.pdf)
> - https://material.playwrightvn.com/14-css-selector.html

---

## Mục lục

1. [CSS Selector trong Playwright là gì?](#1-css-selector-trong-playwright-là-gì)
2. [Cấu trúc HTML mẫu dùng để thực hành](#2-cấu-trúc-html-mẫu-dùng-để-thực-hành)
3. [Basic Selectors](#3-basic-selectors)
   - [Type Selector](#31-type-selector---element)
   - [ID Selector](#32-id-selector---id)
   - [Class Selector](#33-class-selector---class)
   - [Universal Selector](#34-universal-selector---)
4. [Combination Selectors](#4-combination-selectors)
   - [Descendant Selector](#41-descendant-selector---khoảng-trắng)
   - [Child Selector](#42-child-selector---)
   - [Adjacent Sibling Selector](#43-adjacent-sibling-selector---)
   - [General Sibling Selector](#44-general-sibling-selector---)
   - [Comma Combinator](#45-comma-combinator---)
   - [Combine Class Selector](#46-combine-class-selector---elementclass)
   - [Combine Descendant & ID](#47-combine-descendant--id---id-descendant)
   - [Combine Universal Selector](#48-combine-universal-selector---element-)
5. [Pseudo-class Selectors - Structural](#5-pseudo-class-selectors---structural)
   - [:first-child](#51-first-child)
   - [:last-child](#52-last-child)
   - [:nth-child(n)](#53-nth-childn)
   - [:nth-last-child(n)](#54-nth-last-childn)
   - [:only-child](#55-only-child)
   - [:first-of-type](#56-first-of-type)
   - [:last-of-type](#57-last-of-type)
   - [:nth-of-type(n)](#58-nth-of-typen)
   - [:only-of-type](#59-only-of-type)
6. [Pseudo-class Selectors - Other](#6-pseudo-class-selectors---other)
   - [:empty](#61-empty)
   - [:not()](#62-not)
7. [Attribute Selectors](#7-attribute-selectors)
   - [[attr]](#71-attr---có-attribute)
   - [[attr="value"]](#72-attrvalue---khớp-chính-xác)
   - [[attr^="value"]](#73-attrvalue---bắt-đầu-bằng)
   - [[attr$="value"]](#74-attrvalue---kết-thúc-bằng)
   - [[attr*="value"]](#75-attrvalue---chứa-chuỗi)
8. [only-child vs only-of-type](#8-only-child-vs-only-of-type)
9. [nth-child vs nth-of-type](#9-nth-child-vs-nth-of-type)
10. [Cheat Sheet Tổng Hợp](#10-cheat-sheet-tổng-hợp)

---

## 1. CSS Selector trong Playwright là gì?

Thay vì dùng `getByRole()`, `getByText()`... Playwright cũng cho phép dùng **CSS Selector** trực tiếp thông qua `page.locator()`:

```typescript
// Cách dùng CSS selector trong Playwright
await page.locator('css-selector').click()
await page.locator('css-selector').fill('text')
await page.locator('css-selector').textContent()
await page.locator('css-selector').isVisible()
await page.locator('css-selector').count()
await page.locator('css-selector').waitFor()
```

### So sánh getByRole vs CSS Selector

```typescript
// Cách 1: getByRole (khuyến nghị)
await page.getByRole('button', { name: 'Đăng Ký' }).click();

// Cách 2: CSS Selector (linh hoạt hơn)
await page.locator('button[type="submit"]').click();
await page.locator('#signup-form button').click();
await page.locator('.btn-primary').click();
```

> **Khi nào dùng CSS Selector?** Khi element không có role/label/text rõ ràng, hoặc khi cần tìm theo cấu trúc DOM phức tạp (nth-child, sibling, attribute...).

---

## 2. Cấu trúc HTML mẫu dùng để thực hành

Trang https://material.playwrightvn.com/14-css-selector.html là form đăng ký với cấu trúc HTML như sau:

```html
<div class="container">
  <form id="signup-form" class="form signup-form">
    <h2 class="form-title">Đăng Ký</h2>

    <!-- form-group 1: Username -->
    <div class="form-group">
      <label for="username">Tên đăng nhập</label>
      <input type="text" id="username" name="username"
             class="form-control" placeholder="Nhập tên đăng nhập">
      <small class="hint">Username không được trùng nhau</small>
    </div>

    <!-- form-group 2: Email -->
    <div class="form-group">
      <label for="email">Email</label>
      <input type="email" id="email" name="email"
             class="form-control" placeholder="Nhập email">
      <small class="hint error">Email phải hợp lệ</small>
    </div>

    <!-- form-group 3: Password -->
    <div class="form-group">
      <label for="password">Mật khẩu</label>
      <input type="password" id="password" name="password"
             class="form-control">
    </div>

    <!-- form-group 4: Confirm Password -->
    <div class="form-group">
      <label for="confirm-password">Xác nhận mật khẩu</label>
      <input type="password" id="confirm-password" name="confirm_password"
             class="form-control">
    </div>

    <button type="submit" class="btn btn-primary">Đăng Ký</button>

    <div class="links">
      <a href="/forgot-password" class="link">Quên mật khẩu?</a>
      <a href="/login" class="link">Đăng nhập</a>
    </div>
  </form>
</div>
```

---

## 3. Basic Selectors

### 3.1 Type Selector - `element`

**Khái niệm**: Chọn tất cả elements theo tên tag HTML.

**CSS**: `a {}` → chọn tất cả thẻ `<a>`

```html
<!-- Infographic: a  b  c  → chỉ a được chọn -->
<a/>  ← ✅ selected
<b/>
<c/>
```

**Playwright example**:
```typescript
const BASE_URL = 'https://material.playwrightvn.com/14-css-selector.html';

test('Type Selector - chọn theo tag name', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tìm tất cả input trên trang
    const inputCount = await page.locator('input').count();
    console.log(`Có ${inputCount} input trên trang`); // 4 inputs

    // Click vào button (tag = button)
    await page.locator('button').click();

    // Lấy text của heading h2
    const heading = await page.locator('h2').textContent();
    console.log(heading); // "Đăng Ký"

    // Điền vào input đầu tiên
    await page.locator('input').first().fill('testuser');

    // Lấy tất cả link trên trang
    const links = page.locator('a');
    console.log(`Số link: ${await links.count()}`); // 2
});
```

---

### 3.2 ID Selector - `#id`

**Khái niệm**: Chọn element theo thuộc tính `id`. **ID là duy nhất** trên một trang.

**CSS**: `#a {}` → chọn element có `id="a"`

> ⚠️ **Lưu ý từ cheatsheet**: ID không được khuyến khích dùng làm selector vì nó quá cứng nhắc và dễ thay đổi.

```html
<!-- Infographic: #a  #b  #c  → chỉ #a được chọn -->
<div id="a"/>  ← ✅ selected
<div id="b"/>
<div id="c"/>
```

**Playwright example**:
```typescript
test('ID Selector - chọn theo id attribute', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tìm input theo id
    await page.locator('#username').fill('testuser123');
    await page.locator('#email').fill('test@example.com');
    await page.locator('#password').fill('password123');
    await page.locator('#confirm-password').fill('password123');

    // Tìm form theo id
    const form = page.locator('#signup-form');
    await form.waitFor({ state: 'visible' });

    // Verify element tồn tại
    await expect(page.locator('#username')).toBeVisible();
});
```

---

### 3.3 Class Selector - `.class`

**Khái niệm**: Chọn tất cả elements có `class` tương ứng. Nhiều elements có thể cùng class.

**CSS**: `.a {}` → chọn element có `class="a"`

```html
<!-- Infographic: .a  .b  .c  → chỉ .a được chọn -->
<div class="a"/>  ← ✅ selected
<div class="b"/>
<div class="c"/>
```

**Playwright example**:
```typescript
test('Class Selector - chọn theo class name', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tìm tất cả input có class="form-control"
    const inputs = page.locator('.form-control');
    console.log(`Số input.form-control: ${await inputs.count()}`); // 4

    // Điền vào input đầu tiên có class form-control
    await page.locator('.form-control').first().fill('testuser');

    // Click button có class btn-primary
    await page.locator('.btn-primary').click();

    // Tìm link có class "link"
    const links = page.locator('.link');
    await links.first().click(); // Click "Quên mật khẩu?"

    // Tìm element có class "hint"
    const hints = page.locator('.hint');
    console.log(`Số hint: ${await hints.count()}`); // 2
});
```

---

### 3.4 Universal Selector - `*`

**Khái niệm**: Chọn **tất cả** elements trên trang (hoặc trong phạm vi).

**CSS**: `* {}` → chọn mọi element

```html
<!-- Infographic: a  b  c  d  → TẤT CẢ được chọn -->
<a/>  ← ✅
<b/>  ← ✅
<c/>  ← ✅
<d/>  ← ✅
```

**Playwright example**:
```typescript
test('Universal Selector - chọn tất cả elements', async ({ page }) => {
    await page.goto(BASE_URL);

    // Đếm tổng số elements trong form
    const allInForm = await page.locator('#signup-form *').count();
    console.log(`Tổng elements trong form: ${allInForm}`);

    // Đếm tất cả elements trên trang (ít dùng trong test)
    const allElements = await page.locator('*').count();
    console.log(`Tổng elements trên trang: ${allElements}`);
});
```

> ⚠️ `*` một mình ít dùng trong testing vì quá rộng. Thường kết hợp với selector khác.

---

## 4. Combination Selectors

### 4.1 Descendant Selector - `khoảng trắng`

**Khái niệm**: Chọn element **con cháu** (bất kỳ cấp nào) của element cha.

**CSS**: `a b {}` → chọn tất cả `b` nằm BÊN TRONG `a` (dù lồng bao nhiêu cấp)

```html
<!-- Infographic: a b được chọn dù ở cấp nào -->
<a>             ← cha
  <b/>          ← ✅ (con trực tiếp)
  <c>
    <b/>        ← ✅ (con của con = cháu)
  </c>
</a>
<b/>            ← ❌ (không nằm trong a)
```

**Playwright example**:
```typescript
test('Descendant Selector - con/cháu bất kỳ cấp', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tìm input nằm BẤT KỲ đâu trong form (con hoặc cháu)
    await page.locator('form input').first().fill('testuser');
    await page.locator('form input').nth(1).fill('test@example.com');

    // Tìm button nằm trong form
    await page.locator('form button').click();

    // Tìm link nằm trong div.links (trong form)
    const links = page.locator('.links a');
    await links.first().click(); // "Quên mật khẩu?"
    await links.last().click();  // "Đăng nhập"

    // Tìm label nằm trong div.form-group
    const labels = page.locator('.form-group label');
    console.log(`Số labels: ${await labels.count()}`); // 4

    // Tìm input trong div.container (dù lồng nhiều cấp)
    await page.locator('.container input').nth(2).fill('password123');
});
```

---

### 4.2 Child Selector - `>`

**Khái niệm**: Chọn element là **con TRỰC TIẾP** (chỉ 1 cấp) của element cha.

**CSS**: `a > b {}` → chọn `b` là con trực tiếp của `a`

```html
<!-- Infographic: chỉ b trực tiếp trong a được chọn -->
<a>
  <b/>     ← ✅ (con trực tiếp)
  <c>
    <b/>   ← ❌ (cháu, không phải con trực tiếp)
  </c>
</a>
```

**Playwright example**:
```typescript
test('Child Selector > - chỉ con trực tiếp', async ({ page }) => {
    await page.goto(BASE_URL);

    // input là con TRỰC TIẾP của .form-group
    // (không lấy input trong nested div bên trong form-group)
    await page.locator('.form-group > input').first().fill('testuser');
    await page.locator('.form-group > input').nth(1).fill('test@example.com');

    // button là con trực tiếp của form
    await page.locator('form > button').click();

    // div.links là con trực tiếp của form
    const linksDiv = page.locator('form > .links');
    await linksDiv.waitFor({ state: 'visible' });

    // label là con trực tiếp của .form-group
    const labels = page.locator('.form-group > label');
    console.log(`Số labels trực tiếp: ${await labels.count()}`); // 4

    // So sánh: > vs space
    // form > input  → chỉ input là CON TRỰC TIẾP của form (0 vì input nằm trong .form-group)
    // form input    → TẤT CẢ input trong form (4)
    const directInputs = await page.locator('form > input').count(); // 0
    const allInputs = await page.locator('form input').count();      // 4
    console.log(`Direct: ${directInputs}, All: ${allInputs}`);
});
```

---

### 4.3 Adjacent Sibling Selector - `+`

**Khái niệm**: Chọn element **ngay liền kề** sau một element (cùng cấp, ngay sát nhau).

**CSS**: `a + b {}` → chọn `b` đầu tiên ngay SAU `a` (cùng cấp)

```html
<!-- Infographic: a  b  b  b  → chỉ b đầu tiên ngay sau a được chọn -->
<a/>   ← trigger
<b/>   ← ✅ (ngay sau a)
<b/>   ← ❌ (không ngay sau a)
<b/>   ← ❌
```

**Playwright example**:
```typescript
test('Adjacent Sibling + - phần tử ngay liền kề', async ({ page }) => {
    await page.goto(BASE_URL);

    // <small> ngay sau <input> (cùng cấp trong .form-group)
    // Lấy hint text ngay sau username input
    const hintAfterUsername = page.locator('#username + small');
    const hintText = await hintAfterUsername.textContent();
    console.log(hintText); // "Username không được trùng nhau"

    // input ngay sau label
    const inputAfterLabel = page.locator('label + input');
    await inputAfterLabel.first().fill('testuser'); // input sau label "Tên đăng nhập"

    // Verify hint hiển thị sau email input
    const emailHint = page.locator('#email + small');
    await expect(emailHint).toBeVisible();
    console.log(await emailHint.textContent()); // "Email phải hợp lệ"

    // .form-group ngay sau .form-group (form-group thứ 2)
    const secondGroup = page.locator('.form-group + .form-group');
    await secondGroup.locator('input').fill('test@example.com');
});
```

---

### 4.4 General Sibling Selector - `~`

**Khái niệm**: Chọn **tất cả** elements cùng cấp đứng SAU một element (không cần ngay liền kề).

**CSS**: `a ~ b {}` → chọn tất cả `b` đứng SAU `a` cùng cấp

```html
<!-- Infographic: a  b  b  c  b  → tất cả b sau a được chọn -->
<a/>   ← trigger
<b/>   ← ✅
<b/>   ← ✅
<c/>
<b/>   ← ✅ (không liền kề nhưng vẫn sau a)
```

**Playwright example**:
```typescript
test('General Sibling ~ - tất cả siblings phía sau', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tất cả .form-group sau .form-group đầu tiên
    // (form-group 2, 3, 4)
    const groupsAfterFirst = page.locator('.form-group ~ .form-group');
    console.log(`Số groups sau first: ${await groupsAfterFirst.count()}`); // 3

    // Tất cả small sau input (trong cùng .form-group)
    const hintsAfterInput = page.locator('input ~ small');
    console.log(`Số hints: ${await hintsAfterInput.count()}`); // 2

    // Tất cả a (link) sau h2 trong form
    const linksAfterHeading = page.locator('h2 ~ .links');
    await linksAfterHeading.waitFor({ state: 'visible' });

    // Điền vào tất cả input trong các form-group sau form-group đầu
    const inputsInSubsequentGroups = page.locator('.form-group ~ .form-group input');
    await inputsInSubsequentGroups.nth(0).fill('test@example.com'); // email
    await inputsInSubsequentGroups.nth(1).fill('password123');       // password
    await inputsInSubsequentGroups.nth(2).fill('password123');       // confirm
});
```

---

### 4.5 Comma Combinator - `,`

**Khái niệm**: Chọn nhiều selectors cùng lúc (OR logic).

**CSS**: `a, c {}` → chọn cả `a` VÀ `c`

```html
<!-- Infographic: a  b  c  d  → a và c được chọn -->
<a/>  ← ✅
<b/>  ← ❌
<c/>  ← ✅
<d/>  ← ❌
```

**Playwright example**:
```typescript
test('Comma Combinator - chọn nhiều selectors', async ({ page }) => {
    await page.goto(BASE_URL);

    // Đếm tổng số input VÀ button
    const inputsAndButtons = page.locator('input, button');
    console.log(`Inputs + Buttons: ${await inputsAndButtons.count()}`); // 5

    // Tìm cả username field VÀ email field
    const usernameOrEmail = page.locator('#username, #email');
    console.log(`Count: ${await usernameOrEmail.count()}`); // 2

    // Tất cả password fields (cả 2 loại)
    const allPasswords = page.locator('#password, #confirm-password');
    await allPasswords.nth(0).fill('password123');
    await allPasswords.nth(1).fill('password123');

    // Chọn cả link "Quên mật khẩu?" và "Đăng nhập"
    const allLinks = page.locator('a.link, .link');
    console.log(`Links: ${await allLinks.count()}`);

    // Highlight (check visible) cho input, button, link
    const interactables = page.locator('input, button, a');
    console.log(`Interactable elements: ${await interactables.count()}`);
});
```

---

### 4.6 Combine Class Selector - `element.class`

**Khái niệm**: Chọn element vừa là tag đó vừa có class đó.

**CSS**: `b.x {}` → chọn `<b class="x">` (phải là tag `b` VÀ có class `x`)

```html
<!-- Infographic: a.x  b.x  c.x → chỉ b.x được chọn -->
<a class="x"/>  ← ❌ (đúng class nhưng sai tag)
<b class="x"/>  ← ✅ (đúng tag VÀ đúng class)
<c class="x"/>  ← ❌ (đúng class nhưng sai tag)
```

**Playwright example**:
```typescript
test('Combine Class Selector - element.class', async ({ page }) => {
    await page.goto(BASE_URL);

    // input.form-control (phải là thẻ input VÀ có class form-control)
    // Khác với .form-control (bất kỳ tag nào có class đó)
    await page.locator('input.form-control').first().fill('testuser');
    await page.locator('input.form-control').nth(1).fill('test@example.com');

    // button.btn-primary (phải là button VÀ có class btn-primary)
    await page.locator('button.btn-primary').click();

    // a.link (phải là thẻ a VÀ có class link)
    await page.locator('a.link').first().click();   // "Quên mật khẩu?"
    await page.locator('a.link').last().click();    // "Đăng nhập"

    // So sánh:
    // button.btn-primary → chỉ button có class btn-primary
    // .btn-primary       → bất kỳ element nào có class btn-primary
    const byTagAndClass = await page.locator('button.btn-primary').count(); // 1
    const byClassOnly = await page.locator('.btn-primary').count();         // 1 (hoặc nhiều hơn)
    console.log(`Tag+Class: ${byTagAndClass}, ClassOnly: ${byClassOnly}`);

    // div.form-group (chỉ div với class form-group)
    const formGroups = page.locator('div.form-group');
    console.log(`Form groups: ${await formGroups.count()}`); // 4
});
```

---

### 4.7 Combine Descendant & ID - `#id descendant`

**Khái niệm**: Kết hợp ID và Descendant - tìm element con/cháu bên trong element có ID cụ thể.

**CSS**: `#a b {}` → chọn tất cả `b` bên trong element có `id="a"`

```html
<!-- Infographic: chỉ b bên trong #a được chọn -->
<div id="a">       ← #a
  <b/>             ← ✅
  <c>
    <b/>           ← ✅ (cháu cũng được)
  </c>
</div>
<b/>               ← ❌ (không trong #a)
```

**Playwright example**:
```typescript
test('Combine Descendant & ID - #id descendant', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tìm tất cả input trong form có id="signup-form"
    await page.locator('#signup-form input').first().fill('testuser');
    await page.locator('#signup-form input').nth(1).fill('test@example.com');
    await page.locator('#signup-form input').nth(2).fill('password123');
    await page.locator('#signup-form input').nth(3).fill('password123');

    // Tìm button trong signup-form
    await page.locator('#signup-form button').click();

    // Tìm label trong signup-form
    const labels = page.locator('#signup-form label');
    console.log(`Labels: ${await labels.count()}`); // 4
    console.log(await labels.first().textContent()); // "Tên đăng nhập"

    // Tìm link trong signup-form
    const links = page.locator('#signup-form a');
    console.log(`Links: ${await links.count()}`); // 2

    // Tìm .form-group bên trong #signup-form
    const groups = page.locator('#signup-form .form-group');
    console.log(`Groups: ${await groups.count()}`); // 4
});
```

---

### 4.8 Combine Universal Selector - `element *`

**Khái niệm**: Chọn **tất cả** elements con/cháu bên trong một element cụ thể.

**CSS**: `a * {}` → chọn tất cả elements bất kỳ bên trong `a`

```html
<!-- Infographic: tất cả con/cháu của a được chọn -->
<a>
  <b/>   ← ✅
  <c>
    <b/> ← ✅
  </c>
</a>
```

**Playwright example**:
```typescript
test('Combine Universal Selector - element *', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tất cả elements bên trong form
    const allInForm = page.locator('form *');
    console.log(`Tất cả trong form: ${await allInForm.count()}`);

    // Tất cả elements trong .form-group đầu tiên
    const allInFirstGroup = page.locator('.form-group:first-child *');
    console.log(`Trong group đầu: ${await allInFirstGroup.count()}`); // label + input + small = 3

    // Tất cả elements trong .links
    const allInLinks = page.locator('.links *');
    console.log(`Trong links: ${await allInLinks.count()}`); // 2 links

    // Thường dùng để đếm/kiểm tra structure
    await expect(page.locator('#signup-form *')).not.toHaveCount(0);
});
```

---

## 5. Pseudo-class Selectors - Structural

### 5.1 :first-child

**Khái niệm**: Chọn element là **con đầu tiên** của parent.

**CSS**: `b:first-child {}` → chọn `b` nếu nó là con ĐẦU TIÊN của parent

```html
<!-- Case 1: b b b → b đầu tiên được chọn ✅ -->
<b/>  ← ✅ first-child
<b/>
<b/>

<!-- Case 2: a b b → b KHÔNG được chọn ❌ -->
<!-- vì first-child là "a", không phải "b" -->
<a/>  ← first-child (nhưng không phải b)
<b/>  ← ❌ không phải first-child
<b/>  ← ❌
```

> ⚠️ **Lưu ý quan trọng**: `b:first-child` = "b phải LÀ first-child", không phải "b đầu tiên trong list b".

**Playwright example**:
```typescript
test(':first-child - con đầu tiên của parent', async ({ page }) => {
    await page.goto(BASE_URL);

    // .form-group:first-child → div.form-group đầu tiên trong form
    const firstGroup = page.locator('.form-group:first-child');
    await firstGroup.locator('input').fill('testuser');

    // input trong .form-group đầu tiên (username field)
    await page.locator('.form-group:first-child input').fill('testuser');

    // label đầu tiên trong form-group đầu tiên
    const firstLabel = page.locator('.form-group:first-child label');
    console.log(await firstLabel.textContent()); // "Tên đăng nhập"

    // Kiểm tra form-group đầu tiên visible
    await expect(page.locator('.form-group:first-child')).toBeVisible();

    // small là con thứ mấy? Nếu small là con đầu tiên trong parent
    // (trong case này small là con thứ 3, không phải first-child)
});
```

---

### 5.2 :last-child

**Khái niệm**: Chọn element là **con cuối cùng** của parent.

**CSS**: `b:last-child {}` → chọn `b` nếu nó là con CUỐI CÙNG

```html
<!-- Case 1: b b b → b cuối được chọn ✅ -->
<b/>
<b/>
<b/>  ← ✅ last-child

<!-- Case 2: b b c → b KHÔNG được chọn ❌ -->
<!-- vì last-child là "c", không phải "b" -->
<b/>
<b/>
<c/>  ← last-child (nhưng không phải b)
```

**Playwright example**:
```typescript
test(':last-child - con cuối cùng của parent', async ({ page }) => {
    await page.goto(BASE_URL);

    // .form-group:last-child → form-group cuối (confirm password)
    const lastGroup = page.locator('.form-group:last-child');
    await lastGroup.locator('input').fill('password123');

    // input trong form-group cuối (confirm password)
    await page.locator('.form-group:last-child input').fill('password123');

    // Lấy label của form-group cuối
    const lastLabel = page.locator('.form-group:last-child label');
    console.log(await lastLabel.textContent()); // "Xác nhận mật khẩu"

    // Link cuối trong .links
    const lastLink = page.locator('.links a:last-child');
    console.log(await lastLink.textContent()); // "Đăng nhập"
    await lastLink.click();
});
```

---

### 5.3 :nth-child(n)

**Khái niệm**: Chọn element là con thứ **n** của parent (đếm từ 1).

**CSS**: `b:nth-child(2) {}` → chọn `b` nếu nó là con thứ 2

```
n có thể là:
- số cụ thể: nth-child(1), nth-child(2), nth-child(3)...
- even: nth-child(even) → 2, 4, 6, 8...
- odd: nth-child(odd) → 1, 3, 5, 7...
- công thức: nth-child(2n+1) → 1, 3, 5... (= odd)
             nth-child(3n) → 3, 6, 9...
             nth-child(n+3) → từ 3 trở đi
```

> ⚠️ **Lưu ý**: `b:nth-child(2)` = "phần tử thứ 2 trong parent VÀ phải là tag b". Nếu phần tử thứ 2 là tag khác → không match!

**Playwright example**:
```typescript
test(':nth-child(n) - con thứ n', async ({ page }) => {
    await page.goto(BASE_URL);

    // form-group thứ 1 (username)
    await page.locator('.form-group:nth-child(1) input').fill('testuser');

    // form-group thứ 2 (email)
    await page.locator('.form-group:nth-child(2) input').fill('test@example.com');

    // form-group thứ 3 (password)
    await page.locator('.form-group:nth-child(3) input').fill('password123');

    // form-group thứ 4 (confirm password)
    await page.locator('.form-group:nth-child(4) input').fill('password123');

    // form-group chẵn (2, 4) → email và confirm password
    const evenGroups = page.locator('.form-group:nth-child(even)');
    console.log(`Even groups: ${await evenGroups.count()}`); // 2

    // form-group lẻ (1, 3) → username và password
    const oddGroups = page.locator('.form-group:nth-child(odd)');
    console.log(`Odd groups: ${await oddGroups.count()}`); // 2

    // Link thứ 1 trong .links
    const firstLink = page.locator('.links a:nth-child(1)');
    console.log(await firstLink.textContent()); // "Quên mật khẩu?"

    // Link thứ 2 trong .links
    const secondLink = page.locator('.links a:nth-child(2)');
    console.log(await secondLink.textContent()); // "Đăng nhập"
});
```

---

### 5.4 :nth-last-child(n)

**Khái niệm**: Giống `:nth-child` nhưng **đếm ngược từ cuối**.

**CSS**: `c:nth-last-child(2) {}` → chọn `c` là con thứ 2 từ cuối

```html
<!-- Đếm ngược: 4  3  2  1 -->
<a>
  <a/>  ← vị trí 4 từ cuối
  <b/>  ← vị trí 3 từ cuối
  <c/>  ← vị trí 2 từ cuối ✅ (nếu dùng c:nth-last-child(2))
  <d/>  ← vị trí 1 từ cuối (last-child)
</a>
```

**Playwright example**:
```typescript
test(':nth-last-child(n) - con thứ n từ cuối', async ({ page }) => {
    await page.goto(BASE_URL);

    // form-group thứ 1 từ cuối (= form-group cuối = confirm password)
    await page.locator('.form-group:nth-last-child(1) input').fill('password123');

    // form-group thứ 2 từ cuối (= form-group thứ 3 = password)
    await page.locator('.form-group:nth-last-child(2) input').fill('password123');

    // form-group thứ 3 từ cuối (= form-group thứ 2 = email)
    await page.locator('.form-group:nth-last-child(3) input').fill('test@example.com');

    // form-group thứ 4 từ cuối (= form-group đầu = username)
    await page.locator('.form-group:nth-last-child(4) input').fill('testuser');

    // Link thứ 1 từ cuối trong .links
    const lastLink = page.locator('.links a:nth-last-child(1)');
    console.log(await lastLink.textContent()); // "Đăng nhập"
});
```

---

### 5.5 :only-child

**Khái niệm**: Chọn element là **con DUY NHẤT** của parent (parent chỉ có đúng 1 con).

**CSS**: `b:only-child {}` → chọn `b` nếu parent chỉ có duy nhất `b` là con

```html
<!-- ✅ b là only-child -->
<a>
  <b/>   ← ✅ (a chỉ có 1 con duy nhất)
</a>

<!-- ❌ b KHÔNG phải only-child -->
<a>
  <b/>   ← ❌ (a có 2 con)
  <b/>   ← ❌
</a>
```

**Playwright example**:
```typescript
test(':only-child - con duy nhất', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tìm element là con duy nhất trong parent
    // (phụ thuộc HTML thực tế - đây là ví dụ tổng quát)
    const onlyChildren = page.locator(':only-child');
    console.log(`Only children: ${await onlyChildren.count()}`);

    // Nếu form-group có 1 input duy nhất (không có small hint)
    // thì input đó là only-child
    const onlyInputInGroup = page.locator('.form-group input:only-child');
    console.log(`Input is only child: ${await onlyInputInGroup.count()}`);

    // Verify một element cụ thể là only-child
    // (ví dụ: container chỉ có form là con duy nhất)
    const onlyForm = page.locator('.container > form:only-child');
    if (await onlyForm.count() > 0) {
        await expect(onlyForm).toBeVisible();
    }
});
```

---

### 5.6 :first-of-type

**Khái niệm**: Chọn element là **phần tử đầu tiên của loại đó** trong parent (không quan tâm vị trí).

**CSS**: `b:first-of-type {}` → chọn `b` đầu tiên (theo loại tag) trong parent

```html
<!-- a  b  a  b  → b đầu tiên được chọn (bất kể a ở đâu) -->
<a/>   ← first-of-type "a"
<b/>   ← ✅ first-of-type "b"
<a/>   ← 2nd "a"
<b/>   ← 2nd "b" (không được chọn)
```

**Sự khác nhau với :first-child**:
- `:first-child` → phải là con ĐẦU TIÊN trong parent (đếm tất cả types)
- `:first-of-type` → phải là element ĐẦU TIÊN của TYPE đó (bỏ qua các type khác)

**Playwright example**:
```typescript
test(':first-of-type - phần tử đầu tiên của loại', async ({ page }) => {
    await page.goto(BASE_URL);

    // input đầu tiên theo loại trong form (= username)
    await page.locator('input:first-of-type').fill('testuser');

    // a (link) đầu tiên theo loại trong .links
    const firstLink = page.locator('.links a:first-of-type');
    console.log(await firstLink.textContent()); // "Quên mật khẩu?"
    await firstLink.click();

    // label đầu tiên theo loại trong form
    const firstLabel = page.locator('form label:first-of-type');
    console.log(await firstLabel.textContent()); // "Tên đăng nhập"

    // small đầu tiên theo loại
    const firstHint = page.locator('small:first-of-type');
    console.log(await firstHint.textContent()); // "Username không được trùng nhau"

    // button đầu tiên theo loại trong form
    const firstButton = page.locator('form button:first-of-type');
    await firstButton.click();
});
```

---

### 5.7 :last-of-type

**Khái niệm**: Chọn element là **phần tử cuối cùng của loại đó** trong parent.

**CSS**: `b:last-of-type {}` → chọn `b` cuối cùng (theo loại) trong parent

**Playwright example**:
```typescript
test(':last-of-type - phần tử cuối của loại', async ({ page }) => {
    await page.goto(BASE_URL);

    // input cuối cùng theo loại trong form (= confirm password)
    await page.locator('input:last-of-type').fill('password123');

    // a (link) cuối cùng trong .links
    const lastLink = page.locator('.links a:last-of-type');
    console.log(await lastLink.textContent()); // "Đăng nhập"
    await lastLink.click();

    // label cuối cùng trong form
    const lastLabel = page.locator('form label:last-of-type');
    console.log(await lastLabel.textContent()); // "Xác nhận mật khẩu"

    // small cuối cùng theo loại
    const lastHint = page.locator('small:last-of-type');
    console.log(await lastHint.textContent()); // "Email phải hợp lệ"

    // Dùng :last-of-type trong context cụ thể
    const lastInputInForm = page.locator('#signup-form input:last-of-type');
    await lastInputInForm.fill('password123');
});
```

---

### 5.8 :nth-of-type(n)

**Khái niệm**: Chọn element là phần tử thứ **n của loại đó** trong parent.

```
n có thể là:
- số: nth-of-type(1), nth-of-type(2)...
- even: nth-of-type(even) → 2, 4, 6...
- odd: nth-of-type(odd) → 1, 3, 5...
- công thức: nth-of-type(2n+1), nth-of-type(3n)...
```

```html
<!-- a  b  a  b  a  b  a (đếm riêng từng loại) -->
<!-- a: 1  2  3  4  → a:nth-of-type(2) = a thứ 2 -->
<!-- b: 1  2  3    → b:nth-of-type(2) = b thứ 2 -->
```

**Playwright example**:
```typescript
test(':nth-of-type(n) - phần tử thứ n của loại', async ({ page }) => {
    await page.goto(BASE_URL);

    // input thứ 1 theo loại (= username)
    await page.locator('input:nth-of-type(1)').fill('testuser');

    // input thứ 2 theo loại (= email)
    await page.locator('input:nth-of-type(2)').fill('test@example.com');

    // input thứ 3 theo loại (= password)
    await page.locator('input:nth-of-type(3)').fill('password123');

    // input thứ 4 theo loại (= confirm password)
    await page.locator('input:nth-of-type(4)').fill('password123');

    // input chẵn (2, 4) = email và confirm password
    const evenInputs = page.locator('input:nth-of-type(even)');
    await evenInputs.nth(0).fill('test@example.com');
    await evenInputs.nth(1).fill('password123');

    // input lẻ (1, 3) = username và password
    const oddInputs = page.locator('input:nth-of-type(odd)');
    await oddInputs.nth(0).fill('testuser');
    await oddInputs.nth(1).fill('password123');

    // link thứ 1 trong .links
    await page.locator('.links a:nth-of-type(1)').click();
    // link thứ 2 trong .links
    await page.locator('.links a:nth-of-type(2)').click();

    // Dùng công thức 2n+1 (= odd)
    const formulaInputs = page.locator('input:nth-of-type(2n+1)');
    console.log(`Formula inputs: ${await formulaInputs.count()}`); // 2
});
```

---

### 5.9 :only-of-type

**Khái niệm**: Chọn element là **loại duy nhất** của tag đó trong parent.

**CSS**: `b:only-of-type {}` → chọn `b` nếu trong parent chỉ có đúng 1 thẻ `b`

```html
<!-- ✅ b là only-of-type (chỉ 1 b trong parent) -->
<a>
  <b/>          ← ✅ (chỉ có 1 b, dù có a và c)
  <a/><c/>
</a>

<!-- ❌ b KHÔNG phải only-of-type (có 2 b) -->
<a>
  <b/><b/>      ← ❌ (có 2 b)
</a>
```

**Playwright example**:
```typescript
test(':only-of-type - loại duy nhất trong parent', async ({ page }) => {
    await page.goto(BASE_URL);

    // button là only-of-type trong form (nếu form chỉ có 1 button)
    const onlyButton = page.locator('form button:only-of-type');
    if (await onlyButton.count() > 0) {
        await onlyButton.click();
    }

    // h2 là only-of-type trong form (thường chỉ có 1 heading)
    const onlyHeading = page.locator('form h2:only-of-type');
    if (await onlyHeading.count() > 0) {
        console.log(await onlyHeading.textContent()); // "Đăng Ký"
    }

    // Tìm tag duy nhất của loại đó
    const onlyOfType = page.locator(':only-of-type');
    console.log(`Only-of-type elements: ${await onlyOfType.count()}`);
});
```

---

## 6. Pseudo-class Selectors - Other

### 6.1 :empty

**Khái niệm**: Chọn element **không có nội dung** (không có children elements hoặc text).

**CSS**: `a:empty {}` → chọn `a` không có gì bên trong

```html
<a/>           ← ✅ empty (không có children/text)
<a>hello</a>   ← ❌ có text "hello"
<a>
  <b/><b/>     ← ❌ có children
</a>
```

**Playwright example**:
```typescript
test(':empty - element không có nội dung', async ({ page }) => {
    await page.goto(BASE_URL);

    // Input chưa điền = empty (input tự thân là self-closing, luôn "empty" theo CSS)
    // Thực tế dùng :empty cho div, span, p rỗng nhiều hơn

    // Đếm các elements rỗng
    const emptyElements = page.locator(':empty');
    console.log(`Empty elements: ${await emptyElements.count()}`);

    // Tìm div rỗng (nếu có)
    const emptyDivs = page.locator('div:empty');
    console.log(`Empty divs: ${await emptyDivs.count()}`);

    // Thực tế hơn: kiểm tra input value rỗng
    // (dùng :empty không hợp, dùng :placeholder-shown thay thế)
    const emptyInputs = page.locator('input:placeholder-shown');
    console.log(`Inputs chưa điền: ${await emptyInputs.count()}`);
});
```

---

### 6.2 :not()

**Khái niệm**: Chọn elements **KHÔNG khớp** với selector bên trong. (Phủ định)

**CSS**: `a:not(.x) {}` → chọn tất cả `a` mà KHÔNG có class `x`

```html
<!-- a:not(.x) → chọn a và b nhưng không chọn a.x và b.x -->
<a/>            ← ✅ (không có class x)
<b/>            ← ❌ (không phải a)
<a class="x"/>  ← ❌ (có class x)
<b class="x"/>  ← ❌
```

**Playwright example**:
```typescript
test(':not() - loại trừ elements', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tất cả input KHÔNG phải type="password"
    const nonPasswordInputs = page.locator('input:not([type="password"])');
    console.log(`Non-password inputs: ${await nonPasswordInputs.count()}`); // 2 (text + email)

    await page.locator('input:not([type="password"])').nth(0).fill('testuser');
    await page.locator('input:not([type="password"])').nth(1).fill('test@example.com');

    // Tất cả input KHÔNG phải type="text"
    const nonTextInputs = page.locator('input:not([type="text"])');
    console.log(`Non-text inputs: ${await nonTextInputs.count()}`); // email + password + password

    // Link KHÔNG phải link cuối cùng
    await page.locator('a:not(:last-of-type)').click(); // "Quên mật khẩu?"

    // .form-group KHÔNG phải group đầu tiên
    const notFirstGroups = page.locator('.form-group:not(:first-child)');
    console.log(`Groups (not first): ${await notFirstGroups.count()}`); // 3

    // button KHÔNG bị disabled
    await page.locator('button:not([disabled])').click();

    // input KHÔNG có class "error"
    const normalInputs = page.locator('input:not(.error)');
    console.log(`Normal inputs: ${await normalInputs.count()}`);

    // Kết hợp nhiều :not()
    // input không phải password VÀ không phải disabled
    const activeNonPasswordInputs = page.locator('input:not([type="password"]):not([disabled])');
    await activeNonPasswordInputs.first().fill('testuser');
});
```

---

## 7. Attribute Selectors

### 7.1 `[attr]` - Có attribute

**Khái niệm**: Chọn elements **có attribute đó** (bất kể giá trị là gì).

**CSS**: `[for]` → chọn tất cả elements có attribute `for`

```html
<a/>              ← ❌ không có attribute "for"
<a for="x"/>      ← ✅ có attribute "for"
<a for="y"/>      ← ✅
<b for="y"/>      ← ✅ (bất kỳ tag nào có "for")
```

**Playwright example**:
```typescript
test('[attr] - có attribute', async ({ page }) => {
    await page.goto(BASE_URL);

    // Tìm tất cả elements có attribute "for" (= label elements)
    const labels = page.locator('[for]');
    console.log(`Elements có [for]: ${await labels.count()}`); // 4 labels

    // Tìm tất cả input có attribute "placeholder"
    const inputsWithPlaceholder = page.locator('input[placeholder]');
    console.log(`Inputs có placeholder: ${await inputsWithPlaceholder.count()}`); // 2 (username + email)

    // Tìm input có attribute "name"
    const inputsWithName = page.locator('input[name]');
    await inputsWithName.nth(0).fill('testuser');
    await inputsWithName.nth(1).fill('test@example.com');

    // Tìm button có attribute "type"
    const buttonWithType = page.locator('button[type]');
    await buttonWithType.click();

    // Tìm link có attribute "href"
    const linksWithHref = page.locator('a[href]');
    console.log(`Links có href: ${await linksWithHref.count()}`); // 2
    await linksWithHref.first().click();

    // Kết hợp: input có cả [type] và [placeholder]
    const inputsWithBoth = page.locator('input[type][placeholder]');
    console.log(`Inputs có cả type và placeholder: ${await inputsWithBoth.count()}`);
});
```

---

### 7.2 `[attr="value"]` - Khớp chính xác

**Khái niệm**: Chọn elements có attribute với **giá trị CHÍNH XÁC**.

**CSS**: `a[for="x"]` → chọn `a` có `for="x"` (đúng chính xác)

```html
<a/>              ← ❌
<a for="x"/>      ← ✅ chính xác "x"
<a for="y"/>      ← ❌ không phải "x"
<a for="z"/>      ← ❌ không phải "x"
```

**Playwright example**:
```typescript
test('[attr="value"] - giá trị chính xác', async ({ page }) => {
    await page.goto(BASE_URL);

    // input có type="text" chính xác
    await page.locator('input[type="text"]').fill('testuser');

    // input có type="email" chính xác
    await page.locator('input[type="email"]').fill('test@example.com');

    // input có type="password" chính xác (lấy đầu tiên)
    await page.locator('input[type="password"]').first().fill('password123');
    await page.locator('input[type="password"]').last().fill('password123');

    // button có type="submit" chính xác
    await page.locator('button[type="submit"]').click();

    // input có name="username" chính xác
    await page.locator('input[name="username"]').fill('testuser');

    // input có name="email"
    await page.locator('input[name="email"]').fill('test@example.com');

    // input có name="password"
    await page.locator('input[name="password"]').fill('password123');

    // input có name="confirm_password"
    await page.locator('input[name="confirm_password"]').fill('password123');

    // label có for="username"
    const usernameLabel = page.locator('label[for="username"]');
    console.log(await usernameLabel.textContent()); // "Tên đăng nhập"

    // Đếm input type="password"
    const passwordCount = await page.locator('input[type="password"]').count();
    console.log(`Password inputs: ${passwordCount}`); // 2
});
```

---

### 7.3 `[attr^="value"]` - Bắt đầu bằng

**Khái niệm**: Chọn elements có attribute **bắt đầu bằng** chuỗi đó.

**CSS**: `[for^="x"]` → chọn elements có attribute `for` bắt đầu bằng "x"

```html
<!-- [for^="x"] → những gì bắt đầu bằng "x" được chọn ✅ -->
<a for="x"/>    ← ✅ "x" bắt đầu bằng "x"
<a for="xy"/>   ← ✅ "xy" bắt đầu bằng "x"
<a for="yz"/>   ← ❌ "yz" không bắt đầu bằng "x"
<a for="zx"/>   ← ❌ "zx" không bắt đầu bằng "x"
```

**Playwright example**:
```typescript
test('[attr^="value"] - bắt đầu bằng', async ({ page }) => {
    await page.goto(BASE_URL);

    // input có name bắt đầu bằng "pass" (= password, không phải confirm_password)
    // Chú ý: "password" bắt đầu bằng "pass", "confirm_password" không bắt đầu bằng "pass"
    await page.locator('input[name^="pass"]').fill('password123');

    // input có name bắt đầu bằng "confirm"
    await page.locator('input[name^="confirm"]').fill('password123');

    // input có id bắt đầu bằng "user"
    await page.locator('input[id^="user"]').fill('testuser');

    // link có href bắt đầu bằng "/forgot"
    const forgotLink = page.locator('a[href^="/forgot"]');
    await forgotLink.click();

    // link có href bắt đầu bằng "/login"
    const loginLink = page.locator('a[href^="/login"]');
    await loginLink.click();

    // input có placeholder bắt đầu bằng "Nhập"
    const inputsWithNhap = page.locator('input[placeholder^="Nhập"]');
    console.log(`Inputs với placeholder "Nhập...": ${await inputsWithNhap.count()}`);
    await inputsWithNhap.nth(0).fill('testuser');
    await inputsWithNhap.nth(1).fill('test@example.com');

    // type bắt đầu bằng "pass" = type="password"
    await page.locator('input[type^="pass"]').first().fill('password123');
});
```

---

### 7.4 `[attr$="value"]` - Kết thúc bằng

**Khái niệm**: Chọn elements có attribute **kết thúc bằng** chuỗi đó.

**CSS**: `[for$="x"]` → chọn elements có `for` kết thúc bằng "x"

```html
<!-- [for$="x"] → những gì kết thúc bằng "x" được chọn ✅ -->
<a for="x"/>    ← ✅ "x" kết thúc bằng "x"
<a for="xy"/>   ← ❌ "xy" kết thúc bằng "y"
<a for="yz"/>   ← ❌
<a for="zx"/>   ← ✅ "zx" kết thúc bằng "x"
```

**Playwright example**:
```typescript
test('[attr$="value"] - kết thúc bằng', async ({ page }) => {
    await page.goto(BASE_URL);

    // input có name kết thúc bằng "word" = "password" (pass-WORD)
    await page.locator('input[name$="word"]').fill('password123');

    // input có name kết thúc bằng "password" = "confirm_password" VÀ "password"
    const endsWithPassword = page.locator('input[name$="password"]');
    console.log(`Ends with "password": ${await endsWithPassword.count()}`); // 2

    // input có id kết thúc bằng "word" = "password" (pass-WORD)
    await page.locator('input[id$="word"]').fill('password123');

    // input có id kết thúc bằng "password" = "confirm-password" VÀ "password"
    const idEndsPassword = page.locator('input[id$="password"]');
    await idEndsPassword.first().fill('password123');
    await idEndsPassword.last().fill('password123');

    // link có href kết thúc bằng "password" = "/forgot-password"
    const forgotPasswordLink = page.locator('a[href$="password"]');
    await forgotPasswordLink.click();

    // href kết thúc bằng "login"
    const loginLink = page.locator('a[href$="login"]');
    await loginLink.click();

    // input có placeholder kết thúc bằng "nhập"
    const endsNhap = page.locator('input[placeholder$="nhập"]');
    console.log(`Ends with "nhập": ${await endsNhap.count()}`);
});
```

---

### 7.5 `[attr*="value"]` - Chứa chuỗi

**Khái niệm**: Chọn elements có attribute **chứa** chuỗi đó (ở bất kỳ vị trí nào).

**CSS**: `[for*="x"]` → chọn elements có `for` chứa "x"

```html
<!-- [for*="x"] → chứa "x" ở bất kỳ đâu ✅ -->
<a for="x"/>    ← ✅ "x" chứa "x"
<a for="xy"/>   ← ✅ "xy" chứa "x"
<a for="yz"/>   ← ❌ "yz" không chứa "x"
<a for="zx"/>   ← ✅ "zx" chứa "x"
```

**Playwright example**:
```typescript
test('[attr*="value"] - chứa chuỗi', async ({ page }) => {
    await page.goto(BASE_URL);

    // input có name chứa "pass" = "password" và "confirm_password"
    const containsPass = page.locator('input[name*="pass"]');
    console.log(`Contains "pass": ${await containsPass.count()}`); // 2
    await containsPass.first().fill('password123');
    await containsPass.last().fill('password123');

    // input có id chứa "password"
    const idContainsPassword = page.locator('input[id*="password"]');
    await idContainsPassword.first().fill('password123');
    await idContainsPassword.last().fill('password123');

    // link có href chứa "login"
    const linkWithLogin = page.locator('a[href*="login"]');
    await linkWithLogin.click(); // "/login" hoặc "/forgot-password-login"

    // href chứa "password"
    const linksAboutPassword = page.locator('a[href*="password"]');
    await linksAboutPassword.click(); // "/forgot-password"

    // input có placeholder chứa "email"
    await page.locator('input[placeholder*="email"]').fill('test@example.com');

    // input có placeholder chứa "tên" (case-sensitive!)
    await page.locator('input[placeholder*="tên"]').fill('testuser');

    // Tìm theo class chứa "form"
    const formElements = page.locator('[class*="form"]');
    console.log(`Chứa class "form": ${await formElements.count()}`);
});
```

---

## 8. only-child vs only-of-type

Đây là điểm HAY NHẦM nhất. Hãy đọc kỹ:

```html
<a>              <!-- Container 1 -->
  <b/>           <!-- b là only-child ✅ và only-of-type ✅ -->
</a>

<a>              <!-- Container 2 -->
  <b/><b/>       <!-- b KHÔNG phải only-child ❌ và KHÔNG phải only-of-type ❌ -->
</a>

<a>              <!-- Container 3 -->
  <a/><b/><c/>   <!-- b là only-of-type ✅ nhưng KHÔNG phải only-child ❌ -->
</a>             <!-- (vì parent có 3 con: a, b, c) -->
```

```typescript
test('only-child vs only-of-type', async ({ page }) => {
    await page.goto(BASE_URL);

    // b:only-child    → b phải là CON DUY NHẤT của parent (parent chỉ có 1 con)
    // b:only-of-type  → b phải là LOẠI DUY NHẤT của tag b trong parent
    //                    (parent có thể có nhiều con khác loại)

    // Ví dụ với form:
    // h2 trong form: h2 là only-of-type (chỉ có 1 h2 trong form)
    //                nhưng KHÔNG phải only-child (form có nhiều con)
    const onlyH2 = page.locator('form h2:only-of-type');
    console.log(`h2 only-of-type: ${await onlyH2.count()}`); // 1

    const onlyChildH2 = page.locator('form h2:only-child');
    console.log(`h2 only-child: ${await onlyChildH2.count()}`); // 0 (form có nhiều con)

    // button trong form
    const onlyButton = page.locator('form button:only-of-type');
    if (await onlyButton.count() > 0) {
        await onlyButton.click(); // button là loại button duy nhất trong form
    }
});
```

---

## 9. nth-child vs nth-of-type

Điểm khác biệt quan trọng:

```html
<div>              <!-- nth-child đếm TẤT CẢ, nth-of-type đếm RIÊNG TỪNG LOẠI -->
  <a/>             <!-- nth-child(1), a:nth-of-type(1) -->
  <b/>             <!-- nth-child(2), b:nth-of-type(1) -->
  <a/>             <!-- nth-child(3), a:nth-of-type(2) -->
  <b/>             <!-- nth-child(4), b:nth-of-type(2) -->
</div>
```

```typescript
// Với HTML trên:
// a:nth-child(2)    → ❌ KHÔNG match! (vì nth-child(2) là <b/>, không phải <a/>)
// a:nth-of-type(2)  → ✅ match! (a thứ 2 theo loại a)

test('nth-child vs nth-of-type', async ({ page }) => {
    await page.goto(BASE_URL);

    // nth-child(n): đếm TẤT CẢ children, rồi kiểm tra có phải tag đó không
    // .form-group:nth-child(2) → form-group thứ 2 (trong TẤT CẢ children của form)
    await page.locator('.form-group:nth-child(2) input').fill('test@example.com');

    // input:nth-child(2) → element thứ 2 trong parent VÀ phải là input
    // (Cẩn thận: nếu element thứ 2 là label, sẽ không match!)

    // nth-of-type(n): đếm RIÊNG theo loại tag
    // input:nth-of-type(2) → input thứ 2 trong số các inputs (bỏ qua label, small...)
    await page.locator('input:nth-of-type(2)').fill('test@example.com');

    // Trong form, nth-of-type an toàn hơn khi các elements cùng loại
    // xen kẽ với elements khác loại
    console.log('nth-of-type thường an toàn hơn nth-child khi có nhiều loại element xen kẽ');
});
```

---

## 10. Cheat Sheet Tổng Hợp

### Bảng tổng hợp toàn bộ CSS Selectors

```typescript
const BASE_URL = 'https://material.playwrightvn.com/14-css-selector.html';

// ===== BASIC SELECTORS =====
page.locator('input')                    // Type: tất cả <input>
page.locator('#username')                // ID: id="username"
page.locator('.form-control')            // Class: class="form-control"
page.locator('*')                        // Universal: tất cả elements

// ===== COMBINATION SELECTORS =====
page.locator('form input')               // Descendant: input bất kỳ trong form
page.locator('form > button')            // Child: button là con TRỰC TIẾP của form
page.locator('label + input')            // Adjacent Sibling: input NGAY SAU label
page.locator('.form-group ~ .form-group')// General Sibling: tất cả form-group sau cái đầu
page.locator('input, button')            // Comma: input HOẶC button
page.locator('input.form-control')       // Combine Class: input có class form-control
page.locator('#signup-form input')       // ID + Descendant: input trong #signup-form
page.locator('form *')                   // Combine Universal: tất cả trong form

// ===== STRUCTURAL PSEUDO-CLASS =====
page.locator('.form-group:first-child')         // Con đầu tiên
page.locator('.form-group:last-child')          // Con cuối cùng
page.locator('.form-group:nth-child(2)')        // Con thứ 2
page.locator('.form-group:nth-child(even)')     // Con chẵn
page.locator('.form-group:nth-child(odd)')      // Con lẻ
page.locator('.form-group:nth-last-child(2)')   // Con thứ 2 từ cuối
page.locator('input:only-child')                // Con duy nhất
page.locator('input:first-of-type')             // Loại đầu tiên
page.locator('input:last-of-type')              // Loại cuối cùng
page.locator('input:nth-of-type(2)')            // Loại thứ 2
page.locator('input:nth-of-type(even)')         // Loại chẵn
page.locator('input:only-of-type')              // Loại duy nhất

// ===== OTHER PSEUDO-CLASS =====
page.locator('div:empty')                       // Element rỗng
page.locator('input:not([type="password"])')    // KHÔNG phải password
page.locator('a:not(:last-of-type)')            // KHÔNG phải link cuối

// ===== ATTRIBUTE SELECTORS =====
page.locator('[placeholder]')                   // Có attribute placeholder
page.locator('input[placeholder]')              // input có placeholder
page.locator('[type="email"]')                  // type chính xác = "email"
page.locator('input[type="email"]')             // input với type="email"
page.locator('input[name^="pass"]')             // name bắt đầu bằng "pass"
page.locator('a[href^="/login"]')               // href bắt đầu bằng "/login"
page.locator('input[name$="password"]')         // name kết thúc bằng "password"
page.locator('a[href$="login"]')                // href kết thúc bằng "login"
page.locator('input[name*="pass"]')             // name chứa "pass"
page.locator('a[href*="password"]')             // href chứa "password"
```

### Quick Reference - Ký hiệu

| Ký hiệu | Tên | Ý nghĩa |
|---------|-----|---------|
| `element` | Type Selector | Chọn theo tag name |
| `#id` | ID Selector | Chọn theo id attribute |
| `.class` | Class Selector | Chọn theo class attribute |
| `*` | Universal Selector | Chọn tất cả |
| `A B` | Descendant | B là con/cháu của A |
| `A > B` | Child | B là con TRỰC TIẾP của A |
| `A + B` | Adjacent Sibling | B ngay SAU A (cùng cấp) |
| `A ~ B` | General Sibling | Tất cả B sau A (cùng cấp) |
| `A, B` | Comma | A HOẶC B |
| `A.class` | Tag + Class | Tag A VÀ có class |
| `#id B` | ID + Descendant | B trong #id |
| `A *` | Tag + Universal | Tất cả trong A |
| `:first-child` | Pseudo | Con đầu tiên |
| `:last-child` | Pseudo | Con cuối cùng |
| `:nth-child(n)` | Pseudo | Con thứ n |
| `:nth-last-child(n)` | Pseudo | Con thứ n từ cuối |
| `:only-child` | Pseudo | Con duy nhất |
| `:first-of-type` | Pseudo | Loại đầu tiên |
| `:last-of-type` | Pseudo | Loại cuối cùng |
| `:nth-of-type(n)` | Pseudo | Loại thứ n |
| `:only-of-type` | Pseudo | Loại duy nhất |
| `:empty` | Pseudo | Không có nội dung |
| `:not(x)` | Pseudo | Không khớp x |
| `[attr]` | Attribute | Có attribute |
| `[attr="v"]` | Attribute | Giá trị chính xác |
| `[attr^="v"]` | Attribute | Bắt đầu bằng |
| `[attr$="v"]` | Attribute | Kết thúc bằng |
| `[attr*="v"]` | Attribute | Chứa chuỗi |

### Full test example hoàn chỉnh

```typescript
import { test, expect } from '@playwright/test';

const URL = 'https://material.playwrightvn.com/14-css-selector.html';

test.describe('CSS Selectors trong Playwright', () => {
    test.beforeEach(async ({ page }) => {
        await page.goto(URL);
    });

    test('Điền form dùng Attribute Selectors', async ({ page }) => {
        // Dùng attribute selector - rõ ràng và semantic
        await page.locator('input[name="username"]').fill('testuser123');
        await page.locator('input[type="email"]').fill('test@example.com');
        await page.locator('input[name="password"]').fill('StrongPass@123');
        await page.locator('input[name="confirm_password"]').fill('StrongPass@123');
        await page.locator('button[type="submit"]').click();
    });

    test('Điền form dùng ID Selectors', async ({ page }) => {
        await page.locator('#username').fill('testuser123');
        await page.locator('#email').fill('test@example.com');
        await page.locator('#password').fill('StrongPass@123');
        await page.locator('#confirm-password').fill('StrongPass@123');
        await page.locator('#signup-form button').click();
    });

    test('Điền form dùng nth-of-type', async ({ page }) => {
        await page.locator('input:nth-of-type(1)').fill('testuser123');
        await page.locator('input:nth-of-type(2)').fill('test@example.com');
        await page.locator('input:nth-of-type(3)').fill('StrongPass@123');
        await page.locator('input:nth-of-type(4)').fill('StrongPass@123');
        await page.locator('button').click();
    });

    test('Kiểm tra cấu trúc form', async ({ page }) => {
        // Đếm số form groups
        expect(await page.locator('.form-group').count()).toBe(4);

        // Kiểm tra có 2 password fields
        expect(await page.locator('input[type="password"]').count()).toBe(2);

        // Kiểm tra có hint text
        const hints = page.locator('.hint');
        expect(await hints.count()).toBeGreaterThan(0);

        // Kiểm tra button submit tồn tại
        await expect(page.locator('button[type="submit"]')).toBeVisible();

        // Kiểm tra links tồn tại
        expect(await page.locator('a[href]').count()).toBe(2);
    });

    test('Navigate dùng links', async ({ page }) => {
        // Click link đầu tiên
        await page.locator('a:first-of-type').click();
        // hoặc
        await page.locator('.links a').first().click();

        // Click link cuối
        await page.locator('a:last-of-type').click();
        // hoặc
        await page.locator('.links a').last().click();
    });
});
```

---

*Tài liệu tổng hợp từ:*
- *CSS Selectors Cheatsheet by nana jeon (Feb 2019) — [PDF](https://appletree.or.kr/quick_reference_cards/CSS/CSS%20selectors%20cheatsheet.pdf)*
- *https://material.playwrightvn.com/14-css-selector.html*
