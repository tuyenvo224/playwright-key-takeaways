# Lesson 6 - Git Advanced & JavaScript Class

## Nội dung buổi học
1. **Git**: remote, push, pull, stashing, merge request, convention
2. **JavaScript**: Class

---

## PHẦN 1: GIT

### 1.1 Remote Repository

**Remote (remote repository)** là danh sách các repository được lưu trữ ở máy chủ từ xa (remote server), cho phép bạn cộng tác với người khác.

- Mỗi remote được định danh bằng một **tên ngắn gọn** và liên kết đến một **URL**
- Remote phổ biến nhất được đặt tên là `origin`

```bash
# Thêm remote mới
git remote add origin git@github.com:bba/k18-practice.git

# Xem danh sách remote đang có
git remote -v
```

> **Tên ngắn gọn**: `origin`
> **URL**: `git@github.com:bba/k18-practice.git`

---

### 1.2 Clone

**Clone** là hành động lấy code từ 1 repo đã có sẵn về máy tính cá nhân.

```bash
# Clone cơ bản
git clone <link repo>

# Clone và đặt tên thư mục mới
git clone <link repo> <tên mới>
```

**Ví dụ:**
```bash
git clone git@github.com:better-bytes-academy/k18-practice.git

# Clone và đặt tên thư mục là "k18-practice-2"
git clone git@github.com:better-bytes-academy/k18-practice.git k18-practice-2
```

> **Lưu ý:** Có thể clone bằng HTTPS hoặc SSH.
> - **HTTPS**: Dễ dùng nhưng mỗi lần push/pull phải nhập lại mật khẩu → bất tiện
> - **SSH**: Khuyến khích dùng vì không cần nhập mật khẩu mỗi lần

---

### 1.3 Push

**Push** là hành động đưa code từ vùng repository ở local của một nhánh cụ thể lên trên remote.

```bash
git push <remote_name> <branch_name>
```

**Ví dụ:**
```bash
git push origin main
git push origin feat/lesson-6
```

---

### 1.4 Pull

**Pull** là hành động lấy code từ một nhánh cụ thể về nhánh local.

```bash
git pull <remote_name> <branch_name>
```

**Ví dụ:**
```bash
git pull origin main
```

> Có thể pull từ một nhánh khác nhánh hiện tại về nhánh local → lúc này hiện tượng **merge** xảy ra (sẽ học ở các bài sau).

**Luồng hoạt động:**
```
Remote ──pull──> Repository (local) ──> Workspace
       <──push──
       <──fetch──
```

---

### 1.5 Stash

**Stash** là hành động lưu trữ các thay đổi hiện tại (ở vùng staging) vào một vùng nhớ tạm, giúp bạn có thể chuyển đổi qua một nhánh khác dễ dàng hơn mà không cần commit dở.

```bash
# Lưu thay đổi vào stash
git stash
git stash save

# Lưu cả untracked files
git stash -u

# Xem danh sách stash
git stash list

# Lấy thay đổi ra khỏi stash (lấy stash mới nhất)
git stash pop

# Lấy stash theo index cụ thể
git stash pop stash@{1}
```

**Luồng hoạt động của Stash:**
```
Working directory ──(1) git stash──> Stash
                  <──(3) git stash pop──
        (2) git pull/rebase ──> Git Repository
```

> **Khi nào dùng stash?** Khi bạn đang code dở trên nhánh A, cần chuyển sang nhánh B gấp nhưng chưa muốn commit → stash để tạm lưu, sau đó pop ra tiếp tục.

---

### 1.6 Merge Request (Pull Request)

**Merge request** = gộp code từ 1 nhánh sang nhánh còn lại.

- Trên GitHub gọi là **Pull Request (PR)**
- Trên GitLab gọi là **Merge Request (MR)**
- Dùng để review code trước khi merge vào nhánh chính

---

### 1.7 Reviewer

**Reviewer** = người review code.

- Một lần review = một lần học từ người khác
- Đẩy code lên sớm thì sẽ được review kĩ hơn
- Review giúp phát hiện lỗi sớm và cải thiện chất lượng code

---

### 1.8 Git Convention (Quy tắc đặt tên)

**Convention** = bộ quy tắc đặt tên giúp code team **gọn gàng, đồng bộ** và **dễ đoán được ý đồ** của PR/commit.

#### Đặt tên Branch

**Cú pháp:**
```
<type>/<short-description>-<name>
```

**Các type phổ biến:**
| Type | Ý nghĩa |
|------|---------|
| `feat` | Tính năng mới |
| `fix` | Sửa lỗi |
| `conf` | Thay đổi cấu hình (config) |
| `chore` | Các thay đổi "lặt vặt": xóa file không dùng, đổi tên file... |

