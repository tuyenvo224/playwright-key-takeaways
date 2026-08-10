# Lesson 14 — Visual Screenshot Testing với Playwright

Tổng hợp toàn bộ kiến thức về `toHaveScreenshot()` đã học/thảo luận: cơ chế hoạt động, baseline, mask, update snapshot, trace viewer, và các vấn đề môi trường thường gặp.

---

## 1. `toHaveScreenshot()` là gì?

```ts
import { test, expect } from '@playwright/test';

test('example test', async ({ page }) => {
  await page.goto('https://playwright.dev');
  await expect(page).toHaveScreenshot();
});
```

Đây là **visual regression testing** (kiểm thử hồi quy giao diện): chụp ảnh màn hình hiện tại (**actual**) rồi so sánh pixel-by-pixel với một ảnh "chuẩn" đã lưu trước đó (**baseline / expected**).

### Ứng dụng thực tế

- Phát hiện khi một thay đổi code (CSS global, refactor component dùng chung...) vô tình làm **vỡ layout** ở một tính năng khác mà không ai để ý.
- Dùng trong CI/CD để tự động cảnh báo trước khi merge PR.
- Phổ biến trong các dự án có design system, landing page, dashboard — nơi "đúng về hình ảnh" quan trọng không kém "đúng về chức năng".

> ⚠️ **Lưu ý quan trọng**: mục đích của công cụ này là bảo vệ một layout **đã được duyệt/chốt trước đó** khỏi bị thay đổi ngoài ý muốn trong tương lai — **không phải** để kiểm tra "layout tôi đang code có khớp thiết kế Figma không" (xem mục 5).

---

## 2. Vì sao chạy lần 1 FAIL, lần 2 PASS?

### Lần chạy 1 — chưa có baseline

Thư mục `<tên-file>.spec.ts-snapshots/` chưa có ảnh nào (ví dụ `example-test-1-chromium-win32.png`):

- Playwright thấy chưa có baseline → **tự động chụp ảnh và lưu làm baseline luôn** (không cần bạn tự tay copy ảnh "expected" vào).
- Nhưng nó vẫn **báo FAIL** với message kiểu: *"A snapshot doesn't exist... it has been generated"*.
- Đây là **chủ ý thiết kế**: bắt buộc con người phải tự mở ảnh vừa sinh ra để review xem có đúng ý muốn không, trước khi coi nó là chuẩn chính thức (rồi commit vào git).

### Lần chạy 2 — đã có baseline

- Playwright chụp **một ảnh MỚI** tại thời điểm chạy lần 2 (gọi là `actual`).
- So sánh `actual` này với `expected` (ảnh đã lưu từ lần 1, **không chụp lại**).
- Vì trang web không đổi gì giữa 2 lần chạy → 2 ảnh giống nhau → **PASS**.

### ❌ Hiểu lầm thường gặp

> "Lần 2 nó so sánh ảnh vừa snapshot với ảnh cũng vừa snapshot nên chắc chắn pass, vô nghĩa?"

**Sai.** Đây là **2 lần chụp khác nhau ở 2 thời điểm khác nhau**:
- `expected` = ảnh chụp & lưu ở **lần chạy 1**.
- `actual` = ảnh chụp **mới, thật sự mới** ở **lần chạy 2**.

Chỉ vì nội dung trang không đổi nên 2 ảnh giống hệt nhau — nếu giữa 2 lần chạy trang web có gì thay đổi (banner mới, layout đổi), lần 2 vẫn **fail** dù đã có baseline.

---

## 3. Baseline không tự "outdate rồi hết fail" — phải cập nhật thủ công

**Tình huống**: layout đổi hợp lệ (ví dụ: đổi từ theme sáng → tối), baseline cũ giờ không còn đúng nữa.

- Chạy `npx playwright test` bình thường → actual (tối) khác baseline cũ (sáng) → **FAIL**. Đây là tín hiệu đúng, hữu ích — báo "có gì đó đã đổi, kiểm tra xem có chủ ý không".
- **Quan trọng**: nó sẽ **FAIL MÃI MÃI** ở mọi lần chạy sau, không tự hết fail dù chạy lại bao nhiêu lần — vì baseline là 1 file tĩnh, không có cơ chế tự "học" rằng thay đổi này là hợp lệ. Chỉ con người mới quyết định được điều đó.

### 2 cách cập nhật baseline mới

**Cách 1 — xóa file cũ rồi chạy lại (2 bước):**
```bash
# 1. Xóa file baseline cũ trong thư mục *-snapshots
# 2. Chạy lại — Playwright tự sinh & lưu baseline mới, nhưng LẦN NÀY VẪN BÁO FAIL
npx playwright test
# 3. Chạy thêm 1 lần nữa mới thấy PASS
npx playwright test
```

**Cách 2 — dùng flag `--update-snapshots` (khuyên dùng, 1 bước, phổ biến hơn):**
```bash
npx playwright test --update-snapshots
```
Flag này bảo Playwright: "đừng so sánh rồi fail, cứ chụp ảnh mới rồi **ghi đè trực tiếp** lên baseline cũ" → báo **updated/passed ngay**, không cần chạy thêm lần nào.

