# Lesson 14 — Screenshot Testing, Test Emulation & Drag n Drop

> Tổng hợp từ slide bài giảng (`lesson-14-pdf/`) và code thực hành (`lesson-14/`)

## Agenda (Nội dung bài học)

1. **Visual comparison** (so sánh ảnh / screenshot testing)
2. **Test report** (video recording, trace viewer)
3. **Test emulation** (locale, timezone, color scheme, permission, devices...)
4. **Drag n Drop**

---

## 1. Visual Comparison — So sánh ảnh (Screenshot Testing)

### 1.1. Ý tưởng cốt lõi

Thay vì assert từng thuộc tính (text, màu sắc, vị trí...), Playwright cho phép **chụp ảnh toàn trang/element và so sánh với ảnh chuẩn (baseline)** đã lưu trước đó. Nếu giao diện thay đổi dù chỉ một vài pixel, test sẽ fail — rất hữu ích để phát hiện các lỗi UI "vô tình" (layout vỡ, css sai, thiếu ảnh...).

```ts
import { test, expect } from '@playwright/test';

test('example test', async ({ page }) => {
  await page.goto('https://playwright.dev');
  await expect(page).toHaveScreenshot();
});
```

### 1.2. Vòng đời chạy `toHaveScreenshot()`

| Lần chạy | Điều gì xảy ra |
|---|---|
| **Lần đầu tiên** | Chưa có ảnh baseline → Playwright báo lỗi *"A snapshot doesn't exist..., writing actual"* và tự sinh ảnh baseline mới (lưu trong thư mục `*.spec.ts-snapshots/`). |
| **Lần thứ 2 trở đi** | So sánh ảnh chụp hiện tại với baseline đã lưu. Nếu giống nhau → **pass**. |
| **Khi UI thay đổi** | Ảnh hiện tại khác baseline → **fail**, báo số pixel khác biệt, ví dụ: `4761 pixels (ratio 0.01 of all image pixels) are different.` |

Cơ chế nội bộ khi so sánh (xem trong log `Call log`):
- Tắt toàn bộ CSS animation (`disabled all CSS animations`)
- Đợi font load xong (`waiting for fonts to load...`)
- Chụp ảnh, so sánh — nếu chưa khớp thì **đợi 100ms và chụp lại** (retry) trước khi kết luận fail hẳn — giúp tránh false-fail do trang chưa render ổn định.

### 1.3. Khi test fail — 3 loại ảnh debug

Mỗi lần `toHaveScreenshot()` fail, Playwright tự sinh 3 ảnh trong `test-results/...`:

- **expected** — ảnh baseline gốc (ảnh chuẩn)
- **actual** — ảnh chụp thực tế lúc chạy test
- **diff** — ảnh highlight (tô đỏ) đúng những vùng pixel khác nhau giữa 2 ảnh trên

Trong **HTML report** (`npx playwright show-report`) hoặc **Trace Viewer**, phần "Image mismatch" cho phép xem theo **5 chế độ**: `Diff`, `Actual`, `Expected`, `Side by side`, `Slider` (kéo thanh trượt để so sánh chồng ảnh) — rất trực quan để tìm ra điểm khác biệt.

### 1.4. Cập nhật baseline khi thay đổi là có chủ đích

Khi UI thay đổi **hợp lệ** (chủ động sửa giao diện), cần cập nhật lại ảnh baseline thay vì để test fail mãi:

```bash
npx playwright test tests/.../01-screenshot.spec.ts --update-snapshots

# Chạy cho 1 nhóm test cụ thể có gắn tag
npx playwright test -g "@IMAGE" --update-snapshots
```

Có thể xóa file ảnh cũ trong thư mục snapshots rồi chạy lại để Playwright tự sinh ảnh mới.

### 1.5. Mỗi trình duyệt = 1 ảnh baseline riêng

Vì Chromium / Firefox / WebKit render trang hơi khác nhau (font, anti-alias...), Playwright lưu **baseline riêng cho từng browser + OS**:

```
example-test-1-chromium-win32.png
example-test-1-firefox-win32.png
example-test-1-webkit-win32.png
```

