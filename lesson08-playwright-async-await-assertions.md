# Playwright: async/await & Assertions - Tổng hợp kiến thức

---

## 1. async() vs async({ page }) — Khi nào dùng cái nào?

### Khái niệm cốt lõi

`{ page }` là **Playwright Fixture** — một đối tượng đại diện cho tab browser.
Playwright tự động tạo và inject nó vào test khi bạn khai báo `{ page }`.

- Dùng `async()` khi **không cần tương tác với browser**
- Dùng `async({ page })` khi **cần điều khiển browser** (click, goto, lấy text...)

---

### Bảng phân loại

| Hook / Test | Dùng | Lý do |
|---|---|---|
| `test.describe("...", async() => {})` | `async()` | Chỉ là container chứa test, không dùng browser trực tiếp |
| `test.afterAll(async() => {})` | `async()` | Chỉ cleanup logic thuần (log, close connection...) |
| `test("Demo", async() => {})` | `async()` | Chỉ test logic thuần (số, string, array) — không cần browser |
| `test.beforeAll(async({ page }) => {})` | `async({ page })` | Cần `page.goto()` để điều hướng |
| `test.beforeEach(async({ page }) => {})` | `async({ page })` | Cần điều hướng trước mỗi test |
| `test.afterEach(async({ page }) => {})` | `async({ page })` | Cần thao tác browser sau mỗi test |
| `test("UI Test", async({ page }) => {})` | `async({ page })` | Cần click, fill, assert trên UI |

---

### Ví dụ minh họa

```typescript
// KHÔNG cần page — chỉ test logic thuần JavaScript
test("Demo expect", async () => {
    expect(1 + 2).toEqual(3);

    const arr = [1, 2, 3];
    expect(arr).toHaveLength(3);

    const str = "Hello Viet Nam";
    expect(str).toContain("Nam");
});

// CẦN page — tương tác với browser
test("Material Page", async ({ page }) => {
    await page.goto("https://material.playwrightvn.com/");
    const title = await page.title();
    expect(title).toContain("Playwright Việt Nam");
});

// afterAll không cần page nếu chỉ log
test.afterAll(async () => {
    console.log("Tất cả test đã xong");
});

// beforeEach cần page vì phải điều hướng
test.beforeEach(async ({ page }) => {
    await page.goto("https://material.playwrightvn.com/");
});
```

---

### Các fixtures khác ngoài `page`

Playwright còn có nhiều fixture khác, destructure tương tự:

```typescript
test("example", async ({ page, browser, context, request }) => {
    // page     — tab browser hiện tại
    // browser  — đối tượng browser (Chrome, Firefox...)
    // context  — browser context (như một profile riêng)
    // request  — dùng để gọi API (không cần browser)
});
```

---

## 2. await vs không await — Khi nào dùng?

### Nguyên tắc cốt lõi

> **Dùng `await` khi hàm trả về `Promise`** (tức là cần chờ browser/hệ thống xử lý xong)
>
> **Không dùng `await` khi hàm trả về kết quả ngay lập tức** (đồng bộ)

---

### Nhóm CẦN `await`

#### Nhóm 1 — Điều hướng & tương tác DOM

```typescript
// Điều hướng trang — browser cần tải xong
await page.goto("https://example.com");

// Click — browser cần xử lý sự kiện
await page.locator("//button").click();

// Fill vào input
await page.locator("//input[@id='email']").fill("test@example.com");

// Hover
await page.locator("//div[@class='menu']").hover();

// Press key
await page.locator("//input").press("Enter");

// Select option
await page.locator("//select").selectOption("value1");
```

#### Nhóm 2 — Lấy dữ liệu từ browser (trả về Promise<giá_trị>)

```typescript
// Lấy title trang — phải hỏi browser → async
const title = await page.title();                         // Promise<string>

// Lấy URL hiện tại
const url = await page.url();                             // string (đồng bộ — KHÔNG cần await)

// Lấy text của element
const text = await page.locator("//h1").innerText();      // Promise<string>
const textContent = await page.locator("//p").textContent(); // Promise<string | null>

// Lấy giá trị input
const value = await page.locator("//input").inputValue(); // Promise<string>

// Lấy attribute
const href = await page.locator("//a").getAttribute("href"); // Promise<string | null>
```

#### Nhóm 3 — Kiểm tra trạng thái DOM (trả về Promise<boolean>)

```typescript
const isVisible = await page.locator("//button").isVisible();   // Promise<boolean>
const isEnabled = await page.locator("//button").isEnabled();   // Promise<boolean>
const isChecked = await page.locator("//input").isChecked();    // Promise<boolean>
const isEditable = await page.locator("//input").isEditable();  // Promise<boolean>
const isHidden   = await page.locator("//button").isHidden();   // Promise<boolean>
```

