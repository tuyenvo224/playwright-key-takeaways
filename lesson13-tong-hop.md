# Lesson 13 — Object Destructuring, Fixture, Test Generator, Video Recording

> Nguồn: PlaywrightVN — Học automation test từ chưa biết gì (slide + code thực hành `lesson-13/`)

---

## Mục lục

1. [Nội dung buổi học](#1-nội-dung-buổi-học)
2. [Object Destructuring](#2-object-destructuring)
3. [Fixture trong Playwright](#3-fixture-trong-playwright)
4. [Test Generator (Codegen)](#4-test-generator-codegen)
5. [Video Recording](#5-video-recording)
6. [Tổng kết Lesson 13](#6-tổng-kết-lesson-13)

---

## 1. Nội dung buổi học

Buổi học gồm 4 chủ đề:

1. **Object destructuring** — cú pháp JavaScript giúp code ngắn gọn hơn khi lấy dữ liệu từ object.
2. **Fixtures** — cơ chế "bơm sẵn" môi trường/đối tượng cần thiết vào test của Playwright.
3. **Test generator** — công cụ tự sinh code test bằng cách "click" (codegen/record).
4. **Video recording** — quay video lại quá trình chạy test để debug và làm bằng chứng.

---

## 2. Object Destructuring

### 2.1. Destructuring là gì?

- **Destruct** = phá hủy, dỡ bỏ.
- Trong JavaScript: destructuring = **lấy giá trị của các thuộc tính bên trong object ra thành các biến riêng lẻ**.
- Mục đích: giúp code **ngắn gọn** hơn, đỡ phải viết `object.propertyName` nhiều lần.

### 2.2. Ví dụ khi CHƯA dùng destructuring

Mỗi thuộc tính phải viết riêng **1 dòng code**:

```js
const myClass = {
  school: 'BBA',
  course: 'Full-stack QA',
};

const school = myClass.school;
const course = myClass.course;

console.log(school); // => 'BBA'
console.log(course); // => 'Full-stack QA'
```

### 2.3. Ví dụ khi ĐÃ dùng destructuring

Tất cả thuộc tính chỉ tốn **1 dòng code duy nhất**:

```js
const myClass = {
  school: 'BBA',
  course: 'Full-stack QA',
};

// const school = myClass.school;   // ❌ không cần nữa
// const course = myClass.course;   // ❌ không cần nữa

const { school, course } = myClass; // ✅ destructuring

console.log(school); // => 'BBA'
console.log(course); // => 'Full-stack QA'
```

> Lưu ý: tên biến sau khi destructure phải **trùng tên thuộc tính** trong object (`school`, `course`), vì bản chất đây là cách viết tắt của việc gán `object.propertyName` vào biến cùng tên.

### 2.4. Ví dụ thực tế trong code (`01-destructuring.js`)

```js
const myClass = {
    school: 'BBA',
    course: 'Full stack QA'
};

const { course, school } = myClass;

console.log(school);
console.log(course);

// /////////////////////////////

const studen = {
    name: 'Tuyen',
    address: 'Ho Chi Minh'
};

const { name, address } = studen;

console.log(name);
console.log(address);
```

- Object `myClass` có 2 thuộc tính `school`, `course` → destructure ra 2 biến `school`, `course` **trong 1 dòng**, thứ tự viết trong `{ }` không quan trọng.
- Object `studen` có 2 thuộc tính `name`, `address` → destructure tương tự.

### 2.5. Bài thực hành

> Khai báo object `student` có 2 thuộc tính: `name`, `address`. Thực hiện destructuring object này và in ra console.

```js
const student = {
  name: 'Tuyen',
  address: 'Ho Chi Minh'
};

const { name, address } = student;

console.log(name);
console.log(address);
```

### 2.6. Vì sao topic này quan trọng với Playwright?

Trong các bài trước, khi viết test ta luôn viết `async ({ page }) => {...}`. Đây **chính là object destructuring** — Playwright truyền vào callback của `test()` một object chứa rất nhiều fixture (`page`, `context`, `browser`, `request`,...), và ta destructure ra fixture nào mình cần dùng. Hiểu destructuring giúp hiểu bản chất của **Fixture** ở phần tiếp theo.

---

## 3. Fixture trong Playwright

### 3.1. Fixture là gì?

Fixture là **cơ chế của Playwright Test** giúp:

```
Setup Fixture → Initial Data Provided Browser → Test Execution
       ↑                                              ↓
Create Test Data                          Test Assertions Validation
                                                       ↓
                                    Teardown: Browser close / Instance close /
                                              Temporary files deletion
```

Nói đơn giản: fixture là nơi **chuẩn bị sẵn** (setup) dữ liệu/đối tượng cần thiết **trước khi** test chạy, và **tự động dọn dẹp** (teardown) sau khi test chạy xong — test case chỉ việc "nhận" đối tượng đã chuẩn bị sẵn để dùng luôn.

### 3.2. Fixture giải quyết bài toán gì?

Fixture trong Playwright là cơ chế để:

- **Tái sử dụng** setup/teardown code (không phải viết lại `beforeEach`/`afterEach` giống nhau ở nhiều file).
- **Chia sẻ object** giữa các test (ví dụ chia sẻ 1 instance POM).
- **Tạo môi trường test độc lập** cho từng test (mỗi test có `page`/`context` riêng, không ảnh hưởng lẫn nhau).
- **Mở rộng** các built-in fixture có sẵn (`page`, `context`, `browser`...) thành fixture của riêng mình.
- **Nhóm test theo ngữ nghĩa** (theo ý nghĩa nghiệp vụ) thay vì nhóm theo "common setup" (dùng chung đoạn code setup).

### 3.3. Ví dụ khi CHƯA có fixture (viết theo kiểu thủ công)

**Cách 1 — "nông dân" nhất**, lặp lại code ở từng test (`02-fixture.spec.ts`):

```ts
test("Test 1: Registration Page", async({ page }) => {
    await page.goto("https://material.playwrightvn.com/");
    await page.getByText("Bài học 1: Register Page (có đủ các element)").click();
    await expect(page.getByText("User Registration")).toBeVisible();
});

test("Test 2: Product Page", async({ page }) => {
    await page.goto("https://material.playwrightvn.com/");
    await page.getByText("Bài học 2: Product page").click();
    await expect(page.getByText("Simple E-commerce")).toBeVisible();
});
```

→ Đoạn `page.goto(...)` bị lặp lại ở mọi test.

**Cách 2 — tối ưu hơn**, dùng `test.describe` + `test.beforeEach` để gom setup chung:

```ts
test.describe("Material page - without POM", () => {
    test.beforeEach(async({ page }) => {
        await page.goto("https://material.playwrightvn.com/");
    })

    test("Test 1: Registration Page", async({ page }) => {
        await page.getByText("Bài học 1: Register Page (có đủ các element)").click();
        await expect(page.getByText("User Registration")).toBeVisible();
    });

    test("Test 2: Product Page", async({ page }) => {
        await page.getByText("Bài học 2: Product page").click();
        await expect(page.getByText("Simple E-commerce")).toBeVisible();
    });
});
```

**Cách 3 — tối ưu hơn nữa**, kết hợp với POM (`MaterialPage`), nhưng vẫn phải **tự khởi tạo thủ công** trong `beforeEach`:

```ts
test.describe("Material page - with POM", () => {
    let materialPage: MaterialPage;

    test.beforeEach(async({ page }) => {
        materialPage = new MaterialPage(page);
        await materialPage.go();
        await expect(materialPage.page.getByText("Tài liệu học automation test")).toBeVisible();
    })

    test("Test 1: Registration Page", async({ page }) => {
        await materialPage.page.getByText("Bài học 1: Register Page (có đủ các element)").click();
        await expect(materialPage.page.getByText("User Registration")).toBeVisible();
    });

    test("Test 2: Product Page", async({ page }) => {
        await materialPage.page.getByText("Bài học 2: Product page").click();
        await expect(materialPage.page.getByText("Simple E-commerce")).toBeVisible();
    });
});
```

→ Vấn đề: mỗi **file test khác** lại phải copy-paste lại y chang đoạn `beforeEach` này (xem `04-without-fixture.spec.ts` cho "Test 3: Todo Page" — lặp lại toàn bộ setup).

### 3.4. Ví dụ khi ĐÃ có fixture (custom fixture)

Thay vì lặp lại `beforeEach`, ta viết **1 lần** logic setup/teardown vào 1 **custom fixture**, rồi các file test chỉ cần khai báo fixture đó trong tham số của `test(...)`.

**Bước 1 — Viết POM (`00-pom.ts`):**

```ts
import { Page } from "@playwright/test";

export class MaterialPage {
    page: Page;
    baseURL: string;

    constructor(page: Page){
        this.page = page;
        this.baseURL = "https://material.playwrightvn.com/";
    }

    async go(){
        await this.page.goto(this.baseURL);
    }
};
```

**Bước 2 — Viết custom fixture (`00-fixture.ts`):**

```ts
import { test as base, expect } from '@playwright/test';
import { MaterialPage } from './00-pom';

const test = base.extend<{ materialPage: MaterialPage }>({
    materialPage: async ({ page }, use) => {
        const materialPage = new MaterialPage(page);
        await materialPage.go();
        await expect(materialPage.page.getByText("Tài liệu học automation test")).toBeVisible();

        await use(materialPage);   // ⬅ giao materialPage cho test dùng

        console.log('Test end');   // ⬅ chạy SAU khi test xong (teardown)
    }
});

export { test };
```

Giải thích:

- `base.extend<{ materialPage: MaterialPage }>({...})` — mở rộng object `test` mặc định của Playwright, thêm 1 fixture tên `materialPage`, kiểu dữ liệu `MaterialPage`.
- Code **trước** `await use(materialPage)` = phần **setup** (chạy trước test, giống `beforeEach`).
- `await use(materialPage)` = "giao" đối tượng `materialPage` đã setup xong cho test case sử dụng.
- Code **sau** `await use(materialPage)` = phần **teardown** (chạy sau khi test case đã chạy xong, giống `afterEach`).
- File này **export ra `test` của riêng mình** (đã được mở rộng) để các file `.spec.ts` khác import và dùng thay cho `test` gốc của `@playwright/test`.

**Bước 3 — Dùng fixture trong test (`05-with-fixture.spec.ts`):**

```ts
import { expect } from '@playwright/test';
import { test } from './00-fixture';   // ⬅ import test đã custom, không phải từ '@playwright/test'

test("Test 1: Registration Page", async({ materialPage }) => {   // ⬅ destructure fixture materialPage
    await materialPage.page.getByText("Bài học 1: Register Page (có đủ các element)").click();
    await expect(materialPage.page.getByText("User Registration")).toBeVisible();
});

test("Test 2: Product Page", async({ materialPage }) => {
    await materialPage.page.getByText("Bài học 2: Product page").click();
    await expect(materialPage.page.getByText("Simple E-commerce")).toBeVisible();
});
```

Và ở **file test khác** (`06-with-fixture.spec.ts`), chỉ cần import lại đúng fixture, **không phải viết lại setup**:

```ts
import { expect } from '@playwright/test';
import { test } from './00-fixture';

test.describe("Material page - with POM2", () => {
    test("Test 3: Todo Page", async({ materialPage }) => {
        await materialPage.page.getByText("Bài học 3: Todo page").click();
        await expect(materialPage.page.getByText("To-Do List")).toBeVisible();
    });
});
```

**So sánh nhanh:**

| | Không có fixture | Có fixture |
|---|---|---|
| Setup (goto trang, assert trang đã load) | Copy-paste trong `beforeEach` ở **mỗi file** | Viết **1 lần** trong `00-fixture.ts`, import dùng lại mọi nơi |
| Khởi tạo POM | `new MaterialPage(page)` thủ công trong từng file | Tự động có sẵn qua `{ materialPage }` |
| Teardown (dọn dẹp sau test) | Phải tự viết `afterEach` nếu cần | Viết code sau `use()` — Playwright tự gọi |

### 3.5. Các built-in fixture có sẵn của Playwright

Playwright cung cấp sẵn các fixture sau (dùng được ngay, không cần khai báo gì thêm):

| Fixture | Type | Mô tả |
|---|---|---|
| `page` | `Page` | Tạo một page **riêng biệt** cho mỗi test. |
| `context` | `BrowserContext` | Tạo một context riêng biệt cho test. Fixture `page` phía trên cũng cùng context với context này. |
| `browser` | `Browser` | Browser được dùng **chung** giữa các test để tối ưu tài nguyên. |
| `browserName` | `string` | Tên browser đang chạy: `chromium`, `firefox` hoặc `webkit`. |
| `request` | `APIRequestContext` | Một `APIRequestContext` instance độc lập (dùng gọi API — đã học ở Lesson 11/12). |

Ví dụ minh hoạ các fixture này (`lesson-13/02-fixture.spec.ts`):

```ts
import { test } from '@playwright/test';

// Chỉ dùng page — Playwright tự tạo context + page cho test
test('My simple UI test2 - page only', async ({ page }) => {
    await page.goto('https://material.playwrightvn.com')
});

// Dùng context — tự tay tạo page từ context
test('My simple UI test2 with context', async ({ context }) => {
    const page = await context.newPage()
});

test('My simple UI test3 with context', async ({ context }) => {
    const page = await context.newPage()
    await page.goto('https://material.playwrightvn.com')
});

// 1 context có thể chứa NHIỀU page (giống nhiều tab trong cùng 1 trình duyệt, dùng chung storage/cookie)
test('My simple UI test4 with context', async ({ context }) => {
    const page1 = await context.newPage()
    await page1.goto('https://material.playwrightvn.com')

    const page2 = await context.newPage()
    await page2.goto('https://google.com')
});

// Dùng browser — tự tay tạo NHIỀU context độc lập (không chung cookie/storage)
test('My simple UI test with browser', async ({ browser }) => {
    const context1 = await browser.newContext();
    const page1 = await context1.newPage();
    await page1.goto('https://material.playwrightvn.com');

    const context2 = await browser.newContext();
    const page2 = await context2.newPage();
    await page2.goto('https://google.com');
});

// Dùng browserName — bỏ qua (skip) test theo trình duyệt
test('My simple UI test with Browser Name', async ({ browserName }) => {
    test.skip(browserName === 'firefox' || browserName === 'webkit')
    console.log(browserName);
});
```

**Ghi nhớ quan hệ giữa các fixture:**

- `browser` (1 trình duyệt) → có thể tạo ra **nhiều** `context` (mỗi context độc lập, không chung cookie/session).
- `context` (1 phiên làm việc) → có thể tạo ra **nhiều** `page` (giống nhiều tab, **chung** cookie/session trong cùng context).
- `page` (fixture mặc định hay dùng nhất) = Playwright đã tự động tạo sẵn 1 `context` + 1 `page` cho mỗi test, không cần tự tạo tay.

### 3.6. Chạy test bằng dòng lệnh (CLI) thay vì bấm nút Run

Ngoài việc bấm nút ▶ (Run) cạnh mỗi test trong VS Code, có thể **chạy test bằng dòng lệnh trong terminal** — cách này hay được dùng khi muốn chạy nhanh 1 test cụ thể bằng tên, hoặc khi chạy trên máy không có VS Code (CI/CD):

```bash
npx playwright test -g "My simple UI test2 - page only"
```

Kết quả trong terminal:

```
Running 1 test using 1 worker
  1 passed (4.2s)

To open last HTML report run:

  npx playwright show-report
```

Giải thích:

- `npx playwright test` — lệnh gốc để chạy toàn bộ test.
- `-g "<tên test>"` (viết tắt của `--grep`) — chỉ chạy (các) test có **tên khớp** với chuỗi/regex truyền vào, thay vì chạy hết. Ở đây tên phải trùng khớp với tên khai báo trong `test('...')`.
- Nếu bỏ `-g`, lệnh `npx playwright test` sẽ chạy **tất cả** test tìm thấy trong project.
- Sau khi chạy xong, dùng `npx playwright show-report` để mở HTML report xem chi tiết (bao gồm cả video/trace nếu có bật, xem [mục 5.4](#54-video-recording-trong-thực-tế)).

> Bấm nút Run trong VS Code thực chất cũng chỉ là chạy giúp lệnh `npx playwright test` này ở phía sau — hiểu cách chạy bằng CLI giúp linh hoạt hơn khi cần chạy đúng 1 test theo tên, chạy trên terminal khác, hoặc tích hợp vào CI/CD.

### 3.7. Custom 1 fixture (tự tạo fixture riêng)

- Dùng `test.extend()` để **mở rộng** test object mặc định.
- Gồm 2 bước: **khai báo** kiểu dữ liệu của fixture, và **implement** (viết logic setup/teardown bên trong).

Cú pháp tổng quát (minh hoạ bằng fixture `page2` tuỳ chỉnh):

```ts
import { test as base } from '@playwright/test'
// ...
const test = base.extend<{ page2: Page2 }>({
  page2: async ({ }, use) => {
    const page2 = new Page2();
    page2.sayMyName();

    await use(page2);

    console.log("after page2");
  }
})

export { test };
```

- `test as base` — đổi tên `test` gốc thành `base` để tránh trùng tên với biến `test` mới (đã mở rộng) khai báo bên dưới.
- `{ }` (tham số đầu của callback) — nơi có thể destructure các fixture khác (ví dụ `page`) nếu fixture mới cần dùng tới chúng.
- `use` — hàm để "giao" giá trị fixture cho test; code trước `use()` là setup, code sau `use()` là teardown (tương tự phần fixture `materialPage` ở trên).

### 3.8. Nâng cao hơn về fixture

Playwright còn hỗ trợ nhiều tính năng nâng cao với fixture (tham khảo thêm khi cần):

- **Overriding fixtures** — ghi đè lại 1 fixture có sẵn (kể cả built-in) bằng logic riêng.
- **Worker-scoped fixtures** — fixture chỉ khởi tạo **1 lần cho cả worker** (nhiều test dùng chung), thay vì tạo lại mỗi test.
- **Automatic fixtures** — fixture tự động chạy dù test có khai báo dùng tới nó hay không.
- **Fixture timeout** — set thời gian timeout riêng cho từng fixture.
- **Fixtures-options** — truyền tham số/tuỳ chọn vào fixture.
- **Execution order** — thứ tự chạy khi có nhiều fixture phụ thuộc lẫn nhau.

📚 Tài liệu chính thức: https://playwright.dev/docs/test-fixtures

### 3.9. Bài thực hành

> Viết 1 fixture tự động tạo 1 POM và truy cập Material Page.

Đây chính là những gì `00-pom.ts` + `00-fixture.ts` ở trên đã làm: fixture `materialPage` tự tạo instance `MaterialPage`, tự `goto` vào trang, và tự assert trang đã load xong trước khi giao (`use`) cho test case sử dụng.

---

## 4. Test Generator (Codegen)

### 4.1. Test generator là gì?

- Là thao tác **"click sinh ra code"**: mình click/thao tác trên trình duyệt, Playwright **tự động sinh ra code test** tương ứng — không cần gõ tay từng dòng.

### 4.2. Test generator giải quyết bài toán gì?

- Cần **code nhanh** — thay vì tự viết locator + action từng dòng.
- Muốn **"lười"** — thay vì phải tự tay thao tác lại từ đầu để viết code, chỉ cần thao tác 1 lần trên trình duyệt là có ngay code.

### 4.3. Sử dụng test generator trong Playwright

Trong VS Code, extension **Playwright Test** có sẵn các nút ở mục **TESTING** (sidebar):

- **Record new**: tạo **file mới** và bắt đầu record thao tác thành code.
- **Record at cursor**: record và **sinh code ngay tại vị trí con trỏ chuột** đang đứng trong file hiện tại (dùng để chèn thêm code vào giữa 1 test đã có sẵn).
- **Assertion**: cho phép chọn 1 phần tử trên trang để **generate ra dòng so sánh (assertion)** tương ứng (ví dụ `toBeVisible()`, `toHaveText()`...).

> Ngoài ra có thể dùng dòng lệnh: `npx playwright codegen <url>` để mở cửa sổ ghi thao tác tương tự mà không cần qua VS Code.

### 4.4. Ví dụ code được sinh ra sau khi thực hiện gen code (`lesson-13/test-1.spec.ts`)

Sau khi bấm **Record new**, thao tác click/nhập liệu trên trình duyệt tại trang `https://material.playwrightvn.com/`, Playwright tự sinh ra file test sau — **không cần gõ tay dòng nào**:

```ts
import { test, expect } from '@playwright/test';

test('test gencode', async ({ page }) => {
  await page.goto('https://material.playwrightvn.com/');
  await page.getByRole('link', { name: 'Bài học 1: Register Page (c' }).click();
  await expect(page.getByRole('heading', { name: 'User Registration' })).toBeVisible();
});

test('test gencode - bai hoc 2', async ({ page }) => {
  await page.goto('https://material.playwrightvn.com/');
  await page.waitForTimeout(2_000);

  await page.getByRole('link', { name: 'Bài học 2: Product page' }).click();
  await page.getByText('Product 2').click();
  await page.waitForTimeout(2_000);

  await expect(page.locator('body')).toContainText('Product 2');
  await page.getByRole('link', { name: 'Trở về trang chủ' }).click();
  await page.waitForTimeout(2_000);

  await page.getByRole('link', { name: 'Bài học 1: Register Page (c' }).click();
  await page.getByRole('textbox', { name: 'Username:' }).click();
  await page.getByRole('textbox', { name: 'Username:' }).fill('thanhtuyen');
  await page.waitForTimeout(2_000);

  await expect(page.getByRole('textbox', { name: 'Username:' })).toHaveValue('thanhtuyen');
});
```

**Nhận xét về code do generator sinh ra:**

- **`test('test gencode', ...)`** — mỗi lần bấm **Record new** cho 1 luồng thao tác mới, Playwright tạo 1 `test(...)` mới với tên mặc định `test` (có thể sửa lại tên cho có ý nghĩa hơn sau khi gen xong).
- Mỗi cú **click** trên trình duyệt → sinh ra 1 dòng `page.getByRole(...).click()` hoặc `page.getByText(...).click()` — generator **tự chọn locator tốt nhất** dựa trên role/text của phần tử (đúng theo best practice locator đã học).
- Mỗi lần **gõ chữ vào ô input** → sinh ra cặp dòng `.click()` (bấm vào ô trước) rồi `.fill('...')` (điền giá trị).
- **`page.waitForTimeout(2_000)`** — do người thao tác **dừng lại 2 giây** giữa các bước khi record (để xem kết quả); đây **không phải best practice**, khi dọn code lại (refactor) nên **xoá bỏ** các dòng `waitForTimeout` này và thay bằng `expect(...).toBeVisible()` hoặc dựa vào auto-waiting sẵn có của Playwright.
- **Assertion** (`await expect(...).toBeVisible()`, `toContainText(...)`, `toHaveValue(...)`) — được sinh ra khi bấm nút **Assertion** rồi chọn phần tử cần so sánh; generator tự đề xuất loại assertion phù hợp (element có hiển thị, chứa text gì, input có giá trị gì...).
- **Tên locator dài/bị cắt** (`name: 'Bài học 1: Register Page (c'`) — do tên đầy đủ của phần tử quá dài, generator tự cắt bớt; nên kiểm tra và sửa lại cho khớp chính xác hoặc dùng locator ngắn gọn, ổn định hơn nếu cần.

> Code sinh ra từ generator là điểm khởi đầu tốt để **code nhanh**, nhưng vẫn nên xem lại và dọn dẹp (xoá `waitForTimeout`, đặt lại tên test, tách POM nếu cần...) trước khi đưa vào bộ test chính thức.

---

## 5. Video Recording

### 5.1. Video recording là gì?

- Là việc **"quay video"** lại **toàn bộ quá trình chạy test**.
- Đây là tính năng **có sẵn (in-built)** của Playwright — **không cần cài thêm** thư viện bên thứ 3 nào.

### 5.2. Video recording giải quyết bài toán gì?

- Giúp **debug test dễ dàng hơn** — xem lại chính xác trình duyệt đã làm gì, dừng ở đâu khi test fail.
- Giúp **generate ra evidence (bằng chứng)** nhanh chóng — ví dụ để đính kèm báo cáo lỗi (bug report).

### 5.3. Cách bật video recording trong Playwright

Cấu hình trong file **`playwright.config.ts`**, ở mục `use`:

```ts
export default defineConfig({
  use: {
    trace: 'on',
    headless: false,
    video: 'on',
    permissions: ['notifications', 'geolocation'],
  },
  // ...
});
```

Các **mode** của `video`:

| Mode | Ý nghĩa |
|---|---|
| `off` | Tắt, không quay video. |
| `on` | Bật, quay video cho **tất cả** các test. |
| `retain-on-failure` | Quay video hết, nhưng **chỉ giữ lại** video của test **fail**. |
| `on-first-retry` | Chỉ quay video cho những test **fail và đang chạy retry lần đầu**. |

> Lưu ý thêm 2 cấu hình hay đi kèm khi debug:
> - `headless: false` — mở trình duyệt lên để xem trực tiếp khi chạy test (thay vì chạy ẩn/ngầm).
> - `trace: 'on'` — bật ghi lại trace (dùng với Trace Viewer để xem lại từng bước chi tiết hơn cả video).

### 5.4. Video recording trong thực tế

Sau khi chạy test với `video: 'on'`, Playwright lưu file video vào thư mục kết quả test, ví dụ:

```
test-results/
└── test-1-test--bai-hoc-2-chromium/
    ├── trace.zip     ← file trace, mở bằng Trace Viewer
    └── video.webm    ← file video ghi lại quá trình chạy test
```

- Có thể mở trực tiếp file `video.webm` (định dạng WebM) bằng trình phát video, hoặc xem trong **HTML report** (`npx playwright show-report`) — report sẽ tự động nhúng video vào từng test case (đặc biệt hữu ích với các test fail).
- Test được lặp lại nhiều lần trong buổi học để so sánh: chạy 1 test cụ thể bằng `-g` (chỉ định tên test) rồi mở report để xem file `video.webm` tương ứng.

---

## 6. Tổng kết Lesson 13

| Chủ đề | Ý chính cần nhớ |
|---|---|
| **Object destructuring** | `const { a, b } = object;` — lấy nhiều thuộc tính ra biến chỉ trong 1 dòng, tên biến phải trùng tên thuộc tính. Đây là nền tảng để hiểu cú pháp `async ({ page }) => {}` của Playwright. |
| **Fixture** | Cơ chế setup (trước) → giao đối tượng cho test dùng (`use()`) → teardown (sau). Built-in: `page`, `context`, `browser`, `browserName`, `request`. Tự tạo bằng `test.extend()`, export `test` riêng để các file khác import dùng chung — tránh lặp code `beforeEach`. |
| **Test generator** | `Record new` / `Record at cursor` / `Assertion` trong VS Code (hoặc `npx playwright codegen`) — click là ra code, tiết kiệm thời gian viết test cơ bản. |
| **Video recording** | Cấu hình `video` trong `playwright.config.ts` (`off` / `on` / `retain-on-failure` / `on-first-retry`) — tính năng có sẵn, không cần thư viện ngoài, giúp debug và làm bằng chứng lỗi nhanh. |

**Luồng phát triển 1 bộ test "chuẩn" sau bài học này:**

```
1. Viết POM (class chứa page + các hành động)              → 00-pom.ts
2. Viết custom fixture bọc POM đó, tự goto + tự assert       → 00-fixture.ts
3. Test case chỉ cần destructure fixture ra dùng             → *-with-fixture.spec.ts
4. Bật video + trace trong playwright.config.ts để debug     → khi có test fail
5. Dùng test generator (codegen) để sinh nhanh code mới       → khi viết case mới
```
