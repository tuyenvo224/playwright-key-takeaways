# XPath Axes: `descendant` và `descendant-or-self` — Hướng Dẫn Chi Tiết

> Nguồn: https://material.playwrightvn.com/09-xpath-descendant-or-self.html

---

## Tổng Quan

Hai axis này đều dùng để **đi xuống dưới cây DOM** — từ một element đang chọn, tìm tất cả các phần tử nằm bên trong nó ở mọi độ sâu (con, cháu, chắt...).

| Axis | Mô tả | Bao gồm chính nó? |
|------|-------|:-----------------:|
| `descendant` | Chọn tất cả hậu duệ (con, cháu...) của nút hiện tại | Không |
| `descendant-or-self` | Chọn tất cả hậu duệ + chính nút hiện tại | Có (nếu type khớp) |

---

## Cú Pháp

```xpath
//tagBắtĐầu[@attribute='value']/descendant::tagMuốnChọn
//tagBắtĐầu[@attribute='value']/descendant-or-self::tagMuốnChọn
```

**Giải thích từng phần:**
- `//tagBắtĐầu[@attribute='value']` — xác định nút xuất phát (nút hiện tại)
- `/descendant::` — trục điều hướng: đi xuống, tìm ở **mọi độ sâu**
- `tagMuốnChọn` — loại thẻ muốn lọc (`div`, `span`, `p`, `*`...)

---

## So Sánh `descendant` vs `child`

Trước khi đi vào chi tiết, cần hiểu rõ sự khác biệt cốt lõi:

```
[parent]
  ├── [child1]        ← child::* và descendant::* đều chọn
  │     └── [grandchild1]   ← CHỈ descendant::* mới chọn
  └── [child2]        ← child::* và descendant::* đều chọn
        └── [grandchild2]   ← CHỈ descendant::* mới chọn
```

| Axis | Chọn con trực tiếp | Chọn cháu, chắt... |
|------|:-----------------:|:-----------------:|
| `child` | Có | **Không** |
| `descendant` | Có | **Có** |

> **Quy tắc nhớ:** `descendant` giống như `//` — đi sâu vào **bất kỳ độ sâu nào**.

---

## Phần 1: `descendant`

### Định nghĩa

> Chọn **tất cả hậu duệ** của nút hiện tại — bao gồm con, cháu, chắt... ở mọi cấp độ — **không bao gồm** chính nút đó.

---

### Ví dụ 1 — Tìm `div` con trong `div` cha

**HTML:**
```html
<div id="parent1">
    <div id="child1">Nội dung 1</div>
    <div id="child2">Nội dung 2</div>
</div>
```

**XPath:**
```xpath
//div[@id='parent1']/descendant::div
```

**Sơ đồ điều hướng:**
```
[div#parent1]       ← điểm xuất phát (KHÔNG được chọn)
  ├── [div#child1]  ← được chọn (hậu duệ cấp 1)
  └── [div#child2]  ← được chọn (hậu duệ cấp 1)
```

**Kết quả:** `div#child1` và `div#child2`

**Giải thích:**
- Xuất phát từ `div#parent1`
- Đi xuống tìm tất cả `<div>` bên trong
- `div#child1` và `div#child2` đều là hậu duệ → đều được chọn
- `div#parent1` là điểm xuất phát → **không được chọn**

---

### Ví dụ 2 — Tìm `span` cháu trong `div` ông

**HTML:**
```html
<div id="grandparent2">
    <div id="parent2">
        <span id="child2-1">Nội dung 2-1</span>
        <span id="child2-2">Nội dung 2-2</span>
    </div>
</div>
```

**XPath:**
```xpath
//div[@id='grandparent2']/descendant::span
```

**Sơ đồ điều hướng:**
```
[div#grandparent2]         ← điểm xuất phát (KHÔNG được chọn)
  └── [div#parent2]        ← là div, không phải span → KHÔNG được chọn
        ├── [span#child2-1] ← được chọn (hậu duệ cấp 2)
        └── [span#child2-2] ← được chọn (hậu duệ cấp 2)
```