#### Nhóm 4 — Web-first assertions (có retry bên trong)

```typescript
await expect(locator).toBeVisible();
await expect(locator).toBeEnabled({ timeout: 10_000 });
await expect(locator).toHaveText("Hello");
await expect(locator).toHaveValue("input value");
await expect(locator).toHaveClass("active");
await expect(locator).toBeChecked();
await expect(page).toHaveURL("https://example.com");
await expect(page).toHaveTitle("My App");
```

#### Nhóm 5 — Chờ đợi

```typescript
// Chờ cố định (không khuyến khích)
await page.waitForTimeout(3_000);

// Chờ selector xuất hiện
await page.waitForSelector("//div[@class='modal']");

// Chờ navigation
await page.waitForURL("https://example.com/dashboard");

// Chờ network
await page.waitForLoadState("networkidle");
```

#### Nhóm 6 — test.step

```typescript
await test.step("Điền form đăng ký", async () => {
    await page.locator("//input[@id='email']").fill("test@test.com");
    await page.locator("//button[text()='Submit']").click();
});
```

---

### Nhóm KHÔNG cần `await`

#### page.locator() — Chỉ tạo đối tượng Locator, chưa đụng browser

```typescript
// ĐÚNG — không await
const button = page.locator("//button[@id='submit']");
const input  = page.locator("//input[@name='email']");

// Sau đó mới await khi dùng
await button.click();
const isEnabled = await button.isEnabled();
```

#### Non web-first assertions — Kiểm tra giá trị đã có sẵn (đồng bộ)

```typescript
// isVisible đã là boolean rồi → expect đồng bộ
const isVisible = await page.locator("//button").isVisible();
expect(isVisible).toEqual(true);   // KHÔNG await

// title đã là string rồi
const title = await page.title();
expect(title).toContain("Playwright"); // KHÔNG await

// Test logic thuần
expect(1 + 1).toBe(2);               // KHÔNG await
expect([1,2,3]).toHaveLength(3);     // KHÔNG await
expect("Hello").toContain("He");     // KHÔNG await
```

#### page.url() — Trả về string đồng bộ

```typescript
// url() là đồng bộ trong Playwright
const currentUrl = page.url(); // KHÔNG cần await
```

---

### Sơ đồ quyết định

```
Hàm này có trả về Promise không?
        │
        ├── Có → await
        │         ├── page.goto(), click(), fill(), hover()
        │         ├── page.title(), innerText(), getAttribute()
        │         ├── isVisible(), isEnabled(), isChecked()
        │         ├── expect(locator).toBeVisible() (web-first)
        │         └── page.waitForTimeout(), waitForSelector()
        │
        └── Không → KHÔNG await
                  ├── page.locator()   — trả về Locator object
                  ├── page.url()       — trả về string ngay
                  └── expect(value).toEqual() — đồng bộ
```

---

## 3. Non web-first Assertion vs Web-first Assertion

### Định nghĩa

| | Non web-first | Web-first |
|---|---|---|
| **Nguồn gốc** | Jest / Vitest style | Playwright-specific |
| **Input** | Giá trị thông thường (boolean, string...) | Locator object |
| **Retry** | Không — kiểm tra 1 lần duy nhất | Có — tự retry cho đến khi pass hoặc timeout |
| **await** | Không cần (assertion đồng bộ) | Cần (assertion bất đồng bộ) |
| **Phù hợp với** | Element đã sẵn sàng, UI tĩnh | Element xuất hiện/thay đổi theo thời gian |

---

### So sánh chi tiết qua ví dụ

#### Trường hợp: Kiểm tra button có enabled không sau khi trang load

```typescript
// ❌ NON WEB-FIRST — phải waitForTimeout cứng, lãng phí thời gian
test("Non web-first", async ({ page }) => {
    await page.goto("https://material.playwrightvn.com/019-enable-form.html");

    // Luôn chờ ĐÚNG 10 giây dù button enable sau 2 giây
    await page.waitForTimeout(10_000);

    // isEnabled() trả về boolean tại thời điểm gọi — không retry
    const isEnable = await page.locator("//button[@id='submitButton']").isEnabled();
    expect(isEnable).toEqual(true); // kiểm tra boolean đã có sẵn → không await
});

// ✅ WEB-FIRST — thông minh hơn, linh hoạt hơn
test("Web-first", async ({ page }) => {
    await page.goto("https://material.playwrightvn.com/019-enable-form.html");

    // Tự retry liên tục, nếu enable sau 2 giây → đi tiếp ngay, không chờ đủ 10 giây
    await expect(page.locator("//button[@id='submitButton']")).toBeEnabled({ timeout: 10_000 });
    // Nếu không truyền timeout → mặc định 5 giây
});
```

