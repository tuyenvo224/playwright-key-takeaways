# Lesson 10 - Playwright POM nâng cao: Locator động, Assertion & Design Pattern

> Tổng hợp kiến thức từ 4 bài tập thực hành: Personal Notes (search), User Registration (bảng dữ liệu), Shopping Cart (số lượng & tổng tiền), To-Do List (viewport & DOM).

---

## Mục lục

1. [Locator - Khái niệm nền tảng](#1-locator---khái-niệm-nền-tảng)
2. [Đọc dữ liệu từ phần tử](#2-đọc-dữ-liệu-từ-phần-tử)
3. [CSS Selector vs XPath](#3-css-selector-vs-xpath)
4. [Async/Await & TypeScript trong Playwright](#4-asyncawait--typescript-trong-playwright)
5. [Assertion (expect) - Web-first assertion](#5-assertion-expect---web-first-assertion)
6. [Design pattern khi viết POM](#6-design-pattern-khi-viết-pom)
7. [Các lỗi thường gặp & cách debug](#7-các-lỗi-thường-gặp--cách-debug)
8. [Tổng hợp qua 4 bài tập thực hành](#8-tổng-hợp-qua-4-bài-tập-thực-hành)

---

## 1. Locator - Khái niệm nền tảng

### 1.1. Locator là "công thức tìm kiếm", không phải phần tử thật

`page.locator(...)` **không** đi tìm phần tử ngay lập tức — nó chỉ tạo ra 1 object mô tả "cách tìm", việc tìm kiếm thật sự trên DOM chỉ xảy ra khi bạn gọi 1 **action** (`.click()`, `.fill()`...) hoặc **assertion** (`expect(locator).toBeVisible()`...) lên nó.

```ts
const locator = this.page.locator("//li[@id='item']"); // chưa query DOM, chỉ tạo "công thức"
await locator.click(); // lúc này mới thực sự tìm phần tử và click
```

➡ Đây là lý do các hàm như `.locator()`, `.filter()`, `.nth()` **không cần `await`** — chỉ có các action/assertion thật sự tương tác với trang mới cần `await`.

### 1.2. `.all()` - lấy snapshot danh sách phần tử

Khi 1 locator khớp với **nhiều phần tử** (ví dụ nhiều `<li>` trong danh sách), `.all()` trả về 1 mảng các Locator riêng lẻ, mỗi phần tử ứng với 1 node cụ thể **tại thời điểm gọi**.

```ts
const notes = await this.page.locator("//ul[@id='notes-list']/li").all();
// notes là mảng Locator, mỗi phần tử là 1 <li> cụ thể
```

⚠️ **Lưu ý quan trọng:** `.all()` chụp lại (snapshot) danh sách tại đúng thời điểm gọi. Nếu sau đó DOM thay đổi (thêm/xóa phần tử), mảng đã lấy **không tự cập nhật lại**.

### 1.3. `.nth()`, `.first()`, `.last()` - chọn theo vị trí

```ts
cells.nth(0)   // phần tử đầu tiên (index 0)
cells.nth(1)   // phần tử thứ 2
rows.first()   // phần tử đầu tiên trong danh sách khớp
rows.last()    // phần tử cuối cùng trong danh sách khớp
```

**Ví dụ thực tế:** bảng đăng ký user, dòng `<tr>` mới luôn được `appendChild()` vào **cuối** `<tbody>` → dùng `.last()` để luôn lấy đúng dòng vừa đăng ký, bất kể trước đó bảng có bao nhiêu dòng khác:

```ts
async getLastRegisteredRowText(): Promise<string> {
    const lastRow = this.page.locator("//table[@id='userTable']/tbody/tr").last();
    return (await lastRow.innerText()).toLowerCase();
}
```

### 1.4. Locator chaining - thu hẹp phạm vi tìm kiếm

Gọi `.locator()` **lên 1 Locator khác** (thay vì lên `this.page`) sẽ thu hẹp phạm vi tìm kiếm chỉ còn bên trong phần tử đó — tránh nhầm lẫn khi trang có nhiều phần tử lặp lại giống nhau.

```ts
for (const row of rows) {
    const cells = row.locator('td'); // chỉ tìm <td> BÊN TRONG đúng dòng "row" này
    const name = await cells.nth(0).innerText();
    const price = await cells.nth(1).innerText();
}
```

Nếu không chain mà gọi `this.page.locator('td')`, bạn sẽ lấy **tất cả** `<td>` trên toàn trang, không phân biệt được ô nào thuộc dòng nào.

---

## 2. Đọc dữ liệu từ phần tử

### 2.1. `innerText()` vs `textContent()`

| Method | Đặc điểm |
|---|---|
| `innerText()` | Chỉ lấy text **đang thực sự hiển thị** (tôn trọng CSS như `display: none`) |
| `textContent()` | Lấy **toàn bộ** text kể cả phần bị ẩn bằng CSS |

➡ Khi cần kiểm tra đúng những gì **người dùng nhìn thấy trên UI**, ưu tiên dùng `innerText()`.

### 2.2. `inputValue()` vs `getAttribute('value')` - PHÂN BIỆT QUAN TRỌNG

Đây là 1 trong những điểm dễ nhầm lẫn nhất:

| | `getAttribute('value')` | `inputValue()` |
|---|---|---|
| Đọc gì? | **HTML attribute** khai báo tĩnh trong mã nguồn (`<input value="...">`) | **DOM property** `.value` thực tế, phản ánh trạng thái hiện tại |
| Có tự cập nhật theo thao tác người dùng không? | ❌ Không (trừ khi JS chủ động `setAttribute`) | ✅ Có, luôn đúng theo trạng thái mới nhất |
| Dùng cho | Giá trị cố định không đổi (`value` của radio/checkbox, `data-*` tùy biến) | Input/textarea/select có thể bị thay đổi do gõ chữ, chọn option, kéo thanh trượt... |

**Ví dụ minh họa lỗi thực tế:**

```html
<select id="country" name="country">
    <option value="usa">United States</option>
    <option value="uk">United Kingdom</option>
</select>
```

Thẻ `<select>` **không có attribute `value`** khai báo sẵn trong HTML — chỉ các `<option>` con mới có. Vậy:

```ts
await this.page.locator(this.xpathCountry).getAttribute('value'); // ❌ luôn trả về null
await this.page.locator(this.xpathCountry).inputValue();          // ✅ trả về "uk" (option đang chọn)
```

### 2.3. `isChecked()` - dành riêng cho checkbox/radio

```ts
const newsletter = await this.page.locator(this.xpathSubcribe).isChecked(); // trả về boolean
```

**Bẫy thường gặp — custom toggle switch:** Nhiều UI style lại checkbox thành "công tắc gạt" bằng CSS, ẩn hẳn input thật đi:

```html
<label class="switch">
  <input type="checkbox" id="toggleOption">  <!-- bị CSS ẩn: width:0; height:0; opacity:0 -->
  <span class="slider round"></span>          <!-- đây mới là phần hiển thị, người dùng click vào đây -->
</label>
```

➡ **Phải dùng 2 locator khác nhau cho 2 mục đích khác nhau:**
- Để **click** (giả lập thao tác người dùng thật) → click vào `<span class="slider round">` (phần tử có kích thước, "visible" theo Playwright).
- Để **đọc trạng thái** → dùng `input#toggleOption` với `.isChecked()` (không yêu cầu phần tử phải visible, chỉ đọc property `checked`).

```ts
// Click (dùng span - phần tử hiển thị)
await this.page.locator("//span[@class='slider round']").click();

// Đọc trạng thái (dùng input thật - không cần visible)
const enableFeature = await this.page.locator("#toggleOption").isChecked();
```

Nếu gọi `.click()` thẳng lên `input#toggleOption` sẽ bị lỗi **"element is not visible"** timeout 30s, vì Playwright yêu cầu phần tử phải có kích thước thật (bounding box > 0) trước khi click, mà input này bị CSS set kích thước bằng 0.

---

## 3. CSS Selector vs XPath

### 3.1. Cú pháp cơ bản

| | CSS Selector | XPath |
|---|---|---|
| Cách viết trong `.locator()` | Viết trực tiếp: `"input[name='hobbies']"` | Bắt đầu bằng `//`: `"//input[@name='hobbies']"` |
| Chọn phần tử theo quan hệ cha/con phức tạp | Hạn chế | Mạnh (ví dụ tìm cha dựa theo con: `//li[span[...]]`) |

Playwright cho phép trộn lẫn cả 2 loại trong cùng 1 class POM, tùy tình huống nào tiện hơn.

### 3.2. Pseudo-class `:checked` (CSS) - lọc ngay tại locator

```ts
// Chỉ lấy checkbox ĐANG được tick, không cần code lọc thêm
const checkedBoxes = await this.page.locator("input[name='hobbies']:checked").all();
```

`:checked` tự động loại bỏ các checkbox không được tick ngay từ bước tìm kiếm — để trình duyệt lọc hộ, thay vì lấy hết rồi tự code kiểm tra từng cái.

### 3.3. XPath động (dynamic xpath) bằng template literal

Khi cần build locator theo giá trị chỉ biết lúc runtime (tham số truyền vào), dùng template literal `` `...${bien}...` ``:

```ts
getTodoItemLocator(taskText: string) {
    return this.page.locator(`//li[span[text()='${taskText}']]`);
}

// gọi: getTodoItemLocator('Todo 8')
// → build ra: //li[span[text()='Todo 8']]
```

Đọc cú pháp `//li[span[text()='Todo 8']]`:
- `text()='Todo 8'` → hàm XPath so khớp **chính xác** nội dung text.
- `span[text()='Todo 8']` → tìm `<span>` có text khớp.
- `//li[span[...]]` → tìm `<li>` mà **bên trong có chứa** `<span>` thỏa điều kiện đó (điều kiện lọc đặt trong `[...]`).

➡ Ưu điểm: locator theo **nội dung** không phụ thuộc vị trí/index — dù phần tử đổi vị trí trong danh sách (do thêm/xóa phần tử khác), locator này vẫn tìm đúng.

### 3.4. Escape ký tự đặc biệt trong XPath

**Vấn đề:** XPath dùng dấu nháy để bao chuỗi. Nếu giá trị truyền vào chứa dấu nháy đơn (`'`), chuỗi xpath ráp ra sẽ bị "gãy" cú pháp:

```ts
// taskText = "Don't forget"
`//li[span[text()='${taskText}']]`
// → ráp ra: //li[span[text()='Don't forget']]  ❌ lỗi cú pháp
// (XPath hiểu chuỗi kết thúc ngay ở dấu ' trong "Don't", phần còn lại dư thừa gây lỗi)
```

XPath 1.0 **không hỗ trợ** ký tự escape (`\'`) như JS. Các cách xử lý:

1. Đổi sang nháy kép nếu chắc chắn giá trị không chứa `"`: `text()="${taskText}"`
2. Dùng hàm `concat()` của XPath để ghép chuỗi từ nhiều đoạn (phức tạp, ít dùng).
3. **Cách an toàn nhất — tránh dùng XPath cho trường hợp này:**
   ```ts
   this.page.locator('li').filter({ hasText: taskText })
   // hoặc
   this.page.getByText(taskText, { exact: true })
   ```
   Giá trị được truyền như 1 tham số JS bình thường, không ráp vào chuỗi cú pháp → không lo escape.

---

## 4. Async/Await & TypeScript trong Playwright

### 4.1. `async`/`await` cơ bản

- `async` đánh dấu hàm là bất đồng bộ, cho phép dùng `await` bên trong để "chờ" 1 `Promise` hoàn thành trước khi chạy dòng tiếp theo.
- Mọi thao tác hỏi trình duyệt thật sự (`.innerText()`, `.click()`, `.isChecked()`, `.all()`...) đều là bất đồng bộ → cần `await`.
- `.locator()`, `.filter()`, `.nth()` **không** bất đồng bộ (chỉ tạo công thức tìm kiếm) → không cần `await`.

### 4.2. Kiểu trả về `Promise<T>` và Type Inference

```ts
async getVisibleNotesText(): Promise<string[]> { ... }
```

- Vì hàm là `async`, nó **luôn** trả về `Promise`. `<string[]>` mô tả kiểu dữ liệu **bên trong** Promise đó khi resolve xong.
- Nếu **không khai báo** kiểu trả về, TypeScript vẫn **tự suy luận đúng** dựa vào câu lệnh `return` bên trong hàm (type inference) — không bắt buộc phải viết tường minh, nhưng nên viết khi:
  - Hàm dài, nhiều nhánh `return` khác nhau.
  - Là method public trong 1 class dùng chung (dễ đọc signature mà không cần đọc hết thân hàm).

### 4.3. `Record<K, V>` - kiểu object dạng "từ điển"

```ts
const expectedQuantities: Record<string, number> = {
    'Product 1': 2,
    'Product 2': 3,
    'Product 3': 1,
};
```

`Record<string, number>` mô tả 1 object mà mọi key là `string`, mọi value là `number` — tương đương cách viết `interface { [key: string]: number }` nhưng gọn hơn. Dùng để tra cứu động: `expectedQuantities[item.name]`.

### 4.4. Nullish coalescing (`??`)

```ts
starRating: starRating ?? '0',
```

Nếu biến bên trái (`starRating`) là `null` hoặc `undefined` → lấy giá trị bên phải (`'0'`) làm mặc định; ngược lại giữ nguyên giá trị gốc. Dùng để xử lý an toàn khi `getAttribute()` có thể trả về `null` (thuộc tính không tồn tại).

### 4.5. Object shorthand property

```ts
return { username, email, gender, hobbies };
// tương đương: return { username: username, email: email, gender: gender, hobbies: hobbies };
```

Khi tên biến và tên property trùng nhau, có thể viết gọn không cần lặp lại 2 lần.

### 4.6. `.reduce()` - gộp mảng thành 1 giá trị

```ts
const expectedTotal = cartItems.reduce((sum, item) => sum + item.total, 0);
```

- `sum`: giá trị tích lũy qua từng lần lặp (accumulator).
- `item`: phần tử đang xét.
- `0`: giá trị khởi tạo ban đầu của `sum`.

**Diễn giải từng bước với ví dụ (total: 20, 60, 30):**
```
sum = 0                  (khởi tạo)
sum = 0 + 20 = 20        (sau Product 1)
sum = 20 + 60 = 80       (sau Product 2)
sum = 80 + 30 = 110      (sau Product 3) → kết quả cuối cùng
```

Tương đương cách viết vòng lặp thủ công:
```ts
let expectedTotal = 0;
for (const item of cartItems) {
    expectedTotal += item.total;
}
```

---

## 5. Assertion (expect) - Web-first assertion

### 5.1. `toContain` - kiểm tra substring

```ts
expect(rowText).toContain(`gender: ${expectedData.gender}`);
```

Kiểm tra chuỗi lớn có **chứa** chuỗi con hay không — phù hợp khi dữ liệu nằm lẫn trong 1 khối text dài (ví dụ cột "Information" gộp nhiều field bằng `<br>`).

### 5.2. `toBe` vs `toBeCloseTo` - so sánh số

```ts
expect(item.total).toBeCloseTo(item.price * item.quantity, 2);
```

- `.toBe()`: so sánh **bằng chính xác tuyệt đối** — rủi ro với số thực (`float`) vì sai số làm tròn nhị phân (`0.1 + 0.2 !== 0.3` trong JS).
- `.toBeCloseTo(value, precision)`: so sánh **gần bằng**, cho phép sai lệch trong phạm vi `precision` chữ số thập phân — an toàn hơn khi làm việc với tiền tệ/số thực.

### 5.3. `toHaveCount(n)` - đếm số phần tử khớp locator

```ts
await expect(todo5).toHaveCount(0);
```

Kiểm tra **số lượng phần tử tồn tại trong DOM** khớp với locator — dùng đúng khi cần xác nhận phần tử **hoàn toàn không còn trong DOM** (khác với "bị ẩn bằng CSS nhưng vẫn còn trong DOM").

⚠️ Phân biệt rõ 2 khái niệm:
| Trạng thái | Còn trong DOM? | `toBeVisible()` | `toHaveCount(0)` |
|---|---|---|---|
| Bị xóa hẳn (`splice()`) | ❌ Không | fail (không tìm thấy) | ✅ pass |
| Bị ẩn bằng CSS (`display:none`) | ✅ Có | fail (không visible) | ❌ fail (vẫn đếm được 1) |

### 5.4. `toBeInViewport()` - kiểm tra phần tử nằm trong khung nhìn

```ts
await expect(todo8).toBeInViewport();
```

Tự động đo `boundingBox()` của phần tử, so với kích thước viewport hiện tại, kiểm tra có phần giao nhau hay không (mặc định chỉ cần giao nhau 1 phần, có thể tùy chỉnh bằng `{ ratio: 0.5 }` để yêu cầu hiển thị ≥ 50%).

**Ví dụ lỗi thực tế khi test với danh sách dài (100 todo):**
```
Received: viewport ratio 0
```
Nghĩa là phần tử **hoàn toàn nằm ngoài màn hình** (trang quá dài, phần tử ở tít phía dưới, chưa cuộn tới). Đây không phải lỗi code — là đúng bản chất hình học của trang tại thời điểm kiểm tra.

**Cách xử lý nếu muốn xác nhận "có thể cuộn tới xem được":**
```ts
await todo90.scrollIntoViewIfNeeded(); // cuộn tới phần tử nếu nó đang ở ngoài viewport
await expect(todo90).toBeInViewport();  // giờ mới pass
```
`scrollIntoViewIfNeeded()`: chỉ cuộn khi cần thiết (nếu phần tử đã trong viewport thì không làm gì).

### 5.5. Cơ chế auto-waiting / auto-retry của web-first assertion

Khác với `expect()` thông thường (Jest cho unit test, không cần `await`), các assertion của Playwright gắn với Locator (`toBeVisible`, `toHaveCount`, `toBeInViewport`...) đều:
- Trả về `Promise` → **bắt buộc phải có `await`** phía trước.
- Tự động kiểm tra lại nhiều lần trong khoảng thời gian timeout (mặc định 5s), không phải chỉ check 1 lần duy nhất — giúp xử lý tốt các trường hợp DOM đang cập nhật bất đồng bộ (animation, re-render...).

Nếu quên `await`, assertion chạy "ngầm" không đồng bộ với phần code còn lại → test có thể kết thúc trước khi assertion kịp kiểm tra xong, dẫn đến kết quả sai lệch.

---

## 6. Design pattern khi viết POM

### 6.1. Snapshot pattern - chụp lại giá trị trước khi hành động thay đổi trạng thái

Với các giá trị "động", phụ thuộc thao tác UI (vị trí click, giá trị slider...) mà bạn khó đoán trước chính xác, hãy **đọc giá trị thật từ trang ngay trước khi submit**, lưu vào biến, rồi so sánh với kết quả sau đó — thay vì hard-code số liệu mong đợi.

```ts
async captureFormValues() {
    const rating = await this.page.locator(this.xpathRating).inputValue();
    const starRating = await this.page.locator(this.xpathRatingStar).getAttribute('data-rating');
    // ... đọc toàn bộ giá trị hiện có trên form
    return { rating, starRating: starRating ?? '0', /* ... */ };
}
```

Gọi hàm này **ngay trước** khi bấm submit, để có "ảnh chụp" đúng thời điểm dữ liệu chuẩn bị được gửi đi.

### 6.2. Locator theo nội dung (content-based) vs theo vị trí (index-based)

**Vấn đề với index-based:** khi xóa 1 phần tử khỏi mảng (`splice()`), các phần tử phía sau bị dịch index. Nếu vòng lặp xóa theo thứ tự **tăng dần**, các index tính toán trước đó sẽ bị sai lệch sau mỗi lần xóa.

```ts
// Cách theo index - phải xóa GIẢM DẦN để tránh bug dịch chuyển index
async deleteTask(n: number) {
    for (let i = n; i >= 0; i--) {
        if (i % 2 === 0) {
            await this.page.locator(`//button[@onclick='deleteTask(${i})']`).click();
        }
    }
}
```

**Cách theo nội dung** — không còn phụ thuộc thứ tự, dễ đọc hơn, khớp trực tiếp với yêu cầu đề bài:

```ts
async deleteOddTasks(n: number) {
    for (let i = 1; i <= n; i += 2) {  // duyệt thẳng số lẻ 1, 3, 5, 7, 9 — không cần suy ra index
        const taskItem = this.getTodoItemLocator(`Todo ${i}`);
        await taskItem.getByRole('button', { name: 'Delete' }).click();
    }
}
```

➡ Tra theo tên (`getByRole('button', { name: 'Delete' })` bên trong `<li>` đã lọc theo nội dung) tránh luôn việc phải tự đoán lại `id`/`slug` do trang tự sinh ra — chỉ dựa vào **text hiển thị** ("Delete"), vốn ổn định hơn nhiều so với chi tiết cài đặt bên trong.

### 6.3. Property tĩnh (constant) vs Method tham số (dynamic)

| | Dùng khi nào | Ví dụ |
|---|---|---|
| **Property** (`xpathUsername: string = "..."`) | Giá trị **cố định**, biết trước, không đổi | `xpathNewTask`, `xpathAddTaskButton` |
| **Method** (`getTodoItemLocator(taskText: string)`) | Giá trị **phụ thuộc tham số**, chỉ biết lúc runtime | Tìm phần tử theo nội dung động |

➡ Không nên cố ép locator có tham số động thành 1 property tĩnh — đây là 2 khái niệm khác bản chất trong OOP (constant vs computed/parameterized value).

### 6.4. Kiểm tra chéo (cross-check) dữ liệu nhiều lớp

Khi verify số liệu tính toán (như tổng tiền giỏ hàng), nên kiểm tra ở **nhiều cấp độ**:
1. Từng dòng: `Total` của dòng = `Price × Quantity`.
2. Toàn bộ: tổng các dòng `Total` = số hiển thị ở "Total Price" cuối bảng.

```ts
// Cấp độ dòng
for (const item of cartItems) {
    expect(item.total).toBeCloseTo(item.price * item.quantity, 2);
}
// Cấp độ tổng thể
const expectedTotal = cartItems.reduce((sum, item) => sum + item.total, 0);
expect(await productPage.getCartTotalPrice()).toBeCloseTo(expectedTotal, 2);
```

Cách này giúp nếu có lỗi, bạn biết ngay lỗi nằm ở **công thức tính từng dòng** hay ở **bước cộng tổng**.

### 6.5. Tránh "pass giả" khi mảng có thể rỗng

```ts
expect(cartItems.length).toBe(3); // hoặc: expect(notesText.length).toBeGreaterThan(0);

for (const item of cartItems) {
    expect(item.quantity).toBe(expectedQuantities[item.name]);
}
```

Nếu không kiểm tra độ dài mảng trước, khi mảng rỗng (ví dụ do bug khiến giỏ hàng trống), vòng lặp `for` sẽ không chạy lần nào → test **"pass" giả** dù thực chất chẳng kiểm tra được gì.

---

## 7. Các lỗi thường gặp & cách debug

### 7.1. `Error: locator.click: Test timeout of 30000ms exceeded` — "element is not visible"

**Nguyên nhân điển hình:** phần tử bị CSS ẩn đi (kích thước = 0, hoặc `display:none`) nhưng bạn cố `.click()` thẳng vào nó, thay vì phần tử hiển thị đại diện (xem lại mục [2.3](#23-ischecked---dành-riêng-cho-checkboxradio) - custom toggle switch).

**Cách chẩn đoán:** đọc kỹ "Call log" trong lỗi — nếu thấy lặp lại `"element is not visible"` nhiều lần cho tới hết timeout, nghĩa là Playwright đã tìm thấy đúng phần tử nhưng nó không đạt điều kiện "visible" để thao tác.

### 7.2. Index shifting khi xóa phần tử trong mảng (`splice`)

**Triệu chứng:** xóa nhầm sai phần tử khi lặp xóa nhiều item theo index.

**Nguyên nhân:** `array.splice(index, 1)` làm các phần tử **sau** index đó bị dịch lùi 1 vị trí. Nếu bạn xóa tăng dần (từ index nhỏ trước), các index đã tính trước cho các lần xóa sau sẽ bị sai lệch.

**Giải pháp:** xóa **giảm dần** (từ index lớn về nhỏ), hoặc tốt hơn — chuyển sang tra cứu theo **nội dung** thay vì index (xem mục [6.2](#62-locator-theo-nội-dung-content-based-vs-theo-vị-trí-index-based)).

### 7.3. `Received: viewport ratio 0` khi dùng `toBeInViewport()`

**Không phải lỗi code** — là kết quả đúng phản ánh thực tế: phần tử đang nằm ngoài khung nhìn hiện tại (trang dài, chưa cuộn tới). Xem lại mục [5.4](#54-tobeinviewport---kiểm-tra-phần-tử-nằm-trong-khung-nhìn) để quyết định có cần thêm `scrollIntoViewIfNeeded()` hay không, tùy theo ý định thật sự của yêu cầu kiểm tra.

---

## 8. Tổng hợp qua 4 bài tập thực hành

### 8.1. Personal Notes - Search & verify keyword

**Yêu cầu:** thêm 10 note, search theo keyword, kiểm tra tất cả kết quả search đều chứa keyword.

**Điểm kiến thức áp dụng:**
- `.all()` lấy toàn bộ note đang hiển thị sau khi search.
- Cơ chế filter của trang: `title.includes(keyword) || content.includes(keyword)` (không phân biệt hoa/thường) → khi assert phải gộp cả title lẫn content, và `.toLowerCase()` trước khi so sánh.
- Kiểm tra `length > 0` để tránh "pass giả" khi danh sách rỗng.

```ts
async getVisibleNotesText(): Promise<string[]> {
    const notes = await this.page.locator(this.xpathNotesList).all();
    const notesText: string[] = [];
    for (const note of notes) {
        const text = await note.innerText();
        notesText.push(text.toLowerCase());
    }
    return notesText;
}
```

### 8.2. User Registration - Verify bảng dữ liệu

**Yêu cầu:** đăng ký đầy đủ thông tin, kiểm tra dữ liệu hiển thị đúng trong bảng.

**Điểm kiến thức áp dụng:**
- Snapshot pattern: `captureFormValues()` đọc toàn bộ giá trị form **trước khi** bấm submit.
- Phân biệt `inputValue()` (giá trị input/select) vs `getAttribute()` (thuộc tính tĩnh như `value` của radio, `data-rating`) vs `isChecked()` (checkbox/radio).
- CSS pseudo-class `:checked` để lọc nhanh checkbox/radio đang được chọn.
- Custom toggle switch: input thật bị ẩn CSS → tách 2 locator riêng cho click và cho đọc trạng thái.
- `.last()` lấy đúng dòng vừa đăng ký trong bảng.

### 8.3. Shopping Cart - Số lượng & Tổng tiền

**Yêu cầu:** thêm sản phẩm vào giỏ với số lượng cụ thể, kiểm tra số lượng và tổng tiền đúng.

**Điểm kiến thức áp dụng:**
- Locator chaining (`row.locator('td')`) + `.nth()` để đọc từng ô trong 1 dòng.
- `Record<string, number>` để tra cứu số lượng mong đợi theo tên sản phẩm (không phụ thuộc thứ tự dòng).
- `.reduce()` để cộng dồn tổng tiền từ các dòng.
- `.toBeCloseTo()` khi so sánh số tiền (tránh sai số dấu phẩy động).
- Kiểm tra chéo 2 lớp: từng dòng đúng công thức, và tổng toàn bộ khớp với số hiển thị cuối bảng.

```ts
async getCartItems(): Promise<{ name: string; price: number; quantity: number; total: number }[]> {
    const rows = await this.page.locator(this.xpathCartRows).all();
    const items = [];
    for (const row of rows) {
        const cells = row.locator('td');
        items.push({
            name: (await cells.nth(0).innerText()).trim(),
            price: parseFloat((await cells.nth(1).innerText()).replace('$', '')),
            quantity: parseInt(await cells.nth(2).innerText(), 10),
            total: parseFloat((await cells.nth(3).innerText()).replace('$', '')),
        });
    }
    return items;
}
```

### 8.4. To-Do List - Viewport & DOM presence

**Yêu cầu:** thêm 10 todo, xóa các todo số lẻ, kiểm tra 1 todo còn lại nằm trong viewport, 1 todo đã xóa không còn trong DOM.

**Điểm kiến thức áp dụng:**
- XPath động theo nội dung: `//li[span[text()='${taskText}']]` — ổn định dù thứ tự danh sách thay đổi sau khi xóa.
- `toBeInViewport()` cho việc kiểm tra vị trí hiển thị.
- `toHaveCount(0)` cho việc kiểm tra "không còn trong DOM" (khác với "không visible").
- Vấn đề index shifting khi xóa nhiều phần tử — giải pháp xóa giảm dần hoặc tra theo nội dung.
- `scrollIntoViewIfNeeded()` khi cần chủ động cuộn tới phần tử nằm ngoài khung nhìn ban đầu (trường hợp danh sách dài, ví dụ 100 todo).

```ts
getTodoItemLocator(taskText: string) {
    return this.page.locator(`//li[span[text()='${taskText}']]`);
}

async deleteOddTasks(n: number) {
    this.page.on('dialog', async dialog => dialog.accept());
    for (let i = 1; i <= n; i += 2) {
        const taskItem = this.getTodoItemLocator(`Todo ${i}`);
        await taskItem.getByRole('button', { name: 'Delete' }).click();
    }
}
```

---

## Bảng tra cứu nhanh (Cheat sheet)

| Cần làm gì | Dùng gì |
|---|---|
| Lấy nhiều phần tử cùng lúc | `.all()` |
| Chọn phần tử theo vị trí | `.nth()`, `.first()`, `.last()` |
| Thu hẹp phạm vi tìm kiếm vào 1 phần tử cha | `parentLocator.locator(...)` (chaining) |
| Đọc text đang hiển thị | `.innerText()` |
| Đọc giá trị input/select hiện tại | `.inputValue()` |
| Đọc thuộc tính HTML tĩnh | `.getAttribute('...')` |
| Đọc trạng thái checkbox/radio | `.isChecked()` |
| Lọc checkbox/radio đang chọn ngay tại locator | CSS `:checked` |
| Build locator theo giá trị động | Template literal `` `...${var}...` `` |
| Tránh lỗi cú pháp khi giá trị có dấu nháy | `.filter({ hasText })`, `getByText()` |
| Kiểm tra chuỗi con | `expect(...).toContain(...)` |
| So sánh số thực (tiền tệ...) | `expect(...).toBeCloseTo(value, precision)` |
| Đếm số phần tử khớp / kiểm tra không còn trong DOM | `expect(...).toHaveCount(n)` |
| Kiểm tra nằm trong khung nhìn | `expect(...).toBeInViewport()` |
| Chủ động cuộn tới phần tử | `.scrollIntoViewIfNeeded()` |
| Giá trị mong đợi không đoán trước được | Snapshot pattern: đọc từ form trước khi submit |
| Xóa nhiều phần tử theo index | Xóa giảm dần, hoặc tra theo nội dung để tránh index shifting |
