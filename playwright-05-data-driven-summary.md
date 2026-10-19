# Data Driven Testing trong Playwright – Tổng hợp kiến thức

> Ghi chú tổng hợp từ quá trình áp dụng data driven cho dự án `playwright-automation-framework`,
> dựa trên các bài thực hành trong `playwright-advanced`.

---

## Mục lục

1. [Data driven là gì](#1-data-driven-là-gì)
2. [Các kỹ thuật trong bài thực hành `playwright-advanced`](#2-các-kỹ-thuật-trong-bài-thực-hành-playwright-advanced)
3. [Chuẩn bị trong dự án](#3-chuẩn-bị-trong-dự-án)
4. [Áp dụng thực tế](#4-áp-dụng-thực-tế)
   - 4.1 [account-permissions.spec.ts](#41-account-permissionsspects)
   - 4.2 [add-user.spec.ts](#42-add-userspects)
   - 4.3 [login.spec.ts](#43-loginspects)
5. [Kiến thức TypeScript & Playwright đi kèm](#5-kiến-thức-typescript--playwright-đi-kèm)
6. [Các lỗi đã gặp và cách hiểu](#6-các-lỗi-đã-gặp-và-cách-hiểu)
7. [Nguyên tắc rút ra](#7-nguyên-tắc-rút-ra)
8. [VS Code Playwright extension: chọn browser](#8-vs-code-playwright-extension-chọn-browser)
9. [Bảng lệnh hay dùng](#9-bảng-lệnh-hay-dùng)

---

## 1. Data driven là gì

**Data driven testing** là cách viết test **một lần**, rồi cho nó **chạy nhiều lần với nhiều bộ dữ liệu khác nhau**. Phần **luồng thao tác** (code) tách riêng khỏi phần **dữ liệu**.

Ví dụ không dùng data driven: 3 test gần giống hệt nhau, chỉ khác URL và chữ mong đợi.

```ts
test("Test 1", async ({ page }) => {
    await page.goto("https://.../01-xpath-register-page.html");
    await expect(page.getByRole("heading")).toHaveText("User Registration");
});
test("Test 2", async ({ page }) => { /* y hệt, chỉ khác URL + text */ });
test("Test 3", async ({ page }) => { /* y hệt, chỉ khác URL + text */ });
```

Khi dùng data driven: viết 1 khuôn test và 1 danh sách dữ liệu.

```ts
const testData = [
    { id: 1, url: "https://.../01-xpath-register-page.html", expect: "User Registration" },
    { id: 2, url: "https://.../03-xpath-todo-list.html",     expect: "To-Do List" },
];

testData.forEach(item => {
    test(`Test ${item.id}`, async ({ page }) => {
        await page.goto(item.url);
        await expect(page.getByRole("heading")).toHaveText(item.expect);
    });
});
```

`forEach` chạy **lúc Playwright nạp file test**. Mỗi phần tử trong mảng sẽ sinh ra **một test riêng**, có tên riêng và kết quả pass/fail riêng trong báo cáo.

### Khi nào NÊN dùng

- Nhiều test có **cùng luồng thao tác**, chỉ khác **dữ liệu đầu vào** và **kết quả mong đợi**.
- Muốn thêm case mới **chỉ bằng cách thêm dữ liệu**, không phải viết thêm code.

### Khi nào KHÔNG nên dùng

- Mỗi test **gọi các hàm khác nhau**, ví dụ mở form Add User bằng 3 đường khác nhau.
- Test có **luồng nhiều bước và mỗi test khác nhau nhiều**, ví dụ test tạo user trùng.
- Chỉ có 1–2 case, và mỗi case kiểm tra **một thứ khác nhau**. Viết thẳng ra dễ đọc hơn.

---

## 2. Các kỹ thuật trong bài thực hành `playwright-advanced`

| # | Kỹ thuật | File mẫu | Ý tưởng |
|---|---|---|---|
| 1 | Mảng dữ liệu viết ngay trong test | `tests/02-test-data-driven.spec.ts` | Mảng nằm trên `forEach`, mỗi phần tử sinh 1 test |
| 2 | Kết hợp data driven và test riêng | `tests/03-test-mix-data-drive.spec.ts` | Case "đặc biệt" thì tách ra viết riêng |
| 3 | Đọc file JSON qua alias `@data` | `tests/lesson-01/03-data-driven-with-json.spec.ts` | `import data from '@data/xxx.json'` rồi `forEach` |
| 4 | Đọc file CSV | `tests/lesson-01/00-readcsv.ts` | Hàm `readCSV()` tự viết, không cần thư viện |
| 5 | Dữ liệu theo môi trường | `src/utils/config.ts`, `config-2.ts`, `config-3.ts` | Dựa vào `process.env.ENV` để chọn file `dev`/`staging`/`production` |

### 2.1. Mảng ngay trong test

```ts
const testData = [ { ... }, { ... } ];
testData.forEach(item => {
    test(`Test ${item.id}: ...`, async ({ page }) => { ... });
});
```

- **Ưu điểm:** đọc test là thấy luôn dữ liệu, không phải mở file khác.
- **Dùng khi:** dữ liệu ít, hoặc dữ liệu phụ thuộc vào lúc chạy (ngẫu nhiên, lấy từ `beforeEach`).

### 2.2. File JSON

```ts
import testDataJson from '@data/01-data.json';

testDataJson.forEach(item => {
    test(`Test heading of ${item.name}`, async ({ page }) => { ... });
});
```

- **Ưu điểm:** tách hẳn dữ liệu ra khỏi code. Người không biết code cũng có thể thêm case.
- **Dùng khi:** dữ liệu **cố định**, không thay đổi giữa các lần chạy.

Có 2 cách đọc JSON:

```ts
// Cách 1: đọc file bằng Node (đọc được đường dẫn động, ví dụ theo môi trường)
import * as fs from 'node:fs';
import * as path from 'node:path';
const raw = fs.readFileSync(path.resolve('src/data/01-data.json'), 'utf-8');
const pages = JSON.parse(raw);

// Cách 2: import trực tiếp (gọn hơn, TypeScript tự hiểu kiểu dữ liệu)
import testJsonData from '@data/01-data.json';
```

Cách 2 cần bật `"resolveJsonModule": true` trong `tsconfig.json`. **Dự án đã bật sẵn.**

### 2.3. File CSV

```ts
export function readCSV(filePath: string) {
    const content = fs.readFileSync(path.resolve(filePath), 'utf-8');
    const [header, ...rows] = content.trim().split('\n');
    const keys = header.split(',');
    return rows.map(row => {
        const values = row.split(',');
        return Object.fromEntries(keys.map((k, i) => [k.trim(), values[i].trim()]));
    });
}
```

- **Dùng khi:** QA soạn dữ liệu bằng Excel hoặc Google Sheets rồi xuất ra CSV.
- **Hạn chế của hàm mẫu:** tách cột bằng `split(',')` đơn giản, nên **sai nếu dữ liệu có dấu phẩy**. Mọi giá trị đọc ra đều là chuỗi, kể cả số và `true/false`.

### 2.4. Dữ liệu theo môi trường

```ts
// src/utils/config-3.ts: gộp dữ liệu chung của môi trường + dữ liệu riêng của case
export function getConfig(configFile: string, caseId: string) {
    const env = process.env.ENV || 'dev';
    const envJsonData = JSON.parse(fs.readFileSync(path.resolve(`src/data/${env}.json`), 'utf-8'));
    const testDataJson = JSON.parse(fs.readFileSync(path.resolve(`src/data/${configFile}-${env}.json`), 'utf-8'))[caseId];
    return { ...envJsonData, ...testDataJson };   // dữ liệu của case ghi đè dữ liệu chung
}
```

- **Dùng khi:** dự án có **nhiều môi trường** (dev, staging, production) với dữ liệu khác nhau.
- Dự án hiện tại chỉ có **một** `BASE_URL`, nên **chưa cần** kỹ thuật này.

### 2.5. Lưu ý khi chép code mẫu

Bài mẫu viết `test.describe("...", async () => { ... })`. Hàm truyền vào `describe` **nên là hàm thường, không có `async`**, như dự án đang làm:

```ts
test.describe("...", () => { ... });   // ✅ đúng
```

---

## 3. Chuẩn bị trong dự án

Dự án `playwright-automation-framework` **đã sẵn sàng** cho data driven:

| Thành phần | Trạng thái |
|---|---|
| `tsconfig.json` có alias `"@data/*": ["./src/data/*"]` | ✅ Có sẵn |
| `tsconfig.json` có `"resolveJsonModule": true` | ✅ Có sẵn |
| Thư mục `src/data/` | Tạo khi cần (tài liệu `02-folder-design.md` đã ghi chú) |
| Thư viện thêm | ❌ Không cần (JSON có sẵn trong Node) |

Cấu trúc sau khi áp dụng:

```
src/
└── data/
    ├── account-permissions.json
    ├── add-user-invalid.json
    ├── add-user-missing-fields.json
    └── login-invalid.json
```

Import trong test:

```ts
import permissionData from '@data/account-permissions.json';
```

---

## 4. Áp dụng thực tế

### 4.1. `account-permissions.spec.ts`

**Bài toán:** ACC_001 (editor) và ACC_002 (subscriber) có **cùng một luồng**: tạo user, logout, login bằng user mới, kiểm tra tab nào được thấy và tab nào bị ẩn. Chỉ khác **role** và **danh sách quyền**.

**Ý tưởng:** trong JSON, mỗi tab là một trường `true` (được thấy) hoặc `false` (bị ẩn), giống như một **bảng phân quyền**.

#### File `src/data/account-permissions.json`

```json
[
    {
        "id": "ACC_001",
        "role": "editor",
        "dashboard": true,
        "posts": true,
        "media": true,
        "pages": true,
        "comments": true,
        "profile": true,
        "tools": true,
        "appearance": false,
        "plugins": false,
        "users": false
    },
    {
        "id": "ACC_002",
        "role": "subscriber",
        "dashboard": true,
        "posts": false,
        "media": false,
        "pages": false,
        "comments": false,
        "profile": true,
        "tools": false,
        "appearance": false,
        "plugins": false,
        "users": false
    }
]
```

`role` dùng chữ thường (`editor`, `subscriber`). `selectOption()` khớp được cả value lẫn label, nên viết thống nhất chữ thường cho gọn.

#### Code test

```ts
import permissionData from '@data/account-permissions.json';

// ... beforeEach, afterEach giữ nguyên ...

permissionData.forEach(item => {
    test(`@${item.id}:Create account with ${item.role} permission`, async ({ page }) => {
        await test.step(`Add new user with ${item.role} permission`, async () => {
            await userPage.clickAddUserButton();
            await userPage.fillUserName(userName);
            await userPage.fillEmail(email);
            await expect(userPage.emailTextboxLoc).toHaveValue(email);
            await userPage.fillFirstName("E101");
            await userPage.fillLastName("Tuyen");
            await userPage.selectRole(item.role);
            await userPage.fillPassword(psw);

            await userPage.submitAddUser();
            await expect(userPage.msgCreateUserLoc).toBeVisible();
        });

        await test.step("Logout & Login with new account", async () => {
            await userPage.logout();

            const loginPage = new LoginPage(page);
            await loginPage.login(userName, psw);

            await expect(userPage.dashboardTabLoc).toBeVisible({ visible: item.dashboard });
            await expect(userPage.postsTabLoc).toBeVisible({ visible: item.posts });
            await expect(userPage.mediaTabLoc).toBeVisible({ visible: item.media });
            await expect(userPage.pagesTabLoc).toBeVisible({ visible: item.pages });
            await expect(userPage.commentsTabLoc).toBeVisible({ visible: item.comments });
            await expect(userPage.profileTabLoc).toBeVisible({ visible: item.profile });
            await expect(userPage.toolsTabLoc).toBeVisible({ visible: item.tools });
            await expect(userPage.appearanceTabLoc).toBeVisible({ visible: item.appearance });
            await expect(userPage.pluginsTabLoc).toBeVisible({ visible: item.plugins });
            await expect(userPage.usersTabLoc).toBeVisible({ visible: item.users });

            await userPage.logout();
        });
    });
});
```

#### Điểm mấu chốt: `toBeVisible({ visible: ... })`

| Viết | Tương đương |
|---|---|
| `toBeVisible({ visible: true })` | `toBeVisible()`: phải **hiển thị** |
| `toBeVisible({ visible: false })` | `toBeHidden()`: phải **bị ẩn** |

Nhờ vậy **một dòng code xử lý được cả 2 trường hợp**, chỉ cần truyền `true/false` từ JSON vào.

#### Ưu điểm của cách này

- Không cần khai báo kiểu (`type`, `interface`, `as`).
- Gọi thẳng `userPage.postsTabLoc`, không truy cập bằng biến (`userPage[tab]`).
- Gõ sai tên trường trong JSON thì `npm run typecheck` **báo lỗi**, vì TypeScript tự đọc kiểu từ JSON.
- Khi fail, báo cáo chỉ **đúng dòng** của tab bị lỗi.
- Thêm role mới (author, contributor) **chỉ cần thêm một dòng trong JSON**.

**Nhược điểm nhỏ:** khi menu có tab mới, phải sửa **cả JSON lẫn test**. Menu WordPress hầu như không đổi nên trường hợp này hiếm.

---

### 4.2. `add-user.spec.ts`

#### Phân nhóm 15 test: không phải test nào cũng nên chuyển

| Nhóm | Các test | Chuyển? | Lý do |
|---|---|---|---|
| A. Mở form Add User | 3 test, bấm theo 3 đường khác nhau | ❌ | Mỗi test gọi hàm khác nhau |
| B. Thiếu trường bắt buộc | Không nhập gì / chỉ username / chỉ email | ✅ | Cùng luồng, khác dữ liệu |
| C. Dữ liệu không hợp lệ | Username `!@#$`, email `tuyen@@gmail.com` | ✅ | Cùng luồng, khác dữ liệu và lỗi |
| D. Tạo user thành công | 4 test | ⏸ Để sau | Dữ liệu ngẫu nhiên và phải dọn dẹp user |
| E. Trùng username/email | 3 test | ❌ | Luồng 2 bước, mỗi test khác nhau nhiều |

#### Nhóm C: dữ liệu không hợp lệ

**File `src/data/add-user-invalid.json`:**

```json
[
    { "name": "invalid username", "userName": "!@#$", "email": "tuyen-invalid-01@gmail.com", "errorMessage": "This username is invalid because it uses illegal characters" },
    { "name": "invalid email", "userName": "tuyen-invalid-02", "email": "tuyen@@gmail.com", "errorMessage": "The email address is not correct" }
]
```

**Code test** (thay cho 2 test invalid cũ):

```ts
import invalidUserData from '@data/add-user-invalid.json';

invalidUserData.forEach(item => {
    test(`Add new user with ${item.name}`, async ({ page }) => {
        await test.step(`Add new user with ${item.name}`, async () => {
            await userPage.clickUserTab();
            await userPage.clickAddUserButton();

            await userPage.fillUserName(item.userName);
            await userPage.fillEmail(item.email);

            await userPage.submitAddUser();
            await expect(page.getByText(item.errorMessage)).toBeVisible();
            await expect(page).toHaveURL(/user-new\.php$/);
        });
    });
});
```

**Giải thích:**
- `errorMessage` chỉ cần **một đoạn** của thông báo lỗi. `getByText` mặc định tìm theo kiểu "có chứa", không phân biệt hoa thường, nên không cần chép cả câu hay khoảng trắng thừa.
- Username và email **cố định** vẫn an toàn, vì các case này **không tạo ra user**.
- Không có trường `id`, nên tên test **giữ nguyên như cũ**.

#### Nhóm B: thiếu trường bắt buộc

**Mẹo:** dùng chuỗi rỗng `""` để thể hiện "không nhập". Điền `""` vào ô trống thì ô vẫn trống, kết quả giống hệt không điền.

**File `src/data/add-user-missing-fields.json`:**

```json
[
    { "name": "without userName & email", "userName": "", "email": "" },
    { "name": "with userName only", "userName": "tuyen-missing-01", "email": "" },
    { "name": "with email only", "userName": "", "email": "tuyen-missing-02@gmail.com" }
]
```

**Code test** (thay cho 3 test thiếu trường cũ):

```ts
import missingFieldData from '@data/add-user-missing-fields.json';

missingFieldData.forEach(item => {
    test(`Add new user ${item.name}`, async ({ page }) => {
        await test.step(`Add new user ${item.name}`, async () => {
            await userPage.clickUserTab();
            await userPage.clickAddUserButton();

            await userPage.fillUserName(item.userName);
            await userPage.fillEmail(item.email);

            await userPage.submitAddUser();
            await expect(page).toHaveURL(/user-new\.php$/);
        });
    });
});
```

Code không có `if` và không có cờ `true/false`, đọc từ trên xuống là hiểu. Nhóm B và nhóm C dùng **cùng một kỹ thuật**.

#### Lưu ý

- `beforeEach` và `afterEach` **giữ nguyên**. Các test này không gán `createdUserName`, nên `afterEach` tự bỏ qua bước dọn dẹp.
- Khi thêm case invalid mới, **thêm từng case một và chạy thử**. Nếu WordPress lại chấp nhận dữ liệu đó, user sẽ được tạo thật mà **không bị xóa**, và lần chạy sau sẽ lỗi trùng.
- **Username có khoảng trắng là hợp lệ** trong WordPress. Đừng dùng nó làm case "không hợp lệ".
- Sau khi sửa, `--list` vẫn phải ra **đúng 15 test** với tên như cũ.

---

### 4.3. `login.spec.ts`

| Test | Chuyển? |
|---|---|
| `@AUTH_001:Login fail` | ✅ Mở rộng thành nhiều bộ tài khoản sai |
| `@AUTH_002:Login success` | ❌ Giữ nguyên, vì tài khoản đúng nằm trong `.env` |

#### File `src/data/login-invalid.json`

```json
[
    {
        "id": "AUTH_001",
        "name": "username not registered",
        "userName": "test123456",
        "password": "test111",
        "errorMessage": "is not registered on this site"
    },
    {
        "id": "AUTH_003",
        "name": "another username not registered",
        "userName": "tuyen-not-exist-999",
        "password": "abc123",
        "errorMessage": "is not registered on this site"
    },
    {
        "id": "AUTH_004",
        "name": "email not registered",
        "userName": "tuyen-not-exist-999@gmail.com",
        "password": "abc123",
        "errorMessage": "Unknown email address"
    }
]
```

- Có **giữ `id`**, vì các test trong file này đã dùng mã `@AUTH_xxx`. Mã `AUTH_002` đã thuộc về test login thành công, nên case mới đánh số tiếp từ `AUTH_003`.
- Lỗi đầy đủ là `Error: The username test123456 is not registered on this site...`, có chứa username. Lấy đoạn `is not registered on this site` thì **dùng chung được cho mọi username**.
- Khi ô username có dấu `@`, WordPress coi đó là **email** và báo lỗi khác. Nếu case này fail, mở báo cáo xem câu lỗi thật rồi sửa `errorMessage`.

#### Code test (thay cho `@AUTH_001` cũ)

```ts
import invalidLoginData from '@data/login-invalid.json';

invalidLoginData.forEach(item => {
    test(`@${item.id}:Login fail - ${item.name}`, async ({ page }) => {
        await test.step("Input incorrect user name - password", async () => {
            await loginPage.fillUserNameLogin(item.userName);
            await expect(loginPage.userNameLogin).toHaveValue(item.userName);

            await loginPage.fillPasswordLogin(item.password);
            await expect(loginPage.passwordLogin).toHaveValue(item.password);
        });

        await test.step("Click Login button", async () => {
            await loginPage.clickLoginButton();

            await expect(page.getByText(item.errorMessage)).toBeVisible();
        });
    });
});
```

Hàm cũ `loginPage.getErrorMessage(userName)` chỉ tạo được câu lỗi "username không tồn tại", không dùng cho case email được. Vì vậy đổi sang `page.getByText(item.errorMessage)`.

#### Mở rộng: để trống username hoặc password

**Kiểm tra tay trước**: mở trang login, để trống một ô rồi bấm Log In.

| Thấy gì | Xử lý |
|---|---|
| WordPress báo **lỗi đỏ trên trang** (`The username field is empty`) | Thêm dòng vào `login-invalid.json` với `"userName": ""`, không sửa code |
| Trình duyệt hiện **bong bóng "Please fill out this field"** và form không gửi đi | Không có chữ lỗi trên trang để `getByText` tìm, nên **viết test riêng** như bên dưới |

**Test riêng cho trường hợp bong bóng của trình duyệt:**

```ts
test("@AUTH_005:Login fail - empty username", async ({ page }) => {
    await test.step("Input password only", async () => {
        await loginPage.fillPasswordLogin("abc123");
    });

    await test.step("Click Login button", async () => {
        await loginPage.clickLoginButton();

        // Ô Username có thông báo lỗi của trình duyệt -> form bị chặn, không gửi đi
        await expect(loginPage.userNameLogin).not.toHaveJSProperty('validationMessage', '');
    });
});

test("@AUTH_006:Login fail - empty password", async ({ page }) => {
    await test.step("Input username only", async () => {
        await loginPage.fillUserNameLogin("test123456");
    });

    await test.step("Click Login button", async () => {
        await loginPage.clickLoginButton();

        // Ô Password có thông báo lỗi của trình duyệt -> form bị chặn, không gửi đi
        await expect(loginPage.passwordLogin).not.toHaveJSProperty('validationMessage', '');
    });
});
```

Giải thích chi tiết `validationMessage` ở [mục 5.6](#56-tohavejsproperty-và-validationmessage).

**Username đúng nhưng sai password:** username đúng nằm trong `.env`, nên viết test riêng giống `AUTH_002`, dùng `process.env.ADMIN_USERNAME!` kèm một password sai.

---

## 5. Kiến thức TypeScript & Playwright đi kèm

### 5.1. Template string (chuỗi có biến)

Dùng **dấu backtick** `` ` `` (phím cạnh số 1, dưới Esc), **không phải** dấu nháy `'`:

```ts
const role = "editor";
`Create account with ${role} permission`   // -> "Create account with editor permission"
```

Dùng để tạo **tên test khác nhau** cho mỗi dòng dữ liệu.

### 5.2. Type assertion `as` (ép kiểu)

```ts
const permissionCases = permissionData as PermissionCase[];
```

Câu này có nghĩa: "TypeScript ơi, hãy **coi** dữ liệu này là mảng `PermissionCase`".

- **Chỉ có tác dụng lúc viết code**, khi chạy test thì **không kiểm tra gì**.
- Nếu dữ liệu thật sai, ví dụ gõ nhầm trong JSON, `as` **che mất lỗi**. Typecheck không báo, chỉ khi chạy test mới hỏng.
- Cách đơn giản ở mục 4.1 **không cần `as`**, nên tránh được rủi ro này.

### 5.3. Lỗi kiểu khác lỗi khi chạy test

| | Editor / `npm run typecheck` | `npx playwright test` |
|---|---|---|
| Ai kiểm tra | TypeScript compiler | Playwright |
| Kiểm tra gì | **Kiểu dữ liệu** | **Không kiểm tra kiểu**. Chỉ xóa phần khai báo kiểu rồi chạy code |
| Khi có lỗi kiểu | ❌ Gạch đỏ, tab Problems có số | ✅ Test vẫn có thể pass |

Vì vậy: **lỗi kiểu không làm test fail**, rất dễ bị bỏ qua. Hãy luôn chạy `npm run typecheck` trước khi commit hoặc push, hoặc thêm bước này vào CI.

### 5.4. Câu thông báo tự viết trong `expect`

`expect` nhận thêm **tham số thứ 2 (không bắt buộc)** là câu thông báo, sẽ hiện khi assertion fail:

```ts
await expect(locator, 'câu thông báo').toBeVisible();
await expect(userPage.postsTabLoc, `Tab Posts phải hiển thị với role ${role}`).toBeVisible();
```

Báo cáo khi fail:

```
Error: Tab Posts phải hiển thị với role editor

expect(locator).toBeVisible() failed
```

**Lưu ý:** câu thông báo **chỉ hiện khi assertion chạy xong và cho kết quả sai**. Nếu lỗi xảy ra **trước khi kiểm tra**, ví dụ truyền `undefined` vào `expect`, câu này **không hiện** (xem [mục 6.2](#62-lỗi-tobevisible--undefined-mà-không-in-ra-tên-tab)).

### 5.5. `expect.soft`: kiểm tra hết rồi mới báo

| | Gặp lỗi đầu tiên thì... |
|---|---|
| `expect(...)` | **Dừng test ngay** |
| `expect.soft(...)` | **Ghi nhận lỗi, chạy tiếp**. Cuối test vẫn bị đánh fail và liệt kê **tất cả** lỗi |

```ts
await expect.soft(userPage.postsTabLoc).toBeVisible({ visible: item.posts });
await expect.soft(userPage.mediaTabLoc).toBeVisible({ visible: item.media });
// ... nếu cả Posts và Media đều sai, báo cáo hiện cả 2 lỗi
```

Hữu ích khi kiểm tra **nhiều thứ độc lập cùng lúc**, như danh sách quyền.

### 5.6. `toHaveJSProperty` và `validationMessage`

Khi một ô `required` bị để trống, trình duyệt hiện bong bóng "Please fill out this field". Lúc đó ô input có thuộc tính ẩn **`validationMessage`** mang nội dung lỗi:

| Trạng thái ô | `validationMessage` |
|---|---|
| Hợp lệ | `""` (chuỗi rỗng) |
| Bắt buộc mà để trống | `"Please fill out this field."` (hoặc câu tương tự) |

```ts
await expect(loginPage.userNameLogin).not.toHaveJSProperty('validationMessage', '');
```

Đọc là: "ô Username **không** có `validationMessage` rỗng", tức là **ô đang báo lỗi**.

**Vì sao không kiểm tra đúng câu chữ?** Câu này khác nhau theo trình duyệt và ngôn ngữ:

| Browser | Câu thông báo |
|---|---|
| Chrome, Firefox | `Please fill out this field.` |
| Safari (webkit) | `Fill out this field` |
| Máy cài tiếng Việt | Câu tiếng Việt |

Dự án chạy cả 3 browser, nên chỉ kiểm tra **"có lỗi"** (khác rỗng).

### 5.7. `getByText` tìm theo kiểu "có chứa"

```ts
page.getByText("is not registered on this site")
```

- Mặc định tìm phần tử có **chứa** đoạn chữ, **không phân biệt hoa thường**, bỏ qua khoảng trắng thừa.
- Muốn khớp **chính xác cả câu**: `page.getByText("...", { exact: true })`.

---

## 6. Các lỗi đã gặp và cách hiểu

### 6.1. Editor báo lỗi đỏ mà test vẫn "2 passed"

**Bối cảnh:** lúc đầu dùng cách phức tạp: JSON chứa **tên locator** (`"postsTabLoc"`), khai báo `type MenuTab = 'dashboardTabLoc' | ...`, và truy cập bằng `userPage[tab]`.

**Chuyện xảy ra:** sửa trong `type MenuTab` thành `'dashboardTabLoc1'`, nhưng JSON vẫn đúng `"dashboardTabLoc"`.
- Editor: ❌ `Property 'dashboardTabLoc1' does not exist on type 'UserPage'`
- Chạy test: ✅ 2 passed

**Giải thích:** khi chạy, Playwright **xóa bỏ toàn bộ khai báo kiểu**. Code thật chỉ đọc dữ liệu từ JSON, mà JSON đúng, nên test pass. Xem [mục 5.3](#53-lỗi-kiểu-khác-lỗi-khi-chạy-test).

### 6.2. Lỗi `toBeVisible ... undefined` mà không in ra tên tab

**Chuyện xảy ra:** lần này sửa sai **trong JSON** thành `"dashboardTabLoc1"`. Kết quả:

```
Error: toBeVisible can be only used with Locator object, was called with undefined
```

Câu thông báo tự viết `${tab} phải hiển thị...` **không hiện ra**.

**Giải thích:**
1. `userPage["dashboardTabLoc1"]` không tồn tại, nên trả về `undefined`.
2. Trước khi kiểm tra, `toBeVisible()` xem tham số có phải `Locator` không. Gặp `undefined` thì nó **ném lỗi ngay**.
3. Đây là lỗi **dùng sai**, không phải **assertion fail**, nên câu thông báo tự viết không được gắn vào.

### 6.3. Lỗi `Timeout 10000ms exceeded while waiting on the predicate` ở `afterEach`

**Đây là hệ quả của lỗi 6.2:**
1. Test dừng giữa chừng, bỏ qua bước `logout()`, nên trình duyệt **vẫn đăng nhập** bằng user editor.
2. `afterEach` mở trang để đăng nhập admin, nhưng không thấy ô Username vì đang ở trong dashboard.
3. Hàm `login()` chờ hết 10 giây trong `toPass` rồi báo timeout.

**Bài học:** khi thấy nhiều lỗi cùng lúc, **đọc lỗi đầu tiên trước**. Các lỗi sau thường chỉ là hệ quả.

### 6.4. Kết luận: vì sao bỏ cách phức tạp

Cả 3 lỗi trên đều do cách làm **dùng tên locator dạng chuỗi trong JSON** kèm `type`, `as`, `userPage[tab]`. Chuyển sang cách **`true/false` + `toBeVisible({ visible })`** ([mục 4.1](#41-account-permissionsspects)) thì:
- Không còn `type`, `as`, `userPage[tab]`.
- Gõ sai tên trường trong JSON thì typecheck báo lỗi.
- Fail ở tab nào, báo cáo chỉ đúng dòng đó.

---

## 7. Nguyên tắc rút ra

1. **Ưu tiên cách đơn giản, dễ đọc.** Khi danh sách ngắn và ít thay đổi (như 10 tab menu), viết thẳng từng dòng tốt hơn vòng lặp kèm kỹ thuật TypeScript phức tạp.
2. **Dữ liệu cố định để trong JSON, dữ liệu ngẫu nhiên tạo trong test.**

   | Loại dữ liệu | Để ở đâu | Ví dụ |
   |---|---|---|
   | Cố định | File JSON trong `src/data/` | Username sai, email sai, thông báo lỗi, bảng phân quyền |
   | Ngẫu nhiên hoặc phụ thuộc lúc chạy | Tạo trong `beforeEach` hoặc trong test | `randomUserName()`, `randomEmail()` |

3. **Không đưa mật khẩu thật vào JSON.** JSON được push lên git, còn tài khoản thật để trong `.env`.
4. **Tên test phải khác nhau.** Playwright báo lỗi nếu trùng tên. Đưa `id` hoặc `name` của dữ liệu vào tên test bằng template string.
5. **Chỉ chuyển sang data driven khi các test cùng luồng, khác dữ liệu.** Không cố ép mọi test vào data driven.
6. **Chạy `npm run typecheck` trước khi commit**, vì lỗi kiểu không làm test fail.
7. **Dùng `--list` để kiểm tra số lượng và tên test** trước khi chạy thật. Sau khi refactor, số test phải không đổi, trừ khi bạn chủ ý thêm case.
8. **Case mới chưa chắc chắn thì thêm từng cái một và chạy thử**, nhất là case có thể tạo dữ liệu thật trên hệ thống.
9. **Chạy thử trên 1 browser trước** (`--project=chromium`). Config có 3 browser, nên mỗi dòng dữ liệu sinh ra 3 test.

---

## 8. VS Code Playwright extension: chọn browser

**Hiện tượng:** bấm nút Run từng test trong VS Code thì chỉ chạy trên Chrome, và không thấy chỗ chọn browser.

**Nguyên nhân:**
1. Extension **mặc định chỉ bật project đầu tiên** trong `playwright.config.ts`. Mảng `projects` có thứ tự `chromium`, `firefox`, `webkit`, nên chỉ `chromium` được bật.
2. Workspace mở **nhiều dự án**, mỗi dự án có một config. Mục **CONFIGS** có thể đang chọn config của dự án khác (ví dụ `ai-for-automation-test-lesson5`), nên không thấy danh sách browser của dự án này.

**Cách chọn browser:**
1. Ở mục **CONFIGS**, chọn `playwright-automation-framework\playwright.config.ts`.
2. Tìm phần **PROJECTS** và tick `chromium` / `firefox` / `webkit`. Nếu không thấy, bấm **bánh răng ⚙** cạnh CONFIGS.
3. Hoặc: trong Test Explorer, bấm **mũi tên nhỏ cạnh nút Run**, chọn **Select Default Profile**.

**Lưu ý:**
- Cài đặt này **chỉ áp dụng khi bấm Run trong VS Code**. Chạy `npx playwright test` trong terminal mà không có `--project` thì vẫn chạy **cả 3 browser**.
- Khi bật **Show browser — one worker**, test chạy lần lượt từng cái và có giao diện, nên chọn nhiều browser sẽ lâu hơn. Lúc viết và debug test, chỉ bật `chromium` là đủ.

---

## 9. Bảng lệnh hay dùng

| Mục đích | Lệnh |
|---|---|
| Kiểm tra lỗi kiểu TypeScript | `npm run typecheck` |
| Xem danh sách test được sinh ra (không mở browser) | `npx playwright test <file> --project=chromium --list` |
| Chạy 1 file trên chromium | `npx playwright test <file> --project=chromium` |
| Chạy test có tên chứa một đoạn chữ | `npx playwright test -g "Login fail"` |
| Chạy test theo tag | `npx playwright test -g "@AUTH_001"` |
| Chạy có hiện trình duyệt | `npx playwright test <file> --headed` |
| Mở báo cáo HTML sau khi chạy | `npx playwright show-report` |
| Chạy riêng 1 file `.ts` (không phải test), ví dụ thử đọc JSON | `npx tsx <file.ts>` |

**Quy trình chuẩn sau khi refactor sang data driven:**

```
npm run typecheck
npx playwright test tests/dashboard/users/add-user.spec.ts --project=chromium --list
npx playwright test tests/dashboard/users/add-user.spec.ts --project=chromium
```

1. Typecheck **không có lỗi**.
2. `--list` ra **đúng số test và đúng tên**.
3. Chạy thật, **tất cả pass**.
