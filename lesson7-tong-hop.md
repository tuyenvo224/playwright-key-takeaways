# Lesson 7 - Selector Advanced (XPath Axes & Advanced Methods)

## Nội dung buổi học
1. **DOM** - Mối quan hệ giữa các node
2. **XPath Axes** - Điều hướng theo mối quan hệ DOM
3. **XPath Advanced Methods** - Các hàm nâng cao

---

## PHẦN 1: DOM - Mối quan hệ giữa các Node

### 1.1 DOM là gì?

DOM (Document Object Model) mô tả trang HTML dưới dạng **cây (tree)**, mỗi phần tử HTML là một **node**.

```
         html
        /    \
     head    body
              |
             div
           / | \
          h1 form iframe
              |
             div
```

> Hiểu mối quan hệ giữa các node giúp bạn viết XPath chính xác và linh hoạt hơn.

---

### 1.2 Quy ước màu sắc trong bài

| Màu | Ý nghĩa |
|-----|---------|
| Xanh dương | Node gốc (root) |
| Xanh lá | Node hiện tại (node đang xét) |
| Cam | Node cần chú ý |

---

### 1.3 Các loại mối quan hệ

#### `self` - Bản thân
Node đang xét.

```
node gốc
   └── node hiện tại  ← đây chính là self
```

---

#### `parent` - Cha (phía trên trực tiếp)
Node **nằm trên trực tiếp** của node hiện tại (chỉ 1 cấp).

```
node gốc
   ├── node 1
   ├── node cha   ← parent
   └── node 2
        └── node hiện tại
```

---

#### `children` - Con (phía dưới trực tiếp)
Các node **nằm dưới trực tiếp** của node hiện tại (chỉ 1 cấp).

```
node hiện tại
   ├── con 1  ← children
   ├── con 2  ← children
   └── con 3  ← children
```

---

#### `ancestor` - Tổ tiên
Gồm **tất cả các node phía trên**: cha, ông, cụ, ...

```
node gốc   ← ancestor
   └── node cụ   ← ancestor
        └── node ông   ← ancestor
             └── node cha   ← ancestor
                  └── node hiện tại
```

---

#### `descendant` - Hậu duệ
Gồm **tất cả node phía dưới**: con, cháu, chắt, ...

```
node hiện tại
   ├── node con   ← descendant
   ├── node 3     ← descendant
   └── node 4     ← descendant
        ├── node 5    ← descendant
        ├── node cháu ← descendant
        └── node 6    ← descendant
```

---

#### `sibling` - Anh em
Các node **cùng cấp và cùng cha**.

```
node gốc
   ├── sibling đứng trước  ← preceding-sibling
   ├── node hiện tại
   └── sibling đứng sau    ← following-sibling
```

---

#### `following` - Theo sau
Các node **ở phía bên phải (sau) trong document**, **KHÔNG bao gồm con** của node hiện tại.

```
node hiện tại
├── (các con của node hiện tại - KHÔNG lấy)
table   ← following
iframe  ← following
   ├── thead  ← following
   └── tbody  ← following
```

---

#### `preceding` - Phía trước
Các node **ở phía bên trái (trước) trong document**, **trừ các node ancestor**.

```
div    ← preceding
├── h1 ← preceding
├── h2 ← preceding
└── h3 ← preceding
     └── node hiện tại
```

---

#### `following-sibling` - Anh em phía sau
= `following` + `sibling` → Các anh em **cùng cấp, cùng cha** đứng **sau** node hiện tại.

```
node gốc
├── node hiện tại
├── node 1  ← following-sibling
├── node 2  ← following-sibling
└── node 3  ← following-sibling
```

---

#### `preceding-sibling` - Anh em phía trước
= `preceding` + `sibling` → Các anh em **cùng cấp, cùng cha** đứng **trước** node hiện tại.

```
node gốc
├── node 1  ← preceding-sibling
├── node 2  ← preceding-sibling
└── node hiện tại
```

