# Các Hàm Xử Lý Mảng Trong JavaScript

## Tổng Quan

JavaScript cung cấp nhiều hàm xử lý mảng được chia thành 2 nhóm:

**Nhóm 1 — Biến đổi / Tìm kiếm (không thay đổi mảng gốc):**

| Hàm | Trả về | Mục đích |
|-----|--------|----------|
| `map` | Mảng mới (cùng số phần tử) | Biến đổi từng phần tử |
| `filter` | Mảng mới (ít hoặc bằng phần tử) | Lọc phần tử theo điều kiện |
| `find` | Một phần tử hoặc `undefined` | Tìm phần tử đầu tiên thỏa điều kiện |
| `reduce` | Bất kỳ (số, chuỗi, object, mảng) | Gộp toàn bộ mảng thành một giá trị |
| `some` | `true` / `false` | Kiểm tra có ÍT NHẤT 1 phần tử thỏa điều kiện |
| `every` | `true` / `false` | Kiểm tra TẤT CẢ phần tử thỏa điều kiện |
| `sort` | Mảng đã sắp xếp (thay đổi mảng gốc!) | Sắp xếp phần tử |

**Nhóm 2 — Thêm / Xóa phần tử (thay đổi mảng gốc):**

| Hàm | Vị trí | Thêm/Xóa | Trả về |
|-----|--------|----------|--------|
| `push` | Cuối mảng | Thêm | Độ dài mới |
| `pop` | Cuối mảng | Xóa | Phần tử bị xóa |
| `unshift` | Đầu mảng | Thêm | Độ dài mới |
| `shift` | Đầu mảng | Xóa | Phần tử bị xóa |

---

## 1. Hàm `map`

### Cú pháp

```javascript
array.map((currentValue, index, array) => {
  return giá_trị_mới;
});
```

### Tham số trong callback

| Vị trí | Tên (tùy đặt) | Ý nghĩa |
|--------|--------------|---------|
| 1 | `currentValue` | Phần tử hiện tại |
| 2 | `index` | Chỉ số của phần tử (0, 1, 2...) |
| 3 | `array` | Mảng gốc |

> **Quan trọng:** JavaScript không quan tâm **tên biến**, chỉ quan tâm **thứ tự vị trí**.
> Dù đặt tên là `name1`, `index1` hay `abc`, `xyz` — vị trí thứ 2 luôn là index.

### Ví dụ

```javascript
// Tạo danh sách sinh viên từ mảng tên
const students = ['An', 'Bình', 'Cường'];
const studentList = students.map((name1, index1) => ({
  id: index1 + 1,
  name: name1,
  code: `SV00${index1 + 1}`
}));

console.log(studentList);
// [
//   { id: 1, name: 'An',    code: 'SV001' },
//   { id: 2, name: 'Bình',  code: 'SV002' },
//   { id: 3, name: 'Cường', code: 'SV003' }
// ]
```

---

## 2. Hàm `filter`

### Cú pháp

```javascript
array.filter((currentValue, index, array) => điều_kiện);
```

> `filter` chỉ giữ lại phần tử khi callback trả về **truthy**.

### Lọc với giá trị boolean

```javascript
// Hai cách tương đương nhau
products.filter(product => product.inStock === true)
products.filter(product => product.inStock)        // ngắn hơn

// Lọc ngược lại dùng !
products.filter(product => !product.inStock)          // hết hàng
products.filter(product => product.inStock === false) // hết hàng (tường minh)
```

### Ví dụ

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const evenNumbers = numbers.filter(num => num % 2 === 0);
console.log(evenNumbers); // [2, 4, 6, 8, 10]
console.log(numbers);     // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] - mảng gốc không đổi
```

```javascript
const products = [
  { name: 'iPhone 15',   price: 28000000, category: 'phone',     inStock: true  },
  { name: 'Samsung S24', price: 22000000, category: 'phone',     inStock: false },
  { name: 'iPad Pro',    price: 35000000, category: 'tablet',    inStock: true  },
  { name: 'MacBook Air', price: 32000000, category: 'laptop',    inStock: true  },
  { name: 'AirPods',     price: 4000000,  category: 'accessory', inStock: true  }
];

