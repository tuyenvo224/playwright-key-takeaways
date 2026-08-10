# Phân biệt Function Expression, Anonymous Function & Callback

---

## Phần 1: Function Expression vs Anonymous Function

### Câu hỏi thường gặp: "Hai cái này có phải là một không?"

**Không phải một, nhưng có liên quan chặt chẽ.**

Để hiểu rõ, cần phân biệt 2 khái niệm độc lập:

| Khái niệm | Nói về cái gì |
|---|---|
| **Function Expression** | **Cách khai báo hàm** (gán hàm vào biến) |
| **Anonymous Function** | **Tên của hàm** (hàm không có tên) |

---

### 1.1 Function Expression là gì?

> **Function Expression = gán hàm vào một biến**

```js
// Function Declaration (truyền thống) — không phải FE
function add(a, b) {
    return a + b;
}

// Function Expression — gán hàm vào biến
const add = function(a, b) {
    return a + b;
};
```

Điểm mấu chốt: `const add = function...` — hàm được **gán vào biến `add`**.

---

### 1.2 Anonymous Function là gì?

> **Anonymous Function = hàm không có tên**

```js
// Hàm CÓ TÊN
function myAdd(a, b) { return a + b; }
//       ^^^^^ tên hàm

// Hàm KHÔNG TÊN (anonymous)
function(a, b) { return a + b; }
// không có tên ở đây
```

---

### 1.3 Điểm nhầm lẫn thường gặp: Tên biến ≠ Tên hàm

```js
const add = function(a, b) {
//    ^^^            ← đây là TÊN BIẾN
//              ↑
//         hàm này KHÔNG CÓ TÊN → là anonymous function
    return a + b;
};
```

**`add` là tên biến** — chỉ là "cái hộp" chứa hàm.
**Bản thân hàm không có tên** → đây là **anonymous function**.

---

### 1.4 Function Expression có 2 dạng

```js
// Dạng 1: Anonymous Function Expression (phổ biến nhất)
const add = function(a, b) {
//               ↑ không có tên → ANONYMOUS
    return a + b;
};

// Dạng 2: Named Function Expression (hiếm dùng)
const add = function myAdd(a, b) {
//                   ^^^^^ có tên → NAMED
    return a + b;
};
```

> **Trong thực tế, khi viết Function Expression, người ta hầu như luôn bỏ tên hàm.**
> Nên khi thấy `const x = function() {}` → vừa là FE, vừa là anonymous.

---

### 1.5 Sơ đồ tổng hợp

```
                    CÓ TÊN HÀM        KHÔNG TÊN HÀM
                   ┌─────────────┐   ┌─────────────────┐
 GÁN CHO BIẾN      │ Named FE    │   │ Anonymous FE    │  ← vừa là FE
(Function          │ const x =   │   │ const x =       │     vừa là anonymous
 Expression)       │ function    │   │ function() {}   │
                   │ myFunc() {} │   └─────────────────┘
                   └─────────────┘
                   ┌─────────────┐   ┌─────────────────┐
 DÙNG TRỰC TIẾP    │ Function    │   │ Anonymous làm   │  ← anonymous nhưng
(callback, v.v.)   │ Declaration │   │ callback        │     KHÔNG phải FE
                   │ function    │   │ setTimeout(     │
                   │ greet() {}  │   │  function(){},  │
                   └─────────────┘   │  1000)          │
                                     └─────────────────┘
```

---

### 1.6 Tóm tắt phân biệt

| | Function Expression | Anonymous Function |
|---|---|---|
| **Định nghĩa** | Hàm được gán vào biến | Hàm không có tên |
| **Nhận ra bằng cách** | Có dấu `=` trước từ khóa `function` | Không có tên sau từ khóa `function` |
| **Có thể kết hợp?** | FE có thể có tên hoặc không | Anonymous có thể là FE hoặc callback |
| **Ví dụ** | `const fn = function() {}` | `function() {}` (phần sau dấu `=`) |

---

---

## Phần 2: Callback Function

### 2.1 Callback là gì?

> **Callback = một hàm được truyền vào hàm khác như tham số, để được gọi lại sau.**

Từ "callback" nghĩa đen là **"gọi lại"** — bạn đưa hàm cho người khác giữ, khi nào cần thì họ gọi lại hàm đó.

---

### 2.2 Ví dụ đời thực trước khi xem code

**Tình huống:** Bạn đặt đồ ăn online.

1. Bạn đặt đồ ăn → để lại **số điện thoại** (callback)
2. Bạn đi làm việc khác (không cần ngồi chờ)
3. Khi đồ ăn đến → nhân viên **gọi lại** cho bạn theo số đã để lại

Số điện thoại của bạn chính là **callback** — được dùng để "gọi lại" khi có sự kiện xảy ra.

