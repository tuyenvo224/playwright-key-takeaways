# Lesson 12 — POM cho API, các biến thể khác của POM, Async/Await

> Nguồn: PlaywrightVN — Học automation test từ chưa biết gì

---

## Mục lục

1. [Nội dung buổi học](#1-nội-dung-buổi-học)
2. [POM cho API là gì?](#2-pom-cho-api-là-gì)
3. [Code thực hành: POM cho Todo API & Product API](#3-code-thực-hành-pom-cho-todo-api--product-api)
4. [Bài thực hành](#4-bài-thực-hành)
5. [POM API nâng cao](#5-pom-api-nâng-cao)
6. [Một số "biến thể" (styles) khác của POM](#6-một-số-biến-thể-styles-khác-của-pom)
7. [Async, await](#7-async-await)
8. [Tổng kết Lesson 12](#8-tổng-kết-lesson-12)

---

## 1. Nội dung buổi học

Buổi học xoay quanh 3 chủ đề lớn:

1. **POM cho API** — áp dụng tư duy Page Object Model (đã học ở UI test) vào việc gọi API.
2. **Một số "biến thể" của POM** — các cách tổ chức POM khác ngoài cách viết thông thường (kế thừa, POM manager, return POM từ POM khác).
3. **Async, await** — hiểu đúng bản chất để không bị lỗi "quên await" khi viết test.

---

## 2. POM cho API là gì?

### 2.1. Vì sao cần tổ chức POM cho API test

Giống hệt lý do dùng POM cho UI test: **tách phần "gọi API" (thuộc tính + phương thức) ra khỏi phần "test case"**, để code dễ đọc, dễ bảo trì, tái sử dụng được ở nhiều test khác nhau.

```
Non POM Structure                    POM Based Structure
┌─────────────┐                      ┌─────────────┐        ┌─────────────┐
│   ClassA    │                      │   ClassA    │        │   ClassB    │
│ ┌─────────┐ │                      │ ┌─────────┐ │        │ Test Methods│
│ │WebElement│ │                     │ │WebElement│◄───────►│             │
│ │+ Method │ │                      │ │+ Method │ │        └─────────────┘
│ └────┬────┘ │                      │ └─────────┘ │
│      ▼      │                      └─────────────┘
│ Test Methods│           ❌ trộn lẫn         ✅ tách riêng: nơi "biết cách gọi API"
└─────────────┘                                  và nơi "viết test case"
```

### 2.2. Concept — tương tự POM UI

| | POM UI | POM API |
|---|--------|---------|
| **Tên class** | `LoginPage`, `HomePage`... | `TodoApiPage`, `ProductApiPage`... |
| **Thuộc tính** | `page: Page` | `request: APIRequestContext` |
| **Phương thức** | các hàm thao tác UI (`click()`, `fill()`...) | các hàm **gọi API** (`getAll()`, `getTodo()`...) |

**Tư duy khi thiết kế:**
- **Thuộc tính**: những gì *cần thiết cho việc gọi API* — `request` fixture, `baseURL`.
- **Phương thức**: các *hàm gọi API*, mỗi hàm gọi 1 endpoint, trả về response (JSON) đã parse sẵn.

### 2.3. Cấu trúc cơ bản

```ts
import { APIRequestContext } from '@playwright/test';

export class TodoApiPage {
    request: APIRequestContext;

    constructor(request: APIRequestContext) {
        this.request = request;
    }
}
```

- `request: APIRequestContext` — kiểu dữ liệu của fixture `request` mà Playwright cung cấp sẵn để gọi API (đã học ở [Lesson 11](lesson11-tong-hop.md)).
- Constructor nhận `request` truyền vào từ test, lưu lại để các phương thức bên trong dùng chung.

---

## 3. Code thực hành: POM cho Todo API & Product API

Thư mục `lesson-12/` chứa 2 bộ POM cho 2 API khác nhau, theo đúng cấu trúc: 1 file `*.page.ts` (định nghĩa POM) đi kèm 1 file `*.spec.ts` (test case dùng POM đó).

### 3.1. Todo API — `01-todo.api.page.ts`

```ts
import { APIRequestContext, expect } from "@playwright/test";

export class TodoApiPage {
    request: APIRequestContext;
    baseUrl: string;

    constructor(request: APIRequestContext) {
        this.request = request;
        this.baseUrl = 'https://material.playwrightvn.com/api/todo-app/v1';
    }

    // get all todo
    async getAll() {
        const response = await this.request.get(`${this.baseUrl}/todos.php`);
        expect(response.status()).toBe(200);

        const responseJson = response.json();
        return responseJson;
    }

    // get single todo
    async getTodo(id: number) {
        const response = await this.request.get(`${this.baseUrl}/todo.php?id=${id}`);
        expect(response.status()).toBe(200);

        const reponseJson = response.json();
        return reponseJson;
    }

    // create todo

    // update todo

    // update partial todo

    // delete todo
}
```

**Điểm quan trọng:**
- `baseUrl` được lưu sẵn trong constructor → mỗi phương thức chỉ cần nối thêm phần đường dẫn (`${this.baseUrl}/todos.php`) thay vì gõ lại URL đầy đủ.
- Mỗi phương thức (`getAll`, `getTodo`) tự **verify status code** (`expect(response.status()).toBe(200)`) rồi mới trả kết quả JSON về — test case bên ngoài chỉ cần verify **nội dung dữ liệu**, không cần verify lại status.
- ⚠️ Các comment `// create todo`, `// update todo`... là các phương thức **dự kiến làm tiếp** (chưa viết) — mẫu hình thường thấy khi vừa học vừa mở rộng dần POM.
- `getTodo(id: number)` nhận tham số `id`, minh họa cách 1 phương thức POM có thể **nhận input** để tái sử dụng cho nhiều todo khác nhau, thay vì hard-code cứng 1 giá trị.

### 3.2. Test case dùng Todo POM — `02-todo-api-with-pom.spec.ts`

```ts
import { expect, test } from '@playwright/test';
import { TodoApiPage } from './01-todo.api.page';

test('Get all todo', async ({ request }) => {
    const todoApiPage = new TodoApiPage(request);
    const responseJson = await todoApiPage.getAll();

    expect(responseJson.todos.length).toEqual(152);
});

test('Get todo with id =226', async ({ request }) => {
    const todoApiPage = new TodoApiPage(request);
    const responseJson = await todoApiPage.getTodo(226);

    console.log(responseJson);
    expect(responseJson.todo.title).toContain("Tuyen");
    expect(responseJson.todo.priority).toEqual("high");
});
```

- Mỗi test **tự khởi tạo** `new TodoApiPage(request)` — dùng fixture `request` mà Playwright inject sẵn vào callback của `test(...)`.
- Test case giờ đây **rất ngắn gọn**: chỉ còn 2 việc — gọi phương thức POM, rồi `expect` vào dữ liệu trả về. Toàn bộ chi tiết "gọi API thế nào, url ra sao" đã nằm gọn trong `TodoApiPage`.

### 3.3. Product Catalog API — `03-product.api.page.ts`

```ts
// Create POM
import { APIRequestContext, expect } from "@playwright/test";

export class ProductApiPage {
    // Thêm thuộc tính request
    request: APIRequestContext;
    baseUrl: string;

    constructor(request: APIRequestContext) {
        this.request = request;
        this.baseUrl = "https://material.playwrightvn.com/api/product-catalog/v1";
    }

    // Add method getProducts
    async getProducts() {
        // Call API
        const response = await this.request.get(`${this.baseUrl}/products.php`);
        // Verify status code
        expect(response.status()).toBe(200);

        const reponseJson = await response.json();
        return reponseJson;
    }
}
```

### 3.4. Test case dùng Product POM — `04-product-api-with-pom.spec.ts`

```ts
import { test, expect } from "@playwright/test";
import { ProductApiPage } from "./03-product.api.page";

test('Get products', async ({ request }) => {
    const productAPIPage = new ProductApiPage(request);
    const reponseJson = await productAPIPage.getProducts();

    console.log(reponseJson.data.products.length);
    expect(reponseJson.data.products.length).toEqual(10);
});
```

> **So sánh 2 ví dụ:** cấu trúc `TodoApiPage` và `ProductApiPage` giống hệt nhau về khung sườn (`request`, `baseUrl`, constructor, phương thức gọi API + verify status + trả JSON) — đây chính là **pattern chuẩn** để áp dụng POM cho bất kỳ API nào.

---

## 4. Bài thực hành

1. **Todo API** — Swagger: `https://material.playwrightvn.com/api/todo-app/swagger.html`
   - Tạo POM cho Todo API (như mục 3.1–3.2).

2. **Product Catalog API** — Swagger: `https://material.playwrightvn.com/api/product-catalog/swagger.html`
   - Tạo POM cho Product Catalog API.
   - Có 1 phương thức: `getProducts`.
   - Viết test case: assert có **10 product** trả về.

---

## 5. POM API nâng cao

Sau khi đã viết được POM cơ bản, có 2 cải tiến hay dùng để POM "chuyên nghiệp" hơn:

1. **Lưu `baseURL`** trong constructor (đã áp dụng ở mục 3) — để khi gọi API ở các phương thức, chỉ cần viết phần đường dẫn ngắn gọn thay vì URL đầy đủ mỗi lần.
2. **Thêm thuộc tính `token`** — để lưu lại token sau khi login, dùng cho các API cần xác thực (`Authorization: Bearer <token>`, xem lại luồng authentication ở [Lesson 11, mục 10](lesson11-tong-hop.md#10-luồng-authentication-đăng-nhập-lấy-token)).

```ts
export class UserApiPage {
    request: APIRequestContext;
    baseUrl: string;
    token: string;   // <-- thêm thuộc tính lưu token

    constructor(request: APIRequestContext) {
        this.request = request;
        this.baseUrl = 'https://material.playwrightvn.com/api/user-management/v1';
        this.token = '';
    }

    async login(email: string, password: string) {
        const response = await this.request.post(`${this.baseUrl}/login.php`, {
            data: { email, password }
        });
        const json = await response.json();
        this.token = json.data.token;   // lưu token lại để dùng cho các lần gọi sau
    }

    async getUsers() {
        const response = await this.request.get(`${this.baseUrl}/users.php`, {
            headers: { authorization: `Bearer ${this.token}` }
        });
        return response.json();
    }
}
```

> Nhờ lưu `token` ngay trong object POM, các phương thức gọi API cần xác thực phía sau **không cần truyền token qua từng tham số** — cứ gọi `this.token` là có sẵn, miễn là đã gọi `login()` trước đó.

---

## 6. Một số "biến thể" (styles) khác của POM

Ngoài cách viết POM "cơ bản" (mỗi page 1 class độc lập, test tự `new` từng page cần dùng), còn có các cách tổ chức khác — style nào cũng đúng, chọn tùy quy mô dự án và sở thích team.

```
Non POM Structure                    POM Based Structure
┌─────────────┐                      ┌─────────────┐        ┌─────────────┐
│   ClassA    │                      │   ClassA    │        │   ClassB    │
│ ┌─────────┐ │                      │ ┌─────────┐ │        │ Test Methods│
│ │WebElement│ │                     │ │WebElement│◄───────►│             │
│ │+ Method │ │                      │ │+ Method │ │        └─────────────┘
│ └────┬────┘ │                      │ └─────────┘ │
│      ▼      │                      └─────────────┘
│ Test Methods│
└─────────────┘
```

### 6.1. Style 1 — Kế thừa (Inheritance)

**Ý tưởng:** page phía sau **"kế thừa" (extends)** page phía trước — vì page sau thường cần dùng lại các phương thức của page trước đó (ví dụ: `ProductPage` cần dùng lại các hàm điều hướng chung của `HomePage`).

```
[ HomePage ] ──────► [ Product page ]

productPage extends HomePage
```

```ts
// Không kế thừa — phải tạo cả 2 object riêng biệt
const homePage = new HomePage();
const productPage = new ProductPage();

homePage.navigateTo("product");
productPage.addToCart("product 1");
```

```ts
// Có kế thừa — ProductPage extends HomePage
// nên productPage dùng được luôn cả navigateTo() lẫn addToCart()
const productPage = new ProductPage();

productPage.navigateTo("product");
productPage.addToCart("product 1");
```

- **Ưu điểm:** chỉ cần khởi tạo 1 object (`productPage`) là dùng được cả phương thức của chính nó lẫn phương thức "thừa hưởng" từ page cha.
- **Nhược điểm:** nếu chuỗi page càng dài (A → B → C → D...), chuỗi kế thừa càng sâu, dễ rối và khó kiểm soát page nào đang thực sự "sở hữu" phương thức nào.

### 6.2. Style 2 — POM Manager

**Ý tưởng:** dùng **1 class trung tâm** (`PomManager`) để quản lý việc khởi tạo và cấp phát tất cả các page object, thay vì mỗi nơi tự `new` page riêng lẻ.

```
              ┌──► Product page
PomManager ───┤
              └──► HomePage
```

```ts
import { Page } from "@playwright/test";
import { LoginPage } from "./00-pom-login";
import { DashboardPage } from "./00-pom-dashboard";

export class PomManager {
    page: Page;

    constructor(page: Page) {
        this.page = page;
    }

    getLoginPage() {
        return new LoginPage(this.page);
    }

    getDashboardPage() {
        return new DashboardPage(this.page);
    }
}
```

**Đặc điểm của style này:**
- **Quản lý nhiều page object** từ một nơi duy nhất (`PomManager`).
- Các page object có thể được **tạo và truy cập** thông qua các phương thức `get...Page()` của manager.
- Các page object **độc lập với nhau** — không page nào phụ thuộc/kế thừa page nào.
- Page **chỉ được tạo khi thực sự cần thiết** (mỗi lần gọi `get...Page()` mới tạo instance mới), tránh khởi tạo thừa.

> **Lưu ý:** cách gọi qua tên chuỗi như `pomManager.get("home_page")` (dùng key dạng string để tra ra object — theo kiểu **Singleton** / **Factory pattern**) là style phổ biến bên **Java**, nhưng **không phù hợp với TypeScript** trong Playwright vì TS có kiểu dữ liệu tĩnh (static typing) — muốn biết chắc kiểu trả về là `HomePage` hay `ProductPage` ngay lúc viết code (để có gợi ý/autocomplete), nên cách làm khuyến nghị ở TypeScript là **từng phương thức riêng cho từng page** (`getLoginPage()`, `getDashboardPage()`...) như code mẫu ở trên, thay vì 1 phương thức `get(key: string)` dùng chung.

### 6.3. Style 3 — Return POM từ POM khác (Chaining)

**Ý tưởng:** một phương thức của Page Object này, sau khi thực hiện xong hành động điều hướng, sẽ **trả về (`return`) chính Page Object tiếp theo** — cho phép **nối chuỗi (chaining)** các bước lại với nhau, rất hợp với các luồng nhiều bước như:

```
login > add to cart > checkout > confirm
```

```
[ HomePage ] ──────► [ Product page ]
       │                     ▲
       └── navigate("product_page") ──┘

const homePage = new HomePage();
const productPage = homePage.navigate("product_page");
productPage.addToCart();
```

**Code ví dụ đầy đủ — `tests/lesson-12/other-styles/`:**

`01-home.page.ts`:
```ts
import { Page } from '@playwright/test';
import { ProductPage } from './02-product.page';

export class HomePage {
    page: Page;

    constructor(page: Page) {
        this.page = page;
    }

    async navigateToProductPage() {
        await this.page.locator("//div[text=product....").click();
        return new ProductPage(this.page);   // <-- trả về POM khác, không phải trả về chính nó
    }
}
```

`02-product.page.ts`:
```ts
import { Page } from '@playwright/test';

export class ProductPage {
    page: Page;

    constructor(page: Page) {
        this.page = page;
    }

    async addToCart() {
        await this.page.locator("//div[text=product_1....").click();
    }
}
```

`demo-3.spec.ts`:
```ts
import { test } from '@playwright/test';
import { HomePage } from './01-home.page';

test('Add product to cart', async ({ page }) => {   // Fixture
    const homePage = new HomePage(page);
    const productPage = await homePage.navigateToProductPage();

    await productPage.addToCart();
});
```

**Giải thích luồng chạy:**
1. Test tạo `homePage` từ fixture `page` có sẵn của Playwright.
2. Gọi `homePage.navigateToProductPage()` — phương thức này vừa **click điều hướng sang trang Product**, vừa **trả về (`return`) một `ProductPage` mới**, đã gắn sẵn cùng 1 `page` (trình duyệt) đang dùng.
3. Test nhận `productPage` đó và gọi tiếp `productPage.addToCart()` — không cần tự `new ProductPage(page)` thủ công.
4. ⚠️ Vì `navigateToProductPage()` có `await this.page.locator(...).click()` bên trong (một web-first action, trả về Promise) nên bản thân hàm `navigateToProductPage()` cũng là `async` — do đó ở test **phải `await`** khi gọi nó: `const productPage = await homePage.navigateToProductPage();`.

- **Ưu điểm:** test case đọc như một **kịch bản (script)** tự nhiên theo đúng luồng người dùng thao tác (điều hướng → thao tác trên trang mới → điều hướng tiếp...), không cần lo việc tự tạo đúng object nào ở mỗi bước.
- **So với Style 1 (kế thừa):** Style 3 không gộp phương thức của 2 page vào chung 1 class — mỗi page vẫn giữ đúng phương thức của riêng mình, chỉ "chuyền tay nhau" qua giá trị trả về.

---

## 7. Async, await

### 7.1. Async, await là gì?

**Async/Await** là **cách viết code** JavaScript/TypeScript để xử lý các tác vụ **bất đồng bộ (asynchronous)** một cách **dễ đọc hơn** (thay vì phải viết `.then().catch()` nối chuỗi dài dòng).

| Từ khóa | Vai trò |
|---------|---------|
| **`async`** | Đặt trước `function` để biến nó thành **async function** — một hàm luôn **trả về Promise** (dù bên trong có `return` giá trị thường hay không). |
| **`await`** | Đặt trước một **Promise** để **"chờ"** nó hoàn thành (resolve) rồi mới chạy tiếp dòng code phía sau. |

### 7.2. Tại sao cần async, await?

- **Cơ chế hoạt động của JavaScript**: chạy theo mô hình **Event Loop, single thread** (1 luồng xử lý duy nhất) — nếu không có cơ chế bất đồng bộ, mọi tác vụ mất thời gian sẽ **chặn (block)** toàn bộ chương trình cho tới khi xong.
- Trong automation test, rất nhiều **thao tác network/IO** vốn dĩ **mất thời gian**, không thể có kết quả ngay lập tức:

| Thao tác | Vì sao cần chờ |
|----------|----------------|
| **Mở trang web** (`page.goto()`) | Mất vài giây để trình duyệt tải trang |
| **Tìm element** (`page.locator()...`) | Cần chờ element thực sự xuất hiện trên DOM |
| **Click button** | Phải chờ animation/hiệu ứng của UI hoàn tất |
| **Gọi API** (`request.get()`...) | Phải chờ server phản hồi (response) |

→ Nếu không `await` các thao tác này, code sẽ chạy tiếp **ngay lập tức** mà không đợi kết quả thật sự trả về, dẫn tới lỗi (test "tưởng" element đã có nhưng thực ra chưa kịp load, hoặc đọc response API khi request còn chưa xong).

### 7.3. Dùng async/await đúng cách

**✅ LUÔN dùng `await` với:**

```ts
await page.goto();
await page.click();
await page.fill();
await expect(locator).toBeVisible();   // expect() assertions trên Locator
// và bất kỳ method nào trả về Promise
```

**❌ KHÔNG cần `await` với:**

```ts
page.locator("...")          // chỉ tạo ra locator (mô tả), CHƯA thực sự tương tác với DOM
let count = 5;                // biến thông thường
expect(response.status()).toBe(200);   // synchronous operation — giá trị có sẵn ngay, không trả Promise
```

> **Ghi nhớ nhanh:** hễ một phương thức **thực sự thao tác** (điều hướng, click, gõ, gọi mạng) hoặc **trả về Promise** thì phải `await`. Còn những gì chỉ **mô tả/tạo tham chiếu** (như `page.locator(...)`) hoặc **đọc giá trị có sẵn ngay lập tức** (như `response.status()`) thì **không** cần `await`. Quên `await` ở đúng chỗ cần là nguyên nhân phổ biến nhất gây ra test "pass giả" hoặc lỗi khó hiểu.

---

## 8. Tổng kết Lesson 12

| Chủ đề | Điểm chính |
|--------|-----------|
| **POM cho API** | Áp dụng đúng tư duy POM UI vào API: class có thuộc tính `request: APIRequestContext` (+ `baseUrl`, `token`), phương thức là các hàm gọi API — verify status code ngay trong phương thức, trả JSON đã parse về cho test dùng |
| **POM API nâng cao** | Lưu sẵn `baseUrl` trong constructor để gọi API ngắn gọn; thêm thuộc tính `token` để tái sử dụng cho các API cần xác thực (`Authorization: Bearer <token>`) |
| **Style kế thừa** | `ProductPage extends HomePage` — page sau dùng lại được phương thức của page trước, chỉ cần tạo 1 object; nhưng chuỗi kế thừa dài dễ rối |
| **Style POM manager** | 1 class trung tâm (`PomManager`) cấp phát các page object qua các phương thức `get...Page()` riêng biệt — quản lý tập trung, các page độc lập nhau, chỉ tạo khi cần; cách dùng `get(key: string)` kiểu Java (Singleton/Factory) không hợp với TypeScript static typing |
| **Style return POM khác** | Phương thức của page này `return` về page tiếp theo (`return new ProductPage(this.page)`) — test đọc tự nhiên như kịch bản nối chuỗi các bước (`homePage.navigateToProductPage()` → `productPage.addToCart()`) |
| **Async, await là gì** | `async` đặt trước function để hàm luôn trả về Promise; `await` đặt trước Promise để chờ nó hoàn thành trước khi chạy tiếp |
| **Vì sao cần** | JS chạy Event Loop single-thread; các thao tác network/IO trong automation (mở trang, tìm element, click, gọi API) đều cần thời gian chờ thực sự |
| **Dùng đúng cách** | Luôn `await` mọi thứ trả về Promise (`page.goto/click/fill`, `expect()` trên Locator, method bất kỳ trả Promise); không cần `await` với `page.locator()` (chỉ tạo tham chiếu), biến thường, hay thao tác đồng bộ (`response.status()`) |
