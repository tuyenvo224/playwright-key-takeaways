# XPath Axes: `following` và `following-sibling` — Hướng Dẫn Chi Tiết

> Nguồn: https://material.playwrightvn.com/10-xpath-following-sibling.html

---

## Tổng Quan

Cả hai axis này đều dùng để **đi về phía sau** (forward) trong cây DOM — tìm các phần tử xuất hiện sau nút hiện tại. Điểm khác nhau cốt lõi nằm ở **phạm vi tìm kiếm**:

| Axis | Mô tả | Phạm vi |
|------|-------|---------|
| `following` | Chọn tất cả mọi thứ sau thẻ đóng của nút hiện tại | Toàn bộ document |
| `following-sibling` | Chọn tất cả anh/em sau nút hiện tại | Chỉ cùng cấp (cùng cha) |

---

## Cú Pháp

```xpath
//tagBắtĐầu[@attribute='value']/following::tagMuốnChọn
//tagBắtĐầu[@attribute='value']/following-sibling::tagMuốnChọn
```

**Ví dụ:**
```xpath
//div[@id='current']/following::div
//div[@id='current']/following-sibling::div
//div[@id='current']/following-sibling::*          ← tất cả loại thẻ cùng cấp phía sau
//div[@id='current']/following-sibling::div[1]     ← chỉ sibling đầu tiên phía sau
```

---

## Hình Dung Vị Trí Trong Cây DOM

```
<body>
  <div id="wrapper">
    <div id="before">...</div>        ← phía trước current
    <div id="current">...</div>       ← nút hiện tại
    <div id="sibling-1">...</div>     ← following-sibling (cùng wrapper)
    <div id="sibling-2">...</div>     ← following-sibling (cùng wrapper)
  </div>
  <div id="outside-1">...</div>       ← following (ngoài wrapper, sau current)
  <div id="outside-2">...</div>       ← following (ngoài wrapper, sau current)
</body>
```

```
following-sibling → chỉ [sibling-1] và [sibling-2]
following         → [sibling-1], [sibling-2], [outside-1], [outside-2] (tất cả phía sau)
```

---

## Phần 1: `following`

### Định nghĩa

> Chọn **tất cả các nút xuất hiện sau thẻ đóng** của nút hiện tại trong toàn bộ document — không giới hạn cùng cấp, xuyên qua các nhánh cây khác.

**Điều kiện để một nút được chọn bởi `following`:**
1. Xuất hiện **sau** nút hiện tại trong document (theo thứ tự đọc từ trên xuống)
2. **Không** là hậu duệ (descendant) của nút hiện tại
3. **Không** là tổ tiên (ancestor) của nút hiện tại

---

### Ví dụ 1 — Nút hiện tại là `div`, tìm `div` phía sau

**HTML:**
```html
<div id="wrapper1">
    <div id="current1">Nội dung hiện tại 1</div>
    <div id="sibling1-1">Anh chị em sau 1-1</div>   <!-- cùng cấp, trong wrapper -->
</div>
<div id="following1-1">Nội dung sau 1-1</div>        <!-- ngoài wrapper -->
<div id="following1-2">Nội dung sau 1-2</div>        <!-- ngoài wrapper -->
```

**XPath:**
```xpath
//div[@id='current1']/following::div
```

**Sơ đồ điều hướng:**
```
[div#wrapper1]
  ├── [div#current1]      ← điểm xuất phát
  └── [div#sibling1-1]    ← sau current1, nhưng nằm trong wrapper → được chọn
[div#following1-1]        ← sau wrapper1 đóng → được chọn
[div#following1-2]        ← sau wrapper1 đóng → được chọn
```

**Kết quả:** `div#sibling1-1`, `div#following1-1`, `div#following1-2`

**Giải thích:**
- `following::div` tìm tất cả `<div>` xuất hiện **sau thẻ đóng `</div>` của `current1`**
- `div#sibling1-1` tuy cùng cấp với `current1` nhưng xuất hiện sau → **được chọn**
- `div#following1-1` và `div#following1-2` ở ngoài `wrapper1`, sau `current1` → **được chọn**
- `div#wrapper1` là tổ tiên của `current1` → **không được chọn**

---

### Ví dụ 2 — Nút hiện tại là `span`, tìm `div` phía sau

