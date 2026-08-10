# Kiến thức tổng hợp — Playwright Automation Test Framework

Tài liệu này tổng hợp toàn bộ kiến thức, quyết định thiết kế, và các lỗi thực tế đã gặp trong quá trình xây dựng framework test cho [pw-practice-dev.playwrightvn.com/wp-admin](https://pw-practice-dev.playwrightvn.com/wp-admin). Mỗi phần đều có ví dụ code thật lấy từ project, kèm giải thích "vì sao" — không chỉ "làm gì".

---

## 1. Cấu trúc tổng thể của một Automation Test Framework

Một framework test tự động (dùng Playwright + TypeScript) thường được chia thành 4 lớp, mỗi lớp có **đúng một trách nhiệm** (separation of concerns):

```
project/
├── pom/          → Page Object Model: MÔ TẢ trang web (locator + action)
├── fixtures/      → Chuẩn bị SẴN trạng thái cho test (VD: đã login)
├── utils/         → Hàm thuần, KHÔNG phụ thuộc page/browser (sinh dữ liệu random...)
├── tests/         → Kịch bản test thực tế, GỌI vào 3 lớp trên
├── .env           → Thông tin nhạy cảm (URL, tài khoản, mật khẩu) — KHÔNG commit
├── .gitignore      → Loại trừ .env, node_modules... khỏi git
├── playwright.config.ts → Cấu hình chạy test (browser, timeout, report...)
└── tsconfig.json  → Cấu hình TypeScript cho toàn project
```

**Chiều phụ thuộc chỉ đi một hướng:**

```
tests/  ──gọi──>  fixtures/  ──gọi──>  pom/
tests/  ──gọi──>  utils/
fixtures/ ──gọi──> pom/
pom/    ──không phụ thuộc gì khác ngoài Playwright──
```

Nguyên tắc quan trọng: **`pom/` và `utils/` không được biết gì về `tests/`**. Nếu sửa 1 file trong `tests/`, không bao giờ cần sửa `pom/`/`utils/` theo. Ngược lại, sửa `pom/`/`utils/` thì nhiều `tests/` có thể bị ảnh hưởng — đây là lý do 2 lớp này cần được thiết kế cẩn thận và ổn định.

Trong project, cấu trúc con của `pom/` và `tests/` được thiết kế **mirror 1-1** theo module của website (`dashboard/login`, `dashboard/users`, `storefront/...`), giúp dễ tìm: muốn sửa test nào, biết ngay page object tương ứng nằm ở đâu.

---

## 2. Page Object Model (POM)

### 2.1 Locator: kiểu dữ liệu và thời điểm khởi tạo

**Lỗi thường gặp ban đầu:**
```ts
// SAI
userName: string = getByRole('textbox', { name: 'Username or Email Address' });
```
Có 2 lỗi cùng lúc:
- `getByRole(...)` trả về kiểu `Locator`, không phải `string`.
- Đây là *property initializer* — chạy **trước** phần thân constructor. Tại thời điểm này `this.page` chưa được gán, nên gọi `this.page.getByRole(...)` ở đây sẽ lỗi (hoặc nếu gọi `getByRole` không có `this.page.` phía trước thì đó là một hàm không tồn tại).

**Cách đúng — khai kiểu `Locator`, gán trong constructor**, lấy từ [pom/dashboard/login/login.page.ts](../pom/dashboard/login/login.page.ts):
```ts
import { Locator, Page } from "@playwright/test";

export class LoginPage {
    page: Page;
    readonly userNameLogin: Locator;
    readonly passwordLogin: Locator;
    loginButton: Locator;

    constructor(page: Page) {
        this.page = page;                    // (1) gán page TRƯỚC
        this.userNameLogin = this.page.getByRole('textbox', { name: 'Username or Email Address' });  // (2) rồi mới dùng this.page
        this.passwordLogin = this.page.getByRole('textbox', { name: 'Password' });
        this.loginButton = this.page.getByRole('button', { name: 'Log In' });
    }
}
```
Vì sao đúng: `this.page = page;` chạy trong thân constructor, nên tại (2) `this.page` đã có giá trị.

### 2.2 Vì sao dùng constructor-assignment mà không cần getter?

Câu hỏi tự nhiên: "Nếu page reload hoặc điều hướng, locator gán 1 lần trong constructor có bị cũ (stale) không?"

**Không** — vì `Locator` trong Playwright được thiết kế **lazy**: gọi `page.getByRole(...)` chỉ tạo ra một "chiến lược tìm phần tử", **chưa** query DOM. DOM chỉ thực sự được query khi bạn gọi action (`.fill()`, `.click()`, `.hover()`...) hoặc `expect(...)`, và mỗi lần gọi nó tự **auto-retry** tìm lại theo DOM hiện tại. Vì vậy Locator không giống `ElementHandle` (loại cũ, dễ bị stale) — gán 1 lần trong constructor là đủ và đúng chuẩn Playwright khuyến nghị.

**Khi nào thực sự cần method thay vì property cố định?** — khi locator phụ thuộc **tham số runtime**, ví dụ tìm theo tên user cụ thể trong lúc chạy test. Lấy từ [pom/dashboard/users/add-user.page.ts:111-113](../pom/dashboard/users/add-user.page.ts):
```ts
async hoverUserName(userName: string) {
    await this.page.getByRole('link', { name: userName, exact: true }).hover();
}
```
Không thể khai `hoverUserNameLoc: Locator` cố định vì lúc viết code chưa biết `userName` là gì — bắt buộc phải là method nhận tham số.

### 2.3 Ưu tiên locator ngữ nghĩa (semantic) hơn XPath/CSS thô

Thứ tự ưu tiên nên theo: `getByRole` > `getByLabel` > `getByText` > `getByTestId` > CSS/XPath (chỉ dùng khi không còn cách nào khác).

**Bẫy thường gặp khi lấy locator từ DevTools:**

1. **Role bị ghi đè bởi thuộc tính `role` tường minh.** Một thẻ `<a href="...">` có role mặc định là `link`, nhưng nếu DOM có `<a role="menuitem" ...>`, accessible role thực tế là `menuitem`, không phải `link`. `getByRole('link', ...)` trong trường hợp này **sẽ không khớp**. Luôn kiểm tra thuộc tính `role` tường minh trên DOM trước khi chọn role cho `getByRole`.

2. **Text bị cắt bởi CSS (`text-overflow: ellipsis`) không phải text thật.** Nếu copy text hiển thị trên UI có dấu "…", đó thường là hiệu ứng hiển thị, không phải nội dung DOM thật. Phải kiểm tra `textContent`/accessible name thật trong tab Elements/Accessibility của DevTools, không copy trực tiếp từ những gì mắt thấy trên trang.

### 2.4 Naming convention

- Property locator cố định: hậu tố `Loc` (`usersTabLoc`, `msgCreateUserLoc`).
- Method trả về locator động (có tham số): tiền tố `get...` hoặc mô tả hành động (`hoverUserName(userName)`), **không** đặt hậu tố `Loc` vì nó không phải property cố định — tên khác nhau giúp người đọc phân biệt ngay "cái này cố định" hay "cái này cần truyền tham số".
- Action method: động từ mô tả hành động (`fillUserName`, `clickLoginButton`, `submitAddUser`).

### 2.5 `readonly` cho locator

```ts
readonly userNameLogin: Locator;
readonly passwordLogin: Locator;
```
`readonly` chỉ cho phép gán giá trị một lần (tại khai báo hoặc trong constructor). Nếu sau này ai vô tình viết `this.userNameLogin = ...` ở một method khác, TypeScript báo lỗi compile ngay — bắt lỗi sớm hơn là để chạy test rồi mới phát hiện hành vi sai (đây chính là loại bug đã từng gặp: `fillPassWord` gọi nhầm `this.userName.fill(...)` thay vì `this.passWord.fill(...)`).

### 2.6 Kế thừa giữa các Page Object

Project dùng kế thừa (`extends`) để tái sử dụng locator/method chung giữa các trang có liên quan:

```
LoginPage                    (pom/dashboard/login/login.page.ts)
   └── DashboardBasePage     (pom/dashboard/dashboard.base.page.ts) — thêm các tab menu chung (Users, Posts, Media...)
         └── UserPage        (pom/dashboard/users/add-user.page.ts) — thêm locator/action riêng cho màn Add User
```

Nhờ vậy, `UserPage` (chỉ định nghĩa riêng phần "Add User") vẫn gọi được `clickUserTab()`, `hoverProfile()`, `login()` — vốn được định nghĩa ở `DashboardBasePage`/`LoginPage`, không cần viết lại.

**Lưu ý khi kế thừa:** class con vẫn phải tự khai `page: Page;` và gọi `super(page)` trong constructor để lớp cha khởi tạo đúng các locator của nó:
```ts
export class UserPage extends DashboardBasePage {
    page: Page;
    // ... locator riêng của UserPage

    constructor(page: Page) {
        super(page);      // để DashboardBasePage + LoginPage khởi tạo locator của chúng
        this.page = page;
        // ... gán locator riêng của UserPage
    }
}
```

---

## 3. Fixtures — mở rộng `test` của Playwright

### 3.1 Vấn đề cần giải quyết

Nhiều file test đều cần login trước khi test — nếu viết lặp lại 4 dòng login trong `beforeEach` của từng file, khi cách login thay đổi (ví dụ thêm bước xác thực 2 lớp) sẽ phải sửa ở rất nhiều nơi.

### 3.2 Custom fixture — [fixtures/dashboard-fixture.ts](../fixtures/dashboard-fixture.ts)

```ts
import { test as base, expect } from '@playwright/test';
import { DashboardBasePage } from '../pom/dashboard/dashboard.base.page';

const test = base.extend<{ dashboardBasePage: DashboardBasePage }>({
    dashboardBasePage: async ({ page }, use) => {
        const userAdmin = process.env.ADMIN_USERNAME!;
        const pswAdmin = process.env.ADMIN_PASSWORD!;

        const dashboardBasePage = new DashboardBasePage(page);
        await dashboardBasePage.openPlaywrightPage();
        await dashboardBasePage.login(userAdmin, pswAdmin);

        await expect(page).toHaveURL("https://pw-practice-dev.playwrightvn.com/wp-admin/");
        await expect(dashboardBasePage.dashboardHeadingLoc).toHaveText("Dashboard");

        await use(dashboardBasePage);   // <-- trả fixture ra cho test dùng
        // Không có code sau use() -> fixture này KHÔNG có teardown
    }
});

export { test };
```

`base.extend<{...}>({...})` tạo ra một bản `test` **mới**, có thêm 1 "nguyên liệu" (fixture) tên `dashboardBasePage` mà mọi hook/test có thể xin dùng. Cấu trúc bên trong luôn là: code **trước** `use(...)` là **setup**, tham số truyền vào `use(...)` là **giá trị fixture trả về**, code **sau** `use(...)` (nếu có) là **teardown** (dọn dẹp sau khi test xong).

### 3.3 Cách dùng trong file test — [tests/dashboard/users/all-users.spec.ts](../tests/dashboard/users/all-users.spec.ts)

```ts
import { expect } from '@playwright/test';
import { UserPage } from '../../../pom/dashboard/users/add-user.page';
import { test } from '../../../fixtures/dashboard-fixture';   // <-- lấy test từ fixture, KHÔNG phải từ '@playwright/test'

test.describe("Users - all users", () => {
    let userPage: UserPage;

    test("Click Users tab", async ({ dashboardBasePage }) => {   // <-- xin fixture qua tham số
        await test.step("Click Users tab", async () => {
            userPage = new UserPage(dashboardBasePage.page);
            await userPage.clickUserTab();
            await expect(userPage.usersHeadingLoc).toBeVisible();
        });
    });
});
```

### 3.4 Cơ chế quan trọng cần hiểu: fixture chỉ resolve **một lần cho mỗi test**

Fixture được tạo ra (chạy phần setup) **đúng một lần** khi có nơi đầu tiên (hook hoặc test body) yêu cầu nó, rồi **dùng chung** xuyên suốt `beforeEach` → test body → `afterEach` của cùng 1 test — bất kể ai là nơi request đầu tiên. Nhờ vậy, trong [tests/dashboard/users/add-user.spec.ts](../tests/dashboard/users/add-user.spec.ts):

```ts
test.beforeEach("Đã login trang with admin user", async ({ dashboardBasePage }) => {
    userName = randomUserName();
    email = randomEmail();
    psw = defaultTestPassword;
    createdUserName = undefined;

    userPage = new UserPage(dashboardBasePage.page);   // login đã chạy xong TRƯỚC dòng này
});
```
`beforeEach` request `dashboardBasePage` (kích hoạt login), nhưng các `test(...)` phía dưới và `afterEach` chỉ cần request `page` (built-in fixture) — không cần request lại `dashboardBasePage` — vì `page` lúc đó đã ở trạng thái "đã login" do cùng 1 fixture xử lý trước đó trong chính test này.

### 3.5 Khi nào vẫn cần login thủ công dù đã có fixture

Fixture chỉ chạy setup **một lần lúc đầu test**. Nếu trong lúc test có bước **logout** (ví dụ để dọn dẹp dữ liệu ở `afterEach`), muốn login lại thì phải gọi thủ công — vì fixture không tự chạy lại giữa chừng:
```ts
test.afterEach("Login by admin & delete normal user", async ({ page }) => {
    if (!createdUserName) return;

    await userPage.hoverProfile();
    await userPage.clickLogout();          // <-- đã logout

    await userPage.openPlaywrightPage();
    await userPage.login(userAdmin, pswAdmin);   // <-- phải tự login lại, fixture không tự làm
    // ... tiếp tục xóa user
});
```

---

## 4. Test data & cleanup pattern

### 4.1 Vấn đề: không phải test nào cũng tạo ra dữ liệu cần dọn

Trong `add-user.spec.ts`, một số test tạo user thành công (cần xóa sau khi test xong), một số test chỉ thử các trường hợp validate lỗi (không tạo ra gì). Nếu `afterEach` luôn chạy xóa vô điều kiện, các test không tạo user sẽ khiến `afterEach` tìm một user không tồn tại → `hover()` vào locator không có → Playwright chờ (auto-retry) đến hết `timeout` của test → **"Target page, context or browser has been closed"** (test bị đóng giữa chừng do quá thời gian, không phải lỗi logic thật).

### 4.2 Giải pháp: "cờ theo dõi" (tracking flag) — phổ biến nhất trong thực tế

So với 2 hướng khác đã cân nhắc (tách nhiều `describe`, hoặc search UI để "đoán" xem user tồn tại chưa), dùng 1 biến cờ là gọn và chính xác nhất — không cần tách file, không cần query UI thừa để đoán:

```ts
let createdUserName: string | undefined;   // chỉ có giá trị khi user THỰC SỰ đã được tạo

test.beforeEach(async ({ dashboardBasePage }) => {
    createdUserName = undefined;   // reset mỗi test — bắt buộc, vì biến khai ở scope describe (dùng chung mọi test)
    // ...
});

test("Add new user with username & email only", async ({ page }) => {
    // ... tạo user
    await expect(userPage.msgCreateUserLoc).toBeVisible();
    createdUserName = userName;   // chỉ gán SAU KHI xác nhận tạo thành công
});

test.afterEach(async ({ page }) => {
    if (!createdUserName) {
        return;   // test này không tạo user -> khỏi dọn, return sớm hợp lệ trong hook
    }
    // ... xóa user theo createdUserName
});
```

Nguyên tắc: chỉ gán `createdUserName` **ngay sau** dòng `expect(...)` xác nhận thành công. Nếu `expect` đó fail, dòng gán không chạy tới → `afterEach` tự biết không cần dọn — không phải đoán, không cần cờ phụ nào khác.

### 4.3 Test độc lập (test isolation) — không dùng chung dữ liệu giữa 2 test

Câu hỏi thực tế đã gặp: viết test "duplicate username" bằng cách tạo user ở **test A**, rồi kiểm tra trùng ở **test B**? — **Sai**, vì 2 lý do:
1. Playwright không đảm bảo thứ tự chạy test.
2. Chính `afterEach` đã xóa sạch user của test A ngay sau khi A kết thúc — đến khi B chạy, user đó không còn tồn tại nữa, việc kiểm tra trùng sẽ luôn thất bại.

**Cách đúng** — cả 2 bước (tạo user gốc + thử tạo trùng) phải nằm trong **cùng một test**, xem [add-user.spec.ts:226-254](../tests/dashboard/users/add-user.spec.ts):
```ts
test("Add new user with duplicate username", async ({ page }) => {
    const testUserName = `dup-${Date.now()}`;

    await test.step("Tạo user gốc thành công", async () => {
        await userPage.fillUserName(testUserName);
        await userPage.fillEmail(email);
        await userPage.submitAddUser();
        await expect(userPage.msgCreateUserLoc).toBeVisible();
        createdUserName = testUserName;   // chỉ user NÀY thực sự tồn tại -> cần dọn
    });

    await test.step("Thử tạo user thứ 2 với username trùng -> mong đợi lỗi", async () => {
        const anotherEmail = `dup2-${Date.now()}@gmail.com`;   // email PHẢI khác, để chắc chắn lỗi là do username, không lẫn lỗi email trùng
        await userPage.fillUserName(testUserName);
        await userPage.fillEmail(anotherEmail);
        await userPage.submitAddUser();
        await expect(userPage.duplicateUserNameMsgLoc).toBeVisible();
    });
});
```
Vì lần tạo thứ 2 bị chặn (không tạo ra gì mới), `createdUserName` chỉ cần trỏ đúng user gốc — `afterEach` xóa đúng 1 user là đủ, không thừa không thiếu.

### 4.4 "Single source of truth" khi tạo dữ liệu test

Bug thực tế đã gặp: viết `fillUserName('123456')` nhưng lại gán `createdUserName = userName` (biến random khác, không phải `'123456'`) → `afterEach` đi xóa nhầm một username không tồn tại. Nguyên tắc để tránh lặp lại lỗi này:

> Giá trị truyền vào action (`fillUserName(...)`) và giá trị gán cho biến theo dõi (`createdUserName`) phải luôn là **cùng một biến**, không phải 2 chuỗi viết tay ở 2 nơi khác nhau.

```ts
test("Add new user with username is number", async ({ page }) => {
    const testUserName = randomNumericString(10);   // 1 biến — dùng lại ở cả 2 chỗ dưới

    await userPage.fillUserName(testUserName);
    // ...
    await expect(userPage.msgCreateUserLoc).toBeVisible();
    createdUserName = testUserName;   // dùng lại đúng biến, không viết tay chuỗi mới
});
```

---

## 5. `utils/` — sinh dữ liệu test ngẫu nhiên

### 5.1 Nguyên tắc thiết kế hàm: đừng đoán trước nhu cầu (YAGNI / "rule of three")

Ban đầu có ý viết `randomUserName(prefix = 'E101-tuyen')` — thêm tham số "phòng khi cần prefix khác". Nhưng nếu hiện tại **chỉ có đúng 1 prefix được dùng trong toàn project**, đó là linh hoạt dư thừa, làm hàm khó đọc hơn mà không giải quyết nhu cầu thật nào. Nên viết hàm cứng (hardcode) trước:
```ts
export function randomUserName(): string {
    return `E101-tuyen-${Date.now()}`;
}
```
Chỉ thêm tham số khi thực sự thấy **từ 2 chỗ trở lên** cần giá trị khác nhau ("rule of three"). Thêm tham số sau này gần như không tốn chi phí (thêm dạng optional/default không phá vỡ chỗ đang gọi không tham số) — nên "chờ đến khi cần" không có rủi ro.

### 5.2 Ba cách viết tham số — chỉ 1 cách đúng cho mục tiêu "không phá vỡ code cũ"

| Cách viết | Ý nghĩa | Vấn đề |
|---|---|---|
| `function f(prefix): string` | Tham số **bắt buộc**, thiếu kiểu dữ liệu | TypeScript coi là `any` ngầm định; mọi chỗ gọi `f()` không tham số sẽ lỗi compile |
| `function f(prefix?: string): string` | Tham số **tùy chọn** | Hợp lệ, nhưng bên trong hàm `prefix` có thể là `undefined`, phải tự xử lý (`prefix ?? 'default'`) |
| `function f(prefix: string = 'default'): string` | **Default parameter** | Tốt nhất: vừa tùy chọn, vừa có sẵn giá trị mặc định, không cần xử lý `undefined` thủ công |

Áp dụng trong [utils/random-data.util.ts](../utils/random-data.util.ts):
```ts
export function randomUserName(prefix: string = 'E101-tuyen'): string { // default parameter
    return `${prefix}-${Date.now()}`;
}
// randomUserName()          -> dùng mặc định 'E101-tuyen'
// randomUserName('Tuyen')   -> dùng 'Tuyen'
```

### 5.3 Bẫy "global script" khi quên `export`

Nếu một file `.ts` **không có bất kỳ dòng `import`/`export`** nào, TypeScript coi nó là **global script**, không phải module — mọi khai báo top-level (function, const...) sẽ tự động lọt vào **scope toàn cục của cả project**, file khác gọi được mà không cần import! Đây là hành vi **ẩn và nguy hiểm**: chỉ cần sau đó thêm 1 dòng `import` bất kỳ vào file đó (vô tình biến nó thành module thật), toàn bộ chỗ đang gọi ngầm sẽ vỡ lỗi khó hiểu.

**Luôn viết `export`** cho những gì cần dùng ở file khác, và **`import` tường minh** ở nơi sử dụng:
```ts
// utils/random-data.util.ts
export function randomUserName(...): string { ... }

// tests/dashboard/users/add-user.spec.ts
import { randomUserName } from '../../../utils/random-data.util';
```

### 5.4 Dùng `faker.js` cho dữ liệu random phức tạp hơn

Cài: `npm install -D @faker-js/faker`. Bọc `faker` bên trong `utils/`, không import `faker` trực tiếp vào file test — để nếu đổi thư viện sau này, chỉ sửa đúng 1 file:
```ts
import { faker } from '@faker-js/faker';

export function randomNumericString(length: number): string {
    return faker.string.numeric(length);       // toàn số, ví dụ "4827193056"
}
export function randomAlphaString(length: number): string {
    return faker.string.alpha(length);          // toàn chữ, ví dụ "qkfmzplwer"
}
export function randomAlphanumericString(length: number): string {
    return faker.string.alphanumeric(length);   // trộn cả hai
}
```
Lợi ích so với chuỗi hardcode cố định (`'1234567890'`): mỗi lần chạy test ra giá trị khác nhau, tránh lỗi "username đã tồn tại" nếu lần chạy trước dọn dẹp thất bại giữa chừng và để sót dữ liệu.

---

## 6. Assertion trong test — không tự viết locator, luôn lấy từ POM

**Sai (đã gặp ban đầu trong `login.spec.ts`):**
```ts
await loginPage.fillUserName(userName);
await expect(page.getByRole('textbox', { name: 'Username or Email Address' })).toHaveValue(userName);
// ^ tự viết lại y hệt locator đã có sẵn trong LoginPage -> nếu selector đổi, phải sửa 2 chỗ
```

**Đúng — hiện tại trong [tests/dashboard/login/login.spec.ts](../tests/dashboard/login/login.spec.ts):**
```ts
await loginPage.fillUserNameLogin(userName);
await expect(loginPage.userNameLogin).toHaveValue(userName);   // dùng lại chính locator đã khai trong POM
```

Nguyên tắc: POM không chỉ chứa locator để **thao tác** (fill, click), mà còn chứa locator để **assertion**. Test file chỉ orchestrate — gọi action và assert — không bao giờ tự viết `page.getByRole(...)`/`page.getByText(...)` trực tiếp. Với locator **phụ thuộc tham số runtime** (ví dụ message lỗi chứa tên user), POM nên expose qua **method trả về `Locator`**, ví dụ [login.page.ts:50-53](../pom/dashboard/login/login.page.ts):
```ts
async getErrorMessage(userName: string) {
    const errorMsg = this.page.getByText(`Error: The username ${userName} is not registered on this site. ...`);
    return errorMsg;
}
```

---

## 7. Quản lý biến môi trường: `.env`, `.gitignore`, `dotenv`

### 7.1 Vì sao cần bước này

Thông tin nhạy cảm (URL môi trường, tài khoản, mật khẩu admin) không nên hardcode trong code — vì code sẽ được push lên GitHub công khai. Giải pháp chuẩn: đưa vào `.env` (không commit), code đọc qua `process.env`.

### 7.2 `.gitignore` phải loại trừ `.env` — làm TRƯỚC khi tạo `.env`

```
# Environment variables
.env
```
Làm trước để tránh trường hợp lỡ `git add` file `.env` thật trước khi gitignore có hiệu lực.

### 7.3 Cần cả 2 bước: cài `dotenv` VÀ bật code đọc nó — thiếu 1 trong 2 đều không chạy

`.env` chỉ là file text thuần — Node.js **không tự đọc được**. Cần thư viện `dotenv` để đọc file, parse từng dòng `KEY=VALUE`, và gán vào `process.env`.

```
npm install -D dotenv
```
Rồi bật đoạn code (đã có sẵn dạng comment trong [playwright.config.ts](../playwright.config.ts) do Playwright scaffold mặc định):
```ts
import dotenv from 'dotenv';
import path from 'path';
dotenv.config({ path: path.resolve(__dirname, '.env') });
```
- Chỉ cài package mà không bật code gọi `dotenv.config()` → package nằm không, `.env` không bao giờ được đọc.
- Chỉ bật code mà không cài package → lỗi `Cannot find module 'dotenv'` (dòng `import dotenv from 'dotenv'` không tìm được gì để import).

### 7.4 Cú pháp file `.env` — KHÔNG phải cú pháp JavaScript

**Sai** (viết theo kiểu JS, sẽ khiến giá trị bị lẫn dấu nháy/`;` thật vào trong chuỗi):
```
userAdmin = 'betterbytes.academy.admin';
```
**Đúng** — chỉ `KEY=value`, không `const`, không dấu nháy, không `;`:
```
BASE_URL=https://pw-practice-dev.playwrightvn.com/wp-admin
ADMIN_USERNAME=betterbytes.academy.admin
ADMIN_PASSWORD=StrongPass@BetterBytesAcademy
```

### 7.5 Đọc trong code — `process.env` là `string | undefined`

`process` là **global object có sẵn của Node.js** — không cần `import` gì cả (chỉ cần `@types/node` có trong `devDependencies` để TypeScript hiểu kiểu, project đã có sẵn). Nhưng kiểu trả về là `string | undefined` (TypeScript không thể chắc biến môi trường có tồn tại lúc runtime), nên cần `!` (non-null assertion) nếu chắc chắn nó luôn có giá trị:
```ts
this.baseUrl = process.env.BASE_URL!;
```
Áp dụng thực tế trong [dashboard-fixture.ts](../fixtures/dashboard-fixture.ts):
```ts
const userAdmin = process.env.ADMIN_USERNAME!;
const pswAdmin = process.env.ADMIN_PASSWORD!;
```

### 7.6 `.env.example` — file NÊN được commit (khác với `.env`)

Vì `.env` thật bị gitignore, người khác clone project sẽ không biết cần khai báo biến gì. Tạo `.env.example` với cùng tên key nhưng để trống giá trị, và **push file này lên GitHub**:
```
BASE_URL=
ADMIN_USERNAME=
ADMIN_PASSWORD=
```

---

## 8. Cấu hình TypeScript — `tsconfig.json`

### 8.1 Vì sao project cần file này

Không có `tsconfig.json`, editor (VS Code) phải tự đoán cấu hình cho từng file rời rạc ("inferred project"), dễ dẫn đến các global type (như `process` từ `@types/node`) không được nhận diện đúng, gây báo lỗi đỏ dù package đã cài đủ trong `node_modules`.

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "commonjs",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

### 8.2 Bẫy: `"moduleResolution": "node"` bị deprecated

Bản TypeScript mới đổi tên giá trị `"node"` thành `"node10"` (dùng cho Node.js < v10, chỉ hỗ trợ `require`) và đánh dấu deprecated — báo cảnh báo *"will stop functioning in TypeScript 7.0"*. Vì code hiện tại dùng import tương đối không có đuôi file (`from './login/login.page'`), **không nên** đổi sang `"node16"`/`"nodenext"` (yêu cầu ghi đuôi file, sẽ làm gãy toàn bộ import hiện có). Giải pháp đơn giản nhất: **xóa hẳn dòng `moduleResolution`**, để TypeScript tự suy luận từ `"module": "commonjs"` — hành vi giữ nguyên, chỉ không còn khai tường minh giá trị bị deprecated.

---

## 9. Checklist tổng hợp khi tạo Page Object / Test mới

- [ ] Locator khai kiểu `Locator`, gán trong constructor (không phải property initializer gọi `this.page` trước khi gán).
- [ ] Ưu tiên `getByRole`/`getByLabel`/`getByText` hơn XPath; kiểm tra `role` tường minh và text thật trong DevTools trước khi tin vào locator copy ra.
- [ ] Locator cố định → property + hậu tố `Loc`; locator phụ thuộc tham số → method riêng, không đặt hậu tố `Loc`.
- [ ] Thêm `readonly` cho locator không đổi sau khi gán.
- [ ] Test không tự viết `page.getByRole(...)` để assertion — luôn lấy locator từ POM.
- [ ] Nếu test tạo dữ liệu (user, post...) → dùng cờ theo dõi (`createdXxx`), reset ở `beforeEach`, gán sau `expect` xác nhận thành công, kiểm tra trong `afterEach` trước khi dọn dẹp.
- [ ] Test không phụ thuộc dữ liệu còn sót lại từ test khác — mọi bước cần thiết phải nằm trong cùng 1 test.
- [ ] Hàm sinh dữ liệu random đặt trong `utils/`, luôn có `export`; chỉ thêm tham số khi có ≥ 2 nhu cầu giá trị khác nhau thật sự.
- [ ] Không hardcode tài khoản/mật khẩu/URL — đọc từ `.env` qua `process.env.KEY!`.
