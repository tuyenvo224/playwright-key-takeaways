# Lesson 7 — XPath Nâng Cao: Advanced Methods, Functions & Axes

> Tổng hợp từ toàn bộ hình ảnh bài giảng Lesson 7 (72 ảnh)  
> Trang thực hành: User Registration Form

---

## Mục Lục

1. [XPath Advanced Methods](#1-xpath-advanced-methods)
   - [Wildcard `*`](#11-wildcard-)
   - [Chứa thuộc tính](#12-chứa-thuộc-tính)
   - [and / or](#13-and--or)
   - [text()](#14-innertext--text)
   - [normalize-space()](#15-normalize-space)
   - [contains()](#16-contains)
   - [starts-with()](#17-starts-with)
   - [not()](#18-not)
2. [XPath Functions — Bảng Hàm Đầy Đủ](#2-xpath-functions--bảng-hàm-đầy-đủ)
3. [XPath Axes — Điều Hướng DOM](#3-xpath-axes--điều-hướng-dom)
4. [Ví Dụ Thực Tế Với HTML Mẫu](#4-ví-dụ-thực-tế-với-html-mẫu)
5. [Ví Dụ Kết Hợp Nhiều Axes](#5-ví-dụ-kết-hợp-nhiều-axes)
6. [Bảng So Sánh Axes Tương Tự](#6-bảng-so-sánh-axes-tương-tự)

---

## HTML Tham Chiếu — User Registration Form

Toàn bộ ví dụ trong phần 1 được thực hành trên form này:

```html
<div class="container" id="ancestor">
  <h1 id="self">User Registration</h1>
  <form id="registrationForm">
    <div class="form-group" id="parent">
      <label for="username" id="preceding">Username:</label>
      <input type="text" id="username" name="username" required>
    </div>
    <div class="form-group" id="child">
      <label for="email">Email:</label>
      <input type="email" id="email" name="email" required>
    </div>
    <div class="form-group">
      <label for="dob">Date of Birth:</label>
      <input type="date" id="dob" name="dob">
    </div>
    <div class="form-group">
      <label for="gender">Gender:</label>
      <!-- radio buttons: Male / Female -->
    </div>
    <!-- ... các field khác: Interests, Country, Biography, Picture ... -->
  </form>
  <table id="userTable">...</table>
  <iframe src="https://www.example.com" title="Example Iframe" width="100%" height="300px">
    #document
      <h1>Example Domain</h1>
  </iframe>
  <dialog id="dialog">This is a dialog</dialog>
</div>
```

---

## 1. XPath Advanced Methods

### 1.1 Wildcard `*`

**Khái niệm:** `*` là ký tự đại diện, khớp với **bất kỳ thẻ HTML nào**.

**Cú pháp:**
```xpath
//*                        ← tất cả element trong document
//*[@id="value"]           ← element bất kỳ có id cụ thể
//*[@class="form-group"]   ← element bất kỳ có class cụ thể
```

**Ví dụ thực tế:**

```xpath
//*[@id = "registrationForm"]
```
→ Tìm thẻ bất kỳ có `id="registrationForm"` — kết quả: `<form id="registrationForm">`

```xpath
//*[@class = "form-group" and @id = "parent"]
```
→ Tìm thẻ bất kỳ có class `form-group` VÀ id `parent` — kết quả: `<div id="parent">`

**Khi nào dùng:**
- Khi không biết hoặc không quan tâm đến tên thẻ
- Khi muốn tìm theo attribute mà không giới hạn loại thẻ

---

### 1.2 Chứa Thuộc Tính

**Khái niệm:** Chọn element dựa trên sự tồn tại hoặc giá trị của **attribute (thuộc tính)**.

**Cú pháp:**
```xpath
//tag[@attribute]                  ← có attribute này (bất kể giá trị)
//tag[@attribute='value']          ← attribute bằng đúng giá trị
//tag[@attribute!='value']         ← attribute khác giá trị
//tag[contains(@attribute,'val')]  ← attribute chứa chuỗi con
```

**Ví dụ thực tế:**

```xpath
//form[@id="registrationForm"]
```
→ Kết quả: `<form id="registrationForm">` — 1 of 1

```xpath
//label[@for="username"]
```
→ `<label for="username" id="preceding">Username:</label>`

```xpath
//input[@type="text"]
```
→ `<input type="text" id="username" name="username">`

```xpath
//label[@for="username"]/parent::div
```
→ Lên cha div: `<div class="form-group" id="parent">`

---

### 1.3 `and` / `or`

**Khái niệm:** Kết hợp nhiều điều kiện trong cùng một XPath expression.

**Cú pháp:**
```xpath
//tag[điều_kiện_1 and điều_kiện_2]   ← PHẢI thỏa cả hai
//tag[điều_kiện_1 or điều_kiện_2]    ← thỏa một trong hai
```

**Ví dụ thực tế:**

```xpath
//*[@class = "form-group" and @id = "parent"]
```
→ Phải có CẢ `class="form-group"` VÀ `id="parent"` — kết quả: 1 phần tử

```xpath
//*[@id = "parent" or @id = "child"]
```
→ Có `id="parent"` HOẶC `id="child"` — kết quả: 2 phần tử (div#parent + div#child)

```xpath
//label[@for = 'dob' or text() = 'Date of Birth:']
```
→ Label có `for="dob"` HOẶC text là "Date of Birth:" — kết quả: 1 phần tử

```xpath
//label[@for = 'dob' and text() = 'Date of Birth:']
```
→ Label có `for="dob"` VÀ text là "Date of Birth:" — kết quả: 1 phần tử (chính xác hơn)

**So sánh:**
```
and → giao (intersection): phần tử phải thỏa TẤT CẢ điều kiện
or  → hợp (union): phần tử thỏa ÍT NHẤT một điều kiện
```

---

### 1.4 `innerText` / `text()`

**Khái niệm:** `text()` trả về nội dung text **trực tiếp** của một element (không bao gồm text của con cháu).

**Cú pháp:**
```xpath
//tag[text() = 'Nội dung chính xác']       ← khớp chính xác 100%
//tag[text() = 'value']                    ← so sánh bằng
//tag/text()                               ← lấy text node
```

**Ví dụ thực tế:**

```xpath
//*[text() = "User Registration"]
```
→ Tìm BẤT KỲ thẻ nào có text chính xác là "User Registration"
- Kết quả **1 of 2**: khớp với `<title>User Registration</title>` (trong head)
- Kết quả **2 of 2**: khớp với `<h1 id="self">User Registration</h1>`

```xpath
//h1[text() = "User Registration"]
```
→ Giới hạn chỉ tìm trong thẻ `<h1>` — kết quả: 1 phần tử chính xác

```xpath
//title[text() = "User Registration"]
```
→ Chỉ tìm trong `<title>` — kết quả: thẻ title trong `<head>`

**Lưu ý:**
- `text()` khớp chính xác, phân biệt chữ hoa/thường và khoảng trắng
- Nếu element có khoảng trắng thừa → dùng `normalize-space()` thay thế

---

### 1.5 `normalize-space()`

**Khái niệm:** Loại bỏ khoảng trắng thừa ở đầu, cuối chuỗi và thu gọn khoảng trắng ở giữa về 1 khoảng.

**Cú pháp:**
```xpath
//tag[normalize-space() = 'Nội dung']
//tag[normalize-space(text()) = 'Nội dung']
//tag[normalize-space(@attribute) = 'value']
```

**Ví dụ thực tế:**

```xpath
//h1[normalize-space() = "User Registration"]
```
→ Dù element có text `"  User Registration  "` (có khoảng trắng thừa) vẫn khớp
- Kết quả: `<h1 id="self">User Registration</h1>` — 1 of 1

**So sánh `text()` vs `normalize-space()`:**

| XPath | Text trong DOM | Kết quả |
|-------|---------------|---------|
| `//h1[text()="User Registration"]` | `"User Registration"` | ✓ Match |
| `//h1[text()="User Registration"]` | `"  User Registration  "` | ✗ Không match |
| `//h1[normalize-space()="User Registration"]` | `"  User Registration  "` | ✓ Match |

**Khi nào dùng:** Khi text của element có thể có khoảng trắng thừa (do HTML formatting, dữ liệu động...).

---

### 1.6 `contains()`

**Khái niệm:** Kiểm tra xem chuỗi có **chứa** một chuỗi con hay không. Rất linh hoạt, không cần khớp toàn bộ.

**Cú pháp:**
```xpath
//tag[contains(@attribute, 'chuỗi con')]
//tag[contains(text(), 'chuỗi con')]
//tag[contains(normalize-space(), 'chuỗi con')]
```

**Ví dụ thực tế:**

```xpath
//h1[contains(text(), "Registration")]
```
→ Tìm `<h1>` có text chứa "Registration" — kết quả: `<h1>User Registration</h1>` ✓

```xpath
//div[contains(@class, "form-group")]
```
→ Tìm div có class **chứa** "form-group" (dù class có thể là "form-group active")

**So sánh `contains()` vs `=`:**

```xpath
//h1[@class = "title"]           ← class phải BẰNG ĐÚNG "title"
//h1[contains(@class, "title")]  ← class CHỨ "title" (có thể có thêm class khác)
```

**Ứng dụng phổ biến:**
```xpath
//div[contains(@class, 'active')]        ← element đang active
//a[contains(@href, 'login')]            ← link có chứa "login" trong URL
//input[contains(@placeholder, 'email')] ← input có placeholder chứa "email"
```

---

### 1.7 `starts-with()`

**Khái niệm:** Kiểm tra xem chuỗi có **bắt đầu bằng** một prefix cụ thể không.

**Cú pháp:**
```xpath
//tag[starts-with(@attribute, 'prefix')]
//tag[starts-with(text(), 'prefix')]
```

**Ví dụ thực tế:**

```xpath
//h1[starts-with(text(), "User")]
```
→ Tìm `<h1>` có text bắt đầu bằng "User" — kết quả: `<h1>User Registration</h1>` ✓

```xpath
//input[starts-with(@id, 'user')]
```
→ Tìm input có id bắt đầu bằng "user" — khớp `id="username"`, `id="userEmail"`, v.v.

**Ứng dụng phổ biến:**
```xpath
//div[starts-with(@id, 'item-')]      ← các div có id như item-1, item-2...
//a[starts-with(@href, 'https')]      ← link HTTPS
//input[starts-with(@name, 'user')]   ← input có name bắt đầu bằng "user"
```

---

### 1.8 `not()`

**Khái niệm:** Phủ định điều kiện — tìm element **KHÔNG** thỏa điều kiện cho trước.

**Cú pháp:**
```xpath
//tag[not(điều_kiện)]
//tag[not(@attribute = 'value')]
//tag[not(contains(text(), 'chuỗi'))]
//tag[not(starts-with(text(), 'prefix'))]
```

**Ví dụ thực tế:**

```xpath
//h1[not(starts-with(text(), "User"))]
```
→ Tìm `<h1>` có text KHÔNG bắt đầu bằng "User"
- Kết quả: tìm thấy `<h1>Example Domain</h1>` (trong iframe của example.com) — 1 of 1

```xpath
//div[not(@class)]
```
→ Tìm div không có attribute `class`

```xpath
//input[not(@type = 'hidden')]
```
→ Tìm tất cả input không phải `type="hidden"`

**Kết hợp `not()` với `contains()`:**
```xpath
//div[not(contains(@class, 'disabled'))]   ← div không bị disabled
//button[not(contains(@class, 'hidden'))]  ← button đang hiển thị
```

---

## 2. XPath Functions — Bảng Hàm Đầy Đủ

### Nhóm String Functions

| Function | Cú pháp | Mô tả | Ví dụ | Kết quả |
|----------|---------|-------|-------|---------|
| `concat()` | `concat(str1, str2, ...)` | Nối các chuỗi lại với nhau | `concat('Hello', ' ', 'World')` | `'Hello World'` |
| `starts-with()` | `starts-with(str, prefix)` | Kiểm tra chuỗi bắt đầu bằng prefix | `//input[starts-with(@id, 'user')]` | Input có id bắt đầu bằng 'user' |
| `contains()` | `contains(str, substring)` | Kiểm tra chuỗi chứa substring | `//div[contains(@class, 'active')]` | Div có class chứa 'active' |
| `ends-with()` | `ends-with(str, suffix)` | Kiểm tra chuỗi kết thúc bằng suffix | `ends-with('hello.txt', '.txt')` | `true` |
| `string-length()` | `string-length(str?)` | Trả về độ dài chuỗi | `string-length('Hello')` | `5` |
| `normalize-space()` | `normalize-space(str?)` | Loại bỏ khoảng trắng thừa | `normalize-space(' Hello World ')` | `'Hello World'` |
| `translate()` | `translate(str, from, to)` | Thay thế từng ký tự trong chuỗi | `translate('abc', 'ab', 'AB')` | `'ABc'` |
| `lower-case()` | `lower-case(string)` | Chuyển thành chữ thường | `lower-case('HELLO')` | `'hello'` |
| `upper-case()` | `upper-case(string)` | Chuyển thành chữ HOA | `upper-case('hello')` | `'HELLO'` |
| `replace()` | `replace(str, pattern, replacement)` | Thay thế theo regex | `replace('hello', 'l', 'L')` | `'heLLo'` |
| `tokenize()` | `tokenize(str, pattern)` | Tách chuỗi theo pattern | `tokenize('a,b,c', ',')` | `('a','b','c')` |

### Ghi Chú Sử Dụng

```xpath
// Tìm kiếm không phân biệt hoa thường (dùng lower-case + contains)
//button[contains(lower-case(text()), 'submit')]

// Tìm input có id bắt đầu bằng 'user' (case-insensitive)
//input[starts-with(lower-case(@id), 'user')]

// Kết hợp normalize-space + contains để tránh lỗi khoảng trắng
//label[contains(normalize-space(), 'Date of Birth')]
```

---

## 3. XPath Axes — Điều Hướng DOM

### Cú Pháp Tổng Quát

```xpath
//tagXuấtPhát[@attr='val']/axis::tagMuốnChọn
```

### Bảng Đầy Đủ 13 Axes

| Axis | Cú pháp | Mô tả | Ví dụ | Kết quả |
|------|---------|-------|-------|---------|
| **child** | `child::node` | Chọn tất cả node con trực tiếp (cấp 1) | `//div/child::p` | Tất cả `<p>` là con trực tiếp của `<div>` |
| **descendant** | `descendant::node` | Chọn tất cả con cháu (mọi cấp sâu) | `//div/descendant::span` | Tất cả `<span>` bên trong `<div>` ở mọi cấp |
| **descendant-or-self** | `descendant-or-self::node` | Chọn chính nó + tất cả con cháu | `//div/descendant-or-self::*` | `<div>` và tất cả node bên trong |
| **parent** | `parent::node` | Chọn node cha trực tiếp (chỉ 1 cấp) | `//p/parent::div` | Thẻ `<div>` là cha trực tiếp của `<p>` |
| **ancestor** | `ancestor::node` | Chọn tất cả tổ tiên (cha, ông, cụ...) | `//span/ancestor::div` | Tất cả `<div>` là tổ tiên của `<span>` |
| **ancestor-or-self** | `ancestor-or-self::node` | Chọn chính nó + tất cả tổ tiên | `//span/ancestor-or-self::div` | `<span>` và tất cả `<div>` tổ tiên |
| **self** | `self::node` | Chọn chính node hiện tại | `//p/self::p` | Chính node `<p>` đó |
| **following-sibling** | `following-sibling::node` | Chọn các node anh em đứng SAU (cùng cấp) | `//h2/following-sibling::p` | Tất cả `<p>` đứng sau `<h2>` cùng cấp |
| **preceding-sibling** | `preceding-sibling::node` | Chọn các node anh em đứng TRƯỚC (cùng cấp) | `//h3/preceding-sibling::h2` | Tất cả `<h2>` đứng trước `<h3>` cùng cấp |
| **following** | `following::node` | Chọn tất cả node sau trong toàn document | `//h1/following::p` | Tất cả `<p>` xuất hiện sau `<h1>` trong document |
| **preceding** | `preceding::node` | Chọn tất cả node trước trong toàn document (trừ tổ tiên) | `//footer/preceding::div` | Tất cả `<div>` xuất hiện trước `<footer>` |
| **attribute** | `attribute::name` hoặc `@name` | Chọn thuộc tính của node | `//div/attribute::class` hoặc `//div/@class` | Giá trị attribute `class` của `<div>` |
| **namespace** | `namespace::prefix` | Chọn namespace nodes (hiếm dùng trong HTML) | `//element/namespace::*` | Tất cả namespace của element |

### Ví Dụ Axes Trên User Registration Form

```xpath
// 1. parent — lấy div cha của label
//label[@for="username"]/parent::div
```
→ `<div class="form-group" id="parent">` — 1 of 1

```xpath
// 2. parent::* — lấy bất kỳ thẻ cha nào
//label[@for="username"]/parent::*
```
→ 5 kết quả (cha của label là div, cha của div là form, v.v.)

```xpath
// 3. ancestor::div — tất cả tổ tiên là div
//label[@for="username"]/ancestor::div
```
→ 2 kết quả: `div.form-group` và `div.container#ancestor`

```xpath
// 4. ancestor::* — tất cả tổ tiên (bất kỳ thẻ)
//label[@for="username"]/ancestor::*
```
→ 5 kết quả: div → form → div → body → html

```xpath
// 5. ancestor::form — lấy form tổ tiên
//label[@for="username"]/ancestor::form
//input[@type="text"]/ancestor::form
//input[@id="username"]/ancestor::form
```
→ Đều cho kết quả: `<form id="registrationForm">` — 1 of 1

```xpath
// 6. child::div — lấy tất cả div con trực tiếp
//form[@id="registrationForm"]/child::div
```
→ 16 kết quả (16 div.form-group)

```xpath
// 7. child::div[@id="parent"] — lấy div con cụ thể
//form[@id="registrationForm"]/child::div[@id="parent"]
```
→ 1 kết quả: `<div id="parent">`

```xpath
// 8. descendant::* — tất cả con cháu
//form[@id="registrationForm"]/descendant::*
```
→ **83 kết quả** (tất cả element bên trong form)

```xpath
// 9. preceding::* — tất cả phần tử trước form (trừ tổ tiên)
//form[@id="registrationForm"]/preceding::*
```
→ 11 kết quả

```xpath
// 10. preceding-sibling::* — anh em cùng cấp phía trước form
//form[@id="registrationForm"]/preceding-sibling::*
```
→ 1 kết quả: `<h1 id="self">`

```xpath
// 11. following::* — tất cả phần tử sau form trong document
//form[@id="registrationForm"]/following::*
```
→ 14 kết quả (table, iframe, dialog, script...)

```xpath
// 12. following-sibling::* — anh em cùng cấp phía sau form
//form[@id="registrationForm"]/following-sibling::*
```
→ 3 kết quả: `table`, `iframe`, `dialog`

---

## 4. Ví Dụ Thực Tế Với HTML Mẫu

### HTML Mẫu

```html
<body>
  <header id="top">
    <h1>Tiêu đề chính</h1>
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
    </nav>
  </header>

  <main>
    <article>
      <h2 id="title1">Bài viết 1</h2>
      <p class="intro">Đoạn giới thiệu</p>
      <p class="content">Nội dung 1</p>
      <div class="comments">
        <span class="author">Tác giả</span>
        <span class="date">01/01/2024</span>
      </div>
    </article>

    <article>
      <h2 id="title2">Bài viết 2</h2>
      <p class="intro">Giới thiệu 2</p>
      <p class="content">Nội dung 2</p>
    </article>
  </main>

  <footer>
    <p>Copyright 2025</p>
  </footer>
</body>
```

### Bảng Ví Dụ Thực Tế

| XPath Expression | Kết Quả | Giải Thích |
|-----------------|---------|-----------|
| `//h2[@id='title1']/parent::article` | `<article>` chứa "Bài viết 1" | Chọn cha của h2 |
| `//article/child::p` | Tất cả `<p>` con trực tiếp của article | Chỉ p cấp 1, không lấy p trong div |
| `//article/descendant::span` | `<span class="author">` và `<span class="date">` | Tất cả span bên trong article ở mọi cấp |
| `//h2[@id='title1']/following-sibling::p` | `<p class="intro">` và `<p class="content">` của bài 1 | Các p anh em SAU h2 cùng cấp |
| `//p[@class='content']/preceding-sibling::p` | `<p class="intro">` | p anh em đứng TRƯỚC content |
| `//span[@class='author']/ancestor::article` | `<article>` chứa span đó | Tổ tiên article của span |
| `//h2[@id='title1']/following::article` | `<article>` chứa "Bài viết 2" | Article xuất hiện SAU h2 trong toàn document |
| `//footer/preceding::h2` | Cả 2 thẻ `<h2>` | Tất cả h2 xuất hiện trước footer |
| `//nav/child::a` | 2 thẻ `<a>` trong nav | Các link trong nav |
| `//span[@class='date']/parent::div/@class` | `"comments"` | Lấy giá trị class của div cha |

---

## 5. Ví Dụ Kết Hợp Nhiều Axes

Đây là kỹ năng nâng cao — **chuỗi nhiều axes lại với nhau** để điều hướng phức tạp:

```xpath
// Tìm article, rồi lấy tất cả p con trực tiếp
//article/child::p
```

```xpath
// Tìm span, lên cha (div), rồi lấy tất cả span anh em
//span[@class='author']/parent::div/child::span
```

```xpath
// Tìm h2 đầu tiên, lấy tối đa 2 p anh em SAU nó
//h2[@id='title1']/following-sibling::p[position() <= 2]
```

```xpath
// Tìm span date, lên article tổ tiên, xuống lấy h2 con
//span[@class='date']/ancestor::article/child::h2
```

```xpath
// Tìm p intro, lấy TẤT CẢ element anh em sau nó
//p[@class='intro']/following-sibling::*
```

```xpath
// Tìm footer, lấy tất cả div trước nó trong toàn document
//footer/preceding::div
```

### Ví Dụ Nâng Cao Trên Registration Form

```xpath
// Từ input text, qua following, vào div#child, lấy label con
//input[@type='text']/following::div[@id='child']/child::label
// Viết ngắn hơn với //
//input[@type='text']/following::div[@id='child']//label
// Ngắn nhất
//input[@id='username']/following::div[@id='child']/label
```
→ 1 kết quả: `<label for="email">Email:</label>`

```xpath
// Từ input username, lên ancestor form, xuống div#child, lấy label
//input[@id='username']/ancestor::form//div[@id='child']/label
```
→ 1 kết quả: `<label for="email">Email:</label>`

```xpath
// Từ input type text, qua following, lấy div#child, rồi lấy label con theo sibling
//input[@type='text']/following::div[@id='child']/label
//input[@id='username']/following::div[@id='child']//label

// Cách dùng child axis tường minh
//input[@id='username']/ancestor::form//div[@id='child']/label
```

---

## 6. Bảng So Sánh Axes Tương Tự

### Nhóm Con Cháu (Đi Xuống)

| Axis | Phạm vi | Khác biệt |
|------|---------|-----------|
| `child::` | Chỉ con trực tiếp (cấp 1) | Không lấy cháu, chắt |
| `descendant::` | Tất cả con cháu (mọi cấp) | Đi sâu không giới hạn |
| `descendant-or-self::` | Mọi cấp + chính nó | Bao gồm cả chính nó |

### Nhóm Tổ Tiên (Đi Lên)

| Axis | Phạm vi | Khác biệt |
|------|---------|-----------|
| `parent::` | Chỉ cha trực tiếp | Đúng 1 cấp trên |
| `ancestor::` | Tất cả tổ tiên | Đi ngược đến root |
| `ancestor-or-self::` | Tất cả tổ tiên + chính nó | Bao gồm cả chính nó |

### Nhóm Anh Em (Cùng Cấp)

| Axis | Hướng | Phạm vi |
|------|-------|---------|
| `following-sibling::` | Sau | Anh em SAU cùng cha |
| `preceding-sibling::` | Trước | Anh em TRƯỚC cùng cha |

### Nhóm Toàn Document

| Axis | Hướng | Phạm vi | Lưu ý |
|------|-------|---------|-------|
| `following::` | Sau | Tất cả node SAU (mọi cấp, mọi nhánh) | Bao gồm cả sibling và nhánh khác |
| `preceding::` | Trước | Tất cả node TRƯỚC (mọi cấp, mọi nhánh) | **Không** bao gồm tổ tiên |

```
Tóm tắt quan hệ:
following-sibling ⊂ following    (following-sibling là tập con của following)
preceding-sibling ⊂ preceding    (preceding-sibling là tập con của preceding)
child             ⊂ descendant   (child là tập con của descendant)
parent            ⊂ ancestor     (parent là tập con của ancestor)
```

---

## Tổng Hợp Nhanh — Cheat Sheet

### Cấu Trúc XPath

```
//tag[@attr='val']/axis::targetTag[condition]
  |       |            |       |        |
  |       |            |       |        └─ Điều kiện lọc thêm
  |       |            |       └─ Loại thẻ muốn chọn
  |       |            └─ Trục điều hướng
  |       └─ Điều kiện cho nút xuất phát
  └─ Tìm ở bất kỳ đâu trong document
```

### Bảng Tra Nhanh

| Muốn làm gì | XPath |
|------------|-------|
| Tìm bất kỳ thẻ nào | `//*` |
| Tìm theo id | `//*[@id='value']` hoặc `//div[@id='value']` |
| Tìm text chính xác | `//tag[text()='value']` |
| Tìm text chứa chuỗi | `//tag[contains(text(),'value')]` |
| Tìm text bắt đầu bằng | `//tag[starts-with(text(),'prefix')]` |
| Bỏ qua khoảng trắng | `//tag[normalize-space()='value']` |
| Phủ định điều kiện | `//tag[not(điều_kiện)]` |
| Kết hợp điều kiện | `//tag[cond1 and cond2]` hoặc `[cond1 or cond2]` |
| Lên cha | `//tag/parent::*` |
| Lên tổ tiên | `//tag/ancestor::div` |
| Xuống con trực tiếp | `//tag/child::div` hoặc `//tag/div` |
| Xuống mọi cấp | `//tag/descendant::span` hoặc `//tag//span` |
| Anh em phía sau | `//tag/following-sibling::p` |
| Anh em phía trước | `//tag/preceding-sibling::p` |
| Anh em gần nhất | `//tag/following-sibling::p[1]` |
| Lấy attribute | `//tag/@class` hoặc `//tag/attribute::class` |
