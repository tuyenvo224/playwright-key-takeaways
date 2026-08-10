# XPath Axis: preceding và preceding-sibling

## 1. preceding axis là gì?

`preceding::` chọn **tất cả các node xuất hiện trước** context node theo thứ tự document (document order), ngoại trừ:

- Tổ tiên (ancestors)
- Attribute nodes
- Namespace nodes

---

## 2. preceding có bao gồm con cháu của node trước không?

**Có.** Nếu một node phía trước có các thẻ con bên trong, các thẻ con đó cũng được chọn vì:

- Chúng xuất hiện trước context node trong document order
- Chúng **không phải** tổ tiên của context node

**Ví dụ:**

```html
<div id="preceding1-1">
    <span>Con span</span>
    <div id="inner-div">Con div</div>
</div>
<div id="preceding1-2">Nội dung trước 1-2</div>
<div id="wrapper1">
    <div id="current1">Nội dung hiện tại 1</div>
    <div id="sibling1-1">Anh chị em sau 1-1</div>
</div>
```

```xpath
//div[@id='current1']/preceding::div
```

Kết quả: `div#preceding1-1`, `div#inner-div`, `div#preceding1-2`

> `div#wrapper1` **không được chọn** vì là ancestor của `current1`.

---

## 3. Tại sao Attribute nodes bị loại?

Attribute nodes (`id`, `class`, `href`,...) **không tồn tại trong document order** — XPath spec định nghĩa chúng là node đặc biệt, chỉ truy cập được qua `@` hoặc `attribute::`.

Khi `preceding::*` chọn được một element, bạn đã có **toàn bộ thẻ đó** rồi, bao gồm cả attributes bên trong. Attribute không bị "mất", chỉ không xuất hiện như một node riêng lẻ trong kết quả `preceding`.

---

## 4. Namespace nodes là gì?

Namespace nodes khai báo bằng `xmlns` trong XML, dùng để phân biệt các thẻ từ các "không gian tên" khác nhau.

```xml
<root xmlns:svg="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink">
    <svg:circle cx="50" cy="50" r="40"/>
</root>
```

Trong HTML5 thông thường, namespace nodes rất hiếm gặp. Phổ biến trong SVG inline XML, XHTML, SOAP/WSDL.

---

## 5. Ví dụ tổng hợp

### HTML:

```html
<div id="wrapper">
    <p id="para1" class="highlight">Đoạn 1</p>
    <span id="span1" class="text">Span 1</span>
    <div id="current">Hiện tại</div>
    <p id="para2">Đoạn 2</p>
</div>
```

### Cây document order với `div#current` làm context node:

```
1. div#wrapper           ← ancestor → BỊ LOẠI
2.   p#para1             ← trước current → CHỌN
3.     @id="para1"       ← attribute node → BỊ LOẠI
4.     @class="highlight"← attribute node → BỊ LOẠI
5.   span#span1          ← trước current → CHỌN
6.     @id="span1"       ← attribute node → BỊ LOẠI
7.     @class="text"     ← attribute node → BỊ LOẠI
8. → div#current         ← context node
9.   p#para2             ← SAU current → BỊ LOẠI
```

### Các XPath và kết quả tương ứng:

| XPath | Kết quả |
|---|---|
| `//div[@id='current']/preceding::*` | `p#para1`, `span#span1` |
| `//div[@id='current']/preceding::p` | `p#para1` |
| `//div[@id='current']/preceding::span` | `span#span1` (toàn bộ element) |
| `//div[@id='current']/preceding::p/@class` | `"highlight"` |
| `//div[@id='current']/preceding::*/@class` | `"highlight"`, `"text"` |

---

## 6. Dùng preceding trong Playwright

### Chỉ cần tương tác (click, fill):

```javascript
const span = page.locator('//div[@id="current"]/preceding::span');
await span.click();
await span.fill('nội dung');
```

Locator đã đủ, không cần đọc thêm gì.

### Dùng `textContent()`, `getAttribute()` khi nào?

Chủ yếu dùng trong **assertion** — sau khi thực hiện hành động, kiểm tra kết quả có đúng không:

```javascript
// Verify text hiển thị đúng
expect(await span.textContent()).toBe('Span 1');

// Verify class (kiểm tra UI đang ở đúng trạng thái)
expect(await span.getAttribute('class')).toBe('text');

// Verify sau khi fill
await span.fill('Hello');
expect(await span.inputValue()).toBe('Hello');
```

### Tổng hợp theo mục đích:

| Mục đích | Dùng gì |
|---|---|
| Click, fill, hover | `locator` + action là đủ |
| Kiểm tra text hiển thị | `textContent()` + `expect` |
| Kiểm tra trạng thái CSS/class | `getAttribute('class')` + `expect` |
| Debug xem element đang có gì | `textContent()`, `getAttribute()` |

---

## 7. Phân biệt predicate, lấy attribute, và so sánh boolean

Cùng HTML:

```html
<div id="current">Hiện tại</div>
<!-- phía trước có: -->
<span id="span1" class="text">Span 1</span>
```

### 3 cú pháp dễ nhầm lẫn:

| XPath | Cú pháp | Trả về | Dùng được làm locator? |
|---|---|---|---|
| `//div[@id='current']/preceding::span[@class='text']` | `[@attr='value']` trong `[]` | Toàn bộ element `<span>` | Có |
| `//div[@id='current']/preceding::span/@class` | `/@attr` sau `/` | Attribute value `"text"` | Chỉ để đọc |
| `//div[@id='current']/preceding::span/@class='text'` | `/@attr='value'` ngoài `[]` | Boolean `true/false` | Không |

### Giải thích chi tiết:

