# Lesson 10 — Page Object Model (POM)

> Nguồn: PlaywrightVN — Học automation test từ chưa biết gì

---

## Nội dung buổi học

1. TypeScript so sánh với JavaScript
2. TypeScript: kế thừa (class & extends)
3. Page Object Model (POM)

---

## 1. TypeScript vs JavaScript

### TypeScript là gì?

**TypeScript là superset của JavaScript** — tức là mở rộng của JavaScript.

- JavaScript rất "dễ dãi" → sinh ra nhiều lỗi tiềm ẩn
- TypeScript ra đời để "khó tính" hơn → giúp **giảm lỗi** trong lúc viết code

```
JavaScript ──── TypeScript thêm vào: kiểu dữ liệu, class, interface, generic,...
```

### Cách chạy TypeScript

TypeScript **không chạy trực tiếp** — cần biên dịch sang JavaScript trước:

```bash
# Cài TypeScript
npm install -d typescript

# Biên dịch file .ts sang .js
npx tsc <file_path>

# Chạy file .js bằng Node
node <file_path>
```

### Tại sao dùng TypeScript thay JavaScript?

| Lợi ích | Giải thích |
|---|---|
| Có hệ thống kiểu dữ liệu | IDE gợi ý tốt hơn khi hover/autocomplete |
| Phát hiện lỗi sớm | Báo lỗi ngay khi code sai kiểu, không cần chạy mới biết |
| Interface & type alias | Mô tả rõ cấu trúc dữ liệu |
| OOP features | Hỗ trợ class, kế thừa, generic |

**Ví dụ so sánh:**
```typescript
// JavaScript — không biết name kiểu gì
const jsName = "Tuyen"

// TypeScript — rõ ràng là string
const tsName: string = "Tuyen"
```

---

## 2. Define Type — Định nghĩa kiểu dữ liệu

### Các loại kiểu dữ liệu trong TypeScript

```
Any (kiểu cha của tất cả)
├── Built-in types: number, string, boolean, void, null, undefined
└── User-defined types: Arrays, Enums, Classes, Interfaces
```

### Cách định nghĩa kiểu: `type` và `interface`

Định nghĩa kiểu dữ liệu giúp code trở nên **rõ ràng, dễ đọc hơn**.

**Dùng `type`:**
```typescript
type User = {
    name: string;
    age: number;
};

const user1: User = {
    name: "Phong",
    age: 18,
};
```

**Dùng `interface`** (lưu ý: không có dấu `=`):
```typescript
interface User {
    name: string;
    age: number;
}

const user1: User = {
    name: "Phong",
    age: 18,
};
```

**Định nghĩa kiểu cho method:**
```typescript
interface ObjectWithMethod {
    sum(a: number, b: number): number
    //   ↑ param types           ↑ return type
}
```

### Convention đặt tên

- Tên kiểu dữ liệu/class viết theo **PascalCase**: `E101User`, `LoginPage`, `DashboardPage`

### Ví dụ thực tế (từ file `01-define-type.ts`)

```typescript
// Dùng type
type E101User = {
    name: string;
    age: number;
    yearOfExperience: number
};

// Dùng interface
interface Gold {
    loaiVang: string;
    giaMua: number;
    giaBan: number;
    'so luong': number  // key có khoảng trắng thì dùng dấu nháy
};

const gold1: Gold = {
    loaiVang: '9999',
    giaMua: 15,
    giaBan: 16,
    'so luong': 1
};
```

---

## 3. Class & Extends (Kế thừa)

### Class là gì?

**Class** dùng để mô hình hoá một đối tượng, bao gồm:
- **Properties** (thuộc tính): các đặc tính của đối tượng
- **Methods** (phương thức): các hành động mà đối tượng có thể thực hiện

```typescript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    sayHello() {
        console.log(`Hello, my name is ${this.name}`);
    }
}
```

### Tại sao dùng class thay vì object rời?