**Kết quả:** `span#child2-1` và `span#child2-2`

**Giải thích:**
- Xuất phát từ `div#grandparent2`
- `descendant::span` → tìm tất cả `<span>` trong nó ở mọi độ sâu
- `div#parent2` không phải `<span>` → không được chọn
- Hai `<span>` nằm sâu 2 cấp vẫn được tìm thấy — đây là điểm mạnh của `descendant`

**Điểm quan trọng:** `descendant` xuyên qua nhiều cấp lồng nhau, không bị giới hạn bởi độ sâu.

---

### Ví dụ 3 — Tìm `p` cháu trong `div` ông

**HTML:**
```html
<div id="grandparent3">
    <div id="parent3">
        <p id="child3-1">Nội dung 3-1</p>
        <p id="child3-2">Nội dung 3-2</p>
    </div>
</div>
```

**XPath:**
```xpath
//div[@id='grandparent3']/descendant::p
```

**Sơ đồ điều hướng:**
```
[div#grandparent3]       ← điểm xuất phát (KHÔNG được chọn)
  └── [div#parent3]      ← là div, không phải p → KHÔNG được chọn
        ├── [p#child3-1] ← được chọn (hậu duệ cấp 2)
        └── [p#child3-2] ← được chọn (hậu duệ cấp 2)
```

**Kết quả:** `p#child3-1` và `p#child3-2`

---

## Phần 2: `descendant-or-self`

### Định nghĩa

> Chọn **tất cả hậu duệ** của nút hiện tại **và cả chính nút hiện tại** (nếu nó khớp loại thẻ được chỉ định).

---

### Ví dụ 1 — Nút xuất phát **cùng loại** với thẻ tìm kiếm

**HTML:**
```html
<div id="parent1">
    <div id="child1">Nội dung 1</div>
    <div id="child2">Nội dung 2</div>
</div>
```

**XPath:**
```xpath
//div[@id='parent1']/descendant-or-self::div
```

**Sơ đồ điều hướng:**
```
[div#parent1]       ← CŨNG được chọn (chính nút, là div)
  ├── [div#child1]  ← được chọn (hậu duệ cấp 1)
  └── [div#child2]  ← được chọn (hậu duệ cấp 1)
```

**Kết quả:** `div#parent1`, `div#child1`, `div#child2` **(3 phần tử)**

**So sánh với `descendant` ở ví dụ 1:**

| XPath | Kết quả |
|-------|---------|
| `//div[@id='parent1']/descendant::div` | `child1`, `child2` (2 phần tử) |
| `//div[@id='parent1']/descendant-or-self::div` | `parent1`, `child1`, `child2` **(3 phần tử)** |

---

### Ví dụ 2 — Nút xuất phát **khác loại** với thẻ tìm kiếm

**HTML:**
```html
<div id="grandparent2">
    <div id="parent2">
        <span id="child2-1">Nội dung 2-1</span>
        <span id="child2-2">Nội dung 2-2</span>
    </div>
</div>
```

**XPath:**
```xpath
//div[@id='grandparent2']/descendant-or-self::span
```

**Sơ đồ điều hướng:**
```
[div#grandparent2]          ← là div, không phải span → KHÔNG được chọn
  └── [div#parent2]         ← là div, không phải span → KHÔNG được chọn
        ├── [span#child2-1] ← được chọn (hậu duệ + là span)
        └── [span#child2-2] ← được chọn (hậu duệ + là span)
```

**Kết quả:** `span#child2-1` và `span#child2-2` (giống hệt `descendant::span`)

**Giải thích quan trọng:**
- Dù dùng `descendant-or-self`, `div#grandparent2` vẫn **không được chọn**
- Lý do: `descendant-or-self::span` chỉ lấy nút loại `<span>` — kể cả chính nó
- `grandparent2` là `<div>`, không phải `<span>` → bị loại ra
- Kết quả giống hệt `descendant::span`

---

### Ví dụ 3 — Nút xuất phát **khác loại** với thẻ tìm kiếm (p)