---

## PHẦN 2: XPath Axes (Phương thức trục XPath)

### 2.1 Định nghĩa

**XPath axes methods** là các phương pháp để **điều hướng** và **chọn node** trong cây DOM XML/HTML dựa trên **mối quan hệ** giữa các node.

**Cú pháp chung:**
```
//tag/relationship::tagname[@attr='value']
```

**Ví dụ:**
```xpath
//form[@id='test-form']/child::button
//button[text()='Create Test Case']/parent::form
//button[@class='btn-edit']/ancestor::table
```

**Công dụng:**
- Tìm element dựa trên **vị trí tương đối** (parent, child, sibling, ancestor...)
- **Linh hoạt** hơn việc dùng đường dẫn tuyệt đối hoặc tương đối thuần túy

---

### 2.2 Danh sách các Axes

| Axis | Ý nghĩa | Ví dụ thực tế |
|------|---------|--------------|
| `child` | Con trực tiếp | `//form[@id='test-form']/child::button` |
| `descendant` | Tất cả con cháu | `//form[@id='test-form']/descendant::input` |
| `parent` | Cha trực tiếp | `//button[text()='Create Test Case']/parent::form` |
| `ancestor` | Tổ tiên (cha, ông,...) | `//button[@class='btn-edit']/ancestor::table` |
| `following-sibling` | Anh em phía sau | `//label[@for='testName']/following-sibling::input` |
| `preceding-sibling` | Anh em phía trước | `//button[@class='btn-reset']/preceding-sibling::button` |
| `following` | Tất cả node sau trong document | `//h2[text()='Test Cases List']/following::button[@class='btn-run']` |
| `preceding` | Tất cả node trước trong document | `//h2[text()='Test Execution Results']/preceding::td[@class='priority-high']` |
| `ancestor-or-self` | Tổ tiên hoặc chính nó | `//table[@id='test-table']/ancestor-or-self::span[contains(@class,'status')]` |
| `descendant-or-self` | Con cháu hoặc chính nó | `//table[@id='test-table']/descendant-or-self::span[contains(@class,'status')]` |

---

### 2.3 Chi tiết từng Axis với ví dụ

#### `child` - Con trực tiếp

```xpath
# Tìm tất cả button con trực tiếp của form
//form[@id='test-form']/child::button

# Kết quả: button "Create Test Case" và button "Reset Form"
```

> **Khác biệt với `//`**: `child::` chỉ lấy con **trực tiếp** (1 cấp), không lấy cháu.

---

#### `descendant` - Tất cả con cháu

```xpath
# Tìm tất cả input bên trong form (mọi cấp)
//form[@id='test-form']/descendant::input

# Kết quả: input testName, input priority (nằm sâu trong div.form-group)
```

---

#### `parent` - Tìm cha

```xpath
# Từ button "Create Test Case", tìm form cha
//button[text()='Create Test Case']/parent::form

# Kết quả: form#test-form
```

---

#### `ancestor` - Tìm tổ tiên

```xpath
# Từ button "Edit" trong table, tìm table tổ tiên
//button[@class='btn-edit']/ancestor::table

# Kết quả: table#test-table
```

---

#### `following-sibling` - Anh em phía sau

```xpath
# Từ label "Test Case Name", tìm input cùng cấp ngay sau nó
//label[@for='testName']/following-sibling::input
# Kết quả: input#testName

# Từ cột "Test Name" có text "Login Validation", lấy các cột tiếp theo
//td[text()='Login Validation']/following-sibling::td
# Kết quả: cột Type, Priority, Status, Actions
```

---

#### `preceding-sibling` - Anh em phía trước

```xpath
# Từ button "Reset Form", tìm button đứng trước nó
//button[@class='btn-reset']/preceding-sibling::button

# Kết quả: button "Create Test Case"
```

---

#### `following` - Tất cả node sau trong document

