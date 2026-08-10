# XPath Axes: `ancestor` và `ancestor-or-self` — Hướng Dẫn Chi Tiết

> Nguồn: https://material.playwrightvn.com/08-xpath-ancestor-or-self.html

---

## Tổng Quan

Cả hai axis này đều dùng để **đi ngược lên trên cây DOM** — từ một element đang chọn, tìm các phần tử bao bọc nó ở các cấp trên. Đây là kỹ thuật rất phổ biến trong Playwright khi bạn xác định được element con nhưng cần thao tác với container/wrapper bên ngoài.

| Axis | Mô tả | Bao gồm chính nó? |
|------|-------|:-----------------:|
| `ancestor` | Chọn tất cả tổ tiên (cha, ông, cụ...) | Không |
| `ancestor-or-self` | Chọn tất cả tổ tiên + chính nút hiện tại | Có |

---

## Cú Pháp

```xpath
//tagBắtĐầu[@attribute='value']/ancestor::tagMuốnChọn
//tagBắtĐầu[@attribute='value']/ancestor-or-self::tagMuốnChọn
```

**Giải thích cú pháp:**
- `//tagBắtĐầu[@attribute='value']` — xác định nút xuất phát (nút hiện tại)
- `/ancestor::` — trục điều hướng: đi ngược lên trên
- `tagMuốnChọn` — loại thẻ muốn lọc trong số các tổ tiên (`div`, `form`, `ul`, `*`...)

---

## Phần 1: `ancestor`

### Định nghĩa

> Chọn **tất cả tổ tiên** (cha mẹ, ông bà, cụ kị...) của nút hiện tại, **không bao gồm** chính nút đó.

---

### Ví dụ 1 — Cấu trúc 3 cấp, cùng loại thẻ

**HTML:**
```html
<div id="grandparent1">
    <div id="parent1">
        <div id="child1">Nội dung 1</div>
    </div>
</div>
```

**XPath:**
```xpath
//div[@id='child1']/ancestor::div
```

**Sơ đồ điều hướng:**
```
[div#grandparent1]  ← được chọn (tổ tiên cấp 2)
      |
  [div#parent1]     ← được chọn (tổ tiên cấp 1)
      |
  [div#child1]      ← điểm xuất phát (KHÔNG được chọn)
```

**Kết quả:** Chọn `div#grandparent1` và `div#parent1`

**Giải thích:**
- Bắt đầu tại `div#child1`
- Đi ngược lên: `div#parent1` là cha → **chọn**
- Tiếp tục lên: `div#grandparent1` là ông → **chọn**
- Chính `div#child1` → **không chọn** (đây là điểm khác so với `ancestor-or-self`)

---

### Ví dụ 2 — Cấu trúc 4 cấp, nút xuất phát khác loại thẻ (`span`)

**HTML:**
```html
<div id="great-grandparent2">
    <div id="grandparent2">
        <div id="parent2">
            <span id="child2">Nội dung 2</span>
        </div>
    </div>
</div>
```

**XPath:**
```xpath
//span[@id='child2']/ancestor::div
```

**Sơ đồ điều hướng:**
```
[div#great-grandparent2]  ← được chọn (tổ tiên cấp 3)
         |
   [div#grandparent2]     ← được chọn (tổ tiên cấp 2)
         |
     [div#parent2]        ← được chọn (tổ tiên cấp 1)
         |
    [span#child2]         ← điểm xuất phát (KHÔNG được chọn, không phải div)
```

**Kết quả:** Chọn `div#great-grandparent2`, `div#grandparent2`, `div#parent2`

**Giải thích:**
- Xuất phát từ `span#child2`
- `ancestor::div` → chỉ lấy các tổ tiên **là thẻ `<div>`**
- Cả 3 div phía trên đều là tổ tiên → đều được chọn
- `span#child2` không phải div, và cũng là điểm xuất phát → không được chọn

**Điểm quan trọng:** Nút xuất phát (`span`) có thể **khác loại thẻ** với loại thẻ bạn tìm trong ancestor. XPath vẫn hoạt động bình thường.

---

### Ví dụ 3 — Nút xuất phát là thẻ `p`, tìm ancestor là `div`

**HTML:**
```html
<div id="grandparent3">
    <div id="parent3">
        <p id="child3">Nội dung 3</p>
    </div>
</div>
```

**XPath:**
```xpath
//p[@id='child3']/ancestor::div
```