**Không dùng class (rời rạc, khó quản lý):**
```typescript
// Code rải rác, khó quản lý
const user1 = { name: "An", email: "an@example.com", age: 25 };
const user2 = { name: "Bình", email: "binh@example.com", age: 30 };

// Các hàm khai báo riêng lẻ (lazy declare)
function getUserInfo(user: { name: string; email: string; age: number }): string {
    return `${user.name} (${user.email}) - ${user.age} tuổi`;
}
```
Vấn đề: Code rải rác, dễ tạo object sai cấu trúc, khó tái sử dụng.

**Dùng class (gọn gàng, có cấu trúc):**
```typescript
class User {
    email: string;
    name: string;
    age: number;

    constructor(email: string, name: string, age: number) {
        this.email = email;
        this.name = name;
        this.age = age;
    }

    getInfo(): string {
        return `${this.name} (${this.email}) - ${this.age} tuổi`;
    }

    isAdult(): boolean {
        return this.age >= 18;
    }

    updateEmail(newEmail: string) {
        this.email = newEmail;
    }
}

// Khởi tạo gọn trong 1 dòng
const user1 = new User("an@example.com", "An", 25);
const user2 = new User("binh@example.com", "Bình", 30);
```

### Extends — Kế thừa

**Extends** cho phép class con "thừa hưởng" toàn bộ **thuộc tính và phương thức** từ class cha.

- Hàm **`super()`** = gọi tới constructor của class cha

```typescript
class LoginPage {
    userNameLoc: string;
    passwordLoc: string;

    constructor(userName: string, password: string) {
        this.userNameLoc = userName;
        this.passwordLoc = password;
    }

    fillUserName(userName: string) {
        console.log('Filling username', userName);
    }
}

// DashboardPage kế thừa từ LoginPage
class DashboardPage extends LoginPage {
    headingLoc: string;

    constructor(heading: string, userName: string, password: string) {
        super(userName, password); // gọi constructor của LoginPage
        this.headingLoc = heading;
    }

    clickMenu(menuName: string) {
        console.log('Clicking Menu Name');
    }
}

// DashboardPage có thể dùng cả fillUserName (từ LoginPage) lẫn clickMenu
const dashboard = new DashboardPage("heading", "user", "pass");
dashboard.fillUserName("Minh Phong"); // kế thừa từ LoginPage
dashboard.clickMenu("Settings");      // phương thức riêng của DashboardPage
```

---

## 4. Page Object Model (POM)

### POM là gì?

**POM (Page Object Model)** là một **design pattern** — một cấu trúc code "sạch đẹp, dễ bảo trì" cho automation test.

**Core concept:**
- Mỗi trang web = một **class**
- Class đó có **Properties** (locators của các element) và **Methods** (các action trên trang)

```
POM = class với:
  ├── Properties: các thành phần của trang web (locators)
  └── Methods: các hành động trên trang web (bắt đầu bởi động từ: fill, click, verify...)
```

### Vấn đề khi KHÔNG dùng POM

```typescript
// Tất cả locators và logic test trộn lẫn vào nhau
test('Login test', async ({ page }) => {
    await page.goto('https://example.com/login');
    await page.fill('#username', 'admin');        // locator #username bị lặp
    await page.fill('#password', 'password123');  // locator #password bị lặp
    await page.click('button[type="submit"]');
});

test('Login với sai password', async ({ page }) => {
    await page.goto('https://example.com/login');
    await page.fill('#username', 'admin');        // lặp lại lần nữa
    await page.fill('#password', 'wrongpassword');
    await page.click('button[type="submit"]');
    // ...
});
```

**Vấn đề:**
- Locators bị lặp lại ở nhiều nơi (`#username`, `#password`...)
- Nếu UI thay đổi → phải sửa ở **nhiều** test
- Code dài dòng, khó đọc
- Khó maintain khi có nhiều tests
- Không tái sử dụng được code

### Giải pháp: Dùng POM

**Bước 1: Tạo class POM** (file `pom/03-pom.ts`)

```typescript
import { expect, Page } from '@playwright/test';

export class MyLoginPage {
    page: Page;

    // Properties: tất cả locators tập trung ở một chỗ
    logoXpath: string = "//img[@class='logo']";
    usernameXpath: string = "//input[@id='user_login']";
    passwordXpath: string = "//input[@id='user_pass']";
    loginXpath: string = "//input[@id='wp-submit']";
    headingXpath: string = "//div[@class='wrap']/child::h1";
    errorXpath: string = "//div[@id='login_error']";

    constructor(page: Page) {
        this.page = page;
    }

    // Methods: các hành động
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
}
```