**HTML:**
```html
<div id="wrapper2">
    <span id="current2">Nội dung hiện tại 2</span>
    <div id="sibling2-1">Anh chị em sau 2-1</div>   <!-- cùng cấp, trong wrapper -->
</div>
<div id="following2-1">Nội dung sau 2-1</div>
<div id="following2-2">Nội dung sau 2-2</div>
```

**XPath:**
```xpath
//span[@id='current2']/following::div
```

**Sơ đồ điều hướng:**
```
[div#wrapper2]
  ├── [span#current2]     ← điểm xuất phát (là span)
  └── [div#sibling2-1]   ← sau current2, là div → được chọn
[div#following2-1]        ← sau wrapper2 đóng → được chọn
[div#following2-2]        ← sau wrapper2 đóng → được chọn
```

**Kết quả:** `div#sibling2-1`, `div#following2-1`, `div#following2-2`

**Điểm quan trọng:** Nút xuất phát (`span`) có thể **khác loại thẻ** với loại thẻ tìm kiếm (`div`). `following` vẫn tìm đúng tất cả `<div>` xuất hiện sau `span#current2`.

---

### Ví dụ 3 — Nút hiện tại là `p`, tìm `span` phía sau

**HTML:**
```html
<div id="wrapper3">
    <p id="current3">Nội dung hiện tại 3</p>
    <span id="sibling3-1">Anh chị em sau 3-1</span>   <!-- cùng cấp, trong wrapper -->
</div>
<span id="following3-1">Nội dung sau 3-1</span>
<span id="following3-2">Nội dung sau 3-2</span>
```

**XPath:**
```xpath
//p[@id='current3']/following::span
```

**Kết quả:** `span#sibling3-1`, `span#following3-1`, `span#following3-2`

---

## Phần 2: `following-sibling`

### Định nghĩa

> Chọn **tất cả các phần tử cùng cấp (sibling) xuất hiện sau** nút hiện tại — nghĩa là cùng cha (parent), cùng mức lồng nhau, và nằm ở phía sau trong DOM.

**Điều kiện để một nút được chọn bởi `following-sibling`:**
1. Có **cùng cha** (parent) với nút hiện tại
2. Xuất hiện **sau** nút hiện tại trong DOM
3. Khớp loại thẻ được chỉ định

---

### Ví dụ 1 — Tìm `div` sibling phía sau trong cùng wrapper

**HTML:**
```html
<div id="wrapper1">
    <div id="current1">Nội dung hiện tại 1</div>
    <div id="sibling1-1">Anh chị em sau 1-1</div>
    <div id="sibling1-2">Anh chị em sau 1-2</div>
</div>
```

**XPath:**
```xpath
//div[@id='current1']/following-sibling::div
```

**Sơ đồ điều hướng:**
```
[div#wrapper1]  ← cha chung
  ├── [div#current1]    ← điểm xuất phát
  ├── [div#sibling1-1]  ← cùng cha, sau current → được chọn
  └── [div#sibling1-2]  ← cùng cha, sau current → được chọn
```

**Kết quả:** `div#sibling1-1` và `div#sibling1-2`

**So sánh với `following` ở ví dụ 1:**

| XPath | Kết quả |
|-------|---------|
| `//div[@id='current1']/following::div` | `sibling1-1`, `following1-1`, `following1-2` (3 ptu) |
| `//div[@id='current1']/following-sibling::div` | `sibling1-1`, `sibling1-2` (2 ptu, chỉ trong wrapper) |

---

### Ví dụ 2 — Tìm `span` sibling phía sau

**HTML:**
```html
<div id="wrapper2">
    <span id="current2">Nội dung hiện tại 2</span>
    <span id="sibling2-1">Anh chị em sau 2-1</span>
    <span id="sibling2-2">Anh chị em sau 2-2</span>
</div>
```

**XPath:**
```xpath
//span[@id='current2']/following-sibling::span
```

**Sơ đồ điều hướng:**
```
[div#wrapper2]   ← cha chung
  ├── [span#current2]    ← điểm xuất phát
  ├── [span#sibling2-1]  ← cùng cha, sau current → được chọn
  └── [span#sibling2-2]  ← cùng cha, sau current → được chọn
```

**Kết quả:** `span#sibling2-1` và `span#sibling2-2`