**Sơ đồ điều hướng:**
```
[div#grandparent3]  ← được chọn
       |
  [div#parent3]     ← được chọn
       |
  [p#child3]        ← điểm xuất phát (KHÔNG được chọn)
```

**Kết quả:** Chọn `div#grandparent3` và `div#parent3`

**Giải thích:**
- Dù nút xuất phát là `<p>`, ancestor vẫn được tìm đúng
- Cả `div#parent3` và `div#grandparent3` đều là tổ tiên → được chọn

---

## Phần 2: `ancestor-or-self`

### Định nghĩa

> Chọn **tất cả tổ tiên** của nút hiện tại **và cả chính nút hiện tại** trong kết quả.

Đây là sự khác biệt duy nhất so với `ancestor`: kết quả **bao gồm thêm chính nút xuất phát**.

---

### Ví dụ 1 — Cấu trúc 3 cấp, cùng loại thẻ

**HTML:**
```html
<div id="grandparent1">
    <div id="parent1">
        <div id="child1">Nội dung 1</div>
    </div>
</div>
```

**XPath:**
```xpath
//div[@id='child1']/ancestor-or-self::div
```

**Sơ đồ điều hướng:**
```
[div#grandparent1]  ← được chọn (tổ tiên cấp 2)
      |
  [div#parent1]     ← được chọn (tổ tiên cấp 1)
      |
  [div#child1]      ← CŨNG được chọn (chính nút hiện tại)
```

**Kết quả:** Chọn `div#grandparent1`, `div#parent1`, **và `div#child1`** (3 phần tử)

**So sánh với `ancestor` ở ví dụ 1 này:**

| XPath | Kết quả |
|-------|---------|
| `//div[@id='child1']/ancestor::div` | `grandparent1`, `parent1` (2 phần tử) |
| `//div[@id='child1']/ancestor-or-self::div` | `grandparent1`, `parent1`, `child1` **(3 phần tử)** |

---

### Ví dụ 2 — Nút xuất phát là `span`, tìm `ancestor-or-self::div`

**HTML:**
```html
<div id="great-grandparent2">
    <div id="grandparent2">
        <div id="parent2">
            <span id="child2">Nội dung 2</span>
        </div>
    </div>
</div>
```

**XPath:**
```xpath
//span[@id='child2']/ancestor-or-self::div
```

**Sơ đồ điều hướng:**
```
[div#great-grandparent2]  ← được chọn
         |
   [div#grandparent2]     ← được chọn
         |
     [div#parent2]        ← được chọn
         |
    [span#child2]         ← KHÔNG được chọn (là span, không phải div)
```

**Kết quả:** Chọn `div#great-grandparent2`, `div#grandparent2`, `div#parent2` (3 phần tử)

**Giải thích quan trọng:**
- Dù dùng `ancestor-or-self`, nhưng `span#child2` vẫn **không được chọn**
- Lý do: `ancestor-or-self::div` chỉ lấy các nút loại `div` — kể cả chính nó
- Vì `child2` là `<span>`, không phải `<div>`, nên bị loại ra
- Kết quả giống hệt `ancestor::div` trong trường hợp này

**Bài học:** `ancestor-or-self` chỉ bao gồm chính nút nếu nút đó **khớp loại thẻ** bạn chỉ định.

---

### Ví dụ 3 — Nút xuất phát là `p`, tìm `ancestor-or-self::div`

**HTML:**
```html
<div id="grandparent3">
    <div id="parent3">
        <p id="child3">Nội dung 3</p>
    </div>
</div>
```

**XPath:**
```xpath
//p[@id='child3']/ancestor-or-self::div
```

**Kết quả:** Chọn `div#grandparent3` và `div#parent3`

**Giải thích:**
- `p#child3` là điểm xuất phát nhưng không phải `<div>` → không được chọn
- Hai `<div>` phía trên → được chọn
- Kết quả giống hệt `ancestor::div` ở ví dụ 3

---

## Phân Biệt Rõ Hai Axis

### Quy tắc cốt lõi

```
ancestor::tagX         → chỉ chọn các tổ tiên là tagX, BỎ nút hiện tại
ancestor-or-self::tagX → chọn các tổ tiên là tagX + chính nút nếu nó là tagX
```

### Bảng so sánh chi tiết

| Tình huống | `ancestor::div` | `ancestor-or-self::div` |
|-----------|:--------------:|:----------------------:|
| Nút hiện tại là `div` | Không chọn nó | **Chọn nó** |
| Nút hiện tại là `span` | Không chọn nó | Không chọn nó |
| Nút hiện tại là `p` | Không chọn nó | Không chọn nó |
| Các `div` cha phía trên | Chọn hết | Chọn hết |

