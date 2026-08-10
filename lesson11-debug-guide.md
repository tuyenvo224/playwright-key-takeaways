# Cách debug lỗi trong Playwright test bằng VSCode

Ghi lại quy trình debug thực tế đã làm với file [03-full-method.spec.ts](./03-full-method.spec.ts), khi test tạo todo qua API bị lỗi.

## Bối cảnh lỗi

Test gửi `POST` tới `https://material.playwrightvn.com/api/todo-app/v1/todos.php` để tạo todo, sau đó lấy `id` từ `reponseJson.todo.id`. Nghi ngờ response không đúng cấu trúc mong đợi nên cần debug để xem giá trị thực tế trả về là gì.

## Các bước debug

### 0. Nhận biết test đang bị lỗi

Trước khi debug, VSCode đã cho biết vị trí lỗi ngay trong editor:

- Icon **X đỏ** xuất hiện ở đầu các dòng `test.describe(...)` và `test(...)` chứa lỗi (thay vì icon tam giác chạy màu xanh bình thường).
- Dòng gây lỗi được gạch chân đỏ, hover vào sẽ thấy message lỗi ngay tại chỗ, ví dụ: `TypeError: Cannot read properties of undefined (reading 'id')` tại dòng `id = reponseJson.todo.id`.
- Đây chính là gợi ý để biết **nên đặt breakpoint ở đâu**: đặt ngay trước dòng bị báo lỗi để xem giá trị biến trước khi nó gây crash.

### 1. Đặt breakpoint tại dòng nghi ngờ

Chuột phải vào số dòng trong VSCode → menu hiện ra:

- **Add Breakpoint**: dừng chương trình tại dòng đó mỗi lần chạy qua.
- **Add Conditional Breakpoint...**: chỉ dừng khi thỏa điều kiện (VD: `id === undefined`).
- **Add Logpoint...**: in log ra Debug Console mà không cần sửa code, không cần dừng chương trình.
- **Add Triggered Breakpoint...**: breakpoint chỉ kích hoạt khi một breakpoint khác đã chạy trước.

Đặt breakpoint ngay **sau** dòng gọi API/parse JSON, trước dòng dùng dữ liệu đó (ví dụ tại dòng `console.log(reponseJson)`, trước dòng `id = reponseJson.todo.id`).

### 2. Chạy test ở chế độ Debug

Bấm **Debug Test** (icon con bọ ở đầu dòng `test(...)`) thay vì Run Test bình thường. Playwright sẽ chạy test nhưng dừng lại ngay khi gặp breakpoint, thay vì chạy hết rồi chỉ báo lỗi cuối cùng.

### 3. Quan sát tiến trình chạy

Trong lúc debug, VSCode hiển thị inline decoration cho từng dòng đã chạy qua (thời gian thực thi, icon vòng tròn xanh), dòng hiện tại đang tạm dừng được tô vàng. Đồng thời một thanh **Debug Toolbar** nổi lên phía trên editor với các nút:

- **Continue (F5)**: chạy tiếp cho đến breakpoint kế tiếp (hoặc hết chương trình).
- **Step Over (F10)**: chạy qua dòng hiện tại, không đi vào bên trong hàm được gọi.
- **Step Into (F11)**: đi vào bên trong hàm đang được gọi ở dòng hiện tại.
- **Step Out (Shift+F11)**: chạy hết hàm hiện tại và quay ra ngoài hàm gọi nó.
- **Restart (Ctrl+Shift+F5)**: chạy lại debug session từ đầu.
- **Stop (Shift+F5)**: dừng hẳn debug session và kết thúc tiến trình test.
- Giữ phím **Alt** khi bấm nút Stop sẽ đổi thành **Disconnect (Shift+F5)**: chỉ ngắt kết nối debugger, để tiến trình test vẫn tiếp tục chạy ngầm (hữu ích khi debug tiến trình được chạy độc lập, không muốn kill nó).

### 4. Kiểm tra giá trị biến bằng cách hover chuột

Khi đang dừng tại breakpoint, di chuột vào biến (ví dụ `reponseJson`) → VSCode hiện tooltip ngay tại chỗ, ví dụ:

```json
{ "success": false, "message": "Method not allowed" }
```

→ Đây là cách nhanh nhất để xem giá trị thực tế của một biến.

### 5. Kiểm tra chi tiết hơn bằng Debug Sidebar (Variables panel)

Mở panel **RUN AND DEBUG** ở sidebar trái → mục **VARIABLES > Block** → thấy toàn bộ object, bao gồm cả các field ẩn không hiện đầy đủ khi hover (ví dụ `response = APIResponse: 405 Method...`). Cách này hữu ích khi object phức tạp/nested.

### 6. Kết thúc debug session

Sau khi đã lấy được thông tin cần thiết, bấm nút **Stop** (hình vuông đỏ trên Debug Toolbar, phím tắt `Shift+F5`) để dừng hẳn tiến trình debug và test. Nếu chỉ muốn ngắt kết nối debugger mà giữ tiến trình chạy ngầm, giữ `Alt` rồi bấm nút đó để chuyển thành **Disconnect**.

## Kết quả tìm ra nguyên nhân

Status code thực tế là **405 Method Not Allowed** — server từ chối method POST tại endpoint đó (có thể sai URL, sai method, thiếu header, hoặc endpoint yêu cầu method khác). Vì response không có `todo.id`, dòng `id = reponseJson.todo.id` sẽ ném lỗi `Cannot read properties of undefined`.

## Quy trình debug tổng quát (áp dụng lại cho lần sau)

1. Đọc dấu hiệu lỗi có sẵn trong editor trước (icon X đỏ ở `describe`/`test`, gạch chân đỏ, hover xem message lỗi) để xác định đúng dòng cần đặt breakpoint.
2. Đặt breakpoint ngay sau dòng gọi API/parse JSON, trước dòng dùng dữ liệu đó.
3. Chạy bằng **Debug Test**, không phải Run Test — để chương trình dừng lại thay vì chỉ báo lỗi cuối cùng.
4. Dùng các nút trên Debug Toolbar (Continue, Step Over, Step Into, Step Out) để điều khiển tốc độ chạy qua từng dòng/hàm khi cần xem sâu hơn.
5. Hover vào biến để xem nhanh, hoặc mở **Variables panel** để xem đầy đủ (đặc biệt với object lồng nhau).
6. Đọc kỹ `response`/`status` trước khi đọc `data` — nếu status lỗi (4xx/5xx) thì đừng debug tiếp phần parse data, hãy sửa request trước (kiểm tra URL, method, headers, body).
7. Dùng **Conditional Breakpoint** nếu lỗi chỉ xảy ra trong một số trường hợp cụ thể, hoặc **Logpoint** nếu chỉ cần xem giá trị mà không muốn dừng chương trình nhiều lần.
8. Debug xong thì bấm **Stop** để kết thúc session (hoặc **Disconnect** nếu muốn giữ tiến trình chạy ngầm).