---

### Ví dụ 3 — Tìm `p` sibling phía sau

**HTML:**
```html
<div id="wrapper3">
    <p id="current3">Nội dung hiện tại 3</p>
    <p id="sibling3-1">Anh chị em sau 3-1</p>
    <p id="sibling3-2">Anh chị em sau 3-2</p>
</div>
```

**XPath:**
```xpath
//p[@id='current3']/following-sibling::p
```

**Kết quả:** `p#sibling3-1` và `p#sibling3-2`

---

## Phân Biệt Rõ Hai Axis

### Sơ đồ tổng quát

```
<body>
  <section>                          ← tổ tiên, không được chọn
    <div id="A">                     ← tổ tiên, không được chọn
      <p id="current"> ← ĐIỂM XUẤT PHÁT
      <p id="B">       ← following-sibling ✓ | following ✓
      <p id="C">       ← following-sibling ✓ | following ✓
    </div>
    <div id="D">       ← following-sibling của A, không phải của current
                         KHÔNG phải following-sibling của current ✗
                         following ✓ (xuất hiện sau current trong doc)
      <p id="E">       ← following ✓ | following-sibling ✗
    </div>
  </section>
  <footer id="F">      ← following ✓ | following-sibling ✗
</body>
```

### Bảng so sánh chi tiết

| Nút | `following::p` | `following-sibling::p` |
|-----|:--------------:|:----------------------:|
| Sibling `p` cùng cha, phía sau | Có | Có |
| `p` trong nhánh khác, phía sau | Có | **Không** |
| `p` trong footer, phía sau | Có | **Không** |
| Tổ tiên của current | Không | Không |
| Con của current | Không | Không |

### Quy tắc cốt lõi

```
following-sibling = tìm trong phạm vi CỦA CHA (anh/em ruột phía sau)
following         = tìm trong phạm vi TOÀN BỘ DOCUMENT (tất cả phía sau)

following-sibling ⊂ following
(following-sibling là tập con của following)
```

---

## Dùng Index `[n]` Để Lấy Sibling Cụ Thể

```xpath
//div[@id='current']/following-sibling::div[1]   ← sibling ĐẦU TIÊN phía sau
//div[@id='current']/following-sibling::div[2]   ← sibling THỨ HAI phía sau
//div[@id='current']/following-sibling::div[last()] ← sibling CUỐI CÙNG phía sau
```

**Ví dụ HTML:**
```html
<div id="parent">
    <div id="current">Nội dung</div>
    <div id="next-1">Sibling 1</div>   ← following-sibling::div[1]
    <div id="next-2">Sibling 2</div>   ← following-sibling::div[2]
    <div id="next-3">Sibling 3</div>   ← following-sibling::div[3] / div[last()]
</div>
```

> **Lưu ý:** Index `[1]` trong `following-sibling` là phần tử **gần nhất** tính từ `current` về phía sau — khác với `preceding-sibling[1]` đếm theo chiều ngược lại.

---

## Ứng Dụng Thực Tế Trong Playwright

### 1. Tìm input ngay sau label (pattern cực phổ biến)

```typescript
// HTML: <label>Email</label> <input type="email" />
const emailInput = page.locator("//label[text()='Email']/following-sibling::input");
await emailInput.fill("test@example.com");
```

### 2. Tìm thông báo lỗi ngay sau input

```typescript
// HTML: <input name="password"/> <span class="error">Mật khẩu không đúng</span>
const errorMsg = page.locator(
  "//input[@name='password']/following-sibling::span[contains(@class,'error')]"
);
await expect(errorMsg).toBeVisible();
```

### 3. Lấy sibling kế tiếp (next sibling)

```typescript
// Lấy tab ngay sau tab đang active
const nextTab = page.locator("//li[contains(@class,'active')]/following-sibling::li[1]");
await nextTab.click();
```

### 4. Kiểm tra tất cả row sau row đang chọn trong bảng

```typescript
// Tất cả tr sau tr được highlight
const rowsAfter = page.locator("//tr[contains(@class,'selected')]/following-sibling::tr");
const count = await rowsAfter.count();
console.log(`Còn ${count} rows phía sau`);
```

### 5. Dùng `following` để tìm element ở section khác

