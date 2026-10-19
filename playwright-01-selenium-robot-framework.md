# Tổng hợp kiến thức: Playwright, Selenium, Robot Framework

---

## 1. Playwright là gì?

**Playwright** là một framework tự động hóa trình duyệt (browser automation) do Microsoft phát triển, dùng để viết test tự động cho các ứng dụng web.

### Playwright làm được gì?

- **Tự động hóa trình duyệt**: Điều khiển Chrome, Firefox, Safari (WebKit) bằng code
- **End-to-end testing (E2E)**: Mô phỏng hành động người dùng thực — click, nhập text, cuộn trang, v.v.
- **Kiểm tra UI**: Kiểm tra xem giao diện có hiển thị đúng không
- **Chụp ảnh màn hình / quay video**: Lưu lại bằng chứng khi test thất bại
- **Chạy song song**: Chạy nhiều test cùng lúc, tiết kiệm thời gian

### Ví dụ đơn giản (TypeScript)

```typescript
import { test, expect } from '@playwright/test';

test('kiểm tra trang chủ', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle('Example Domain');
  await page.click('text=More information');
});
```

### Tại sao dùng Playwright?

| Ưu điểm | Chi tiết |
|---|---|
| Hỗ trợ đa trình duyệt | Chrome, Firefox, Safari |
| Auto-wait | Tự chờ element xuất hiện, không cần `sleep` |
| Selector mạnh | Tìm element theo text, role, CSS, XPath |
| Báo cáo đẹp | HTML report có ảnh/video kèm theo |

---

## 2. Playwright gồm những ngôn ngữ nào?

Playwright hỗ trợ **5 ngôn ngữ lập trình**:

| Ngôn ngữ | File extension | Ghi chú |
|---|---|---|
| **JavaScript** | `.js` | Phổ biến nhất, không cần khai báo kiểu |
| **TypeScript** | `.ts` | Được khuyến nghị dùng, có type checking |
| **Python** | `.py` | Phù hợp với người quen Python |
| **Java** | `.java` | Dùng trong môi trường enterprise |
| **C#** | `.cs` | Dùng với .NET ecosystem |

### Ví dụ cùng 1 hành động ở các ngôn ngữ khác nhau

**TypeScript/JavaScript:**
```typescript
await page.click('button#submit');
```

**Python:**
```python
page.click('button#submit')
```

**Java:**
```java
page.click("button#submit");
```

**C#:**
```csharp
await page.ClickAsync("button#submit");
```

### Nên dùng ngôn ngữ nào?

**TypeScript** là lựa chọn tốt nhất vì:
- Được Playwright team khuyến nghị chính thức
- Có **autocomplete** và **báo lỗi ngay khi gõ** trong VS Code
- Tài liệu và ví dụ trên internet nhiều nhất
- Phù hợp với môi trường web/Zalo Mini App

---

## 3. Cài Playwright như thế nào?

**Yêu cầu trước:** Cần có Node.js (version 18+)

```bash
# Tạo thư mục project mới
mkdir my-tests
cd my-tests

# Cài Playwright
npm init playwright@latest
```

Trong quá trình cài, nó hỏi:
- Dùng TypeScript hay JavaScript? → chọn **TypeScript**
- Thư mục chứa test? → nhấn Enter (mặc định `tests/`)
- Thêm GitHub Actions? → **No**
- Cài trình duyệt? → **Yes**

### Có UI không?

**Có 2 chế độ:**

**1. Không có UI (Headless) — mặc định**
```bash
npx playwright test
```

**2. Có UI — dùng khi debug/viết test**

```bash
# UI Mode (khuyến nghị cho người mới)
npx playwright test --ui

# Headed mode (mở trình duyệt thật)
npx playwright test --headed

# Codegen (tự ghi lại thao tác thành code)
npx playwright codegen https://example.com
```

### Quy trình thực tế