→ Khi chạy đa trình duyệt (`projects` trong config), lần đầu cả 3 browser đều chưa có snapshot nên đều "writing actual", pass hết ở lần chạy này.

### 1.6. Chụp toàn trang — `fullPage: true`

Mặc định `toHaveScreenshot()` chỉ chụp phần đang hiển thị trong viewport. Muốn chụp **toàn bộ trang kể cả phần phải cuộn xuống mới thấy**, truyền thêm option `fullPage: true`:

```ts
test("Example for screenshot full page", async ({ page }) => {
    await page.goto('https://playwright.dev');
    await expect(page).toHaveScreenshot({
        fullPage: true,
    });
})
```

Lưu ý: ảnh full page thường **cao hơn nhiều** so với ảnh viewport thường, và cũng nhạy hơn với nội dung động nằm ở những vùng xa phía dưới trang (ví dụ footer, danh sách dài...) — nên càng cần cân nhắc dùng `mask` đi kèm nếu trang có phần nội dung hay đổi.

### 1.7. Tên file ảnh baseline được đặt tự động theo tên test

Playwright tự sinh tên file `.png` trong thư mục `*.spec.ts-snapshots/` dựa theo: **tên `describe` (nếu có) + tên test** (dấu cách chuyển thành dấu gạch ngang) + **số thứ tự lần gọi `toHaveScreenshot()`** trong test đó + **tên project** + **platform (OS)**. Ví dụ với test nằm trong `describe("Test video recording & screenshot full page")` và tên test `"Test creenshot & mask color"`, ảnh baseline sinh ra là:

```
Test-video-recording-screenshot-full-page-Test-creenshot-mask-color-1-chromium-win32.png
```

→ Nếu một test gọi `toHaveScreenshot()` nhiều lần, số thứ tự (`-1-`, `-2-`...) sẽ tăng dần để phân biệt từng lần chụp trong cùng 1 test.

### 1.8. Kết hợp `mask` + `fullPage` trong thực tế (ví dụ đăng nhập WordPress admin)

Có thể dùng đồng thời nhiều kỹ thuật (che nhiều vùng động, chụp full page) trong cùng một luồng nghiệp vụ thật — ví dụ đăng nhập vào trang quản trị rồi kiểm tra giao diện Dashboard/Tags:

```ts
test.describe("Test video recording & screenshot full page", () => {

    test.beforeEach("Go to practice page", async ({ page }) => {
        await test.step("Go to page", async () => {
            await page.goto("https://pw-practice-dev.playwrightvn.com/wp-admin");
        })
        // ... login steps (xem test.step bên dưới)
    });

    test("Test creenshot & mask color", async ({ page }) => {
        await expect(page).toHaveScreenshot({
            mask: [
                page.locator('#dashboard_activity'),
                page.locator('#dashboard_right_now'),
            ],
            maskColor: '#7134eb',
        });
    });

    test('Test screenshot full page & mask color', async ({ page }) => {
        await page.getByRole('link', { name: 'Posts', exact: true }).click();
        await page.getByRole('link', { name: 'Tags', exact: true }).click();

        await expect(page).toHaveScreenshot({
            fullPage: true,
            mask: [page.locator('#the-list')],
            maskColor: '#1f10f1',
        });
    });
});
```

Điểm hay ở ví dụ này: trang Dashboard của WordPress có nhiều widget đổi nội dung liên tục (activity feed, "right now" stats, danh sách bài viết `#the-list`) — nếu không `mask` các vùng này, ảnh sẽ luôn khác biệt dù giao diện không hề lỗi.

### 1.9. Che vùng nội dung động — `mask` & `maskColor`

**Vấn đề:** Một số phần tử trên trang thay đổi liên tục (banner quảng cáo, đồng hồ, ảnh random...) khiến ảnh không bao giờ khớp 100% dù giao diện thực chất không lỗi.

**Giải pháp:** Dùng option `mask` để **che phủ (tô 1 màu solid)** những vùng đó trước khi so sánh, giúp bỏ qua sự khác biệt ở khu vực này.

```ts
await expect(page).toHaveScreenshot({
  mask: [
    page.locator('#ads-here'),                    // theo CSS selector
    page.locator('//a[@href="index.html"]'),      // theo XPath
  ],
  maskColor: '#1f10f1', // tùy chỉnh màu che (mặc định là màu hồng/pink)
});
```

