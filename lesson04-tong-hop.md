# Lesson 4 - JavaScript (Nâng Cao)

## Nội dung buổi học
1. Phạm vi của biến (Scope)
2. break & continue
3. Vòng lặp nâng cao
4. Câu điều kiện nâng cao
5. Utils functions (String & Array)

---

## 1. Phạm vi của biến (Variable Scope)

> **Phạm vi (scope)** xác định nơi mà biến có thể được truy cập.

JavaScript có **3 loại phạm vi**:

| Loại | Từ khóa | Giới hạn |
|------|---------|----------|
| Block scope | `let`, `const` | Trong cặp `{}` |
| Function scope | `var`, `let`, `const` | Trong hàm |
| Global scope | `var`, `let`, `const` | Toàn bộ file |

### 1.1 Block Scope (Phạm vi khối)

Biến khai báo bên trong `{}`. Chỉ `let` và `const` bị giới hạn bởi block, **`var` thì không**.

```javascript
if (true) {
  var varVariable = "var không có block scope";
  let letVariable = "let có block scope";
  const constVariable = "const cũng có block scope";
}

console.log(varVariable);   // OK - var không bị giới hạn bởi block
console.log(letVariable);   // Error: letVariable is not defined
console.log(constVariable); // Error: constVariable is not defined
```

> **Mẹo nhớ:** `var` = "vượt rào" ra khỏi `{}`; `let`/`const` = "ở yên trong block"

### 1.2 Function Scope (Phạm vi hàm)

Biến khai báo bên trong hàm. **Tất cả** `var`/`let`/`const` ra ngoài hàm đều bị **undefined**.

```javascript
function myFunction() {
  var functionScoped = "Chỉ có thể truy cập trong hàm này";
  let alsoFunctionScoped = "Tương tự";

  console.log(functionScoped); // OK
}

console.log(functionScoped); // Error: functionScoped is not defined
```

### 1.3 Global Scope (Phạm vi toàn cục)

Biến khai báo **ngoài** tất cả hàm và khối `{}`. Truy cập được từ bất kỳ đâu.

```javascript
var globalVar = "Tôi là biến toàn cục";
let globalLet = "Tôi cũng là biến toàn cục";

function testFunction() {
  console.log(globalVar); // Truy cập được
  console.log(globalLet); // Truy cập được
}
```

---

## 2. break và continue

### 2.1 break

**`break`** dùng để **thoát hoàn toàn** khỏi vòng lặp ngay lập tức.

```javascript
// Thoát khi tìm thấy giá trị
for (let i = 0; i < 10; i++) {
  if (i === 5) {
    break; // Thoát vòng lặp khi i = 5
  }
  console.log(i);
}
// Output: 0, 1, 2, 3, 4
```

**Ví dụ thực tế:** Tìm số chẵn đầu tiên trong mảng

```javascript
const numbers = [1, 3, 8, 7, 9, 11];
let firstEven = null;

for (let num of numbers) {
  if (num % 2 === 0) {
    firstEven = num;
    break; // Dừng ngay khi tìm thấy
  }
}
// firstEven = 8
```

### 2.2 continue

**`continue`** dùng để **bỏ qua** lần lặp hiện tại và **chuyển sang lần lặp tiếp theo**.

```javascript
// Bỏ qua số chẵn, chỉ in số lẻ
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) {
    continue; // Bỏ qua số chẵn
  }
  console.log(i);
}
// Output: 1, 3, 5, 7, 9
```

**Ví dụ thực tế:** Lọc điểm >= 80

```javascript
const scores = [85, 92, 78, 95, 60, 88];
console.log("Điểm >= 80:");

for (let i = 0; i < scores.length; i++) {
  const score = scores[i];
  if (score < 80) {
    continue; // Bỏ qua điểm < 80
  }
  console.log(score);
}
// Output: 85, 92, 95, 88
```

> **So sánh nhanh:**
> - `break` → thoát hẳn khỏi vòng lặp
> - `continue` → bỏ qua lần này, tiếp tục lần sau

---

## 3. Vòng lặp nâng cao

### 3.1 for...in Loop

Dùng để **duyệt qua các thuộc tính (key)** của một **object**.

