# Lesson 11 — API Testing với Playwright

> Nguồn: PlaywrightVN — Học automation test từ chưa biết gì
> Tổng hợp từ: slide bài học (`lesson-11-pdf/`, 51 ảnh) + code thực hành (`lesson-11/*.spec.ts`)

---

## Mục lục

1. [Nội dung buổi học](#1-nội-dung-buổi-học)
2. [API là gì?](#2-api-là-gì)
3. [Tại sao cần test API?](#3-tại-sao-cần-test-api)
4. [Các thành phần của một API](#4-các-thành-phần-của-một-api)
5. [Định dạng dữ liệu dùng trong API: JSON](#5-định-dạng-dữ-liệu-dùng-trong-api-json)
6. ["Gọi" API là gì? — Các công cụ gọi API](#6-gọi-api-là-gì--các-công-cụ-gọi-api)
7. [API với Postman](#7-api-với-postman)
8. [API với Playwright](#8-api-với-playwright)
9. [Làm việc với các HTTP method (CRUD) — code thực hành](#9-làm-việc-với-các-http-method-crud--code-thực-hành)
10. [Luồng Authentication (đăng nhập lấy token)](#10-luồng-authentication-đăng-nhập-lấy-token)
11. [Bài thực hành](#11-bài-thực-hành)
12. [Phụ lục: các vấn đề hay gặp khi thực hành](#12-phụ-lục-các-vấn-đề-hay-gặp-khi-thực-hành)
13. [Tổng kết Lesson 11](#13-tổng-kết-lesson-11)

---

## 1. Nội dung buổi học

Buổi học xoay quanh 3 chủ đề lớn:

1. **API là gì?**
2. **API với Postman**
3. **API với Playwright**

---

## 2. API là gì?

### Định nghĩa

- **API** = viết tắt của **A**pplication **P**rogramming **I**nterface.
- Là **bộ quy tắc giúp phần mềm giao tiếp với nhau**.

```
[ Client ]  <-----  API  ----->  [ Server ]
```

### Ví dụ thực tế

- **Ứng dụng thời tiết** trên điện thoại dùng API để lấy dữ liệu thời tiết từ server.
- **Website thanh toán** dùng API của ngân hàng để xử lý giao dịch.
- **Ứng dụng đặt xe** dùng API Google Maps để hiển thị bản đồ.

### Luồng REST API (client — server)

```
CLIENT --(HTTP: GET/POST/PUT/DELETE)--> URL (/surveys, /surveys/123, ...) --> SERVER
CLIENT <----------- JSON { survey_id, score, message, response_id } <----------- SERVER
```

> **Ghi nhớ:** API giống như "cầu nối" hoặc "hợp đồng" giúp các hệ thống khác nhau làm việc cùng nhau mà **không cần biết chi tiết bên trong của nhau**.

---

## 3. Tại sao cần test API?

| # | Lý do | Giải thích |
|---|-------|------------|
| 1 | **Đảm bảo hoạt động đúng** | API trả về dữ liệu chính xác, xử lý logic đúng như thiết kế |
| 2 | **Phát hiện lỗi sớm** | Bắt bug trước khi ảnh hưởng đến frontend hoặc người dùng cuối |
| 3 | **Kiểm tra bảo mật** | Đảm bảo API không bị truy cập trái phép, không lộ dữ liệu nhạy cảm |
| 4 | **Kiểm tra hiệu năng** | API phản hồi đủ nhanh, chịu tải được nhiều request cùng lúc |
| 5 | **Tránh phụ thuộc** | Frontend/Mobile có thể test độc lập, không cần đợi giao diện hoàn thành |
| 6 | **Dễ bảo trì** | Khi sửa code, chạy lại test để chắc chắn không làm hỏng tính năng cũ |

Sơ đồ minh họa lý do "tránh phụ thuộc" — vì có lớp API ở giữa nên back-end và front-end có thể phát triển/test song song, độc lập với nhau:

```
[BACK-END] ---> ( API ) ---> [FRONT-END]
```

---

## 4. Các thành phần của một API

### 4.1. Endpoint (URL)

Địa chỉ để truy cập tài nguyên, ví dụ:

```
https://material.playwrightvn.com/api/todo-app/v1/todos.php
```

### 4.2. HTTP Method

Phương thức thao tác với tài nguyên:

| Method | Ý nghĩa |
|--------|---------|
| `GET` | Lấy dữ liệu |
| `POST` | Tạo mới |
| `PUT` | Cập nhật toàn bộ |
| `PATCH` | Cập nhật một phần |
| `DELETE` | Xóa |
| `HEAD` | Giống `GET` nhưng chỉ lấy header, không lấy body |
| `OPTIONS` | Hỏi server xem resource hỗ trợ những method nào |

### 4.3. Request (yêu cầu gửi đi)

Một request thường gồm 3 phần:

- **Headers**: thông tin bổ sung (token xác thực, content-type...)
- **Parameters**: tham số trên URL (query params)
- **Body**: dữ liệu gửi lên (JSON, XML...)

Ví dụ body dạng JSON:

```json
{
  "title": "hello"
}
```

### 4.4. Response (phản hồi trả về)

Một response thường gồm 3 phần:

- **Status Code**: mã trạng thái (`200 OK`, `201 Created`, `404 Not Found`, `500 Error`...)
- **Headers**: thông tin phản hồi
- **Body**: dữ liệu trả về (thường là JSON)

Ví dụ response khi tạo todo thành công (`201 Created`):

```json
{
  "success": true,
  "todo": {
    "id": 13,
    "user_id": 1,
    "title": "hello",
    "description": null
  }
}
```

### 4.5. API Documentation

- Tài liệu mô tả chi tiết: endpoint, HTTP method, URL, body cần gửi...
- Công cụ phổ biến nhất để viết tài liệu API: **Swagger**.
- Ví dụ Swagger UI của bài học: `https://material.playwrightvn.com/api/todo-app/swagger.html`

| Method | Path | Mô tả |
|--------|------|-------|
| GET | `/todos.php` | Lấy toàn bộ danh sách todo |
| GET | `/todo.php` | Lấy 1 todo theo ID |
| POST | `/todo.php` | Tạo mới 1 todo |
| PUT | `/todo.php` | Cập nhật toàn bộ 1 todo |
| PATCH | `/todo.php` | Cập nhật một phần 1 todo |
| DELETE | `/todo.php` | Xóa 1 todo |

---

## 5. Định dạng dữ liệu dùng trong API: JSON

### JSON là gì?

**JSON (JavaScript Object Notation)** là định dạng dữ liệu phổ biến nhất để trao đổi thông tin giữa client và server.

**Đặc điểm:**
- Dễ đọc, dễ viết cho cả người và máy
- Nhẹ, truyền tải nhanh
- Hỗ trợ hầu hết mọi ngôn ngữ lập trình

```json
{
  "hoTen": "Nguyễn Văn A",
  "tuoi": 25,
  "email": "nguyenvana@example.com",
  "daKetHon": false,
  "diaChi": {
    "duong": "123 Lê Lợi",
    "quocGia": "Việt Nam"
  },
  "soThich": ["đọc sách", "âm nhạc"],
  "diemThi": [8.5, 9.0, 7.5]
}
```

### Cấu trúc cơ bản của JSON

Một **Object** là cặp key-value nằm trong dấu `{}`:

```json
{
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
}
```

- **key**: luôn có kiểu **string** (phải nằm trong dấu ngoặc kép `" "`), các key **không được trùng nhau**.
- **value**: có thể là một trong các kiểu dữ liệu sau:

| Kiểu | Ví dụ |
|------|-------|
| String | `"text"` (bắt buộc dùng dấu ngoặc kép) |
| Number | `123`, `3.14` |
| Boolean | `true`, `false` |
| Null | `null` |
| Object | `{ "name": "Phong" }` |
| Array | `["guitar", "code"]` |

**Lưu ý về cú pháp:**
- Key phải để trong dấu ngoặc kép.
- **Không** có dấu phẩy sau phần tử **cuối cùng** trong object.

```json
{
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"   // không có dấu phẩy ở phần tử cuối
}
```

> **Tip:** JSON gần giống với Object trong JavaScript, nhưng key trong JSON **luôn** phải để trong dấu ngoặc kép (JS object thì không bắt buộc).

---

## 6. "Gọi" API là gì? — Các công cụ gọi API

**Gọi API** = gửi **request** tới API kèm các thông tin cần thiết để lấy về **kết quả (response)**.

```
[Client]  <--JSON-->  [GET / POST / PUT / DELETE]  <--HTTP-->  [Server + Database]

Client gửi request  →  qua các HTTP method  →  Server trả về response
```

### Các cách gọi API phổ biến

| Cách gọi | Công cụ |
|----------|---------|
| Command line | cURL |
| Có giao diện (GUI) | Postman |
| Automation / code | Playwright, RestAssured, JMeter |

### Ví dụ gọi bằng cURL

```bash
curl https://material.playwrightvn.com/api/todo-app/v1/todos.php
```

Kết quả trả về là JSON chứa danh sách todo, ví dụ:

```json
{
  "success": true,
  "todos": [
    {
      "id": 1,
      "user_id": 1,
      "title": "Complete project documentation",
      "description": "Write comprehensive docs for the API",
      "status": "pending",
      "priority": "high",
      "due_date": "2025-10-25 17:00:00"
    }
  ]
}
```

### Ví dụ gọi bằng Postman

1. Chọn method **GET**, nhập URL: `https://material.playwrightvn.com/api/todo-app/v1/todos.php`
2. Bấm **Send** để lấy kết quả — nhận về status `200 OK` cùng JSON response.

### Ví dụ gọi bằng Playwright

```ts
import { test } from '@playwright/test';

test('API test cơ bản – response text', async ({ request }) => {
  const URL = 'https://material.playwrightvn.com/api/todo-app/v1/todos.php';

  const response = await request.get(URL);
  const responseText = await response.text();
  console.log(responseText);
});
```

---

## 7. API với Postman

Giao diện Postman chia làm 3 vùng chính:

### 7.1. Sidebar (bên trái)

| Mục | Chức năng |
|-----|-----------|
| **Collections** | Tổ chức các API request thành nhóm |
| **Environments** | Quản lý các biến môi trường (dev, staging, production) |
| **History** | Lịch sử các request đã gửi |
| **Mock Servers** | Tạo server giả lập |
| **Monitors** | Theo dõi API tự động |

### 7.2. Main Workspace (ở giữa)

Nơi làm việc chính để tạo và test API:
- Chọn method + nhập URL + bấm **Send**
- Tab **Params** — query params; **Authorization** — thông tin xác thực; **Headers** — header của request; **Body** — dữ liệu gửi lên
- Khu vực response bên dưới: **Body**, **Cookies**, **Headers**, **Test Results**, kèm status code và thời gian phản hồi

### 7.3. Right Sidebar (bên phải)

- **Documentation** — tài liệu mô tả API
- **Comments** — bình luận trao đổi
- **Code snippets** — sinh code mẫu (curl, JS, Python...) từ request hiện tại
- **Variables in request** — xem các biến đang dùng trong request, chia theo 4 phạm vi:
  - **Environment** (E — xanh lá): ví dụ `base_url = http://localhost...`
  - **Collection** (C — vàng): biến riêng của 1 collection
  - **Globals** (G): biến toàn cục cho cả workspace
  - **Local Vault**: lưu API secret cục bộ, không đồng bộ lên cloud

---

## 8. API với Playwright

### 8.1. `request` fixture

Playwright cung cấp sẵn **fixture `request`** để gọi API **mà không cần** thao tác qua trình duyệt — gọi trực tiếp trong code, nhanh và không phụ thuộc UI.

**Cú pháp cơ bản:**

```ts
test("name", async ({ request }) => {
  // code here
});
```

### 8.2. Gửi GET request

```ts
const url = 'https://material.playwrightvn.com/api/todo-app/v1/todos.php';
const response = await request.get(url);   // → lấy kết quả, gán vào biến response
```

### 8.3. Đọc dữ liệu response: `.text()` vs `.json()`

```ts
const responseText = await response.text();   // lấy response ở dạng string
const responseJSON = await response.json();   // lấy response ở dạng object
```

- **`response.text()`**: trả về response dạng **chuỗi (string)** — hay dùng để `console.log` xem nhanh.
- **`response.json()`**: trả về response đã **parse sẵn thành object** — dùng phổ biến hơn vì tiện truy cập field và kiểm tra dữ liệu (`responseJSON.todos[0].title`...).

### 8.4. Assertion (kiểm tra kết quả)

**Kiểm tra status code:**

```ts
const response = await request.get(URL);
expect(response.status()).toBe(200);
```

**Kiểm tra dữ liệu trong response:**

```ts
const responseJSON = await response.json();
expect(responseJSON.todos.length).toBe(7);
```

> **Lưu ý về `await` với `expect`:** `response.status()` trả về giá trị có sẵn ngay lập tức (không phải Promise) nên `expect(...).toBe(...)` **không cần `await`**. Ngược lại, các *web-first assertion* thao tác trên `Locator` (dùng trong UI test) như `toBeVisible()`, `toBeEnabled()` có cơ chế tự động chờ/poll DOM nên **luôn trả về Promise và bắt buộc phải có `await`**. Xem chi tiết ở [mục 12.4](#124-vì-sao-có-expect-cần-await-có-expect-thì-không).

---

## 9. Làm việc với các HTTP method (CRUD) — code thực hành

Thư mục `lesson-11/` chứa các file thực hành đúng theo trình tự học: GET → POST → full CRUD → có authentication.

### 9.1. GET — Lấy danh sách todo (`01-simple-get.spec.ts`)

```ts
import { expect, test } from '@playwright/test';

test('Get all todo', async ({ request }) => {
    const response = await request.get("https://material.playwrightvn.com/api/todo-app/v1/todos.php");
    const reponseJson = await response.json();
    console.log(reponseJson);
    console.log(reponseJson.todos.length);

    expect(response.status()).toBe(200);
    expect(reponseJson.todos.length).toEqual(149);
})
```

- Gọi `GET /todos.php` → kiểm tra `status = 200` và tổng số todo trả về đúng như kỳ vọng.
- `reponseJson.todos.length` cho biết số lượng phần tử trong mảng `todos`.

### 9.2. POST — Tạo mới todo (`02-simple-post.spec.ts`)

```ts
import { request, test } from '@playwright/test';

test('Create todo', async ({ request }) => {
    const response = await request.post('https://material.playwrightvn.com/api/todo-app/v1/todo.php', {
        data: {
            "title": "Tuyen test add todo 1",
            "description": "Write comprehensive docs for the API",
            "status": "pending",
            "priority": "high",
            "due_date": "2025-10-25T17:00:00",
            "user_id": 1
        }
    });

    const reponseJson = await response.json();
    console.log(reponseJson);
});
```

- `request.post(url, { data: {...} })` — object `data` chính là **body** của request, Playwright tự động serialize thành JSON.
- Server trả về object `todo` vừa được tạo (kèm `id` do server sinh ra).

### 9.3. Full CRUD trong 1 luồng — Create rồi Delete (`03-full-method.spec.ts`)

```ts
import { test } from '@playwright/test';

test.describe('Full method', async () => {
    let id: any;
    test('1. Create todo', async ({ request }) => {
        const response = await request.post('https://material.playwrightvn.com/api/todo-app/v1/todo.php', {
            data: {
                "title": "Tuyen test add todo",
                "description": "Write comprehensive docs for the API",
                "status": "pending",
                "priority": "high",
                "due_date": "2025-10-25T17:00:00",
                "user_id": 1
            }
        });

        const reponseJson = await response.json();
        console.log(reponseJson);
        id = reponseJson.todo.id;
        console.log(id);
    });

    test('2. Delete todo', async ({ request }) => {
        const responseDelete = await request.delete('https://material.playwrightvn.com/api/todo-app/v1/todo.php', {
            data: {
                "id": id
            }
        });

        const responseDeleteJson = await responseDelete.json();
        console.log(responseDeleteJson);
    });
});
```

- Test 1 tạo todo, lấy `id` trả về từ server và **lưu vào biến `id` khai báo ở `describe`** (biến dùng chung giữa các test trong cùng file).
- Test 2 dùng lại biến `id` đó để gọi `DELETE`, xóa đúng todo vừa tạo ở Test 1.
- ⚠️ Cách chia sẻ biến này **chỉ an toàn khi 2 test chạy chung 1 process/worker theo đúng thứ tự** — xem phân tích chi tiết và cách khắc phục bền vững hơn ở [mục 12.2](#122-chia-sẻ-biến-giữa-các-test--vì-sao-chạy-chung-thì-được-chạy-riêng-thì-không) và [12.3](#123-playwright-chạy-test-tuần-tự-hay-song-song).

---

## 10. Luồng Authentication (đăng nhập lấy token)

### 10.1. Lý thuyết

Với các API **cần xác thực**, luồng gọi API gồm **2 bước**:

1. **Đăng nhập** với `username`/`password` → API trả về **token**.
2. **Dùng token đó trong header** để gọi các API tiếp theo (API "phía sau").

**Ví dụ minh họa:**
- Gọi `POST /login` → nhận về `token: "abc123....xyz"`
- Gọi `GET /update-user` kèm header chứa token đó.
- Nếu **thiếu** header token này → request sẽ bị lỗi (thường là `401 Unauthorized`).

### 10.2. Code thực hành (`04-authenticated.spec.ts`)

```ts
import { request, test } from '@playwright/test';

test('Test API', async ({ request }) => {
    const baseURL = 'https://material.playwrightvn.com/api/user-management/v1';

    // Login -> Token
    const loginResponse = await request.post(`${baseURL}/login.php`, {
        data: {
            "email": "admin@example.com",
            "password": "password"
        }
    });

    const loginResponseJson = await loginResponse.json();
    const token = loginResponseJson.data.token;

    // call API with token
    const userResponse = await request.get(`${baseURL}/users.php`, {
        headers: {
            authorization: `Bearer ${token}`
        }
    });

    const userResponseJson = await userResponse.json();
    console.log(userResponseJson);
});
```

**Giải thích từng bước:**

1. `request.post('${baseURL}/login.php', { data: { email, password } })` — gửi thông tin đăng nhập.
2. `loginResponseJson.data.token` — lấy token từ response của bước login.
3. `request.get('${baseURL}/users.php', { headers: { authorization: 'Bearer ${token}' } })` — đính kèm token vào **header `authorization`** theo chuẩn `Bearer <token>` để gọi API đã được bảo vệ.

> Đây chính là mẫu (pattern) chuẩn để test bất kỳ API nào yêu cầu đăng nhập: **login 1 lần, lấy token, dùng lại token đó cho các request cần xác thực tiếp theo.**

---

## 11. Bài thực hành

Swagger: `https://material.playwrightvn.com/api/user-management/swagger.html`

1. Gọi API **Get users** khi **chưa đăng nhập** → verify nhận lỗi `401`.
2. Gọi API **login** với tài khoản admin → verify login **thành công** (nhận được token).
3. Gọi API **Get users** kèm token ở bước 2 → verify lấy danh sách user **thành công**.

---

## 12. Phụ lục: các vấn đề hay gặp khi thực hành

*(Tóm tắt từ các buổi hỏi-đáp thực tế khi làm bài API testing; chi tiết đầy đủ xem file `lesson11-note.md`)*

### 12.1. Vì sao data trả về từ Playwright "trông" khác với Postman

- Gọi cùng 1 API nhưng `console.log()` trong Playwright/Node có thể hiện thị mảng bị **cắt ngắn** (`... 41 more items`), trong khi Postman hiển thị đầy đủ.
- **Nguyên nhân**: `console.log()` của Node dùng `util.inspect` bên dưới, **mặc định giới hạn số phần tử hiển thị của mảng và độ sâu object** — đây chỉ là giới hạn hiển thị ở terminal, **không phải API trả thiếu dữ liệu**.
- **Cách xem đầy đủ:**
  ```ts
  console.log(JSON.stringify(reponseJson, null, 2));
  // hoặc
  console.dir(reponseJson, { depth: null, maxArrayLength: null });
  ```
- Ngoài ra, API luyện tập (`material.playwrightvn.com`) là **API dùng chung cho nhiều học viên**, nên số lượng bản ghi thay đổi liên tục — data khác nhau giữa 2 lần gọi là **bình thường**, không phải lỗi.

### 12.2. Chia sẻ biến giữa các test — vì sao chạy chung thì được, chạy riêng thì không

- Biến khai báo ở `describe` (như `let id` trong [mục 9.3](#93-full-crud-trong-1-luồng--create-rồi-delete-03-full-methodspects)) chỉ tồn tại trong **bộ nhớ của 1 lần chạy file (1 process)**.
- **Chạy chung `describe`**: file load 1 lần, Test 1 → Test 2 chạy nối tiếp trong cùng process → biến `id` vẫn còn → xóa thành công.
- **Chạy riêng từng test**: mỗi lần bấm Run là 1 process mới → biến bị khởi tạo lại (`undefined`) → gửi `id: undefined` lên server → xóa thất bại.
- **Cách khắc phục bền vững** (không phụ thuộc cách chạy):
  1. Gộp create + delete vào **cùng 1 test**.
  2. Dùng `test.beforeAll`/`test.beforeEach` để luôn tạo dữ liệu mới trước khi test chạy.
  3. `test.describe.serial(...)` chỉ đảm bảo thứ tự khi chạy **cả nhóm**, không giải quyết được khi chạy lẻ từng test.

### 12.3. Playwright chạy test tuần tự hay song song?

| Cấu hình | Test trong cùng 1 file/describe | Các file khác nhau |
|----------|----------------------------------|---------------------|
| `fullyParallel: false` (mặc định) | Tuần tự, chung 1 worker | Song song (đa worker) |
| `fullyParallel: true` | **Có thể** song song, đa worker | Song song (đa worker) |
| `test.describe.serial(...)` | Luôn tuần tự, chung 1 worker | Không ảnh hưởng |
| Nhiều `projects` trong config | Mỗi project chạy độc lập, **song song** với nhau | — |

- Với `fullyParallel: true`, **không có gì đảm bảo** 2 test trong cùng describe chạy chung 1 worker hay đúng thứ tự — việc chia sẻ biến giữa các test (mục 12.2) có thể "may mắn" chạy đúng nhưng **không phải hành vi được đảm bảo**.
- Ép chạy tuần tự thật sự: `test.describe.serial('...', async () => { ... })`.

### 12.4. Vì sao có `expect` cần `await`, có `expect` thì không

| Loại assertion | Ví dụ | Cần `await`? |
|-----------------|-------|---------------|
| Web-first assertion trên `Locator` (UI, có auto-retry/poll DOM) | `toBeVisible`, `toBeEnabled`, `toHaveText`, `toHaveURL` | **Có** (trả về Promise) |
| Assertion đồng bộ trên giá trị có sẵn | `toBe`, `toEqual`, `toContain`, `toHaveLength` | **Không** (không trả về Promise) |

> **Mẹo:** `response.status()` là giá trị đọc ngay lập tức từ response → `expect(response.status()).toBe(200)` không cần `await`. Nhưng quên `await` ở 1 web-first assertion trong UI test có thể khiến test **pass giả** (false green) — luôn cẩn trọng với loại assertion đang dùng.

---

## 13. Tổng kết Lesson 11

| Chủ đề | Điểm chính |
|--------|-----------|
| **API là gì** | Bộ quy tắc giúp Client và Server giao tiếp với nhau, không cần biết chi tiết bên trong của nhau |
| **Vì sao cần test API** | Đúng chức năng, phát hiện lỗi sớm, bảo mật, hiệu năng, tránh phụ thuộc FE/BE, dễ bảo trì |
| **Thành phần của API** | Endpoint, HTTP Method, Request (Headers/Params/Body), Response (Status/Headers/Body), Documentation (Swagger) |
| **JSON** | Object là cặp key-value trong `{}`; key luôn là string; value có thể là string/number/boolean/null/object/array; không dấu phẩy ở phần tử cuối |
| **Công cụ gọi API** | cURL (command line), Postman (GUI), Playwright/RestAssured/JMeter (automation) |
| **Postman** | 3 vùng: Sidebar trái (Collections/Environments/History), Main Workspace (gửi request/xem response), Right Sidebar (Documentation/Comments/Variables) |
| **Playwright API testing** | Dùng fixture `request` để gọi API trực tiếp, không cần trình duyệt: `request.get/post/put/delete(url, { data, headers })`; đọc kết quả bằng `.text()` hoặc `.json()` |
| **CRUD thực hành** | GET danh sách, POST tạo mới, kết hợp Create → Delete trong `describe`, dùng biến chung để truyền `id` giữa các test |
| **Authentication** | Login lấy `token` → gắn `Authorization: Bearer <token>` vào header các API cần xác thực tiếp theo |
| **Cạm bẫy thường gặp** | `console.log` bị Node giới hạn hiển thị (dùng `JSON.stringify` để xem đầy đủ); biến chia sẻ giữa test chỉ sống trong 1 process; `fullyParallel` ảnh hưởng tới thứ tự chạy; web-first assertion cần `await`, assertion đồng bộ thì không |