Điểm quan trọng:
- Có thể truyền **nhiều locator** cùng lúc vào mảng `mask` (kết hợp cả CSS selector và XPath).
- `maskColor` không bắt buộc — nếu bỏ qua, Playwright dùng màu hồng mặc định.
- Vùng bị mask sẽ hiển thị là 1 khối màu đặc trong cả ảnh baseline lẫn ảnh thực tế → không còn ảnh hưởng tới kết quả so sánh.

### 1.10. Bẫy thường gặp: baseline sai domain / sai viewport

- Nếu code `page.goto()` trỏ nhầm sang domain/trang khác so với lúc tạo baseline → ảnh khác biệt gần như hoàn toàn (ví dụ 31% pixel khác nhau) → fail nặng. Đây không phải lỗi tool mà là lỗi code test.
- Nếu đổi `viewport` (kích thước màn hình) trong config nhưng baseline cũ được chụp ở kích thước khác → test fail vì kích thước ảnh không khớp. Cần `--update-snapshots` lại baseline sau khi đổi viewport.

⚠️ Lưu ý code quan trọng khi cấu hình `viewport` cùng `devices`:
```ts
{
  name: 'chromium',
  use: {
    ...devices['Desktop Chrome'],
    // Phải khai báo viewport SAU khi destructure devices,
    // vì devices cũng tự định nghĩa sẵn viewport riêng —
    // khai báo sau mới ghi đè được.
    viewport: { width: 1280, height: 720 },
  },
},
```

---

## 2. Video Recording & Test Report

### 2.1. Quay video khi chạy test

Cấu hình trong `playwright.config.ts`:

```ts
export default defineConfig({
  use: {
    video: {
      mode: 'on',                              // on / off / retain-on-failure / on-first-retry
      size: { width: 640, height: 480 },
    },
  },
});
```

Video giúp xem lại toàn bộ quá trình chạy test (hữu ích khi debug lỗi khó tái hiện).

**Cấu hình thực tế đang dùng trong project** (`playwright.config.ts`):

```ts
use: {
  trace: 'on',
  headless: false,
  video: 'on',
},
```

⚠️ Lưu ý: `trace: 'on'` và `video: 'on'` sẽ ghi trace/video cho **mọi test, kể cả test pass** → dung lượng report tăng nhanh, phù hợp khi đang học/debug. Trong CI hoặc project thật, nên cân nhắc dùng `'on-first-retry'` hoặc `'retain-on-failure'` để chỉ giữ lại trace/video của các test **fail hoặc retry**, tránh tốn dung lượng lưu trữ không cần thiết. `headless: false` cũng chỉ nên bật khi cần quan sát trực tiếp trình duyệt lúc phát triển test, không nên bật khi chạy trên CI.

### 2.2. Trace Viewer

Bật trace: `trace: 'on'` trong config. Khi test chạy xong, xem lại bằng:

```bash
npx playwright show-trace path/to/trace.zip
```

Trace Viewer cho phép xem:
- **Timeline** thao tác theo mốc thời gian (mỗi action mất bao lâu)
- Thông tin môi trường: browser engine, OS, viewport, user agent, device scale...
- Các tab: **Actions, Call log, Errors, Console, Network, Source, Attachments**
- Xem **snapshot Before/After** của từng action → biết chính xác lúc nào trang thay đổi

→ Đây là công cụ debug mạnh nhất của Playwright khi cần biết chính xác điều gì đã xảy ra trong lúc test chạy.

### 2.3. Gom nhóm bước bằng `test.step()`

Khi 1 test có nhiều thao tác (điền form, click nút, kiểm tra kết quả...), có thể gom từng nhóm hành động lại bằng `test.step()` để **Trace Viewer và HTML report hiển thị theo từng bước có tên rõ ràng**, dễ đọc hơn là 1 danh sách action phẳng:

```ts
test.beforeEach("Go to practice page", async ({ page }) => {
  await test.step("Go to page", async () => {
    await page.goto("https://pw-practice-dev.playwrightvn.com/wp-admin");
  });

  await test.step("Input username & password", async () => {
    await page.getByRole('textbox', { name: 'Username or Email Address' }).fill(userName);
    await page.getByRole('textbox', { name: 'Password' }).fill(psw);
  });

  await test.step("Click login button", async () => {
    await page.getByRole('button', { name: 'Log In' }).click();
    await expect(page).toHaveURL("https://pw-practice-dev.playwrightvn.com/wp-admin/");
  });
});
```

Lợi ích: khi test fail, biết ngay fail ở **bước nào** (ví dụ "Click login button") thay vì phải dò từng dòng code; cũng dùng được cho cả `beforeEach` chứ không chỉ trong `test()`.

---

## 3. Test Emulation — Giả lập môi trường trình duyệt

Playwright hỗ trợ giả lập (emulate) nhiều thứ để test các kịch bản khác nhau mà không cần thiết bị/vị trí thật:

- Devices (thiết bị di động)
- Viewport (kích thước màn hình)
- Locale & timezone (ngôn ngữ & múi giờ)
- Color scheme (giao diện sáng/tối)
- Geolocation (vị trí địa lý)
- Permission (quyền truy cập camera, mic, thông báo...)

📚 Tài liệu: `playwright.dev/docs/emulation`

### 3.1. Locale & Timezone

**Cấu hình toàn cục** trong `playwright.config.ts` — áp dụng cho **mọi test**:

```ts
export default defineConfig({
  use: {
    locale: 'en-GB',                 // giả lập ngôn ngữ trình duyệt
    timezoneId: 'Europe/Paris',      // giả lập múi giờ
  },
});
```

Kiểm chứng bằng cách mở trang "Detect User Agent" — trang sẽ hiển thị đúng TimeZone = `Europe/Paris`, Locale = `en-GB` như đã cấu hình.

**Cấu hình ở cấp test/file** bằng `test.use()` — sẽ **ghi đè** cấu hình toàn cục:

```ts
import { test, expect } from '@playwright/test';

test.use({
  locale: 'de-DE',
  timezoneId: 'Europe/Berlin',
});

test('example test', async ({ page }) => {
  await page.goto('https://material.playwrightvn.com/017-detect-user-agent.html');
});
```
→ Kết quả: trang hiển thị TimeZone = `Europe/Berlin`, Locale = `de-DE` (ghi đè global thành công).

**Cấu hình theo nhóm test** bằng `test.describe()` — chỉ áp dụng cho các test bên trong group đó, các group khác không bị ảnh hưởng:

```ts
test.describe('Group using emulation', () => {
  test.use({
    locale: 'de-DE',
    timezoneId: 'Europe/Berlin',
  });

  test('example test', async ({ page }) => {
    await page.goto('https://material.playwrightvn.com/017-detect-user-agent.html');
  });
});

test.describe('Group not using emulation', () => {
  test('example test', async ({ page }) => {
    await page.goto('https://material.playwrightvn.com/017-detect-user-agent.html');
  });
});
```

**Thứ tự ưu tiên (từ hẹp tới rộng):** `test.use()` trong 1 test/describe cụ thể > cấu hình global trong `playwright.config.ts`.

### 3.2. Permissions (quyền truy cập)

**Vấn đề:** Nếu trang web yêu cầu quyền (vị trí, camera, mic, thông báo) mà chưa cấu hình trước, trình duyệt sẽ hiện **popup xin quyền** (Allow / Allow this time / Never allow) → có thể làm test bị đứng hoặc không lấy được dữ liệu cần thiết (ví dụ: geolocation).

**Giải pháp:** Khai báo sẵn `permissions` trong config để tự động cấp quyền, không cần popup:

```ts
export default defineConfig({
  use: {
    permissions: ['notifications', 'microphone', 'camera', 'geolocation'],
  },
});
```

Sau khi cấu hình, kiểm tra lại trang sẽ thấy Location/Camera/Microphone đều ở trạng thái **"Đã cấp quyền"** thay vì "Chưa cấp quyền".