> Test code (`toHaveScreenshot()`) **không bao giờ cần sửa** dù là gen lần đầu, so sánh, hay update baseline — chỉ khác cách gọi CLI hoặc việc xóa file ảnh hay không.

---

## 4. Baseline phải commit vào Git

File ảnh trong thư mục `*-snapshots/` (ví dụ `example-test-1-chromium-win32.png`) cần được **commit vào git** để cả team/CI dùng chung 1 baseline — nếu không, mỗi máy sẽ tự sinh baseline riêng và test trở nên vô nghĩa (luôn so ảnh máy đó với chính nó).

### Tên file baseline chứa tên OS (`win32`, `linux`, `darwin`)

Vì font rendering / anti-aliasing khác nhau giữa Windows / macOS / Linux → baseline chụp trên Windows sẽ **fail** nếu CI chạy trên Linux (dù giao diện nhìn y hệt bằng mắt thường).

**Best practice**: chạy visual test trong Docker hoặc CI với môi trường cố định để tránh sai lệch do khác OS.

---

## 5. KHÔNG dùng `toHaveScreenshot()` để so sánh với thiết kế Figma khi đang dev

### Về mặt kỹ thuật: có thể

Chỉ cần đặt đúng tên/đường dẫn file PNG vào thư mục `-snapshots`, Playwright sẽ dùng nó làm baseline — kể cả ảnh đó là export từ Figma.

### Nhưng về mặt thực tế: không nên, vì gần như luôn fail giả

| Nguyên nhân | Giải thích |
|---|---|
| Nguồn render khác nhau | Figma xuất ảnh vector/raster lý tưởng; browser render bằng pixel thật — font rendering, anti-aliasing, subpixel positioning khác nhau dù nhìn "giống hệt" bằng mắt |
| So sánh quá chặt | `toHaveScreenshot()` mặc định so khá chặt (threshold nhỏ), thiết kế cho việc "so ứng dụng của mình với chính nó ở 2 thời điểm", không phải "so thiết kế với triển khai" |
| Nới threshold = mất ý nghĩa | Nếu nới lỏng đủ để pass, nó không còn đáng tin cậy để bắt regression thật sau này |
| Baseline đổi liên tục | Khi đang code, feature còn thay đổi liên tục → phải update baseline liên tục → tạo noise, không tạo giá trị |

### Vậy nên dùng gì trong giai đoạn dev?

- **So layout với Figma khi đang code**: review bằng mắt/design QA, hoặc dùng công cụ chuyên dụng (Figma Dev Mode, Percy, Chromatic, Applitools Eyes — các tool này có AI tolerance cho khác biệt nhỏ, tốt hơn nhiều so với pixel-diff thuần của Playwright).
- **Sau khi feature được duyệt (design sign-off)**: lúc này mới chạy `toHaveScreenshot()` để sinh baseline chính thức và commit — bảo vệ layout đã chốt khỏi bị các thay đổi tương lai làm vỡ.

> Tóm gọn: `toHaveScreenshot()` trả lời *"layout có bị đổi so với lúc tôi chốt nó không?"*, không trả lời *"layout tôi đang code có khớp Figma không?"* — 2 bài toán khác nhau, cần công cụ khác nhau.

---

## 6. `mask` — che các phần tử động khi chụp ảnh

```ts
await expect(page).toHaveScreenshot({
  mask: [
    page.locator('#ads-here'),
    page.locator('//a[@href="index.html"]'),
  ],
  maskColor: '#1f10f1',
});
```

### Vì sao cần `mask`?

Các phần tử động (quảng cáo, đồng hồ, avatar random, nội dung đổi mỗi lần load...) sẽ khiến ảnh actual khác ảnh baseline mỗi lần chạy → test **flaky** dù layout thực chất không đổi. `mask` che vùng đó bằng 1 màu đặc trước khi so sánh, để bỏ qua sự khác biệt trong vùng đó.

### ⚠️ Bẫy: alpha channel = 0 trong `maskColor`

```ts
maskColor: '#4104eb00'   // ❌ SAI
```

Mã hex 8 ký tự có format `#RRGGBBAA`:
- `41 04 eb` → RGB (màu tím/indigo)
- `00` → **alpha (độ đục) = 0 = hoàn toàn trong suốt**

→ Ô màu này về bản chất **không che được gì cả** (giống tô bằng cọ không mực). Vùng `#ads-here` vẫn hiển thị nội dung gốc trong ảnh chụp → test vẫn có thể fail/flaky, đúng vấn đề mà `mask` sinh ra để giải quyết nhưng lại không hoạt động.

**Cách sửa** — đảm bảo alpha = 100% (đục hoàn toàn):
```ts
maskColor: '#4104eb'      // 6 ký tự, mặc định opaque
// hoặc
maskColor: '#4104ebff'    // alpha = ff = 100%
```
Mặc định của Playwright nếu không set `maskColor` là màu hồng đục `#FF00FF`.

> **Mẹo nhận biết trong VSCode**: extension color-decorator sẽ hiển thị 1 ô vuông màu ngay trước mã hex trong code. Nếu ô đó hiện trắng/trống dù mã màu có vẻ "có màu" → kiểm tra ngay 2 ký tự cuối (alpha), rất có thể alpha đang = `00`.