```bash
# Chạy tất cả test
npx playwright test

# Xem báo cáo HTML
npx playwright show-report
```

---

## 4. Làm sao biết Playwright đã được cài?

### Cách 1: Kiểm tra version (nhanh nhất)

```bash
npx playwright --version
```

Kết quả trên máy: **Version 1.59.1** ✅

### Cách 2: Xem trong file `package.json`

```json
{
  "devDependencies": {
    "@playwright/test": "^1.52.0"
  }
}
```

### Cách 3: Xem thư mục `node_modules`

```
my-tests/
├── node_modules/
│   └── @playwright/      ← có thư mục này là đã cài
├── package.json
└── tests/
```

### Kiểm tra trình duyệt đã cài chưa

```bash
npx playwright install --dry-run
```

Thư mục lưu trình duyệt trên Windows:
```
C:\Users\TuyenVo\AppData\Local\ms-playwright\
```

---

## 5. Selenium là gì?

**Selenium** là một framework tự động hóa trình duyệt **lâu đời nhất** (ra đời năm 2004).

### So sánh Selenium vs Playwright

| Tiêu chí | Selenium | Playwright |
|---|---|---|
| **Năm ra đời** | 2004 | 2020 |
| **Tác giả** | ThoughtWorks | Microsoft |
| **Ngôn ngữ** | Java, Python, JS, C#, Ruby | JS, TS, Python, Java, C# |
| **Tốc độ** | Chậm hơn | Nhanh hơn |
| **Auto-wait** | Không có, phải tự viết `wait` | Có sẵn |
| **Cài trình duyệt** | Phải cài driver riêng (ChromeDriver) | Tự cài luôn |
| **Cộng đồng** | Rất lớn, tài liệu nhiều | Đang phát triển nhanh |
| **Độ phổ biến** | Rất cao (nhiều công ty cũ) | Đang thay thế Selenium |

### Ví dụ so sánh code

**Selenium (Python):**
```python
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait

driver = webdriver.Chrome()
driver.get("https://example.com")

# Phải tự chờ element
wait = WebDriverWait(driver, 10)
wait.until(EC.element_to_be_clickable((By.ID, "submit")))
driver.find_element(By.ID, "submit").click()
```

**Playwright (Python):**
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    page.click("#submit")  # tự động chờ, không cần wait
```

---

## 6. Robot Framework là gì?

**Robot Framework** là một framework test tự động **dạng keyword-driven** — viết test bằng **ngôn ngữ tự nhiên** thay vì viết code thuần túy.

### Ví dụ

```robot
*** Test Cases ***
Kiểm tra đăng nhập thành công
    Mở trình duyệt    https://example.com
    Nhập username     admin
    Nhập password     123456
    Click nút         Đăng nhập
    Kiểm tra hiển thị    Chào mừng admin
```

### So sánh 3 framework

| Tiêu chí | Robot Framework | Playwright | Selenium |
|---|---|---|---|
| **Năm ra đời** | 2005 | 2020 | 2004 |
| **Ngôn ngữ viết test** | Gần tiếng Anh (keyword) | Code thật | Code thật |
| **Độ khó** | Dễ nhất | Trung bình | Khó nhất |
| **Phù hợp với** | QA không biết code | Developer, QA biết code | Developer |
| **Tốc độ** | Chậm hơn | Nhanh nhất | Chậm |
| **Báo cáo** | Rất đẹp, có sẵn | Cần cấu hình thêm | Cần tool thêm |

### Robot Framework hoạt động thế nào?

```
Robot Framework (core)
        │
        ├── SeleniumLibrary  → dùng Selenium để điều khiển trình duyệt
        ├── Browser Library  → dùng Playwright để điều khiển trình duyệt
        ├── RequestsLibrary  → test API
        └── DatabaseLibrary  → test database
