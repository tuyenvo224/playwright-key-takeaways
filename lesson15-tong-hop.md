# Lesson 15 — Global Setup & Teardown, Build Test Framework (POM) & CI/CD

> Tổng hợp từ slide bài giảng (`lesson-15-pdf/`) và code thực hành (`lesson-15/`)

## Agenda (Nội dung bài học)

1. **Review toàn khoá** (course wrap up)
2. **Global setup & teardown**
3. **Build test framework** (tư duy tổ chức code theo Page Object Model)
4. **CI/CD** với GitHub Actions và GitLab CI
5. **Further learning** — học gì tiếp theo

---

## 1. Global setup & teardown

### 1.1. Khái niệm

| Khái niệm | Ý nghĩa |
|---|---|
| **GlobalSetup** | Chạy **trước khi** tất cả các test chạy, chỉ chạy **MỘT LẦN DUY NHẤT** cho toàn bộ test suite |
| **GlobalTeardown** | Chạy **sau khi** tất cả các test chạy xong, chỉ chạy **MỘT LẦN DUY NHẤT** |

### 1.2. So sánh với Fixture

| | `Fixture` | `Global setup & teardown` |
|---|---|---|
| Tần suất chạy | Chạy lại **mỗi khi 1 test chạy** | Chỉ chạy **1 lần duy nhất** cho cả run |
| Dùng khi nào | Logic cần chạy **riêng cho từng test** (VD: mở trang mới, tạo data riêng cho từng test) | Logic **chỉ cần chạy 1 lần** cho toàn bộ test suite (VD: login 1 lần rồi lưu session dùng chung, seed dữ liệu ban đầu, khởi động 1 service phụ trợ...) |

> Nguyên tắc chọn: nếu logic phải lặp lại cho từng test riêng lẻ → dùng **fixture**; nếu chỉ cần chạy đúng 1 lần trước/sau toàn bộ suite → dùng **global setup/teardown**.

### 1.3. Code thực hành

**`setup/global.setup.ts`** — chạy 1 lần trước tất cả test:

```typescript
import { test as setup } from '@playwright/test';

setup("Setup", async () => {
    console.log("Global setup is running");
});
```

**`setup/global.teardown.ts`** — chạy 1 lần sau khi tất cả test chạy xong:

```typescript
import { test as teardown } from '@playwright/test';

teardown("Teardown", async () => {
    console.log('Global teardown is running');
});
```

**`01-global-test.spec.ts`** — test case ví dụ nằm giữa setup và teardown:

```typescript
import { test } from '@playwright/test';
// Trước khi test chạy: global setup is running

test("Example test", async ({}) => {
    console.log("Example test for global setup & global teardown");
});

// Sau khi test chạy: global teardown is running
```

### 1.4. Khai báo trong `playwright.config.ts`

Global setup/teardown được khai báo như 2 **project** riêng biệt, không phải option đơn giản — nhờ vậy Playwright coi chúng như test thực sự (có thể debug, xem log như 1 test case):

```typescript
projects: [
  {
    name: 'e101-setup',
    testMatch: /global\.setup\.ts/,
    teardown: 'e101-teardown',   // khai báo project nào là teardown tương ứng
  },
  {
    name: 'e101-teardown',
    testMatch: /global\.teardown\.ts/,
  },
  {
    name: 'chromium',
    use: { ...devices['Desktop Chrome'] },
    dependencies: ['e101-setup'], // chromium phụ thuộc setup, phải chạy setup xong mới tới chromium
  },
],
```

Điểm quan trọng:
- Project `e101-setup` có thuộc tính `teardown` trỏ tới tên project teardown tương ứng — Playwright tự biết khi nào cần chạy teardown (sau khi toàn bộ project phụ thuộc vào setup đã chạy xong).
- Project `chromium` (chứa các test thật) khai báo `dependencies: ['e101-setup']` → bắt buộc `e101-setup` phải chạy xong trước, chromium mới được chạy.

### 1.5. Kết quả chạy thực tế (đã verify)

Chạy `01-global-test.spec.ts`, log terminal xác nhận đúng thứ tự thực thi:

```
[e101-setup] > setup/global.setup.ts:3:6 > Setup
Global setup is running
[chromium] > 01-global-test.spec.ts:4:5 > Example test
Example test for global setup & global teardown
[e101-teardown] > setup/global.teardown.ts:3:9 > Teardown
Global teardown is running

3 passed (3.4s)
```

