# XPath Axis Methods - Hướng Dẫn Chi Tiết

> Nguồn: https://material.playwrightvn.com/07-xpath-axe-methods.html

XPath Axis (trục XPath) là cách để **điều hướng qua cây DOM** từ một nút (node) đang chọn sang các nút khác có quan hệ với nó. Hiểu rõ các axis giúp bạn viết được các selector mạnh mẽ, linh hoạt trong Playwright và các công cụ test khác.

---

## Cú Pháp Tổng Quát

```
//tagName[@attribute='value']/axis::nodeType
```

Ví dụ:
```
//div[@id='child']/ancestor::div
```
Đọc là: *"Từ div có id='child', đi theo trục ancestor, lấy tất cả div tổ tiên"*

---

## Sơ Đồ Quan Hệ DOM (Tổng Quan)

```
         [grandparent]          ← ancestor
               |
           [parent]             ← ancestor / parent
           /       \
      [child]    [sibling]      ← child / following-sibling / preceding-sibling
       /   \
  [sub1] [sub2]                 ← descendant
```

---

## 13 Axis Methods Chi Tiết

---

### 1. `ancestor` — Chọn tất cả tổ tiên

**Mô tả:** Từ nút hiện tại, đi ngược lên trên cây DOM, chọn **tất cả** các nút cha, ông, cụ... cho đến root.

**Cú pháp:**
```xpath
//div[@id='child']/ancestor::div
```

**HTML ví dụ:**
```html
<div id="grandparent">
  <div id="parent">
    <div id="child">Nội dung</div>
  </div>
</div>
```

**Giải thích từng bước:**
- Bắt đầu tại `div#child`
- `ancestor::div` → đi ngược lên, tìm tất cả `<div>` là tổ tiên
- Kết quả: `div#parent` **và** `div#grandparent` đều được chọn

**Khi nào dùng?**
- Khi bạn đang ở một element con và cần tìm container/wrapper bao ngoài nó.
- Ví dụ: Tìm `<form>` chứa một `<input>` cụ thể.

```xpath
//input[@name='email']/ancestor::form
```

---

### 2. `ancestor-or-self` — Chọn tổ tiên + chính nó

**Mô tả:** Tương tự `ancestor`, nhưng **bao gồm cả chính nút hiện tại** trong kết quả.

**Cú pháp:**
```xpath
//div[@id='child']/ancestor-or-self::div
```

**HTML ví dụ:**
```html
<div id="grandparent">
  <div id="parent">
    <div id="child">Nội dung</div>
  </div>
</div>
```

**Giải thích:**
- Bắt đầu tại `div#child`
- Kết quả: `div#child` + `div#parent` + `div#grandparent` — **3 div** đều được chọn

**So sánh `ancestor` vs `ancestor-or-self`:**

| | `ancestor` | `ancestor-or-self` |
|---|---|---|
| Chọn nút hiện tại | Không | Có |
| Chọn cha | Có | Có |
| Chọn ông | Có | Có |

---

### 3. `child` — Chọn các con trực tiếp

**Mô tả:** Chọn **chỉ các phần tử con trực tiếp** (cấp 1) của nút hiện tại. Không đi sâu hơn.

**Cú pháp:**
```xpath
//div[@id='parent']/child::*
//div[@id='parent']/child::div
//div[@id='parent']/child::p
```

**HTML ví dụ:**
```html
<div id="parent">
  <div id="child1">Con 1</div>
  <p id="child2">Con 2</p>
  <div id="child3">
    <span>Cháu</span>   <!-- KHÔNG được chọn -->
  </div>
</div>
```

**Giải thích:**
- `child::*` → chọn tất cả con trực tiếp: `div#child1`, `p#child2`, `div#child3`
- `child::div` → chỉ chọn con là `<div>`: `div#child1`, `div#child3`
- `<span>` (cháu) **không** được chọn vì không phải con trực tiếp

**Lưu ý:** `child::` là axis **mặc định** trong XPath, nên hai cách sau là tương đương:
```xpath
//div[@id='parent']/child::div
//div[@id='parent']/div          ← viết tắt
```

---

### 4. `descendant` — Chọn tất cả hậu duệ

**Mô tả:** Chọn **tất cả** các phần tử nằm bên trong nút hiện tại, không giới hạn độ sâu (con, cháu, chắt...).

**Cú pháp:**
```xpath
//div[@id='parent']/descendant::div
//div[@id='parent']/descendant::*
```