```xpath
# Từ h2 "Test Cases List", tìm tất cả button "Run Test" phía sau
//h2[text()='Test Cases List']/following::button[@class='btn-run']

# Kết quả: Tất cả 5 button "Run Test" trong bảng
```

> **Lưu ý**: `following` lấy tất cả node sau node hiện tại, **KHÔNG** lấy con của node hiện tại.

---

#### `preceding` - Tất cả node trước trong document

```xpath
# Từ h2 "Test Execution Results", tìm tất cả td có text "High" phía trước
//h2[text()='Test Execution Results']/preceding::td[@class='priority-high']

# Kết quả: TC001 và TC003 priority cells
```

---

#### `ancestor-or-self` - Tổ tiên hoặc chính nó

```xpath
# Tìm tất cả span status trong table (bao gồm cả chính nó nếu là span)
//table[@id='test-table']/ancestor-or-self::span[contains(@class, 'status')]

# Kết quả: Tất cả span status-passed, status-running, status-failed, status-pending
```

---

#### `descendant-or-self` - Con cháu hoặc chính nó

```xpath
# Tìm tất cả span status trong table (bao gồm cả chính nó nếu là span)
//table[@id='test-table']/descendant-or-self::span[contains(@class, 'status')]

# Kết quả: Tất cả span status-passed, status-running, status-failed, status-pending
```

---

## PHẦN 3: XPath Advanced Methods (Hàm nâng cao)

### 3.1 Wildcard `*`

`*` khớp với **tất cả các loại thẻ**.

```xpath
//div        # khớp thẻ div
//*          # khớp tất cả các loại thẻ

# Ứng dụng với axes:
//label[@for='username']/ancestor::*   # Tìm tất cả tổ tiên (5 results)
```

---

### 3.2 `@attribute` - Thuộc tính

Dùng `@` để truy cập thuộc tính của element.

```xpath
//tagname[@attribute='value']

# Ví dụ:
//input[@type='text']
//div[@class='form-group']
//form[@id='registrationForm']
//*[@id='registrationForm']   # Wildcard + attribute
```

---

### 3.3 `text()` - Lấy text bên trong element

`text()` lấy **text node trực tiếp** của element (khớp chính xác).

```xpath
//element[text()='exact text']

# Ví dụ:
//button[text()='Create Test Case']
//h2[text()='Test Cases List']
//td[text()='Login Validation']
```

> **Chú ý**: `text()` khớp **chính xác** toàn bộ chuỗi. Nếu text có khoảng trắng thừa, dùng `normalize-space()`.

---

### 3.4 `contains()` - Kiểm tra chứa chuỗi con

Tìm element có chứa **một phần text**, không cần khớp chính xác.

```xpath
//element[contains(@attribute, 'substring')]
//element[contains(text(), 'substring')]

# Ví dụ:
//span[contains(@class, 'status')]          # class chứa "status"
//button[contains(text(), 'Test')]          # text chứa "Test"
//div[contains(@class, 'form-group')]
//input[contains(@id, 'user')]              # id chứa "user"
```

**So sánh `text()` vs `contains()`:**

| | `text()='Login'` | `contains(text(), 'Login')` |
|-|-----------------|----------------------------|
| Khớp | "Login" (chính xác) | "Login Validation", "Login Page",... |
| Dùng khi | Biết text chính xác | Text có thể thay đổi một phần |

---

### 3.5 `normalize-space()` - Chuẩn hóa khoảng trắng

Loại bỏ khoảng trắng thừa ở **đầu, cuối và giữa** text.

```xpath
normalize-space(string)

# Ví dụ:
//element[normalize-space(text())='Login']
# Sẽ khớp với: "  Login  ", " Login", "Login "
```

> Dùng khi text trên UI có khoảng trắng không đồng nhất.

---

### 3.6 `starts-with()` - Bắt đầu bằng

Tìm element có text hoặc attribute **bắt đầu bằng** chuỗi nhất định.