→ Thứ tự chạy đúng như thiết kế: **Setup → Test → Teardown**, và mỗi phần chỉ chạy **đúng 1 lần**, không lặp lại theo từng test.

---

## 2. Course wrap up — Review toàn khoá

Đây là buổi tổng kết cuối khoá, ôn lại toàn bộ lộ trình đã học:

```
Review toàn khoá
   → Build test framework
      → CI/CD với GitHub Actions và GitLab CI
         → Further learning (học gì tiếp theo)
```

Tinh thần chính của buổi wrap-up: không học kiến thức mới đơn lẻ nữa, mà **ráp toàn bộ kiến thức đã học** (locator, POM, fixture, assertion, emulation, screenshot testing, unit test...) lại thành **1 framework test hoàn chỉnh**, sau đó **đưa framework đó chạy tự động trên CI/CD**.

---

## 3. Build Test Framework — Tư duy tổ chức code (Page Object Model)

### 3.1. Tư duy khi build framework

Ý tưởng cốt lõi của **Page Object Model (POM)**: tách code thành nhiều lớp, mỗi lớp đại diện cho **1 khu vực/trang của ứng dụng**, giúp code dễ tái sử dụng, dễ bảo trì.

```
                              ┌── Dashboard ─┐
                              │              ├── login
                Web ──────────┤              ├── dashboard ──┬── product
                │             │              │               └── category
   base.page.ts │             └──────────────┘   dashboard-base.page.ts
                │
                └── Storefront ──┬── Home
                                 ├── Post detail
                                 └── Post list
                     storefront-base.page.ts

                                                        utils (helper dùng chung)
```

Nguyên tắc:
- **`base.page.ts`**: chứa các thao tác/logic **dùng chung cho mọi page** (VD: hàm goto, hàm chờ loading...).
- Mỗi khu vực lớn (Dashboard, Storefront) có **1 base page riêng** (`dashboard-base.page.ts`, `storefront-base.page.ts`) kế thừa từ base chung, chứa logic dùng chung cho khu vực đó.
- Từng trang cụ thể (login, product, category, home, post detail...) là **1 class page riêng**, kế thừa từ base tương ứng, chỉ chứa locator/action đặc thù của đúng trang đó.
- **`utils/`**: thư mục riêng chứa các hàm tiện ích dùng chung, không thuộc về 1 page cụ thể nào (VD: hàm random string, hàm format ngày tháng...).

### 3.2. Cấu trúc thư mục thực tế (ví dụ từ demo)

```
├── README.md
├── pom
│   ├── dashboard
│   │   ├── login
│   │   └── dashboard
│   │       ├── home.page.ts
│   │       └── update.page.ts
│   │   └── post
│   │       ├── post.page.ts
│   │       ├── add-post.page.ts
│   │       ├── category.page.ts
│   │       └── tags.page.ts
│   └── storefront
│       ├── home.page.ts
│       ├── single-post.page.ts
│       └── category.page.ts
├── tests
│   ├── dashboard
│   │   ├── login
│   │   └── dashboard
│   │       ├── home.spec.ts
│   │       └── update.spec.ts
│   │   └── post
│   │       ├── post.spec.ts
│   │       ├── add-post.spec.ts
│   │       ├── category.spec.ts
│   │       └── tags.spec.ts
│   └── storefront
│       ├── home.spec.ts
│       ├── single-post.spec.ts
│       └── category.spec.ts
└── utils
```

Điểm mấu chốt: cấu trúc thư mục **`tests/`** được tổ chức **rập khuôn y hệt** cấu trúc thư mục **`pom/`** — mỗi page object (`xxx.page.ts`) có đúng 1 file test tương ứng (`xxx.spec.ts`) cùng vị trí. Cách tổ chức song song này giúp:
- Dễ tìm test tương ứng với 1 page bất kỳ.
- Dễ mở rộng: thêm 1 trang mới → biết ngay cần thêm file gì ở đâu.

### 3.3. Repo boilerplate tham khảo

Playwright VN cung cấp sẵn 1 boilerplate framework mẫu (setup theo đúng tư duy POM ở trên) để tham khảo/dùng làm khung sườn cho project thật:

🔗 https://github.com/playwrightvn/playwright-typescript-boilerplate

---

## 4. CI/CD — Continuous Integration / Continuous Delivery

### 4.1. CI (Continuous Integration) là gì?