**Ví dụ branch name:**
```
feat/checkout
fix/fill-info
feat/lesson-6-long
conf/update-timeout
```

| Bad (Không nên) | Good (Nên dùng) |
|----------------|----------------|
| `branch1` | `feat/lesson-5-minhphong306` |
| `feat` | `conf/update-timeout` |
| `feat/lan1` | |
| `tmp` | |

#### Viết Commit Message

**Cú pháp:**
```
<type>: <short-description>
```

**Ví dụ commit message:**
```
feat: add solution for test 1
fix: add missing continue
conf: update global setup conf
```

| Bad (Không nên) | Good (Nên dùng) |
|----------------|----------------|
| `fix code` | `feat: add solution for test 1` |
| `fix 1` | `fix: add missing continue` |
| `abcadfs` | `conf: update global setup conf` |
| `feat: add solution and subtract method and add method to test 1` (quá dài) | |
| `feat:lesson1` (thiếu space) | |

---

## PHẦN 2: JAVASCRIPT - CLASS

### 2.1 Class là gì?

**Class** là một "bản thiết kế" (blueprint) để tạo ra các object có cùng đặc điểm và hành vi.

> Hãy tưởng tượng:
> - **Class** giống như bản vẽ thiết kế một chiếc xe hơi
> - **Object** (đối tượng) là chiếc xe thực tế được sản xuất từ bản vẽ đó

---

### 2.2 Tại sao cần dùng Class?

**Trước khi có Class (cách cũ)** - phải tạo object thủ công, lặp lại code nhiều:

```javascript
// Tạo user thủ công - lặp lại code nhiều
let user1 = {
  name: "Nam",
  age: 25,
  sayHello: function() {
    console.log("Xin chào, tôi là " + this.name);
  }
};

let user2 = {
  name: "Lan",
  age: 23,
  sayHello: function() {
    console.log("Xin chào, tôi là " + this.name);
  }
};

// Phải copy-paste code → dễ sai và khó maintain
```

**Sau khi có Class (cách mới - tốt hơn):**

```javascript
class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  sayHello() {
    console.log("Xin chào, tôi là " + this.name);
  }
}

// Tạo nhiều user dễ dàng, code gọn
let user1 = new User("Nam", 25);
let user2 = new User("Lan", 23);

user1.sayHello(); // "Xin chào, tôi là Nam"
user2.sayHello(); // "Xin chào, tôi là Lan"
```

**Lợi ích của Class:**
- Code ngắn gọn, không lặp lại
- Dễ bảo trì và mở rộng
- Tổ chức code rõ ràng hơn

---

### 2.3 Khai báo Class

```javascript
class TenClass {
  // Nội dung class ở đây
}
```

**Quy tắc đặt tên class:** Dùng **PascalCase** (chữ đầu mỗi từ viết hoa)

```javascript
// Ví dụ
class TestCase {
  // Class để quản lý test case
}

class Product {
  // Class để quản lý sản phẩm
}

class Student {
  // Class để quản lý học sinh
}
```

---

### 2.4 Constructor (Hàm tạo)

**Constructor** là hàm khởi tạo, tự động chạy khi tạo một object mới từ class.

```javascript
class Student {
  // Constructor - hàm khởi tạo
  constructor(name, grade) {
    this.name = name;   // Thuộc tính name
    this.grade = grade; // Thuộc tính grade
  }
}

// Tạo object mới - constructor tự chạy
let student1 = new Student("Hùng", 8.5);
console.log(student1.name);  // "Hùng"
console.log(student1.grade); // 8.5
```

---

### 2.5 Thuộc tính (Properties)

**Thuộc tính** là các biến lưu trữ dữ liệu của object. Mỗi object có giá trị riêng.

```javascript
class TestCase {
  constructor(id, title, priority) {
    // Thuộc tính instance (mỗi object có giá trị riêng)
    this.id = id;
    this.title = title;
    this.priority = priority;
    this.status = "Not Run"; // Giá trị mặc định
  }
}

let tc1 = new TestCase(1, "Login test", "High");
let tc2 = new TestCase(2, "Logout test", "Medium");

console.log(tc1.title);  // "Login test"
console.log(tc2.title);  // "Logout test"
console.log(tc1.status); // "Not Run"
```

---

### 2.6 Phương thức (Methods)

**Phương thức** là các hàm định nghĩa hành vi của object.

