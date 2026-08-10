# Lesson 9 - Thực hành Playwright Selector & Debug Flaky Test

> File này tổng hợp phần **thực hành thực tế** trên trang WordPress admin (tạo/xóa user, login/logout...) và các **lỗi thực tế đã gặp + cách fix**.
> Phần lý thuyết đầy đủ về từng loại locator xem tại: `lesson9-playwright-selectors.md` và `lesson9-tong-hop.md`.

---

## Mục lục

1. [Quy trình chọn selector cho 1 phần tử bất kỳ](#1-quy-trình-chọn-selector-cho-1-phần-tử-bất-kỳ)
2. [Cách xem chính xác Role & Name bằng DevTools](#2-cách-xem-chính-xác-role--name-bằng-devtools)
3. [Bẫy thường gặp: role tường minh ghi đè role mặc định](#3-bẫy-thường-gặp-role-tường-minh-ghi-đè-role-mặc-định)
4. [Lỗi Strict Mode Violation và cách scope locator](#4-lỗi-strict-mode-violation-và-cách-scope-locator)
5. [Bảng ví dụ thực hành (WordPress Admin)](#5-bảng-ví-dụ-thực-hành-wordpress-admin)
6. [Debug flaky test do dữ liệu test bị tái sử dụng](#6-debug-flaky-test-do-dữ-liệu-test-bị-tái-sử-dụng)
7. [Chuỗi lỗi domino (cascading failure) khi test timeout](#7-chuỗi-lỗi-domino-cascading-failure-khi-test-timeout)
8. [Checklist tổng hợp](#8-checklist-tổng-hợp)

---

## 1. Quy trình chọn selector cho 1 phần tử bất kỳ

Thứ tự ưu tiên (nhắc lại ngắn gọn — chi tiết xem `lesson9-playwright-selectors.md`):

```
getByRole()  >  getByLabel()  >  getByPlaceholder()  >  getByText()  >  getByTestId()  >  CSS/XPath
```

**Nguyên tắc vàng:** chọn theo cách **người dùng thật sự nhìn thấy/tương tác**, không phải theo cấu trúc DOM/class kỹ thuật — vì DOM/class dễ đổi khi code thay đổi, còn hành vi người dùng thì ổn định hơn.

**Quy trình thực hành khi gặp 1 phần tử mới trong DevTools:**
1. Đọc thẻ HTML: có `role="..."` tường minh không? → nếu có, **role thật = giá trị đó**, không phải role mặc định của thẻ
2. Có `<label for="...">` liên kết không? → ưu tiên `getByLabel()`
3. Có text/`value` hiển thị rõ ràng không? → dùng làm `name`
4. Nếu không chắc → mở tab **Accessibility** trong DevTools để xem chính xác (mục 2 bên dưới)

---

## 2. Cách xem chính xác Role & Name bằng DevTools

**Bước 1:** Mở DevTools (`F12` hoặc chuột phải → Inspect)

**Bước 2:** Bật công cụ chọn phần tử (`Ctrl+Shift+C`) → click vào phần tử trên trang

**Bước 3:** Trong panel bên phải của tab **Elements**, tìm tab **Accessibility** (nếu không thấy, bấm `>>` để mở rộng danh sách tab)

**Bước 4:** Đọc 2 trường quan trọng:
- **Name** → giá trị truyền vào `{ name: '...' }`
- **Role** → giá trị truyền vào tham số đầu của `getByRole(role, ...)`

> Cách này **chính xác tuyệt đối** vì đây đúng là thuật toán mà Playwright dùng để tính accessible name/role — không cần đoán qua HTML.

**Thứ tự tính accessible name (Name) khi không dùng DevTools:**
```
1. aria-labelledby  →  ưu tiên cao nhất
2. aria-label
3. <label for="id"> liên kết
4. text nội dung / value (input submit) / alt (img)
5. title attribute   →  ưu tiên thấp nhất
```

---

## 3. Bẫy thường gặp: role tường minh ghi đè role mặc định

**Đây là bug thực tế đã gặp và gây treo test (hover chờ mãi không ra):**

```html
<a class="ab-item" role="menuitem" href="https://.../wp-admin/profile.php">
  Howdy, <span class="display-name">Better Bytes Academy Admin</span>
</a>
```

Thẻ `<a href="...">` **mặc định có role là `link`**, nhưng ở đây có `role="menuitem"` được khai báo tường minh → **role thực tế bị ghi đè thành `menuitem`**.

```ts
// ❌ SAI — treo mãi vì role thật không phải "link"
await page.getByRole('link', { name: /Howdy/ }).hover();

// ✅ ĐÚNG — dùng đúng role đã bị override
await page.getByRole('menuitem', { name: /Howdy/ }).hover();
await page.getByRole('menuitem', { name: 'Log Out' }).click();
```

**Hậu quả khi dùng sai role:** Playwright không báo lỗi ngay (vì đây không phải "0 hay nhiều phần tử", mà là auto-wait cho locator **không bao giờ khớp**) → cứ retry chờ đến khi hết timeout (mặc định 30s) → test fail với lỗi `Test timeout of 30000ms exceeded`, kèm log `waiting for getByRole('link', { name: /Howdy/ })`.

**Bài học:** luôn kiểm tra HTML có `role="..."` tường minh hay không trước khi giả định role mặc định của thẻ.

---

## 4. Lỗi Strict Mode Violation và cách scope locator

**Lỗi thực tế đã gặp:**
```
Error: strict mode violation: getByRole('link', { name: 'Add User' }) resolved to 2 elements:
  1) <a href="user-new.php">Add User</a> — trong menu điều hướng bên trái
  2) <a class="page-title-action" href="...">Add User</a> — nút cạnh tiêu đề trang
```

Xảy ra khi locator khớp **≥ 2 phần tử cùng lúc** — Playwright không tự đoán, báo lỗi ngay thay vì chọn đại.

**Cách fix — thu hẹp phạm vi (scope) bằng cách bọc trong 1 vùng cha duy nhất:**

```ts
// Scope vào #wpbody-content — chỉ còn đúng 1 "Add User" (nút cạnh tiêu đề)
await page.locator('#wpbody-content').getByRole('link', { name: 'Add User' }).click();

// Hoặc dùng landmark "main" (chuẩn accessibility hơn, không phụ thuộc id kỹ thuật)
await page.getByRole('main').getByRole('link', { name: 'Add User' }).click();
```

**Cách khác để phân biệt 2 phần tử trùng tên:**
```ts
// exact match khi có link "Home" và "Home Page"
await page.getByRole('link', { name: 'Home', exact: true }).click();

// Scope theo dòng (row) trong bảng danh sách — tránh nhầm user này với user khác
const row = page.getByRole('row', { name: userName });
await row.getByRole('link', { name: 'Delete' }).click();
```

**Cách đọc lỗi nhanh:** Playwright luôn liệt kê rõ **danh sách các phần tử bị trùng** kèm code gợi ý (`aka locator(...)`) ngay trong message lỗi — đọc kỹ phần này trước khi tự đoán.

---

## 5. Bảng ví dụ thực hành (WordPress Admin)

| Phần tử | HTML đặc trưng | Selector tốt nhất |
|---|---|---|
| Nút Log In | `<input type="submit" value="Log In">` | `getByRole('button', { name: 'Log In' })` |
| Ô Username có label | `<label for="user_login">` + `<input id="user_login">` | `getByLabel('Username or Email Address')` |
| Thông báo lỗi (không có role) | `<div id="login_error"><p>...</p></div>` | `locator('#login_error')` + `toContainText()` |
| Heading trang | `<h1>Dashboard</h1>` (không có `role`) | `getByRole('heading', { name: 'Dashboard', level: 1 })` |
| Link menu sidebar | `<a href="users.php">` (không có `role`) | `getByRole('link', { name: 'Users', exact: true })` |
| Ô search có label ẩn | `<label class="screen-reader-text" for="...">` | `getByLabel('Search Users:')` — vẫn hoạt động dù label bị ẩn bằng CSS |
| Radio có label | `<input type="radio" id="x">` + `<label for="x">` | `getByRole('radio', { name: '...' }).check()` |
| Dropdown chọn Role | `<select id="role">` | `getByLabel('Role').selectOption('administrator')` — lưu ý `value` khác `label` hiển thị |
| Link "Howdy" / "Log Out" | `role="menuitem"` tường minh | `getByRole('menuitem', { name: ... })` — **không phải `link`** |
| Dòng user trong bảng | `<tr id="user-xxxx">` | `getByRole('row', { name: userName })` để scope trước khi thao tác |
| Thông báo "No users found." | `<td class="colspanchange">` | `getByText('No users found.')` |

---

## 6. Debug flaky test do dữ liệu test bị tái sử dụng

**Triệu chứng:** test lúc pass lúc fail không rõ lý do, đặc biệt hay fail khi có **retry**.

**Nguyên nhân thực tế đã gặp:**
```ts
// ❌ SAI — khai báo NGOÀI test(), chỉ tính 1 LẦN DUY NHẤT khi file được load
test.describe("ACCOUNT-Account", () => {
  const userName = "E101-tuyen-" + Math.floor(Math.random() * 10000);
  const email = "tuyen-" + Math.floor(Math.random() * 10000) + "@gmail.com";
  ...
});
```

**Vì sao gây flaky:**
1. Test attempt đầu tiên tạo user `E101-tuyen-8207` thành công
2. Một bước sau đó fail (do lỗi khác) → Playwright tự động **retry** cả test
3. Vì `userName`/`email` là hằng số cố định, **giá trị bị lặp lại y hệt** ở lần retry
4. WordPress từ chối vì **user đã tồn tại** → form "Add User" bị reset (Email trống, báo lỗi đỏ) → assertion `toBeVisible()` phía sau fail
5. Kết quả: test "lúc pass lúc fail" tùy vào có bị retry hay không

**Cách fix — khai báo biến ở scope `describe` (để dùng chung với `afterEach`), nhưng GÁN giá trị mới bên trong `beforeEach`:**

```ts
test.describe("ACCOUNT-Account", () => {
  let userName: string;
  let psw: string;
  let email: string;

  test.beforeEach("Đã login trang with admin user", async ({ page }) => {
    // Sinh dữ liệu mới ở đây — chạy lại mỗi lần test/retry
    userName = "E101-tuyen-" + Math.floor(Math.random() * 10000);
    psw = "Editor123456!@!@";
    email = "tuyen-" + Math.floor(Math.random() * 10000) + "@gmail.com";

    await page.goto("https://pw-practice-dev.playwrightvn.com/wp-admin");
    await page.getByRole('textbox', { name: 'Username or Email Address' }).fill("betterbytes.academy.admin");
    await page.getByRole('textbox', { name: 'Password' }).fill("StrongPass@BetterBytesAcademy");
    await page.getByRole('button', { name: 'Log In' }).click();
  });

  test.afterEach("Login by admin & delete normal user", async ({ page }) => {
    // Vẫn đọc được userName/email mới nhất vì cùng scope describe
  });

  test("@ACC_001:Create account with editor permission", async ({ page }) => {
    // userName/email đã có giá trị do beforeEach chạy trước
    await page.getByLabel('Username (required)').fill(userName);
    await page.getByLabel('Email (required)').fill(email);
  });
});
```

**Vì sao cách này đúng:**
- `let` ở scope `describe` → mọi hook (`beforeEach`, `afterEach`) và mọi `test()` bên trong đều đọc/ghi chung được
- Playwright **chạy lại `beforeEach` ở mỗi lần retry** → dữ liệu luôn mới, không còn trùng username
- `beforeEach` chạy **trước mỗi test riêng biệt** → các test khác nhau (`@ACC_001`, `@ACC_002`...) không đụng dữ liệu của nhau

**Cách bắt lỗi này sớm hơn** (thay vì để nó trôi xuống bước sau mới fail mơ hồ):
```ts
await page.getByLabel('Email (required)').fill(email);
await expect(page.getByLabel('Email (required)')).toHaveValue(email); // fail ngay tại đây nếu email rỗng
```

---

## 7. Chuỗi lỗi domino (cascading failure) khi test timeout

Khi debug, cần phân biệt **lỗi gốc** và **lỗi hệ quả** — chỉ nên fix đúng lỗi gốc:

```
[Lỗi gốc]
  hover() vào locator sai role → treo mãi (auto-wait không bao giờ thỏa)
        ↓ (đủ 30s)
[Hệ quả 1]
  "Test timeout of 30000ms exceeded" → Playwright hủy ngang test, đóng page/context
        ↓
[Hệ quả 2]
  afterEach() chạy dọn dẹp nhưng page đã đóng
  → Error: "Target page, context or browser has been closed"
```

**Bài học:** khi thấy nhiều lỗi cùng lúc, luôn tìm lỗi xảy ra **sớm nhất theo thời gian** (thường nằm trong `test.step` hoặc dòng code cụ thể, không phải trong hook `afterEach`/`afterAll`) — đó mới là nơi cần fix. Các lỗi ở hook dọn dẹp thường chỉ là nạn nhân của lỗi gốc phía trước.

---

## 8. Checklist tổng hợp

- [ ] Kiểm tra phần tử có `role="..."` tường minh trong HTML không, trước khi giả định role mặc định
- [ ] Dùng tab **Accessibility** trong DevTools để xác nhận chính xác Role/Name khi không chắc
- [ ] Khi gặp `strict mode violation`, đọc kỹ danh sách phần tử trùng Playwright liệt kê, rồi **scope** locator (`.locator('#id').getByRole(...)` hoặc `.getByRole('main').getByRole(...)`)
- [ ] Dữ liệu test ngẫu nhiên (username, email...) phải được **sinh mới trong `beforeEach`**, không đặt cố định ở scope `describe`/module — tránh flaky khi retry
- [ ] Khai báo biến cần dùng chung giữa `test()` và `afterEach` bằng `let` ở scope `describe`, gán giá trị trong `beforeEach`
- [ ] Khi có nhiều lỗi cùng lúc, xác định lỗi xảy ra **sớm nhất** để fix đúng gốc, bỏ qua các lỗi hệ quả ở hook dọn dẹp
- [ ] Với radio/checkbox dùng `.check()` thay vì `.click()`; với `<select>` dùng `selectOption()` và chú ý `value` có thể khác text hiển thị
