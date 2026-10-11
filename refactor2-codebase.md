# Tổng hợp kiến thức — Session refactor & CI

> Ghi chú học tập tổng hợp lại toàn bộ những gì đã làm, đã phân tích và đã học được trong session làm việc với dự án `playwright-automation-framework`: từ đổi tên dự án, refactor cấu trúc, cập nhật tài liệu, commit/push, đọc kết quả CI, cho tới điều tra và sửa các lỗi flaky (race condition) và cấu hình CI chọn browser.
>
> Tài liệu **thiết kế** chính thức của dự án vẫn nằm ở [project-documentations/](../project-documentations/). File này là ghi chú để ôn lại và tra cứu.

---

## Mục lục

**Phần 1 — Refactor, tài liệu, commit**

1. [Tổng quan session](#1-tổng-quan-session)
2. [Đổi tên dự án](#2-đổi-tên-dự-án)
3. [Cấu trúc dự án sau refactor](#3-cấu-trúc-dự-án-sau-refactor)
4. [TypeScript: path alias, typecheck, readonly](#4-typescript-path-alias-typecheck-readonly)
5. [Biến môi trường: `.env`, `.env.example`, `dotenv`](#5-biến-môi-trường-env-envexample-dotenv)
6. [Viết assertion đúng — tránh assertion "luôn pass"](#6-viết-assertion-đúng--tránh-assertion-luôn-pass)
7. [Git: add, commit, push và upstream](#7-git-add-commit-push-và-upstream)

**Phần 2 — CI và xử lý lỗi**

8. [GitHub Actions (CI)](#8-github-actions-ci)
9. [Đọc kết quả CI](#9-đọc-kết-quả-ci)
10. [Checklist review trước khi commit](#10-checklist-review-trước-khi-commit)
11. [Việc còn tồn đọng](#11-việc-còn-tồn-đọng)
12. [Điều tra test fail bằng HTML report và trace](#12-điều-tra-test-fail-bằng-html-report-và-trace)
13. [Race condition và 3 lỗi flaky đã sửa](#13-race-condition-và-3-lỗi-flaky-đã-sửa) ⭐ quan trọng nhất
14. [Timeout trong Playwright](#14-timeout-trong-playwright)
15. [CI chọn browser và chạy tay](#15-ci-chọn-browser-và-chạy-tay)
16. [GitHub CLI (`gh`)](#16-github-cli-gh)

**Phần 3 — Vận hành và quyết định**

17. [Chi phí GitHub Actions và rủi ro repo public](#17-chi-phí-github-actions-và-rủi-ro-repo-public)
18. [Viết commit message tốt](#18-viết-commit-message-tốt)
19. [Chạy song song nhiều worker (đã cân nhắc, giữ 1 worker)](#19-chạy-song-song-nhiều-worker-đã-cân-nhắc-giữ-1-worker)

---

## 1. Tổng quan session

Các việc đã làm, theo thứ tự thời gian:

| # | Việc | Kết quả |
|---|---|---|
| 1 | Đổi tên dự án `playwright-codebase` → `playwright-automation-framework` | Thay 6 chỗ trong 4 file |
| 2 | Review code sau refactor lần 1 (gom code vào `src/`, thêm `BasePage`, path alias) | Viết lại/cập nhật README + 6 file trong `project-documentations/` |
| 3 | Sửa README: `BASE_URL` dùng placeholder thay vì URL thật | Đồng nhất với 2 biến còn lại |
| 4 | Review các lần refactor tiếp theo (xóa file login test cũ, đổi tên class, thêm `readonly`, sửa `.env.example`) | Cập nhật `03-pom-design.md` |
| 5 | Cấu hình CI: truyền 3 secrets vào bước chạy test, thêm bước typecheck | Sửa `playwright.yml` + phần CI trong README |
| 6 | Review toàn bộ trước khi commit | Không có lỗi chặn → commit & push (`7637d20`) |
| 7 | Đọc kết quả CI lần chạy đầu | 68 passed, 3 flaky, 7 failed (Firefox) |
| 8 | Tìm hiểu GitHub CLI (`gh`) | Máy công ty không cài được (cần quyền admin) → tiếp tục làm trên web |
| 9 | Tải report CI về `ci-report/`, phân tích error-context + trace | Tìm ra: form Add User **không được gửi đi** trên Firefox |
| 10 | Sửa lỗi trang Add User tải nhiều JS | `waitForPageLoad()` sau khi mở form |
| 11 | Sửa lỗi menu Log Out (hover) — lần 1 chưa đúng, lần 2 đúng | `logout()` = `mouse.move` + hover + click, bọc `toPass()` |
| 12 | Tăng timeout cho test phân quyền | `test.describe.configure({ timeout: 60000 })` |
| 13 | Phát hiện nguyên nhân gốc: WordPress **giành focus** khi đang `fill` | `fillAndVerify()` + `login()` kiểm tra cả form |
| 14 | CI mặc định chỉ chạy chromium, chạy tay được đủ 3 browser | `workflow_dispatch` trong `playwright.yml` + README |
| 15 | Cập nhật tài liệu | `01-website-feature.md` thêm mục 1.4 và mục 4 (timing) |
| 16 | Commit bản fix (`ee78d77`), chạy tay CI với `all` | **Vẫn fail**: 8 test Add User trên firefox, 6 trên webkit — dù local đã pass |
| 17 | Phân tích report lần 2: đo thời điểm trên 14 trace + đọc mã nguồn `user-profile.js` | Tìm ra **nguyên nhân gốc**: form chỉ sẵn sàng khi `zxcvbn.min.js` tải xong |
| 18 | Thêm `waitForAddUserFormReady()` (chờ ô Password có giá trị), override `clickAddUserTab()` | Commit `e86bb67` |
| 19 | Cập nhật `01-website-feature.md` + file ghi chú này theo nguyên nhân đầy đủ | |
| 20 | Chạy tay CI với `all` để xác nhận bản sửa | ✅ **Cả chromium, firefox, webkit đều pass** — lỗi Add User đã được xử lý tận gốc |
| 21 | Tìm hiểu chi phí chạy CI | Repo public → miễn phí. Phát hiện rủi ro: trace trong artifact chứa mật khẩu admin ([mục 17](#17-chi-phí-github-actions-và-rủi-ro-repo-public)) |
| 22 | Đặt commit message cho bản sửa zxcvbn | Không dùng lại message cũ, viết tiêu đề + thân rõ ràng ([mục 18](#18-viết-commit-message-tốt)) |
| 23 | Cân nhắc chạy nhiều worker trên CI | Phân tích rủi ro trùng dữ liệu `Date.now()` → **quyết định giữ 1 worker** ([mục 19](#19-chạy-song-song-nhiều-worker-đã-cân-nhắc-giữ-1-worker)) |

**Quy tắc làm việc của dự án** (trong [AGENTS.md](../AGENTS.md)): luôn trình bày những việc sẽ làm, **chờ xác nhận** rồi mới bắt đầu. Đó là lý do mỗi yêu cầu trong session đều có một bước "kế hoạch → bạn duyệt → thực hiện".

---

## 2. Đổi tên dự án

### Cách tìm text cũ

Tìm **không phân biệt hoa/thường** và bao cả các biến thể (`playwright-codebase`, `playwright_codebase`, `Playwright Codebase`), loại trừ `node_modules/`:

```bash
grep -rniI "codebase" --exclude-dir=node_modules --exclude-dir=.git .
```

- `-r`: tìm đệ quy trong thư mục
- `-n`: in số dòng
- `-i`: không phân biệt hoa/thường
- `-I`: bỏ qua file nhị phân

Sau khi thay, **chạy lại đúng lệnh tìm đó** để chắc chắn không còn sót (kết quả rỗng = sạch).

### Vì sao sửa `package-lock.json` trực tiếp, không chạy `npm install`?

`package-lock.json` có 2 chỗ chứa tên dự án (`"name"` ở gốc và trong `packages[""]`). Có 2 cách cập nhật:

| Cách | Ưu điểm | Rủi ro |
|---|---|---|
| Sửa tay 2 dòng `name` | Chỉ đổi đúng tên, không đụng gì khác | Không có |
| Chạy `npm install` | Tự động | Có thể **nâng version** các dependency (vì `^` trong `package.json` cho phép), tạo ra thay đổi ngoài ý muốn |

→ Chỉ muốn đổi tên thì sửa tay là an toàn nhất.

---

## 3. Cấu trúc dự án sau refactor

### 3.1 Trước và sau

```
TRƯỚC                                SAU
playwright-.../                      playwright-automation-framework/
├── pom/                             ├── src/
│   ├── dashboard/                   │   ├── pages/
│   │   ├── login/                   │   │   ├── base.page.ts        ← MỚI
│   │   └── ...                      │   │   ├── login/              ← tách khỏi dashboard
│   └── storefront/                  │   │   ├── dashboard/
├── fixtures/                        │   │   └── storefront/
├── utils/                           │   ├── fixtures/
├── tests/                           │   └── utils/
│   ├── dashboard/                   ├── tests/
│   │   └── login/                   │   ├── login/                  ← tách khỏi dashboard
│   └── storefront/                  │   ├── dashboard/
└── ...                              │   └── storefront/
                                     └── ...
```

### 3.2 Vì sao gom vào `src/`?

- **Ranh giới rõ ràng:** `src/` là "thư viện" (page object, fixture, hàm tiện ích), `tests/` là "nơi dùng thư viện" (kịch bản test).
- **Gốc project gọn:** chỉ còn `src/`, `tests/`, tài liệu và file cấu hình.
- **Dễ khai báo path alias:** mọi alias cùng trỏ vào một gốc `src/`.

### 3.3 Mirror 1-1 giữa `src/pages/` và `tests/`

```
src/pages/login/login.page.ts                <-->   tests/login/login.spec.ts
src/pages/dashboard/users/add-user.page.ts   <-->   tests/dashboard/users/add-user.spec.ts
src/pages/storefront/home.page.ts            <-->   tests/storefront/home.spec.ts
```

Muốn sửa test nào → biết ngay page object nằm ở đâu.

### 3.4 Vì sao login tách ra khỏi `dashboard/`?

Trang login là "cửa vào" khu vực admin nhưng **chưa thuộc** admin (chưa có sidebar, admin bar). Nên nó là một module ngang hàng với `dashboard/` và `storefront/`.

### 3.5 Sơ đồ kế thừa mới

```
BasePage                    (giữ page, đọc BASE_URL, goto(), waitForPageLoad(), getTitle(), fillAndVerify())
 ├── LoginPage              (form login, login(), getErrorMessage())
 ├── DashboardBasePage      (menu sidebar, logout(), heading Dashboard/At a Glance/Activity)
 │    ├── HomePage
 │    ├── UpdatesPage
 │    ├── UserPage          (gộp Add User + All Users)
 │    └── ProfilePage
 └── StoreFrontHomePage
```

**Thiết kế cũ:** `LoginPage → DashboardBasePage → ...`, tức là mọi trang admin "là một" trang login.

**Vì sao bỏ?** Trang admin **không phải** trang login. Kế thừa `LoginPage` khiến mọi page admin mang theo locator form login vô nghĩa. Thiết kế mới:

- Phần **mọi trang đều cần** → `BasePage`.
- Cần login → tạo riêng `new LoginPage(page)` rồi dùng (trong fixture, hoặc khi login lại giữa test).

### 3.6 Quy tắc constructor mới

`BasePage` đã khai `readonly page` và gán `this.page = page`. Class con **chỉ cần** gọi `super(page)`:

```ts
export class UpdatesPage extends DashboardBasePage {
    readonly wordPressUpdatesHeadingLoc: Locator;

    constructor(page: Page) {
        super(page);   // BasePage gán this.page, DashboardBasePage khởi tạo locator menu
        this.wordPressUpdatesHeadingLoc = this.page.getByRole('heading', { name: 'WordPress Updates', exact: true });
    }
}
```

❌ Không còn tự khai lại `page: Page` hay gán `this.page = page` ở class con.

### 3.7 Fixture thay đổi theo

```ts
const loginPage = new LoginPage(page);                  // (1) login bằng LoginPage
await loginPage.openPlaywrightPage();
await loginPage.login(userAdmin, pswAdmin);

const dashboardBasePage = new DashboardBasePage(page);  // (2) cùng page, giờ đã ở trang admin
await expect(page).toHaveURL(/\/wp-admin\/$/);          // regex, không hardcode domain
```

Điểm mấu chốt: **nhiều page object có thể dùng chung một `page`**. `LoginPage` và `DashboardBasePage` chỉ là 2 "góc nhìn" khác nhau lên cùng một tab trình duyệt.

---

## 4. TypeScript: path alias, typecheck, readonly

### 4.1 Path alias

Khai trong `tsconfig.json`:

```jsonc
"paths": {
  "@data/*": ["./src/data/*"],
  "@pages/*": ["./src/pages/*"],
  "@fixtures/*": ["./src/fixtures/*"],
  "@utils/*": ["./src/utils/*"]
}
```

So sánh:

```ts
// Trước — phụ thuộc vào file test nằm sâu bao nhiêu tầng
import { UserPage } from '../../../pom/dashboard/users/add-user.page';

// Sau — giống nhau ở mọi file
import { UserPage } from '@pages/dashboard/users/add-user.page';
```

**Lợi ích:**
- Di chuyển file test sang thư mục khác không làm gãy import.
- Đọc import là biết ngay đang dùng page, fixture hay util.
- Playwright **tự đọc** `paths` trong `tsconfig.json` khi chạy test, không cần cấu hình thêm.

**Lưu ý:**
- Đường dẫn bắt đầu bằng `./` (tương đối so với vị trí `tsconfig.json`).
- **Không** khai `baseUrl`: dự án dùng TypeScript 7, bản này đã bỏ `baseUrl` (trong `tsconfig.json` dòng `"baseUrl"` đang bị comment lại).
- `@data/*` được khai sẵn nhưng thư mục `src/data/` **chưa tồn tại**.

### 4.2 `npm run typecheck`

Script trong `package.json`:

```json
"scripts": {
  "typecheck": "tsc --noEmit"
}
```

- `tsc`: trình biên dịch TypeScript.
- `--noEmit`: chỉ **kiểm tra lỗi**, không sinh ra file `.js`.

**Vì sao cần?** Playwright khi chạy test chỉ **chuyển** TypeScript thành JavaScript để chạy, **không kiểm tra type**. Vì vậy:
- Gọi sai tên method, truyền sai kiểu tham số → test vẫn được chạy, chỉ lỗi khi tới đúng dòng đó (hoặc tệ hơn là chạy sai mà không báo).
- Import sai đường dẫn → chỉ phát hiện khi chạy file đó.

**Ví dụ thật trong session:** file cũ `tests/dashboard/login/login.spec.ts` vẫn import `'../../../pom/dashboard/login/login.page'` (thư mục `pom/` đã bị xóa). `npm run typecheck` bắt được ngay:

```
tests/dashboard/login/login.spec.ts(2,27): error TS2307: Cannot find module '../../../pom/dashboard/login/login.page'
```

→ Nên chạy `npm run typecheck` trước mỗi lần commit, và đã thêm vào CI (xem [mục 8](#8-github-actions-ci)).

### 4.3 `readonly` cho locator

```ts
readonly userNameLogin: Locator;
readonly passwordLogin: Locator;
```

`readonly` = chỉ được gán **một lần** (trong constructor). Nếu sau này lỡ viết nhầm `this.userNameLogin = ...` ở chỗ khác (lỗi copy-paste rất hay gặp khi nhiều biến tên gần giống nhau), TypeScript báo lỗi ngay lúc `typecheck`, thay vì để lỗi trôi tới lúc chạy test.

Trong session, `readonly` đã được bổ sung cho **toàn bộ** locator trong mọi Page Object, kể cả chuỗi selector `storeFrontPageLoc`.

### 4.4 Lỗi format nhỏ đã gặp

| Lỗi | Sửa thành |
|---|---|
| `readonly  storeFrontPageLoc` (2 dấu cách) | `readonly storeFrontPageLoc` |
| `storeFrontPageLoc:string` | `storeFrontPageLoc: string` (có dấu cách sau `:`) |
| Dấu cách thừa cuối dòng | Xóa |
| `clickstoreFrontPage` | `clickStoreFrontPage` (camelCase) |
| `StroreFrontHomePage` | `StoreFrontHomePage` (sai chính tả) |

**Mẹo VS Code:** bật `"files.trimTrailingWhitespace": true` trong Settings để tự xóa dấu cách cuối dòng khi lưu file.

---

## 5. Biến môi trường: `.env`, `.env.example`, `dotenv`

### 5.1 Ba thành phần

| Thành phần | Vai trò | Có commit lên GitHub? |
|---|---|---|
| `.env` | Chứa giá trị **thật** (URL, tài khoản, mật khẩu) | ❌ Không — nằm trong `.gitignore` |
| `.env.example` | Chứa **tên biến**, giá trị rỗng — để người khác biết cần khai báo gì | ✅ Có |
| `dotenv` | Thư viện đọc `.env` và nạp vào `process.env` | — |

`playwright.config.ts` gọi `dotenv.config(...)` một lần. Playwright luôn load file config trước khi chạy test, nên mọi chỗ khác dùng được `process.env.BASE_URL`.

### 5.2 Cú pháp `.env` đúng

```bash
BASE_URL=
ADMIN_USERNAME=
ADMIN_PASSWORD=

# Comment bắt đầu bằng # ở đầu dòng
```

| ❌ Sai | ✅ Đúng | Lý do |
|---|---|---|
| `BASE_URL` (thiếu `=`) | `BASE_URL=` | Không có `=` thì `dotenv` không coi là biến |
| `/* comment */` | `# comment` | `.env` không phải JavaScript |
| ` # comment` (dấu cách trước `#`) | `# comment` | `dotenv` vẫn bỏ qua nhưng không chuẩn, editor có thể không highlight là comment |
| `const BASE_URL = "..."`; | `BASE_URL=...` | `dotenv` sẽ đọc luôn ký tự thừa vào giá trị |

File dùng xuống dòng kiểu Windows (CRLF) vẫn được `dotenv` xử lý bình thường.

### 5.3 Dấu `!` trong `process.env.BASE_URL!`

TypeScript luôn coi `process.env.X` có kiểu `string | undefined` (vì biến có thể không tồn tại). Dấu `!` nói với TypeScript: "tôi chắc chắn biến này có giá trị". Nếu thực tế thiếu biến, giá trị sẽ là `undefined` → test sẽ fail khi dùng.

### 5.4 Vì sao README dùng placeholder cho `BASE_URL`?

```
BASE_URL=<URL trang admin, VD: https://pw-practice-dev.playwrightvn.com/wp-admin>
ADMIN_USERNAME=<tài khoản admin>
ADMIN_PASSWORD=<mật khẩu admin>
```

URL này **không phải bí mật** (là trang practice công khai), nhưng ghi giá trị thật thì **không nhất quán**:
- 2 biến còn lại dùng placeholder, riêng `BASE_URL` lại là giá trị thật.
- Trái với quy ước "không hardcode URL".
- Nếu sau này có nhiều môi trường (staging, prod), người đọc dễ tưởng đây là giá trị bắt buộc.

→ Dùng placeholder nhưng **giữ ví dụ** để người mới biết URL phải trỏ tới `/wp-admin`.

**Phân biệt:** thông tin **bí mật** (password, token) → tuyệt đối không ghi. Thông tin **phụ thuộc môi trường** (URL) → nên để placeholder để nhất quán.

---

## 6. Viết assertion đúng — tránh assertion "luôn pass"

### 6.1 Bài học từ `tests/storefront/home.spec.ts`

Test: từ trang admin, click link ra trang chủ storefront, rồi kiểm tra URL.

```ts
// Phiên bản 1 — hardcode URL
await expect(page).toHaveURL('https://pw-practice-dev.playwrightvn.com/');

// Phiên bản 2 — bỏ hardcode, nhưng...
await expect(page).toHaveURL(/playwrightvn/);
```

**Phiên bản 2 có vấn đề nghiêm trọng:** regex `/playwrightvn/` chỉ kiểm tra URL **có chứa** chữ `playwrightvn`. Nhưng **trước khi click**, trình duyệt đang ở:

```
https://pw-practice-dev.playwrightvn.com/wp-admin/
                      ^^^^^^^^^^^^ cũng chứa "playwrightvn"
```

→ Nếu click **không hoạt động**, trang vẫn ở `/wp-admin/` → assertion vẫn **pass**. Test không còn bắt được lỗi nó được viết ra để bắt.

### 6.2 Cách đúng: kiểm tra đúng đặc điểm của trang đích

Trang chủ storefront = **sau tên miền chỉ có dấu `/`**:

```ts
await expect(page).toHaveURL(/^https?:\/\/[^/]+\/$/);
```

Giải thích regex từng phần:

| Phần | Nghĩa |
|---|---|
| `^` | Bắt đầu chuỗi |
| `https?` | `http` hoặc `https` (`s` có hoặc không) |
| `:\/\/` | `://` (dấu `/` phải escape thành `\/`) |
| `[^/]+` | Một hoặc nhiều ký tự **không phải** `/` → chính là tên miền |
| `\/` | Dấu `/` |
| `$` | Kết thúc chuỗi — sau `/` không còn gì nữa |

Kết quả:

| URL | Khớp? |
|---|---|
| `https://pw-practice-dev.playwrightvn.com/` | ✅ |
| `https://pw-practice-dev.playwrightvn.com/wp-admin/` | ❌ (sau `/` còn `wp-admin/`) |
| `https://staging.example.com/` | ✅ (không phụ thuộc domain) |

Đây cũng là cách fixture đang làm: `/\/wp-admin\/$/` (URL **kết thúc** bằng `/wp-admin/`).

### 6.3 Nguyên tắc chung

> Một assertion tốt phải **fail khi hành động không xảy ra**. Hãy tự hỏi: "Nếu bước trước bị lỗi, dòng `expect` này có fail không?" Nếu câu trả lời là "không" thì assertion đó vô dụng.

Cũng cần chú ý text gắn với môi trường, ví dụ `"Playwright practice site - Dev environment"` — chữ "Dev environment" sẽ khác nếu chạy trên môi trường khác.

---

## 7. Git: add, commit, push và upstream

### 7.1 Quy trình cơ bản

```bash
git add .          # đưa mọi thay đổi vào "vùng chuẩn bị" (staging)
git status         # xem lại danh sách trước khi commit
git commit -m "..."
git push -u origin main
```

### 7.2 `git add .` làm những gì?

Với Git 2.x, `git add .` ghi nhận **cả 3 loại** thay đổi:

| Ký hiệu trong `git status --short` | Nghĩa | Ví dụ trong session |
|---|---|---|
| `??` | File/thư mục mới (untracked) | `src/`, `tests/login/` |
| ` M` | File đã sửa | `README.md`, `tsconfig.json`... (19 file) |
| ` D` | File đã xóa | toàn bộ `pom/`, `fixtures/`, `utils/` cũ (56 file) |

Không cần lệnh riêng để ghi nhận file bị xóa.

### 7.3 `.gitignore` — kiểm tra trước khi `git add .`

```bash
git check-ignore -v .env
# .gitignore:11:.env    .env     ← có kết quả = .env đang bị ignore, an toàn
```

Nếu lệnh này không in ra gì nghĩa là `.env` **sẽ bị commit** → phải thêm vào `.gitignore` trước.

Trong session, `.gitignore` còn được thêm:
- `/ci-report/` — report tải từ CI về để điều tra, không cần đưa lên GitHub (nặng ~40MB, chứa trace có cả tài khoản admin dạng chữ rõ).
- `/notes/` — file ghi chú cá nhân này.

### 7.4 Upstream là gì? (phần dễ nhầm nhất)

**Tình huống:** bạn đã từng push lên GitHub, nhánh `main` đã có trên GitHub — vậy sao vẫn cần `-u`?

**Trả lời:** "upstream" là một **thiết lập nằm trong máy bạn**, không nằm trên GitHub. Nó được lưu trong `.git/config`:

```ini
[branch "main"]
    remote = origin
    merge = refs/heads/main
```

Nghĩa là: "khi đứng ở nhánh `main` mà gõ `git push`/`git pull` không tham số → làm việc với `origin/main`".

**Kiểm tra:**

```bash
git branch -r                       # origin/main  ← GitHub có nhánh main
git config --get-regexp '^branch\.' # (trống)      ← main local CHƯA theo dõi origin/main
```

**Vì sao đã push rồi mà vẫn chưa có?** Lần trước push bằng `git push origin main` (không có `-u`). Lệnh này đẩy code lên và tạo nhánh trên GitHub, nhưng **không ghi** thiết lập upstream.

| Lệnh | Chưa có upstream | Đã có upstream |
|---|---|---|
| `git push origin main` | ✅ | ✅ |
| `git push` (không tham số) | ❌ *"The current branch main has no upstream branch"* | ✅ |
| `git pull` (không tham số) | ❌ | ✅ |
| `git status` báo ahead/behind | Không | Có |

→ Chạy `git push -u origin main` **một lần**, từ đó chỉ cần `git push` / `git pull`. Thêm `-u` không thay đổi gì trên GitHub, chỉ ghi cấu hình vào máy bạn.

### 7.5 Khi push bị từ chối

Nếu gặp lỗi **rejected / non-fast-forward**: trên GitHub có commit mà local chưa có. **Không dùng `--force`**, vì nó sẽ ghi đè và làm mất commit trên GitHub. Cần `git pull` để gộp về trước, rồi mới push.

### 7.6 CI chỉ chạy code ĐÃ PUSH — kiểm tra local và GitHub có đồng bộ không

Khi CI fail mà local pass, câu hỏi đầu tiên nên hỏi: **CI có đang chạy đúng code mới không?** GitHub Actions lấy code **trên GitHub** (`actions/checkout`), không phải code trên máy bạn. Code sửa xong nhưng chưa commit, hoặc commit rồi chưa push → CI vẫn chạy bản cũ.

```bash
git status --short                                   # còn thay đổi chưa commit không?
git fetch origin                                     # cập nhật thông tin nhánh trên GitHub về máy (không đổi code của bạn)
git log --oneline -3 origin/main                     # GitHub đang ở commit nào?
git rev-list --left-right --count main...origin/main # đếm commit lệch nhau
```

Kết quả lệnh cuối là 2 con số `A	B`:

| Kết quả | Nghĩa |
|---|---|
| `0	0` | ✅ Local và GitHub giống hệt nhau — CI chạy đúng code của bạn |
| `2	0` | Local **đi trước** 2 commit — cần `git push` |
| `0	1` | GitHub **đi trước** 1 commit — cần `git pull` |

Trong session, lần CI vẫn fail sau khi sửa: kiểm tra ra `0 0` (cùng ở commit `ee78d77`) → loại trừ được khả năng "chưa push", và tập trung vào phân tích report — dẫn tới tìm ra nguyên nhân zxcvbn.

### 7.7 `git add .` an toàn khi nào?

`git add .` đưa **mọi** thay đổi trong thư mục vào staging. An toàn khi:
- `.gitignore` đã chặn đủ những thứ không được commit (trong dự án: `.env`, `ci-report/`, `notes/`, `test-results/`, `playwright-report/`, `node_modules/`).
- Không có file tạm/lạ nào mới xuất hiện.

Có thể kiểm tra nhanh nhiều đường dẫn cùng lúc:

```bash
git check-ignore .env ci-report notes test-results playwright-report
# In ra tên nào = tên đó đang bị ignore
```

**Thói quen nên giữ:** sau `git add .`, luôn `git status` **trước khi commit** — để chắc không có file report, file tạm mới mà `.gitignore` chưa chặn lọt vào.

**Sau lần push `-u` đầu tiên**, nhánh `main` đã có upstream → các lần sau chỉ cần:

```bash
git add .
git status
git commit -m "..."
git push          # không cần "-u origin main" nữa
```

---

## 8. GitHub Actions (CI)

### 8.1 Workflow hiện tại

File [.github/workflows/playwright.yml](../.github/workflows/playwright.yml):

```yaml
on:
  push:            { branches: [ main, master ] }     # tự chạy khi push
  pull_request:    { branches: [ main, master ] }     # tự chạy khi tạo PR
  workflow_dispatch:                                  # chạy tay từ tab Actions   ← MỚI
    inputs:
      browsers: { type: choice, options: [chromium, all], default: chromium }

jobs:
  test:
    env:
      BROWSERS: ${{ inputs.browsers || 'chromium' }}  # push/PR -> chromium   ← MỚI
    steps:
    - uses: actions/checkout@v4                       # 1. Lấy code về
    - uses: actions/setup-node@v4                     # 2. Cài Node.js
    - run: npm ci                                     # 3. Cài dependency theo đúng package-lock.json
    - run: npm run typecheck                          # 4. Kiểm tra type
    - run: ... cài chromium hoặc cả 3 browser          # 5. Theo BROWSERS              ← SỬA
    - run: ... chạy --project=chromium hoặc tất cả     # 6. Theo BROWSERS + 3 secrets  ← SỬA
    - uses: actions/upload-artifact@v4                # 7. Upload report HTML (kể cả khi fail)
```

(Viết gọn để dễ nhìn — file thật dùng cú pháp YAML nhiều dòng. Chi tiết phần chọn browser xem [mục 15](#15-ci-chọn-browser-và-chạy-tay).)

### 8.2 `npm ci` khác `npm install` thế nào?

`npm ci` cài **chính xác** theo `package-lock.json` và **fail** nếu `package-lock.json` không khớp `package.json`. Vì vậy trước khi push cần chắc 2 file này đồng bộ (trong session đã kiểm tra: khớp).

### 8.3 Secrets — vì sao cần và truyền thế nào

Trên CI **không có file `.env`** (vì nó không được commit). Giải pháp: lưu giá trị vào **GitHub Secrets** rồi truyền vào bước chạy test.

**Bước 1 — Khai báo trên GitHub:** Repo → **Settings → Secrets and variables → Actions → New repository secret**, tạo 3 secret:

| Name | Giá trị |
|---|---|
| `BASE_URL` | URL trang `/wp-admin/` |
| `ADMIN_USERNAME` | tài khoản admin |
| `ADMIN_PASSWORD` | mật khẩu admin |

**Bước 2 — Truyền vào workflow:** khối `env:` ở bước "Run Playwright tests":

```yaml
env:
  BASE_URL: ${{ secrets.BASE_URL }}
  ADMIN_USERNAME: ${{ secrets.ADMIN_USERNAME }}
  ADMIN_PASSWORD: ${{ secrets.ADMIN_PASSWORD }}
```

**Vì sao code không cần sửa gì?**
- Workflow đặt sẵn 3 biến vào `process.env`.
- `dotenv.config()` không tìm thấy `.env` → không đọc gì.
- `dotenv` **không ghi đè** biến đã tồn tại.
→ `process.env.BASE_URL` lấy đúng giá trị từ secret.

**Lưu ý:**
- Tên secret phải **trùng khớp chính xác** (chữ hoa, không dấu cách).
- Khi dán giá trị, không để thừa dấu cách/dòng trống — đây là lỗi hay gặp nhất khiến login fail trên CI dù local chạy đúng.
- Sau khi lưu, GitHub **không cho xem lại** giá trị, chỉ cho ghi đè. Trang Secrets chỉ hiện **tên** — nhìn danh sách tên chỉ xác nhận được "đã khai báo", không xác nhận được giá trị đúng. Cách chắc chắn nhất: xem lần chạy CI đầu tiên, test login pass = giá trị đúng.
- GitHub tự che secret thành `***` trong log, nhưng đừng `console.log` chúng.
- Pull Request từ **fork** không nhận được secrets → test cần login sẽ fail.

### 8.4 Vì sao bước typecheck đặt **trước** bước cài browser?

Nguyên tắc **fail fast** — kiểm tra rẻ và nhanh chạy trước:
- Typecheck: vài giây.
- Cài trình duyệt + thư viện: 1–2 phút.

Nếu có lỗi type, job dừng ngay sau vài giây, không tốn thời gian tải browser.

### 8.5 Artifact

Report HTML được upload thành artifact `playwright-report`, giữ 30 ngày. Tải về ở cuối trang của lần chạy, mục **Artifacts**. `if: ${{ !cancelled() }}` = chạy bước này kể cả khi test fail (chỉ bỏ qua khi workflow bị hủy) — vì khi test fail mới là lúc cần report nhất.

---

## 9. Đọc kết quả CI

### 9.1 Vì sao CI chạy lâu (18 phút)?

Do các cấu hình trong [playwright.config.ts](../playwright.config.ts) cộng dồn:

| Cấu hình | Giá trị trên CI | Ảnh hưởng |
|---|---|---|
| `workers` | `1` | Test chạy **tuần tự** từng cái một (local chạy song song nhiều worker) |
| `projects` | chromium, firefox, webkit | 26 test × 3 trình duyệt = **78 lần chạy** |
| `retries` | `2` | Test fail được chạy lại tối đa **3 lần**, mỗi lần có thể chờ hết timeout 30s |
| `timeout-minutes` (workflow) | `60` | Quá 60 phút thì job bị **cắt** |

Thêm vào đó: mỗi test đều login qua fixture, test về user còn tạo user → logout → login lại → xóa user.

**Trường hợp xấu nhất:** nếu login trên CI hỏng (ví dụ secret sai), gần như cả 78 test fail ở fixture → 78 × 3 lần × 30s ≈ hơn 1 giờ → bị cắt ở 60 phút. Khi thấy log đầy `✘` và `retry #1`, `retry #2` ngay ở fixture thì nên **Cancel workflow** sớm.

**Đã áp dụng:** CI mặc định chỉ chạy chromium (26 test) — xem [mục 15](#15-ci-chọn-browser-và-chạy-tay). Hướng còn lại (chưa áp dụng): tăng `workers` lên 2 — cần cẩn thận vì các test dùng chung một tài khoản admin.

### 9.2 Ý nghĩa các trạng thái

Kết quả lần chạy đầu sau refactor:

```
7 failed
3 flaky
68 passed (18.2m)
```

| Trạng thái | Nghĩa |
|---|---|
| **passed** | Pass ngay lần đầu |
| **flaky** | Fail lần đầu nhưng pass khi chạy lại → test **không ổn định**, thường do timing (trang load chậm, thao tác khi phần tử chưa sẵn sàng) |
| **failed** | Fail cả 3 lần (lần đầu + 2 lần retry) |

Chỉ cần còn test **failed** là bước "Run Playwright tests" trả về `exit code 1` → job đỏ.

### 9.3 Đọc ra được gì từ kết quả — và kết luận cuối cùng

**Nhận xét ban đầu** (chỉ từ dòng tổng kết):
1. **68 passed → secrets đúng.** Login qua fixture chạy được trên cả 3 trình duyệt. Nếu secret sai, gần như toàn bộ test đã fail.
2. **7 failed đều là `[firefox]` trong `add-user.spec.ts`**, cùng test đó chromium và webkit đều pass.
3. Các test fail **đứng liền nhau** trong file (dòng 142 → 287), ngay trước đó test ở dòng 127 bị flaky.

Lúc này chỉ có **giả thuyết** (khác biệt riêng của Firefox, hoặc website chậm) — chưa đủ bằng chứng để kết luận.

**Kết luận sau khi điều tra** (chi tiết cách điều tra ở [mục 12](#12-điều-tra-test-fail-bằng-html-report-và-trace), cơ chế và cách sửa ở [mục 13](#13-race-condition-và-3-lỗi-flaky-đã-sửa)):

| Lỗi | Nguyên nhân thật | Đã sửa bằng |
|---|---|---|
| 7 test Add User fail (Firefox) — và 8 test firefox + 6 test webkit ở lần chạy tay `all` | **Gốc:** form Add User chỉ khởi tạo xong khi `zxcvbn.min.js` (~400KB, tải **sau** `load`) tải xong. Test thao tác trước thời điểm đó → 2 hệ quả: (a) **chữ bị điền nhầm ô** (WordPress giành focus về ô Password), (b) **bấm submit mà form không được gửi đi** | `waitForAddUserFormReady()` (chờ ô Password có giá trị) — ✅ đã xác nhận trên CI (cả 3 browser pass). `fillAndVerify()` giữ làm lớp bảo vệ thêm |
| `add-user.spec.ts:127` flaky (Firefox) | Cùng nguyên nhân trên (lần retry may mắn: zxcvbn tải xong kịp) | Như trên |
| `account-permissions` flaky (WebKit) | Menu Log Out mở bằng hover có độ trễ, click hụt rồi chờ mãi | `logout()` với `mouse.move` + `toPass()` |
| (Phát hiện thêm khi chạy local) Login lại trong `afterEach` fail | Trang login tự focus ô Username, mật khẩu đè vào ô Username | `login()` kiểm tra cả form |

> **Bài học về điều tra lỗi:** cùng một lỗi Add User, kết luận đã được điều chỉnh **3 lần**:
>
> | Lần | Bằng chứng có được | Kết luận lúc đó | Đánh giá sau cùng |
> |---|---|---|---|
> | 1 | Trace: không có request POST | "Form không được gửi đi vì JS chưa khởi tạo xong" → sửa bằng `waitForPageLoad()` | Hiện tượng đúng, cách sửa chưa đủ |
> | 2 | Snapshot: username bị nối vào ô Password | "WordPress giành focus khi đang fill" → sửa bằng `fillAndVerify()` | Đúng nhưng chỉ là **một hệ quả** |
> | 3 | Các ô đã đúng mà vẫn không POST; đo 14/14 trace + đọc mã nguồn `user-profile.js` | "Form chỉ sẵn sàng khi zxcvbn tải xong" → sửa bằng `waitForAddUserFormReady()` | **Nguyên nhân gốc** — giải thích được cả 2 hệ quả |
>
> Vì vậy: luôn phân biệt rõ **"đã xác nhận"** với **"đang nghi ngờ"**; một cách sửa làm hết lỗi **ở local** chưa chắc đã chạm tới nguyên nhân gốc; và sẵn sàng điều chỉnh kết luận khi có bằng chứng mới.

### 9.4 Cách điều tra test fail trên CI

**Cách A — Tải report từ CI** (khuyên dùng, vì dùng đúng lỗi đã xảy ra):

1. Mở lần chạy → cuối trang, mục **Artifacts** → tải `playwright-report`.
2. Giải nén vào một thư mục, ví dụ `ci-report/` (nhớ thêm vào `.gitignore`).
3. Chạy:
   ```bash
   npx playwright show-report ci-report
   ```
4. Mở từng test fail để xem lỗi. Config có `trace: 'on-first-retry'` nên test được retry đều có **trace** — xem lại từng bước kèm ảnh chụp màn hình, DOM, network.

**Cách B — Chạy lại ở local:**

```bash
npx playwright test tests/dashboard/users/add-user.spec.ts --project=firefox
```

⚠️ Lệnh này **thao tác trên website thật** (tạo rồi xóa user). Nếu test fail giữa chừng có thể còn sót user. Và nếu nguyên nhân là do website chậm thì chạy local có thể **không tái hiện** được.

---

## 10. Checklist review trước khi commit

Các bước đã dùng trong session, có thể tự chạy lại:

```bash
# 1. Xem tổng quan thay đổi
git status --short

# 2. Chắc chắn .env (và ci-report/) không bị commit
git check-ignore -v .env ci-report

# 3. Không lỗi TypeScript
npm run typecheck

# 4. Playwright load được tất cả file test (không chạy test, chỉ liệt kê)
npx playwright test --list

# 5. package-lock.json khớp package.json (để npm ci trên CI không fail)
node -e "const l=require('./package-lock.json'),p=require('./package.json');console.log(JSON.stringify(l.packages[''].devDependencies)===JSON.stringify(p.devDependencies)?'in sync':'MISMATCH')"

# 6. Không còn import trỏ tới cấu trúc cũ
grep -rn "pom/\|'\.\./\.\./\.\./" src tests

# 7. Không có mật khẩu/URL hardcode ngoài ý muốn
grep -rnE "https?://" src tests

# 8. (Khi sửa lỗi flaky) chạy lặp nhiều lần trên browser hay lỗi — thao tác trên site thật!
npx playwright test tests/dashboard/users --project=firefox --repeat-each=3

# 9. (Trước khi merge thay đổi lớn) chạy đủ 3 browser — vì CI push/PR chỉ chạy chromium
npx playwright test
```

Ngoài lệnh, cần tự kiểm tra:
- [ ] Các file bị xóa (` D`) đều là cấu trúc cũ đã được chuyển đi, không xóa nhầm.
- [ ] Document (README, `project-documentations/`) khớp với code: cây thư mục, tên class, số dòng tham chiếu, code mẫu, link. **Đối chiếu với code thật, không với ý định** — trong session từng ghi nhầm "`clickAddUserTab()` có chờ `load`" trong khi thực tế là `clickUserTab()`.
- [ ] Không có tên cũ/sai chính tả còn sót trong document.
- [ ] Assertion trong test thực sự fail được khi hành động không xảy ra (xem [mục 6](#6-viết-assertion-đúng--tránh-assertion-luôn-pass)).
- [ ] Sau các lần test fail giữa chừng: vào **Users** trên site, tìm user test còn sót (email dạng `tuyen-<số>@gmail.com`) và xóa.

**Về "flaky":** chạy pass **1 lần** chưa chứng minh được đã hết lỗi flaky — vì lỗi vốn chỉ thỉnh thoảng xảy ra. Cần chạy lặp (`--repeat-each`) và theo dõi thêm vài lần chạy CI.

---

## 11. Việc còn tồn đọng

### Đã xong

| Việc | Cách giải quyết |
|---|---|
| ~~7 test Firefox fail trong `add-user.spec.ts`~~ | Tìm ra nguyên nhân gốc (zxcvbn) → `waitForAddUserFormReady()` + `fillAndVerify()` ([mục 13.3](#133-lỗi-1--trang-add-user-form-chỉ-sẵn-sàng-khi-zxcvbn-tải-xong)) — ✅ đã xác nhận trên CI |
| ~~3 test flaky~~ | `fillAndVerify()`, `logout()` mới, timeout 60s ([mục 13](#13-race-condition-và-3-lỗi-flaky-đã-sửa)) |
| ~~Tăng tốc CI~~ | Push/PR chỉ chạy chromium, chạy tay được `all` ([mục 15](#15-ci-chọn-browser-và-chạy-tay)) |
| ~~Commit & push bản fix flaky + CI chọn browser~~ | Commit `ee78d77` |
| ~~`clickAddUserTab()` chưa chờ form sẵn sàng~~ | Override trong `UserPage`, gọi `waitForAddUserFormReady()` (commit `e86bb67`) |
| ~~Xác nhận bản sửa zxcvbn trên CI~~ | Chạy tay `all`: cả 3 browser pass ✅ |

### Còn lại

| # | Việc | Mức độ | Ghi chú |
|---|---|---|---|
| 1 | Kiểm tra **user test bị sót** trên site | 🟠 Trung bình | Từ các lần `afterEach` bị cắt giữa chừng hoặc test fail — tìm `tuyen-` trong Users |
| 2 | Xem lại việc **repo public** làm lộ mật khẩu qua trace trong artifact | 🟠 Trung bình | Trace ghi nguyên văn giá trị `fill()`, kể cả mật khẩu admin; GitHub không che secrets bên trong artifact. Hướng: chuyển repo private, xóa artifact cũ, tắt trace trên CI, hoặc đổi mật khẩu |
| 3 | `tests/storefront/home.spec.ts:14` vẫn hardcode URL | 🟡 Thấp | Đề xuất: `toHaveURL(/^https?:\/\/[^/]+\/$/)` ([mục 6](#6-viết-assertion-đúng--tránh-assertion-luôn-pass)) |
| 4 | Text `"... - Dev environment"` gắn với môi trường dev | 🟡 Thấp | Cần xem lại nếu chạy trên môi trường khác |
| 5 | ~34 dòng code cũ để dạng comment (`//await ...`) | ⚪ Tùy chọn | Trong fixture, `add-user.page.ts`, `login.page.ts`, các file test — gồm cả bản cũ của `login()`, `hoverProfile()/clickLogout()`. Có git lưu lịch sử rồi nên có thể xóa cho gọn |
| 6 | Alias `@data/*` chưa có thư mục `src/data/` | ⚪ Tùy chọn | Tạo khi cần dữ liệu test tĩnh |
| 7 | Tăng `workers` trên CI | ⚪ Tùy chọn | Cân nhắc vì các test dùng chung tài khoản admin |
| 8 | Cài GitHub CLI | ⚪ Tùy chọn | Cần IT cài (máy công ty) — xem [mục 16](#16-github-cli-gh) |
| 9 | Theo dõi thêm vài lần chạy CI với `all` | ⚪ Tùy chọn | Lỗi timing là flaky — 1 lần pass đã là bằng chứng tốt, nhưng theo dõi thêm vài lần (nhất là khi site hoặc WordPress cập nhật) giúp chắc chắn hơn |

---

## 12. Điều tra test fail bằng HTML report và trace

### 12.1 Bên trong thư mục report có gì?

Sau khi tải artifact `playwright-report` và giải nén vào `ci-report/`:

```
ci-report/
├── index.html      ← trang report; DỮ LIỆU KẾT QUẢ được nhúng ngay trong file này
├── data/
│   ├── <hash>.md   ← error-context: thông tin lỗi + "ảnh chụp" cấu trúc trang lúc fail
│   └── <hash>.zip  ← trace: ghi lại toàn bộ quá trình chạy test
└── trace/          ← chương trình Trace Viewer để mở file trace
```

| Thành phần | Chứa gì | Dùng để |
|---|---|---|
| `index.html` | Danh sách test, trạng thái, thông báo lỗi từng lần chạy (nhúng dạng file zip mã hóa base64) | Xem tổng quan, đọc thông báo lỗi |
| `error-context.md` | Thông báo lỗi + **page snapshot** (cây accessibility của trang lúc fail: heading, ô nhập và **giá trị trong ô**, nút...) + đoạn code test đang chạy | Biết **trang đang ở trạng thái nào** khi fail |
| trace `.zip` | Timeline mọi thao tác (`fill`, `click`, `expect`...) kèm thời điểm, **toàn bộ request mạng**, DOM, ảnh chụp màn hình | Biết **chuyện gì đã xảy ra** theo thứ tự thời gian |

Cách xem dễ nhất: `npx playwright show-report ci-report` → mở test fail → tab **Errors**, **Trace**.

### 12.2 Error-context: "ảnh chụp" trạng thái trang

Đây là bằng chứng quyết định trong session. Ví dụ snapshot lúc test `duplicate email` fail:

```yaml
- heading "Add User" [level=1]
- textbox "Username (required)"                                       ← trống!
- textbox "Email (required)": tuyen-1791699032022@gmail.com
- textbox "Password": (8(qGAtN$)XH8ymOi)h6rs!VE101-tuyen-1791699032022  ← username bị nối vào đây!
- button "Add User"
```

Chỉ nhìn thông báo lỗi (*"New user created không hiện ra"*) thì không đoán được. Nhìn snapshot thì thấy ngay: **chữ bị điền nhầm ô**.

### 12.3 Trace: timeline thao tác và request mạng

Trace cho biết **thao tác nào xảy ra lúc nào** và **có request nào được gửi đi không**. So sánh test fail và test pass trên cùng Firefox:

```
TEST FAIL (username & email only)                 TEST PASS (@ACC_002)
+4.69s click link "Add User" → GET user-new.php   ...
+5.95s fill  Username                             +6.06s fill Password
+6.46s fill  Email                                +6.09s click button "Add User"
+6.48s click button "Add User"                           → POST user-new.php 302 ✅
       → (KHÔNG có request nào) ❌                        → GET users.php?update=add
+6.54s expect "New user created." → chờ 5s → fail
```

**Kết luận chắc chắn rút ra:** server không hề nhận được form → không phải do server chậm, không phải do locator message sai.

### 12.4 Quy trình điều tra (áp dụng được cho mọi lỗi)

```
1. HIỆN TƯỢNG     Test nào fail? Trên browser nào? Có pattern gì (liền nhau, cùng file)?
       ↓
2. BẰNG CHỨNG     Thông báo lỗi → snapshot trang lúc fail → trace (thao tác + network)
       ↓
3. GIẢ THUYẾT     Giải thích được TẤT CẢ bằng chứng chưa? So sánh với test pass xem khác gì?
       ↓
4. SỬA + KIỂM CHỨNG   Sửa trong POM → chạy lặp nhiều lần (--repeat-each)
       ↓
5. ĐIỀU CHỈNH     Còn fail? → có bằng chứng mới → quay lại bước 2, sẵn sàng sửa giả thuyết
```

Trong session, lỗi Add User đã đi qua vòng lặp này **3 lần** (chi tiết ở bảng "Bài học" trong [mục 9.3](#93-đọc-ra-được-gì-từ-kết-quả--và-kết-luận-cuối-cùng)):

```
Vòng 1: không có POST          → waitForPageLoad()          → vẫn fail (local Firefox)
Vòng 2: chữ bị nối vào Password → fillAndVerify()            → local pass ✅, nhưng CI vẫn fail ❌
Vòng 3: ô đã đúng mà vẫn không POST → đo timing + đọc mã nguồn → waitForAddUserFormReady()
```

### 12.5 Hai kỹ thuật điều tra dùng ở vòng 3

**(1) Đo thời điểm trên NHIỀU trace, không chỉ một.** Một trace chỉ cho thấy "chuyện gì đã xảy ra lần này". So sánh thời điểm giữa các sự kiện trên **tất cả** lần fail mới lộ ra quy luật:

| | zxcvbn tải mất | Bấm "Add User" so với lúc zxcvbn tải xong |
|---|---|---|
| Firefox (8 test) | 209–418ms | sớm hơn 80–292ms |
| WebKit (6 test) | 635–870ms | sớm hơn 484–762ms |

**14/14** lần fail đều bấm submit **trước** khi zxcvbn tải xong → không thể là trùng hợp.

Một chi tiết nhỏ nhưng quan trọng: trong trace, `waitForPageLoad()` trả về **đúng cùng mili-giây** với lúc request `zxcvbn.min.js` bắt đầu → chứng minh chờ `load` không bao giờ đủ.

**(2) Đọc mã nguồn JavaScript của chính trang đang test.** File JS của WordPress là tài nguyên tĩnh công khai, tải về đọc được (đúng phiên bản `?ver=7.0.7` mà trang đang dùng). Đọc `user-profile.js` thấy ngay `generatePassword()` chờ zxcvbn rồi mới điền mật khẩu + focus — xác nhận cả **thứ tự khởi tạo** lẫn **thủ phạm giành focus** ở vòng 2. Đọc mã nguồn còn giúp tìm ra **dấu hiệu để chờ** (ô Password có giá trị).

**Giới hạn cần nói rõ:** đã xác nhận chắc chắn về **thời điểm** (14/14) và **thứ tự khởi tạo** (mã nguồn). Còn chính xác dòng code nào chặn submit khi form chưa khởi tạo thì **chưa xác định được 100%** — trace không ghi lại JS chạy bên trong trình duyệt. Cách sửa nhắm vào điều đã xác nhận (chờ khởi tạo xong), nên vẫn đúng dù chưa biết chi tiết cuối cùng đó.

---

## 13. Race condition và 3 lỗi flaky đã sửa

### 13.1 Race condition là gì?

**Race condition** (điều kiện tranh chấp) = kết quả phụ thuộc vào **thứ tự/thời điểm** hai việc chạy song song. Ở đây hai "người" cùng thao tác trên một trang:

```
Playwright (test)                     JavaScript của WordPress
──────────────────                    ────────────────────────
fill Username ─┐                      (đang tải script...)
               │                      setTimeout → focus ô khác  ← chen vào giữa!
fill Email  ───┘
click Add User
```

Nếu JS của WordPress chạy **trước** hoặc **sau** thao tác của test → không sao. Nếu chạy **chen vào giữa** → lỗi. Vì thời điểm phụ thuộc tốc độ máy, nên:
- **Lúc pass lúc fail** (flaky).
- **Chromium nhanh** → ít gặp; **Firefox/WebKit trên máy CI chậm hơn** → hay gặp.
- Chạy lại (retry) có khi pass → che giấu lỗi thật.

### 13.2 Cơ chế then chốt: `fill()` gồm 2 bước

```
locator.fill("abc")
   ├─ Bước 1: focus vào ô được chỉ định
   └─ Bước 2: chèn "abc" vào ô ĐANG ĐƯỢC FOCUS   ← không phải "ô được chỉ định"!
```

Nếu giữa bước 1 và bước 2, JS của trang chuyển focus sang ô khác → **chữ bị chèn vào ô khác**. Đây là gốc rễ của lỗi 1 và lỗi 2.

### 13.3 Lỗi 1 — Trang Add User: form chỉ sẵn sàng khi zxcvbn tải xong

**Hiện tượng:** test Add User fail trên Firefox/WebKit chạy CI — bấm "Add User" nhưng không có message nào (cả thành công lẫn lỗi), trace **không có request POST**. Ở local lại pass.

#### Nguyên nhân gốc

Trang `user-new.php` tải hơn 60 file JS. Riêng **`zxcvbn.min.js`** (thư viện đo độ mạnh mật khẩu, ~400KB) được tải theo kiểu `zxcvbn-async`: WordPress chỉ **bắt đầu** tải nó **sau** sự kiện `load`. Form chỉ khởi tạo xong khi file này tải xong — mã nguồn `wp-admin/js/user-profile.js`:

```js
function generatePassword() {
    if ( typeof zxcvbn !== 'function' ) {
        setTimeout( generatePassword, 50 );   // zxcvbn chưa tải -> chờ 50ms thử lại, form CHƯA khởi tạo
        return;
    }
    $pass1.val( $pass1.data( 'pw' ) );        // zxcvbn tải xong -> điền mật khẩu tự sinh vào ô Password
    ...
    $( $pass1 ).trigger( 'focus' );           // ... rồi FOCUS vào ô Password
}
```

Dòng thời gian trên CI (Firefox):

```
click link "Add User" ─► trang tải ─► sự kiện 'load' ─► bắt đầu tải zxcvbn (~200–870ms) ─► generatePassword(): điền mật khẩu + focus
                                          ▲                                                    ▲
                           waitForPageLoad() trả về ở đây                       form chỉ SẴN SÀNG từ đây
                                          │◄──────── test fill + bấm submit rơi vào khoảng này ──►│
```

#### Hai hệ quả của cùng một nguyên nhân

| Thao tác rơi vào lúc | Hậu quả | Phát hiện ở vòng |
|---|---|---|
| Đang `fill` **đúng lúc** `generatePassword()` chạy | Focus bị kéo sang ô Password → chữ **nối vào cuối mật khẩu**, ô đang điền **trống** | Vòng 2 (snapshot) |
| Bấm "Add User" **trước khi** `generatePassword()` chạy | **Form không được gửi đi** dù các ô đã đúng | Vòng 3 (đo timing 14/14) |

#### Vì sao local pass mà CI fail?

Máy CI của GitHub đặt ở nước ngoài, cấu hình yếu → tải zxcvbn mất 200–870ms. Máy local tải nhanh hơn nhiều → lúc test kịp bấm submit thì zxcvbn thường đã tải xong. Lỗi **vẫn tồn tại ở local** nhưng gần như không bao giờ rơi trúng.

> **Bài học:** "chạy local pass" **không** chứng minh lỗi timing đã hết. Sau vòng 2, local đã pass hoàn toàn — nhưng CI vẫn fail vì mới chặn được **một hệ quả**, chưa chạm tới **nguyên nhân gốc**.

#### Cách sửa

```ts
// add-user.page.ts
async clickAddUserButton() {
    await this.addUserButtonLoc.click();
    await this.waitForAddUserFormReady();
}

// Override clickAddUserTab() của DashboardBasePage: vào form qua submenu cũng phải chờ form sẵn sàng.
async clickAddUserTab() {
    await super.clickAddUserTab();
    await this.waitForAddUserFormReady();
}

async waitForAddUserFormReady() {
    await this.waitForPageLoad();
    await expect(this.passwordTextBoxLoc).not.toHaveValue('');   // ô Password đã có mật khẩu tự sinh
}
```

**Vì sao một điều kiện chờ chặn được cả 2 hệ quả?** Trong `generatePassword()`, việc điền mật khẩu và focus chạy **liền nhau trong cùng một lần**. Khi Playwright thấy ô Password có giá trị thì focus cũng đã chuyển xong → không còn gì giành focus nữa, và form đã sẵn sàng để submit.

**Vì sao chờ "ô Password có giá trị" mà không chờ "file zxcvbn tải xong"?** Vì đó là **dấu hiệu nhìn thấy được trên giao diện** — đúng tinh thần Playwright (kiểm tra cái người dùng thấy), không phụ thuộc chi tiết kỹ thuật bên trong (tên file, cách tải). `expect(...).not.toHaveValue('')` tự thử lại tối đa 5s — dư cho mức chậm nhất đã đo (870ms).

**Vì sao `expect` tự "chờ" được?** Các assertion dạng `expect(locator).toXxx()` của Playwright là **web-first assertion**: chúng **không** kiểm tra một lần rồi báo kết quả ngay, mà **liên tục kiểm tra lại** cho tới khi điều kiện đúng hoặc hết timeout (mặc định 5 giây):

```
t=0ms    ô Password = ""          → chưa đúng, thử lại
t=100ms  ô Password = ""          → chưa đúng, thử lại
...
t=870ms  ô Password = "NYAvdF!*…" → ĐÚNG → expect pass, code chạy tiếp
(nếu tới t=5000ms vẫn "" → expect fail, báo lỗi)
```

Vì vậy dùng `expect` làm "điều kiện chờ" vừa **chờ đúng lâu cần thiết** (máy nhanh thì đi tiếp ngay), vừa **báo lỗi rõ ràng** nếu trang thật sự hỏng — tốt hơn hẳn `waitForTimeout(1000)` (chờ cố định, thừa hoặc thiếu). Lưu ý: chỉ `expect(locator)...` / `expect(page)...` mới tự chờ; `expect(giá trị thường).toBe(...)` thì kiểm tra **một lần**.

**Override là gì?** `clickAddUserTab()` gốc nằm ở `DashboardBasePage` — lớp này không có `passwordTextBoxLoc` nên không gọi được `waitForAddUserFormReady()`. Viết lại method **cùng tên** trong `UserPage`: `super.clickAddUserTab()` chạy bản gốc (click submenu), rồi chờ form. Test gọi `userPage.clickAddUserTab()` tự động dùng bản của `UserPage` — không phải sửa file test.

**Lớp bảo vệ thêm — `fillAndVerify()`** (giữ lại từ vòng 2):

```ts
// base.page.ts — điền xong phải kiểm tra, sai thì điền lại
async fillAndVerify(locator: Locator, value: string) {
    await expect(async () => {
        await locator.fill(value);
        await expect(locator).toHaveValue(value, { timeout: 1000 });
    }).toPass({ timeout: 10000 });
}
// add-user.page.ts: fillUserName / fillEmail / fillFirstName / fillLastName / fillPassword đều dùng fillAndVerify
```

**Ô Password bị nối thêm chữ có sao không?** Không: test không điền Password thì không quan tâm mật khẩu là gì; test có điền thì `fillPassword()` dùng `fill()` **ghi đè toàn bộ** ô.

**Trạng thái:** ✅ commit `e86bb67` **đã được xác nhận trên CI**: chạy tay workflow với lựa chọn `all` — cả chromium, firefox, webkit đều pass (lần chạy trước đó: 8 test fail trên firefox, 6 trên webkit).

### 13.4 Lỗi 2 — Trang login: mật khẩu đè vào ô Username

**Hiện tượng** (phát hiện khi chạy local Firefox): `afterEach` login lại admin bị treo. Snapshot:

```yaml
- textbox "Username or Email Address": StrongPass@BetterBytesAcademy   ← mật khẩu nằm trong ô Username!
```

**Nguyên nhân:** sau khi trang login tải ~200ms, WordPress tự **focus + bôi đen ô Username** (hàm `wp_attempt_focus`). Nếu rơi đúng lúc Playwright đang `fill` ô Password → mật khẩu **đè lên** (vì ô Username đang bị bôi đen) → login sai.

**Cách sửa — điền hết rồi mới kiểm tra hết:**

```ts
async login(userName: string, password: string) {
    await expect(async () => {
        await this.userNameLogin.fill(userName);
        await this.passwordLogin.fill(password);
        await expect(this.userNameLogin).toHaveValue(userName, { timeout: 1000 });
        await expect(this.passwordLogin).toHaveValue(password, { timeout: 1000 });
    }).toPass({ timeout: 10000 });
    await this.loginButton.click();
}
```

### 13.5 Kiểm tra từng ô hay kiểm tra cả form? (dễ nhầm)

Câu hỏi đã đặt ra trong session: *"sao `login()` không dùng `fillAndVerify()`?"*

**Nếu dùng `fillAndVerify()` cho từng ô:**

```
fillAndVerify(Username, "admin")
   fill → kiểm tra Username = "admin" ✅ → XONG, không bao giờ quay lại

fillAndVerify(Password, "pass")
   fill → WordPress kéo focus về Username → "pass" đè vào Username
   kiểm tra Password = "" ❌ → điền lại → Password = "pass" ✅ → XONG

click Log In → Username = "pass" ❌ → login sai
```

Ô Username **đã kiểm tra xong trước khi bị hỏng** → không ai phát hiện.

**Kiểm tra cả form (cách đang dùng):**

```
Vòng 1: fill Username, fill Password (bị đè sang Username)
        kiểm tra Username = "pass" ❌ → thử lại CẢ CỤM
Vòng 2: fill Username, fill Password (WordPress đã focus xong, không giành nữa)
        kiểm tra cả 2 ✅ → click Log In
```

**Vậy sao trang Add User lại dùng từng ô được?** Vì **ô bị giành focus khác nhau**:

| | Trang login | Trang Add User |
|---|---|---|
| JS kéo focus về ô nào | **Username** — ô test **cần đúng giá trị** | **Password** — ô mật khẩu tự sinh |
| Ô bị hỏng | Username — ô **đã kiểm tra trước đó** | Password — bị nối chữ, nhưng vô hại (xem 13.3) |
| Ô đang điền | — | Bị **trống** → `fillAndVerify` của chính ô đó phát hiện ngay ✅ |
| Kết luận | Phải kiểm tra **cả form** | Kiểm tra **từng ô** là đủ |

**Quy tắc:** form nhiều ô rồi submit → cần đảm bảo **trạng thái cuối cùng của cả form** đúng ngay trước khi submit. "Đúng từng ô tại thời điểm vừa điền" là chưa đủ nếu các ô có thể làm hỏng nhau.

**Có nên tách thành helper chung `fillAllAndVerify()`?** Hiện chỉ `login()` cần → viết trực tiếp là đủ. Theo nguyên tắc **"rule of three"** ([04-utils-design.md](../project-documentations/04-utils-design.md)): chỉ tách helper chung khi có 2–3 nơi cần dùng.

### 13.6 Lỗi 3 — Menu Howdy / Log Out (hover)

**Hiện tượng:** `account-permissions` flaky trên WebKit. Log lỗi:

```
- waiting for getByRole('menuitem', { name: 'Log Out' })
  - element is not stable        ← menu đang trong hiệu ứng mở ra
- retrying click action
  - element is not visible       ← menu đã đóng lại
- retrying click action ...      ← chờ mãi tới hết timeout
```

**Nguyên nhân:** menu con của "Howdy" mở bằng JS `hoverIntent` (trong `admin-bar.js`): có **độ trễ + hiệu ứng**, và chỉ mở khi chuột **đi vào** phần tử. Code cũ tách rời `hover()` rồi `click()` → click lúc menu chưa mở xong / đã đóng → `click()` chỉ **chờ** menu hiện lại, nhưng không ai hover lại → treo.

**Lần sửa 1 (chưa đúng):**

```ts
await expect(async () => {
    await this.hoverProfileLoc.hover();
    await this.logoutLoc.click({ timeout: 2000 });
}).toPass({ timeout: 15000 });
```

Vẫn fail: *"Timeout 15000ms exceeded while waiting on the predicate"*. Vì sao thử lại mà không ăn thua? **Chuột đã nằm sẵn ở đúng vị trí đó** → `hover()` lần 2, 3... đưa chuột tới chỗ cũ → trình duyệt **không phát sinh sự kiện "đi vào" mới** (`mouseover`/`mouseenter`) → JS không biết để mở menu.

**Lần sửa 2 (đúng):** mỗi lần thử, **đưa chuột ra ngoài trước**:

```ts
async logout() {
    await expect(async () => {
        await this.page.mouse.move(0, 0);              // đưa chuột ra góc trang
        await this.hoverProfileLoc.hover();            // → tạo lại sự kiện "đi vào"
        await this.logoutLoc.click({ timeout: 2000 }); // click hụt → báo lỗi sau 2s, không treo
    }).toPass({ timeout: 15000 });
}
```

> **Bài học:** khi thử lại, phải thử lại **từ bước tạo ra trạng thái cần thiết**, không chỉ lặp lại thao tác cuối. Ở đây trạng thái cần thiết là "chuột vừa đi vào menu", nên phải đưa chuột ra rồi vào lại.

**Phương án dự phòng** (chưa dùng): đọc `href` của link Log Out (link luôn có trong DOM, chỉ bị ẩn) rồi `page.goto(href)` — ổn định 100% vì không hover, nhưng không còn thao tác như người dùng thật. Chấp nhận được vì logout ở đây chỉ là bước chuẩn bị, không phải chức năng đang test.

### 13.7 `expect(...).toPass()` hoạt động thế nào?

```ts
await expect(async () => {
    // khối code: nếu có lệnh nào throw lỗi (fill/click/expect fail) → coi như "chưa pass"
}).toPass({ timeout: 10000 });
```

- Chạy khối code → nếu **có lỗi** → chờ một chút → **chạy lại từ đầu khối** → lặp tới khi pass hoặc hết `timeout`.
- **Timeout bên trong phải ngắn** (`{ timeout: 1000 }` cho `toHaveValue`, `{ timeout: 2000 }` cho `click`): nếu để mặc định (5s cho `expect`, cả phần test còn lại cho `click`), một vòng thử có thể ăn hết thời gian → không kịp thử lại lần nào.
- Khác với `retries` trong config: `retries` chạy lại **cả test** từ đầu; `toPass` chỉ chạy lại **một cụm thao tác** ngay tại chỗ.

### 13.8 Nguyên tắc chung (đã ghi vào [01-website-feature.md §4](../project-documentations/01-website-feature.md#4-lưu-ý-chung-về-timing-race-condition))

1. Trước khi thao tác, chờ **dấu hiệu nhìn thấy được** cho biết JS của trang đã khởi tạo xong (VD: ô Password có mật khẩu tự sinh) — **không chỉ chờ `load`**. Tìm dấu hiệu bằng cách đọc mã nguồn JS của trang hoặc quan sát trace.
2. Thao tác xong phải **kiểm tra kết quả thật**, sai thì **thử lại** bằng `toPass()` — không giả định thao tác luôn thành công.
3. Thử lại **từ bước tạo ra trạng thái cần thiết** + timeout ngắn cho từng thao tác bên trong.
4. Form nhiều ô rồi submit → kiểm tra **trạng thái cuối cùng của cả form** nếu các ô có thể làm hỏng nhau.
5. **Không** dùng `page.waitForTimeout(3000)` (chờ cố định): máy nhanh thì phí thời gian, máy chậm thì vẫn có thể chưa đủ.
6. Bọc cách xử lý trong **POM** (`login()`, `logout()`, `fillAndVerify()`) → mọi test tự động được bảo vệ, không phải sửa từng file test.

---

## 14. Timeout trong Playwright

### 14.1 Các tầng timeout

| Tầng | Mặc định | Áp dụng cho | Ví dụ trong session |
|---|---|---|---|
| **Test timeout** | 30s | Toàn bộ 1 test **gồm cả `beforeEach`/`afterEach`** | Nâng lên 60s cho `account-permissions` |
| **Expect timeout** | 5s | Mỗi lệnh `expect(...)` tự chờ tối đa bao lâu | `toHaveValue(..., { timeout: 1000 })` trong `toPass` |
| **Action timeout** | Không giới hạn riêng (bị chặn bởi test timeout) | `click`, `fill`, `hover`... | `click({ timeout: 2000 })` trong `logout()` |
| **`toPass` timeout** | — (phải tự đặt) | Tổng thời gian thử lại một cụm | `toPass({ timeout: 15000 })` |
| **Job timeout (CI)** | — | Cả job GitHub Actions | `timeout-minutes: 60` |

### 14.2 Bẫy: timeout của test **tính chung** với hook

```
|<──────────────────── 30 giây ────────────────────>|
| beforeEach | test body ............ | afterEach ...✂ bị cắt!
             (thử logout mất 15s)       (login admin → tìm → xóa user)
```

Đã gặp thực tế: test body tốn ~15s thử logout → `afterEach` không đủ thời gian → *"Test timeout of 30000ms exceeded while running afterEach hook"* → **user vừa tạo không được xóa**, sót lại trên site.

### 14.3 Các cách tăng timeout

| Cách viết | Phạm vi | Khi nào dùng |
|---|---|---|
| `test.describe.configure({ timeout: 60000 })` (đầu `describe`) | Mọi test trong `describe` | ✅ Đang dùng cho `account-permissions` — cả 2 test đều dài |
| `test.slow()` (trong 1 test) | Riêng test đó, **nhân 3** (30s → 90s) | 1–2 test chậm bất thường |
| `test.setTimeout(60000)` (trong 1 test) | Riêng test đó | Cần con số cụ thể cho 1 test |
| `timeout` trong `playwright.config.ts` | **Toàn bộ** project | ❌ Không nên — test hỏng thật sẽ mất lâu hơn mới báo lỗi |

Vị trí đặt `configure`:

```ts
test.describe("ACCOUNT-Account", () => {
    test.describe.configure({ timeout: 60000 }); // luồng phân quyền dài, cần nhiều thời gian hơn mặc định 30s

    let userPage: UserPage;
    // ...
```

**Vì sao 60s?** Lần pass trên CI đã đo tới ~22s — sát 30s. Luồng: tạo user → logout → login user mới → kiểm tra menu → logout → login admin → tìm → xóa user. Nếu logout phải thử lại thêm vài giây là vượt.

---

## 15. CI chọn browser và chạy tay

### 15.1 Mục tiêu

- **Push / Pull Request** → tự động chạy **chỉ chromium** (nhanh: 26 test thay vì 78).
- **Khi cần** → chạy **đủ 3 browser** mà **không cần commit** gì thêm.

### 15.2 `workflow_dispatch` — chạy workflow bằng tay

```yaml
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
  workflow_dispatch:                 # cho phép chạy tay từ tab Actions
    inputs:
      browsers:
        description: 'Browser cần chạy'
        type: choice                 # ô chọn (dropdown)
        options: [chromium, all]
        default: chromium
```

`workflow_dispatch` thêm nút **Run workflow** trên GitHub. `inputs` khai báo các ô nhập/chọn hiện ra khi bấm nút đó.

### 15.3 Biến `BROWSERS` và giá trị mặc định

```yaml
jobs:
  test:
    env:
      BROWSERS: ${{ inputs.browsers || 'chromium' }}
```

- Chạy tay → `inputs.browsers` = giá trị bạn chọn (`chromium` hoặc `all`).
- Chạy do push/PR → **không có input** → `inputs.browsers` rỗng → `|| 'chromium'` lấy giá trị mặc định.

`||` ở đây nghĩa là "nếu vế trái rỗng thì lấy vế phải" — giống `||` trong JavaScript.

### 15.4 Lệnh nhiều dòng `run: |` và `if` của bash

```yaml
- name: Install Playwright Browsers
  run: |
    if [ "$BROWSERS" = "all" ]; then
      npx playwright install --with-deps            # cài cả 3
    else
      npx playwright install --with-deps chromium   # chỉ cài chromium
    fi
- name: Run Playwright tests
  run: |
    if [ "$BROWSERS" = "all" ]; then
      npx playwright test                           # chạy cả 3 project
    else
      npx playwright test --project=chromium        # chỉ project tên 'chromium'
    fi
```

- `run: |` — dấu `|` cho phép viết **nhiều dòng** lệnh shell.
- Máy CI là Ubuntu nên dùng cú pháp **bash**: `if [ ... ]; then ... else ... fi`.
- `--project=chromium` — tên phải khớp **chính xác** `name: 'chromium'` trong `playwright.config.ts`.
- Phải sửa **cả 2 bước**: chỉ sửa bước chạy test thì CI vẫn mất thời gian tải 2 browser không dùng.
- Đọc input qua biến môi trường `$BROWSERS` (thay vì nhét thẳng `${{ inputs.browsers }}` vào script) là cách an toàn được khuyến nghị — giá trị được truyền như dữ liệu, không bị ghép thành code shell.

### 15.5 Cách chạy tay

1. Repo trên GitHub → tab **Actions**.
2. Cột trái: chọn workflow **Playwright Tests**.
3. Bên phải: nút **Run workflow** → chọn branch → ô **Browser cần chạy**: `chromium` hoặc `all` → **Run workflow**.

| Nút | Làm gì | Đổi được browser? |
|---|---|---|
| **Run workflow** | Tạo một lần chạy **mới** với input bạn chọn | ✅ |
| **Re-run jobs** (ở một lần chạy cũ) | Chạy lại **đúng cấu hình cũ** | ❌ |

**Điều kiện:** nút **Run workflow** chỉ xuất hiện khi file workflow có `workflow_dispatch` **đã nằm trên nhánh mặc định** (`main`) — tức là phải commit & push thay đổi này 1 lần.

### 15.6 Đánh đổi

| | Chỉ chromium | Cả 3 browser |
|---|---|---|
| Số test | 26 | 78 |
| Thời gian (ước tính) | ~5–7 phút | ~18 phút |
| Bắt được lỗi riêng Firefox/WebKit | ❌ | ✅ |

Chính các lỗi race condition ở [mục 13](#13-race-condition-và-3-lỗi-flaky-đã-sửa) **chỉ lộ ra trên Firefox/WebKit** → nếu chỉ chạy chromium sẽ không phát hiện. Nên: trước khi merge thay đổi lớn (nhất là sửa các trang login/Add User/admin bar) → chạy tay với `all`, hoặc `npx playwright test` ở local.

### 15.7 Cách khác (không chọn): lọc project trong `playwright.config.ts`

```ts
projects: [
  { name: 'chromium', ... },
  ...(process.env.CI ? [] : [ { name: 'firefox', ... }, { name: 'webkit', ... } ]),
],
```

Không chọn vì: trộn logic CI vào config, vẫn phải sửa bước cài browser trong workflow, và không cho chọn lúc chạy tay.

---

## 16. GitHub CLI (`gh`)

### 16.1 Là gì, khác `git` thế nào?

| | `git` | `gh` (GitHub CLI) |
|---|---|---|
| Làm việc với | **Mã nguồn**, lịch sử commit | **Các tính năng của trang GitHub** |
| Ví dụ | commit, branch, push, pull | Secrets, Actions (CI), Pull Request, Issue |
| Gắn với GitHub? | Không — dùng được với GitLab, Bitbucket... | Chỉ GitHub |

Nói đơn giản: `git push` đẩy code lên GitHub; những việc bạn làm **trên giao diện web** sau đó (xem CI, tải report, khai báo secrets, tạo PR...) thì `gh` cho làm bằng lệnh.

### 16.2 Tương ứng với các việc đã làm trên web trong session

| Việc đã làm trên web | Lệnh `gh` |
|---|---|
| Khai báo secret | `gh secret set BASE_URL` (lệnh hỏi giá trị, không lưu vào lịch sử terminal) |
| Kiểm tra secrets đã có | `gh secret list` |
| Xem các lần chạy CI | `gh run list` |
| Theo dõi CI đang chạy | `gh run watch` |
| Xem log các bước fail | `gh run view <run-id> --log-failed` |
| Tải report | `gh run download <run-id> -n playwright-report -D ci-report` |
| Chạy lại job fail | `gh run rerun <run-id> --failed` |
| Chạy tay workflow, chọn browser | `gh workflow run playwright.yml -f browsers=all` |

Khác: `gh pr create` (tạo PR), `gh pr checkout 12` (lấy code PR về máy), `gh repo view --web` (mở repo trên trình duyệt).

### 16.3 Cài đặt — và vấn đề máy công ty

Cài thông thường trên Windows:

```bash
winget install --id GitHub.cli
gh auth login     # đăng nhập 1 lần: GitHub.com → SSH → đăng nhập qua trình duyệt
```

**Máy công ty:** gói cài (MSI) ghi vào `C:\Program Files\` → **cần quyền Administrator** → không tự cài được.

| Hướng | Đánh giá |
|---|---|
| Bản **portable** (file `.zip`, giải nén vào thư mục user) | Kỹ thuật thì không cần admin, **nhưng** việc cần admin để cài thường là **chính sách bảo mật** của công ty — "lách" có thể vi phạm quy định. Thêm nữa, `gh` lưu **token truy cập GitHub** trên máy → IT thường muốn biết. Chỉ làm khi chắc chắn được phép |
| **Nhờ IT cài** | ✅ Cách chuẩn. Gửi IT: tên phần mềm (GitHub CLI, GitHub Inc., https://cli.github.com/), lệnh cài, mục đích sử dụng |
| **Không dùng `gh`** | ✅ Hoàn toàn đủ — mọi việc làm được trên web (tab Actions, Settings → Secrets, mục Artifacts) |

**Phân biệt:** "làm được về mặt kỹ thuật" ≠ "nên làm". Với máy công ty, chính sách bảo mật được ưu tiên.

`gh` là **tùy chọn**: không có nó vẫn làm được mọi thứ, chỉ thêm vài bước thủ công (tải report, chụp màn hình log...).

---

## 17. Chi phí GitHub Actions và rủi ro repo public

### 17.1 Chạy CI có tốn tiền không?

Phụ thuộc **repo public hay private** và **loại máy chạy (runner)**:

| Trường hợp | Chi phí |
|---|---|
| **Repo public** + runner tiêu chuẩn (`ubuntu-latest`...) — **trường hợp của dự án này** | ✅ Miễn phí, không giới hạn phút |
| Repo **private**, gói GitHub Free | Miễn phí **2.000 phút/tháng**, đầu tháng đặt lại về 0 |
| Repo private, gói Pro / Team | 3.000 phút/tháng |
| **Larger runner** (máy cấu hình cao, phải tự chọn trong workflow) | Luôn tính phí, kể cả repo public |

- Hết quota mà tài khoản **chưa khai báo phương thức thanh toán** → GitHub chỉ **chặn không cho chạy tiếp** tới tháng sau, **không tự trừ tiền**.
- Ước tính nếu chuyển private: mỗi lần chạy chromium ~5–7 phút → 2.000 phút ≈ 300 lần chạy/tháng.
- Đầu năm 2026 GitHub giảm giá runner trả phí, giữ nguyên quota miễn phí. Giá có thể thay đổi → muốn chắc chắn, xem **Settings → Billing** của tài khoản. Nguồn: [GitHub Docs — Actions billing](https://docs.github.com/billing/managing-billing-for-github-actions/about-billing-for-github-actions).

**Cách kiểm tra repo public hay private:** trang repo trên GitHub có nhãn **Public**/**Private** cạnh tên repo; hoặc gọi API công khai (không cần đăng nhập):

```bash
curl -s https://api.github.com/repos/<user>/<repo> | grep '"visibility"'
# "visibility": "public"     ← repo private sẽ trả về lỗi 404 (người ngoài không thấy được)
```

### 17.2 ⚠️ Rủi ro quan trọng hơn chi phí: lộ mật khẩu qua trace

Repo public = **ai cũng xem được** code, log CI, và **tải được artifact** `playwright-report`.

Khi phân tích report trong session, phát hiện **trace ghi lại nguyên văn mọi giá trị được `fill()`**, kể cả tài khoản và mật khẩu admin:

```
fill  "Username or Email Address"  value: "betterbytes.academy.admin"
fill  "Password"                   value: "StrongPass@..."
```

| GitHub che secrets ở đâu? | |
|---|---|
| Trong **log** của bước chạy | ✅ Tự thay thành `***` |
| Bên trong **file artifact** (report, trace) | ❌ **Không che** |

Config đang có `trace: 'on-first-retry'` → mỗi khi có test được retry trên CI, report upload lên chứa trace kèm mật khẩu.

**Đánh giá mức độ** — tùy tài khoản này là gì:
- Tài khoản **thực hành dùng chung** của khóa học, mật khẩu không dùng ở đâu khác → rủi ro thấp.
- Tài khoản **riêng**, hoặc mật khẩu dùng lại ở nơi khác → cần xử lý ngay.

**Các hướng xử lý** (chưa áp dụng — vẫn nằm trong [mục 11](#11-việc-còn-tồn-đọng)):

| Hướng | Ưu | Nhược |
|---|---|---|
| Chuyển repo sang **private** (Settings → General → Danger Zone) | Triệt để nhất | CI tính vào quota 2.000 phút/tháng (vẫn đủ dùng) |
| **Xóa artifact cũ** có trace (mở lần chạy → Artifacts → thùng rác) | Gỡ phần đã lộ | Không ngăn được lần sau |
| **Tắt trace trên CI** (`trace: process.env.CI ? 'off' : 'on-first-retry'`) hoặc giảm `retention-days` | Ngăn lộ về sau | Khi CI fail sẽ **khó điều tra** — chính trace đã giúp tìm ra lỗi zxcvbn |
| **Đổi mật khẩu** admin rồi cập nhật secret `ADMIN_PASSWORD` + `.env` | Vô hiệu hóa mật khẩu đã lộ | Không ngăn được lần sau |

> **Bài học:** "secrets đã được che" chỉ đúng với **log**. Mọi thứ test **ghi lại** (trace, screenshot, video, report) đều có thể chứa dữ liệu nhạy cảm — cần biết artifact được lưu ở đâu và **ai** tải được.

---

## 18. Viết commit message tốt

### 18.1 Vì sao không nên dùng lại message cũ?

Trong session, suýt có 2 commit cùng message:

```
ee78d77 fix: resolve flaky test     ← 10 file: fillAndVerify, login(), logout(), timeout, CI chọn browser, docs
??????? fix: resolve flaky test     ← 1 file: chờ form Add User khởi tạo xong (zxcvbn)
```

Git **không chặn** message trùng, nhưng sau này xem `git log` sẽ không phân biệt được commit nào sửa gì → muốn tìm lại thay đổi "chờ zxcvbn" (để revert, để tham khảo) phải mở từng commit ra xem.

### 18.2 Cấu trúc một commit message

```bash
git commit -m "fix: wait for Add User form to finish initializing before interacting" -m "WordPress only fills the generated password and focuses the Password field after zxcvbn.min.js loads (after the 'load' event). Submitting before that does not send the form, causing Add User tests to fail on CI (firefox/webkit)."
```

| Phần | Viết thế nào | Ví dụ |
|---|---|---|
| `-m` thứ nhất = **tiêu đề** | Ngắn (~50–72 ký tự), nói commit **làm gì**, **thể mệnh lệnh** ("wait for…", không viết "waited for…" / "waiting…") | `fix: wait for Add User form to finish initializing before interacting` |
| `-m` thứ hai = **thân** (tùy chọn) | Giải thích **vì sao** — bối cảnh, nguyên nhân | Đoạn giải thích zxcvbn ở trên |

Mẹo kiểm tra tiêu đề: đọc thử *"Nếu áp dụng commit này, nó sẽ … [tiêu đề]"* — phải thành câu hợp lý.

### 18.3 Conventional Commits (chuẩn dự án đang dùng)

```
<loại>(<phạm vi — tùy chọn>): <mô tả>
```

| Loại | Dùng khi | Ví dụ |
|---|---|---|
| `feat` | Thêm tính năng / thay đổi lớn | `feat: Refactor project structure into src/, add path aliases, update docs and CI` (đã dùng) |
| `fix` | Sửa lỗi | `fix: wait for Add User form to finish initializing before interacting` (đã dùng) |
| `docs` | Chỉ sửa tài liệu | `docs: explain zxcvbn root cause in website feature notes` |
| `test` | Thêm/sửa test | `test: add storefront category tests` |
| `ci` | Sửa cấu hình CI | `ci: run chromium only on push, allow manual run for all browsers` |
| `refactor` | Đổi cấu trúc code, không đổi hành vi | `refactor: move logout into DashboardBasePage` |

**Phạm vi** (trong ngoặc) giúp rõ hơn khu vực bị ảnh hưởng: `fix(users): ...`, `ci(workflow): ...`.

Lợi ích: lọc nhanh lịch sử (`git log --oneline --grep "^fix"`), và nhiều công cụ tự sinh changelog dựa vào chuẩn này.

### 18.4 Không sửa message của commit đã push

Commit `ee78d77` có message chưa tốt, nhưng **đã push lên GitHub** → **không nên sửa**:
- Sửa message = tạo commit mới thay thế (`git commit --amend` / `rebase`) → phải `git push --force` → **viết lại lịch sử chung**, ai đã lấy code về sẽ bị lệch.
- Commit **chưa push** thì sửa thoải mái: `git commit --amend -m "message mới"`.

→ Từ giờ chỉ cần đặt message rõ ràng hơn, không cần quay lại sửa commit cũ.

---

## 19. Chạy song song nhiều worker (đã cân nhắc, giữ 1 worker)

> **Quyết định:** sau khi phân tích, **giữ nguyên 1 worker trên CI**. Mục này ghi lại phân tích để dùng khi muốn tăng tốc sau này.

### 19.1 Worker là gì, cấu hình ở đâu?

**Worker** = một tiến trình chạy test riêng (có trình duyệt riêng). N worker = N test chạy **cùng lúc**.

```ts
// playwright.config.ts
fullyParallel: true,                       // test trong CÙNG 1 file cũng được chia cho nhiều worker
workers: process.env.CI ? 1 : undefined,   // CI: 1 worker (tuần tự) | local: Playwright tự chọn theo số CPU
```

| Giá trị `workers` | Nghĩa |
|---|---|
| `1` | Chạy tuần tự từng test (hiện tại trên CI) |
| `2`, `3`... | Số worker cố định |
| `'50%'` | Theo phần trăm số nhân CPU của máy |
| `undefined` | Mặc định của Playwright (khoảng một nửa số nhân CPU) — đang dùng ở local |

Máy CI `ubuntu-latest` cho repo public có 4 vCPU → nếu tăng thì **2 worker là mức khởi đầu an toàn**; 4 worker dễ làm máy quá tải khi chạy cả 3 browser.

### 19.2 Bẫy chính: dữ liệu test trùng nhau khi chạy song song

Username/email đang sinh bằng `Date.now()` (thời gian tính theo **mili-giây**):

```ts
// src/utils/random-data.util.ts
return `${prefix}-${Date.now()}`;              // E101-tuyen-1791704015141
// tests/dashboard/users/add-user.spec.ts (5 chỗ)
const testUserName = `dup-${Date.now()}`;
```

```
1 worker:   test A gọi Date.now() ── (vài giây sau) ── test B gọi Date.now()   → luôn khác nhau ✅

2 worker:   worker 1: test A gọi Date.now() = 1791704015141
            worker 2: test B gọi Date.now() = 1791704015141   ← CÙNG mili-giây!
            → 2 test tạo cùng username → test sau nhận lỗi "This username is already registered" ❌
```

Đây chính là kiểu **lỗi flaky** rất khó điều tra: hiếm khi xảy ra, và thông báo lỗi trông như lỗi nghiệp vụ.

**Cách sửa (khi muốn tăng worker):** ghép thêm phần ngẫu nhiên, đặt trong `utils/` đúng nguyên tắc bọc thư viện:

```ts
// src/utils/random-data.util.ts
export function uniqueSuffix(): string {
    return `${Date.now()}-${faker.string.alphanumeric(4)}`;   // VD: 1791704015141-x7Kq
}

export function randomUserName(prefix: string = 'E101-tuyen'): string {
    return `${prefix}-${uniqueSuffix()}`;
}
// randomEmail tương tự; 5 chỗ `dup-${Date.now()}` trong add-user.spec.ts đổi thành `dup-${uniqueSuffix()}`
```

### 19.3 Các yếu tố khác đã kiểm tra

| Yếu tố | Đánh giá |
|---|---|
| Nhiều worker cùng login **một tài khoản admin** | ✅ WordPress cho phép nhiều phiên đăng nhập cùng lúc; mỗi worker có trình duyệt riêng, logout ở worker này chỉ hủy phiên của chính nó |
| `afterEach` tìm và xóa user | ✅ Tìm theo đúng username vừa tạo — với username duy nhất thì không xóa nhầm user của worker khác |
| Các lỗi timing (zxcvbn, focus, hover) | ✅ Đã xử lý bằng **chờ dấu hiệu sẵn sàng** — không phụ thuộc tốc độ, nên vẫn đúng khi máy chậm hơn do chạy song song |
| Tải lên website practice (site dùng chung) | ⚠️ Request song song gấp đôi → site có thể phản hồi chậm hơn → nên tăng dần và quan sát |

### 19.4 Đánh đổi

| | 1 worker (đang dùng) | 2 worker |
|---|---|---|
| Thời gian chạy `all` (78 test) | ~18 phút | ước tính ~9–11 phút |
| Thời gian chạy chromium (26 test) | ~5–7 phút | ước tính ~3–4 phút |
| Điều kiện | Không cần gì thêm | Phải sửa dữ liệu test cho duy nhất (19.2) |
| Rủi ro | Thấp nhất | Site chịu tải gấp đôi; cần chạy CI vài lần để kiểm tra |

**Nếu sau này muốn tăng worker**, làm theo thứ tự:
1. Thêm `uniqueSuffix()` + sửa 5 chỗ trong `add-user.spec.ts`.
2. Đổi `workers: process.env.CI ? 2 : undefined`.
3. `npm run typecheck` + chạy thử local.
4. Commit, push, chạy tay CI `all` 2–3 lần.
5. Cập nhật [04-utils-design.md](../project-documentations/04-utils-design.md) và phần CI trong README.

> **Bài học:** tăng tốc độ bằng chạy song song **không chỉ là đổi một con số**. Phải rà lại mọi thứ mà các test **dùng chung** (dữ liệu, tài khoản, website) — thứ gì "chỉ đúng khi chạy tuần tự" sẽ thành lỗi flaky khi chạy song song.