// Lọc sản phẩm còn hàng
const availableProducts = products.filter(product => product.inStock);
console.log(availableProducts); // 4 sản phẩm

// Lọc sản phẩm giá dưới 30 triệu
const affordableProducts = products.filter(product => product.price < 30000000);

// Lọc nhiều điều kiện: điện thoại còn hàng
const availablePhones = products.filter(product =>
  product.category === 'phone' && product.inStock
);
console.log(availablePhones); // [iPhone 15]
```

---

## 3. Hàm `find`

### Cú pháp

```javascript
array.find((currentValue, index, array) => điều_kiện);
```

### So sánh `find` và `filter`

| | `filter` | `find` |
|---|---|---|
| Trả về | Mảng mới | Một phần tử |
| Số lượng kết quả | Tất cả thỏa điều kiện | Chỉ phần tử **đầu tiên** |
| Không tìm thấy | `[]` (mảng rỗng) | `undefined` |
| Dừng sớm? | Không (duyệt hết mảng) | Có (dừng ngay khi tìm thấy) |

### Ví dụ

```javascript
const numbers = [1, 5, 3, 8, 2, 10, 7];

// Tìm số chẵn đầu tiên
const firstEven = numbers.find(num => num % 2 === 0);
console.log(firstEven); // 8 (không phải 2 hay 10 vì 8 xuất hiện trước)

// Không tìm thấy
const negative = numbers.find(num => num < 0);
console.log(negative); // undefined
```

```javascript
const users = [
  { id: 101, name: 'An',    role: 'admin', active: true  },
  { id: 102, name: 'Bình',  role: 'user',  active: false },
  { id: 103, name: 'Cường', role: 'user',  active: true  },
  { id: 104, name: 'Dũng',  role: 'admin', active: true  }
];

// Tìm user theo ID
const userId = 103;
const user = users.find(user => user.id === userId);
console.log(user); // { id: 103, name: 'Cường', role: 'user', active: true }

// Tìm admin đầu tiên
const firstAdmin = users.find(u => u.role === 'admin');
console.log(firstAdmin); // { id: 101, name: 'An', ... }

// Tìm user không active
const inactiveUser = users.find(u => !u.active);
console.log(inactiveUser); // { id: 102, name: 'Bình', ... }
```

---

## 4. Hàm `reduce`

### Cú pháp

```javascript
array.reduce((accumulator, currentValue, index, array) => {
  return kết_quả_mới;
}, giáTrịBanĐầu);
```

### Tham số trong callback

| Vị trí | Tên | Ý nghĩa |
|--------|-----|---------|
| 1 | `accumulator` | Giá trị tích lũy (kết quả của lần lặp trước) |
| 2 | `currentValue` | Phần tử hiện tại đang xử lý |
| 3 | `index` | Index của phần tử hiện tại (ít dùng) |
| 4 | `array` | Mảng gốc (rất ít dùng) |

### Giá trị ban đầu

```javascript
array.reduce(callback, giáTrịBanĐầu)
//                     ↑ giá trị khởi tạo cho accumulator ở vòng lặp đầu tiên
```

| Giá trị ban đầu | Kết quả cuối |
|----------------|-------------|
| `0` | Số |
| `''` | Chuỗi |
| `[]` | Mảng mới |
| `{}` | Object |

### Cách hoạt động từng bước

```javascript
const numbers = [1, 2, 3, 4, 5];

const sum = numbers.reduce((accumulator, current) => {
  return accumulator + current;
}, 0);

// Bước 1: accumulator = 0,  current = 1  -> return 1
// Bước 2: accumulator = 1,  current = 2  -> return 3
// Bước 3: accumulator = 3,  current = 3  -> return 6
// Bước 4: accumulator = 6,  current = 4  -> return 10
// Bước 5: accumulator = 10, current = 5  -> return 15

console.log(sum); // 15
```

### Khi nào bỏ giá trị ban đầu?

```javascript
const words = ['Xin', 'chào', 'Việt', 'Nam'];

// Không có giá trị ban đầu
// → phần tử đầu tiên ('Xin') tự động làm accumulator
// → vòng lặp bắt đầu từ phần tử thứ 2
const sentence = words.reduce((acc, word) => acc + ' ' + word);
console.log(sentence); // 'Xin chào Việt Nam'