```typescript
// Tìm tất cả div xuất hiện sau div#header trong toàn trang
const contentDivs = page.locator("//div[@id='header']/following::div[@class='content']");
```

### 6. Tìm button "Sửa" trong cùng hàng với tên cụ thể

```typescript
// HTML table: <td>John</td> <td><button>Sửa</button></td>
const editBtn = page.locator("//td[text()='John']/following-sibling::td//button[text()='Sửa']");
await editBtn.click();
```

### 7. Lấy giá trị ô kế tiếp trong table

```typescript
// Lấy giá trị ô ngay sau ô "Tên:"
const value = await page.locator("//td[text()='Tên:']/following-sibling::td[1]").textContent();
```

### 8. Kết hợp `following-sibling` với điều kiện phức tạp

```typescript
// Tìm span.badge ngay sau h3 có text "Thông báo"
const badge = page.locator(
  "//h3[contains(text(),'Thông báo')]/following-sibling::span[contains(@class,'badge')]"
);
const count = await badge.textContent();
```

---

## Các Lỗi Thường Gặp

### Lỗi 1: Dùng `following-sibling` khi cần `following`

```xpath
<!-- HTML: -->
<!-- <div id="current"/> nằm trong <section/> -->
<!-- <div id="target"/> nằm NGOÀI <section/>, sau section -->

<!-- SAI: following-sibling không tìm được vì khác cha -->
//div[@id='current']/following-sibling::div[@id='target']   → không có kết quả

<!-- ĐÚNG: dùng following để tìm toàn document -->
//div[@id='current']/following::div[@id='target']           → tìm thấy ✓
```

### Lỗi 2: Nhầm `following-sibling` với `following` khi chọn nhầm quá nhiều

```xpath
<!-- Nếu chỉ muốn lấy sibling cùng cấp mà dùng following -->
//li[@id='item2']/following::li   → lấy cả li trong các ul khác phía sau ✗

<!-- Đúng hơn: giới hạn chỉ cùng cấp -->
//li[@id='item2']/following-sibling::li  → chỉ li trong cùng ul ✓
```

### Lỗi 3: Quên rằng `following-sibling` phụ thuộc vào cùng cha

```xpath
<!-- HTML: -->
<!-- <div id="A"><span id="x"/></div> <div id="B"><span id="y"/></div> -->

<!-- span#x và span#y khác cha → KHÔNG phải sibling của nhau -->
//span[@id='x']/following-sibling::span   → KHÔNG tìm thấy span#y ✗

<!-- Phải dùng following -->
//span[@id='x']/following::span           → tìm thấy span#y ✓
```

### Lỗi 4: Nhầm index của `following-sibling`

```xpath
<!-- Index đếm từ current ra ngoài -->
<!-- [div.A] [div.B=current] [div.C] [div.D] [div.E] -->

//div[@id='B']/following-sibling::div[1]  → div.C (gần nhất)
//div[@id='B']/following-sibling::div[2]  → div.D
//div[@id='B']/following-sibling::div[3]  → div.E
```

---

## Tóm Tắt

```
                  [parent / wrapper]
                  /        |        \
    [before] [current] [sibling1] [sibling2]    ← tất cả cùng cấp
                  |
            following-sibling → [sibling1], [sibling2]

    [other-section]                              ← khác nhánh, phía sau
      └── [deep-child]

            following → [sibling1], [sibling2], [other-section], [deep-child]

Ghi nhớ nhanh:
  following-sibling = anh/em ruột phía sau (cùng cha)
  following         = tất cả phía sau trong document
  following-sibling ⊂ following (following-sibling là tập con của following)

Dùng [1] để lấy sibling kế tiếp gần nhất.
Dùng điều kiện [@class, text()] để lọc chính xác hơn.
```

---

## Bảng So Sánh 4 Axis Định Hướng Ngang

| Axis | Hướng | Phạm vi | Ví dụ dùng |
|------|-------|---------|-----------|
| `following-sibling` | Sau | Cùng cấp (cùng cha) | Input sau label, row sau row active |
| `preceding-sibling` | Trước | Cùng cấp (cùng cha) | Label trước input, tab trước tab active |
| `following` | Sau | Toàn document | Element ở section khác phía sau |
| `preceding` | Trước | Toàn document | Element ở section khác phía trước |