### Sơ đồ tổng quát

```
ancestor-or-self:
    [div]  ← chọn
      |
    [div]  ← chọn
      |
    [div]  ← chọn (chính nút)  ← ancestor chỉ khác ở điểm này

ancestor:
    [div]  ← chọn
      |
    [div]  ← chọn
      |
    [div]  ← KHÔNG chọn
```

---

## Ứng Dụng Thực Tế Trong Playwright

### 1. Tìm form chứa một input cụ thể

```typescript
// Bắt đầu từ input, tìm form bao ngoài
const form = page.locator("//input[@name='email']/ancestor::form");
await form.screenshot();
```

### 2. Highlight toàn bộ row khi click vào một ô trong bảng

```typescript
// Click vào ô, rồi thao tác với toàn bộ hàng tr
const row = page.locator("//td[text()='John Doe']/ancestor::tr");
await row.click();
```

### 3. Tìm card/container chứa button "Xóa"

```typescript
// Tìm div.card bao quanh button "Xóa"
const card = page.locator("//button[text()='Xóa']/ancestor::div[contains(@class,'card')]");
await card.locator("h2").textContent(); // lấy tiêu đề của card đó
```

### 4. Điều hướng từ error message lên form group

```typescript
// Tìm div.form-group chứa span báo lỗi
const formGroup = page.locator(
  "//span[@class='error-message' and text()='Trường này bắt buộc']/ancestor::div[@class='form-group']"
);
```

### 5. Dùng `ancestor-or-self` để chọn linh hoạt

```typescript
// Dùng khi không chắc element hiện tại có phải div không
const container = page.locator("//div[@id='target']/ancestor-or-self::div[@class='wrapper']");
```

### 6. Kết hợp với `[1]` để lấy tổ tiên gần nhất

```typescript
// Lấy div cha GẦN NHẤT (không phải tất cả)
const nearestParent = page.locator("//span[@id='child']/ancestor::div[1]");
```

> **Lưu ý:** `ancestor::div[1]` trả về tổ tiên **gần nhất** (tính từ nút hiện tại lên trên), không phải tổ tiên đầu tiên trong DOM.

---

## Các Lỗi Thường Gặp

### Lỗi 1: Nhầm `ancestor` với `parent`

```xpath
// parent chỉ lấy CHA TRỰC TIẾP (1 cấp)
//div[@id='child']/parent::div       → chỉ lấy div#parent

// ancestor lấy TẤT CẢ tổ tiên (nhiều cấp)
//div[@id='child']/ancestor::div     → lấy div#parent VÀ div#grandparent
```

### Lỗi 2: Nghĩ rằng `ancestor-or-self` luôn trả về nhiều hơn `ancestor` 1 phần tử

Sai — chỉ trả về nhiều hơn nếu **chính nút hiện tại khớp loại thẻ** được chỉ định:

```xpath
// child1 là div → ancestor-or-self trả về nhiều hơn 1 phần tử
//div[@id='child1']/ancestor-or-self::div   → grandparent, parent, child1 (3 ptu)
//div[@id='child1']/ancestor::div           → grandparent, parent (2 ptu)

// child2 là span → kết quả BẰNG NHAU
//span[@id='child2']/ancestor-or-self::div  → great-grandparent, grandparent, parent (3 ptu)
//span[@id='child2']/ancestor::div          → great-grandparent, grandparent, parent (3 ptu)
```

### Lỗi 3: Quên rằng `ancestor` có thể chọn nhiều phần tử

```xpath
// Nếu muốn chỉ lấy 1 div cha gần nhất → phải thêm [1]
//div[@id='child']/ancestor::div       → nhiều phần tử
//div[@id='child']/ancestor::div[1]    → CHỈ div cha gần nhất
```

---

## Tóm Tắt

```
BẮT ĐẦU TỪ: nút hiện tại
    ↑  ancestor::div       → tất cả div phía trên, KHÔNG gồm chính nó
    ↑  ancestor-or-self::div → tất cả div phía trên, CÓ gồm chính nó (nếu là div)

Ghi nhớ:
- "or-self" = thêm chính mình vào kết quả (nếu type khớp)
- ancestor = parent + parent của parent + ... (đi hết lên root)
- Thêm [1] nếu chỉ muốn lấy tổ tiên GẦN NHẤT
- Thêm điều kiện [@class='...'] để lọc tổ tiên cụ thể hơn
```
