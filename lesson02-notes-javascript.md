# Tóm tắt JavaScript Basic — Lesson 1

---

## 1. JavaScript là gì?

- Ra đời năm **1995** bởi **Brendan Eich**, là ngôn ngữ lập trình phổ biến nhất thế giới
- Vai trò trong web: **HTML** = xương, **CSS** = da, **JS** = não
- Chạy trên browser nhờ engine: Chrome (**V8**), Firefox (SpiderMonkey), Edge (Chakra)
- Chạy trên máy tính (không có browser) → cần **NodeJS**

---

## 2. Hello World

```js
console.log("Hello world!");   // nháy kép
console.log('Hello world!');   // nháy đơn — đều được
```

Chạy file:
```bash
node tên-file.js
node thư-mục/tên-file.js   # nếu file nằm trong thư mục
```

---

## 3. Comment

```js
// Comment 1 dòng

/* Comment
   nhiều dòng */

const name = "Playwright"; // có thể thêm vào cuối dòng
```

> Code được comment → **bị bỏ qua**, không thực thi

---

## 4. Biến & Hằng

| Từ khóa | Đặc điểm | Dùng khi |
|---|---|---|
| `const` | Không thay đổi được | **Mặc định dùng cái này** |
| `let` | Thay đổi được, phạm vi block | Chắc chắn cần gán lại |
| `var` | Cũ, dễ gây lỗi, phạm vi global | **Không dùng** |

```js
const PI = 3.14;       // hằng số — không thay đổi
let count = 0;         // biến
count = 1;             // OK — gán lại được

// var vs let
var x = 1;
var x = 2;             // OK (nguy hiểm — dễ gây nhầm)

let y = 1;
let y = 2;             // LỖI: Identifier 'y' has already been declared

// Phạm vi block
{
    var a = 10;
    let b = 11;
}
console.log(a);        // 10 — var thoát ra ngoài block
console.log(b);        // LỖI: b is not defined
```

> **Quy tắc:** Mặc định dùng `const` → chỉ dùng `let` khi cần gán lại → **không dùng `var`**

---

## 5. Kiểu dữ liệu (Data Types)

JavaScript có 8 kiểu dữ liệu, tập trung 3 kiểu chính:

| Kiểu | Mô tả | Ví dụ |
|---|---|---|
| `Number` | Số nguyên và số thực | `25`, `19.99`, `Infinity`, `NaN` |
| `String` | Chuỗi ký tự | `"John"`, `'Hello'`, `` `Age: 10` `` |
| `Boolean` | Giá trị logic | `true`, `false` |

```js
const age = 25;                     // Number — số nguyên
const price = 19.99;               // Number — số thực
const name = "John";               // String — nháy kép
const message = 'Hello';           // String — nháy đơn
const template = `Age: ${age}`;    // String — backtick (template literal)
const isActive = true;             // Boolean
const isComplete = false;          // Boolean
```

Kiểm tra kiểu dữ liệu:
```js
typeof age        // "number"
typeof name       // "string"
typeof isActive   // "boolean"
```

---

## 6. Toán tử so sánh

| Toán tử | Tên | Ý nghĩa |
|---|---|---|
| `===` | Strict equality | So sánh **giá trị VÀ kiểu** — **nên dùng** |
| `==` | Loose equality | So sánh giá trị, tự chuyển đổi kiểu |
| `!==` | Strict not equal | Khác giá trị hoặc khác kiểu |
| `!=` | Not equal | Khác giá trị (có chuyển kiểu) |
| `>` | Lớn hơn | |
| `<` | Nhỏ hơn | |
| `>=` | Lớn hơn hoặc bằng | |
| `<=` | Nhỏ hơn hoặc bằng | |

```js
// == (Loose Equality) — tự chuyển kiểu
5 == "5"      // true  (chuyển string → number)
true == 1     // true  (true → 1)
false == 0    // true  (false → 0)

// === (Strict Equality) — không chuyển kiểu — NÊN DÙNG
5 === "5"     // false (khác kiểu)
5 === 5       // true  (cùng kiểu, cùng giá trị)

// != và !==
5 != "5"      // false (có chuyển kiểu)
5 !== "5"     // true  (khác kiểu)
```

> **Quy tắc:** Luôn dùng `===`, chỉ dùng `==` khi cố ý bỏ qua kiểu dữ liệu

---

## 7. Toán tử logic

| Toán tử | Tên | Kết quả đúng khi |
|---|---|---|
| `&&` | AND | **Cả 2 vế** đúng |
| `\|\|` | OR | **Một trong 2 vế** đúng |

| A | B | A && B | A \|\| B |
|---|---|---|---|
| true | true | true | true |
| true | false | false | true |
| false | true | false | true |
| false | false | false | false |

```js
// Ví dụ thực tế
const isMoreThan6months = true;
const isOfficialEmployee = true;

// Lương tháng 13: làm trên 6 tháng HOẶC nhân viên chính thức
const receive13 = isMoreThan6months || isOfficialEmployee;  // true

// Lương tháng 14: làm trên 6 tháng VÀ nhân viên chính thức
const receive14 = isMoreThan6months && isOfficialEmployee;  // true
```

---

## 8. Toán tử toán học & một ngôi

```js
// Toán học — cộng trừ nhân chia
const result = 5 + 10;   // 15
const div = 10 / 0;      // Infinity (lưu ý: chia cho 0 không báo lỗi)

// Một ngôi — tăng/giảm 1 đơn vị
let x = 5;
++x;   // prefix:  tăng TRƯỚC rồi trả về → x = 6
x++;   // postfix: trả về TRƯỚC rồi mới tăng → trả về 6, x = 7
--x;   // tương tự nhưng giảm
x--;

// Ví dụ quan trọng
let a = 10;
let b = ++a;   // a tăng lên 11 trước → b = 11

let c = 10;
let d = c++;   // trả về 10 cho d trước → d = 10, c = 11
```

---

## 9. Câu điều kiện `if`

```js
// Cú pháp
if (<điều kiện>) {
    // chạy khi điều kiện đúng
}

// Ví dụ đơn giản
let hour = 8;
if (hour <= 11) {
    console.log("Good morning");
}

// Kết hợp nhiều điều kiện
if (hour >= 6 && hour <= 11) {
    console.log("Good morning");
}

// if...else
if (hour < 12) {
    console.log("Buổi sáng");
} else {
    console.log("Buổi chiều/tối");
}
```

Các dạng điều kiện: `if` / `if...else` / `if...else if...else` / `switch...case`

---

## 10. Vòng lặp `for (i)`

```js
// Cú pháp
for (<khởi tạo>; <điều kiện lặp>; <cập nhật>) {
    // code
}

// Ví dụ: in "Xin chào!" 5 lần (i = 0, 1, 2, 3, 4)
for (let i = 0; i < 5; i++) {
    console.log("Xin chào!");
}
// Khi i = 5 → i < 5 sai → dừng vòng lặp
```

Giải thích 3 phần:
- **Khởi tạo** (`let i = 0`): chạy **1 lần duy nhất** khi bắt đầu
- **Điều kiện lặp** (`i < 5`): đúng → chạy tiếp, sai → dừng
- **Cập nhật** (`i++`): chạy vào **cuối mỗi vòng lặp**

Các loại vòng lặp khác (học ở bài sau): `for (of)`, `for (each)`, `for (in)`, `while`, `do...while`

---

## VSCode Tips

| Phím tắt | Chức năng |
|---|---|
| `Alt + Shift + F` (Windows) | Format code |
| `Option + Shift + F` (Mac) | Format code |