// Có giá trị ban đầu ''
// → vòng lặp từ phần tử đầu tiên
const sentence2 = words.reduce((acc, word) => acc + ' ' + word, '');
console.log(sentence2); // ' Xin chào Việt Nam' ← có dấu cách thừa ở đầu!
```

> **Quy tắc:** Bỏ giá trị ban đầu khi kiểu kết quả **giống kiểu phần tử** trong mảng.
> Với `[]` hoặc `{}` — bắt buộc phải có giá trị ban đầu.

### Ví dụ gộp thành chuỗi

```javascript
const words = ['Xin', 'chào', 'Việt', 'Nam'];

const sentence = words.reduce((acc, word) => acc + ' ' + word);
console.log(sentence); // 'Xin chào Việt Nam'

const csv = words.reduce((acc, word) => acc + ',' + word);
console.log(csv); // 'Xin,chào,Việt,Nam'
```

### Ví dụ gộp thành mảng

```javascript
const numbers = [1, 2, 3, 4, 5];

// Lọc số chẵn và nhân đôi
const result = numbers.reduce((acc, num) => {
  if (num % 2 === 0) {
    acc.push(num * 2);
  }
  return acc;
}, []);
console.log(result); // [4, 10]
```

### Ví dụ gộp thành object

```javascript
const users = [
  { id: 1, name: 'An' },
  { id: 2, name: 'Bình' },
  { id: 3, name: 'Cường' }
];

const usersById = users.reduce((acc, user) => {
  acc[user.id] = user;
  // acc[user.id] nghĩa là: tạo key có tên = GIÁ TRỊ của user.id (động)
  // Khác với acc.id — đó là key cố định tên "id"
  return acc;
}, {});

console.log(usersById);
// {
//   1: { id: 1, name: 'An' },
//   2: { id: 2, name: 'Bình' },
//   3: { id: 3, name: 'Cường' }
// }

// Truy cập nhanh theo id
console.log(usersById[2]); // { id: 2, name: 'Bình' }
```

> **Giải thích `acc[user.id] = user`:**
> - `acc` — object đang tích lũy
> - `user.id` — lấy giá trị id (1, 2, 3...)
> - `acc[user.id]` — tạo key **động** theo giá trị id (dùng `[]` thay vì `.` vì key là biến)
> - `= user` — gán toàn bộ object user làm value

### Ví dụ tính tiền giỏ hàng

```javascript
const cart = [
  { product: 'Laptop',   price: 20000000, quantity: 1 },
  { product: 'Mouse',    price: 300000,   quantity: 2 },
  { product: 'Keyboard', price: 800000,   quantity: 1 },
  { product: 'Monitor',  price: 5000000,  quantity: 2 }
];

const totalAmount = cart.reduce((total, item) => {
  return total + (item.price * item.quantity);
}, 0);

console.log(`Tổng tiền: ${totalAmount.toLocaleString('vi-VN')}đ`);
// Tổng tiền: 31.400.000đ

const totalItems = cart.reduce((count, item) => count + item.quantity, 0);
console.log(`Tổng số sản phẩm: ${totalItems}`); // 6
```

---

## 5. Hàm `some`

### Cú pháp

```javascript
array.some((currentValue, index, array) => điều_kiện);
```

> Trả về `true` nếu **ít nhất 1** phần tử thỏa điều kiện.
> **Dừng ngay** khi tìm thấy phần tử đầu tiên thỏa điều kiện.

### Ví dụ

```javascript
const numbers = [1, 3, 5, 7, 8, 9];

const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true (vì có số 8)

const hasGreaterThan10 = numbers.some(num => num > 10);
console.log(hasGreaterThan10); // false

// Minh họa dừng sớm
const hasEvenWithLog = numbers.some(num => {
  console.log(`Checking: ${num}`);
  return num % 2 === 0;
});
// Checking: 1
// Checking: 3
// Checking: 5
// Checking: 7
// Checking: 8
// => Dừng, không kiểm tra 9
```

### Ví dụ kiểm tra quyền

```javascript
const user = {
  name: 'Nguyễn Văn A',
  roles: ['user', 'editor']
};

const adminRoles = ['admin', 'superadmin'];
const editorRoles = ['editor', 'admin'];