**`[@class='text']` — Predicate (bộ lọc)**
```xpath
//div[@id='current']/preceding::span[@class='text']
```
Nằm trong dấu `[]` → lọc element theo điều kiện → trả về **element**

**`/@class` — Lấy attribute**
```xpath
//div[@id='current']/preceding::span/@class
```
Đứng sau `/` → truy cập vào attribute của element → trả về **giá trị `"text"`**

**`/@class='text'` — So sánh boolean**
```xpath
//div[@id='current']/preceding::span/@class='text'
```
So sánh ngoài `[]` → XPath trả về **`true` hoặc `false`**, không trả về node nào → không dùng làm locator trong Playwright được.

### Ghi nhớ nhanh:

- `[...]` → lọc → trả về element
- `/...` → đi sâu vào → trả về attribute hoặc node con
- `=` ngoài `[]` → so sánh → trả về boolean

---

## 8. Phân biệt `/` và `//` trong XPath

### `/` (1 gạch) — child trực tiếp

`/label` là rút gọn của `child::label` — chỉ chọn các `label` là **con trực tiếp** của node hiện tại.

### `//` (2 gạch) — mọi cấp con cháu

`//label` là rút gọn của `/descendant-or-self::node()/child::label` — chọn `label` ở **bất kỳ cấp nào** bên trong node hiện tại.

### Ví dụ minh họa:

```html
<div id="child">
    <label>Email:</label>          <!-- con trực tiếp -->
    <div class="inner">
        <label>Nested:</label>     <!-- con của con -->
    </div>
</div>
```

| XPath | Kết quả | Số lượng |
|---|---|---|
| `//div[@id='child']/label` | `<label>Email:</label>` | 1 |
| `//div[@id='child']//label` | `<label>Email:</label>` và `<label>Nested:</label>` | 2 |

**Giải thích:**
- `/label` → chỉ tìm label là **con trực tiếp** → chỉ ra `Email:`
- `//label` → tìm label ở **mọi cấp** bên trong div → ra cả `Email:` lẫn `Nested:`

### Ví dụ thực tế từ bài học:

```xpath
//input[@id="username"]/following::div[@id="child"]/label
```
→ Rút gọn của:
```xpath
//input[@id="username"]/following::div[@id="child"]/child::label
```
→ Chỉ lấy `label` là con trực tiếp của `div#child`

```xpath
//input[@id="username"]/following::div[@id="child"]//label
```
→ Rút gọn của:
```xpath
//input[@id="username"]/following::div[@id="child"]/descendant-or-self::node()/child::label
```
→ Lấy `label` ở bất kỳ cấp nào bên trong `div#child`

---

## 9. Bảng rút gọn cú pháp XPath

| Rút gọn | Dạng đầy đủ | Ý nghĩa |
|---|---|---|
| `/label` | `/child::label` | Con trực tiếp tên `label` |
| `//label` | `/descendant-or-self::node()/child::label` | Mọi cấp con cháu tên `label` |
| `@class` | `attribute::class` | Attribute tên `class` |
| `.` | `self::node()` | Chính node hiện tại |
| `..` | `parent::node()` | Node cha |

**Lưu ý quan trọng:**
- Chỉ có `child::` mới rút gọn thành `/`
- `//` **không phải** rút gọn của `child::` mà là của `descendant-or-self::`
- Các axis khác như `following::`, `preceding::`, `ancestor::` **không có cú pháp rút gọn** — phải viết đầy đủ

### Ví dụ từng loại rút gọn:

```html
<form id="form">
    <div id="parent">
        <label for="username">Username:</label>
        <input type="text" id="username">
        <div id="child">
            <label>Email:</label>
            <input type="email" id="email">
        </div>
    </div>
</form>
```

**`/` → child::**
```xpath
//div[@id='parent']/label
<!-- Kết quả: <label for="username">Username:</label> — chỉ label con trực tiếp -->
```

**`//` → descendant-or-self::**
```xpath
//div[@id='parent']//label
<!-- Kết quả: cả 2 label (Username và Email) -->
```

**`@` → attribute::**
```xpath
//input[@id='username']/@type
<!-- Kết quả: "text" -->
```

**`.` → self::**
```xpath
//input[@id='username']/.
<!-- Kết quả: chính thẻ input đó -->
```

**`..` → parent::**
```xpath
//input[@id='username']/..
<!-- Kết quả: <div id="parent"> — thẻ cha của input -->
```

---

## 10. Lưu ý khi dùng `//` trong Playwright

Khi XPath trả về nhiều hơn 1 element, Playwright sẽ báo lỗi **"strict mode violation"** nếu bạn gọi action trực tiếp:

```javascript
// ❌ Lỗi nếu //label tìm được 2 label
await page.locator('//div[@id="child"]//label').click();
// Error: strict mode violation — found 2 elements

// ✅ Chỉ định rõ index
await page.locator('//div[@id="child"]//label').first().click();
await page.locator('//div[@id="child"]//label').nth(1).click();

// ✅ Hoặc dùng / để chỉ lấy con trực tiếp (nếu chắc chắn cấu trúc HTML)
await page.locator('//div[@id="child"]/label').click();
```

**Nguyên tắc chọn `/` hay `//`:**
- Dùng `/` khi chắc chắn element nằm **ngay bên trong** (tránh chọn nhầm)
- Dùng `//` khi cấu trúc HTML có thể thay đổi hoặc element nằm sâu hơn — nhưng cần đảm bảo chỉ match 1 kết quả

---

## 11. Tóm tắt quy tắc preceding

- Chọn tất cả nodes **trước** context node trong document order
- **Loại trừ:** ancestors, attribute nodes, namespace nodes
- **Bao gồm:** con cháu của các nodes phía trước (nếu chúng xuất hiện trước context node)
- Attribute không bị mất — vẫn truy cập được qua element đã chọn