```javascript
const person = {
  name: "John",
  age: 30,
  city: "Hanoi"
};

for (let key in person) {
  console.log(key + ": " + person[key]);
}
// Output:
// name: John
// age: 30
// city: Hanoi
```

> **Lưu ý:** `for...in` với Array **không được khuyến khích** vì nó duyệt theo index (0, 1, 2...) thay vì giá trị.

```javascript
// Với Array (không khuyến khích dùng for...in)
const colors = ["red", "green", "blue"];
for (let index in colors) {
  console.log(index + ": " + colors[index]);
}
// Output: 0: red | 1: green | 2: blue
```

### 3.2 forEach Method

Method của Array để **thực thi một function cho mỗi phần tử**.

> **Lưu ý quan trọng:** `forEach` **KHÔNG THỂ** dùng `break` hoặc `continue`.

```javascript
const numbers = [1, 2, 3, 4, 5];

numbers.forEach(function(value) {
  console.log(value);
});
// Output: 1, 2, 3, 4, 5
```

---

## 4. Câu điều kiện nâng cao

### 4.1 if...else

```javascript
let score = 75;

if (score >= 60) {
  console.log("Bạn đã qua môn");
} else {
  console.log("Bạn cần học lại");
}
```

### 4.2 if...else if...else (nhiều điều kiện)

```javascript
let score = 85;

if (score >= 90) {
  console.log("Xuất sắc");
} else if (score >= 80) {
  console.log("Giỏi");
} else if (score >= 70) {
  console.log("Khá");
} else if (score >= 60) {
  console.log("Trung bình");
} else {
  console.log("Yếu");
}
// Output: Giỏi
```

### 4.3 Ternary Operator (Toán tử điều kiện)

Cách viết **ngắn gọn** thay cho `if...else` đơn giản.

**Cú pháp:** `điều_kiện ? giá_trị_nếu_true : giá_trị_nếu_false`

```javascript
let age = 20;
let status = (age >= 18) ? "Người lớn" : "Trẻ em";
console.log(status); // "Người lớn"
```

**Ternary lồng nhau** (nên cẩn thận với độ phức tạp):

```javascript
let score = 75;
let grade = score >= 90 ? "A" :
            score >= 80 ? "B" :
            score >= 70 ? "C" :
            score >= 60 ? "D" : "F";
// grade = "C"
```

---

## 5. Utils Functions

> **Utils = tiện ích** — là các **hàm có sẵn** của JavaScript, giúp code nhanh hơn, gọn hơn.

### 5.1 String Utils (Xử lý chuỗi)

#### Bỏ khoảng trắng — `trim()`

| Hàm | Tác dụng |
|-----|----------|
| `trim()` | Bỏ khoảng trắng cả 2 đầu |
| `trimStart()` | Bỏ khoảng trắng bên trái |
| `trimEnd()` | Bỏ khoảng trắng bên phải |

```javascript
let text = "   Hello World   ";

console.log(text.trim());      // "Hello World"
console.log(text.trimStart()); // "Hello World   "
console.log(text.trimEnd());   // "   Hello World"
```

#### Chuyển đổi HOA/thường — `toUpperCase()` / `toLowerCase()`

```javascript
let str = "JavaScript";

console.log(str.toUpperCase()); // "JAVASCRIPT"
console.log(str.toLowerCase()); // "javascript"
```

#### Kiểm tra chuỗi con — `includes()`

```javascript
let text = "Hello World";

console.log(text.includes("World"));      // true
console.log(text.includes("JavaScript")); // false
console.log(text.includes("world"));      // false (phân biệt hoa thường)
```

#### Cắt chuỗi — `split()`

```javascript
let text = "Hello World JavaScript";
console.log(text.split(" ")); // ["Hello", "World", "JavaScript"]

let email = "user@gmail.com";
console.log(email.split("@")); // ["user", "gmail.com"]

let date = "2024-01-25";
console.log(date.split("-")); // ["2024", "01", "25"]
```

#### Thay thế chuỗi — `replace()`

```javascript
let text = "Hello World";
console.log(text.replace("World", "JavaScript")); // "Hello JavaScript"
```

---

### 5.2 Array Utils (Xử lý mảng)

