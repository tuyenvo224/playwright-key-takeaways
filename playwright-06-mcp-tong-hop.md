# Playwright MCP cho QA/QC – Tổng hợp


## Mục lục
1. [Playwright MCP là gì](#1-playwright-mcp-là-gì)
2. [Cài đặt trên Windows](#2-cài-đặt-trên-windows)
3. [25 tool của Playwright MCP](#3-25-tool-của-playwright-mcp)
4. [Thực hành: chụp màn hình và xuất code test](#4-thực-hành-chụp-màn-hình-và-xuất-code-test)
5. [Script tiện ích và code automation test](#5-script-tiện-ích-và-code-automation-test)
6. [Playwright MCP và Chrome DevTools MCP](#6-playwright-mcp-và-chrome-devtools-mcp)
7. [Vì sao MCP chậm và cách dùng SKILL + script](#7-vì-sao-mcp-chậm-và-cách-dùng-skill--script)
8. [Tổ chức automation cho dự án lớn](#8-tổ-chức-automation-cho-dự-án-lớn)
9. [Ví dụ thực tế trên hoctest.com](#9-ví-dụ-thực-tế-trên-hoctestcom)
10. [Có nên dùng AI Browser Agent chạy E2E test?](#10-có-nên-dùng-ai-browser-agent-chạy-e2e-test)
11. [Kinh nghiệm rút ra](#11-kinh-nghiệm-rút-ra)

---

## 1. Playwright MCP là gì

**Playwright MCP** là một MCP server của Microsoft. Nó cho AI (Claude Code, Cursor…) **điều khiển trình duyệt trực tiếp** bằng các tool: mở trang, bấm, gõ, chụp màn hình…

- Không cài MCP thì AI **không có** các tool này. AI chỉ có thể viết script Playwright rồi chạy, hoặc dùng WebFetch để đọc HTML (WebFetch không chụp được màn hình).
- MCP "nhìn" trang bằng **snapshot cây accessibility** (role, name), không dựa vào ảnh.

### Playwright MCP có phải là AI Browser Agent không?

**Không.** Playwright MCP chỉ là **công cụ** để AI dùng trình duyệt.

| Thành phần | Vai trò | Ví dụ |
| --- | --- | --- |
| **AI (bộ não)** | Đọc yêu cầu, suy nghĩ, quyết định bước tiếp theo | Claude Code |
| **Công cụ trình duyệt (đôi tay)** | Thực hiện thao tác: mở trang, bấm, gõ, chụp | Playwright MCP |
| **AI Browser Agent** | Bộ não + đôi tay: tự lên kế hoạch và tự thao tác trình duyệt | Claude Code + Playwright MCP |

- Playwright MCP đứng một mình chỉ nhận lệnh kiểu "click vào phần tử e30" rồi làm theo. Nó không tự hiểu test case, không tự quyết định bấm vào đâu.
- Khi Claude Code dùng Playwright MCP, cả hệ thống mới là AI Browser Agent. Ví dụ, khi khám phá hoctest.com, Claude tự quyết định thử từ khóa "Playwright", rồi thử từ khóa không tồn tại, rồi nhận ra bug tìm kiếm. MCP chỉ thực hiện thao tác.
- Các công cụ khác cùng mô hình: Browser Use…

### Playwright MCP có sinh code không?

- **Bản thân MCP không tạo file code.** Nó chỉ thao tác trên trình duyệt.
- Sau mỗi bước, MCP trả về đoạn **"Ran Playwright code"** tương ứng, ví dụ:
  ```js
  await page.getByRole('button', { name: 'Tìm kiếm' }).click();
  ```
- **Có yêu cầu** ("xuất code cho case vừa chạy") thì AI mới gom các đoạn này lại thành file test. **Không yêu cầu** thì chỉ có kết quả Pass/Fail.
- Chạy xong mới đổi ý vẫn được: các đoạn code vẫn còn trong cuộc trò chuyện, chỉ cần yêu cầu xuất code.

| | Agent chạy trực tiếp qua MCP | Chạy file code đã sinh |
| --- | --- | --- |
| Mỗi lần chạy | Agent suy nghĩ lại từng bước | Chạy đúng các bước đã ghi |
| Kết quả lặp lại | Có thể khác nhau | Ổn định |
| Chi phí | Tốn token mỗi lần | Gần như không |
| Hợp với | Khám phá, test thử, viết code lần đầu | Regression, CI |

---

## 2. Cài đặt trên Windows

### Yêu cầu
- Node.js (máy thực hành: v22.16.0).

### Lệnh cài

README chính thức: `claude mcp add playwright npx @playwright/mcp@latest`

Trên **Windows** nên bọc thêm `cmd /c`, vì `npx` thực chất là file `npx.cmd`:
```
claude mcp add playwright -s user -- cmd /c npx @playwright/mcp@latest
```
- `--` dùng để tách phần lệnh chạy server ra khỏi các tùy chọn của `claude mcp add`.

### Phạm vi cài (`-s`)

| Phạm vi | Ai dùng được | Lưu ở đâu |
| --- | --- | --- |
| `local` (mặc định) | Chỉ mình, chỉ project hiện tại | `~/.claude.json` |
| `user` | Chỉ mình, mọi project | `~/.claude.json` |
| `project` | Mọi người mở project | File `.mcp.json` trong project |

Với `-s project`, file `.mcp.json` được tạo ở thư mục gốc project:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "cmd",
      "args": ["/c", "npx", "@playwright/mcp@latest"]
    }
  }
}
```
- Đưa project lên Git thì người khác có sẵn MCP, không cần tự cài.
- Lần đầu mở project, Claude Code sẽ hỏi họ có cho phép chạy server không.
- Máy người nhận vẫn cần cài Node.js.
- **Không** ghi token hay mật khẩu vào file này.

**Gợi ý:** học một mình thì dùng `-s user`. Cả team dùng chung project thì dùng `-s project`.

### Các lỗi đã gặp

| Lỗi | Nguyên nhân | Cách sửa |
| --- | --- | --- |
| `playwright: cmd C:/ npx ... ✗ Failed to connect` | **Git Bash** tự đổi `/c` thành `C:/` | Dùng `cmd //c` trong Git Bash, hoặc chuyển sang PowerShell |
| `cmd /c npx ... ✗ Failed to connect` (lệnh đã đúng) | Lần đầu khởi động chậm: `npx` phải kiểm tra bản `@latest` và nạp Playwright | Chạy lại `claude mcp list`, lần sau sẽ ✓ Connected |
| Đã cài mà AI chưa có tool | Phiên chat được mở trước khi cài | Reload Window |

Cài sai thì xóa rồi cài lại: `claude mcp remove playwright -s user`

### Chuyển terminal sang PowerShell trong VS Code
- Ở khung Terminal, bấm **mũi tên ˅** cạnh dấu **+** rồi chọn **PowerShell**.
- Hoặc đặt mặc định: `Ctrl + Shift + P` → **Terminal: Select Default Profile** → **PowerShell**.

### Kiểm tra sau khi cài
1. `claude mcp list` hiện `playwright: ... ✓ Connected`.
2. Reload Window: `Ctrl + Shift + P` → **Developer: Reload Window**.
3. Trong ô chat gõ `/mcp` để xem trạng thái server (hoặc gõ `/` rồi chọn **MCP servers**).
4. Xem cấu hình: `claude mcp get playwright`, hoặc mở `C:\Users\<user>\.claude.json` và tìm mục `"mcpServers"`.

---

## 3. 25 tool của Playwright MCP

Theo bản 0.0.83 dùng trong buổi thực hành.

**Điều hướng và quản lý trang**
| Tool | Chức năng |
| --- | --- |
| `browser_navigate` | Mở URL |
| `browser_navigate_back` | Quay lại trang trước |
| `browser_tabs` | Xem, mở, đóng, chuyển tab |
| `browser_resize` | Đổi kích thước cửa sổ |
| `browser_close` | Đóng trang |

**Thao tác với phần tử**
| Tool | Chức năng |
| --- | --- |
| `browser_click` | Bấm |
| `browser_type` | Gõ chữ |
| `browser_fill_form` | Điền nhiều ô cùng lúc |
| `browser_select_option` | Chọn trong dropdown |
| `browser_hover` | Rê chuột |
| `browser_drag` | Kéo thả giữa 2 phần tử |
| `browser_drop` | Thả file hoặc dữ liệu lên phần tử |
| `browser_press_key` | Nhấn phím |
| `browser_file_upload` | Upload file |
| `browser_handle_dialog` | Xử lý alert, confirm, prompt |

**Đọc và quan sát trang**
| Tool | Chức năng |
| --- | --- |
| `browser_snapshot` | Lấy cấu trúc trang (cây accessibility) |
| `browser_find` | Tìm phần tử |
| `browser_take_screenshot` | Chụp màn hình vùng hiển thị, toàn trang, hoặc một phần tử |
| `browser_wait_for` | Chờ chữ xuất hiện hoặc biến mất, hoặc chờ một khoảng thời gian |

**Debug**
| Tool | Chức năng |
| --- | --- |
| `browser_console_messages` | Log và lỗi console |
| `browser_network_requests` | Danh sách request |
| `browser_network_request` | Chi tiết một request |
| `browser_evaluate` | Chạy JavaScript trên trang |
| `browser_emulate_media` | Giả lập dark/light mode… |
| `browser_run_code_unsafe` | Chạy code Playwright tùy ý (cần dùng cẩn thận) |

---

## 4. Thực hành: chụp màn hình và xuất code test

### Bước 1: Chụp màn hình bằng MCP
Yêu cầu: *"Dùng Playwright MCP mở https://hoctest.com và chụp màn hình trang chủ"* (chụp cả vùng hiển thị và toàn trang).

AI dùng 2 tool là `browser_navigate` và `browser_take_screenshot`. MCP trả về code:
```js
await page.goto('https://hoctest.com');
await page.screenshot({ path: '.../hoctest-homepage-viewport.png', scale: 'css', type: 'png' });
await page.screenshot({ fullPage: true, path: '.../hoctest-homepage-fullpage.png', scale: 'css', type: 'png' });
```
MCP tự tạo thư mục `.playwright-mcp/` để lưu snapshot và log console. Có thể xóa khi không cần.

### Bước 2: Xuất code test
Yêu cầu: *"xuất code cho case vừa chạy"*, kết quả là file [hoctest-homepage-screenshot.spec.ts](../17/hoctest-homepage-screenshot.spec.ts):
```ts
import { test, expect } from '@playwright/test';

test('Mở trang chủ hoctest.com và chụp màn hình', async ({ page }) => {
  await page.goto('https://hoctest.com');
  await expect(page).toHaveTitle('Trang chủ - HocTest.Com');

  await page.screenshot({ path: 'products/2026-10/17/hoctest-homepage-viewport.png' });
  await page.screenshot({ path: 'products/2026-10/17/hoctest-homepage-fullpage.png', fullPage: true });
});
```
Dòng `toHaveTitle` được thêm vào để file trở thành test thật. Title lấy đúng từ kết quả MCP.

### Bước 3: Cài thư viện và chạy

| Lệnh | Ý nghĩa |
| --- | --- |
| `npm init -y` | Tạo `package.json`, chạy 1 lần cho mỗi project. `-y` = đồng ý hết giá trị mặc định |
| `npm i -D @playwright/test` | Cài Playwright Test. `i` = install, `-D` = ghi vào `devDependencies` (thư viện chỉ dùng khi phát triển/test) |
| `npx playwright install chromium` | Tải trình duyệt Chromium |
| `npx playwright test <file>` | Chạy test. Bỏ `<file>` thì chạy tất cả file `.spec.ts`/`.test.ts` |

Sau khi cài sẽ có thêm `node_modules/` (code thư viện) và `package-lock.json` (phiên bản chính xác). Người khác chỉ cần chạy `npm install` là có đủ thư viện.

Các tùy chọn hay dùng khi chạy test:
- `--headed`: hiện trình duyệt (mặc định chạy ẩn).
- `--ui`: mở UI Mode.
- `--debug`: chạy từng bước để gỡ lỗi.
- `npx playwright show-report`: xem báo cáo HTML.

### Lỗi gạch đỏ ở `import ... from '@playwright/test'`
Đã cài rồi mà vẫn gạch đỏ là do TypeScript server của VS Code còn giữ lỗi từ lúc **chưa cài**. Sửa bằng `Ctrl + Shift + P` → **TypeScript: Restart TS Server**, hoặc đóng và mở lại file.

---

## 5. Script tiện ích và code automation test

Cùng là code Playwright. **Có `expect` (kiểm tra kết quả) thì mới là test.**

| | Script tiện ích | Code automation test |
| --- | --- | --- |
| Mục đích | Làm một việc cụ thể | Kiểm tra app đúng hay sai, chạy lại nhiều lần |
| Assert | Không | Có (`expect(...)`) |
| Cấu trúc | Một file chạy bằng `node` | `@playwright/test`: test case, config, report |

```js
// Script tiện ích: chỉ chụp ảnh
await page.goto('https://hoctest.com');
await page.screenshot({ path: 'home.png' });
```
```js
// Test: có kiểm tra
test('Trang chủ hiển thị logo', async ({ page }) => {
  await page.goto('https://hoctest.com');
  await expect(page.getByRole('img', { name: /logo/i })).toBeVisible();
});
```

---

## 6. Playwright MCP và Chrome DevTools MCP

| Tiêu chí | Playwright MCP (Microsoft) | Chrome DevTools MCP (Google) |
| --- | --- | --- |
| Mục đích chính | Tự động thao tác để test | Debug và phân tích trang |
| Trình duyệt | Chromium, Firefox, WebKit | Chỉ Chrome |
| Sinh code test | Có (code Playwright sau mỗi bước) | Không |
| Network / Console | Cơ bản | Mạnh hơn |
| Hiệu năng | Không chuyên | Mạnh: performance trace, LCP, giả lập mạng hoặc CPU chậm |
| Hợp với | Test chức năng, E2E, tạo automation | Điều tra bug, kiểm tra API, tốc độ tải trang |

**Kết luận cho QA/QC:** dùng **Playwright MCP làm công cụ chính**. Chỉ bật Chrome DevTools MCP khi cần điều tra lỗi hoặc đo hiệu năng, vì mỗi MCP nạp thêm vài chục tool vào ngữ cảnh và tốn token.

---

## 7. Vì sao MCP chậm và cách dùng SKILL + script

### Vì sao Playwright MCP chậm
1. **Nạp sẵn 25 tool** vào ngữ cảnh ngay khi mở phiên chat.
2. **Mỗi bước trả về snapshot** cả trang, trang lớn có thể đến hàng nghìn dòng.
3. **Làm từng bước một**: case 10 bước nghĩa là 10 lượt AI suy nghĩ, gọi tool, rồi đọc kết quả.

### SKILL + script là gì
- **Script**: file code Playwright, AI chạy qua terminal.
- **SKILL**: thư mục trong `.claude/skills/`, có `SKILL.md` hướng dẫn AI khi nào dùng và chạy thế nào. Lúc mở phiên chat chỉ nạp 1–2 dòng mô tả, khi cần mới đọc nội dung đầy đủ.

| | Playwright MCP | SKILL + script |
| --- | --- | --- |
| Lúc mở phiên chat | Nạp 25 tool | Nạp 1–2 dòng mô tả |
| Mỗi bước | Snapshot cả trang | Script chỉ in những gì cần |
| Case nhiều bước | Nhiều lượt gọi tool | Một lần chạy script |
| Sản phẩm | Phải yêu cầu mới có code | Script đã là code |
| Hạn chế | — | Không "nhìn rồi quyết định" như MCP; phải viết script trước |

### Hai loại script

| Loại | Ví dụ | Giao diện đổi có phải sửa không |
| --- | --- | --- |
| **Script công cụ** (chung) | Chụp màn hình, liệt kê locator, xem console | **Không** |
| **Script test** (theo tính năng) | Test đăng nhập, test mua khóa học | **Có** |

Skill thường đi kèm **script công cụ**, nên vẫn dùng được cho dự án mới. Thứ nên để đến khi tính năng ổn định mới viết là **script test**.

### 25 tool MCP ≠ 25 script
- MCP **giữ trình duyệt mở** giữa các lần gọi tool. Mỗi script thì mở trình duyệt mới rồi đóng, nên tách `click.mts`, `type.mts` sẽ mất trạng thái giữa các bước.
- Script được chia theo **công việc**, không theo **thao tác**. Các thao tác click, type nằm bên trong code của script.

### Khi nào dùng loại nào

| Playwright MCP | SKILL + script | Code test thuần (`@playwright/test`) |
| --- | --- | --- |
| Tính năng mới, giao diện đổi liên tục | Việc lặp lại: chụp màn hình, kiểm tra console | Tính năng đã ổn định |
| Test thăm dò (exploratory) | Cần nhanh, ít token | Regression, chạy CI |
| Tái hiện bug theo mô tả | Team làm theo cùng một quy trình | Cần kết quả lặp lại 100% |
| Lần đầu viết automation, lấy locator | Chạy, sửa, viết thêm test theo khuôn có sẵn | |

> MCP dùng cho việc **chưa biết trước**. Skill + script dùng cho việc **đã biết cách làm**.

---

## 8. Tổ chức automation cho dự án lớn

**Không viết một script cho mỗi test case trong skill.** Chia làm 3 lớp:

| Lớp | Là gì |
| --- | --- |
| **1. Page Object, fixture** | Gói thao tác thành hàm: `home.search()`, `home.filterByCategory()` |
| **2. Test case (spec)** | Ghép các hàm ở lớp 1 với `expect` |
| **3. Skill** | Hướng dẫn AI viết, chạy, sửa test, kèm vài script công cụ |

```
playwright.config.ts
tests/<du-an>/
├── pages/            ← Lớp 1
├── fixtures.ts       ← Lớp 1 (vd: trạng thái đã đăng nhập)
├── auth.setup.ts     ← đăng nhập 1 lần, lưu session dùng lại
└── specs/            ← Lớp 2
.claude/skills/       ← Lớp 3
```

**Quy trình khi có test case mới:**
1. Đưa test case thủ công cho AI.
2. AI đọc `SKILL.md` để biết quy ước.
3. Thiếu locator thì chạy script inspect (hoặc dùng MCP nếu trang phức tạp), rồi bổ sung Page Object.
4. Viết test vào `specs/`.
5. Chạy test, sửa đến khi pass, hoặc xác định là bug của app.

**Lợi ích:** giao diện đổi chỉ cần sửa một chỗ trong Page Object; case mới chủ yếu ghép từ khối có sẵn; skill gần như không đổi.

---

## 9. Ví dụ thực tế trên hoctest.com

### Bước 1: Dùng MCP khám phá và phát hiện bug
Định viết test cho luồng **tìm kiếm khóa học**, nhưng khi khám phá bằng MCP thì thấy:

| Từ khóa | Kết quả |
| --- | --- |
| `Playwright` | "Hiển thị 4 khóa học": ra cả "AI for tester" và "Database" |
| `xyzkhongtontai123` | Vẫn "Hiển thị 4 khóa học" |

Kết luận: **tìm kiếm không lọc theo từ khóa**. Luồng **lọc theo danh mục** (tab Playwright / AI trên trang chủ) thì vẫn hoạt động đúng.

Đây là ví dụ cho thấy giá trị của MCP trong test thăm dò: **tìm ra bug trước khi viết code**.

### Bước 2: Khung automation

| File | Nội dung |
| --- | --- |
| [playwright.config.ts](../../../playwright.config.ts) | baseURL `https://hoctest.com`, Chromium, chụp ảnh và trace khi fail |
| [HomePage.ts](../../../tests/hoctest-com/pages/HomePage.ts) | `goto()`, `search()`, `filterByCategory()`, `categoryTab()` |
| [SearchResultPage.ts](../../../tests/hoctest-com/pages/SearchResultPage.ts) | Danh sách khóa học trong kết quả |
| [filter-course.spec.ts](../../../tests/hoctest-com/specs/filter-course.spec.ts) | Lọc theo Playwright, AI |
| [search-course.spec.ts](../../../tests/hoctest-com/specs/search-course.spec.ts) | Tìm "Playwright", tìm từ khóa không tồn tại |

Ví dụ test chỉ ghép các hàm của Page Object:
```ts
test(`Lọc khóa học phổ biến theo danh mục ${category}`, async ({ page }) => {
  const home = new HomePage(page);
  await home.goto();
  await home.filterByCategory(category);

  await expect(home.categoryTab(category)).toHaveAttribute('aria-selected', 'true');
  await expect(home.popularCourses.filter({ hasText: course })).toBeVisible();
});
```

### Bước 3: Kết quả chạy `npx playwright test`

| Test | Kết quả |
| --- | --- |
| Lọc theo danh mục Playwright | ✅ Pass |
| Lọc theo danh mục AI | ✅ Pass |
| Tìm "Playwright" chỉ hiện khóa học liên quan | ❌ Fail do bug |
| Tìm từ khóa không tồn tại thì không hiện khóa học | ❌ Fail do bug (mong đợi 0, thực tế 4) |

Test fail **đúng**: nó báo bug và sẽ pass khi bug được sửa.

### Bước 4: Bug report
- [bug-search-khong-loc-theo-tu-khoa.md](bug-search-khong-loc-theo-tu-khoa.md), theo mẫu Redmine của project.
- Ảnh bằng chứng: [bug-search-xyzkhongtontai123.png](bug-search-xyzkhongtontai123.png), chụp bằng skill `web-testing`.

### Bước 5: Hai skill đã tạo

**`web-testing`**: chụp màn hình ([SKILL.md](../../../.claude/skills/web-testing/SKILL.md), [screenshot.mts](../../../.claude/skills/web-testing/scripts/screenshot.mts))
```
node --experimental-strip-types --no-warnings .claude/skills/web-testing/scripts/screenshot.mts <url> [--full] [--out <path>] [--width <n>] [--height <n>]
```
Output chỉ có 1 dòng:
```
OK title="Trang chủ - HocTest.Com" saved=products/2026-10/18/skill-test-viewport.png
```

**`write-test`**: quy trình viết test và script liệt kê locator ([SKILL.md](../../../.claude/skills/write-test/SKILL.md), [inspect.mts](../../../.claude/skills/write-test/scripts/inspect.mts))
```
node --experimental-strip-types --no-warnings .claude/skills/write-test/scripts/inspect.mts <url>
```
Ví dụ output:
```
getByRole('textbox', { name: 'Tìm khóa học...' })
getByRole('button', { name: 'Tìm kiếm' })
getByRole('tab', { name: 'Playwright' })
getByRole('link', { name: 'Trang chủ' })  // x2, cần thu hẹp phạm vi
```

**Lưu ý kỹ thuật (Node 22.16):**
- Chạy TypeScript cần cờ `--experimental-strip-types`. Từ Node 22.18 không cần nữa.
- Dùng đuôi `.mts` và cờ `--no-warnings` để bỏ cảnh báo, giữ output ngắn.

---

## 10. Có nên dùng AI Browser Agent chạy E2E test?

> **AI Browser Agent = AI + công cụ trình duyệt** (vd: Claude Code + Playwright MCP), xem [mục 1](#playwright-mcp-có-phải-là-ai-browser-agent-không). Các vấn đề bên dưới chủ yếu đến từ việc **AI phải suy nghĩ lại mỗi lần chạy**, không phải do Playwright, vì bên dưới MCP vẫn là Playwright, vốn là công cụ ổn định.

### AI Browser Agent chạy trực tiếp: 3 vấn đề lớn
1. **Chậm và tốn tiền** mỗi lần chạy.
2. **Kết quả không ổn định** (flaky): khó biết lỗi do app hay do agent.
3. **Không nên đưa vào CI** cho regression test.

Chỉ nên dùng cho **exploratory testing** (khám phá tính năng mới).

### Dùng AI viết Playwright test
- **Regression test phải deterministic**: chạy 10 lần ra cùng 1 kết quả.
- AI viết test, còn **Playwright/Selenium chạy trong CI** cho nhanh và ổn định.
- Quy trình tối ưu hơn:
  - **Viết Gherkin trước khi code** (requirement và test luôn).
  - **AI đề xuất, con người review**: không để AI tự sửa cho test tự pass.
- **Đẩy bớt test xuống Unit/API test**, chỉ giữ E2E cho luồng quan trọng.

### Liên hệ với buổi thực hành
- Dùng MCP để **khám phá** hoctest.com và phát hiện bug tìm kiếm → đúng vai trò exploratory.
- Sau đó **viết code test** `@playwright/test` để chạy lại ổn định → đúng hướng "AI viết test, Playwright chạy".
- Test tìm kiếm được giữ ở trạng thái **fail** vì đó là bug thật, không sửa test cho pass → đúng nguyên tắc "không để AI tự sửa tự pass".

---

## 11. Kinh nghiệm rút ra

> **Dùng AI để viết và đề xuất sửa test, nhưng chạy regression test vẫn cần Playwright/Selenium ổn định. Test phải đáng tin mới có giá trị.**

**Quy trình gợi ý cho dự án mới:**

| Giai đoạn | Công cụ | Việc làm |
| --- | --- | --- |
| Tính năng đang phát triển | Playwright MCP | Test thăm dò, tìm bug |
| Tính năng bắt đầu ổn định | MCP + skill `write-test` | Đi qua luồng chính một lần, lấy locator, viết code test theo Page Object |
| Tính năng đã ổn định | `@playwright/test` + CI | Chạy regression định kỳ |
| Suốt dự án | SKILL + script | Việc lặp lại: chụp màn hình, liệt kê locator, chạy và sửa test |

**Ghi nhớ:**
- MCP: linh hoạt, hợp với việc chưa biết trước, nhưng chậm và tốn token.
- SKILL + script: nhanh, tiết kiệm, hợp với việc đã biết cách làm.
- Code test: ổn định, là thứ dùng cho regression.
- Test fail vì bug thật thì giữ nguyên và log bug, không sửa test cho pass.