**HTML ví dụ:**
```html
<div id="parent">
  <div id="child1">
    <span id="grandchild1">Cháu 1</span>
  </div>
  <div id="child2">
    <p id="grandchild2">Cháu 2</p>
  </div>
</div>
```

**Giải thích:**
- `descendant::div` → chọn `div#child1`, `div#child2`
- `descendant::*` → chọn **tất cả**: `div#child1`, `span#grandchild1`, `div#child2`, `p#grandchild2`

**So sánh `child` vs `descendant`:**

| | `child` | `descendant` |
|---|---|---|
| Con trực tiếp | Có | Có |
| Cháu, chắt... | Không | Có |
| Tương đương với | `//parent/div` | `//parent//div` |

---

### 5. `descendant-or-self` — Chọn hậu duệ + chính nó

**Mô tả:** Tương tự `descendant`, nhưng **bao gồm cả nút hiện tại** trong kết quả.

**Cú pháp:**
```xpath
//div[@id='parent']/descendant-or-self::div
```

**HTML ví dụ:**
```html
<div id="parent">
  <div id="child1">Con 1</div>
  <div id="child2">Con 2</div>
</div>
```

**Giải thích:**
- Kết quả: `div#parent` + `div#child1` + `div#child2` — **cả 3** được chọn

**Thực tế quan trọng:** `//div` trong XPath là viết tắt của `/descendant-or-self::node()/div`, đó là lý do `//` có thể tìm ở bất kỳ độ sâu nào.

---

### 6. `parent` — Chọn cha trực tiếp

**Mô tả:** Chọn **duy nhất** phần tử cha trực tiếp (cấp 1 phía trên) của nút hiện tại.

**Cú pháp:**
```xpath
//div[@id='child']/parent::div
//div[@id='child']/parent::*
```

**HTML ví dụ:**
```html
<div id="grandparent">
  <div id="parent">
    <div id="child">Nội dung</div>
  </div>
</div>
```

**Giải thích:**
- Bắt đầu tại `div#child`
- `parent::div` → chỉ chọn `div#parent` (không chọn `div#grandparent`)

**Viết tắt:** `..` trong XPath tương đương với `parent::node()`
```xpath
//div[@id='child']/..         ← tương đương parent::node()
//div[@id='child']/parent::*  ← tường minh hơn
```

**Ứng dụng thực tế:**
```xpath
// Tìm li chứa link có text "Login", rồi lấy ul cha của nó
//a[text()='Login']/parent::li/parent::ul
```

---

### 7. `self` — Chọn chính nút hiện tại

**Mô tả:** Chọn **chính nút đang xét**, không đi đâu cả.

**Cú pháp:**
```xpath
//div[@id='current']/self::div
```

**HTML ví dụ:**
```html
<div id="current">Nội dung hiện tại</div>
```

**Giải thích:**
- Kết quả: chỉ `div#current` — chính element đó.
- `self::div` sẽ trả về kết quả nếu nút hiện tại **là** `<div>`, ngược lại không trả về gì.

**Viết tắt:** `.` trong XPath tương đương `self::node()`
```xpath
//div[@id='current']/.        ← tương đương self::node()
```

**Khi nào dùng?**
- Dùng trong các biểu thức phức tạp để tự tham chiếu chính nó.
- Kiểm tra type của node: `self::div` chỉ match nếu node là `<div>`.

---

### 8. `following-sibling` — Chọn anh/em phía sau

**Mô tả:** Từ nút hiện tại, chọn **tất cả các phần tử cùng cấp (sibling) xuất hiện PHÍA SAU** nó trong DOM.

**Cú pháp:**
```xpath
//div[@id='current']/following-sibling::div
//div[@id='current']/following-sibling::*
//div[@id='current']/following-sibling::div[1]   ← sibling đầu tiên phía sau
```

**HTML ví dụ:**
```html
<div id="sibling-before">Anh em trước</div>   <!-- KHÔNG được chọn -->
<div id="current">Nút hiện tại</div>
<div id="sibling-after-1">Anh em sau 1</div>   <!-- Được chọn -->
<div id="sibling-after-2">Anh em sau 2</div>   <!-- Được chọn -->
```

**Giải thích:**
- `following-sibling::div` → chọn `div#sibling-after-1` và `div#sibling-after-2`
- `following-sibling::div[1]` → chỉ chọn `div#sibling-after-1`

**Ứng dụng thực tế:**
```xpath
// Tìm label "Email", rồi lấy input ngay sau nó
//label[text()='Email']/following-sibling::input[1]
```

