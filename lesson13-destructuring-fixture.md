# Lesson 13 — Tổng hợp kiến thức 

> Tài liệu này tổng hợp lại toàn bộ kiến thức rút ra từ việc đọc code thực tế trong 3 thư mục bài làm:
> - `tests/students-submission/tuyen-vo/lesson-13/js-destructuring/` — object destructuring
> - `tests/students-submission/tuyen-vo/lesson-13/demo/` — fixture (câu chuyện từ chưa có đến có custom fixture)
> - `tests/students-submission/tuyen-vo/lesson-13/playwright-fixture/` — bài tập áp dụng fixture + POM cho 4 trang khác nhau
>
> Kết hợp thêm kiến thức nền tảng JavaScript/TypeScript và ví dụ thực tế ngoài dự án học.

---

## Mục lục

1. [Object Destructuring](#1-object-destructuring)
2. [Playwright Fixture](#2-playwright-fixture)
3. [Page Object Model (POM) kết hợp Fixture](#3-page-object-model-pom-kết-hợp-fixture)
4. [Case study thực tế: lỗi đã gặp khi refactor sang fixture](#4-case-study-thực-tế-lỗi-đã-gặp-khi-refactor-sang-fixture)
5. [Các Playwright API hay dùng trong bài làm](#5-các-playwright-api-hay-dùng-trong-bài-làm)
6. [Ví dụ thực tế trong dự án thật](#6-ví-dụ-thực-tế-trong-dự-án-thật)
7. [Bảng tổng kết / cheat sheet](#7-bảng-tổng-kết--cheat-sheet)

---

## 1. Object Destructuring

### 1.1. Destructuring là gì?

Destructuring (phá cấu trúc) là cú pháp JavaScript/TypeScript cho phép **lấy nhiều giá trị từ 1 object (hoặc mảng) ra thành các biến riêng lẻ, chỉ trong 1 dòng code**, thay vì phải viết `object.propertyName` lặp lại nhiều lần.

**Không dùng destructuring:**
```js
const person = { firstName: "John", lastName: "Doe", age: 30 };

const firstName = person.firstName;
const lastName = person.lastName;
const age = person.age;
```

**Dùng destructuring** ([js-destructuring/01-destructuring.js](js-destructuring/01-destructuring.js)):
```js
const person = { firstName: "John", lastName: "Doe", age: 30, occupation: "Software Engineer" };

const { firstName, lastName, age } = person;
console.log(firstName);
console.log(lastName);
console.log(age);
```
`occupation` không được lấy ra vì không có mặt trong `{ }` bên trái — không lỗi gì, chỉ đơn giản bị bỏ qua. Tên biến sau khi destructure **mặc định phải trùng tên property**, vì bản chất đây là cách viết tắt của `object.propertyName`.

Cùng logic áp dụng cho object bất kỳ, ví dụ [js-destructuring/02-destructuring.js](js-destructuring/02-destructuring.js) với object `car { brand, model, year, color }`.

### 1.2. Giá trị mặc định (default value)

Khi property **không tồn tại** (hoặc là `undefined`) trong object, có thể khai báo giá trị thay thế ngay trong cú pháp destructure bằng dấu `=`.

[js-destructuring/03-destructuring.js](js-destructuring/03-destructuring.js):
```js
const user = {};
const { name = 'Guest' } = user;
console.log(name); // 'Guest' — vì user không có key "name"
```

[js-destructuring/04-destructuring.js](js-destructuring/04-destructuring.js): tương tự với số — `const { price = 0 } = product;`.

> Lưu ý: giá trị mặc định **chỉ** áp dụng khi property là `undefined` (không tồn tại hoặc gán `undefined` tường minh). Nếu property tồn tại với giá trị `null`, `0`, `''`, `false`... thì giá trị đó vẫn được giữ nguyên, **không** bị thay bằng default.

### 1.3. Đổi tên biến khi destructure (renaming / alias)

Cú pháp `{ propertyName: newVariableName }` — lấy giá trị của `propertyName` nhưng gán vào 1 biến có **tên khác**. Sau dòng này, biến tên `propertyName` **không tồn tại**, chỉ có biến `newVariableName`.

[js-destructuring/05-destructuring.js](js-destructuring/05-destructuring.js):
```js
const book = { title: "Dac Nhan Tam" };
const { title: bookTitle } = book;
console.log(bookTitle); // "Dac Nhan Tam"
// console.log(title);  // ❌ lỗi — biến "title" không tồn tại
```

[js-destructuring/06-destructuring.js](js-destructuring/06-destructuring.js) minh hoạ tương tự: `{ director: filmDirector } = movie`.

**Lý do cần đổi tên:** tránh trùng tên biến đã có sẵn trong scope, hoặc muốn tên biến rõ nghĩa hơn theo ngữ cảnh code hiện tại.

### 1.4. Destructuring lồng nhau (nested destructuring)

Khi giá trị của 1 property lại là **1 object khác**, có thể "đào sâu" vào bên trong để lấy thẳng ra property con, bỏ qua object cha ở giữa.

[js-destructuring/07-destructuring.js](js-destructuring/07-destructuring.js):
```js
const person = {
    address: { street: "Cong Hoa", city: "Ho Chi Minh", country: "viet Nam" }
};

const { address: { street } } = person;
console.log(street); // "Cong Hoa"
// biến "address" KHÔNG tồn tại — chỉ có biến "street"
```

Điểm dễ nhầm: `{ address: { street } }` **khác hoàn toàn** với renaming ở mục 1.3 (`{ title: bookTitle }`). Vì vế phải sau dấu `:` ở đây là **1 object pattern** (`{ street }`), không phải 1 tên biến đơn — nên JS/TS hiểu là "đi tiếp vào bên trong `address` để destructure `street`", chứ không phải "đổi tên address". Viết tương đương không dùng destructure: `const street = person.address.street;`.

[js-destructuring/08-destructuring.js](js-destructuring/08-destructuring.js) áp dụng tương tự: `{ details: { model } } = product` → chỉ lấy `product.details.model`.

### 1.5. Kết hợp nhiều kỹ thuật trong 1 dòng

[demo/01-destructure.js](demo/01-destructure.js) minh hoạ việc **gộp cả 3 kỹ thuật** (default value + renaming + lấy bình thường) vào chung 1 câu lệnh:

```js
const student = { name: 'Tuyen', address: 'Ho Chi Minh' };

const { fullName = 'Tuyen Vo', name: lastName, address } = student;

console.log(lastName);  // "Tuyen" (đổi tên từ "name")
console.log(fullName);  // "Tuyen Vo" (default, vì student không có key "fullName")
console.log(address);   // "Ho Chi Minh" (lấy bình thường)
```

Điều này cho thấy 3 kỹ thuật **không loại trừ nhau** — có thể tự do phối hợp trong cùng 1 câu lệnh destructure.

### 1.6. Vì sao destructuring quan trọng với Playwright

Cú pháp quen thuộc `async ({ page }) => {...}` mà bạn viết trong **mọi** file test **chính là object destructuring**. Playwright gọi callback của `test()` với 1 object chứa rất nhiều fixture (`page`, `context`, `browser`, `request`, `browserName`, và cả **custom fixture** bạn tự định nghĩa như `materialPage`, `registerPage`...) — và bạn chỉ destructure ra đúng (các) fixture mình cần dùng trong test đó.

```ts
test('...', async ({ page, browserName }) => { ... });          // lấy 2 fixture
test('...', async ({ materialPage }) => { ... });                // lấy 1 custom fixture
test('...', async ({ registerPage: myRegisterPage }) => { ... }); // vừa lấy vừa đổi tên
```

Không hiểu destructuring → sẽ khó hiểu bản chất tại sao viết `{ page }` lại "tự nhiên có sẵn" 1 page để dùng, hoặc tại sao có thể đổi tên biến fixture khi cần (`{ personalNotesPage: personalNote }`).

---

## 2. Playwright Fixture

### 2.1. Fixture là gì và giải quyết vấn đề gì?

Fixture là cơ chế của Playwright Test giúp **chuẩn bị sẵn (setup)** dữ liệu/đối tượng cần thiết **trước khi** test chạy, và **tự động dọn dẹp (teardown)** sau khi test chạy xong — test case chỉ việc "nhận" đối tượng đã chuẩn bị sẵn để dùng luôn, không phải tự khởi tạo.

Fixture giải quyết các bài toán:
- **Tái sử dụng** setup/teardown code — không phải viết lại `beforeEach` giống nhau ở nhiều file.
- **Chia sẻ object** giữa các test (ví dụ 1 instance POM đã điều hướng sẵn).
- **Cô lập môi trường** — mỗi test có `page`/`context` riêng, không ảnh hưởng lẫn nhau.
- **Mở rộng** fixture có sẵn (`page`, `context`, `browser`) thành fixture riêng của dự án.

### 2.2. Câu chuyện tiến hoá: từ code lặp lại đến custom fixture

Thư mục `demo/` kể lại đúng hành trình 4 bước này — nên đọc theo đúng thứ tự file để thấy vấn đề dần lộ ra và được giải quyết ra sao.

**Bước 1 — Code thô, lặp lại** ([demo/03-without-fixture.spec.ts](demo/03-without-fixture.spec.ts), phần đầu):
```ts
test("Test 1: Registration Page", async({ page }) => {
    await page.goto("https://material.playwrightvn.com/");
    await page.getByText("Bài học 1: Register Page (có đủ các element)").click();
    await expect(page.getByText("User Registration")).toBeVisible();
});

test("Test 2: Product Page", async({ page }) => {
    await page.goto("https://material.playwrightvn.com/");   // <-- lặp lại
    await page.getByText("Bài học 2: Product page").click();
    await expect(page.getByText("Simple E-commerce")).toBeVisible();
});
```
→ Bước `page.goto(...)` bị copy-paste ở mọi test.

**Bước 2 — Gom bằng `test.describe` + `test.beforeEach`** (cùng file, phần giữa):
```ts
test.describe("Material page - without POM", () => {
    test.beforeEach(async({ page }) => {
        await page.goto("https://material.playwrightvn.com/");
    })

    test("Test 1: Registration Page", async({ page }) => {
        await page.getByText("Bài học 1: Register Page (có đủ các element)").click();
        await expect(page.getByText("User Registration")).toBeVisible();
    });
    // ...
});
```
`beforeEach` chạy 1 lần **trước mỗi test** trong `describe` này → hết lặp code điều hướng trong file, nhưng vẫn thao tác trực tiếp bằng `page.getByText(...)`.

**Bước 3 — Kết hợp POM + `beforeEach`** (cùng file, phần cuối):
```ts
test.describe("Material page - with POM", () => {
    let materialPage : MaterialPage;

    test.beforeEach(async({ page }) => {
        materialPage = new MaterialPage(page);
        await materialPage.go();
        await expect(materialPage.page.getByText("Tài liệu học automation test")).toBeVisible();
    })

    test("Test 1: Registration Page", async({ page }) => {
        await materialPage.page.getByText("Bài học 1: Register Page (có đủ các element)").click();
        await expect(materialPage.page.getByText("User Registration")).toBeVisible();
    });
});
```

**Vấn đề lộ ra ở file khác** ([demo/04-without-fixture.spec.ts](demo/04-without-fixture.spec.ts)) — file test cho **Todo page**, nhưng phải **copy nguyên khối** `let materialPage; beforeEach(...)` y hệt file 03:
```ts
test.describe("Material page - with POM2", () => {
    let materialPage : MaterialPage;
    test.beforeEach(async({ page }) => {
        materialPage =  new MaterialPage(page);
        await materialPage.go();
        await expect(materialPage.page.getByText("Tài liệu học automation test")).toBeVisible();
    })

    test("Test 3: Todo Page", async({ page }) => {
        await materialPage.page.getByText("Bài học 3: Todo page").click();
        await expect(materialPage.page.getByText("To-Do List")).toBeVisible();
    });
});
```

**Bài học cốt lõi:** cách 3 (POM + `beforeEach`) giải quyết được lặp code **trong 1 file**, nhưng khi có nhiều file test khác nhau, phải **copy-paste y hệt đoạn `beforeEach` sang mọi file mới**. Nếu sau này logic `go()` đổi (ví dụ thêm bước đăng nhập trước khi vào trang), phải sửa ở tất cả các file — dễ sót, dễ sai lệch.

**Bước 4 — Giải pháp: custom fixture** ([demo/00-fixture.ts](demo/00-fixture.ts) + [demo/05-with-fixture.spec.ts](demo/05-with-fixture.spec.ts) + [demo/06-with-fixture.spec.ts](demo/06-with-fixture.spec.ts)):

```ts
// 00-fixture.ts — viết setup MỘT LẦN DUY NHẤT
import { test as base, expect } from '@playwright/test';
import { MaterialPage } from './00-pom';

const test = base.extend<{materialPage: MaterialPage}>({
    materialPage: async ({ page }, use) => {
        const materialPage = new MaterialPage(page);
        await materialPage.go();
        await expect(materialPage.page.getByText("Tài liệu học automation test")).toBeVisible();

        await use(materialPage);        // (A) test chạy tại đây

        console.log('Test end');        // (B) chạy SAU khi test xong (teardown)
    }
});

export { test };
```

```ts
// 05-with-fixture.spec.ts — file 1
import { test } from './00-fixture';   // import test ĐÃ ĐƯỢC MỞ RỘNG, không phải từ '@playwright/test'

test("Test 1: Registration Page", async({ materialPage }) => {
    await materialPage.page.getByText("...").click();
    await expect(materialPage.page.getByText("User Registration")).toBeVisible();
});
```

```ts
// 06-with-fixture.spec.ts — file 2 (Todo), KHÔNG cần viết lại beforeEach nào cả
import { test } from './00-fixture';

test.describe("Material page - with POM2", () => {
    test("Test 3: Todo Page", async({ materialPage }) => {
        await materialPage.page.getByText("Bài học 3: Todo page").click();
        await expect(materialPage.page.getByText("To-Do List")).toBeVisible();
    });
});
```

### 2.3. Cơ chế setup / use / teardown

```ts
materialPage: async ({ page }, use) => {
    // ---- SETUP: chạy TRƯỚC mỗi test dùng fixture này ----
    const materialPage = new MaterialPage(page);
    await materialPage.go();

    await use(materialPage);   // ---- giao đối tượng cho test dùng, test chạy tại đây ----

    // ---- TEARDOWN: chạy SAU khi test đã chạy xong (dù pass hay fail) ----
    console.log('Test end');
}
```
- Code **trước** `await use(...)` = phần **setup**, tương đương `beforeEach`.
- `await use(value)` = "giao" `value` cho test case sử dụng làm giá trị của fixture đó.
- Code **sau** `await use(...)` = phần **teardown**, tương đương `afterEach` — luôn chạy kể cả khi test bên trong throw lỗi.

### 2.4. Built-in fixture có sẵn của Playwright

Minh hoạ trong [playwright-fixture/01-fixture-context.spec.ts](playwright-fixture/01-fixture-context.spec.ts), [playwright-fixture/02-fixture-browser.spec.ts](playwright-fixture/02-fixture-browser.spec.ts) và [demo/02-fixture.spec.ts](demo/02-fixture.spec.ts):

| Fixture | Kiểu | Mô tả | Quan hệ |
|---|---|---|---|
| `page` | `Page` | Playwright tự tạo sẵn 1 context + 1 page riêng cho mỗi test — dùng nhiều nhất (~99% test). | 1 `page` thuộc về 1 `context` |
| `context` | `BrowserContext` | 1 "phiên làm việc" độc lập; có thể tạo **nhiều** `page` (`context.newPage()`) — các page này **dùng chung** cookie/session (giống nhiều tab cùng 1 trình duyệt, cùng đăng nhập). | 1 `context` chứa nhiều `page` |
| `browser` | `Browser` | Trình duyệt gốc; có thể tạo **nhiều** `context` độc lập (`browser.newContext()`) — mỗi context **không chung** cookie/session (mô phỏng nhiều user khác nhau). | 1 `browser` chứa nhiều `context` |
| `browserName` | `string` | `'chromium' \| 'firefox' \| 'webkit'` — dùng để `test.skip(...)` có điều kiện theo trình duyệt. | — |
| `request` | `APIRequestContext` | Gọi API độc lập, không qua UI (đã học ở lesson 11/12). | — |

```ts
// Chỉ dùng page — đơn giản, phổ biến nhất
test('...', async ({ page }) => {
    await page.goto('https://material.playwrightvn.com');
});

// Dùng context — nhiều tab CHUNG session (VD: user đăng nhập rồi mở thêm tab)
test('...', async ({ context }) => {
    const page1 = await context.newPage();
    await page1.goto('https://material.playwrightvn.com');

    const page2 = await context.newPage();   // vẫn chung session với page1
    await page2.goto('https://e-commerce-dev.betterbytesvn.com/');
});

// Dùng browser — nhiều context ĐỘC LẬP (VD: 2 user khác nhau trong cùng 1 test)
test('...', async ({ browser }) => {
    const context1 = await browser.newContext();
    const page1 = await context1.newPage();

    const context2 = await browser.newContext();   // session hoàn toàn tách biệt context1
    const page2 = await context2.newPage();
});
```

**Khi nào dùng cái nào:**
- `page`: mặc định cho hầu hết test đơn giản.
- `context`: cần nhiều tab **cùng 1 phiên đăng nhập** (VD: test mở link ở tab mới rồi thao tác tiếp).
- `browser`: cần **2 user độc lập** tương tác với nhau trong cùng 1 test (VD: test chat, test A gửi lời mời kết bạn cho B).

### 2.5. Tự viết custom fixture (`test.extend`)

Cú pháp tổng quát:
```ts
import { test as base } from '@playwright/test';

const test = base.extend<{ fixtureName: FixtureType }>({
    fixtureName: async ({ /* fixture khác cần dùng, VD page */ }, use) => {
        // setup
        const value = /* tạo giá trị */;
        await use(value);
        // teardown
    }
});

export { test };
```
- `test as base`: đổi tên `test` gốc thành `base` để tránh trùng tên với biến `test` mới (đã mở rộng) khai báo ngay bên dưới.
- `base.extend<T>(...)`: mở rộng object `test` mặc định, thêm (các) fixture mới theo type `T`.
- Tham số đầu của callback (dạng `{ page }`) — nơi destructure các fixture **khác** (kể cả built-in như `page`, hoặc custom fixture khác) mà fixture mới này cần dùng tới.
- File cuối cùng phải `export { test }` để các file `.spec.ts` khác **import đúng `test` đã custom này**, thay vì import `test` gốc từ `@playwright/test`.

### 2.6. Khai báo type cho `extend`: inline object type vs. type alias

Hai cách viết sau **hoàn toàn tương đương về mặt type**, chỉ khác cách tổ chức code:

```ts
// Cách 1 — inline object type literal (viết type ngay tại chỗ dùng)
const test = base.extend<{ materialPage: MaterialPage }>({...});
```

```ts
// Cách 2 — type alias (đặt tên type riêng rồi tham chiếu lại)
type MaterialFixtures = {
    registerPage: RegisterPage;
    productPage: ProductPage;
    todoPage: TodoPage;
    personalNotesPage: PersonalNotesPage;
};

const test = base.extend<MaterialFixtures>({...});
```

- **Cách 1** phù hợp khi chỉ có **1-2 fixture đơn giản** — viết inline cho gọn, không cần đặt tên riêng.
- **Cách 2** nên dùng khi có **nhiều fixture** — nếu vẫn viết inline, dòng `extend<{...}>` sẽ rất dài và khó đọc. Tách ra `type MaterialFixtures` giúp dòng `extend<MaterialFixtures>` gọn, dễ đọc lướt, và có thể tái sử dụng type này ở nơi khác nếu cần.
- Không có khác biệt gì ở runtime — type bị xoá hoàn toàn khi TypeScript biên dịch ra JavaScript, đây thuần tuý là vấn đề khả năng đọc/bảo trì code.

### 2.7. Thiết kế fixture khi có NHIỀU POM khác nhau cho NHIỀU file test

Đây là tình huống thực tế gặp phải ở `playwright-fixture/` — có 4 file test (`test1` → Register, `test2` → Product, `test3` → Todo, `test4` → Personal Notes), mỗi file cần 1 class POM khác nhau (đều kế thừa từ `MaterialPage` base). Có 2 hướng thiết kế:

**Hướng A — 1 fixture dùng chung, tự bọc class con trong từng test file** (đây là cách **đã áp dụng thực tế** trong `test1.spec.ts` hiện tại):

```ts
// 02-fixture.ts — chỉ có đúng 1 fixture, trả về class MaterialPage (base)
const test = base.extend<{materialPage: MaterialPage}>({
    materialPage: async ({ page }, use) => {
        const materialPage = new MaterialPage(page);
        await materialPage.openMaterialPage();
        await expect(materialPage.page.getByText('Tài liệu học automation test')).toBeVisible();
        await use(materialPage);
    }
});
```

```ts
// test1.spec.ts — tự tạo RegisterPage từ page gốc bên trong materialPage
test('User Resistration', async ({ materialPage }) => {
    const registerPage = new RegisterPage(materialPage.page);   // <-- lấy .page ra, không truyền thẳng materialPage

    await test.step('Click Bai hoc 1', async () => {
        await materialPage.gotoPage('Bài 1');   // điều hướng dùng chung method của base class
    });

    await test.step('Input user name', async () => {
        await registerPage.fillUsername('Tuyen Vo');   // thao tác riêng dùng method của class con
    });
    // ...
});
```

**Hướng B — Nhiều fixture riêng biệt, mỗi fixture trả sẵn đúng class con cần dùng:**

```ts
type MaterialFixtures = {
    registerPage: RegisterPage;
    productPage: ProductPage;
    todoPage: TodoPage;
    personalNotesPage: PersonalNotesPage;
};

const test = base.extend<MaterialFixtures>({
    registerPage: async ({ page }, use) => {
        const registerPage = new RegisterPage(page);
        await registerPage.openMaterialPage();
        await use(registerPage);
    },
    productPage: async ({ page }, use) => { /* tương tự */ },
    todoPage: async ({ page }, use) => { /* tương tự */ },
    personalNotesPage: async ({ page }, use) => { /* tương tự */ },
});
```
Mỗi test file chỉ destructure đúng fixture cần dùng — Playwright chỉ khởi tạo fixture nào **thực sự được destructure** trong test đó (lazy), nên khai báo cả 4 trong 1 file `extend` không hề lãng phí:
```ts
test('User Resistration', async ({ registerPage }) => { ... });   // chỉ registerPage được khởi tạo
test('Add products', async ({ productPage }) => { ... });         // chỉ productPage được khởi tạo
```

**So sánh:**

| | Hướng A (1 fixture chung) | Hướng B (nhiều fixture riêng) |
|---|---|---|
| Số lượng fixture cần viết | 1 | 1 cho mỗi loại POM (ở đây là 4) |
| Trong test file | Phải tự `new XxxPage(materialPage.page)` | Không cần dòng `new` nào — dùng thẳng |
| Độ dài file fixture | Ngắn | Dài hơn (lặp cấu trúc 4 lần) |
| Phù hợp khi | Muốn ít thay đổi, học fixture cơ bản | Muốn thực hành đúng pattern "fixture chuyên biệt hoá" hay gặp ở dự án thật |

---

## 3. Page Object Model (POM) kết hợp Fixture

### 3.1. Cấu trúc kế thừa dùng trong bài làm

File [playwright-fixture/01-pom.ts](playwright-fixture/01-pom.ts) thiết kế 1 class cha và 4 class con:

```
MaterialPage (base)
 ├─ page: Page
 ├─ constructor(page)
 ├─ openMaterialPage()      → goto trang chủ Material
 └─ gotoPage(pageName)      → click vào đúng link "Bài 1/2/3/4"

 ├── RegisterPage extends MaterialPage       (fillUsername, fillEmail, checkGender, captureFormValues...)
 ├── ProductPage extends MaterialPage        (addProduct1/2/3, getCartItems, getCartTotalPrice...)
 ├── TodoPage extends MaterialPage           (fillTodo, getTodoItemLocator, deleteOddTasks...)
 └── PersonalNotesPage extends MaterialPage  (addNotes, searchNotes, getVisibleNotesText...)
```

Class cha `MaterialPage` gom lại **hành vi và locator dùng chung** cho cả 4 trang con (mở trang chủ, điều hướng tới từng bài học). Mỗi class con kế thừa toàn bộ hành vi đó (`super(page)` gọi constructor cha) và bổ sung thêm locator/method **riêng** cho đúng trang của mình.

**Lợi ích của kế thừa ở đây:** nếu URL trang chủ Material đổi, hoặc cách điều hướng đổi (VD: thêm bước xác nhận cookie trước khi vào trang), chỉ cần sửa **1 chỗ** trong `MaterialPage` — cả 4 class con tự động được cập nhật theo, không phải sửa lặp lại 4 lần.

### 3.2. Vì sao fixture + POM kết hợp là bộ đôi mạnh

- **POM** giải quyết: gom locator + hành động thành method có tên rõ nghĩa, tránh lặp code thao tác DOM trong file test.
- **Fixture** giải quyết: gom bước **khởi tạo + điều hướng ban đầu** của POM đó, tránh lặp code setup giữa các file test.

Khi kết hợp, file test cuối cùng chỉ còn lại đúng **logic nghiệp vụ** của kịch bản test, không còn bất kỳ dòng code "dọn dẹp môi trường" nào:

```ts
test('User Resistration', async ({ registerPage }) => {
    await registerPage.gotoPage('Bài 1');
    await registerPage.fillUsername('Tuyen Vo');
    await registerPage.fillEmail('tuyen@gmail.com');
    // ...toàn bộ đọc lên như 1 kịch bản nghiệp vụ, không lẫn code kỹ thuật setup
});
```

---

## 4. Case study thực tế: lỗi đã gặp khi refactor sang fixture

Đây là lỗi **thật sự đã xảy ra** trong quá trình làm bài, rất đáng ghi lại vì là lỗi rất hay gặp khi mới chuyển từ POM thuần (lesson 10) sang POM + fixture (lesson 13).

### 4.1. Lỗi 1 — biến bị comment nhưng vẫn được dùng ở dưới

Code gốc trong `test1.spec.ts`:
```ts
test('User Resistration', async ({ materialPage }) => {
    //const registerPage = new RegisterPage(materialPage);   // <-- dòng khai báo bị comment

    await test.step('Input user name', async () => {
        await registerPage.fillUsername('Tuyen Vo');   // <-- vẫn gọi registerPage ở đây!
    });
    // ... nhiều chỗ khác cũng gọi registerPage
```

**Lỗi:** biến `registerPage` không hề tồn tại trong scope (vì dòng khai báo nó đã bị comment) → TypeScript báo **"Cannot find name 'registerPage'"** tại mọi chỗ dùng nó, runtime sẽ là `ReferenceError: registerPage is not defined`.

**Bài học:** khi comment tạm 1 dòng để debug/thử nghiệm, phải rà soát lại toàn bộ các dòng phía dưới có phụ thuộc vào biến đó hay không, trước khi để code ở trạng thái "final".

### 4.2. Lỗi 2 — sai kiểu dữ liệu khi bỏ comment (type mismatch)

Nếu chỉ đơn giản bỏ comment dòng trên:
```ts
const registerPage = new RegisterPage(materialPage);   // ❌ vẫn lỗi
```
sẽ phát sinh lỗi type mới: constructor của `RegisterPage` yêu cầu tham số kiểu `Page` (Playwright's Page object), nhưng `materialPage` (lấy từ fixture) có kiểu là `MaterialPage` — 2 kiểu này **không tương thích**. TypeScript báo: *"Argument of type 'MaterialPage' is not assignable to parameter of type 'Page'"*.

**Cách sửa đã áp dụng (Hướng A):** lấy đúng property `page` bên trong `materialPage` ra để truyền vào:
```ts
const registerPage = new RegisterPage(materialPage.page);   // ✅ đúng — materialPage.page có kiểu Page
```

**Bài học:** khi 1 fixture trả về **1 instance của class POM** (chứ không phải trả thẳng `Page`), muốn tạo thêm 1 class POM khác từ cùng `page` gốc, phải nhớ lấy `.page` ra trước, không truyền thẳng cả instance class vào.

---

## 5. Các Playwright API hay dùng trong bài làm

Tổng hợp nhanh các API xuất hiện trong `01-pom.ts` (register/product/todo/personal notes), rất hay bị dùng nhầm lẫn:

| API | Dùng cho | Trả về | Lưu ý |
|---|---|---|---|
| `.inputValue()` | `<input>`, `<textarea>`, `<select>` | giá trị hiện tại đã nhập/chọn (string) | Đọc đúng những gì user đã gõ/chọn thực tế trên DOM |
| `.getAttribute('value')` | Bất kỳ element nào có attribute `value` khai báo sẵn trong HTML | string \| null | Đọc attribute **gốc trong HTML**, không phải giá trị runtime |
| `.isChecked()` | checkbox / radio | boolean | Đọc thẳng property `checked`, **không yêu cầu element phải visible** |
| `.click()` | Bất kỳ element nào | — | **Yêu cầu element phải visible** (bounding box > 0) — sẽ timeout nếu element bị `width:0;height:0` (checkbox ẩn dưới custom toggle switch) |
| `.toBeInViewport()` | Kiểm tra element có đang nằm trong khung nhìn trình duyệt | assertion | Có "auto-wait/retry" — cần element **tồn tại trong DOM** |
| `.toHaveCount(0)` | Kiểm tra element đã bị xoá khỏi DOM hẳn (không phải chỉ ẩn bằng CSS) | assertion | Khác với `.toBeHidden()` — element ẩn bằng CSS vẫn đếm được 1, chỉ khi xoá khỏi DOM mới đếm được 0 |
| `.toBeCloseTo(value, precision)` | So sánh số thực (float) | assertion | An toàn hơn `.toBe()` khi có sai số làm tròn nhị phân (VD: `0.1 + 0.2 !== 0.3` trong JS) |

**Case điển hình đã gặp trong bài (`xpathEnableFeatureClick` vs `xpathEnableFeatureState`):** 1 custom toggle switch có `<input type="checkbox">` bị CSS ẩn hoàn toàn (`width:0;height:0`), phần hiển thị thật là `<span class="slider round">`. Vì vậy:
- Để **click** (thao tác như user thật) → phải nhắm vào `<span>` (phần tử visible).
- Để **đọc trạng thái checked** (trong `captureFormValues()`) → phải nhắm vào `<input>` bằng `.isChecked()` (không cần visible).

Đây là bài học chung: **1 hành động và 1 kiểm tra trạng thái trên cùng 1 "control" có thể cần 2 locator khác nhau**, tuỳ theo API đó có yêu cầu visibility hay không.

---

## 6. Ví dụ thực tế trong dự án thật

### 6.1. Destructuring — parse response API lồng nhau

```js
const response = await request.get('/api/user/1');
const { data: { profile: { email, avatarUrl } } } = await response.json();
```
Thay vì viết `json.data.profile.email`, `json.data.profile.avatarUrl` lặp lại — destructure lồng lấy thẳng ra 2 biến cần dùng.

### 6.2. Destructuring — đọc config với giá trị mặc định

```js
const { BASE_URL = 'https://staging.example.com', HEADLESS = 'true' } = process.env;
```
Nếu biến môi trường chưa được set (VD: chạy local chưa cấu hình `.env`), test vẫn chạy được với giá trị mặc định thay vì bị `undefined`.

### 6.3. Fixture — đăng nhập sẵn dùng chung cho hàng chục file test

```ts
// fixtures/auth.fixture.ts
export const test = base.extend<{ loggedInPage: Page }>({
    loggedInPage: async ({ page }, use) => {
        await page.goto('/login');
        await page.fill('#username', 'admin');
        await page.fill('#password', 'secret');
        await page.click('#login-btn');
        await expect(page.getByText('Dashboard')).toBeVisible();

        await use(page);   // page ở trạng thái "đã đăng nhập" được giao cho test

        // teardown: ví dụ xoá data test vừa tạo trong DB, hoặc logout
    }
});
```
Mọi file test nghiệp vụ (`order.spec.ts`, `profile.spec.ts`, `settings.spec.ts`...) chỉ cần:
```ts
test('Xem đơn hàng', async ({ loggedInPage }) => {
    await loggedInPage.goto('/orders');
    // ...
});
```
— không phải copy-paste đoạn "goto + fill + click + verify Dashboard" vào từng file, tránh đúng cái bẫy đã thấy ở `demo/04-without-fixture.spec.ts`.

### 6.4. Fixture — mô phỏng nhiều role/nhiều user trong 1 test

```ts
test('Admin duyệt bài viết do user thường tạo', async ({ browser }) => {
    const userContext = await browser.newContext({ storageState: 'auth/user.json' });
    const adminContext = await browser.newContext({ storageState: 'auth/admin.json' });

    const userPage = await userContext.newPage();
    await userPage.goto('/create-post');
    // user tạo bài viết...

    const adminPage = await adminContext.newPage();
    await adminPage.goto('/admin/review');
    // admin duyệt bài viết vừa tạo...
});
```
Đây chính là ứng dụng thực tế của fixture `browser` (mỗi `newContext()` độc lập session) đã học ở mục 2.4.

### 6.5. POM + fixture chuyên biệt hoá (Hướng B) — dự án e-commerce thật

```ts
type ShopFixtures = {
    homePage: HomePage;
    cartPage: CartPage;
    checkoutPage: CheckoutPage;
};

const test = base.extend<ShopFixtures>({
    homePage: async ({ page }, use) => {
        const homePage = new HomePage(page);
        await homePage.goto();
        await use(homePage);
    },
    cartPage: async ({ page }, use) => {
        await use(new CartPage(page));
    },
    checkoutPage: async ({ page }, use) => {
        await use(new CheckoutPage(page));
    },
});
```
Mỗi team/module (giỏ hàng, thanh toán, trang chủ) có fixture riêng, file test chỉ destructure đúng cái cần — giống hệt cấu trúc bạn đã cân nhắc cho 4 trang Register/Product/Todo/Personal Notes.

---

## 7. Bảng tổng kết / cheat sheet

| Chủ đề | Ghi nhớ chính |
|---|---|
| **Destructuring cơ bản** | `const { a, b } = obj;` — tên biến phải trùng tên property (trừ khi đổi tên) |
| **Default value** | `const { a = defaultVal } = obj;` — chỉ áp dụng khi `a` là `undefined` |
| **Renaming** | `const { a: newName } = obj;` — biến `a` không tồn tại, chỉ có `newName` |
| **Nested destructuring** | `const { a: { b } } = obj;` — đào sâu vào `a` để lấy `b`, biến `a` không tồn tại |
| **Fixture setup/teardown** | Code trước `use()` = setup (như `beforeEach`); code sau `use()` = teardown (như `afterEach`) |
| **Built-in fixtures** | `page` (1 page/context riêng) ⊂ `context` (nhiều page, chung session) ⊂ `browser` (nhiều context, độc lập session) |
| **Custom fixture** | `base.extend<Type>({ fixtureName: async ({...}, use) => {...} })`, rồi `export { test }` |
| **Type inline vs alias** | Không khác gì về runtime — alias (`type Xxx = {...}`) chỉ giúp dễ đọc khi có nhiều fixture |
| **Hướng A vs Hướng B** | A = 1 fixture chung + tự `new XxxPage(base.page)` trong test; B = nhiều fixture riêng, mỗi cái trả sẵn đúng class con |
| **Bug thường gặp** | Comment 1 dòng khai báo biến nhưng quên rà lại chỗ dùng phía dưới → ReferenceError; truyền nhầm instance class POM thay vì `.page` gốc → type mismatch |
| **API dễ nhầm** | `.isChecked()` (không cần visible) vs `.click()` (bắt buộc visible); `.toHaveCount(0)` (đã xoá khỏi DOM) vs `.toBeHidden()` (chỉ ẩn CSS, vẫn còn trong DOM) |

**Luồng phát triển 1 bộ test "chuẩn" áp dụng cho bài `playwright-fixture/`:**
```
1. Viết POM base (MaterialPage) + các POM con (RegisterPage, ProductPage...)   → 01-pom.ts
2. Viết custom fixture bọc POM, tự openMaterialPage() + assert trang đã load   → 02-fixture.ts
3. Test case chỉ destructure fixture ra dùng, không còn code setup thủ công   → test1-4.spec.ts
4. Khi có bug (ReferenceError, type mismatch...) → đọc kỹ thông báo lỗi TypeScript,
   xác định đúng biến/kiểu dữ liệu đang sai trước khi sửa
```