**Kết quả thực tế:**
- Nếu button enable sau **2 giây**:
  - Non web-first: tốn **10 giây** (chờ cứng)
  - Web-first: tốn **2 giây** (detect xong đi tiếp)

---

#### Trường hợp: Kiểm tra element có visible không

```typescript
// Non web-first
const isVisible = await page.locator("//button[@type='submit']").isVisible();
expect(isVisible).toEqual(true);

// Web-first — ngắn gọn và thông minh hơn
await expect(page.locator("//button[@type='submit']")).toBeVisible();
```

---

#### Trường hợp: Kiểm tra URL sau khi click

```typescript
// Non web-first — lấy URL rồi so sánh
await page.locator("//a[text()='Login']").click();
await page.waitForTimeout(2_000); // chờ cứng
const url = page.url();
expect(url).toContain("/login");

// Web-first — retry cho đến khi URL khớp
await page.locator("//a[text()='Login']").click();
await expect(page).toHaveURL(/\/login/); // regex hoặc string
```

---

### Danh sách các web-first assertions phổ biến

```typescript
// Element assertions
await expect(locator).toBeVisible();
await expect(locator).toBeHidden();
await expect(locator).toBeEnabled();
await expect(locator).toBeDisabled();
await expect(locator).toBeChecked();
await expect(locator).toBeEditable();
await expect(locator).toBeEmpty();
await expect(locator).toBeFocused();

// Content assertions
await expect(locator).toHaveText("exact text");
await expect(locator).toContainText("partial text");
await expect(locator).toHaveValue("input value");
await expect(locator).toHaveAttribute("href", "https://...");
await expect(locator).toHaveClass("active");
await expect(locator).toHaveId("my-id");
await expect(locator).toHaveCount(3); // số lượng element

// Page assertions
await expect(page).toHaveURL("https://example.com");
await expect(page).toHaveTitle("My App");

// Với timeout tùy chỉnh
await expect(locator).toBeVisible({ timeout: 5_000 });
await expect(locator).toHaveText("Done", { timeout: 15_000 });
```

---

### Khi nào dùng non web-first, khi nào dùng web-first?

```
Element đã chắc chắn có trên trang, UI tĩnh?
        │
        ├── Có → Dùng được cả hai, nhưng web-first vẫn an toàn hơn
        │
        └── Không (element load chậm, animation, AJAX...)
                  └── PHẢI dùng web-first assertion
```

**Quy tắc chung:** Luôn ưu tiên web-first assertion. Chỉ dùng non web-first khi bạn đã có giá trị cụ thể cần so sánh (boolean, string, number đã lấy về rồi).

---

## 4. Giải thích sâu: `await expect` — Ý nghĩa thực sự là gì?

### Câu hỏi hay gặp

> *"Tôi đã `await` ở `isEnabled()` rồi — tức là đã chờ xong rồi. Vậy tại sao web-first lại cần thêm `await` trước `expect` nữa?"*

Đây là câu hỏi rất hay. Để hiểu được, hãy nhìn vào **thứ mà `expect` nhận vào**.

---

### Phân tích từng dòng

#### Non web-first — `await` nằm ở chỗ LẤY GIÁ TRỊ

```typescript
// Bước 1: await ở đây — chờ browser trả về boolean
const isEnabled = await page.locator("//button").isEnabled();
//                ↑ await này để LẤY giá trị từ browser
//                  sau dòng này: isEnabled = true  (boolean bình thường rồi)

// Bước 2: expect nhận boolean đã có sẵn → so sánh đơn giản → đồng bộ
expect(isEnabled).toEqual(true);
// KHÔNG cần await — vì isEnabled đã là true/false rồi
// expect chỉ đang so sánh: "true có bằng true không?" → xong ngay
```

#### Web-first — `await` nằm ở chỗ `expect` vì nó nhận LOCATOR

```typescript
// KHÔNG có bước lấy giá trị trước
// expect nhận thẳng LOCATOR — chưa có giá trị gì cả
await expect(page.locator("//button")).toBeEnabled();
//    ↑ await ở đây vì expect phải TỰ:
//      1. Vào browser tìm element
//      2. Kiểm tra có enabled chưa?
//      3. Chưa → chờ vài ms → kiểm tra lại (retry)
//      4. Enabled rồi → resolve Promise → mới đi tiếp
```

---

### Hình dung bằng sơ đồ

