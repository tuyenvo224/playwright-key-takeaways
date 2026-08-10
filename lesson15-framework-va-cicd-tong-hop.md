# Framework Automation Test & CI/CD - Tổng Hợp Toàn Bộ Kiến Thức

> Tổng hợp từ buổi trao đổi trong session này. Toàn bộ ví dụ code lấy từ bài nộp thực tế tại
> [tests/students-submission/tuyen-vo/](tests/students-submission/tuyen-vo/) — không dùng ví dụ ngoài.

---

## Mục lục

1. [Framework Automation Test là gì?](#1-framework-automation-test-là-gì)
2. [Framework giúp ích gì? (đối chiếu code thật)](#2-framework-giúp-ích-gì-đối-chiếu-code-thật)
3. [Page Object Model (POM) - qua ví dụ thực tế](#3-page-object-model-pom---qua-ví-dụ-thực-tế)
4. [Fixtures - qua ví dụ thực tế](#4-fixtures---qua-ví-dụ-thực-tế)
5. [API Test & API Page Object - qua ví dụ thực tế](#5-api-test--api-page-object---qua-ví-dụ-thực-tế)
6. [Testing Pyramid: E2E và các loại test khác](#6-testing-pyramid-e2e-và-các-loại-test-khác)
7. [Visual Regression Test - qua ví dụ thực tế](#7-visual-regression-test---qua-ví-dụ-thực-tế)
8. [Data-Driven Testing - nhận diện cơ hội cải tiến từ code thật](#8-data-driven-testing---nhận-diện-cơ-hội-cải-tiến-từ-code-thật)
9. [CI (Continuous Integration) là gì?](#9-ci-continuous-integration-là-gì)
10. [CD (Continuous Delivery) là gì?](#10-cd-continuous-delivery-là-gì)
11. [Pipeline CI/CD thực tế chạy tuần tự như thế nào?](#11-pipeline-cicd-thực-tế-chạy-tuần-tự-như-thế-nào)
12. [Ai viết loại test nào? Dev vs QA/QC](#12-ai-viết-loại-test-nào-dev-vs-qaqc)
13. [Khi nào nên viết automation test cho tính năng mới?](#13-khi-nào-nên-viết-automation-test-cho-tính-năng-mới)
14. [Tổng kết toàn bộ kiến thức bằng sơ đồ](#14-tổng-kết-toàn-bộ-kiến-thức-bằng-sơ-đồ)

---

## 1. Framework Automation Test là gì?

**Automation Test Framework** là bộ khung gồm quy tắc, cấu trúc thư mục, thư viện và công cụ được tổ chức sẵn để viết, chạy và quản lý các bài kiểm thử tự động một cách hiệu quả, nhất quán và dễ bảo trì.

Phân biệt 2 khái niệm hay nhầm:

| Khái niệm | Định nghĩa |
|---|---|
| **Automation Test** | Dùng code để tự động hoá việc kiểm thử phần mềm (thay vì test tay) |
| **Automation Test Framework** | "Bộ xương" tổ chức các automation test đó — quy ước đặt tên, cấu trúc thư mục, cách quản lý dữ liệu, cách báo cáo kết quả |

### Ví dụ ẩn dụ

```
Automation Test Framework =
    Bản thiết kế (kiến trúc thư mục)
  + Móng nhà (config, base setup)
  + Quy tắc xây dựng (design pattern như POM)
  + Vật liệu chuẩn bị sẵn (fixtures, API page object, test data)

→ Chỉ cần tập trung "xây từng căn phòng" (viết test case),
  thay vì mỗi lần đều làm lại từ đầu.
```

---

## 2. Framework giúp ích gì? (đối chiếu code thật)

### 2.1 Tái sử dụng code (Reusability)

So sánh 2 file thật của bạn cho cùng 1 việc — login vào trang WordPress:

**Không dùng POM** — [lesson-08/authentication.spec.ts](tests/students-submission/tuyen-vo/lesson-08/authentication.spec.ts):

```typescript
test("@AUTH_002:Login success", async ({ page }) => {
    const userName = "betterbytes.academy.admin";
    const psw = "StrongPass@BetterBytesAcademy";

    await page.locator("//input[@id='user_login']").fill(userName);
    await expect(page.locator("//input[@id='user_login']")).toHaveValue(userName);

    await page.locator("//input[@id='user_pass']").fill(psw);
    await expect(page.locator("//input[@id='user_pass']")).toHaveValue(psw);

    await page.locator("//input[@id='wp-submit']").click();
    await expect(page).toHaveURL("https://pw-practice-dev.playwrightvn.com/wp-admin/");
    await expect(page.locator("//div[@class='wrap']/child::h1")).toHaveText("Dashboard");
});
```

**Có dùng POM** — [lesson-10/05-with-pom.spec.ts](tests/students-submission/tuyen-vo/lesson-10/05-with-pom.spec.ts):

```typescript
test('Login success', async ({ page }) => {
    const loginPage = new MyLoginPage(page);

    await loginPage.page.goto("https://pw-practice-dev.playwrightvn.com/wp-admin");
    await loginPage.fillUserName('betterbytes.academy.admin');
    await loginPage.fillPassword('StrongPass@BetterBytesAcademy');
    await loginPage.clickLogin();
    await loginPage.verifySucces('Dashboard');
});
```

→ Cùng làm 1 việc, nhưng bản dùng POM **ngắn gọn hơn, đọc như tiếng Anh tự nhiên**, và toàn bộ locator XPath được giấu bên trong class — không lặp lại ở từng file test.

### 2.2 Dễ bảo trì (Maintainability)

Nếu ID của input password đổi từ `user_pass` sang tên khác:

- **Cách không có POM**: phải tìm và sửa ở mọi file test có XPath `//input[@id='user_pass']` (cả [authentication.spec.ts](tests/students-submission/tuyen-vo/lesson-08/authentication.spec.ts) lẫn các file khác dùng lại XPath này).
- **Cách có POM**: chỉ cần sửa **1 dòng duy nhất** trong [lesson-10/pom/03-pom.ts](tests/students-submission/tuyen-vo/lesson-10/pom/03-pom.ts):

```typescript
passwordXpath: string = "//input[@id='user_pass']";   // chỉ sửa ở đây
```

Mọi test dùng `MyLoginPage` tự động được cập nhật theo.

---

## 3. Page Object Model (POM) - qua ví dụ thực tế

### 3.1 Khái niệm

POM tách **locator + hành động UI** ra khỏi **logic test (assertion, flow)**. Mỗi trang/màn hình được đại diện bởi 1 class.

### 3.2 Toàn bộ file POM thật — [lesson-10/pom/03-pom.ts](tests/students-submission/tuyen-vo/lesson-10/pom/03-pom.ts)

```typescript
import { expect, Page } from '@playwright/test';

export class MyLoginPage {
    page: Page;
    logoXpath: string = "//img[@class='logo']";
    usernameXpath: string = "//input[@id='user_login']";
    passwordXpath: string = "//input[@id='user_pass']";
    rememberMeXpeth: string = "//input[@id='rememberme']";
    loginXpath: string = "//input[@id='wp-submit']";
    headingXpath: string = "//div[@class='wrap']/child::h1";
    errorXpath: string = "//div[@id='login_error']"

    constructor(page: Page) {
        this.page = page;
    }

    async fillUserName(username: string) {
        await this.page.locator(this.usernameXpath).fill(username);
    }

    async fillPassword(password: string) {
        await this.page.locator(this.passwordXpath).fill(password);
    }

    async clickLogin() {
        await this.page.locator(this.loginXpath).click();
    }

    async verifySucces(nameHeading: string) {
        await expect(this.page.locator(this.headingXpath)).toHaveText(nameHeading);
    }

    async verifyFail() {
        await expect(this.page.locator(this.errorXpath)).toBeVisible();
    }
};
```

**Điểm hay trong file POM này:**
- Không chỉ có hành động (`fillUserName`, `clickLogin`) mà còn có cả **hàm assertion đóng gói sẵn** (`verifySucces`, `verifyFail`) — giúp test case gọi 1 dòng thay vì viết `expect(...)` lặp lại.
- Các locator (`usernameXpath`, `passwordXpath`...) được khai báo tập trung ở đầu class — dễ nhìn, dễ sửa.

### 3.3 Cách dùng trong test — [lesson-10/05-with-pom.spec.ts](tests/students-submission/tuyen-vo/lesson-10/05-with-pom.spec.ts)

```typescript
import { test } from '@playwright/test';
import { MyLoginPage } from './pom/03-pom';

test('Login fail', async ({ page }) => {
    const loginPage = new MyLoginPage(page);

    await loginPage.page.goto("https://pw-practice-dev.playwrightvn.com/wp-admin");
    await loginPage.fillUserName('betterbytes.academy.admin');
    await loginPage.fillPassword('StrongPass@BetterBytesAcademy123');
    await loginPage.clickLogin();
    await loginPage.verifyFail();
});
```

---

## 4. Fixtures - qua ví dụ thực tế

Fixture giúp chuẩn bị dữ liệu/môi trường **trước khi** test chạy (VD: mở sẵn trang, verify trang đã load) — tránh lặp code ở đầu mỗi test.

### 4.1 File fixture — [lesson-13/00-fixture.ts](tests/students-submission/tuyen-vo/lesson-13/00-fixture.ts)

```typescript
import { test as base, expect } from '@playwright/test';
import { MaterialPage } from './00-pom';

const test = base.extend<{ materialPage: MaterialPage }>({
    materialPage: async ({ page }, use) => {
        const materialPage = new MaterialPage(page);
        await materialPage.go();
        await expect(materialPage.page.getByText("Tài liệu học automation test")).toBeVisible();

        await use(materialPage);   // <-- test chạy tại đây

        console.log('Test end');   // chạy sau khi test xong (teardown)
    }
});

export { test };
```

### 4.2 Dùng fixture trong test — [lesson-13/02-fixture.spec.ts](tests/students-submission/tuyen-vo/lesson-13/02-fixture.spec.ts)

```typescript
import { expect } from '@playwright/test';
import { test } from './00-fixture'

test("Test 1: Registration Page", async ({ materialPage }) => {
    await materialPage.page.getByText("Bài học 1: Register Page (có đủ các element)").click();
    await expect(materialPage.page.getByText("User Registration")).toBeVisible();
});

test("Test 2: Product Page", async ({ materialPage }) => {
    await materialPage.page.getByText("Bài học 2: Product page").click();
    await expect(materialPage.page.getByText("Simple E-commerce")).toBeVisible();
});
```

**Điểm quan trọng:** mỗi test không cần tự viết `goto()` + verify trang đã load — fixture `materialPage` đã tự động làm sẵn 2 việc đó trước khi test bắt đầu. Đây chính là lợi ích cốt lõi của fixture: **loại bỏ code lặp ở bước chuẩn bị**.

---

## 5. API Test & API Page Object - qua ví dụ thực tế

### 5.1 API Test đơn giản — [lesson-11/01-simple-get.spec.ts](tests/students-submission/tuyen-vo/lesson-11/01-simple-get.spec.ts)

```typescript
import { expect, test } from '@playwright/test';

test('Get all todo', async ({ request }) => {
    const response = await request.get("https://material.playwrightvn.com/api/todo-app/v1/todos.php");
    const responseJson = await response.json();

    expect(response.status()).toBe(200);
    expect(responseJson.todos.length).toEqual(162);
})
```

→ Đây là **API test** thuần — gọi thẳng API bằng fixture `request` của Playwright, **không cần mở browser, không qua UI** → chạy nhanh hơn nhiều so với E2E test.

### 5.2 API Page Object — [lesson-12/pom/01-user.api.page.ts](tests/students-submission/tuyen-vo/lesson-12/pom/01-user.api.page.ts)

Cũng giống như UI có POM, API test cũng nên đóng gói logic gọi API vào 1 class riêng:

```typescript
import { APIRequestContext, expect } from "@playwright/test";

export class UserApiPage {
    request: APIRequestContext;
    baseUrl: string;
    token: string;

    constructor(request: APIRequestContext) {
        this.request = request;
        this.baseUrl = 'https://material.playwrightvn.com/api/user-management/v1';
        this.token = '';
    }

    async login(email: string, password: string) {
        const loginResponse = await this.request.post(`${this.baseUrl}/login.php`, {
            data: { "email": email, "password": password }
        });
        expect(loginResponse.status()).toBe(200);

        const loginResponsejson = await loginResponse.json();
        this.token = loginResponsejson.data.token;
        return loginResponsejson;
    }

    async createUser(name: string, email: string, password: string, facebook: string, avatar: string, hobbies: string, role: string) {
        const createResponse = await this.request.post(`${this.baseUrl}/users.php`, {
            headers: { authorization: `Bearer ${this.token}` },
            data: { name, email, password, facebook, avatar, hobbies, role }
        });
        expect(createResponse.status()).toBe(201);
        return await createResponse.json();
    }
    // ... getUsersList(), deleteUser() tương tự
};
```

### 5.3 Dùng API Page Object trong test — [lesson-12/02-login-user-api-with-pom.spec.ts](tests/students-submission/tuyen-vo/lesson-12/02-login-user-api-with-pom.spec.ts)

```typescript
import { expect, test } from '@playwright/test';
import { UserApiPage } from './pom/01-user.api.page';

test('Login success', async ({ request }) => {
    const userApiPage = new UserApiPage(request);

    await test.step('Login by admin user', async () => {
        const loginReponseJson = await userApiPage.login('admin@example.com', 'password')
        const token = loginReponseJson.data.token;
        expect(token).toBeTruthy();
    });

    await test.step('Login by normal user', async () => {
        const loginResponseJson = await userApiPage.login('john@example.com', 'password')
        const token = loginResponseJson.data.token;
        expect(token).toBeTruthy();
    });
});
```

→ **POM không chỉ dành cho UI** — nguyên tắc "đóng gói logic vào 1 class, test gọi lại" áp dụng y hệt cho API test.

---

## 6. Testing Pyramid: E2E và các loại test khác

### 6.1 E2E (End-to-End) Test là gì?

**E2E Test** = kiểm thử toàn bộ luồng của ứng dụng, từ đầu đến cuối, giống hệt người dùng thật thao tác — qua UI thật.

Ví dụ trong project: [lesson-08/authentication.spec.ts](tests/students-submission/tuyen-vo/lesson-08/authentication.spec.ts) và [lesson-10/05-with-pom.spec.ts](tests/students-submission/tuyen-vo/lesson-10/05-with-pom.spec.ts) — mở browser thật, gõ input thật, click nút thật, xác nhận điều hướng đúng trang.

### 6.2 Testing Pyramid (kim tự tháp kiểm thử)

```
              /\
             /E2E\          ← Ít nhất, chậm nhất, đắt nhất để viết/maintain
            /------\           nhưng độ tin cậy cao nhất (test thật)
           /Integr- \
          /  ation   \      ← Vừa phải
         /------------\
        /   Unit Test   \   ← Nhiều nhất, nhanh nhất, rẻ nhất
       /------------------\
```

### 6.3 Bảng phân loại theo TẦNG (test level)

| Loại | Kiểm tra gì | Ví dụ thực tế trong project |
|---|---|---|
| **Unit Test** | 1 hàm/1 class riêng lẻ, cô lập | (chưa có trong bài nộp — thường do dev viết ở source code, không nằm trong `tests/`) |
| **API Test** | Gọi thẳng REST API, không qua UI | [lesson-11/01-simple-get.spec.ts](tests/students-submission/tuyen-vo/lesson-11/01-simple-get.spec.ts), [lesson-12/pom/01-user.api.page.ts](tests/students-submission/tuyen-vo/lesson-12/pom/01-user.api.page.ts) |
| **E2E Test** | Toàn bộ luồng qua UI thật | [lesson-08/authentication.spec.ts](tests/students-submission/tuyen-vo/lesson-08/authentication.spec.ts), [lesson-10/05-with-pom.spec.ts](tests/students-submission/tuyen-vo/lesson-10/05-with-pom.spec.ts) |

### 6.4 Bảng phân loại theo MỤC ĐÍCH chuyên biệt (áp dụng ở bất kỳ tầng nào)

| Loại | Mục đích |
|---|---|
| **Visual Regression Test** | So sánh ảnh chụp UI trước/sau, phát hiện thay đổi giao diện ngoài ý muốn — xem mục 7 |
| **Contract Test** | Đảm bảo API giữa 2 service đúng "hợp đồng" đã thoả thuận |
| **Performance/Load Test** | Đo tốc độ, khả năng chịu tải |
| **Accessibility Test** | Kiểm tra thân thiện với screen reader, keyboard |
| **Security Test** | Tìm lỗ hổng bảo mật |
| **Smoke Test** | Tập con nhỏ của E2E, chạy nhanh sau deploy để kiểm tra "hệ thống có sống không" |
| **Regression Test** | Không phải loại code riêng — là **mục đích tái sử dụng** toàn bộ test cũ để đảm bảo không hỏng gì |

### 6.5 Vì sao E2E nằm ở đỉnh kim tự tháp?

- **Chậm**: phải khởi động browser thật (như test trong `lesson-08/`, `lesson-10/` đều `page.goto()` tới URL thật)
- **Dễ vỡ (flaky)**: phụ thuộc network, timing, dữ liệu môi trường
- **Đắt để maintain**: khi UI đổi, phải sửa lại locator/POM
- Nhưng **độ tin cậy cao nhất**: test đúng như trải nghiệm người dùng thật

→ Nguyên tắc: viết nhiều test rẻ (Unit, API — như [lesson-11](tests/students-submission/tuyen-vo/lesson-11/)/[lesson-12](tests/students-submission/tuyen-vo/lesson-12/)), ít E2E hơn (chỉ cover luồng nghiệp vụ quan trọng nhất).

---

## 7. Visual Regression Test - qua ví dụ thực tế

File [lesson-14/02-screenshot.spec.ts](tests/students-submission/tuyen-vo/lesson-14/02-screenshot.spec.ts):

```typescript
import { test, expect } from '@playwright/test';

test('example test', async ({ page }) => {
  await page.goto('https://playwright.dev/docs/intro');
  await expect(page).toHaveScreenshot();
});
```

`toHaveScreenshot()` chụp ảnh trang hiện tại rồi **so sánh pixel-by-pixel** với ảnh baseline đã lưu trước đó (nằm trong thư mục `lesson-14/02-screenshot.spec.ts-snapshots/` với các file như `example-test-1-chromium-win32.png`, `example-test-1-firefox-win32.png`, `example-test-1-webkit-win32.png` — mỗi browser 1 ảnh baseline riêng vì render có thể khác nhau).

→ Nếu UI thay đổi (dù chỉ 1 pixel), test này sẽ **fail** và hiện ảnh diff — giúp phát hiện các thay đổi giao diện ngoài ý muốn mà các assertion thông thường (`toBeVisible()`, `toHaveText()`...) không phát hiện được.

---

## 8. Data-Driven Testing - nhận diện cơ hội cải tiến từ code thật

**Data-Driven Testing** = tách dữ liệu test ra khỏi code để 1 test case chạy được với nhiều bộ dữ liệu khác nhau, không cần viết lại code cho từng case.

Nhìn lại [lesson-08/authentication.spec.ts](tests/students-submission/tuyen-vo/lesson-08/authentication.spec.ts) — 2 test case `@AUTH_001:Login fail` và `@AUTH_002:Login success` có **cấu trúc giống hệt nhau**, chỉ khác giá trị `userName`/`psw`:

```typescript
// Case 1
const userName = "betterbytes.academy.admin123";  // sai
const psw = "StrongPass@BetterBytesAcademy";

// Case 2
const userName = "betterbytes.academy.admin";      // đúng
const psw = "StrongPass@BetterBytesAcademy";
```

Đây chính là dấu hiệu điển hình để áp dụng data-driven — nếu tách phần dữ liệu ra:

```typescript
const loginCases = [
  { userName: "betterbytes.academy.admin123", psw: "StrongPass@BetterBytesAcademy", expected: "fail" },
  { userName: "betterbytes.academy.admin",    psw: "StrongPass@BetterBytesAcademy", expected: "success" },
];

for (const c of loginCases) {
  test(`Login - ${c.expected}`, async ({ page }) => {
    // logic dùng chung, chỉ đổi c.userName / c.psw / c.expected
  });
}
```

→ Thêm 1 case đăng nhập mới (VD: sai cả username lẫn password) chỉ cần thêm 1 dòng trong mảng `loginCases`, không cần copy cả khối test.

---

## 9. CI (Continuous Integration) là gì?

**CI** là quy trình tự động hoá việc tích hợp code từ nhiều người vào 1 repository chính một cách thường xuyên (ít nhất vài lần/ngày). Mỗi lần tích hợp (commit/merge), hệ thống tự động **build** ứng dụng, **chạy test** để xác nhận code mới không làm hỏng code cũ.

CI nhấn mạnh vào việc **phát hiện lỗi sớm** và giữ codebase luôn "healthy".

### CI pass nghĩa là gì? (điểm quan trọng)

> "CI pass" **chỉ có nghĩa là**: code mới không làm hỏng những gì **đã có test từ trước**.
> "CI pass" **KHÔNG có nghĩa là**: tính năng mới đã được kiểm chứng đúng.

Ví dụ: nếu bạn thêm 1 test case mới vào [authentication.spec.ts](tests/students-submission/tuyen-vo/lesson-08/authentication.spec.ts) cho 1 luồng login mới, CI chỉ xác nhận được luồng đó **khi test cho nó đã tồn tại**. Nếu dev build tính năng mới nhưng chưa ai viết automation test cho nó, CI pass không nói lên được gì về tính năng đó.

---

## 10. CD (Continuous Delivery) là gì?

**CD** mở rộng từ CI bằng cách tự động hoá thêm bước **deploy** code lên các môi trường staging hoặc production sau khi test pass. Mục tiêu là đảm bảo codebase luôn ở trạng thái **"deployable"** (sẵn sàng deploy bất cứ lúc nào).

**Khác với Continuous Deployment**: CD thường yêu cầu **phê duyệt thủ công** trước khi deploy lên production, trong khi Continuous Deployment là hoàn toàn tự động, không cần người duyệt.

---

## 11. Pipeline CI/CD thực tế chạy tuần tự như thế nào?

CI và CD **không chạy song song** — chúng là các giai đoạn (stage) nối tiếp nhau trong **1 pipeline duy nhất**:

```
1. Dev push code → CI tự động trigger
        ↓
2. CI chạy: build + test (unit/API/E2E hiện có — regression của tính năng cũ)
        ↓ PASS
3. Tự động deploy lên Staging/QA environment
        ↓
4. QA vào Staging, MANUAL TEST tính năng mới
        ↓ PASS
5. Người có quyền (QA lead / Release manager) bấm "Approve" / "Deploy to Production"
        ↓
6. CD chạy: deploy code lên Production
```

### Vì sao không thể "CI pass là deploy prod luôn"?

Vì CI chỉ đảm bảo **code cũ không hỏng** (regression), chứ **không đảm bảo tính năng MỚI đã đúng** — do tính năng mới thường **chưa có automation test** (xem mục 13). Cổng gác thực sự cho tính năng mới là **bước Manual QA test trên Staging**, không phải CI.

### Công cụ thực hiện "bước tạm dừng chờ duyệt"

| Công cụ | Cơ chế manual gate |
|---|---|
| **GitHub Actions** | `environment` với "required reviewers" |
| **GitLab CI** | Job khai báo `when: manual` |
| **Jenkins** | Bước `input()` trong pipeline script |
| **Azure DevOps** | "Release approvals" |

---

## 12. Ai viết loại test nào? Dev vs QA/QC

Có 3 lớp test với vai trò và thời điểm viết khác nhau — dễ nhầm lẫn nếu gộp chung:

| Loại test | Ai viết | Viết khi nào | Test qua đâu |
|---|---|---|---|
| **Unit test** | Dev | Cùng lúc với code tính năng | Gọi thẳng hàm/class, không qua UI |
| **Manual test** | QA/QC | Ngay khi tính năng vừa xong, trước khi release | Tay, click trực tiếp trên UI |
| **Automation/E2E test** | QA/SDET | **Sau khi** tính năng đã manual test pass, ổn định | Qua UI/API — như các file trong [lesson-08](tests/students-submission/tuyen-vo/lesson-08/), [lesson-10](tests/students-submission/tuyen-vo/lesson-10/), [lesson-11](tests/students-submission/tuyen-vo/lesson-11/) |

**Lý do automation test viết sau khi ổn định**: viết sớm khi tính năng còn đang đổi liên tục sẽ khiến test vỡ liên tục theo UI/flow → tốn công maintain nhiều hơn công sức tiết kiệm được. Do đó, cổng gác thực sự cho tính năng MỚI trước khi lên production là **Manual QA test** (mục 11), chứ không phải automation.

---

## 13. Khi nào nên viết automation test cho tính năng mới?

**Mặc định: KHÔNG nên viết automation E2E ngay** khi tính năng còn đang phát triển/chưa ổn định. Nhưng có các trường hợp ngoại lệ nên viết sớm:

| Trường hợp | Vì sao |
|---|---|
| Tính năng **critical** (tiền, thanh toán, login — như luồng login trong [lesson-08/authentication.spec.ts](tests/students-submission/tuyen-vo/lesson-08/authentication.spec.ts)) | Rủi ro lỗi cao → chấp nhận tốn công viết lại test vài lần còn hơn để lỗi lọt prod |
| Team làm theo **BDD/TDD/ATDD** | Test đóng vai trò đặc tả (spec) sống, viết trước code |
| Spec/design đã chốt rõ, ít khả năng đổi lớn | Viết sớm ít rủi ro vỡ test |
| Test ở tầng **API/Integration** thay vì UI E2E (như [lesson-11](tests/students-submission/tuyen-vo/lesson-11/)/[lesson-12](tests/students-submission/tuyen-vo/lesson-12/)) | API ổn định hơn UI nhiều → viết sớm rẻ hơn |
| Nhiều tính năng khác phụ thuộc vào nó | Cần lưới an toàn regression ngay |

**Nguyên tắc chọn:**

```
Viết automation SỚM khi:  Rủi ro nếu lỗi CAO   x   Khả năng đổi flow THẤP
Chờ ổn định mới viết khi: Rủi ro nếu lỗi THẤP  x   Khả năng đổi flow CAO
```

---

## 14. Tổng kết toàn bộ kiến thức bằng sơ đồ

```
FRAMEWORK AUTOMATION TEST
├── POM (Page Object Model)         → tách locator + action khỏi test logic
│     ví dụ: pom/03-pom.ts + 05-with-pom.spec.ts
├── Fixtures                        → chuẩn bị dữ liệu/môi trường trước test
│     ví dụ: 00-fixture.ts + 02-fixture.spec.ts
├── API Page Object                 → POM áp dụng cho API test
│     ví dụ: 01-user.api.page.ts + 02-login-user-api-with-pom.spec.ts
├── Data-Driven Testing             → tách dữ liệu khỏi code
│     cơ hội áp dụng: authentication.spec.ts (2 case trùng cấu trúc)
└── Visual Regression Test          → so sánh screenshot
      ví dụ: 02-screenshot.spec.ts

TESTING PYRAMID
Unit (rẻ, nhiều) → API/Integration → E2E (đắt, ít, tin cậy nhất)

CI/CD PIPELINE
Push code → CI (build + test cũ) → Deploy Staging
    → QA Manual Test tính năng MỚI → Manual Approval → CD Deploy Production

AI VIẾT TEST GÌ
Dev → Unit test (cùng lúc code)
QA  → Manual test (gate thật cho tính năng mới) + Automation test (sau khi ổn định, làm lưới an toàn regression)
```

---

*Tài liệu tổng hợp toàn bộ nội dung trao đổi trong session, đối chiếu 100% ví dụ code với các file thực tế trong [tests/students-submission/tuyen-vo/](tests/students-submission/tuyen-vo/).*