Danh sách đầy đủ giá trị hợp lệ (theo tài liệu `BrowserContext.grantPermissions`):
`accelerometer, ambient-light-sensor, background-sync, camera, clipboard-read, clipboard-write, geolocation, gyroscope, local-fonts, local-network-access, magnetometer, microphone, midi-sysex, midi, notifications, payment-handler, storage-access, screen-wake-lock`

Cũng có thể set `permissions` riêng cho từng `project` (ví dụ chỉ chromium mới cần notifications + microphone + geolocation):

```ts
projects: [
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome'],
      permissions: ['notifications', 'microphone', 'geolocation'],
    },
  },
],
```

**Kết hợp nhiều loại emulation trong cùng 1 `test.use()`:** `permissions`, `locale`, `timezoneId`... đều là các field của cùng object `use`, nên có thể khai báo chung một lần cho gọn thay vì tách nhiều `test.use()`:

```ts
test.use({
  permissions: ["notifications", 'microphone', 'camera', 'geolocation'],
  locale: 'en-GB',
  timezoneId: 'Europe/Paris',
});

test('Example for permission', async ({ page }) => {
  await page.goto('https://material.playwrightvn.com/017-detect-user-agent.html');
});
```

### 3.3. Color Scheme (Dark/Light mode)

Giả lập giao diện sáng/tối của hệ điều hành/trình duyệt:

```ts
test.use({
  colorScheme: 'dark', // hoặc 'light'
});

test('example test', async ({ page }) => {
  await page.goto('https://playwright.dev/');
});
```

Khi set `colorScheme: 'dark'`, trang web (nếu có hỗ trợ dark mode theo `prefers-color-scheme`) sẽ tự động hiển thị giao diện nền tối — hữu ích để test giao diện dark mode mà không cần đổi setting hệ điều hành thật.

📌 **Cấu hình thực tế trong project** (`playwright.config.ts`) đang set global `permissions: ['notifications', 'microphone', 'camera', 'geolocation']` và `colorScheme: 'dark'` cho **mọi test** — nghĩa là mặc định mọi test trong repo này chạy với quyền đã được cấp sẵn và giao diện dark mode, trừ khi 1 test/describe cụ thể dùng `test.use()` để ghi đè riêng.

### 3.4. Devices & Viewport (thiết bị & kích thước màn hình)

Playwright có sẵn rất nhiều device descriptor dựng sẵn (định nghĩa trong `playwright-core/types/types.d.ts`), ví dụ: `iPhone 13`, `iPad (gen 5)`, `Galaxy S24`, `Desktop Chrome`, `Desktop Safari`... Dùng trong `projects` để test responsive/mobile:

```ts
projects: [
  {
    name: 'Mobile Safari',
    use: { ...devices['iPad (gen 5)'] },
  },
  {
    name: 'firefox',
    use: { ...devices['Desktop Firefox'] },
  },
  {
    name: 'webkit',
    use: { ...devices['Desktop Safari'] },
  },
],
```

→ Mỗi project sẽ chạy lại toàn bộ test suite với thiết bị/trình duyệt tương ứng, và (nếu có screenshot test) mỗi project cũng có baseline ảnh riêng theo tên project + OS.

---

## 4. Drag n Drop — Kéo thả

Playwright hỗ trợ 2 cách thực hiện thao tác kéo-thả:

### 4.1. Cách 1 — Dùng method `dragTo()` có sẵn (đơn giản, khuyên dùng)

```ts
await page.locator('#item-to-be-dragged').dragTo(page.locator('#item-to-drop-at'));
```

Ví dụ thực hành — giải puzzle bằng cách kéo từng mảnh ghép vào đúng vị trí (dùng vòng lặp cho 4 mảnh ghép):