---

## 7. Trace Viewer — Network tab hiển thị khác nhau giữa các máy

Khi mở trace (`localhost:9323/trace/...`), có thể thấy tab Network chi tiết khác nhau:

| Bản cũ | Bản mới (Playwright hiện tại) | Ý nghĩa |
|---|---|---|
| Request | Headers | Header của request gửi đi (referer, user-agent, sec-ch-ua...) |
| Body | Payload | Dữ liệu gửi kèm request (query params, POST body...) |
| Response | Response | Nội dung response trả về |

### Nguyên nhân

Trace Viewer **không phải một phần cố định đóng gói trong file `trace.zip`** — nó là ứng dụng web được đóng gói sẵn bên trong package `@playwright/test`/`playwright-core` **cài trên máy của người mở trace**. Ai mở trace bằng version Playwright nào thì thấy UI của version đó.

Playwright thỉnh thoảng redesign Network tab để giống Chrome DevTools hơn (*Headers / Payload / Preview / Response / Timing*). Nếu 2 máy có version Playwright khác nhau (kiểm tra bằng `npx playwright --version`), UI hiển thị sẽ khác — **không phải lỗi**, chỉ khác tên/nhóm tab, dữ liệu bên trong tương đương nhau.

---

## 8. Ghi chú môi trường (VSCode / Windows)

### Mở Settings trong VSCode trên Windows

- Phím tắt: `Ctrl + ,`
- Menu: `File → Preferences → Settings`
- Command Palette: `Ctrl + Shift + P` → `Preferences: Open Settings (UI)`
- Tìm nhanh 1 setting cụ thể: gõ từ khóa vào ô search phía trên (ví dụ `copy relative path separator`).

### Chạy test qua Test Explorer nhưng không thấy browser UI mở lên

Dù đã bật `headless: false` trong config **và** tick "Show browser" trong extension Playwright, đôi khi vẫn không thấy cửa sổ trình duyệt. Nguyên nhân thường gặp:

1. **Test chạy quá nhanh** (2-3 giây): trình duyệt mở → chụp ảnh → đóng ngay khi test xong, không có bước chờ nào (`waitForTimeout`, `pause`) nên rất dễ bỏ lỡ bằng mắt thường.
2. **WebKit không tự "cướp focus"** như Chromium: trên Windows, cửa sổ WebKit đôi khi mở **phía sau** cửa sổ đang active (VSCode) thay vì tự nhảy lên trước, nên có thể nó đã mở & đóng mà bạn không thấy vì bị che.

**Cách xác nhận / khắc phục:**
```ts
// Thêm tạm dòng chờ để có thời gian quan sát
await page.waitForTimeout(5000);
```
hoặc chạy debug mode (chắc chắn dừng lại, browser luôn được focus):
```bash
npx playwright test path/to/file.spec.ts --debug
```
hoặc bật **"Show trace viewer"** trong panel PLAYWRIGHT của extension — xem lại toàn bộ step-by-step kèm ảnh before/after sau khi chạy xong, không cần canh live view.

---

## 9. Tóm tắt quy trình chuẩn khi làm việc với visual screenshot testing

```mermaid
flowchart TD
    A["Feature đã code xong,<br/>được design duyệt (sign-off)"] --> B["Chạy toHaveScreenshot() lần đầu"]
    B --> C["Playwright tự sinh baseline<br/>+ báo FAIL để bắt buộc review"]
    C --> D["Dev tự kiểm tra ảnh vừa sinh<br/>bằng mắt — đúng ý muốn chưa?"]
    D -->|Đúng| E["Commit ảnh baseline vào git"]
    E --> F["Từ nay, mọi lần chạy test<br/>sẽ bảo vệ layout này khỏi bị đổi ngoài ý muốn"]
    F --> G{"Có thay đổi code khác<br/>ảnh hưởng layout này?"}
    G -->|Không đổi gì| H["actual == baseline → PASS"]
    G -->|Layout đổi ngoài ý muốn| I["actual != baseline → FAIL<br/>→ phát hiện regression!"]
    G -->|Layout đổi CÓ CHỦ Ý<br/>(redesign hợp lệ)| J["Chạy: npx playwright test --update-snapshots"]
    J --> E
```

### Checklist khi viết test loại này

- [ ] Baseline đã được review bằng mắt trước khi commit, chưa vội tin ảnh tự sinh ra là "đúng"
- [ ] Baseline đã commit vào git (không phải file local tự sinh mỗi máy)
- [ ] Nếu test chạy trên nhiều OS/CI khác local, cân nhắc dùng Docker để tránh sai khác do font rendering
- [ ] Dùng `mask` cho mọi phần tử động (ads, thời gian thực, nội dung random) — nhớ kiểm tra `maskColor` không bị alpha = 0
- [ ] Không dùng công cụ này để so sánh với Figma khi tính năng còn đang phát triển
- [ ] Khi thay đổi layout hợp lệ, dùng `--update-snapshots` để cập nhật baseline, không sửa test code
