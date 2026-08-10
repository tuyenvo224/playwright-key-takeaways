# XPath Axes: `preceding` và `preceding-sibling` — Hướng Dẫn Chi Tiết

> Nguồn: https://material.playwrightvn.com/11-xpath-preceding-sibling.html

---

## Tổng Quan

Cả hai axis này đều dùng để **đi về phía trước** (backward) trong cây DOM — tìm các phần tử xuất hiện **trước** nút hiện tại. Đây là bộ đôi đối xứng hoàn toàn với `following` / `following-sibling`.

| Axis | Mô tả | Phạm vi |
|------|-------|---------|
| `preceding` | Chọn tất cả nút xuất hiện trước nút hiện tại *(trừ tổ tiên, attribute, namespace)* | Toàn bộ document |
| `preceding-sibling` | Chọn tất cả anh/em trước nút hiện tại | Chỉ cùng cấp (cùng cha) |

---

## Cú Pháp

```xpath
//tagBắtĐầu[@attribute='value']/preceding::tagMuốnChọn
//tagBắtĐầu[@attribute='value']/preceding-sibling::tagMuốnChọn
```

**Ví dụ:**
```xpath
//div[@id='current']/preceding::div
//div[@id='current']/preceding-sibling::div
//div[@id='current']/preceding-sibling::*           ← tất cả loại thẻ cùng cấp phía trước
//div[@id='current']/preceding-sibling::div[1]      ← sibling gần nhất phía trước
//div[@id='current']/preceding-sibling::div[last()] ← sibling xa nhất phía trước
```

---

## Hình Dung Vị Trí Trong Cây DOM

```
<body>
  <div id="outside-1">...</div>    ← preceding ✓ | preceding-sibling ✗
  <div id="outside-2">...</div>    ← preceding ✓ | preceding-sibling ✗

  <div id="wrapper">               ← tổ tiên → KHÔNG được chọn bởi preceding
    <div id="sibling-before-1">    ← preceding ✓ | preceding-sibling ✓
    <div id="sibling-before-2">    ← preceding ✓ | preceding-sibling ✓
    <div id="current">...</div>    ← ĐIỂM XUẤT PHÁT
    <div id="sibling-after">       ← ở phía SAU → không thuộc preceding/preceding-sibling
  </div>

  <div id="outside-3">...</div>    ← ở phía SAU current → không thuộc preceding/preceding-sibling
</body>
```

```
preceding-sibling → [sibling-before-1], [sibling-before-2]
preceding         → [outside-1], [outside-2], [sibling-before-1], [sibling-before-2]
                    (tất cả phía trước, TRỪ tổ tiên)
```

---

## Phần 1: `preceding`

### Định nghĩa

> Chọn **tất cả các nút xuất hiện trước nút hiện tại** trong toàn bộ document —  
> **ngoại trừ:** tổ tiên (ancestor), attribute nodes, namespace nodes.

**Điều kiện để một nút được chọn bởi `preceding`:**
1. Xuất hiện **trước** nút hiện tại trong document (theo thứ tự đọc từ trên xuống)
2. **Không** là tổ tiên (cha, ông, cụ...) của nút hiện tại
3. **Không** là attribute hoặc namespace node

---

### Ví dụ 1 — Nút hiện tại là `div`, tìm `div` phía trước

**HTML:**
```html
<div id="preceding1-1">Nội dung trước 1-1</div>   <!-- ngoài wrapper, trước wrapper -->
<div id="preceding1-2">Nội dung trước 1-2</div>   <!-- ngoài wrapper, trước wrapper -->
<div id="wrapper1">                                <!-- tổ tiên → KHÔNG được chọn -->
    <div id="current1">Nội dung hiện tại 1</div>  <!-- điểm xuất phát -->
    <div id="sibling1-1">Anh chị em sau 1-1</div> <!-- phía SAU current → không thuộc preceding -->
</div>
```

**XPath:**
```xpath
//div[@id='current1']/preceding::div
```

**Sơ đồ điều hướng:**
```
[div#preceding1-1]  ← trước wrapper, không phải tổ tiên → được chọn
[div#preceding1-2]  ← trước wrapper, không phải tổ tiên → được chọn
[div#wrapper1]      ← tổ tiên của current1 → KHÔNG được chọn
  ├── [div#current1]   ← điểm xuất phát
  └── [div#sibling1-1] ← phía SAU current1 → KHÔNG được chọn
```

