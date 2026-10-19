# Ghi chú: Refactor BasePage trong dự án Playwright

> Tổng hợp kiến thức từ quá trình refactor `DashboardBasePage` theo góp ý của mentor.
> Dự án: `playwright-codebase` (Playwright + TypeScript, test trang WordPress).

---

## Mục lục

1. [Bối cảnh: góp ý của mentor](#1-bối-cảnh-góp-ý-của-mentor)
2. [Nguyên tắc kế thừa "là một"](#2-nguyên-tắc-kế-thừa-là-một)
3. [Các bước refactor](#3-các-bước-refactor)
4. [`page` và page object: một tivi, nhiều remote](#4-page-và-page-object-một-tivi-nhiều-remote)
5. [Fixture](#5-fixture)
6. [Các hàm trong BasePage](#6-các-hàm-trong-basepage)
7. [Regex trong `toHaveURL`](#7-regex-trong-tohaveurl)
8. [TypeScript và `tsc --noEmit`](#8-typescript-và-tsc---noemit)
9. [Lỗi tsconfig với TypeScript 7](#9-lỗi-tsconfig-với-typescript-7)
10. [Checklist review cuối](#10-checklist-review-cuối)
11. [Hướng nâng cao sau này](#11-hướng-nâng-cao-sau-này)

---

## 1. Bối cảnh: góp ý của mentor

> `DashboardBasePage` đang extends `LoginPage`.
> - 2 trang này là 2 trang độc lập (Dashboard không phải là một dạng "trang Login"). Nếu kế thừa thì nó sẽ mang theo cả những locator/action của trang Login không cần thiết.
> - Thường sẽ tạo `BasePage` ở gốc (chứa `page: Page`, các hàm `goto`, `waitForLoadState`... dùng chung), sau đó cả Login và Dashboard đều extend từ phần base này.

### Cấu trúc trước khi refactor

```
LoginPage                     (ô username, password, nút Log In, + 3 heading của Dashboard ❌)
 ├── DashboardBasePage        ❌ Dashboard "là một" trang Login?
 │    ├── HomePage
 │    ├── UpdatesPage
 │    ├── UserPage
 │    └── ProfilePage
 └── StoreFrontHomePage      ❌ Storefront cũng "là một" trang Login?
```

### Cấu trúc sau khi refactor

```
BasePage                      (page, baseUrl, goto, waitForPageLoad, getTitle)
 ├── LoginPage                (chỉ những gì của trang Login)
 ├── DashboardBasePage        (menu, logout, heading Dashboard)
 │    ├── HomePage
 │    ├── UpdatesPage
 │    ├── UserPage
 │    └── ProfilePage
 └── StoreFrontHomePage
```

### Hậu quả của cách kế thừa cũ (thấy rõ ngay trong code)

| Dấu hiệu | Vì sao là vấn đề |
|---|---|
| `userPage.login(...)` chạy được | Trang Users mà lại có hàm đăng nhập, sai về ý nghĩa |
| VS Code gợi ý `userNameLogin`, `loginButton` trên mọi trang Dashboard | Các trang đó không hề có những phần tử này |
| 3 heading của Dashboard nằm trong `LoginPage` | Code bị đặt sai chỗ vì kế thừa sai |
| Sửa `LoginPage` là ảnh hưởng ~45 trang Dashboard và Storefront | Liên kết quá chặt, sửa một chỗ hỏng nhiều chỗ |

### Hướng của mentor có tốt không?

**Có.** Nhưng cần hiểu đúng là nó "tốt hơn" ở chỗ **code dễ hiểu và dễ bảo trì**, chứ không làm test chạy nhanh hơn.

---

## 2. Nguyên tắc kế thừa "là một"

Chỉ dùng `class A extends B` khi câu **"A là một B"** nghe hợp lý.

| Quan hệ | "là một"? | Kết luận |
|---|---|---|
| `HomePage extends DashboardBasePage` | Home **là một** trang trong Dashboard | ✅ Đúng |
| `UserPage extends DashboardBasePage` | Users **là một** trang trong Dashboard | ✅ Đúng |
| `LoginPage extends BasePage` | Login **là một** trang web | ✅ Đúng |
| `DashboardBasePage extends LoginPage` | Dashboard **là một** trang Login? | ❌ Sai |
| `StoreFrontHomePage extends LoginPage` | Storefront **là một** trang Login? | ❌ Sai |

### Lưu ý để không đi quá đà

- **BasePage phải thật gọn.** Chỉ chứa thứ **mọi trang** đều dùng (`page`, `goto`, chờ trang tải xong). Đừng biến nó thành nơi chứa mọi hàm tiện ích, vì như vậy sẽ lặp lại đúng vấn đề cũ.
- **Không kế thừa quá nhiều tầng.** `BasePage → DashboardBasePage → trang cụ thể` là đủ.

---

## 3. Các bước refactor

### Bước 1: Tạo `BasePage` ở gốc

File mới: `src/pages/base.page.ts`

```ts
import { Page } from "@playwright/test";

export class BasePage {
    readonly page: Page;
    readonly baseUrl: string;

    constructor(page: Page) {
        this.page = page;
        this.baseUrl = process.env.BASE_URL!;
    }

    async goto(path: string = "") {
        await this.page.goto(this.baseUrl + path);
    }

    async waitForPageLoad(state: "load" | "domcontentloaded" | "networkidle" = "load") {
        await this.page.waitForLoadState(state);
    }

    async getTitle() {
        return this.page.title();
    }
}
```

> `process.env.BASE_URL!`: dấu `!` báo cho TypeScript biết "giá trị này chắc chắn có, đừng báo lỗi `undefined`". Giá trị được đọc từ file `.env` thông qua `dotenv` trong `playwright.config.ts`.

### Bước 2: `LoginPage` extends `BasePage`

File chuyển sang `src/pages/login/login.page.ts`, vì Login không thuộc Dashboard.

```ts
import { BasePage } from "@pages/base.page";
import { Locator, Page } from "@playwright/test";

export class LoginPage extends BasePage {
    readonly userNameLogin: Locator;
    readonly passwordLogin: Locator;
    loginButton: Locator;

    constructor(page: Page) {
        super(page);   // gọi constructor của BasePage → gán this.page, this.baseUrl
        this.userNameLogin = this.page.getByRole('textbox', { name: 'Username or Email Address' });
        this.passwordLogin = this.page.getByRole('textbox', { name: 'Password' });
        this.loginButton = this.page.getByRole('button', { name: 'Log In' });
    }

    async openPlaywrightPage() {
        await this.goto();   // dùng hàm của BasePage
    }

    async login(userName: string, password: string) { ... }
    // fillUserNameLogin, fillPasswordLogin, clickLoginButton, getErrorMessage...
}
```

Những gì đã **xoá** khỏi LoginPage:

- `page: Page;`, `baseUrl: string;`, `this.page = page;`, `this.baseUrl = ...`: BasePage đã lo phần này.
- 3 locator heading của Dashboard: chuyển sang `DashboardBasePage`.

> **`super(page)`**: trong class con, constructor **bắt buộc** phải gọi `super(...)` trước khi dùng `this`. `super(page)` nghĩa là "chạy constructor của class cha với tham số `page`".

### Bước 3: `DashboardBasePage` extends `BasePage`

```ts
import { BasePage } from "@pages/base.page";

export class DashboardBasePage extends BasePage {
    // ... các locator menu, logout ...
    dashboardHeadingLoc: Locator;
    atAGlanceHeadingLoc: Locator;
    activityHeadingLoc: Locator;

    constructor(page: Page) {
        super(page);
        // ... các locator menu ...
        this.dashboardHeadingLoc = this.page.getByRole('heading', { name: 'Dashboard' });
        this.atAGlanceHeadingLoc = this.page.getByRole('heading', { name: 'At a Glance' });
        this.activityHeadingLoc = this.page.getByRole('heading', { name: 'Activity' });
    }
}
```

> Xét đúng nghĩa thì 3 heading này thuộc trang Home. Đặt ở `DashboardBasePage` là để fixture dùng được khi kiểm tra đăng nhập thành công.

### Bước 4: Dọn các class con

Ở `HomePage`, `UpdatesPage`, `UserPage` và `ProfilePage`: xoá `page: Page;` và `this.page = page;`. Chỉ cần `super(page)`.

### Bước 5: Storefront

```ts
export class StoreFrontHomePage extends BasePage { ... }   // trước đây: extends LoginPage
```

### Bước 6: Sửa fixture

Fixture cần **2 page object riêng**: một để đăng nhập, một để kiểm tra kết quả.

```ts
dashboardBasePage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.openPlaywrightPage();
    await loginPage.login(process.env.ADMIN_USERNAME!, process.env.ADMIN_PASSWORD!);

    const dashboardBasePage = new DashboardBasePage(page);
    await expect(page).toHaveURL(/\/wp-admin\/$/);
    await expect(dashboardBasePage.dashboardHeadingLoc).toHaveText("Dashboard");
    // ...
    await use(dashboardBasePage);
}
```

### Bước 7: Sửa các spec

Các chỗ trước đây gọi hàm login **trên page Dashboard** đều phải đổi sang dùng `LoginPage`.

**`afterEach` (đăng nhập lại admin để dọn dữ liệu):**

```ts
// ❌ Trước
await userPage.openPlaywrightPage();
await userPage.login(userAdmin, pswAdmin);

// ✅ Sau
const loginPage = new LoginPage(page);
await loginPage.openPlaywrightPage();
await loginPage.login(userAdmin, pswAdmin);
await userPage.clickUserTab();   // phần còn lại giữ nguyên
```

**Step "Logout & Login with new account":**

```ts
await userPage.hoverProfile();
await userPage.clickLogout();          // vẫn dùng userPage: logout là menu của Dashboard

const loginPage = new LoginPage(page);
await loginPage.login(userName, psw);  // không cần openPlaywrightPage()
```

> **Vì sao ở đây không cần mở trang login?** Khi logout, WordPress **tự chuyển** về `wp-login.php?loggedout=true`. Còn trong `afterEach` thì không chắc tab đang ở trang nào, nên phải mở trang login trước.

**`login.spec.ts` (test AUTH_002):** assert heading qua `new DashboardBasePage(page)` thay vì qua `loginPage`.

### Bước 8: Kiểm tra

```bash
npx tsc --noEmit
npx playwright test tests/login tests/dashboard/users tests/dashboard/dashboard tests/storefront/home.spec.ts
```

> **Mẹo:** chạy `tsc --noEmit` ngay sau bước 3. Danh sách lỗi lúc đó chính là **danh sách những chỗ cần sửa** ở bước 6–7.

### Bảng các chỗ bị ảnh hưởng khi bỏ kế thừa LoginPage

| File | Đang dùng gì từ LoginPage |
|---|---|
| `src/fixtures/dashboard-fixture.ts` | `openPlaywrightPage()`, `login()`, 3 heading |
| `tests/dashboard/dashboard/home.spec.ts` | 3 heading |
| `tests/dashboard/users/add-user.spec.ts` | `openPlaywrightPage()`, `login()` trong `afterEach` |
| `tests/dashboard/users/account-permissions.spec.ts` | `fill...Login()`, `clickLoginButton()` (afterEach và 2 step) |
| `src/pages/storefront/home.page.ts` | Cũng `extends LoginPage` |

---

## 4. `page` và page object: một tivi, nhiều remote

### Phân biệt

| | Là gì | Số lượng trong 1 test |
|---|---|---|
| **`page`** | Đối tượng thật của Playwright, đại diện cho **một tab trình duyệt** | 1 |
| **Page object** (`LoginPage`, `UserPage`...) | Class bạn tự viết, **giữ tham chiếu** tới `page` cùng các locator và hàm thao tác | Bao nhiêu cũng được |

```ts
const loginPage     = new LoginPage(page);          // loginPage.page     ──┐
const dashboardPage = new DashboardBasePage(page);  // dashboardPage.page ──┴──► cùng 1 tab
```

**Hình dung:** `page` là **cái tivi**, mỗi page object là **một cái remote** với bộ nút khác nhau. Đổi remote thì vẫn điều khiển cái tivi đó, chỉ là có nút khác để bấm.

### Diễn biến khi đăng nhập

```ts
await loginPage.login(user, psw);                  // tab chuyển sang /wp-admin/
const dashboardPage = new DashboardBasePage(page); // cầm remote mới, vẫn cùng tab
await expect(dashboardPage.dashboardHeadingLoc).toHaveText("Dashboard");
```

`dashboardPage` không cần biết trang trước đó là gì. Nó chỉ nhìn vào **trạng thái hiện tại** của tab.

### Locator là "lười"

```ts
this.dashboardHeadingLoc = this.page.getByRole('heading', { name: 'Dashboard' });
```

Dòng này **chưa tìm gì trên trang cả**, nó chỉ ghi lại *cách tìm*. Playwright chỉ thật sự tìm phần tử khi bạn gọi `click()`, `fill()` hoặc `expect(...)`.

Vì vậy, tạo page object **trước hay sau** khi chuyển trang đều được:

```ts
const loginPage     = new LoginPage(page);
const dashboardPage = new DashboardBasePage(page);   // tạo ngay từ đầu cũng OK

await loginPage.openPlaywrightPage();
await loginPage.login(user, psw);
await expect(dashboardPage.dashboardHeadingLoc).toHaveText("Dashboard");   // vẫn đúng
```

### Vì sao không dùng tiếp `loginPage` để assert?

Vì sau khi refactor, `loginPage` **không còn** `dashboardHeadingLoc`. Mỗi page object chỉ chứa những gì của trang đó. **Trang nào đang hiển thị thì dùng page object của trang đó**, còn tab bên dưới thì vẫn là một.

---

## 5. Fixture

### Fixture `dashboardBasePage` làm gì

Trước mỗi test có dùng nó, fixture sẽ: mở trang, đăng nhập admin, kiểm tra đã vào Dashboard, rồi trả `dashboardBasePage` cho test qua `use(...)`.

### Ý nghĩa dòng `userPage = new UserPage(dashboardBasePage.page);`

```ts
test.beforeEach(async ({ dashboardBasePage }) => {
    userPage = new UserPage(dashboardBasePage.page);
});
```

| Phần | Ý nghĩa |
|---|---|
| `dashboardBasePage` | Do fixture trả về. Lúc này đã đăng nhập xong |
| `.page` | Cái tab bên trong, đang ở trạng thái **đã đăng nhập** |
| `new UserPage(...)` | Tạo page object của trang Users, gắn vào tab đó (đây không phải gọi hàm mà là tạo đối tượng) |
| `userPage = ...` | Lưu vào biến ở scope `describe` để `test` và `afterEach` dùng lại |

Vì `UserPage extends DashboardBasePage`, `userPage` có cả:

- Hàm riêng của trang Users: `fillSearchUser()`, `clickDeleteUser()`...
- Hàm chung của Dashboard: `clickUserTab()`, `hoverProfile()`, `clickLogout()`...

### Cách viết tương đương

```ts
test.beforeEach(async ({ dashboardBasePage, page }) => {
    userPage = new UserPage(page);   // page === dashboardBasePage.page
});
```

> ⚠️ **Vẫn phải để `dashboardBasePage` trong tham số.** Chính việc nhắc tên nó mới **kích hoạt fixture đăng nhập**. Bỏ đi thì fixture không chạy và test bắt đầu ở trạng thái chưa đăng nhập.

---

## 6. Các hàm trong BasePage

### `goto(path = "")`

Mở `baseUrl + path`. Gọi `goto()` không truyền gì thì mở `baseUrl`.

### `waitForPageLoad(state = "load")`

Gói lại `page.waitForLoadState()` để mọi trang đều gọi được.

**Cú pháp:**

- `state: "load" | "domcontentloaded" | "networkidle"`: chỉ nhận 1 trong 3 giá trị (union type). Gõ sai thì TypeScript báo lỗi ngay.
- `= "load"`: giá trị mặc định khi không truyền tham số.

**3 trạng thái** (từ sớm đến muộn):

| Trạng thái | Chờ đến khi | Ghi chú |
|---|---|---|
| `domcontentloaded` | HTML đọc xong, có cây DOM | Ảnh, CSS có thể chưa xong |
| `load` | Tải xong toàn bộ (ảnh, CSS, script) | Mặc định |
| `networkidle` | Không có request mạng nào trong ≥ 500ms | Playwright **khuyên không dùng**: chậm, dễ làm test chập chờn |

**Thực tế phần lớn trường hợp KHÔNG cần hàm này**, vì Playwright có **auto-wait**:

- `page.goto()` tự chờ tới trạng thái `load`.
- `click()`, `fill()` và `expect(...).toBeVisible()` tự chờ phần tử sẵn sàng.

Cách chờ chuẩn là **chờ phần tử cụ thể**, chứ không chờ cả trang:

```ts
await expect(dashboardPage.dashboardHeadingLoc).toBeVisible();
```

### `getTitle()`

Trả về nội dung thẻ `<title>`, tức chữ hiện trên tab trình duyệt. Phải `await` mới lấy được chuỗi.

```ts
// ❌ Lấy 1 lần duy nhất, trang chưa chuyển xong là fail
expect(await userPage.getTitle()).toBe("Users ‹ ...");

// ✅ Tự chờ và thử lại tới khi đúng hoặc hết thời gian
await expect(userPage.page).toHaveTitle(/Users/);
```

`getTitle()` chỉ hữu ích khi cần **lấy giá trị để dùng tiếp** (ghi log, so sánh trước và sau), không phải để assert.

> **Nguyên tắc chung:** luôn ưu tiên `await expect(...)` của Playwright (có tự chờ và thử lại) hơn việc tự lấy giá trị rồi so sánh.

---

## 7. Regex trong `toHaveURL`

### Chuỗi và regex

```ts
await expect(page).toHaveURL("https://pw-practice-dev.playwrightvn.com/wp-admin/"); // chuỗi: khớp chính xác 100%
await expect(page).toHaveURL(/wp-admin/);                                          // regex: chỉ cần CHỨA "wp-admin"
```

- Hai dấu `/` bao quanh là **ký hiệu mở và đóng** regex, giống dấu `"` bao quanh chuỗi.
- Ưu điểm của regex: **không gắn cứng domain**, nên chạy được trên nhiều môi trường (dev, staging...).

### `/wp-admin/` quá lỏng

Khi đăng nhập **thất bại**, URL có thể là:

```
https://.../wp-login.php?redirect_to=https%3A%2F%2F...%2Fwp-admin%2F
```

URL này cũng **chứa** `wp-admin`, nên assertion vẫn pass dù chưa đăng nhập được.

### Viết chặt hơn

```ts
await expect(page).toHaveURL(/\/wp-admin\/$/);
```

| Ký hiệu | Ý nghĩa |
|---|---|
| `\/` | Dấu `/` **thật** trong URL |
| `wp-admin` | Chữ cần tìm |
| `$` | URL phải **kết thúc** ở đây |

Như vậy URL phải **kết thúc bằng `/wp-admin/`**: khớp với `https://bất-kỳ-domain/wp-admin/` và không khớp với URL trang login ở trên.

### Vì sao phải viết `\/`? (escape)

JavaScript đọc regex từ trái sang phải. Gặp `/` đầu tiên thì biết regex mở, gặp `/` **tiếp theo** thì hiểu là regex **đóng**.

```
/  wp-admin  /
↑            ↑
mở           đóng
```

Nếu nội dung cũng có dấu `/` mà viết thẳng ra:

```ts
//wp-admin/$/     // ❌ JavaScript thấy "//" và hiểu là COMMENT, phần sau bị bỏ qua
```

Đặt `\` phía trước để báo **"ký tự ngay sau tôi là chữ thường, không phải ký hiệu"**:

```
/  \/  wp-admin  \/  $  /
↑  ↑             ↑   ↑  ↑
│  / thật        │   │  đóng regex
mở regex       / thật  kết thúc chuỗi
```

**Giống hệt cách bạn đã làm với chuỗi:**

```ts
"Cô ấy nói "xin chào""        // ❌ dấu " thứ 2 bị hiểu là đóng chuỗi
"Cô ấy nói \"xin chào\""      // ✅ \" là dấu " thật
```

### Escape dấu chấm: `\.`

```ts
await expect(page).toHaveURL(/user-new\.php$/);
```

Trong regex, `.` nghĩa là **"ký tự bất kỳ"**. Muốn tìm dấu chấm thật thì phải viết `\.`. Nếu không escape, `/user-new.php$/` sẽ khớp cả `user-newXphp`.

### Cách viết không cần escape: `new RegExp`

```ts
await expect(page).toHaveURL(new RegExp("/wp-admin/$"));
```

Regex được tạo từ **chuỗi**, không có dấu `/` bao quanh, nên dấu `/` bên trong không bị hiểu nhầm. Kết quả giống hệt `/\/wp-admin\/$/`.

### Bảng tóm tắt regex hay dùng

| Regex | Khớp khi URL... |
|---|---|
| `/wp-admin/` | chứa `wp-admin` ở bất kỳ đâu |
| `/\/wp-admin\/$/` | kết thúc bằng `/wp-admin/` |
| `/user-new\.php$/` | kết thúc bằng `user-new.php` |
| `/Users/` (trong `toHaveTitle`) | tiêu đề chứa `Users` |

---

## 8. TypeScript và `tsc --noEmit`

### Lỗi "This is not the tsc command you are looking for"

**Nguyên nhân:**

- Dự án **chưa cài `typescript`**: `package.json` chỉ có `@playwright/test`, `@faker-js/faker`, `@types/node`, `dotenv`.
- Playwright vẫn chạy được `.ts` vì có sẵn bộ chuyển đổi riêng bên trong, nhưng bộ này **chỉ dịch code để chạy, không kiểm tra lỗi kiểu**.
- Gõ `npx tsc` thì npx tải nhầm một package **trùng tên** là `tsc`. Package đó không phải TypeScript, chỉ in ra cảnh báo.

**Cách sửa:**

```bash
npm install -D typescript
npx tsc --noEmit
```

### Đọc kết quả

| Kết quả | Ý nghĩa |
|---|---|
| Không in ra gì, quay về `$` | ✅ Không có lỗi (với `tsc`, không có output nghĩa là pass) |
| `file.ts:42:24 - error TS2339: Property 'login' does not exist on type 'UserPage'.` | ❌ Có lỗi; bấm vào đường dẫn để nhảy tới đúng dòng |
| `Found 5 errors in 3 files.` | Dòng tổng kết |

`--noEmit` nghĩa là chỉ kiểm tra lỗi, **không sinh file `.js`**, nên không làm "rác" thư mục dự án.

### Thêm script cho tiện

```json
"scripts": {
  "typecheck": "tsc --noEmit"
}
```

Sau đó chỉ cần chạy `npm run typecheck`.

### `tsc` và chạy test khác nhau thế nào

| | Kiểm tra gì |
|---|---|
| `npx tsc --noEmit` | Code **đúng kiểu** không (gọi hàm, thuộc tính có tồn tại không) |
| `npx playwright test` | Hành vi có **đúng** không (trang có hoạt động như mong đợi không) |

Refactor xong thì cần **cả hai** đều pass.

---

## 9. Lỗi tsconfig với TypeScript 7

Khi cài `typescript` bản mới nhất (7.x), `tsc` báo 5 lỗi trong `tsconfig.json`:

```
tsconfig.json(10,5):  error TS5102: Option 'baseUrl' has been removed. Please remove it from your configuration.
tsconfig.json(12,19): error TS5090: Non-relative paths are not allowed. Did you forget a leading './'?
... (lặp lại cho các dòng paths còn lại)
```

- **Lỗi 1:** TypeScript 7 đã **xoá hẳn** tuỳ chọn `baseUrl`.
- **Lỗi 2–5:** trước đây `"src/pages/*"` được tính từ `baseUrl`. Không còn `baseUrl` thì đường dẫn trong `paths` phải **bắt đầu bằng `./`**, tức tính từ vị trí file tsconfig.json.

### Cách sửa

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "commonjs",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "types": ["node"],
    "paths": {
      "@data/*": ["./src/data/*"],
      "@pages/*": ["./src/pages/*"],
      "@fixtures/*": ["./src/fixtures/*"],
      "@utils/*": ["./src/utils/*"]
    }
  }
}
```

Các import `@pages/...` vẫn trỏ đúng thư mục như cũ.

### Nếu cứ giữ `baseUrl` thì sao?

| Công cụ | Ảnh hưởng |
|---|---|
| `npx playwright test` | Vẫn chạy bình thường (Playwright tự đọc tsconfig) |
| `npx tsc --noEmit` | **Luôn báo lỗi**, nên lệnh kiểm tra gần như mất tác dụng |
| VS Code | tsconfig bị gạch đỏ, gợi ý code cho import `@pages/...` có thể không ổn định |

**Lựa chọn khác:** cài `npm install -D typescript@5` (bản 5.x vẫn hỗ trợ `baseUrl`). Cách này chỉ là hoãn lại, khi nâng cấp TypeScript vẫn phải sửa, nên sửa luôn là gọn nhất.

---

## 10. Checklist review cuối

### ✅ Đã hoàn thành

- [x] Tạo `BasePage` ở gốc (`src/pages/base.page.ts`)
- [x] `LoginPage extends BasePage`, chuyển sang `src/pages/login/`, bỏ 3 heading Dashboard
- [x] `DashboardBasePage extends BasePage`, nhận 3 heading
- [x] `StoreFrontHomePage extends BasePage`
- [x] Các class con bỏ `page: Page` và `this.page = page` thừa
- [x] Fixture: `LoginPage` để đăng nhập, `DashboardBasePage` để assert
- [x] Các spec dùng `LoginPage` khi cần đăng nhập lại
- [x] Không còn chỗ nào `extends LoginPage`
- [x] Ghi rõ kiểu `baseUrl: string`, bỏ import không dùng
- [x] Regex chặt: `/\/wp-admin\/$/`, `/user-new\.php$/`
- [x] Sửa tsconfig cho TypeScript 7
- [x] `npx tsc --noEmit` không báo lỗi

### Việc tuỳ chọn

- [ ] `git init`, commit một bản, rồi xoá các dòng code cũ đang để dạng comment
- [ ] URL gắn cứng ở `tests/storefront/home.spec.ts` (nằm ngoài phạm vi refactor)
- [ ] Chạy test để xác nhận hành vi:

```bash
npx playwright test tests/login tests/dashboard/users tests/dashboard/dashboard tests/storefront/home.spec.ts
```

> **Vì sao nên dùng git thay vì comment code cũ?** Comment làm code rối và dễ bị quên. Git lưu lại toàn bộ lịch sử, nên khi cần bạn có thể xem lại hoặc khôi phục bản cũ bất cứ lúc nào.

---

## 11. Hướng nâng cao sau này

Hai việc này **không thuộc** lần refactor hiện tại, nhưng là bước tiếp theo đáng làm.

### 11.1. Fixture trả về sẵn page object

Thay vì test nào cũng tự `new UserPage(...)`:

```ts
const test = base.extend<{ dashboardBasePage: DashboardBasePage; userPage: UserPage }>({
    dashboardBasePage: async ({ page }, use) => { /* đăng nhập như hiện tại */ },
    userPage: async ({ dashboardBasePage }, use) => {
        await use(new UserPage(dashboardBasePage.page));
    },
});

// Trong test:
test("...", async ({ userPage }) => {
    await userPage.clickUserTab();
});
```

Lợi ích: test ngắn hơn, không còn lặp lại code tạo đối tượng.

### 11.2. `storageState`: chỉ đăng nhập một lần

Hiện tại **mỗi test đều đăng nhập lại** qua giao diện, khá chậm. Với `storageState`, bạn đăng nhập một lần trong bước setup, lưu cookie ra file, rồi các test dùng lại file đó và bắt đầu ở trạng thái đã đăng nhập.

Đây mới là thay đổi **thật sự giúp test chạy nhanh hơn**. Tham khảo: <https://playwright.dev/docs/auth>

---

## Tóm tắt nhanh

| Chủ đề | Ghi nhớ một câu |
|---|---|
| Kế thừa | Chỉ `extends` khi câu "A **là một** B" hợp lý |
| BasePage | Gọn, chỉ chứa thứ mọi trang đều cần |
| `page` và page object | Một tab, nhiều "remote"; trang nào đang hiển thị thì dùng page object của trang đó |
| Locator | "Lười": chỉ tìm phần tử khi thao tác hoặc assert |
| Fixture | Phải nhắc tên trong tham số thì fixture mới chạy |
| Chờ trang | Ưu tiên auto-wait và `expect(...)`, hạn chế `waitForLoadState` |
| Regex | `\` để escape ký tự đặc biệt (`\/`, `\.`), `$` để chốt cuối chuỗi |
| `tsc --noEmit` | Không in gì nghĩa là pass; cần cài `typescript` trước |
| TypeScript 7 | Bỏ `baseUrl`, `paths` phải bắt đầu bằng `./` |