```javascript
class TestCase {
  constructor(id, title) {
    this.id = id;
    this.title = title;
    this.status = "Not Run";
  }

  // Phương thức instance
  execute() {
    this.status = "Running";
    console.log(`Đang chạy test: ${this.title}`);
  }

  pass() {
    this.status = "Passed";
    console.log(`✓ ${this.title} - PASSED`);
  }

  fail(reason) {
    this.status = "Failed";
    console.log(`✗ ${this.title} - FAILED: ${reason}`);
  }

  getInfo() {
    return `Test #${this.id}: ${this.title} [${this.status}]`;
  }
}

// Sử dụng
let test1 = new TestCase(1, "Kiểm tra đăng nhập");

test1.execute(); // "Đang chạy test: Kiểm tra đăng nhập"
test1.pass();    // "✓ Kiểm tra đăng nhập - PASSED"
console.log(test1.getInfo()); // "Test #1: Kiểm tra đăng nhập [Passed]"
```

**Ví dụ thực tế từ buổi học - Class Student:**

```javascript
class Student { // Student: PascalCase
  constructor(name, role) {
    this.name = name;
    this.role = role;
  }

  sayMyName() {
    console.log(`My name is ${this.name}`);
  }

  saySomething(message) {
    return `Say something: ${message}`;
  }
}

const tuyenVo = new Student("Tuyen vo", "Quality Control");
const luNa = new Student("Anh Ho", "Business Analytic");

console.log(tuyenVo);               // Student { name: 'Tuyen vo', role: 'Quality Control' }
console.log(tuyenVo.name, tuyenVo.role); // Tuyen vo Quality Control
tuyenVo.sayMyName();                // My name is Tuyen vo
const msg = tuyenVo.saySomething("E101 Playwright");
console.log(msg);                   // Say something: E101 Playwright
```

---

### 2.7 Tóm tắt Class

**Class giúp bạn:**
- Tạo nhiều object cùng kiểu dễ dàng
- Tổ chức code gọn gàng, rõ ràng
- Tái sử dụng code hiệu quả

**Các thành phần chính:**

| Thành phần | Mô tả | Ví dụ |
|-----------|-------|-------|
| `constructor` | Khởi tạo object, chạy tự động khi `new` | `constructor(name, age) { ... }` |
| Thuộc tính | Lưu trữ dữ liệu (`this.property`) | `this.name = name` |
| Phương thức | Định nghĩa hành vi (functions) | `sayHello() { ... }` |

---

## PHẦN 3: THỰC HÀNH DEMO

### Workflow Git thực tế trong team

```bash
# 1. Clone repo về máy
git clone git@github.com:better-bytes-academy/k18-practice.git

# 2. Xem danh sách remote
git remote -v

# 3. Tạo nhánh mới theo convention
git checkout -b practice/ten-nguoi

# 4. Thêm file, commit theo convention
git add .
git commit -m "practice: add my name file"

# 5. Push lên remote
git push origin practice/ten-nguoi

# 6. Tạo Pull Request trên GitHub để reviewer xem xét
```

### Workflow Stash thực tế

```bash
# Đang code dở trên nhánh A, cần chuyển sang nhánh B
git stash           # Lưu tạm thay đổi

git checkout main   # Chuyển sang main
git pull origin main  # Cập nhật code mới nhất

git checkout practice/thanh-tuyen  # Quay lại nhánh của mình
git stash pop       # Lấy lại thay đổi đã stash
```

### Push lên nhiều remote khác nhau

```bash
# Thêm remote thứ hai (personal repo)
git remote add personal git@github.com:username/my-personal-repo.git

# Push lên remote personal
git push personal practice/ten-nguoi

# Lưu ý: nếu tên remote đã tồn tại sẽ báo lỗi
# error: remote personal already exists
# → Đặt tên khác: personal2, personal3,...
git remote add personal2 git@github.com:username/my-repo-2.git
```

---

## TỔNG KẾT

| Lệnh Git | Chức năng |
|---------|-----------|
| `git remote add <name> <url>` | Thêm remote mới |
| `git remote -v` | Xem danh sách remote |
| `git clone <url>` | Clone repo về máy |
| `git push <remote> <branch>` | Đẩy code lên remote |
| `git pull <remote> <branch>` | Lấy code từ remote về |
| `git stash` | Lưu tạm thay đổi |
| `git stash pop` | Lấy lại thay đổi đã stash |
| `git stash list` | Xem danh sách stash |

| Khái niệm JS | Giải thích |
|-------------|-----------|
| `class` | Bản thiết kế để tạo object |
| `constructor` | Hàm chạy khi `new ClassName()` |
| `this.property` | Thuộc tính của object |
| Method | Hàm bên trong class |
| `new ClassName()` | Tạo object mới từ class |