// Kiểm tra user có quyền editor không?
const canEdit = user.roles.some(role => editorRoles.includes(role));
console.log(canEdit); // true (có role 'editor')

// Kiểm tra user có phải admin không?
const isAdmin = user.roles.some(role => adminRoles.includes(role));
console.log(isAdmin); // false

// Hàm kiểm tra quyền tổng quát
function hasPermission(userRoles, requiredRoles) {
  return userRoles.some(role => requiredRoles.includes(role));
}

console.log(hasPermission(user.roles, ['viewer', 'editor'])); // true
```

---

## 6. Hàm `every`

### Cú pháp

```javascript
array.every((currentValue, index, array) => điều_kiện);
```

> Trả về `true` nếu **tất cả** phần tử thỏa điều kiện.
> **Dừng ngay** khi gặp phần tử **đầu tiên KHÔNG** thỏa điều kiện.

### So sánh `some` và `every`

| | `some` | `every` |
|---|---|---|
| Trả về `true` khi | ít nhất 1 thỏa | tất cả thỏa |
| Trả về `false` khi | tất cả không thỏa | ít nhất 1 không thỏa |
| Dừng sớm khi | tìm thấy `true` đầu tiên | tìm thấy `false` đầu tiên |

### Ví dụ

```javascript
const numbers = [2, 4, 6, 8, 10];

const allEven = numbers.every(num => num % 2 === 0);
console.log(allEven); // true

const allGreaterThan5 = numbers.every(num => num > 5);
console.log(allGreaterThan5); // false (2 và 4 không > 5)

// Minh họa dừng sớm
const checkWithLog = numbers.every(num => {
  console.log(`Checking: ${num}`);
  return num < 5;
});
// Checking: 2
// Checking: 4
// Checking: 6
// => Dừng tại 6, không kiểm tra 8, 10
console.log(checkWithLog); // false
```

### Ví dụ validation đơn hàng

```javascript
const orderItems = [
  { product: 'iPhone 15', quantity: 1, inStock: 5,  price: 25000000 },
  { product: 'AirPods',   quantity: 2, inStock: 10, price: 4000000  },
  { product: 'Case',      quantity: 1, inStock: 20, price: 500000   }
];

// Tất cả sản phẩm còn đủ hàng?
const allAvailable = orderItems.every(item => item.inStock >= item.quantity);
console.log(allAvailable); // true

// Tổng hợp validation
function canPlaceOrder(items) {
  return items.every(item =>
    item.inStock >= item.quantity &&
    item.price > 0 &&
    item.quantity > 0 &&
    item.quantity <= 10
  );
}
```

---

## 7. Hàm `sort`

### Cú pháp

```javascript
array.sort();                        // mặc định (sắp xếp theo chuỗi)
array.sort((a, b) => a - b);        // số tăng dần
array.sort((a, b) => b - a);        // số giảm dần
```

> **Cảnh báo:** `sort` **thay đổi mảng gốc** (khác với `map`, `filter`, `find`, `reduce`).

### BUG phổ biến với số

```javascript
const numbers = [10, 5, 40, 25, 1000, 1];

numbers.sort();
console.log(numbers); // [1, 10, 1000, 25, 40, 5] - SAI!
// Vì sort mặc định chuyển thành string: "10" < "5" (so sánh ký tự)
```

### Cách đúng — dùng compare function

```javascript
// Tăng dần
const numbers2 = [10, 5, 40, 25, 1000, 1];
numbers2.sort((a, b) => a - b);
console.log(numbers2); // [1, 5, 10, 25, 40, 1000] - ĐÚNG!

// Giảm dần
const numbers3 = [10, 5, 40, 25, 1000, 1];
numbers3.sort((a, b) => b - a);
console.log(numbers3); // [1000, 40, 25, 10, 5, 1]
```

### Compare function trả về gì?

```javascript
// Compare function trả về:
// - Số âm:  a đứng trước b
// - Số 0:   giữ nguyên thứ tự
// - Số dương: b đứng trước a

const arr = [3, 1, 2];
arr.sort((a, b) => {
  if (a < b) return -1;  // a trước b
  if (a > b) return 1;   // b trước a
  return 0;              // bằng nhau
});
console.log(arr); // [1, 2, 3]

