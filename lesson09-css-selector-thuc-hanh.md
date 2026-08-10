# Lesson 9 - Thực Hành: Chuyển XPath → CSS Selector & Bẫy Thường Gặp Khi Test

> Tài liệu này tổng hợp lại toàn bộ các tình huống thực hành chuyển đổi XPath (lấy từ DevTools) sang CSS Selector trên trang WordPress Admin, cùng những lỗi thực tế đã gặp khi viết test Playwright.

---

## Mục lục

1. [Nguyên tắc chung: XPath → CSS](#1-nguyên-tắc-chung-xpath--css)
2. [3 giới hạn của CSS Selector so với XPath](#2-3-giới-hạn-của-css-selector-so-với-xpath)
3. [Ví dụ thực hành từ dễ đến khó](#3-ví-dụ-thực-hành-từ-dễ-đến-khó)
4. [Quy tắc xử lý khi selector bị trùng (nhiều kết quả)](#4-quy-tắc-xử-lý-khi-selector-bị-trùng-nhiều-kết-quả)
5. [Bài toán thực tế: thao tác trên dòng dữ liệu động (dynamic row)](#5-bài-toán-thực-tế-thao-tác-trên-dòng-dữ-liệu-động-dynamic-row)
6. [Case study: bug test "pass giả" (false positive)](#6-case-study-bug-test-pass-giả-false-positive)
7. [Cheat sheet tổng hợp](#7-cheat-sheet-tổng-hợp)

---

## 1. Nguyên tắc chung: XPath → CSS

| XPath dùng để lọc theo... | Có chuyển sang CSS được không? | Cách chuyển |
|---|---|---|
| **Attribute** (`@class`, `@id`, `@href`...) | ✅ Có | Chuyển thẳng: `tag[@class='x']` → `tag.x` hoặc `tag[class='x']` |
| **Quan hệ con trực tiếp** (`/child::tag`) | ✅ Có | `>` : `parent > child` |
| **Nội dung chữ** (`[text()='...']`) | ❌ Không | Phải tìm `id`/`class` của phần tử cha để thay thế |
| **Anh/chị đứng trước** (`/preceding-sibling::tag`) | ❌ Không | Chọn thẳng phần tử đó bằng class/id riêng của nó |

**Quy tắc vàng:** mỗi khi thấy XPath, tách nó thành 2 phần — "nó dựa vào **thuộc tính có sẵn**" (chuyển được) hay "nó dựa vào **nội dung/vị trí tương đối**" (CSS bó tay, phải tìm cách khác).

---

## 2. 3 giới hạn của CSS Selector so với XPath

### a) Không lọc được theo text content

```
XPath:  //h2[text()='Activity']
CSS:    ❌ không có cú pháp tương đương
```

CSS Level 3/4 **không hỗ trợ** chọn phần tử dựa vào nội dung chữ bên trong. Đây là giới hạn thiết kế, không phải do viết sai cú pháp.

**Cách xử lý:** tìm `id`/`class` ổn định của phần tử cha gần nhất.

```html
<div id="dashboard_activity" class="postbox">
  <h2 class="hndle ...">Activity</h2>
```

```css
#dashboard_activity h2.hndle
```

### b) Không có "preceding-sibling" (chỉ đi được về sau, không đi được về trước)

```
XPath:  //img[@class='avatar avatar-26 photo']/preceding-sibling::span
CSS:    ❌ không có tổ hợp nào đi "ngược" được
```

CSS chỉ có 2 tổ hợp sibling, và cả hai đều đi **về phía sau**:
- `A + B` → B ngay sau A
- `A ~ B` → B bất kỳ đứng sau A

Không có `B ~ A` để nói "B đứng trước A".

**Cách xử lý:** đừng cố diễn đạt qua quan hệ sibling — chọn thẳng phần tử đó bằng class/id của chính nó.

```css
.ab-item .display-name   /* thay vì cố tìm quan hệ với img */
```

### c) Class bị lặp lại → chọn nhầm nhiều phần tử

Không phải giới hạn cú pháp, mà là bẫy thực tế: nhiều phần tử khác nhau có thể dùng chung 1 class (VD tất cả menu item admin đều có `.wp-menu-name`, tất cả dòng user trong bảng đều có `.username`). Nếu chỉ chọn theo class, CSS sẽ khớp **tất cả**, không riêng phần tử bạn muốn.

**Cách xử lý:** luôn bọc thêm `id` của phần tử cha để giới hạn phạm vi tìm kiếm (xem mục 4).

---

## 3. Ví dụ thực hành từ dễ đến khó

### Ví dụ 1 — Chọn theo id cha + child combinator

```html
<div id="login_error" class="notice notice-error">
  <p>...</p>
</div>
```
```css
#login_error > p
```

### Ví dụ 2 — Nhiều class cùng lúc trên 1 thẻ (viết liền, không dấu cách)

```html
<div class="notice notice-error">
```
```css
div.notice.notice-error p     /* .notice.notice-error viết liền = cùng 1 thẻ */
```
> Lưu ý: `div.notice .notice-error` (có dấu cách) lại mang nghĩa khác hẳn — nghĩa là 2 thẻ lồng nhau.

### Ví dụ 3 — Class không có id → phải kèm tên thẻ để tránh nhầm

```html
<div class="wrap">
  <h1>Dashboard</h1>
</div>
```
```css
div.wrap > h1
```

### Ví dụ 4 — Attribute selector chuyển thẳng (XPath dùng `@class`)

```
XPath: //h1[@class='wp-heading-inline']
CSS:   h1.wp-heading-inline
```

### Ví dụ 5 — Class chung cho nhiều menu item → phải kèm id để phân biệt

```html
<li id="menu-users">
  <a href="users.php">
    <div class="wp-menu-name">Users</div>   <!-- class này lặp ở MỌI menu item -->
  </a>
</li>
```
```css
#menu-users .wp-menu-name
```

### Ví dụ 6 — Kết hợp attribute + child, nhưng class bị lặp ở mọi dòng bảng

```html
<tr id="user-6669">
  <td class="username column-username has-row-actions column-primary">
    <strong><a>...</a></strong>
  </td>
</tr>
```
```css
#user-6669 td.username > strong
```
> `td.username...` một mình sẽ khớp với **cột username của mọi dòng** trong bảng — bắt buộc kèm `#user-6669` (id thật của user trong DB) để chỉ đúng 1 dòng.

### Ví dụ 7 — preceding-sibling → chọn thẳng bằng class riêng

```html
<a class="ab-item">
  <span class="display-name">Better Bytes Academy Admin</span>
  <img class="avatar avatar-26 photo">
</a>
```
```css
#wp-admin-bar-my-account > a.ab-item > span.display-name
```

---

## 4. Quy tắc xử lý khi selector bị trùng (nhiều kết quả)

Class `display-name` ở ví dụ 7 thực ra xuất hiện **2-3 lần** trên trang (toolbar trên cùng + trong menu dropdown "Howdy"). Nếu chỉ viết `span.display-name` → khớp nhầm sang cả phần tử trong dropdown.

**Cách xử lý chuẩn: leo lên tìm `id` của cha gần nhất mà duy nhất trên trang.**

```html
<li id="wp-admin-bar-my-account">        <!-- vùng 1: toolbar trên cùng -->
  <a class="ab-item"><span class="display-name">...</span></a>
  <div class="ab-sub-wrapper">
    <li id="wp-admin-bar-user-info">     <!-- vùng 2: dropdown, span trùng tên -->
      <a class="ab-item"><span class="display-name">...</span></a>
```

```css
/* Sai: khớp cả 2 vùng */
span.display-name

/* Đúng: chỉ khớp vùng 1 */
#wp-admin-bar-my-account > a.ab-item > span.display-name
```

**Ghi nhớ:** mỗi khi CSS selector trả về nhiều hơn 1 kết quả trong khi chỉ cần 1, đừng cố "vá" selector bằng cách thêm class — hãy tìm id ổn định của khối cha bao quanh đúng phần tử cần lấy.

---

## 5. Bài toán thực tế: thao tác trên dòng dữ liệu động (dynamic row)

**Vấn đề:** muốn hover vào **bất kỳ user nào** để hiện nút Delete rồi xóa — nhưng mỗi user có `id` khác nhau (VD `#user-6669`), không thể hardcode.

**Giải pháp:** tách làm 2 bước — dùng CSS để mô tả **cấu trúc chung** (áp dụng cho mọi dòng), dùng tính năng lọc-theo-text của Playwright (`filter({ hasText })`) để chọn **đúng dòng** theo dữ liệu động. Đây chính là cách thay thế cho việc CSS không lọc được theo text (mục 2a), nhưng lần này xử lý ở tầng automation tool thay vì CSS thuần.

```javascript
async function deleteUser(page, username) {
  // Bước 1: định vị đúng row theo username (Playwright filter, không phải CSS)
  const row = page.locator('#the-list tr').filter({ hasText: username });

  // Bước 2: hover để hiện row-actions (Edit | Delete | ...)
  await row.hover();

  // Bước 3: thao tác TƯƠNG ĐỐI trong phạm vi row đã lọc — CSS cấu trúc chung
  await row.locator('.row-actions .delete a').click();
}

await deleteUser(page, 'E101-tuyen-5121');
```

**Bài học:** với test case cần thao tác theo dữ liệu động (username, order id...), không có "1 CSS selector vạn năng" — luôn là (1) lọc đúng row bằng text, rồi (2) dùng CSS tương đối bên trong row đó.

---

## 6. Case study: bug test "pass giả" (false positive)

### Bối cảnh

Test verify: user quyền **Editor** (không có quyền quản lý user) thì menu "Users" phải bị ẩn.

```javascript
await expect(page.locator("//div[text()='Users']")).toBeHidden();  // PASS ✅ (nhưng sai!)
await expect(page.locator('#menu-users .wp-menu-name')).toBeHidden(); // FAIL ❌ (đúng bản chất)
```

### Nguyên nhân

WordPress **không ẩn hoàn toàn** menu "Users" với role Editor — nó chỉ **đổi label** thành **"Profile"** (link tới trang sửa hồ sơ cá nhân), nhưng vẫn giữ `id="menu-users"` trong HTML.

```html
<!-- Với Admin -->
<li id="menu-users">...<div class="wp-menu-name">Users</div></li>

<!-- Với Editor -->
<li id="menu-users">...<div class="wp-menu-name">Profile</div></li>   <!-- vẫn tồn tại, vẫn visible -->
```

### Vì sao XPath "pass" dù sai bản chất?

- `//div[text()='Users']` tìm div có **text chính xác** là "Users" → **không tìm thấy phần tử nào** (vì giờ text là "Profile")
- `toBeHidden()` trong Playwright coi "**không tìm thấy phần tử**" cũng là thỏa mãn điều kiện hidden → **test pass**, nhưng pass vì lý do sai (không phải vì bị ẩn, mà vì đổi tên nên không khớp XPath text nữa)
- Locator theo `id` (`#menu-users .wp-menu-name`) thì **vẫn tìm thấy** phần tử (đang hiển thị, chỉ đổi text) → fail đúng với thực tế

### Cách sửa đúng

```javascript
// Verify đúng bản chất: menu vẫn hiện, nhưng đổi thành "Profile" chứ không phải "Users"
await expect(page.locator('#menu-users .wp-menu-name')).toHaveText('Profile');
await expect(page.locator('#menu-users .wp-menu-name')).not.toHaveText('Users');
```

**Bài học lớn nhất:** một test "xanh" (pass) không đồng nghĩa với "đúng". Khi selector có thể trả về **0 phần tử**, luôn tự hỏi: *"test pass vì đúng thực sự, hay vì locator không tìm thấy gì và assertion coi đó là hợp lệ?"* — đặc biệt cẩn thận với `toBeHidden()`, `toHaveCount(0)`, `not.toBeVisible()`.

---

## 7. Cheat sheet tổng hợp

| Tình huống | Selector mẫu |
|---|---|
| Con trực tiếp theo id cha | `#parent-id > tag` |
| Nhiều class trên cùng 1 thẻ | `tag.class1.class2` (viết liền, không dấu cách) |
| Class không có id | kèm tên thẻ: `tag.class` |
| Attribute → CSS | `tag[@attr='v']` (XPath) → `tag.v` hoặc `tag[attr='v']` (CSS) |
| Class bị lặp ở nhiều nơi | luôn kèm `id` của cha gần nhất: `#unique-parent .class` |
| Lọc theo text (CSS không làm được) | Playwright: `locator.filter({ hasText: '...' })` hoặc `getByText()` |
| preceding-sibling (CSS không làm được) | chọn thẳng bằng class/id riêng của chính phần tử đó |
| Dòng dữ liệu động trong bảng | `parentSelector tr` + `.filter({hasText})` rồi thao tác tương đối bên trong |
| Assertion dễ bị pass giả | cẩn thận `toBeHidden()` khi locator có thể trả về 0 phần tử — kiểm tra `toHaveText()` thay vì chỉ check ẩn/hiện |

---

*Tài liệu được tổng hợp từ buổi thực hành thực tế trên trang WordPress Admin (pw-practice-dev.playwrightvn.com), kết hợp DevTools Elements panel (XPath search) và viết test Playwright tại `tests/students-submission/tuyen-vo/lesson-9/account-css-selector.spec.ts`.*