**Kết quả:** `div#preceding1-1` và `div#preceding1-2`

**Giải thích từng điểm:**
- `div#preceding1-1`, `div#preceding1-2` — nằm trước `current1` trong document, không phải tổ tiên → **được chọn**
- `div#wrapper1` — dù nằm trước `current1` nhưng là **tổ tiên** → **không được chọn**
- `div#sibling1-1` — nằm **sau** `current1` → không thuộc `preceding`

---

### Ví dụ 2 — Nút hiện tại là `span`, tìm `div` phía trước

**HTML:**
```html
<div id="preceding2-1">Nội dung trước 2-1</div>
<div id="preceding2-2">Nội dung trước 2-2</div>
<div id="wrapper2">
    <span id="current2">Nội dung hiện tại 2</span>
    <div id="sibling2-1">Anh chị em sau 2-1</div>
</div>
```

**XPath:**
```xpath
//span[@id='current2']/preceding::div
```

**Sơ đồ điều hướng:**
```
[div#preceding2-1]  ← trước wrapper2, không phải tổ tiên → được chọn
[div#preceding2-2]  ← trước wrapper2, không phải tổ tiên → được chọn
[div#wrapper2]      ← tổ tiên của current2 → KHÔNG được chọn
  ├── [span#current2]   ← điểm xuất phát (span)
  └── [div#sibling2-1]  ← phía SAU current2 → KHÔNG được chọn
```

**Kết quả:** `div#preceding2-1` và `div#preceding2-2`

**Điểm quan trọng:** Nút xuất phát (`span`) có thể **khác loại thẻ** với loại thẻ tìm kiếm (`div`) — `preceding` vẫn hoạt động bình thường.

---

### Ví dụ 3 — Nút hiện tại là `p`, tìm `span` phía trước

**HTML:**
```html
<span id="preceding3-1">Nội dung trước 3-1</span>
<span id="preceding3-2">Nội dung trước 3-2</span>
<div id="wrapper3">
    <p id="current3">Nội dung hiện tại 3</p>
    <span id="sibling3-1">Anh chị em sau 3-1</span>
</div>
```

**XPath:**
```xpath
//p[@id='current3']/preceding::span
```

**Sơ đồ điều hướng:**
```
[span#preceding3-1]  ← trước wrapper3, không phải tổ tiên → được chọn
[span#preceding3-2]  ← trước wrapper3, không phải tổ tiên → được chọn
[div#wrapper3]
  ├── [p#current3]      ← điểm xuất phát
  └── [span#sibling3-1] ← phía SAU current3 → KHÔNG được chọn
```

**Kết quả:** `span#preceding3-1` và `span#preceding3-2`

---

## Phần 2: `preceding-sibling`

### Định nghĩa

> Chọn **tất cả các phần tử cùng cấp (sibling) xuất hiện trước** nút hiện tại — nghĩa là cùng cha (parent), cùng mức lồng nhau, và nằm ở phía trước trong DOM.

---

### Ví dụ 1 — Tìm `div` sibling phía trước

**HTML:**
```html
<div id="wrapper1">
    <div id="sibling1-1">Anh chị em trước 1-1</div>   <!-- trước current -->
    <div id="sibling1-2">Anh chị em trước 1-2</div>   <!-- trước current -->
    <div id="current1">Nội dung hiện tại 1</div>       <!-- điểm xuất phát -->
    <div id="sibling1-3">Anh chị em sau 1-1</div>      <!-- SAU current → không chọn -->
</div>
```

**XPath:**
```xpath
//div[@id='current1']/preceding-sibling::div
```

**Sơ đồ điều hướng:**
```
[div#wrapper1]  ← cha chung
  ├── [div#sibling1-1]  ← cùng cha, TRƯỚC current → được chọn
  ├── [div#sibling1-2]  ← cùng cha, TRƯỚC current → được chọn
  ├── [div#current1]    ← điểm xuất phát
  └── [div#sibling1-3]  ← cùng cha nhưng SAU current → KHÔNG được chọn
```

**Kết quả:** `div#sibling1-1` và `div#sibling1-2`

**So sánh trực tiếp `preceding` vs `preceding-sibling`:**