```xpath
//element[starts-with(@attribute, 'prefix')]
//element[starts-with(text(), 'prefix')]

# Ví dụ:
//div[starts-with(@id, 'form')]         # id bắt đầu bằng "form"
//span[starts-with(@class, 'status')]   # class bắt đầu bằng "status"
//h1[starts-with(text(), 'User')]       # text bắt đầu bằng "User"
```

---

### 3.7 `not()` - Phủ định

Tìm element **KHÔNG** thỏa mãn điều kiện.

```xpath
//element[not(condition)]

# Ví dụ:
//h1[not(starts-with(text(), 'User'))]   # h1 KHÔNG bắt đầu bằng "User"
//input[not(@type='hidden')]             # input KHÔNG phải type hidden
//div[not(@class='form-group')]          # div KHÔNG có class form-group
```

---

### 3.8 `and` và `or` - Kết hợp điều kiện

```xpath
# AND - Tất cả điều kiện phải đúng
//element[@condition1 and @condition2]

# OR - Một trong các điều kiện đúng
//element[@condition1 or @condition2]

# Ví dụ:
//input[@type='text' and @required]
//button[@class='btn-submit' or @class='btn-reset']
//div[@class='form-group' and contains(@id, 'parent')]
```

---

## PHẦN 4: TỔNG HỢP NHANH

### Cú pháp XPath Axes

```
//tag/axis::tagname[@attr='value']
```

### Bảng tổng hợp tất cả

| Khái niệm | Cú pháp | Ý nghĩa |
|-----------|---------|---------|
| **DOM Relations** | | |
| self | - | Node hiện tại |
| parent | `/parent::tag` | Cha trực tiếp |
| children | `/child::tag` | Con trực tiếp |
| ancestor | `/ancestor::tag` | Tổ tiên (cha, ông,...) |
| descendant | `/descendant::tag` | Con cháu (mọi cấp) |
| sibling | - | Anh em cùng cấp |
| following-sibling | `/following-sibling::tag` | Anh em phía sau |
| preceding-sibling | `/preceding-sibling::tag` | Anh em phía trước |
| following | `/following::tag` | Tất cả node sau |
| preceding | `/preceding::tag` | Tất cả node trước |
| **XPath Functions** | | |
| Wildcard | `//*` | Khớp mọi thẻ |
| Attribute | `[@attr='val']` | Lọc theo thuộc tính |
| text() | `[text()='text']` | Khớp text chính xác |
| contains() | `[contains(@attr,'sub')]` | Chứa chuỗi con |
| normalize-space() | `[normalize-space()='text']` | Bỏ khoảng trắng thừa |
| starts-with() | `[starts-with(@attr,'pre')]` | Bắt đầu bằng |
| not() | `[not(condition)]` | Phủ định điều kiện |
| and / or | `[@a and @b]` | Kết hợp điều kiện |

---

### Ví dụ thực hành tổng hợp

HTML mẫu:
```html
<form id="registrationForm">
  <div class="form-group" id="parent">
    <label for="username" id="preceding">Username:</label>
    <input type="text" id="username" name="username" required>
  </div>
  <div class="form-group" id="child">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
  </div>
</form>
```

```xpath
# Tìm form theo id
//*[@id='registrationForm']

# Tìm div con trực tiếp của form
//form[@id='registrationForm']/child::div

# Tìm tất cả input bên trong form (mọi cấp)
//form[@id='registrationForm']/descendant::input

# Từ label username, tìm input kế bên (following-sibling)
//label[@for='username']/following-sibling::input

# Từ input username, tìm form tổ tiên
//input[@id='username']/ancestor::form

# Từ input username, tìm label của email (following)
//input[@id='username']/following::label[@for='email']

# Tìm tất cả tổ tiên của label username (* = mọi thẻ)
//label[@for='username']/ancestor::*

# Kết hợp axes phức tạp
//input[@id='username']/ancestor::form/descendant-or-self::*[@id='child']/label
```