---

### 9. `preceding-sibling` — Chọn anh/em phía trước

**Mô tả:** Từ nút hiện tại, chọn **tất cả các phần tử cùng cấp xuất hiện PHÍA TRƯỚC** nó trong DOM.

**Cú pháp:**
```xpath
//div[@id='current']/preceding-sibling::div
//div[@id='current']/preceding-sibling::div[1]   ← sibling gần nhất phía trước
```

**HTML ví dụ:**
```html
<div id="sibling-1">Anh em trước 1</div>   <!-- Được chọn -->
<div id="sibling-2">Anh em trước 2</div>   <!-- Được chọn -->
<div id="current">Nút hiện tại</div>
<div id="sibling-after">Anh em sau</div>   <!-- KHÔNG được chọn -->
```

**Giải thích:**
- `preceding-sibling::div` → chọn `div#sibling-1` và `div#sibling-2`
- `preceding-sibling::div[1]` → chọn `div#sibling-2` (gần nhất tính từ current về trước)

**Lưu ý quan trọng về index:**
> Với `preceding-sibling`, index `[1]` là phần tử **gần nhất** với nút hiện tại (đếm ngược), không phải phần tử đầu tiên trong DOM.

**So sánh `preceding-sibling` vs `following-sibling`:**

```
[sibling-1] [sibling-2] [current] [sibling-3] [sibling-4]
|_________preceding_________|         |____following____|
```

---

### 10. `following` — Chọn tất cả nút phía sau (toàn document)

**Mô tả:** Chọn **tất cả các nút xuất hiện sau thẻ đóng** của nút hiện tại trong toàn bộ document, bao gồm cả nút ở nhánh cây khác — không giới hạn cùng cấp.

**Cú pháp:**
```xpath
//div[@id='current']/following::div
//div[@id='current']/following::*
```

**HTML ví dụ:**
```html
<div id="section1">
  <div id="current">Nút hiện tại</div>
</div>
<div id="section2">               <!-- Được chọn (khác nhánh) -->
  <p id="para">Đoạn văn</p>       <!-- Được chọn -->
</div>
<footer id="footer">Cuối</footer> <!-- Được chọn -->
```

**Giải thích:**
- `following::div` → chọn `div#section2`
- `following::*` → chọn `div#section2`, `p#para`, `footer#footer`

**Phân biệt `following` vs `following-sibling`:**

| | `following` | `following-sibling` |
|---|---|---|
| Phạm vi | Toàn bộ document | Chỉ cùng cấp (cùng cha) |
| Các nhánh khác | Có | Không |

---

### 11. `preceding` — Chọn tất cả nút phía trước (toàn document)

**Mô tả:** Chọn **tất cả các nút xuất hiện trước nút hiện tại** trong toàn bộ document, **ngoại trừ các tổ tiên** (ancestor).

**Cú pháp:**
```xpath
//div[@id='current']/preceding::div
```

**HTML ví dụ:**
```html
<header>Tiêu đề</header>           <!-- Được chọn -->
<div id="section1">
  <p>Đoạn trước</p>                <!-- Được chọn -->
</div>
<div id="parent">                  <!-- KHÔNG được chọn (là ancestor) -->
  <div id="current">Nút hiện tại</div>
</div>
```

**Giải thích:**
- `preceding::*` → chọn `header`, `div#section1`, `p` (tất cả nút trước)
- `div#parent` **không được chọn** dù ở trước, vì nó là **ancestor** của `current`

**Phân biệt `preceding` vs `ancestor`:**

| | `preceding` | `ancestor` |
|---|---|---|
| Tổ tiên trực tiếp | Không | Có |
| Nút cùng cấp trước | Có | Không |
| Nút nhánh khác trước | Có | Không |

---

### 12. `attribute` — Chọn thuộc tính của nút

**Mô tả:** Chọn **các thuộc tính (attributes)** của nút hiện tại như `id`, `class`, `href`, `name`...

**Cú pháp:**
```xpath
//div[@id='child']/attribute::*        ← tất cả attributes
//div[@id='child']/attribute::id       ← chỉ attribute id
//div[@id='child']/attribute::class    ← chỉ attribute class
```

**Viết tắt:** `@` là cách viết tắt của `attribute::`
```xpath
//div/attribute::id   ←→   //div/@id
//div/attribute::*    ←→   //div/@*
```

**HTML ví dụ:**
```html
<div id="child" class="example" data-type="card">Nội dung</div>
```

