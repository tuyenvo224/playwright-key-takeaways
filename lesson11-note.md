# Lesson 11 - API Testing với Playwright

---

## Mục lục

1. [Vì sao data trả về từ Playwright "trông" khác với Postman](#1-vì-sao-data-trả-về-từ-playwright-trông-khác-với-postman)
2. [Chia sẻ state (biến) giữa các test — vì sao lúc chạy chung thì được, chạy riêng thì không](#2-chia-sẻ-state-biến-giữa-các-test--vì-sao-lúc-chạy-chung-thì-được-chạy-riêng-thì-không)
3. [Playwright chạy test tuần tự hay song song?](#3-playwright-chạy-test-tuần-tự-hay-song-song)
4. [Vì sao có `expect` cần `await`, có `expect` thì không](#4-vì-sao-có-expect-cần-await-có-expect-thì-không)
5. [Tổng kết Lesson 11](#5-tổng-kết-lesson-11)

---

## 1. Vì sao data trả về từ Playwright "trông" khác với Postman

### Hiện tượng

Gọi cùng 1 API (`GET https://material.playwrightvn.com/api/todo-app/v1/todos.php`):

- Trên **Postman**: tab Body hiển thị **toàn bộ** mảng `todos` không thiếu phần tử nào.
- Trên **Playwright** (`console.log(reponseJson)` xem trong VSCode Test Results): mảng bị cắt ngắn, cuối danh sách hiện dòng `... 41 more items`.

### Nguyên nhân thật sự — KHÔNG PHẢI Playwright trả thiếu data

API vẫn trả về **đầy đủ** JSON giống hệt Postman. Vấn đề nằm ở cách **`console.log()` của Node.js hiển thị object/array**:

- Node dùng `util.inspect` bên dưới `console.log`, mặc định **giới hạn số phần tử hiển thị của mảng** (chỉ in một số phần tử đầu, phần còn lại gộp thành `... N more items`) và **giới hạn độ sâu (depth)** khi in object lồng nhau.
- Đây thuần túy là giới hạn hiển thị ở terminal, **không liên quan gì đến dữ liệu HTTP response thực tế**.

Trong khi đó, Postman render thẳng **raw JSON response body** ra UI (tab Body), không đi qua `console.log`/`util.inspect`, nên không bị cắt.

### Cách xem đầy đủ data trong Playwright

```ts
// Cách 1: ép ra chuỗi JSON đầy đủ, không bị Node cắt bớt
console.log(JSON.stringify(reponseJson, null, 2));

// Cách 2: chỉ log số lượng để kiểm tra nhanh
console.log('total todos:', reponseJson.todos.length);

// Cách 3: console.dir với option depth/maxArrayLength = null (không giới hạn)
console.dir(reponseJson, { depth: null, maxArrayLength: null });
```

### Một nguyên nhân phụ: đây là API dùng chung

`material.playwrightvn.com` là API luyện tập **dùng chung cho nhiều học viên**. Vì vậy:

- Số lượng `todos` và các `id` cụ thể sẽ **khác nhau giữa các lần gọi**, vì học viên khác liên tục tạo/sửa/xóa todo trên cùng 1 database.
- Việc data ở Postman (chụp tại thời điểm A) khác với data ở Playwright (chạy tại thời điểm B) là **bình thường**, không phải lỗi code.

> **Ghi nhớ:** Nếu thấy dữ liệu trả về "có vẻ" thiếu khi debug bằng `console.log`, luôn nghi ngờ cách hiển thị của console trước khi nghi ngờ API.

---

## 2. Chia sẻ state (biến) giữa các test — vì sao lúc chạy chung thì được, chạy riêng thì không

### Code ví dụ (từ `03-full-method.spec.ts`)

```ts
import { test } from '@playwright/test';

test.describe('Full method', async () => {
  let id: number;

  test('1. Create todo', async ({ request }) => {
    const response = await request.post('https://material.playwrightvn.com/api/todo-app/v1/todo.php', {
      data: {
        title: 'Tuyen test add todo',
        description: 'Write comprehensive docs for the API',
        status: 'pending',
        priority: 'high',
        due_date: '2025-10-25T17:00:00',
        user_id: 1
      }
    });
    const reponseJson = await response.json();
    id = reponseJson.todo.id;   // gán vào biến `id` khai báo ở describe
    console.log(id);
  });

  test('2. Delete todo', async ({ request }) => {
    await request.delete('https://material.playwrightvn.com/api/todo-app/v1/todo.php', {
      data: { id }   // đọc lại biến `id` từ test 1
    });
  });
});
```

### Hiện tượng

| Cách chạy | Kết quả |
|-----------|---------|
| Bấm Run trên cả `describe` (Test 1 → Test 2 liên tiếp) | Xóa **thành công** |
| Bấm Run riêng Test 1, rồi bấm Run riêng Test 2 | Xóa **thất bại**, không xóa được gì |

### Giải thích

Biến `let id: number;` chỉ tồn tại **trong bộ nhớ của 1 lần thực thi file** (1 process/worker).

**Khi chạy chung `describe`:**
- Playwright load file **một lần**, chạy Test 1 rồi Test 2 nối tiếp nhau (thường trong cùng 1 worker khi không có gì ép buộc chạy song song — xem thêm mục 3).
- Test 1 gán `id = reponseJson.todo.id` → biến này **vẫn còn nguyên trong bộ nhớ** khi Test 2 chạy kế tiếp → Test 2 gửi đúng `id` vừa tạo → xóa thành công.

**Khi chạy riêng lẻ từng test:**
- Mỗi lần bấm "Run" trên 1 test là Playwright **load lại file từ đầu trong 1 process mới hoàn toàn**.
- Lần chạy Test 1: `let id` khởi tạo lại, được gán giá trị, tạo todo thành công — nhưng khi process kết thúc, **biến `id` bị hủy hoàn toàn cùng với process đó**.
- Lần chạy Test 2 (sau đó, ở 1 lần bấm Run khác): file lại được load lại từ đầu (process mới) → `let id: number;` quay về trạng thái `undefined` → gửi `data: { id: undefined }` lên server → server không tìm thấy todo nào khớp → **không xóa được gì**.

### Cách khắc phục (đáng tin cậy, không phụ thuộc cách chạy)

1. **Gộp create + delete vào cùng 1 test** nếu mục đích chỉ là dọn dữ liệu test:
   ```ts
   test('Create then delete todo', async ({ request }) => {
     const createRes = await request.post(/* ... */);
     const { todo } = await createRes.json();
     await request.delete(/* ... */, { data: { id: todo.id } });
   });
   ```
2. **Dùng `test.beforeAll` / `test.beforeEach`** để luôn tạo todo mới và lưu `id` trước khi test khác chạy, đảm bảo có giá trị dù chạy cả nhóm hay chạy lẻ.
3. Dùng `test.describe.serial(...)` để ép thứ tự chạy tuần tự trong cùng 1 worker — nhưng **chỉ có tác dụng khi chạy cả nhóm**, không giải quyết được vấn đề khi bấm Run riêng lẻ từng test (vì mỗi lần Run riêng vẫn là 1 process độc lập).

> Cách 1 và 2 là bền vững nhất vì không phụ thuộc vào việc bạn chạy toàn bộ `describe` hay chạy riêng từng test.

---

## 3. Playwright chạy test tuần tự hay song song?

### Không phải lúc nào cũng chạy tuần tự — phụ thuộc vào config

Xem `playwright.config.ts`:

```ts
export default defineConfig({
  testDir: './tests',
  fullyParallel: true,   // <-- điểm mấu chốt
  workers: process.env.CI ? 1 : undefined,
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'Mobile Safari', use: { ...devices['iPad (gen 5)'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
});
```

### Mặc định của Playwright (`fullyParallel: false`)

- Các **file test khác nhau** chạy song song (mỗi file được gán 1 worker).
- Các **test trong cùng 1 file/describe** chạy **tuần tự**, trong **cùng 1 worker**, đúng theo thứ tự khai báo.

### Khi bật `fullyParallel: true` (repo này đang cấu hình như vậy)

- Playwright được phép tách **từng test** ra chạy độc lập, mỗi test có thể được gán cho **worker khác nhau**, kể cả khi chúng nằm trong cùng 1 `describe`/cùng 1 file.
- Nghĩa là: **không có gì đảm bảo** Test 1 và Test 2 chạy chung 1 process, cũng không đảm bảo đúng thứ tự — chúng có thể chạy song song hoặc bị đảo thứ tự tùy vào worker nào rảnh trước.
- Việc chạy chung `describe` ở mục 2 "thành công" nhiều khả năng chỉ là **may mắn** do số lượng test ít, scheduler dồn chung 1 worker — **không phải hành vi được đảm bảo**. Nếu số lượng test/worker thay đổi, việc chia sẻ biến `id` hoàn toàn có thể fail dù chạy chung.

### Multi-project — thêm 1 lớp song song nữa

Repo cấu hình **4 projects**: `chromium`, `Mobile Safari`, `firefox`, `webkit`.

- Khi bấm Run trên `describe`, Playwright thực chất chạy **cả describe đó tối đa 4 lần song song** (1 lần cho mỗi project, trên các worker khác nhau).
- Điều này càng làm việc chia sẻ biến giữa các test trở nên không an toàn — vì mỗi project là 1 lần thực thi độc lập, biến `id` của project `chromium` không liên quan gì tới project `firefox`.

### Cách ép chạy tuần tự thật sự (bất kể `fullyParallel`)

```ts
test.describe.serial('Full method', async () => {
  // các test trong đây luôn chạy tuần tự, cùng 1 worker,
  // theo đúng thứ tự khai báo — nếu 1 test fail, các test sau bị skip
});
```

Ngược lại, muốn ép chạy song song rõ ràng (dù `fullyParallel: false`):

```ts
test.describe.configure({ mode: 'parallel' });
```

### Bảng tóm tắt

| Cấu hình | Test trong cùng 1 file/describe | Các file khác nhau |
|----------|----------------------------------|---------------------|
| `fullyParallel: false` (mặc định) | Tuần tự, chung 1 worker | Song song (đa worker) |
| `fullyParallel: true` (repo này) | **Có thể** song song, đa worker | Song song (đa worker) |
| `test.describe.serial(...)` | Luôn tuần tự, chung 1 worker (ghi đè `fullyParallel`) | Không ảnh hưởng |
| Nhiều `projects` trong config | Mỗi project chạy độc lập, **song song** với nhau | — |

---

## 4. Vì sao có `expect` cần `await`, có `expect` thì không

### So sánh 2 đoạn code

```ts
// (A) UI test — có await
await expect(page.locator("//h1[@class='wp-heading-inline']")).toBeVisible();
await expect(page.locator("//a[@class='page-title-action']")).toBeEnabled();
```

```ts
// (B) API test — không có await
expect(response.status()).toBe(200);
```

### Giải thích: 2 loại assertion khác nhau

**(A) Web-first assertions (assertion bất đồng bộ, có auto-retry):**

- Các hàm như `.toBeVisible()`, `.toBeEnabled()`, `.toHaveText()`, `.toHaveURL()`, `.toHaveCount()`...
- Nhận vào một **`Locator`** — tại thời điểm gọi, Playwright **chưa chắc** phần tử đã tồn tại/hiển thị hay chưa.
- Bên trong, Playwright sẽ **tự động poll lại DOM nhiều lần** (mặc định tối đa 5 giây) cho đến khi điều kiện đúng hoặc hết timeout.
- Vì có cơ chế chờ/lặp lại này, hàm **trả về một `Promise`** → bắt buộc phải có `await`. Nếu quên, test có thể kết thúc trước khi assertion kịp xác nhận xong → bỏ sót lỗi (false pass) hoặc unhandled promise.

**(B) Assertion đồng bộ (sync assertion, kiểu Jest cổ điển):**

- Các hàm như `.toBe()`, `.toEqual()`, `.toContain()`, `.toHaveLength()`...
- `response.status()` không phải Locator — nó chỉ đọc ra 1 giá trị (`200`) đã có sẵn ngay lập tức từ response, không cần chờ/poll gì cả.
- `.toBe(200)` so sánh giá trị **ngay lập tức**: đúng thì pass, sai thì `throw` lỗi ngay — **không trả về `Promise`**.
- Vì không có `Promise` để chờ, thêm `await` phía trước cũng không có tác dụng gì (một số linter còn cảnh báo "unnecessary await" nếu thêm vào).

### Quy tắc chung để nhớ

| Loại assertion | Ví dụ | Cần `await`? |
|-----------------|-------|---------------|
| Thao tác trên `Locator`, có thể đổi theo thời gian (UI) | `toBeVisible`, `toBeEnabled`, `toHaveText`, `toHaveURL`, `toHaveCount` | **Có** |
| So sánh giá trị tĩnh đã có sẵn (number/string/object) | `toBe`, `toEqual`, `toContain`, `toHaveLength` | **Không** |

> **Mẹo kiểm tra nhanh:** Nếu bạn không chắc 1 matcher có cần `await` hay không, cứ thêm `await` vào — nếu hàm đó không trả `Promise`, `await` một giá trị thường không gây lỗi hay thay đổi hành vi. Nhưng **ngược lại thì nguy hiểm**: quên `await` ở một web-first assertion có thể khiến test pass giả (false green). ESLint plugin `eslint-plugin-playwright` (nếu có cấu hình trong repo) sẽ cảnh báo "missing await on assertion" để giúp phát hiện lỗi này.

---

## 5. Tổng kết Lesson 11

| Chủ đề | Điểm chính |
|--------|-----------|
| **Console.log vs Postman** | `console.log()` của Node bị giới hạn hiển thị mảng/object (depth, số phần tử) → dùng `JSON.stringify(obj, null, 2)` để xem đầy đủ. API dùng chung nên data thay đổi liên tục theo thời gian, không phải lỗi test. |
| **Chia sẻ biến giữa các test** | Biến khai báo ở `describe` chỉ tồn tại trong 1 lần chạy file (1 process). Chạy chung `describe` thì biến còn nguyên; chạy riêng từng test thì file load lại, biến mất. Giải pháp bền vững: gộp logic vào 1 test, hoặc dùng `beforeAll`/`beforeEach`. |
| **Tuần tự vs song song** | Mặc định Playwright chạy tuần tự các test trong cùng file. Bật `fullyParallel: true` (như repo này) cho phép từng test chạy song song, đa worker — phá vỡ giả định chia sẻ state. Nhiều `projects` trong config còn nhân thêm số lần chạy song song. Muốn ép tuần tự thật sự: dùng `test.describe.serial(...)`. |
| **`await` với `expect`** | Web-first assertions (`toBeVisible`, `toBeEnabled`...) thao tác trên Locator, có cơ chế tự động chờ/retry → trả về `Promise` → **bắt buộc `await`**. Assertion đồng bộ (`toBe`, `toEqual`...) so sánh giá trị có sẵn ngay lập tức, không trả `Promise` → **không cần `await`**. |