**HTML:**
```html
<div id="grandparent3">
    <div id="parent3">
        <p id="child3-1">Nội dung 3-1</p>
        <p id="child3-2">Nội dung 3-2</p>
    </div>
</div>
```

**XPath:**
```xpath
//div[@id='grandparent3']/descendant-or-self::p
```

**Kết quả:** `p#child3-1` và `p#child3-2` (giống hệt `descendant::p`)

---

## Phân Biệt Rõ Hai Axis

### Quy tắc cốt lõi

```
descendant::tagX         → chỉ chọn hậu duệ là tagX, BỎ nút hiện tại
descendant-or-self::tagX → chọn hậu duệ là tagX + chính nút nếu nó là tagX
```

### Bảng so sánh theo tình huống

| Tình huống | `descendant::div` | `descendant-or-self::div` |
|-----------|:-----------------:|:-------------------------:|
| Nút hiện tại là `div` | Không chọn nó | **Chọn nó** |
| Nút hiện tại là `span` | Không chọn nó | Không chọn nó |
| Các `div` con bên trong | Chọn hết | Chọn hết |
| Các `div` cháu bên trong | Chọn hết | Chọn hết |

### Sơ đồ tổng quát

```
descendant-or-self::div:          descendant::div:
  [div]  ← chọn (chính nút)         [div]  ← KHÔNG chọn
    |                                  |
  [div]  ← chọn                     [div]  ← chọn
    |                                  |
  [div]  ← chọn                     [div]  ← chọn
```

---

## Mối Liên Hệ Với `//` Trong XPath

Đây là kiến thức nâng cao nhưng rất quan trọng:

> **`//div` thực ra là viết tắt của `/descendant-or-self::node()/div`**

Nghĩa là khi bạn viết:
```xpath
//div[@id='parent']//span
```

Thực chất nó tương đương với:
```xpath
//div[@id='parent']/descendant-or-self::node()/span
```

Đó là lý do tại sao `//` có thể tìm element ở **bất kỳ độ sâu nào** trong cây DOM.

**Ví dụ so sánh:**
```xpath
//div[@id='parent']/span           → CHỈ span là con TRỰC TIẾP của div#parent
//div[@id='parent']//span          → span ở BẤT KỲ độ sâu nào bên trong div#parent
//div[@id='parent']/descendant::span → tương đương với dòng trên
```

---

## Ứng Dụng Thực Tế Trong Playwright

### 1. Tìm tất cả input bên trong một form (mọi độ sâu)

```typescript
// Dù input nằm trong div > fieldset > div, vẫn tìm được
const inputs = page.locator("//form[@id='register']/descendant::input");
const count = await inputs.count();
```

### 2. Lấy tất cả text hiển thị trong một section

```typescript
// Lấy tất cả thẻ p và span trong section
const textElements = page.locator("//section[@class='content']/descendant::*[self::p or self::span]");
```

### 3. Kiểm tra tất cả button trong một component

```typescript
// Đảm bảo không có button disabled trong card
const buttons = page.locator("//div[@class='card']/descendant::button");
for (const btn of await buttons.all()) {
    await expect(btn).not.toBeDisabled();
}
```

### 4. Dùng `descendant-or-self` để chọn container và toàn bộ nội dung

```typescript
// Chọn cả div#wrapper và tất cả div con cháu
const allDivs = page.locator("//div[@id='wrapper']/descendant-or-self::div");
```

### 5. Tìm element ẩn sâu nhiều cấp

```typescript
// HTML: form > div.row > div.col > div.field > input
// Không cần biết cấu trúc lồng nhau, descendant tìm được
const emailInput = page.locator("//form/descendant::input[@type='email']");
await emailInput.fill("test@example.com");
```

### 6. Kết hợp với điều kiện để lọc hậu duệ cụ thể

```typescript
// Tìm tất cả link có class 'active' trong nav
const activeLinks = page.locator("//nav/descendant::a[contains(@class,'active')]");

// Tìm tất cả li chứa text 'Error' trong ul bất kỳ
const errorItems = page.locator("//ul/descendant::li[contains(text(),'Error')]");
```

### 7. So sánh `//` với `descendant::` trong Playwright