**Giải thích:**
- `attribute::*` → trả về giá trị của `id="child"`, `class="example"`, `data-type="card"`
- `attribute::class` → trả về `"example"`

**Ứng dụng thực tế:**
```xpath
// Lấy href của link
//a[@text()='Click here']/@href

// Kiểm tra class của element
//div[contains(@class, 'active')]
```

---

### 13. `namespace` — Chọn namespace của nút

**Mô tả:** Chọn **các khai báo namespace** của nút hiện tại. Chủ yếu dùng trong **XML** hoặc XHTML, ít gặp trong HTML thông thường.

**Cú pháp:**
```xpath
//element/namespace::*
```

**XML ví dụ:**
```xml
<root xmlns:bar="http://example.com"
      xmlns:baz="http://example2.com">
  <bar:child>Nội dung</bar:child>
</root>
```

**Giải thích:**
- `namespace::*` → trả về tất cả namespace nodes: `bar`, `baz`, và namespace mặc định `xml`

**Lưu ý:** Axis này **rất ít dùng** trong web testing thông thường. Nó chủ yếu xuất hiện khi làm việc với XML documents, SOAP web services, hoặc SVG.

---

## Bảng Tổng Hợp Nhanh

| Axis | Hướng đi | Phạm vi | Ghi nhớ nhanh |
|------|----------|---------|---------------|
| `ancestor` | Lên trên | Tất cả tổ tiên | "đi tìm ông bà" |
| `ancestor-or-self` | Lên trên | Tổ tiên + chính nó | "ông bà + tôi" |
| `child` | Xuống 1 cấp | Con trực tiếp | "con ruột" |
| `descendant` | Xuống nhiều cấp | Tất cả hậu duệ | "con cháu chắt" |
| `descendant-or-self` | Xuống nhiều cấp | Hậu duệ + chính nó | "con cháu + tôi" |
| `parent` | Lên 1 cấp | Cha trực tiếp | "cha ruột" |
| `self` | Tại chỗ | Chính nút | "chính tôi" |
| `following-sibling` | Ngang - sau | Anh/em cùng cấp sau | "em cùng cha" |
| `preceding-sibling` | Ngang - trước | Anh/em cùng cấp trước | "anh cùng cha" |
| `following` | Sau toàn document | Mọi nút sau khi đóng tag | "tất cả phía sau" |
| `preceding` | Trước toàn document | Mọi nút trước (trừ ancestor) | "tất cả phía trước" |
| `attribute` | Vào trong | Các thuộc tính | "xem giấy tờ" |
| `namespace` | Vào trong | Namespace declarations | "xem quốc tịch XML" |

---

## Viết Tắt Thường Dùng

| Viết tắt | Tương đương đầy đủ |
|----------|-------------------|
| `//div` | `/descendant-or-self::node()/div` |
| `..` | `parent::node()` |
| `.` | `self::node()` |
| `@class` | `attribute::class` |
| `div` (không có axis) | `child::div` |

---

## Ví Dụ Tổng Hợp Trong Playwright

```typescript
// 1. Tìm input bên trong form có id="login"
await page.locator("//form[@id='login']/descendant::input[@type='text']")

// 2. Tìm label của input đang được focus (preceding-sibling)
await page.locator("//input[@name='email']/preceding-sibling::label")

// 3. Tìm thông báo lỗi ngay sau input (following-sibling)
await page.locator("//input[@name='password']/following-sibling::span[@class='error']")

// 4. Tìm row của table chứa text "John" rồi bấm nút Edit trong row đó
await page.locator("//td[text()='John']/parent::tr//button[text()='Edit']")

// 5. Tìm menu item đang active và lấy menu cha
await page.locator("//li[contains(@class,'active')]/ancestor::ul")

// 6. Kiểm tra href của link
const href = await page.locator("//a[text()='Trang chủ']/@href")
```

---

## Mẹo Ghi Nhớ

```
        ancestor ↑↑↑
           ↑
        parent ↑
           |
[preceding] ← [self] → [following]
           |
        child ↓
           ↓
      descendant ↓↓↓

← preceding-sibling | following-sibling →  (cùng cấp với self)
```

> **Quy tắc vàng:**
> - Đi **lên**: dùng `parent`, `ancestor`
> - Đi **xuống**: dùng `child`, `descendant`
> - Đi **ngang**: dùng `following-sibling`, `preceding-sibling`
> - Đi **xa hơn**: thêm `following`, `preceding` (toàn document)
> - Xem **bên trong**: dùng `attribute`