// Viết gọn với phép trừ (tương đương)
arr.sort((a, b) => a - b);
```

### Sort chuỗi

```javascript
const fruits = ['banana', 'apple', 'orange', 'grape'];
fruits.sort();
console.log(fruits); // ['apple', 'banana', 'grape', 'orange']
```

---

## 8. Thêm / Xóa Phần Tử Mảng

### `push` — Thêm vào cuối

```javascript
const fruits = ['apple', 'banana'];
const newLength = fruits.push('orange');

console.log(fruits);    // ['apple', 'banana', 'orange']
console.log(newLength); // 3 (trả về độ dài mới)

// Thêm nhiều phần tử cùng lúc
fruits.push('grape', 'mango');
console.log(fruits); // ['apple', 'banana', 'orange', 'grape', 'mango']
```

### `pop` — Xóa ở cuối

```javascript
const fruits = ['apple', 'banana', 'orange', 'grape'];
const lastFruit = fruits.pop();

console.log(lastFruit); // 'grape' (trả về phần tử bị xóa)
console.log(fruits);    // ['apple', 'banana', 'orange']

// Pop từ mảng rỗng
const empty = [];
console.log(empty.pop()); // undefined
```

### `unshift` — Thêm vào đầu

```javascript
const fruits = ['banana', 'orange'];
const newLength = fruits.unshift('apple');

console.log(fruits);    // ['apple', 'banana', 'orange']
console.log(newLength); // 3 (trả về độ dài mới)

// Thêm nhiều phần tử cùng lúc
fruits.unshift('grape', 'mango');
console.log(fruits); // ['grape', 'mango', 'apple', 'banana', 'orange']
```

### `shift` — Xóa ở đầu

```javascript
const fruits = ['apple', 'banana', 'orange', 'grape'];
const firstFruit = fruits.shift();

console.log(firstFruit); // 'apple' (trả về phần tử bị xóa)
console.log(fruits);     // ['banana', 'orange', 'grape']

// Shift từ mảng rỗng
const empty = [];
console.log(empty.shift()); // undefined
```

### Tổng kết push / pop / unshift / shift

```javascript
const arr = [2, 3, 4];

arr.push(5);      // [2, 3, 4, 5]  — thêm cuối
arr.pop();        // [2, 3, 4]     — xóa cuối
arr.unshift(1);   // [1, 2, 3, 4]  — thêm đầu
arr.shift();      // [2, 3, 4]     — xóa đầu
```

---

## 9. Arrow Function — Implicit Return

Khi arrow function chỉ có **1 dòng logic**, có thể bỏ `{}` và `return`:

```javascript
// Cách đầy đủ - có {} nên phải có return
cart.reduce((count, item) => {
  return count + item.quantity;
}, 0);

// Cách viết tắt - không có {} nên tự động return (implicit return)
cart.reduce((count, item) => count + item.quantity, 0);
```

### Quy tắc

| Cách viết | Có `{}` | Cần `return` |
|-----------|---------|--------------|
| Đầy đủ | Có | Bắt buộc |
| Viết tắt | Không | Tự động (implicit return) |

### Lưu ý khi return object

Khi viết tắt mà trả về **object**, phải bọc trong `()` để JS không nhầm `{}` là function body:

```javascript
// Sai - JS hiểu {} là function body, không phải object
students.map((name, index) => { id: index, name: name })

// Đúng - bọc object trong ()
students.map((name, index) => ({ id: index, name: name }))
```

---

## Tổng Kết So Sánh

```javascript
const numbers = [1, 2, 3, 4, 5];

// map - biến đổi từng phần tử, trả về mảng cùng số lượng
numbers.map(n => n * 2);                    // [2, 4, 6, 8, 10]

// filter - lọc, trả về mảng con
numbers.filter(n => n % 2 === 0);           // [2, 4]

// find - tìm phần tử đầu tiên thỏa điều kiện
numbers.find(n => n % 2 === 0);             // 2

// reduce - gộp thành một giá trị
numbers.reduce((acc, n) => acc + n, 0);     // 15

// some - có ít nhất 1 thỏa không?
numbers.some(n => n > 4);                   // true

// every - tất cả có thỏa không?
numbers.every(n => n > 0);                  // true
```