```ts
test('Puzzle 01 - using Drag to', async ({ page }) => {
  await page.goto('https://material.playwrightvn.com/05-xpath-drag-and-drop.html');

  for (let i = 1; i <= 4; i++) {
    let fromloc = page.locator(`#piece-${i}`);
    let toLoc = page.locator(`//div[@data-piece='${i}']`);
    await fromloc.dragTo(toLoc);
  }
});
```

### 4.2. Cách 2 — Giả lập thủ công bằng hover + mouse down/up

Dùng khi `dragTo()` không hoạt động đúng với 1 số trang web đặc thù (ví dụ trang dùng thư viện kéo-thả custom, không phản hồi tốt với HTML5 drag event mà `dragTo()` giả lập):

```ts
await page.locator('#item-to-be-dragged').hover();
await page.mouse.down();
await page.locator('#item-to-drop-at').hover();
await page.mouse.up();
```

Áp dụng cho bài puzzle tương tự:

```ts
test('Puzzle 01 - using Drag to', async ({ page }) => {
  await page.goto('https://material.playwrightvn.com/05-xpath-drag-and-drop.html');

  for (let i = 1; i <= 4; i++) {
    let fromloc = page.locator(`#piece-${i}`);
    let toLoc = page.locator(`//div[@data-piece='${i}']`);

    await fromloc.hover();
    await page.mouse.down();
    await toLoc.hover();
    await page.mouse.up();
  }
});
```

**So sánh nhanh:**

| | `dragTo()` | Thủ công (hover + mouse down/up) |
|---|---|---|
| Độ đơn giản | Rất đơn giản, 1 dòng code | Nhiều bước hơn |
| Độ tin cậy | Hoạt động tốt với hầu hết trang web chuẩn | Cần dùng khi trang web có cơ chế kéo-thả đặc biệt mà `dragTo()` không mô phỏng đúng được (ví dụ cần giữ chuột 1 khoảng thời gian, di chuyển qua nhiều điểm trung gian...) |
| Khi nào dùng | Ưu tiên dùng trước | Dùng khi cách 1 không hoạt động như mong đợi |

### 4.3. Điều hướng qua UI thay vì `goto()` thẳng tới trang đích

Thay vì `page.goto()` thẳng đến URL của bài puzzle, có thể mô phỏng đúng hành vi người dùng thật: vào trang chủ rồi **click** vào link tương ứng để đến trang cần test:

```ts
test('Puzzle 01 - using Drag to', async ({ page }) => {
  await page.goto('https://material.playwrightvn.com/');
  await page.getByText('Bài học 5: Puzzle drag and drop game').click();

  for (let i = 1; i <= 4; i++) {
    let fromloc = page.locator(`#piece-${i}`);
    let toLoc = page.locator(`//div[@data-piece='${i}']`);
    await fromloc.dragTo(toLoc);
  }
});
```

Cách này giúp test bao phủ luôn cả bước điều hướng (navigation) trong luồng người dùng thật, thay vì chỉ test riêng lẻ trang đích — đánh đổi là test sẽ phụ thuộc thêm vào link/text ở trang chủ (nếu đổi tên link, test có thể fail dù tính năng drag-drop vẫn đúng).

---

## Tổng kết bài học

| Chủ đề | Nội dung chính |
|---|---|
| **Visual comparison** | `toHaveScreenshot()`, `fullPage: true` (chụp toàn trang), baseline theo từng browser, tên file baseline sinh tự động theo tên describe/test, `--update-snapshots`, debug bằng Diff/Actual/Expected/Side-by-side/Slider, che nội dung động bằng `mask` + `maskColor` (kết hợp được với `fullPage`) |
| **Test report** | Quay video (`video: { mode, size }`), xem lại chi tiết bằng Trace Viewer, gom nhóm hành động rõ ràng trong report bằng `test.step()` |
| **Test emulation** | `locale`, `timezoneId` (global / test level / describe-group level), `permissions` (tránh popup xin quyền, có thể khai báo chung với `locale`/`timezoneId` trong 1 `test.use()`), `colorScheme` (dark/light), `devices` + `viewport` (responsive/mobile) |
| **Drag n Drop** | `dragTo()` (khuyên dùng trước) và cách thủ công `hover() → mouse.down() → hover() → mouse.up()` (dùng khi cần); có thể điều hướng tới trang đích qua `goto()` trực tiếp hoặc qua click UI như người dùng thật |

**Nguyên tắc chung cần nhớ:** cấu hình càng ở phạm vi hẹp (test/describe) thì mức ưu tiên càng cao và sẽ ghi đè cấu hình ở phạm vi rộng hơn (global config) — áp dụng cho cả `locale/timezoneId`, `colorScheme`, `permissions`.