| XPath | Kết quả |
|-------|---------|
| `//div[@id='current1']/preceding::div` | `preceding1-1`, `preceding1-2` (ngoài wrapper) |
| `//div[@id='current1']/preceding-sibling::div` | `sibling1-1`, `sibling1-2` (trong wrapper) |

> Trong ví dụ của trang gốc, hai axis này trả về **các element khác nhau hoàn toàn** vì cấu trúc HTML được dàn dựng khác nhau — preceding ví dụ đặt element ngoài wrapper, preceding-sibling ví dụ đặt element trong wrapper.

---

### Ví dụ 2 — Tìm `span` sibling phía trước

**HTML:**
```html
<div id="wrapper2">
    <span id="sibling2-1">Anh chị em trước 2-1</span>
    <span id="sibling2-2">Anh chị em trước 2-2</span>
    <span id="current2">Nội dung hiện tại 2</span>
    <span id="sibling2-3">Anh chị em sau 2-1</span>
</div>
```

**XPath:**
```xpath
//span[@id='current2']/preceding-sibling::span
```

**Sơ đồ điều hướng:**
```
[div#wrapper2]  ← cha chung
  ├── [span#sibling2-1]  ← cùng cha, TRƯỚC current → được chọn
  ├── [span#sibling2-2]  ← cùng cha, TRƯỚC current → được chọn
  ├── [span#current2]    ← điểm xuất phát
  └── [span#sibling2-3]  ← SAU current → KHÔNG được chọn
```

**Kết quả:** `span#sibling2-1` và `span#sibling2-2`

---

### Ví dụ 3 — Tìm `p` sibling phía trước

**HTML:**
```html
<div id="wrapper3">
    <p id="sibling3-1">Anh chị em trước 3-1</p>
    <p id="sibling3-2">Anh chị em trước 3-2</p>
    <p id="current3">Nội dung hiện tại 3</p>
    <p id="sibling3-3">Anh chị em sau 3-1</p>
</div>
```

**XPath:**
```xpath
//p[@id='current3']/preceding-sibling::p
```

**Sơ đồ điều hướng:**
```
[div#wrapper3]  ← cha chung
  ├── [p#sibling3-1]  ← TRƯỚC current → được chọn
  ├── [p#sibling3-2]  ← TRƯỚC current → được chọn
  ├── [p#current3]    ← điểm xuất phát
  └── [p#sibling3-3]  ← SAU current → KHÔNG được chọn
```

**Kết quả:** `p#sibling3-1` và `p#sibling3-2`

---

## Phân Biệt Rõ Hai Axis

### Sơ đồ tổng quát đầy đủ

```
[div#A]           ← preceding ✓  |  preceding-sibling ✗  (khác cha của current)
  └── [p#B]       ← preceding ✓  |  preceding-sibling ✗  (khác cha của current)

[div#wrapper]     ← TỔ TIÊN → preceding ✗  |  preceding-sibling ✗
  ├── [div#C]     ← preceding ✓  |  preceding-sibling ✓  (cùng cha, trước current)
  ├── [div#D]     ← preceding ✓  |  preceding-sibling ✓  (cùng cha, trước current)
  ├── [div#current]  ← ĐIỂM XUẤT PHÁT
  └── [div#E]     ← SAU current → preceding ✗  |  preceding-sibling ✗

[div#F]           ← SAU current → preceding ✗  |  preceding-sibling ✗
```

### Bảng so sánh chi tiết

| Vị trí nút | `preceding::div` | `preceding-sibling::div` |
|------------|:----------------:|:------------------------:|
| Div ngoài, trước wrapper | Có | **Không** |
| Div anh/em, trước current (cùng cha) | Có | **Có** |
| Wrapper (tổ tiên) | **Không** | **Không** |
| Div anh/em, sau current | Không | Không |
| Div ngoài, sau current | Không | Không |

### Quy tắc cốt lõi

```
preceding-sibling = anh/em ruột phía trước (cùng cha)
preceding         = tất cả phía trước trong document (trừ tổ tiên)

preceding-sibling ⊂ preceding
(preceding-sibling là tập con của preceding)
```

---

## Đặc Điểm Quan Trọng: Thứ Tự Index `[n]`

> **Cảnh báo:** Index trong `preceding-sibling` hoạt động **ngược chiều** so với `following-sibling`!

