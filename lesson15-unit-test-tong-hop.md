# Unit Test - Tổng Hợp Kiến Thức Chi Tiết

> Tổng hợp từ buổi trao đổi trong session này. Toàn bộ ví dụ code lấy từ các file thực tế đã viết trong
> [tests/students-submission/tuyen-vo/lesson-6/](tests/students-submission/tuyen-vo/lesson-6/) — không dùng ví dụ ngoài.

---

## Mục lục

1. [Unit Test là gì?](#1-unit-test-là-gì)
2. [Vì sao cần Unit Test? So sánh với E2E Test](#2-vì-sao-cần-unit-test-so-sánh-với-e2e-test)
3. [Nguyên tắc F.I.R.S.T của Unit Test tốt](#3-nguyên-tắc-first-của-unit-test-tốt)
4. [Cấu trúc AAA: Arrange - Act - Assert](#4-cấu-trúc-aaa-arrange---act---assert)
5. [Ví dụ đầy đủ #1: Unit test cho class Order (tính tổng tiền)](#5-ví-dụ-đầy-đủ-1-unit-test-cho-class-order-tính-tổng-tiền)
6. [Ví dụ đầy đủ #2: Unit test cho class Customer](#6-ví-dụ-đầy-đủ-2-unit-test-cho-class-customer)
7. [Ví dụ đầy đủ #3: Unit test cho class Library (tìm kiếm)](#7-ví-dụ-đầy-đủ-3-unit-test-cho-class-library-tìm-kiếm)
8. [Testability: vì sao code cần "dễ test"](#8-testability-vì-sao-code-cần-dễ-test)
9. [Edge Case là gì? Vì sao quan trọng](#9-edge-case-là-gì-vì-sao-quan-trọng)
10. [Test Doubles: Mock, Stub, Spy, Fake](#10-test-doubles-mock-stub-spy-fake)
11. [Các assertion (matcher) thường dùng](#11-các-assertion-matcher-thường-dùng)
12. [Code Coverage là gì?](#12-code-coverage-là-gì)
13. [Unit Test dùng công cụ gì? Playwright Test hay Jest/Vitest?](#13-unit-test-dùng-công-cụ-gì-playwright-test-hay-jestvitest)
14. [Cách chạy Unit Test trong project này](#14-cách-chạy-unit-test-trong-project-này)
15. [Ai viết Unit Test và viết khi nào?](#15-ai-viết-unit-test-và-viết-khi-nào)
16. [Checklist viết Unit Test tốt](#16-checklist-viết-unit-test-tốt)

---

## 1. Unit Test là gì?

**Unit Test** là kiểm thử **1 đơn vị code nhỏ nhất** (thường là 1 hàm hoặc 1 method của 1 class) một cách **cô lập** — không phụ thuộc vào database thật, network thật, hay UI thật.

```
"Đơn vị" (Unit) = phần code nhỏ nhất có thể test độc lập
                 → thường là: 1 function, 1 method

"Cô lập" (Isolated) = không gọi ra bên ngoài (API, DB, file system...)
                     → chỉ test đúng logic bên trong hàm/class đó
```

### Ví dụ dễ hình dung

```typescript
// Đây là 1 "unit" — 1 hàm tính toán thuần tuý
function calculateDiscount(price: number, discountPercent: number): number {
    return price - (price * discountPercent / 100);
}

// Unit test cho hàm này — không cần mở browser, không cần gọi API
test('Giảm giá 10% cho sản phẩm 100.000đ phải ra 90.000đ', () => {
    expect(calculateDiscount(100_000, 10)).toBe(90_000);
});
```

---

## 2. Vì sao cần Unit Test? So sánh với E2E Test

Đây chính là **tầng đáy** của Testing Pyramid đã học — nhiều nhất, nhanh nhất, rẻ nhất:

```
              /\
             /E2E\          ← Ít nhất, chậm nhất, đắt nhất
            /------\
           /Integr- \
          /  ation   \
         /------------\
        /   Unit Test   \   ← Nhiều nhất, nhanh nhất, rẻ nhất  ★ Ở đây
       /------------------\
```

### Bảng so sánh trực tiếp Unit Test vs E2E Test

| Tiêu chí | Unit Test | E2E Test |
|---|---|---|
| Kiểm tra gì | 1 hàm/1 class riêng lẻ | Toàn bộ luồng qua UI thật |
| Cần mở browser? | Không | Có |
| Tốc độ | Rất nhanh (mili-giây) | Chậm (giây) |
| Độ ổn định | Rất ổn định, hiếm khi flaky | Dễ flaky (network, timing, UI đổi) |
| Ai viết | Dev, cùng lúc với code tính năng | QA/SDET, sau khi tính năng ổn định |
| Ví dụ thực tế | [`02-js-class-order.spec.ts`](tests/students-submission/tuyen-vo/lesson-6/02-js-class-order.spec.ts) | [`authentication.spec.ts`](tests/students-submission/tuyen-vo/lesson-08/authentication.spec.ts) |

### Bằng chứng thực tế: đo tốc độ chạy

Khi chạy 14 unit test cho `Order`, `Customer`, `Library` trong `lesson-6/`, tổng thời gian chỉ **4.7 giây cho toàn bộ 14 test** (bao gồm cả thời gian khởi động Playwright) — mỗi test riêng lẻ chỉ mất **9-26 mili-giây**. Trong khi đó, 1 E2E test login như `authentication.spec.ts` thường mất vài giây chỉ cho **1 test case** vì phải mở browser, load trang thật.

---

## 3. Nguyên tắc F.I.R.S.T của Unit Test tốt

| Chữ | Ý nghĩa | Giải thích |
|---|---|---|
| **F**ast | Nhanh | Chạy trong mili-giây, để dev chạy được hàng trăm/nghìn lần mỗi ngày |
| **I**ndependent | Độc lập | Test này không phụ thuộc kết quả của test khác, thứ tự chạy không quan trọng |
| **R**epeatable | Lặp lại được | Chạy ở máy nào, lúc nào cũng ra kết quả giống nhau (không phụ thuộc ngày giờ, mạng...) |
| **S**elf-Validating | Tự xác nhận | Test tự trả về pass/fail rõ ràng, không cần người đọc log để tự suy luận |
| **T**imely | Đúng lúc | Viết **cùng lúc hoặc ngay sau** khi viết code, không để dồn lại viết sau |

### Đối chiếu với ví dụ thực tế

Xét test này trong [`02-js-class-order.spec.ts`](tests/students-submission/tuyen-vo/lesson-6/02-js-class-order.spec.ts):

```typescript
test('Đơn hàng mới khởi tạo có tổng tiền = 0 và danh sách sản phẩm rỗng', () => {
    const order = new Order(2, 'Khách mới');

    expect(order.totalAmount).toBe(0);
    expect(order.items).toEqual([]);
});
```

- **Fast**: chạy trong 13ms
- **Independent**: tạo `Order` mới hoàn toàn trong test này, không dùng lại `order` từ test khác
- **Repeatable**: không phụ thuộc gì bên ngoài — chạy bao nhiêu lần cũng ra kết quả y hệt
- **Self-Validating**: `expect(...).toBe(0)` — pass/fail rõ ràng, không cần đọc console.log để đoán

---

## 4. Cấu trúc AAA: Arrange - Act - Assert

Hầu hết unit test tốt đều theo 3 bước rõ ràng:

```
Arrange (Chuẩn bị)  → Tạo dữ liệu/đối tượng cần thiết cho test
Act     (Hành động) → Gọi hàm/method cần test
Assert  (Xác nhận)  → Kiểm tra kết quả có đúng như mong đợi không
```

### Ví dụ minh hoạ rõ 3 bước — từ [`02-js-class-crm.spec.ts`](tests/students-submission/tuyen-vo/lesson-6/02-js-class-crm.spec.ts)

```typescript
test('updateEmail cập nhật đúng email mới', () => {
    // ── Arrange ──
    const customer = new Customer(1, 'Tuyền', 'tuyen@gmail.com', '012345678');

    // ── Act ──
    customer.updateEmail('tuyen123@gmail.com');

    // ── Assert ──
    expect(customer.email).toBe('tuyen123@gmail.com');
});
```

> **Mẹo nhận biết test "tốt"**: nếu nhìn vào 1 test mà không phân biệt được đâu là Arrange, đâu là Act, đâu là Assert — test đó có thể đang làm quá nhiều việc, nên tách nhỏ ra.

---

## 5. Ví dụ đầy đủ #1: Unit test cho class Order (tính tổng tiền)

### Code cần test — logic gốc từ `02-js-class-order.ts`

```typescript
interface Item {
    name: string;
    price: number;
    amount: number;
    discount: number;
}

class Order {
    orderID: number;
    customerName: string;
    items: Item[];
    totalAmount: number;

    constructor(orderID: number, customerName: string) {
        this.orderID = orderID;
        this.customerName = customerName;
        this.items = [];
        this.totalAmount = 0;
    }

    addItems(item: Item): void {
        this.items.push(item);
        this.calculateTotal();
    }

    calculateTotal() {
        let total = 0;
        this.items.forEach((item) => {
            let tt = item.price * item.amount - item.discount;
            total += tt;
        });
        this.totalAmount = total;
    }
}
```

### Toàn bộ unit test — [`02-js-class-order.spec.ts`](tests/students-submission/tuyen-vo/lesson-6/02-js-class-order.spec.ts)

```typescript
test.describe('Order - calculateTotal', () => {
    test('Tính đúng tổng tiền khi thêm 1 sản phẩm', () => {
        const order = new Order(1, 'Tuyen');
        const item: Item = { name: 'Sản phẩm 1', price: 10_000, amount: 2, discount: 5_000 };

        order.addItems(item);

        // 10_000 * 2 - 5_000 = 15_000
        expect(order.totalAmount).toBe(15_000);
    });

    test('Tự động cộng dồn tổng tiền khi thêm nhiều sản phẩm', () => {
        const order = new Order(1, 'Tuyen');
        order.addItems({ name: 'SP1', price: 10_000, amount: 2, discount: 5_000 }); // 15_000
        order.addItems({ name: 'SP2', price: 30_000, amount: 5, discount: 15_000 }); // 135_000

        expect(order.totalAmount).toBe(150_000);
        expect(order.items.length).toBe(2);
    });

    test('Đơn hàng mới khởi tạo có tổng tiền = 0 và danh sách sản phẩm rỗng', () => {
        const order = new Order(2, 'Khách mới');

        expect(order.totalAmount).toBe(0);
        expect(order.items).toEqual([]);
    });

    test('Discount lớn hơn price*amount làm tổng tiền âm (kiểm tra edge case)', () => {
        const order = new Order(3, 'Khách VIP');
        order.addItems({ name: 'Sản phẩm khuyến mãi', price: 1_000, amount: 1, discount: 5_000 });

        expect(order.totalAmount).toBe(-4_000);
    });
});
```

### Phân tích từng test case

| Test case | Kiểm tra khía cạnh gì |
|---|---|
| Tính đúng tổng tiền khi thêm 1 sản phẩm | **Happy path** cơ bản nhất — trường hợp đúng như mong đợi |
| Tự động cộng dồn khi thêm nhiều sản phẩm | Kiểm tra `calculateTotal()` chạy lại đúng mỗi lần `addItems()` được gọi |
| Đơn hàng mới khởi tạo | Kiểm tra **giá trị mặc định/ban đầu** của constructor |
| Discount > price*amount | **Edge case** — dữ liệu bất thường, phát hiện được nếu logic có lỗ hổng (VD: cần chặn discount âm) |

---

## 6. Ví dụ đầy đủ #2: Unit test cho class Customer

### Code cần test — logic gốc từ `02-js-class-crm.ts`

```typescript
class Customer {
    id: number;
    name: string;
    email: string;
    phone: string;

    constructor(id: number, name: string, email: string, phone: string) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.phone = phone;
    }

    updateEmail(newEmail: string) {
        this.email = newEmail;
    }
}
```

### Toàn bộ unit test — [`02-js-class-crm.spec.ts`](tests/students-submission/tuyen-vo/lesson-6/02-js-class-crm.spec.ts)

```typescript
test.describe('Customer - updateEmail', () => {
    test('updateEmail cập nhật đúng email mới', () => {
        const customer = new Customer(1, 'Tuyền', 'tuyen@gmail.com', '012345678');

        customer.updateEmail('tuyen123@gmail.com');

        expect(customer.email).toBe('tuyen123@gmail.com');
    });

    test('updateEmail không làm thay đổi các field khác', () => {
        const customer = new Customer(1, 'Tuyền', 'tuyen@gmail.com', '012345678');

        customer.updateEmail('tuyen123@gmail.com');

        expect(customer.name).toBe('Tuyền');
        expect(customer.id).toBe(1);
        expect(customer.phone).toBe('012345678');
    });

    test('Customer khởi tạo đúng các giá trị ban đầu', () => {
        const customer = new Customer(2, 'An', 'an@gmail.com', '098765432');

        expect(customer.id).toBe(2);
        expect(customer.name).toBe('An');
        expect(customer.email).toBe('an@gmail.com');
        expect(customer.phone).toBe('098765432');
    });
});
```

> **Điểm hay của test "updateEmail không làm thay đổi field khác"**: đây là 1 dạng test hay bị bỏ sót — nhiều người chỉ test "cái vừa thay đổi có đúng không" mà quên kiểm tra **các phần không nên bị ảnh hưởng** có bị ảnh hưởng nhầm hay không (side-effect ngoài ý muốn).

---

## 7. Ví dụ đầy đủ #3: Unit test cho class Library (tìm kiếm)

### Vấn đề gặp phải với code gốc — và cách unit test "ép" code phải tốt hơn

Code gốc trong `02-js-class-library.ts`:

```typescript
findBook(title: string): void {
    const book = this.books.filter(book => book.title === title)
    console.log('Sách cần tìm theo tiêu đề chính xác:', book);
}
```

Hàm này **không `return` gì cả** — chỉ `console.log()`. Với code như vậy, **không thể viết unit test được**, vì test cần có giá trị trả về để `expect()` so sánh. Đây là ví dụ thực tế cho khái niệm **Testability** (mục 8): code khó/không test được thường là dấu hiệu code cần cải thiện.

### Bản chỉnh sửa dùng cho unit test (return thay vì console.log)

```typescript
interface Book {
    title: string;
    author: string;
    year: number;
}

class Library {
    name: string;
    location: string;
    books: Book[];

    constructor(name: string, location: string) {
        this.name = name;
        this.location = location;
        this.books = [];
    }

    addBook(book: Book): void {
        this.books.push(book);
    }

    findBook(title: string): Book | undefined {
        return this.books.find(book => book.title === title);
    }

    findSimilar(title: string): Book[] {
        return this.books.filter(book => book.title.toLowerCase().includes(title.toLowerCase()));
    }
}
```

### Toàn bộ unit test — [`02-js-class-library.spec.ts`](tests/students-submission/tuyen-vo/lesson-6/02-js-class-library.spec.ts)

```typescript
function createLibraryWithBooks(): Library {
    const library = new Library('Thư viện Đại học quốc gia TPHCM', 'Hồ Chí Minh');
    library.addBook({ title: 'Mắt biếc 1', author: 'Nguyễn Nhật Ánh', year: 1990 });
    library.addBook({ title: 'Đất rừng Phương Nam', author: 'Đoàn Giỏi', year: 1957 });
    library.addBook({ title: 'Chí Phèo', author: 'Nam Cao', year: 1941 });
    library.addBook({ title: 'mắt biếc 2', author: 'Nguyễn Nhật Ánh', year: 1990 });
    return library;
}

test.describe('Library - findBook', () => {
    test('Tìm đúng sách theo tiêu đề chính xác', () => {
        const library = createLibraryWithBooks();
        const result = library.findBook('mắt biếc 2');

        expect(result?.author).toBe('Nguyễn Nhật Ánh');
        expect(result?.year).toBe(1990);
    });

    test('Trả về undefined nếu không tìm thấy sách', () => {
        const library = createLibraryWithBooks();
        expect(library.findBook('Sách không tồn tại')).toBeUndefined();
    });
});

test.describe('Library - findSimilar', () => {
    test('Tìm được nhiều sách chứa từ khoá, không phân biệt hoa/thường', () => {
        const library = createLibraryWithBooks();
        const results = library.findSimilar('Mắt'); // chữ M hoa, nhưng có sách "mắt biếc 2" chữ thường

        expect(results.length).toBe(2);
        expect(results.map(b => b.title)).toEqual(
            expect.arrayContaining(['Mắt biếc 1', 'mắt biếc 2'])
        );
    });

    test('Trả về mảng rỗng nếu không có sách nào khớp từ khoá', () => {
        const library = createLibraryWithBooks();
        expect(library.findSimilar('Không tồn tại')).toEqual([]);
    });

    test('addBook thêm đúng sách vào danh sách', () => {
        const library = new Library('Thư viện Quận 1', 'Hồ Chí Minh');
        library.addBook({ title: 'Dế Mèn Phiêu Lưu Ký', author: 'Tô Hoài', year: 1941 });

        expect(library.books.length).toBe(1);
        expect(library.books[0].title).toBe('Dế Mèn Phiêu Lưu Ký');
    });
});
```

### Kỹ thuật đáng chú ý: hàm `createLibraryWithBooks()`

Đây là 1 **helper function** dùng để dựng sẵn dữ liệu test (test data builder) — tránh lặp lại 4 dòng `addBook()` ở mọi test case. Đây chính là tinh thần "tái sử dụng code" đã học ở phần Framework, áp dụng ngay trong unit test.

---

## 8. Testability: vì sao code cần "dễ test"

**Testability** = mức độ dễ dàng để viết test cho 1 đoạn code.

### Bài học rút ra từ `Library.findBook()`

| Cách viết | Có test được không? | Vì sao |
|---|---|---|
| `console.log(book)` | ❌ Không | Không có gì để `expect()` so sánh — kết quả chỉ nằm trên màn hình |
| `return book` | ✅ Có | Test gọi hàm, nhận giá trị trả về, so sánh bằng `expect()` |

### Nguyên tắc chung để code dễ test hơn

1. **Return giá trị thay vì chỉ in ra màn hình** — như ví dụ `findBook()` ở trên
2. **Hàm càng "thuần" (pure function) càng dễ test** — cùng input luôn ra cùng output, không phụ thuộc biến ngoài, không gây side-effect
3. **Tách logic tính toán khỏi logic hiển thị/I/O** — VD: tách riêng hàm `calculateTotal()` khỏi việc `console.log` kết quả
4. **Constructor/hàm không nên tự gọi network/DB bên trong** — nếu cần, nên "tiêm" (inject) dependency từ ngoài vào để dễ thay thế bằng mock khi test

> Đây cũng là lý do vì sao unit test không chỉ giúp "phát hiện bug" mà còn **ép code được viết theo hướng rõ ràng, dễ đọc, dễ tái sử dụng hơn** — một tác dụng phụ rất có giá trị.

---

## 9. Edge Case là gì? Vì sao quan trọng

**Edge Case** = trường hợp dữ liệu đầu vào **bất thường, biên, hoặc hiếm gặp** — không phải luồng "đẹp" thông thường (happy path).

### Ví dụ edge case đã viết trong project

```typescript
test('Discount lớn hơn price*amount làm tổng tiền âm (kiểm tra edge case)', () => {
    const order = new Order(3, 'Khách VIP');
    order.addItems({ name: 'Sản phẩm khuyến mãi', price: 1_000, amount: 1, discount: 5_000 });

    expect(order.totalAmount).toBe(-4_000);
});
```

Test này phát hiện ra 1 vấn đề tiềm ẩn: **code hiện tại cho phép tổng tiền bị âm** nếu discount lớn hơn giá trị sản phẩm. Đây có thể là bug thực sự (đơn hàng không nên có tổng tiền âm) — nhưng nếu không có unit test này, vấn đề có thể không ai phát hiện ra cho tới khi có khách hàng thật gặp phải.

### Các loại edge case thường gặp cần nghĩ tới khi viết unit test

| Loại | Ví dụ |
|---|---|
| Giá trị rỗng/trống | Mảng rỗng, chuỗi rỗng `""`, `null`, `undefined` |
| Giá trị biên | 0, số âm, số cực lớn |
| Không tìm thấy | `findBook('Không tồn tại')` → phải trả `undefined`, không được crash |
| Trùng lặp | 2 sản phẩm giống hệt nhau trong đơn hàng |
| Sai định dạng | Email sai định dạng, số điện thoại có ký tự lạ |
| Thứ tự | Thêm item theo thứ tự khác nhau có ra kết quả giống nhau không |

---

## 10. Test Doubles: Mock, Stub, Spy, Fake

Khi 1 hàm cần gọi ra thứ gì đó bên ngoài (API, database, thời gian hệ thống...), unit test **không gọi cái thật** — mà thay bằng 1 phiên bản giả gọi là **Test Double**.

| Loại | Định nghĩa | Ví dụ |
|---|---|---|
| **Stub** | Trả về giá trị giả cố định, không quan tâm được gọi thế nào | Giả lập `getCurrentDate()` luôn trả về `"2026-07-19"` để test không phụ thuộc ngày thực tế |
| **Mock** | Giống Stub nhưng còn **kiểm tra xem hàm có được gọi đúng cách không** (gọi mấy lần, với tham số gì) | Kiểm tra `sendEmail()` có được gọi đúng 1 lần với đúng địa chỉ email không |
| **Spy** | "Theo dõi" lời gọi thật, nhưng vẫn cho chạy logic thật | Ghi lại `console.log` có được gọi hay không, nhưng vẫn log ra bình thường |
| **Fake** | Cài đặt giả nhưng có logic hoạt động thật (đơn giản hơn bản thật) | Dùng 1 mảng JavaScript giả lập database thay vì kết nối DB thật |

### Vì sao 3 ví dụ Order/Customer/Library ở trên KHÔNG cần Test Double?

Vì cả 3 class này **không gọi ra bên ngoài** — không gọi API, không đọc file, không dùng `Date.now()`. Mọi thứ chỉ là tính toán/xử lý dữ liệu trong bộ nhớ → đây là lý do chúng rất dễ unit test mà không cần mock gì cả. Nếu `Order` có thêm hàm gọi API để lưu đơn hàng, lúc đó mới cần mock phần gọi API đó.

---

## 11. Các assertion (matcher) thường dùng

Dự án dùng `expect()` của `@playwright/test` — các matcher phổ biến nhất đã dùng trong 3 file test ở trên:

| Matcher | Ý nghĩa | Ví dụ đã dùng |
|---|---|---|
| `.toBe(value)` | So sánh giá trị **chính xác** (giống `===`) | `expect(order.totalAmount).toBe(15_000)` |
| `.toEqual(value)` | So sánh **object/mảng theo giá trị** (deep equal) | `expect(order.items).toEqual([])` |
| `.toBeUndefined()` | Kiểm tra giá trị là `undefined` | `expect(library.findBook('...')).toBeUndefined()` |
| `.toBeTruthy()` / `.toBeFalsy()` | Kiểm tra giá trị đúng/sai theo kiểu boolean lỏng | `expect(token).toBeTruthy()` |
| `expect.arrayContaining([...])` | Mảng có chứa các phần tử chỉ định (không cần đúng thứ tự) | `expect(results.map(b => b.title)).toEqual(expect.arrayContaining([...]))` |

> **Phân biệt `.toBe()` vs `.toEqual()`**: `.toBe()` dùng cho giá trị nguyên thuỷ (number, string, boolean); `.toEqual()` dùng cho object/mảng vì `.toBe()` sẽ so sánh theo tham chiếu bộ nhớ (2 mảng `[]` khác nhau về tham chiếu dù giá trị giống hệt).

---

## 12. Code Coverage là gì?

**Code Coverage** = tỷ lệ phần trăm dòng code (hoặc nhánh logic) **đã được chạy qua** bởi các unit test.

```
Code Coverage = (Số dòng code được test chạy qua / Tổng số dòng code) × 100%
```

Ví dụ: class `Order` có 3 method (`constructor`, `addItems`, `calculateTotal`). Nếu unit test chỉ gọi `addItems()` (mà bên trong nó tự gọi `calculateTotal()`) nhưng có 1 nhánh `if` trong `calculateTotal()` chưa từng được test chạm tới → coverage cho nhánh đó là 0%, dù tổng thể coverage của cả file có thể vẫn cao.

> **Lưu ý quan trọng**: Coverage cao (VD: 90%) **không đồng nghĩa với chất lượng test tốt** — có thể code chạy qua nhưng không có `expect()` nào kiểm tra kết quả đúng hay sai. Coverage chỉ nên dùng để **phát hiện phần code CHƯA được test chạm tới**, không nên dùng làm mục tiêu duy nhất.

---

## 13. Unit Test dùng công cụ gì? Playwright Test hay Jest/Vitest?

| Công cụ | Khi nào phù hợp |
|---|---|
| **Jest / Vitest** | Lựa chọn phổ biến nhất cho unit test thuần tuý trong hệ sinh thái JS/TS — nhanh, nhiều tính năng mock sẵn có |
| **`@playwright/test`** (đang dùng trong project) | Vẫn chạy tốt cho unit test nếu **không dùng fixture `page`/`browser`** — như 3 file ví dụ ở trên. Ưu điểm: không cần cài thêm dependency, dùng chung 1 test runner cho cả Unit + API + E2E test |

### Vì sao 3 file ví dụ trong project này dùng `@playwright/test` mà không phải Jest?

Vì project đã cài sẵn `@playwright/test` (dùng cho E2E test) — tận dụng lại đúng `test`/`expect` đó để viết unit test, tránh phải cài thêm Jest/Vitest chỉ để làm 1 việc tương tự. Cách viết `test()`, `test.describe()`, `expect()` giống hệt nhau — chỉ khác là **không khai báo fixture `page`** trong tham số callback.

```typescript
// E2E test — cần fixture `page` để mở browser
test('Login', async ({ page }) => { await page.goto('...'); });

// Unit test — không cần fixture nào, cũng không cần async nếu logic đồng bộ
test('Tính tổng tiền', () => { expect(order.totalAmount).toBe(15_000); });
```

---

## 14. Cách chạy Unit Test trong project này

```bash
# Chạy riêng 3 file unit test vừa tạo
npx playwright test "tests/students-submission/tuyen-vo/lesson-6/02-js-class-order.spec.ts" \
  "tests/students-submission/tuyen-vo/lesson-6/02-js-class-crm.spec.ts" \
  "tests/students-submission/tuyen-vo/lesson-6/02-js-class-library.spec.ts" \
  --project=chromium --reporter=list
```

Kết quả thực tế khi chạy (đã verify):

```
14 passed (4.7s)
```

> Dùng `--project=chromium` dù unit test không cần browser — vì trong [playwright.config.ts](playwright.config.ts) của project, mọi test (kể cả unit test không dùng `page`) vẫn chạy dưới 1 "project" đã khai báo. Điều này không ảnh hưởng tốc độ vì Playwright chỉ khởi động browser **khi fixture `page`/`context`/`browser` thực sự được dùng** trong test.

---

## 15. Ai viết Unit Test và viết khi nào?

Nhắc lại bảng phân vai đã thảo luận trong session:

| Loại test | Ai viết | Viết khi nào |
|---|---|---|
| **Unit test** | **Dev** | **Cùng lúc** với code tính năng — không đợi tính năng ổn định |
| Manual test | QA/QC | Ngay khi tính năng vừa xong, trước khi release |
| Automation/E2E test | QA/SDET | Sau khi tính năng đã manual test pass, ổn định |

**Vì sao Unit test viết được ngay, không cần đợi ổn định như E2E?**

Vì unit test không phụ thuộc UI — dù UI/flow nghiệp vụ có thay đổi liên tục, logic tính toán cốt lõi bên trong (như `calculateTotal()`) thường **ổn định hơn nhiều** so với giao diện. Viết unit test sớm giúp dev tự tin refactor code mà không sợ làm hỏng logic đã đúng trước đó.

---

## 16. Checklist viết Unit Test tốt

1. ☐ Tên test mô tả rõ **hành vi được kiểm tra**, không chỉ tên hàm (VD: "Discount lớn hơn price*amount làm tổng tiền âm" thay vì "test calculateTotal 2")
2. ☐ Mỗi test chỉ kiểm tra **1 hành vi** — nếu 1 test có nhiều `expect()` không liên quan, nên tách ra
3. ☐ Tuân theo cấu trúc **Arrange - Act - Assert** rõ ràng
4. ☐ Có test cho **happy path** (trường hợp đúng bình thường)
5. ☐ Có test cho **edge case** (rỗng, số 0, âm, không tìm thấy...)
6. ☐ Test **độc lập** — không phụ thuộc thứ tự chạy hay kết quả của test khác
7. ☐ Không dùng dữ liệu/thời gian thực tế thay đổi được (VD: `Date.now()`) trừ khi đã mock
8. ☐ Đặt tên biến/helper rõ ràng (VD: `createLibraryWithBooks()`) để test dễ đọc, tránh lặp code dựng dữ liệu

---

*Tài liệu tổng hợp toàn bộ kiến thức Unit Test trao đổi trong session, với ví dụ đã được chạy thực tế và xác nhận pass (14/14) trên các file trong [tests/students-submission/tuyen-vo/lesson-6/](tests/students-submission/tuyen-vo/lesson-6/).*