```typescript
// Hai cách này cho kết quả giống nhau
const way1 = page.locator("//div[@id='parent']//button");
const way2 = page.locator("//div[@id='parent']/descendant::button");

// Dùng // khi viết nhanh, dùng descendant:: khi cần rõ ràng hơn
```

---

## Các Lỗi Thường Gặp

### Lỗi 1: Nhầm `descendant` với `child`

```xpath
// Ví dụ HTML:
// <div id="parent">
//   <div id="child">
//     <span id="grandchild">Text</span>
//   </div>
// </div>

// child chỉ lấy CẤP 1 trực tiếp
//div[@id='parent']/child::span       → KHÔNG tìm thấy (span không phải con trực tiếp)
//div[@id='parent']/child::div        → chỉ lấy div#child

// descendant đi sâu mọi cấp
//div[@id='parent']/descendant::span  → tìm thấy span#grandchild ✓
//div[@id='parent']/descendant::div   → tìm thấy div#child ✓
```

### Lỗi 2: Nghĩ `descendant-or-self` luôn trả về nhiều hơn `descendant` 1 phần tử

```xpath
// Chỉ nhiều hơn khi nút hiện tại KHỚP loại thẻ tìm kiếm

// Case 1: parent1 là div, tìm div → descendant-or-self nhiều hơn 1
//div[@id='parent1']/descendant::div          → child1, child2         (2 ptu)
//div[@id='parent1']/descendant-or-self::div  → parent1, child1, child2 (3 ptu)

// Case 2: grandparent2 là div, tìm span → kết quả BẰNG NHAU
//div[@id='grandparent2']/descendant::span          → child2-1, child2-2 (2 ptu)
//div[@id='grandparent2']/descendant-or-self::span  → child2-1, child2-2 (2 ptu)
```

### Lỗi 3: Dùng `descendant` khi chỉ cần `child`

```xpath
// Nếu cấu trúc HTML rõ ràng, chỉ cần lấy con trực tiếp
// → dùng child:: hoặc / để tránh chọn nhầm các cấp sâu hơn

// Sai (có thể chọn nhầm nhiều hơn mong muốn):
//ul[@id='menu']/descendant::li

// Đúng hơn nếu chỉ cần li trực tiếp:
//ul[@id='menu']/child::li
//ul[@id='menu']/li         ← viết tắt
```

### Lỗi 4: Nhầm thứ tự kết quả của `descendant`

```xpath
// descendant trả về theo thứ tự document (từ trên xuống)
// <div id="a">
//   <div id="b">
//     <div id="c"/>
//   </div>
//   <div id="d"/>
// </div>
//div[@id='a']/descendant::div → kết quả: b, c, d (theo thứ tự xuất hiện)
```

---

## Tóm Tắt

```
BẮT ĐẦU TỪ: nút hiện tại
    ↓  descendant::div       → tất cả div phía dưới (mọi độ sâu), KHÔNG gồm chính nút
    ↓  descendant-or-self::div → tất cả div phía dưới + chính nó (nếu là div)

So sánh nhanh:
  child::       → chỉ 1 cấp dưới
  descendant::  → tất cả các cấp dưới (= // trong XPath)

Ghi nhớ:
  - "or-self" = thêm chính mình vào kết quả (nếu type khớp)
  - descendant = child + child của child + ... (đi hết xuống leaf)
  - // trong XPath chính là descendant-or-self::node() rút gọn
  - Dùng [điều kiện] để lọc hậu duệ cụ thể hơn
```

---

## Bảng So Sánh Tổng Hợp 3 Axis Liên Quan

| Axis | Chiều | Phạm vi | Dùng khi nào |
|------|-------|---------|--------------|
| `child` | Xuống | Chỉ cấp 1 trực tiếp | Biết rõ element nằm ở cấp ngay dưới |
| `descendant` | Xuống | Mọi độ sâu | Cần tìm element nằm sâu, không biết chính xác cấp mấy |
| `descendant-or-self` | Xuống + chính nó | Mọi độ sâu + bản thân | Cần bao gồm cả nút hiện tại trong kết quả |