#### Thêm phần tử vào mảng

| Hàm | Vị trí thêm | Cú pháp |
|-----|-------------|---------|
| `push()` | Cuối mảng | `arr.push(phần_tử)` |
| `unshift()` | Đầu mảng | `arr.unshift(phần_tử)` |
| `splice()` | Vị trí bất kỳ | `arr.splice(vị_trí, 0, phần_tử)` |

```javascript
let arr = [1, 2, 3];

arr.push(4);       // [1, 2, 3, 4]
arr.unshift(0);    // [0, 1, 2, 3, 4]
arr.splice(2, 0, 1.5); // [0, 1, 1.5, 2, 3, 4]
```

#### Xóa phần tử khỏi mảng

| Hàm | Vị trí xóa | Cú pháp |
|-----|------------|---------|
| `pop()` | Cuối mảng | `arr.pop()` |
| `shift()` | Đầu mảng | `arr.shift()` |
| `splice()` | Vị trí bất kỳ | `arr.splice(vị_trí, số_lượng)` |

```javascript
let arr = [1, 2, 3, 4, 5];

arr.pop();         // [1, 2, 3, 4]  - xóa phần tử cuối
arr.shift();       // [2, 3, 4]     - xóa phần tử đầu
arr.splice(1, 1);  // [2, 4]        - xóa 1 phần tử tại index 1
```

#### Tìm kiếm phần tử — `find()` & `filter()`

| Hàm | Trả về |
|-----|--------|
| `find()` | Phần tử **đầu tiên** thỏa điều kiện |
| `filter()` | **Tất cả** phần tử thỏa điều kiện (mảng mới) |

```javascript
const numbers = [5, 12, 8, 130, 44];

let first = numbers.find(num => num > 10);
console.log(first); // 12

let all = numbers.filter(num => num > 10);
console.log(all); // [12, 130, 44]
```

#### Biến đổi mảng — `map()`

Tạo mảng mới bằng cách **áp dụng một hàm lên từng phần tử**. Trả về mảng mới có **cùng độ dài**.

```javascript
let numbers = [1, 2, 3, 4, 5];

let doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
```

#### Sắp xếp mảng — `sort()`

```javascript
let numbers = [40, 100, 1, 5, 25, 10];

// Sắp xếp tăng dần
numbers.sort((a, b) => a - b);
console.log(numbers); // [1, 5, 10, 25, 40, 100]

// Sắp xếp giảm dần
numbers.sort((a, b) => b - a);
console.log(numbers); // [100, 40, 25, 10, 5, 1]
```

**Cách sort hoạt động:**
- Trả về **số âm** → `a` đứng trước `b`
- Trả về **số dương** → `b` đứng trước `a`
- Trả về **0** → giữ nguyên thứ tự

---

## Tổng kết nhanh

| Chủ đề | Keyword/Hàm | Tác dụng |
|--------|-------------|----------|
| **Scope** | `var` | Không bị giới hạn bởi block `{}` |
| **Scope** | `let`/`const` | Bị giới hạn trong block `{}` hoặc hàm |
| **Vòng lặp** | `break` | Thoát khỏi vòng lặp ngay lập tức |
| **Vòng lặp** | `continue` | Bỏ qua lần lặp hiện tại |
| **Vòng lặp** | `for...in` | Duyệt key của object |
| **Vòng lặp** | `forEach` | Duyệt từng phần tử mảng |
| **Điều kiện** | `? :` (ternary) | Rút gọn if...else |
| **String** | `trim()` | Bỏ khoảng trắng |
| **String** | `toUpperCase/toLowerCase` | Đổi hoa/thường |
| **String** | `includes()` | Kiểm tra chuỗi con |
| **String** | `split()` | Cắt chuỗi thành mảng |
| **String** | `replace()` | Thay thế chuỗi |
| **Array** | `push/unshift/splice` | Thêm phần tử |
| **Array** | `pop/shift/splice` | Xóa phần tử |
| **Array** | `find/filter` | Tìm kiếm phần tử |
| **Array** | `map()` | Biến đổi mảng |
| **Array** | `sort()` | Sắp xếp mảng |

> **Tài liệu tham khảo:**
> - String methods: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String
> - Array methods: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
