# Lesson 1 - Getting Started with Playwright

> Khóa học: **Học automation test từ chưa biết gì** | Better Bytes Academy

---

## Mục lục
1. [Playwright là gì?](#1-playwright-là-gì)
2. [Tại sao chọn Playwright TypeScript?](#2-tại-sao-chọn-playwright-typescript)
3. [Các công cụ cần cài đặt](#3-các-công-cụ-cần-cài-đặt)
4. [Cài đặt & cấu hình Git](#4-cài-đặt--cấu-hình-git)
5. [Cài đặt Visual Studio Code](#5-cài-đặt-visual-studio-code)
6. [Kết nối Git với GitHub qua SSH](#6-kết-nối-git-với-github-qua-ssh)
7. [Cài đặt Playwright](#7-cài-đặt-playwright)
8. [Hiểu code Playwright cơ bản](#8-hiểu-code-playwright-cơ-bản)
9. [Đưa code lên GitHub](#9-đưa-code-lên-github)

---

## 1. Playwright là gì?

Playwright là một **framework automation testing** cho trình duyệt web.

| Thông tin | Chi tiết |
|-----------|----------|
| Tiền thân | Puppeteer |
| Phát triển bởi | Microsoft |
| Ra đời | 31/01/2020 |
| Version mới nhất | v1.56 |
| Website | https://playwright.dev/ |

### Ưu điểm nổi bật

#### Cross Browser - Chạy được nhiều trình duyệt
- Hỗ trợ: **Chrome, Edge, Firefox, Safari**
- Hỗ trợ lên tới **143 phiên bản** trình duyệt khác nhau
- Hỗ trợ **143 thiết bị** (Blackberry, Galaxy, iPhone...)

#### Cross Platform - Chạy được nhiều hệ điều hành
- Code **một lần**, chạy được trên: **Windows, Linux, MacOS**

#### Tính năng "xịn sò" - Auto Waiting
- Playwright **tự động chờ** element sẵn sàng trước khi thao tác
- **Auto-retry assertion** giúp giảm **flaky tests** (test lúc pass lúc fail)
- Ví dụ: Playwright sẽ tự chờ button hiện ra, chờ element ổn định rồi mới click

#### Report đầy đủ thông tin
- Hiển thị pass/fail **rõ ràng theo từng trình duyệt** (chromium, firefox, webkit)
- **Click vào từng test** để xem chi tiết: lỗi ở đâu, dòng code nào
- **Trace viewer**: xem lại từng bước thực thi, thời gian chạy, lỗi ở dòng nào

#### Code Gen - Tự sinh code
- Thao tác trực tiếp trên trình duyệt, Playwright **tự động sinh ra code** tương ứng

---

## 2. Tại sao chọn Playwright TypeScript?

### Lý do học Playwright
- **Dễ cài đặt**: Chỉ cần vài lệnh là xong
- **Cú pháp đơn giản**: Dễ đọc, dễ hiểu, giống như "lời thoại" tự nhiên
- **Trending**: Đang được nhiều công ty sử dụng, nhiều cơ hội việc làm

### Công ty đang dùng tool khác có nên học không?
**Có!** Vì:
- Khóa học dạy về **tư duy** và **bản chất** của automation testing
- Hiểu bản chất thì các tools (Cypress, Selenium, Appium...) đều tương tự nhau
- Mentor có kiến thức đa dạng, có thể hỗ trợ tool của công ty bạn

---

## 3. Các công cụ cần cài đặt

Trước khi bắt đầu, cần cài sẵn:

| Công cụ | Là gì? | Dùng để làm gì? |
|---------|--------|-----------------|
| **NVM** | Node Version Manager | Quản lý nhiều phiên bản Node.js |
| **Node.js** | Runtime environment | Chạy code JavaScript/TypeScript |
| **Git** | Version control | Quản lý source code |
| **GitHub** | Cloud platform | Lưu trữ & chia sẻ code |

### NVM là gì?
- **NVM** = Node Version Manager = Công cụ quản lý các **phiên bản Node.js**
- **Node.js** = Công cụ để **chạy code** JavaScript/TypeScript trên máy tính
- Nên cài Node.js **thông qua NVM** thay vì cài trực tiếp
  - Lý do: Dễ dàng **chuyển đổi giữa các phiên bản** Node.js (vì mỗi dự án có thể cần version khác nhau)

---

## 4. Cài đặt & cấu hình Git

### Git & GitHub là gì?
- **Git**: Công cụ **quản lý source code** trên máy local
- **GitHub**: Nền tảng online để **lưu trữ & chia sẻ code**, làm việc nhóm

### Cấu hình Git lần đầu
Sau khi cài Git, cần cấu hình thông tin cá nhân:

```bash
# Đặt tên người dùng
git config --global user.name "Tên của bạn"

# Đặt email
git config --global user.email "email@example.com"

# Đặt branch mặc định là main
git config --global init.defaultBranch main
```

---

## 5. Cài đặt Visual Studio Code

### VS Code là gì?
- **VS Code** = Visual Studio Code = **IDE** (Integrated Development Environment)
- Hiểu đơn giản: đây là **công cụ để viết code**
- Tải về tại: https://code.visualstudio.com/

### Cài thêm Playwright Extension
- Tìm và cài extension: **"Playwright Test for VS Code"** (của Microsoft)
- Extension giúp chạy test, debug trực tiếp trong VS Code

### Đổi terminal sang Git Bash (quan trọng với Windows)
PowerShell hay bị lỗi lặt vặt khi dùng với Playwright, nên đổi sang **Git Bash**:

1. Nhấn **Ctrl + Shift + P** để mở hộp thoại lệnh
2. Gõ tìm: `Terminal default`
3. Chọn: **Terminal: Select Default Profile**
4. Chọn **Git Bash**
5. Kiểm tra lại bằng cách mở terminal mới: **Menu > Terminal > New Terminal**
   - Nếu thấy `bash` ở góc phải terminal là thành công

---

## 6. Kết nối Git với GitHub qua SSH

### SSH Key là gì?
SSH Key là **cặp khóa bảo mật** gồm 2 file:

| File | Vai trò | Lưu ở đâu |
|------|---------|-----------|
| `id_rsa` | **Private key** - Giữ bí mật tuyệt đối | Máy local (`~/.ssh/`) |
| `id_rsa.pub` | **Public key** - Có thể chia sẻ | Upload lên GitHub |

> **Ví dụ dễ hiểu**: id_rsa là **chìa khóa**, id_rsa.pub là **ổ khóa**. Bạn giữ chìa, đặt ổ khóa ở GitHub.

Thư mục `~/.ssh/` nghĩa là:
- Windows: `C:\Users\{tên_user}\.ssh\`
- Linux/MacOS: `/Users/{tên_user}/.ssh/`

### Bước 1: Tạo SSH Key

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

- Nhấn **Enter** liên tục để giữ vị trí lưu mặc định
- Nhấn **Enter** để bỏ qua passphrase (mật khẩu của key - để trống)
- Thấy hình randomart hiển thị là **thành công**

> Nếu thấy `already exists` → Key đã tồn tại rồi, **không cần tạo lại**, gõ `n` để dừng.

### Bước 2: Lấy nội dung Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

Sao chép toàn bộ nội dung xuất hiện (bắt đầu bằng `ssh-rsa ...`)

### Bước 3: Thêm Public Key vào GitHub

1. Truy cập: `https://github.com/settings/ssh/new`
2. **Title**: Đặt tên dễ nhớ (ví dụ: "ssh key - personal computer")
3. **Key**: Dán nội dung public key vừa copy
4. Nhấn **Add SSH Key**

---

## 7. Cài đặt Playwright

### Chuẩn bị thư mục
Tạo **một thư mục cha** chứa toàn bộ code của khoá học (ví dụ: `k19/`), bên trong tạo thư mục cho project:
```
k19/
├── pw-course/      ← Thư mục Playwright project
├── practice-1/
└── git-course/
```

### Cài đặt

```bash
# 1. Tạo thư mục và vào bên trong
mkdir demo-1
cd demo-1

# 2. Mở VS Code
code .

# 3. Chạy lệnh khởi tạo Playwright (trong terminal Git Bash)
npm init playwright@latest
```

- Nhấn **Enter** liên tục theo các câu hỏi hiện ra
- Chờ đến khi thấy **"Happy hacking!"** là thành công

### Cấu trúc project sau khi cài

```
pw-course/
├── node_modules/           ← Thư viện (tự sinh, không sửa)
├── tests/
│   └── example.spec.ts     ← File code test mẫu
├── .github/
├── .gitignore
├── package.json
├── package-lock.json
└── playwright.config.ts    ← File cấu hình Playwright
```

### Các lệnh hay dùng

```bash
npx playwright test                    # Chạy toàn bộ test
npx playwright test --ui               # Chạy ở chế độ UI (giao diện đẹp)
npx playwright test --project=chromium # Chỉ chạy trên Chrome
npx playwright test example            # Chạy file test cụ thể
npx playwright test --debug            # Chạy chế độ debug
npx playwright codegen                 # Tự động sinh code từ thao tác
```

---

## 8. Hiểu code Playwright cơ bản

Code Playwright rất dễ đọc, giống như "lời thoại" tự nhiên:

| Từ khóa | Nghĩa |
|---------|-------|
| `goto` | Đi đến (mở URL) |
| `expect` | Mong muốn (kiểm tra) |
| `toHave...` | Có... |
| `click` | Click |
| `getByRole` | Tìm element theo role |

### Ví dụ 1: Kiểm tra tiêu đề trang

```typescript
import { test, expect } from '@playwright/test';

test('has title', async ({ page }) => {
  // Đi đến trang playwright.dev
  await page.goto('https://playwright.dev/');

  // Kiểm tra tiêu đề trang có chứa chữ "Playwright" không
  await expect(page).toHaveTitle(/Playwright/);
});
```

### Ví dụ 2: Kiểm tra click link

```typescript
test('get started link', async ({ page }) => {
  // Đi đến trang playwright.dev
  await page.goto('https://playwright.dev/');

  // Click vào link có chữ "Get started"
  await page.getByRole('link', { name: 'Get started' }).click();

  // Kiểm tra heading "Installation" có hiển thị trên màn hình không
  await expect(page.getByRole('heading', { name: 'Installation' })).toBeVisible();
});
```

> **Đừng lo** nếu chưa hiểu hết code. Tập trung xây dựng **nền tảng lập trình** trong 4 buổi đầu, bạn sẽ hiểu dần dần.

---

## 9. Đưa code lên GitHub

### Tổng quan quy trình

```
[Code trên máy] → git add → git commit → git push → [GitHub]
```

### Bước 1: Tạo Repository trên GitHub (làm 1 lần duy nhất)

1. Truy cập: `https://github.com/new`
2. Điền **tên repository**
3. Chọn **Public**
4. Nhấn **Create repository**

### Bước 2: Khởi tạo repo local & liên kết (làm 1 lần duy nhất)

```bash
# Khởi tạo repo local
git init

# Liên kết với GitHub repo vừa tạo
git remote add origin <ssh_link_từ_github>
```

### Bước 3: Đẩy code lên GitHub (làm mỗi khi có thay đổi)

```bash
# Thêm tất cả file vào staging
git add .

# Tạo commit với message mô tả
git commit -m "init project"

# Đẩy code lên GitHub
git push origin main
```

### Sơ đồ tóm tắt

| Lệnh | Mục đích | Làm khi nào |
|------|----------|-------------|
| `git init` | Khởi tạo repo local | 1 lần duy nhất |
| `git remote add origin <url>` | Liên kết với GitHub | 1 lần duy nhất |
| `git add .` | Thêm file vào staging | Mỗi khi có thay đổi |
| `git commit -m "message"` | Lưu snapshot | Mỗi khi có thay đổi |
| `git push origin main` | Đẩy lên GitHub | Mỗi khi có thay đổi |

### Mời Mentor chấm bài (Invite Collaborator)

1. Vào repo settings: `https://github.com/<username>/<repo_name>/settings/access`
2. Click **Add collaborator**
3. Tìm kiếm theo username/email của mentor
4. Gửi lời mời

---

## Tóm tắt nhanh Lesson 1

```
Playwright
  ├── Framework automation test của Microsoft
  ├── Cross browser (Chrome/Firefox/Safari/Edge)
  ├── Cross platform (Windows/Linux/MacOS)
  ├── Auto waiting → ít flaky test
  └── Report + Trace viewer chi tiết

Công cụ cần có
  ├── NVM + Node.js  → chạy code
  ├── Git            → quản lý code local
  ├── GitHub         → lưu code online
  └── VS Code        → viết code (+ Playwright extension)

Quy trình làm việc
  ├── Viết test → npx playwright test
  ├── Xem report → npx playwright show-report
  └── Push code → git add . → git commit → git push
```
