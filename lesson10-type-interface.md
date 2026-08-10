# Bài 10: Type vs Interface trong TypeScript

## 1. Giống nhau

Cả `type` và `interface` đều dùng để định nghĩa **cấu trúc dữ liệu** (shape của object). Với các trường hợp cơ bản, hai cách viết dưới đây tương đương nhau:

```ts
// Interface
interface User {
  name: string;
  age: number;
}

// Type
type User = {
  name: string;
  age: number;
};

const dog: Animal = {
  name: "Buddy",
  age: 3,
};
```

Vì vậy với object đơn giản, dùng `type` hay `interface` đều được — sự khác biệt chỉ nằm ở các tính năng nâng cao bên dưới.

---

## 2. Khác biệt chính

### 2.1. Interface có thể mở rộng (Declaration Merging)

Một điểm đặc biệt của `interface`: nếu khai báo **cùng tên** nhiều lần, TypeScript sẽ **tự động gộp (merge)** các thuộc tính lại với nhau, thay vì báo lỗi.

```ts
interface Animal {
  name: string;
}

interface Animal {
  age: number;
}

// Kết quả: Animal có cả name và age
const dog: Animal = {
  name: "Buddy",
  age: 3,
};
```

Ngược lại, `type` **không thể** khai báo lại cùng tên — sẽ báo lỗi ngay:

```ts
type Animal = { name: string };
type Animal = { age: number }; // ❌ Lỗi: Duplicate identifier 'Animal'
```

> 💡 **Kiến thức bổ sung:** Declaration Merging rất hữu ích khi làm việc với **thư viện bên thứ ba**. Ví dụ khi bạn muốn mở rộng interface `Window` của trình duyệt hoặc `Request` của Express mà không cần sửa code gốc của thư viện — chỉ cần khai báo thêm một `interface` cùng tên ở file của bạn.

### 2.2. Type linh hoạt hơn với Union & Intersection

`type` hỗ trợ **Union** (`|`) để gộp nhiều kiểu có thể xảy ra, và **Intersection** (`&`) để gộp nhiều kiểu lại làm một:

```ts
// Union Types
type Status = "active" | "inactive" | "pending";
const myStatus: Status = "pending";

type ID = string | number;
const myId: ID = 100;
const hisId: ID = "100";
// const herId: ID = true; ❌ Lỗi vì boolean không nằm trong union

// Intersection
type Person = { name: string };
type Employee = { salary: number };
type Worker = Person & Employee; // { name: string; salary: number }
```

`interface` **không hỗ trợ Union**, nhưng có thể đạt hiệu quả tương tự Intersection bằng từ khóa `extends`:

```ts
interface Person {
  name: string;
}

interface Employee extends Person {
  salary: number;
}

const abc: Employee = {
  name: "abc",
  salary: 200,
};
```

> 💡 **Bổ sung:** `extends` của interface không chỉ "gộp" mà còn kiểm tra tính tương thích — nếu interface con định nghĩa lại một thuộc tính đã có ở interface cha với kiểu dữ liệu khác (không tương thích), TypeScript sẽ báo lỗi ngay khi khai báo. Trong khi đó `&` (intersection) của `type` sẽ cố gắng gộp và có thể tạo ra kiểu `never` cho thuộc tính bị xung đột (ví dụ cùng tên nhưng một bên là `string`, một bên là `number` → kiểu đó thành `never`, không dùng được).

### 2.3. Type có thể đặt tên cho Primitives, Union, Tuple

```ts
// Primitives
type Name = string;
type Age = number;

// Tuple
type Point = [number, number];

// Union
type Result = Success | Error;

// Interface KHÔNG làm được điều này ❌
```

`interface` chỉ dùng để mô tả **hình dạng của object** (hoặc class), không thể dùng để đặt tên cho kiểu nguyên thủy (`string`, `number`...), tuple, hay union.

```ts
type Name = string;
const myName: string = "Phong";
const myName2: Name = "Phong"; // Name chỉ là "alias" của string
```

### 2.4. Type có thể dùng Mapped Types

`type` hỗ trợ **Mapped Types** — kỹ thuật tạo type mới bằng cách "duyệt" qua các thuộc tính của một type khác:

```ts
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type User = {
  name: string;
  age: number;
};

type ReadonlyUser = Readonly<User>;
// { readonly name: string; readonly age: number }
```