```
[div#A]  [div#B]  [div#C]  [div#current]
                                 |
preceding-sibling::div[1]  → [div#C]  ← gần nhất (đếm từ current về trước)
preceding-sibling::div[2]  → [div#B]
preceding-sibling::div[3]  → [div#A]  ← xa nhất
preceding-sibling::div[last()] → [div#A]  ← cũng là xa nhất (đầu tiên trong DOM)
```

**Ví dụ HTML cụ thể:**
```html
<div id="parent">
    <div id="first">Đầu tiên</div>      <!-- preceding-sibling::div[3] -->
    <div id="second">Thứ hai</div>      <!-- preceding-sibling::div[2] -->
    <div id="third">Thứ ba</div>        <!-- preceding-sibling::div[1] ← gần nhất -->
    <div id="current">Hiện tại</div>    <!-- điểm xuất phát -->
</div>
```

```xpath
//div[@id='current']/preceding-sibling::div[1]      → div#third  (gần nhất)
//div[@id='current']/preceding-sibling::div[last()]  → div#first  (xa nhất / đầu DOM)
```

**So sánh với `following-sibling`:**
```
[div#current]  [div#X]  [div#Y]  [div#Z]
                  |
following-sibling::div[1]  → [div#X]  ← gần nhất (đếm từ current ra phía sau)
following-sibling::div[2]  → [div#Y]
```

> **Tóm lại:** Cả hai đều đếm `[1]` = **gần nhất với current**. Chỉ khác hướng đếm (trước/sau).

---

## Ứng Dụng Thực Tế Trong Playwright

### 1. Tìm label của một input (pattern cực phổ biến)

```typescript
// HTML: <label>Tên đăng nhập</label> <input id="username"/>
// Từ input, tìm label đứng trước nó
const label = page.locator("//input[@id='username']/preceding-sibling::label");
const labelText = await label.textContent();
```

### 2. Lấy tab ngay trước tab đang active

```typescript
// HTML: <li>Tab1</li> <li class="active">Tab2</li>
const prevTab = page.locator("//li[contains(@class,'active')]/preceding-sibling::li[1]");
await prevTab.click();
```

### 3. Kiểm tra heading của section trước khi điền form

```typescript
// Xác nhận form đang ở bước đúng dựa vào heading trước nó
const heading = page.locator(
  "//form[@id='checkout-form']/preceding-sibling::h2"
);
await expect(heading).toHaveText("Bước 2: Thông tin thanh toán");
```

### 4. Lấy ô label trong table biết ô value

```typescript
// HTML table: <td class="label">Họ tên:</td> <td class="value">Nguyễn Văn A</td>
// Từ value, tìm label đứng trước
const labelCell = page.locator(
  "//td[@class='value' and text()='Nguyễn Văn A']/preceding-sibling::td[@class='label']"
);
const labelText = await labelCell.textContent(); // "Họ tên:"
```

### 5. Đếm số bước đã qua trong stepper/wizard

```typescript
// HTML: <li class="done">B1</li> <li class="done">B2</li> <li class="active">B3</li>
const completedSteps = page.locator(
  "//li[contains(@class,'active')]/preceding-sibling::li[contains(@class,'done')]"
);
const count = await completedSteps.count();
console.log(`Đã hoàn thành ${count} bước`);
```

### 6. Dùng `preceding` để tìm element ở section khác phía trước

```typescript
// Tìm tất cả div.card xuất hiện trước div#current-section trong toàn trang
const prevCards = page.locator(
  "//div[@id='current-section']/preceding::div[contains(@class,'card')]"
);
```

### 7. Xác nhận thứ tự các item trong list

```typescript
// Kiểm tra "B" xuất hiện trước "C" trong cùng một list
const bBeforeC = page.locator(
  "//li[text()='C']/preceding-sibling::li[text()='B']"
);
await expect(bBeforeC).toHaveCount(1); // phải tồn tại đúng 1 phần tử
```

### 8. Kết hợp `preceding-sibling` với `ancestor` để điều hướng phức tạp