```
NON WEB-FIRST:

  page.locator("//button").isEnabled()
              │
              │  (await — chờ browser trả về)
              ↓
           true / false   ← đã là giá trị cụ thể rồi
              │
              ↓
  expect( true ).toEqual( true )
              │
              └── so sánh 2 giá trị → không cần await


WEB-FIRST:

  expect( page.locator("//button") ).toBeEnabled()
              │
              │  (nhận Locator — chưa có giá trị)
              │
              ↓
    [tự vào browser kiểm tra]
    [chưa enabled → retry]
    [chưa enabled → retry]
    [enabled rồi → resolve Promise]
              │
              └── cần await vì trả về Promise
```

---

### Ví dụ minh họa rõ nhất

```typescript
// Giả sử button sẽ enable sau 3 giây

// ========== NON WEB-FIRST ==========
await page.waitForTimeout(5_000);           // chờ cứng 5 giây trước
const isEnabled = await page.locator("//button").isEnabled();
//  ↑ await ở đây: "chờ browser cho tôi biết trạng thái lúc NÀY"
//    lúc này là giây thứ 5 → button đã enable → isEnabled = true

expect(isEnabled).toEqual(true);
//  ↑ KHÔNG await: đây chỉ là "true có bằng true không?" → trả lời ngay


// ========== WEB-FIRST ==========
await expect(page.locator("//button")).toBeEnabled({ timeout: 5_000 });
//  ↑ await ở đây: "chờ cho đến khi button enable (tối đa 5 giây)"
//    giây 1: chưa enable → thử lại
//    giây 2: chưa enable → thử lại
//    giây 3: ENABLE RỒI → pass → đi tiếp ngay
//    → chỉ tốn 3 giây thay vì 5 giây
```

---

### Bảng so sánh `await` ở đâu

| | Non web-first | Web-first |
|---|---|---|
| **`await` nằm ở đâu** | Tại `isEnabled()` — lấy giá trị | Tại `expect()` — chạy retry |
| **`expect` nhận vào** | `true` / `false` (boolean) | Locator object |
| **`expect` làm gì** | So sánh 2 giá trị → ngay lập tức | Vào browser kiểm tra, retry |
| **Cần `await` ở `expect`?** | Không | Có |

---

### 1 câu để nhớ mãi mãi

> **Non web-first:** Bạn tự đi lấy thức ăn về rồi mới ăn — `await` là lúc đi lấy.
>
> **Web-first:** Bạn gọi bồi bàn mang đến, ngồi chờ họ mang tới — `await` là lúc ngồi chờ bồi bàn.

---

## 5. Tổng hợp — Bảng tra cứu nhanh

### async() vs async({ page })

```
Cần dùng page.goto() / click() / locator()...?
    ├── Có  → async({ page })
    └── Không → async()
```

### await hay không?

| Hàm | await? | Lý do |
|---|---|---|
| `page.goto()` | ✅ | Promise<Response> |
| `page.locator()` | ❌ | Trả về Locator ngay |
| `locator.click()` | ✅ | Promise<void> |
| `locator.fill()` | ✅ | Promise<void> |
| `locator.isVisible()` | ✅ | Promise<boolean> |
| `locator.isEnabled()` | ✅ | Promise<boolean> |
| `page.title()` | ✅ | Promise<string> |
| `page.url()` | ❌ | Trả về string ngay |
| `page.waitForTimeout()` | ✅ | Promise<void> |
| `page.waitForSelector()` | ✅ | Promise<ElementHandle> |
| `expect(value).toEqual()` | ❌ | Đồng bộ |
| `expect(locator).toBeVisible()` | ✅ | Promise (web-first, có retry) |
| `test.step()` | ✅ | Promise<T> |

---

## 5. Anti-patterns cần tránh

```typescript
// ❌ SAI — await với locator() (không cần thiết, gây confusion)
const button = await page.locator("//button");

// ✅ ĐÚNG
const button = page.locator("//button");
await button.click();

// ❌ SAI — quên await với isVisible() → luôn là Promise object, không phải boolean
const isVisible = page.locator("//button").isVisible(); // isVisible là Promise, không phải true/false
expect(isVisible).toEqual(true); // Luôn fail!

// ✅ ĐÚNG
const isVisible = await page.locator("//button").isVisible();
expect(isVisible).toEqual(true);

// ❌ SAI — dùng waitForTimeout thay vì web-first assertion
await page.waitForTimeout(5_000);
const isEnabled = await page.locator("//button").isEnabled();
expect(isEnabled).toBe(true);

// ✅ ĐÚNG — web-first assertion tự retry
await expect(page.locator("//button")).toBeEnabled({ timeout: 5_000 });
```

---

*Tổng hợp từ các bài học Playwright - Better Bytes Academy*
