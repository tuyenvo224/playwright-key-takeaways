# Tổng hợp kiến thức — Session refactor & CI

> Ghi chú học tập tổng hợp lại toàn bộ những gì đã làm, đã phân tích và đã học được trong session làm việc với dự án `playwright-automation-framework`: từ đổi tên dự án, refactor cấu trúc, cập nhật tài liệu, cho tới commit/push và đọc kết quả CI trên GitHub Actions.
>
> Tài liệu **thiết kế** chính thức của dự án vẫn nằm ở [project-documentations/](../project-documentations/). File này là ghi chú để ôn lại và tra cứu.

---

## Mục lục

1. [Tổng quan session](#1-tổng-quan-session)
2. [Đổi tên dự án](#2-đổi-tên-dự-án)
3. [Cấu trúc dự án sau refactor](#3-cấu-trúc-dự-án-sau-refactor)
4. [TypeScript: path alias, typecheck, readonly](#4-typescript-path-alias-typecheck-readonly)
5. [Biến môi trường: `.env`, `.env.example`, `dotenv`](#5-biến-môi-trường-env-envexample-dotenv)
6. [Viết assertion đúng — tránh assertion "luôn pass"](#6-viết-assertion-đúng--tránh-assertion-luôn-pass)
7. [Git: add, commit, push và upstream](#7-git-add-commit-push-và-upstream)
8. [GitHub Actions (CI)](#8-github-actions-ci)
9. [Đọc kết quả CI](#9-đọc-kết-quả-ci)
10. [Checklist review trước khi commit](#10-checklist-review-trước-khi-commit)
11. [Việc còn tồn đọng](#11-việc-còn-tồn-đọng)

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
| 6 | Review toàn bộ trước khi commit | Không có lỗi chặn → commit & push |
| 7 | Đọc kết quả CI lần chạy đầu | 68 passed, 3 flaky, 7 failed (Firefox) — chưa tìm ra nguyên nhân |

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
BasePage                    (giữ page, đọc BASE_URL, goto(), waitForPageLoad(), getTitle())
 ├── LoginPage              (form login, login(), getErrorMessage())
 ├── DashboardBasePage      (menu sidebar, logout, heading Dashboard/At a Glance/Activity)
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

---

## 8. GitHub Actions (CI)

### 8.1 Workflow hiện tại

File [.github/workflows/playwright.yml](../.github/workflows/playwright.yml), chạy khi push hoặc tạo Pull Request vào `main`/`master`:

```yaml
steps:
- uses: actions/checkout@v4                 # 1. Lấy code về
- uses: actions/setup-node@v4               # 2. Cài Node.js
- name: Install dependencies
  run: npm ci                               # 3. Cài dependency theo đúng package-lock.json
- name: Type check
  run: npm run typecheck                    # 4. Kiểm tra type  ← THÊM MỚI
- name: Install Playwright Browsers
  run: npx playwright install --with-deps   # 5. Cài 3 trình duyệt + thư viện hệ thống
- name: Run Playwright tests
  run: npx playwright test                  # 6. Chạy test
  env:                                      #    ← THÊM MỚI
    BASE_URL: ${{ secrets.BASE_URL }}
    ADMIN_USERNAME: ${{ secrets.ADMIN_USERNAME }}
    ADMIN_PASSWORD: ${{ secrets.ADMIN_PASSWORD }}
- uses: actions/upload-artifact@v4          # 7. Upload report HTML
  if: ${{ !cancelled() }}                   #    chạy cả khi test fail
```

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

Hoặc dùng GitHub CLI (lệnh sẽ hỏi giá trị, không lưu vào lịch sử terminal):

```bash
gh secret set BASE_URL
```

**Bước 2 — Truyền vào workflow:** khối `env:` ở bước "Run Playwright tests" (xem 8.1).

**Vì sao code không cần sửa gì?**
- Workflow đặt sẵn 3 biến vào `process.env`.
- `dotenv.config()` không tìm thấy `.env` → không đọc gì.
- `dotenv` **không ghi đè** biến đã tồn tại.
→ `process.env.BASE_URL` lấy đúng giá trị từ secret.

**Lưu ý:**
- Tên secret phải **trùng khớp chính xác** (chữ hoa, không dấu cách).
- Khi dán giá trị, không để thừa dấu cách/dòng trống — đây là lỗi hay gặp nhất khiến login fail trên CI dù local chạy đúng.
- Sau khi lưu, GitHub **không cho xem lại** giá trị, chỉ cho ghi đè.
- GitHub tự che secret thành `***` trong log, nhưng đừng `console.log` chúng.
- Pull Request từ **fork** không nhận được secrets → test cần login sẽ fail.

### 8.4 Vì sao bước typecheck đặt **trước** bước cài browser?

Nguyên tắc **fail fast** — kiểm tra rẻ và nhanh chạy trước:
- Typecheck: vài giây.
- Cài 3 trình duyệt + thư viện: 1–2 phút.

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

**Hướng tăng tốc** (chưa áp dụng, cần cân nhắc):
- Trên CI chỉ chạy chromium → giảm khoảng 3 lần thời gian.
- Tăng `workers` lên 2 → cần cẩn thận vì các test dùng chung một tài khoản admin.

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

### 9.3 Đọc ra được gì từ kết quả

1. **68 passed → secrets đúng.** Login qua fixture chạy được trên cả 3 trình duyệt. Nếu secret sai, gần như toàn bộ test đã fail.
2. **7 failed đều là `[firefox]` trong `add-user.spec.ts`**, cùng test đó chromium và webkit đều pass.
3. Các test fail **đứng liền nhau** trong file (dòng 142 → 287), ngay trước đó test ở dòng 127 bị flaky.

**Hai hướng nghi ngờ** (⚠️ **chưa có kết luận** — cần xem thông báo lỗi cụ thể):
- Khác biệt riêng của Firefox với form Add User (cách xử lý ô Email, ô Password tự sinh, hoặc khoảng trắng/xuống dòng trong message không khớp `getByText(...)`).
- Website practice bị chậm/chặn request trong khoảng thời gian đó, lỗi kéo dài sang các test liền sau.

### 9.4 Cách điều tra test fail trên CI

**Cách A — Tải report từ CI** (khuyên dùng, vì dùng đúng lỗi đã xảy ra):

1. Mở lần chạy → cuối trang, mục **Artifacts** → tải `playwright-report`.
2. Giải nén vào một thư mục, ví dụ `ci-report/`.
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

# 2. Chắc chắn .env không bị commit
git check-ignore -v .env

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
```

Ngoài lệnh, cần tự kiểm tra:
- [ ] Các file bị xóa (` D`) đều là cấu trúc cũ đã được chuyển đi, không xóa nhầm.
- [ ] Document (README, `project-documentations/`) khớp với code: cây thư mục, tên class, số dòng tham chiếu, code mẫu, link.
- [ ] Không có tên cũ/sai chính tả còn sót trong document.
- [ ] Assertion trong test thực sự fail được khi hành động không xảy ra (xem [mục 6](#6-viết-assertion-đúng--tránh-assertion-luôn-pass)).

---

## 11. Việc còn tồn đọng

| # | Việc | Mức độ | Ghi chú |
|---|---|---|---|
| 1 | Tìm nguyên nhân **7 test Firefox fail** trong `add-user.spec.ts` | 🔴 Cao | Xem [9.4](#94-cách-điều-tra-test-fail-trên-ci) — tải report CI, xem lỗi + trace |
| 2 | Xử lý **3 test flaky** | 🟠 Trung bình | `account-permissions.spec.ts` (ACC_001 webkit, ACC_002 firefox), `add-user.spec.ts:127` (firefox) |
| 3 | `tests/storefront/home.spec.ts:14` vẫn hardcode URL | 🟡 Thấp | Đề xuất: `toHaveURL(/^https?:\/\/[^/]+\/$/)` |
| 4 | Text `"... - Dev environment"` gắn với môi trường dev | 🟡 Thấp | Cần xem lại nếu chạy trên môi trường khác |
| 5 | 14 dòng code bị comment (`//await expect(page).toHaveURL('https://...')`) | ⚪ Tùy chọn | Trong `add-user.spec.ts`, `login.spec.ts`, fixture — giữ làm ghi chú hoặc xóa cho gọn |
| 6 | Alias `@data/*` chưa có thư mục `src/data/` | ⚪ Tùy chọn | Tạo khi cần dữ liệu test tĩnh |
| 7 | Tăng tốc CI | ⚪ Tùy chọn | Chỉ chạy chromium trên CI, hoặc tăng `workers` (cân nhắc tài khoản admin dùng chung) |