```

### Báo cáo của Robot Framework

- `report.html` — tổng quan pass/fail
- `log.html` — từng bước chi tiết có ảnh chụp màn hình
- `output.xml` — dữ liệu thô

---

## 7. Selenium có UI không?

**Selenium core** — **không có UI** riêng, chỉ là thư viện code.

### Các công cụ đi kèm có UI

- **Selenium IDE**: Extension trên Chrome/Firefox, ghi lại thao tác click → tự sinh code

### So sánh UI giữa 3 framework

| Framework | Có UI riêng? | Ghi thao tác? | Báo cáo HTML? |
|---|---|---|---|
| **Selenium** | Selenium IDE (extension) | Có (IDE) | Cần cài thêm |
| **Playwright** | Có (`--ui` mode) | Có (`codegen`) | Có sẵn |
| **Robot Framework** | Không | Không | Có sẵn, rất đẹp |

---

## 8. Playwright chạy được ở những nơi nào?

| Nơi chạy | Ghi chú |
|---|---|
| **VS Code** | Có extension riêng, phổ biến nhất |
| **Terminal / Command Line** | CMD, PowerShell, Git Bash |
| **WebStorm / IntelliJ** | JetBrains IDEs |
| **GitHub Actions** | Phổ biến nhất, miễn phí |
| **GitLab CI** | Tích hợp sẵn với GitLab |
| **Jenkins** | Dùng nhiều trong doanh nghiệp |
| **Azure DevOps** | Microsoft, tích hợp tốt với Playwright |
| **Docker** | Chạy trong container trên server Linux |

---

## 9. Extension "Playwright Test for VSCode" là gì?

### Những thứ extension này cho phép làm:

1. **Chạy test bằng nút bấm** — Click nút tam giác xanh cạnh mỗi test
2. **Xem kết quả pass/fail ngay trong editor** — Xanh/đỏ tại dòng code
3. **Debug từng bước** — Đặt breakpoint, xem biến, xem trình duyệt
4. **Mở Playwright UI Mode** — Click 1 nút là mở giao diện có timeline, screenshot
5. **Ghi thao tác (Record/Codegen)** — Click Record → bạn click trên web → tự sinh code
6. **Pick locator** — Hover chuột lên element → tự sinh selector

### Trước và sau khi cài extension

| | Không có extension | Có extension |
|---|---|---|
| Chạy test | Gõ lệnh terminal | Click nút tam giác |
| Xem lỗi | Đọc terminal | Thấy ngay trong code |
| Debug | Khó | Dễ, có breakpoint |
| Ghi thao tác | Gõ lệnh `codegen` | Click nút Record |

### Cách cài

1. Mở VS Code
2. Nhấn `Ctrl + Shift + X`
3. Tìm **"Playwright Test for VSCode"**
4. Chọn cái của **Microsoft** → nhấn **Install**

### Cách kiểm tra đã cài chưa

- Nhấn `Ctrl + Shift + X` → tìm Playwright → nếu thấy nút **"Uninstall"** → đã cài
- Hoặc nhìn thanh bên trái có icon **Testing (bình thử nghiệm)** → đã cài

---

## 10. Playwright được viết code ở đâu?

### Viết code trong VS Code

```
my-project/
├── tests/
│   ├── login.spec.ts       ← viết code ở đây
│   ├── homepage.spec.ts
│   └── checkout.spec.ts
├── playwright.config.ts
└── package.json
```

### Cấu trúc 1 file test cơ bản

```typescript
import { test, expect } from '@playwright/test';