> 💡 **Bổ sung:** `Readonly<T>` ở trên chính là cách TypeScript định nghĩa **utility type** có sẵn `Readonly<T>` (built-in). Các utility type phổ biến khác cũng dùng Mapped Types tương tự: `Partial<T>` (mọi thuộc tính thành optional), `Required<T>` (mọi thuộc tính thành bắt buộc), `Pick<T, K>` (chọn một số thuộc tính), `Omit<T, K>` (loại bỏ một số thuộc tính), `Record<K, V>` (tạo object với key kiểu K, value kiểu V).

`interface` **không thể** dùng Mapped Types.

### 2.5. Interface extends nhiều interface dễ đọc hơn

```ts
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

// Rõ ràng, dễ đọc
interface ColorfulCircle extends Colorful, Circle {}

// Type cũng làm được nhưng dùng &
type ColorfulCircle = Colorful & Circle;
```

Cả hai cách đều cho kết quả tương đương, nhưng cú pháp `extends A, B` của interface thường được đánh giá là **dễ đọc và tường minh hơn** khi kế thừa nhiều nguồn, đặc biệt trong các dự án lớn.

---

## 3. Khi nào dùng gì?

### ✅ Dùng Interface khi:

- Định nghĩa object/class structure
- Cần Declaration Merging (khi làm việc với thư viện, API bên ngoài)
- Làm việc với OOP (class `implements` interface)

```ts
interface Shape {
  area(): number;
}

class Circle implements Shape {
  constructor(private radius: number) {}

  area() {
    return Math.PI * this.radius ** 2;
  }
}
```

### ✅ Dùng Type khi:

- Cần Union hoặc Intersection types
- Làm việc với primitives, tuples
- Cần Mapped Types, Conditional Types
- Muốn type chính xác, không bị merge ngoài ý muốn

```ts
type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: string };

type UserResponse = ApiResponse<User>;
```

> 💡 **Bổ sung — Conditional Types:** Đây là một tính năng mạnh khác chỉ `type` mới làm được, cho phép type "rẽ nhánh" tùy theo điều kiện, tương tự toán tử ba ngôi (`? :`) nhưng ở cấp độ kiểu dữ liệu:
>
> ```ts
> type IsString<T> = T extends string ? "yes" : "no";
>
> type A = IsString<string>; // "yes"
> type B = IsString<number>; // "no"
> ```
>
> Conditional Types thường kết hợp với `infer` để "trích xuất" kiểu dữ liệu con, ví dụ như cách TypeScript định nghĩa `ReturnType<T>` (lấy kiểu trả về của một hàm) trong thư viện chuẩn.

---

## 4. Best Practice

### Quy tắc đơn giản:

- **Interface** cho objects (đặc biệt là public API — ví dụ định nghĩa props của component, tham số của hàm, cấu trúc trả về của API)
- **Type** cho mọi thứ còn lại (unions, tuples, utilities, mapped/conditional types)

### Nhất quán trong dự án:

- Chọn một convention và giữ nguyên trong toàn bộ codebase — tránh tình trạng chỗ này dùng `interface`, chỗ kia dùng `type` cho cùng một mục đích, gây khó đọc và khó bảo trì.

---

## 5. Bảng tổng hợp nhanh

| Tính năng | `interface` | `type` |
|---|---|---|
| Định nghĩa shape của object | ✅ | ✅ |
| Declaration Merging (khai báo lại để gộp) | ✅ | ❌ (lỗi trùng tên) |
| Kế thừa (`extends`) | ✅ | ✅ (qua `&`) |
| Union types (`\|`) | ❌ | ✅ |
| Intersection types (`&`) | Gián tiếp qua `extends` | ✅ |
| Đặt tên cho primitives (string, number...) | ❌ | ✅ |
| Tuple | ❌ | ✅ |
| Mapped Types (`[P in keyof T]`) | ❌ | ✅ |
| Conditional Types (`T extends U ? X : Y`) | ❌ | ✅ |
| Dùng với `class implements` | ✅ (khuyến khích) | ✅ (vẫn được) |

### Ghi nhớ ngắn gọn:

> **Interface = mô tả hình dạng object, có thể mở rộng/gộp.**
> **Type = linh hoạt hơn, xử lý được mọi kiểu dữ liệu (union, tuple, mapped, conditional) nhưng không gộp được.**