- CI là quy trình **tự động hóa việc tích hợp code** từ nhiều người vào 1 repository chính một cách thường xuyên, ít nhất vài lần mỗi ngày.
- Mỗi lần tích hợp (commit hoặc merge), hệ thống sẽ **tự động build ứng dụng, chạy các tests** để xác nhận rằng code mới **không làm hỏng code cũ**.
- CI nhấn mạnh vào việc **phát hiện lỗi sớm** và giữ cho codebase luôn "healthy" (không có lỗi).

**Lợi ích của CI:**
- Giảm rủi ro
- Tăng tốc độ phát triển
- Cải thiện chất lượng
- Hỗ trợ collaboration (làm việc nhóm)

### 4.2. CD (Continuous Delivery) là gì?

- CD **mở rộng từ CI** bằng cách tự động hóa thêm bước **deploy code** lên các môi trường staging hoặc production sau khi tests pass.
- Mục tiêu: đảm bảo codebase luôn ở trạng thái **"deployable"** (sẵn sàng để deploy bất cứ lúc nào).
- **Khác với Continuous Deployment**: CD (Continuous **Delivery**) thường yêu cầu **phê duyệt thủ công** trước khi deploy lên production, trong khi Continuous **Deployment** là **hoàn toàn tự động** (không cần người duyệt).

**Lợi ích của CD:**
- Release nhanh chóng
- Giảm downtime
- Tăng độ tin cậy
- Phù hợp với automation

```
        code ──build──▶ test ──▶ plan ──release──▶ deploy ──▶ operate ──▶ monitor
        └──────────────── CI ────────────────┘└──────────────── CD ─────────────┘
```

### 4.3. Triển khai CI/CD với GitHub Actions

File cấu hình: **`.github/workflows/playwright.yml`**