---

### 2.3 Ví dụ code cơ bản

```js
// setTimeout nhận vào 2 tham số:
// - Tham số 1: một HÀM (callback) → sẽ được gọi SAU
// - Tham số 2: thời gian chờ (ms)

setTimeout(function() {
    console.log("Được gọi sau 2 giây!");
}, 2000);

console.log("Tôi chạy TRƯỚC!");
```

**Kết quả:**
```
Tôi chạy TRƯỚC!
(chờ 2 giây...)
Được gọi sau 2 giây!
```

`function() { console.log(...) }` là callback — nó được **truyền vào** `setTimeout` và **gọi lại** sau 2 giây.

---

### 2.4 Tự viết hàm có callback để hiểu rõ cơ chế

```js
// Bước 1: Viết hàm nhận callback làm tham số
function chaoKhach(tenKhach, callback) {
    console.log("Xin chào " + tenKhach + "!");
    callback(); // ← GỌI LẠI hàm được truyền vào
}

// Bước 2: Truyền hàm vào khi gọi
chaoKhach("Tuyền", function() {
    console.log("Cảm ơn bạn đã đến!");
});
```

**Kết quả:**
```
Xin chào Tuyền!
Cảm ơn bạn đã đến!
```

**Giải thích từng bước:**
1. Gọi `chaoKhach("Tuyền", function(){...})`
2. Hàm chạy: in ra `"Xin chào Tuyền!"`
3. Đến dòng `callback()` → gọi lại hàm `function() { console.log("Cảm ơn...") }`
4. In ra `"Cảm ơn bạn đã đến!"`

---

### 2.5 Callback có thể nhận tham số

```js
function tinhToan(a, b, callback) {
    const ketQua = a + b;
    callback(ketQua); // truyền kết quả vào callback
}

tinhToan(5, 3, function(result) {
    console.log("Kết quả là: " + result); // Kết quả là: 8
});
```

---

### 2.6 Callback dùng để làm gì?

**1. Xử lý bất đồng bộ (async)** — làm gì đó SAU KHI một việc hoàn thành:

```js
// Chờ 3 giây rồi mới in
setTimeout(function() {
    console.log("3 giây trôi qua!");
}, 3000);
```

**2. Xử lý sự kiện** — làm gì đó KHI người dùng click:

```js
button.addEventListener('click', function() {
    console.log("Bạn vừa click!");
});
```

**3. Xử lý từng phần tử trong mảng:**

```js
const danhSach = ["An", "Bình", "Cường"];

danhSach.forEach(function(ten) {
    console.log("Xin chào " + ten);
});
// Xin chào An
// Xin chào Bình
// Xin chào Cường
```

---

### 2.7 Tại sao callback hay đi kèm với Anonymous Function?

Vì callback thường **chỉ dùng 1 lần tại chỗ đó** → không cần đặt tên:

```js
// Cách 1: Đặt tên hàm (thừa nếu chỉ dùng 1 lần)
function inKetQua() {
    console.log("Xong!");
}
setTimeout(inKetQua, 1000);

// Cách 2: Anonymous — gọn hơn, không cần đặt tên
setTimeout(function() {
    console.log("Xong!");
}, 1000);

// Cách 3: Arrow function — gọn nhất
setTimeout(() => {
    console.log("Xong!");
}, 1000);
```

> Slide nói: *"Anonymous function dùng khi function chỉ cần dùng một lần hoặc làm callback"* — chính là lý do này.

---

### 2.8 Tóm tắt về Callback

| Câu hỏi | Trả lời |
|---|---|
| Callback là gì? | Hàm được truyền vào hàm khác như tham số |
| Khi nào nó được gọi? | Khi hàm chứa nó quyết định gọi (có thể ngay lập tức hoặc sau) |
| Hình thức thường gặp? | Anonymous function hoặc arrow function |
| Dùng để làm gì? | Xử lý sự kiện, bất đồng bộ, lặp qua mảng, ... |

---

## Tổng kết toàn bộ

```
HÀM TRONG JAVASCRIPT
│
├── Phân loại theo CÁCH KHAI BÁO
│   ├── Function Declaration:   function add() {}
│   └── Function Expression:    const add = function() {}
│
├── Phân loại theo TÊN
│   ├── Named:     function myFunc() {}
│   └── Anonymous: function() {}   (không có tên)
│
└── Phân loại theo CÁCH DÙNG
    ├── Hàm thường: gọi trực tiếp
    └── Callback:   truyền vào hàm khác như tham số
```

**Liên hệ giữa 3 khái niệm:**

```
Callback ──── thường là ──── Anonymous Function
                                    │
                             thường được viết dưới dạng
                                    │
                             Function Expression
                         (const fn = function() {})
```