**Bước 2: Dùng class POM trong test** (file `05-with-pom.spec.ts`)

```typescript
import { test } from '@playwright/test';
import { MyLoginPage } from './pom/03-pom';

test('Login success', async ({ page }) => {
    const loginPage = new MyLoginPage(page); // khởi tạo đối tượng

    await test.step("Login page", async () => {
        await loginPage.page.goto("https://pw-practice-dev.playwrightvn.com/wp-admin");
    });

    await test.step("Fill user name", async () => {
        await loginPage.fillUserName('betterbytes.academy.admin');
    });

    await test.step("Fill password", async () => {
        await loginPage.fillPassword('StrongPass@BetterBytesAcademy');
    });

    await test.step("Click Login", async () => {
        await loginPage.clickLogin();
        await loginPage.verifySucces('Dashboard');
    });
});

test('Login fail', async ({ page }) => {
    const loginPage = new MyLoginPage(page);

    await test.step("Login page", async () => {
        await loginPage.page.goto("https://pw-practice-dev.playwrightvn.com/wp-admin");
    });

    await test.step("Fill user name", async () => {
        await loginPage.fillUserName('betterbytes.academy.admin');
    });

    await test.step("Fill password", async () => {
        await loginPage.fillPassword('StrongPass@BetterBytesAcademy123'); // sai pass
    });

    await test.step("Click Login", async () => {
        await loginPage.clickLogin();
        await loginPage.verifyFail();
    });
});
```

### 3 lý do dùng POM

| # | Lý do | Giải thích |
|---|---|---|
| 1 | **Dễ maintain (Bảo trì)** | UI thay đổi → chỉ sửa 1 chỗ trong class POM, không cần sửa từng test |
| 2 | **Code dễ đọc hơn** | Test chỉ còn: `loginPage.fillUserName(...)` — rõ ràng, ngắn gọn |
| 3 | **Tái sử dụng code (Reusability)** | Nhiều test cùng dùng chung `MyLoginPage`, không viết lại |

### Tiêu chuẩn của POM

Lưu ý: **không có 1 chuẩn chung** cho POM. Cách viết phụ thuộc vào:
- Framework đang dùng
- Ngôn ngữ lập trình
- Author/người viết
- Sở thích và kinh nghiệm của team

Cấu trúc cơ bản phổ biến nhất với Playwright:

```typescript
import { Page } from '@playwright/test';

export class LoginPage {
    constructor(private page: Page) {}

    // properties (locators)
    // methods (actions)
}
```

### Cấu trúc file gợi ý

```
project/
├── pom/                    # Thư mục chứa các Page Object
│   ├── LoginPage.ts
│   ├── DashboardPage.ts
│   └── ...
└── tests/                  # Thư mục chứa test specs
    ├── login.spec.ts
    └── ...
```

---

## 5. So sánh: Không POM vs Có POM

| Tiêu chí | Không POM | Có POM |
|---|---|---|
| Locator | Lặp lại ở nhiều test | Tập trung 1 chỗ trong class |
| Khi UI thay đổi | Phải sửa nhiều file test | Chỉ sửa 1 class |
| Độ dài test | Dài, nhiều chi tiết | Ngắn, rõ ý định |
| Tái sử dụng | Không | Có |
| Dễ đọc | Khó | Dễ |

---

## Tóm tắt nhanh (Quick Reference)

```typescript
// 1. Định nghĩa type
type User = { name: string; age: number; }
interface User { name: string; age: number; }

// 2. Tạo class
class LoginPage {
    usernameXpath = "//input[@id='user_login']";
    constructor(private page: Page) {}
    async fillUserName(username: string) {
        await this.page.locator(this.usernameXpath).fill(username);
    }
}

// 3. Kế thừa class
class DashboardPage extends LoginPage {
    constructor(page: Page) {
        super(page); // bắt buộc gọi super()
    }
}

// 4. Dùng trong test
test('...', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.fillUserName('admin');
});
```