test('tên test của bạn', async ({ page }) => {
    await page.goto('https://example.com');
    await page.click('#button');
    await expect(page).toHaveTitle('Example');
});
```

### Không biết viết code thì làm thế nào?

```bash
npx playwright codegen https://example.com
```

→ Mở 2 cửa sổ:
- Cửa sổ 1: Trình duyệt để bạn thao tác
- Cửa sổ 2: Code tự động được sinh ra

### Các editor khác có thể dùng

| Editor | Ghi chú |
|---|---|
| **VS Code** | Phổ biến nhất, có extension riêng |
| **WebStorm** | Mạnh cho JS/TS |
| **IntelliJ IDEA** | Dùng nhiều trong doanh nghiệp |
| **Notepad++** | Đơn giản, không có autocomplete |
| **Sublime Text** | Nhẹ, nhanh |
| **Vim / Neovim** | Dành cho người thích terminal |

---

## 11. Selenium — Viết code ở đâu và chạy ở đâu?

### Viết code ở đâu?

| Editor | Ghi chú |
|---|---|
| **VS Code** | Phổ biến, có extension hỗ trợ |
| **PyCharm** | Tốt nhất nếu dùng Python |
| **IntelliJ IDEA** | Tốt nhất nếu dùng Java |
| **Eclipse** | Phổ biến trong môi trường Java/doanh nghiệp |
| **WebStorm** | Tốt nếu dùng JavaScript |

### Chạy ở đâu?

| Nơi chạy | Ghi chú |
|---|---|
| **Terminal / Command Line** | Cách cơ bản nhất |
| **VS Code / PyCharm / IntelliJ** | Chạy ngay trong IDE |
| **GitHub Actions** | Tự động chạy trên cloud |
| **Jenkins** | Phổ biến trong doanh nghiệp lớn |
| **Selenium Grid** | Chạy song song nhiều máy, nhiều trình duyệt |
| **Docker** | Chạy trên server Linux |

### Selenium Grid

```
Selenium Grid
      │
      ├── Máy 1: Chrome  (Windows)
      ├── Máy 2: Firefox (Linux)
      ├── Máy 3: Safari  (Mac)
      └── Máy 4: Edge    (Windows)
```

---

## 12. Selenium viết ở Eclipse thì chạy như thế nào?

### Chạy ngay trong Eclipse

**Cách 1: Click chuột phải**
```
Click phải vào file test
→ Chọn "Run As"
→ Chọn "JUnit Test" hoặc "TestNG Test"
→ Test chạy luôn
```

**Cách 2: Nút Run trên toolbar**
```
Nhấn nút ▶ (Run) trên thanh toolbar
```

**Cách 3: Phím tắt**
```
Ctrl + F11  →  chạy test
```

### Xem kết quả ở đâu?

**Panel JUnit/TestNG:**
```
✅ testDangNhap        — PASS (1.2s)
❌ testThanhToan       — FAIL
✅ testDangXuat        — PASS (0.8s)
```

**Panel Console:**
```
Starting ChromeDriver...
Browser opened
Navigating to https://example.com
Test PASSED
```

### Cấu trúc project Selenium trong Eclipse

```
SeleniumProject/
├── src/
│   ├── test/
│   │   ├── DangNhapTest.java    ← viết test ở đây
│   │   └── ThanhToanTest.java
│   └── main/
│       └── BasePage.java
├── pom.xml                      ← khai báo thư viện Selenium
└── testng.xml                   ← cấu hình chạy test
```

### Quy trình thực tế

```
Viết code Java trong Eclipse
        ↓
Click phải → Run As → JUnit/TestNG Test
        ↓
Chrome/Firefox tự mở ra và chạy test
        ↓
Xem kết quả trong panel JUnit/Console
        ↓
Xem báo cáo HTML (nếu dùng TestNG/Extent Reports)
```

### Những thứ cần cài để chạy Selenium trong Eclipse

| Thứ cần cài | Mục đích |
|---|---|
| **Java JDK** | Ngôn ngữ lập trình |
| **Eclipse IDE** | Editor để viết code |
| **Selenium JAR** | Thư viện Selenium |
| **ChromeDriver** | Kết nối Selenium với Chrome |
| **TestNG hoặc JUnit** | Framework chạy test |
| **Maven** | Quản lý thư viện (pom.xml) |

→ Setup **phức tạp hơn Playwright** nhiều.

---

*Tài liệu được tổng hợp ngày 09/05/2026*