```typescript
// Từ một ô trong bảng, lấy header của cột đó
// HTML: <thead><tr><th>Tên</th><th>Tuổi</th></tr></thead>
//        <tbody><tr><td>An</td><td>25</td></tr></tbody>
// Đứng tại td "25", muốn lấy header "Tuổi"

const columnIndex = await page.locator("//td[text()='25']")
  .evaluate(el => [...el.parentElement.children].indexOf(el) + 1);

const header = page.locator(`//thead//th[${columnIndex}]`);
```

---

## Các Lỗi Thường Gặp

### Lỗi 1: Dùng `preceding-sibling` khi element ở nhánh cây khác

```xpath
<!-- HTML: -->
<!-- <div id="target"/> nằm TRƯỚC wrapper, NGOÀI wrapper -->
<!-- <div id="wrapper"><div id="current"/></div> -->

<!-- SAI: preceding-sibling không tìm được vì khác cha -->
//div[@id='current']/preceding-sibling::div[@id='target']   → không có kết quả ✗

<!-- ĐÚNG: dùng preceding -->
//div[@id='current']/preceding::div[@id='target']           → tìm thấy ✓
```

### Lỗi 2: Nhầm `preceding` chọn được tổ tiên

```xpath
<!-- preceding KHÔNG bao gồm tổ tiên! -->
<div id="grandparent">
  <div id="parent">
    <div id="current"/>
  </div>
</div>

//div[@id='current']/preceding::div
→ KHÔNG chọn div#parent, KHÔNG chọn div#grandparent (đây là tổ tiên)
→ Muốn lấy tổ tiên phải dùng: ancestor::div
```

### Lỗi 3: Nhầm index `[1]` là element đầu tiên trong DOM

```xpath
<!-- preceding-sibling[1] = GẦN NHẤT với current (không phải đầu DOM) -->
<div id="parent">
    <div id="A"/>   ← preceding-sibling::div[last()] hoặc [3]
    <div id="B"/>   ← preceding-sibling::div[2]
    <div id="C"/>   ← preceding-sibling::div[1]  ← GẦN NHẤT
    <div id="current"/>
</div>

//div[@id='current']/preceding-sibling::div[1]  → div#C  (không phải div#A!)
```

### Lỗi 4: Dùng `preceding` khi chỉ cần `preceding-sibling`, chọn nhầm quá nhiều

```xpath
<!-- Chỉ muốn lấy li trước li active trong cùng ul -->
//li[@class='active']/preceding::li   → lấy cả li trong các ul khác phía trước ✗

<!-- Đúng hơn -->
//li[@class='active']/preceding-sibling::li  → chỉ li trong cùng ul ✓
```

---

## Bộ Đôi Đối Xứng: Preceding ↔ Following

```
                    TRƯỚC                        SAU
            ──────────────────            ──────────────────
Toàn doc:   preceding::div      [CURRENT]   following::div
Cùng cấp:  preceding-sibling::div          following-sibling::div
Lên trên:   ancestor::div                  (không có đối xứng)
Xuống dưới: (không có đối xứng)            descendant::div
```

Mọi đặc điểm của `following` / `following-sibling` đều có phiên bản đối xứng trong `preceding` / `preceding-sibling`, chỉ đổi chiều hướng.

---

## Tóm Tắt

```
BẮT ĐẦU TỪ: nút hiện tại
    ← preceding-sibling::div  → anh/em ruột phía TRƯỚC (cùng cha)
    ← preceding::div          → tất cả phía TRƯỚC trong document (trừ tổ tiên)

Ghi nhớ quan trọng:
  ✓ preceding      KHÔNG bao gồm tổ tiên (dùng ancestor để lấy tổ tiên)
  ✓ preceding-sibling  chỉ lấy cùng cấp (cùng cha)
  ✓ Index [1]      = phần tử GẦN NHẤT với current (không phải đầu DOM)
  ✓ preceding-sibling ⊂ preceding (là tập con)
  ✓ Đối xứng hoàn toàn với following / following-sibling (chỉ đổi chiều)
```

---

## Bảng Tổng Hợp 4 Axis Định Hướng Ngang

| Axis | Hướng | Phạm vi | Bao gồm tổ tiên? | Ví dụ thực tế |
|------|-------|---------|:-----------------:|---------------|
| `preceding-sibling` | Trước | Cùng cấp (cùng cha) | Không | Label trước input, tab trước tab active |
| `following-sibling` | Sau | Cùng cấp (cùng cha) | Không | Input sau label, error sau input |
| `preceding` | Trước | Toàn document | **Không** | Element ở section khác phía trước |
| `following` | Sau | Toàn document | Không | Element ở section khác phía sau |