```yaml
name: Playwright Tests
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: lts/*
    - name: Install dependencies
      run: npm ci
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    - name: Run Playwright tests
      run: npx playwright test
    - uses: actions/upload-artifact@v4
      if: ${{ !cancelled() }}
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

Giải thích từng bước (step):

| Step | Ý nghĩa |
|---|---|
| `on: push / pull_request` (branches `main`, `master`) | Workflow tự động chạy mỗi khi có push hoặc tạo pull request nhắm vào nhánh `main`/`master` |
| `runs-on: ubuntu-latest` | Chạy trên máy ảo Ubuntu do GitHub cấp miễn phí |
| `actions/checkout@v4` | Clone code của repo về máy ảo CI |
| `actions/setup-node@v4` (`node-version: lts/*`) | Cài Node.js bản LTS mới nhất |
| `npm ci` | Cài dependencies **đúng phiên bản khoá trong `package-lock.json`** (nhanh & ổn định hơn `npm install` cho môi trường CI) |
| `npx playwright install --with-deps` | Cài các trình duyệt (Chromium/Firefox/WebKit) + các thư viện hệ thống cần thiết |
| `npx playwright test` | Chạy toàn bộ test suite |
| `actions/upload-artifact@v4` (`if: ${{ !cancelled() }}`) | Upload `playwright-report/` làm artifact để xem lại kết quả test, **kể cả khi job bị fail** (chỉ bỏ qua nếu job bị cancel), lưu giữ trong 30 ngày |

### 4.4. Triển khai CI/CD với GitLab

Khác với GitHub Actions (đã có sẵn workflow ngay trong repo GitHub), khi muốn dùng **GitLab CI/CD** cần:

**Bước 1 — Tạo project mới trên GitLab**

Vào GitLab → **Projects → New project/repository → Create blank project**, đặt tên project (VD: `k19-practice`), chọn Visibility Level (Private/Internal/Public) → **Create project**.

**Bước 2 — Thêm GitLab làm 1 remote mới cho repo local**

Repo local đang có sẵn remote GitHub (VD: `origin`), thêm thêm 1 remote trỏ tới GitLab song song:

```bash
git remote add gitlab git@gitlab.com:minhphong306/k19-practice.git
```

**Bước 3 — Push code (kèm cả file workflow) lên GitLab**

```bash
git push gitlab lesson-14
```

> Lưu ý: 1 repo local hoàn toàn có thể **có nhiều remote cùng lúc** (vừa push lên GitHub, vừa push lên GitLab) — hữu ích khi cần backup code hoặc khi công ty dùng GitLab nội bộ song song với GitHub.

**Bước 4 — Thiết lập SSH Key để xác thực với GitLab**

Để `git push`/`git clone` qua SSH không phải nhập username/password mỗi lần, cần khai báo SSH public key với tài khoản GitLab:

1. Vào **Avatar góc phải → Edit profile / Preferences** (có thể tuỳ chỉnh giao diện Light/Dark, Navigation theme...).
2. Vào mục **SSH Keys** trong User Settings.
3. Bấm **Add new key**, dán nội dung SSH public key (`~/.ssh/id_rsa.pub` hoặc `id_ed25519.pub`) vào ô **Key**, đặt **Title** để dễ nhận biết (VD: "MacBook key"), chọn **Expiration date** (GitLab bắt buộc key phải có ngày hết hạn).
4. Sau khi thêm key thành công, máy đó có quyền `git push`/`git pull` qua giao thức SSH (`git@gitlab.com:...` hoặc `git@gitlab.gotit.vn:...` với GitLab tự host nội bộ công ty) mà không cần nhập lại mật khẩu.

**Bước 5 — Viết file cấu hình CI/CD cho GitLab**

GitLab CI/CD dùng file cấu hình riêng là **`.gitlab-ci.yml`** ở root project (khác với `.github/workflows/*.yml` của GitHub Actions) để định nghĩa các stage/job cần chạy (install, test, upload report...) — tư duy các bước bên trong tương tự GitHub Actions (checkout code có sẵn tự động, cài Node, `npm ci`, `npx playwright install --with-deps`, `npx playwright test`).

### 4.5. So sánh nhanh GitHub Actions vs GitLab CI

| | GitHub Actions | GitLab CI |
|---|---|---|
| File cấu hình | `.github/workflows/*.yml` | `.gitlab-ci.yml` (ở root repo) |
| Nơi chạy | GitHub-hosted runner (hoặc self-hosted) | GitLab runner (SaaS hoặc self-hosted, VD: `gitlab.gotit.vn` nội bộ công ty) |
| Trigger | `on: push / pull_request`, khai báo branches | `rules` / `only` / `workflow:rules` theo branch, tag, merge request... |
| Kết quả test | Upload artifact (`actions/upload-artifact`) | Khai báo `artifacts:` trong job |

---

## 5. Further learning — Học gì tiếp theo?

**Tự học:**
- "Cày" 1 lượt tài liệu chính thức: **Playwright documentation**
- Theo dõi các kênh:
  - Youtube Playwright Việt Nam
  - Youtube Playwright Official
  - Discord Playwright Official
  - Repo Github Playwright Official
- Join cộng đồng **Playwright Việt Nam (Better Bytes Academy)** — có thể đi theo lộ trình **Trợ giảng → Giảng viên chính thức**

**Khoá học nâng cao:**
- Khoá học advance Playwright Việt Nam

---

## Tổng kết bài học

| Chủ đề | Nội dung chính |
|---|---|
| **Global setup & teardown** | Chạy **1 lần duy nhất** trước/sau toàn bộ test suite (khác `fixture` — chạy lại mỗi test); khai báo bằng 2 `project` riêng (`testMatch`, `teardown`) và `dependencies` trong `playwright.config.ts` |
| **Course wrap up** | Ráp toàn bộ kiến thức đã học thành 1 framework hoàn chỉnh, rồi đưa lên CI/CD chạy tự động |
| **Build test framework (POM)** | Tổ chức code theo `base.page.ts` → base riêng cho từng khu vực (Dashboard/Storefront) → page cụ thể; thư mục `tests/` tổ chức song song với `pom/`; có `utils/` dùng chung |
| **CI (Continuous Integration)** | Tự động build + test mỗi lần tích hợp code, phát hiện lỗi sớm |
| **CD (Continuous Delivery)** | Tự động deploy sau khi test pass, cần phê duyệt thủ công trước production (khác Continuous Deployment — tự động hoàn toàn) |
| **GitHub Actions** | Cấu hình tại `.github/workflows/playwright.yml`: checkout → setup Node → `npm ci` → cài browser → `npx playwright test` → upload report |
| **GitLab CI** | Tạo project trên GitLab, thêm remote `git remote add gitlab ...`, `git push gitlab <branch>`, thiết lập SSH Key để xác thực, cấu hình bằng `.gitlab-ci.yml` |
| **Further learning** | Tự học qua doc/Youtube/Discord/Github chính thức, tham gia cộng đồng Playwright Việt Nam, học khoá nâng cao |

**Nguyên tắc chung cần nhớ:** Global setup/teardown và CI/CD đều phục vụ cùng 1 mục tiêu — **giảm việc lặp lại thủ công** và **đảm bảo test chạy ổn định, tự động, đúng thứ tự** dù là trong 1 lần chạy local (setup/teardown) hay mỗi lần code được đẩy lên repository (CI/CD).
