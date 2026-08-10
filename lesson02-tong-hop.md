# Lesson 2 - Tóm Tắt Kiến Thức

> **Nội dung bài học:**
> 1. Version Control System (VCS)
> 2. Git
> 3. JavaScript Basic

---

## PHẦN 1: VERSION CONTROL SYSTEM (VCS)

### 1.1 VCS là gì?

**Version Control System = Hệ thống quản lý phiên bản**

Giống như tính năng "Version history" trong Google Docs — bạn biết được:
- **Ai** đã thay đổi?
- **Khi nào** thay đổi?
- **Thay đổi gì?**

### 1.2 Lợi ích của VCS

- Xem lịch sử thay đổi (thời gian, nội dung, tác giả)
- Quay trở về bản trước đó khi cần (Restore)
- Hỗ trợ cộng tác nhóm

### 1.3 Ba loại VCS

| Loại | Lưu ở đâu | Cộng tác | Làm offline | Tốc độ |
|------|-----------|----------|-------------|--------|
| **Local** | Máy cá nhân | Không | Có | Nhanh |
| **Centralized** | Máy chủ tập trung | Có | Không | Phụ thuộc mạng |
| **Distributed** | Nhiều máy | Có | Có | Nhanh |

### 1.4 Tại sao Distributed VCS phổ biến nhất?

- Làm việc offline được
- Mỗi máy có bản sao đầy đủ (backup tự động)
- Nhanh hơn vì không phụ thuộc mạng
- **Git** chính là một Distributed VCS

---

## PHẦN 2: GIT

### 2.1 Lịch sử Git

- Tác giả: **Linus Torvalds** (cha đẻ hệ điều hành Linux)
- Ra đời năm **2005** sau khi chia tay BitKeeper (một DVCS có bản quyền)
- Linus viết Git chỉ trong **vài tuần**
- Hiện nay Git chiếm **93.87%** thị phần VCS trên thế giới

### 2.2 Git ≠ GitHub

> Đây là **hai khái niệm hoàn toàn khác nhau!**

| Git | GitHub |
|-----|--------|
| Phần mềm cài trên máy | Dịch vụ web |
| Công cụ dòng lệnh | Giao diện website |
| Quản lý phiên bản local | Nơi lưu trữ Git repo trên cloud |
| Miễn phí, mã nguồn mở | Có tính năng bổ sung (Actions, Copilot) |

### 2.3 Ba Trạng Thái (Three States) — Quan trọng nhất!

```
working directory  --[git add]-->  staging area  --[git commit]-->  repository
```

| Vùng | Màu trong VS Code | Mô tả |
|------|------------------|-------|
| **Working Directory** | Đỏ | File đang chỉnh sửa, chưa add |
| **Staging Area** | Xanh lá | File đã `git add`, sẵn sàng commit |
| **Repository** | — | Lịch sử commit đã lưu |

**Ví dụ minh hoạ từng bước:**

```
# Bắt đầu: có 3 file trong working directory
test1.spec.ts  .gitignore  package.json

# Bước 1: add 1 file
$ git add test1.spec.ts
→ test1.spec.ts đi vào Staging Area

# Bước 2: commit
$ git commit -m "feat: add test 1"
→ test1.spec.ts vào Repository với message "feat: add test 1"

# Bước 3: add tất cả file còn lại
$ git add .
→ .gitignore và package.json vào Staging Area

# Bước 4: commit lần 2
$ git commit -m "chore: add other files"
→ .gitignore và package.json vào Repository
```

---

## PHẦN 3: CÁC LỆNH GIT CƠ BẢN

### 3.1 Khởi tạo repository

```bash
git init
```

### 3.2 Cấu hình thông tin (làm 1 lần)

```bash
# Cấu hình chung cho toàn bộ máy
git config --global user.name "Tên bạn"
git config --global user.email "email@example.com"

# Cấu hình riêng cho từng repo (nếu cần khác nhau)
git config user.name "Alex"
git config user.email "alex@company.com"
```

> **Lưu ý:** Phải chạy `git init` trước rồi mới chạy `git config` cho repo đó.

### 3.3 Thêm file vào Staging Area

```bash
git add file1.txt              # Add 1 file
git add file1.txt file2.txt    # Add nhiều file cùng lúc
git add folder/file4.txt       # Add file trong subfolder (phải ghi đường dẫn)
git add .                      # Add tất cả file
```

### 3.4 Commit

```bash
git commit -m "feat: add new feature"
```

### 3.5 Kiểm tra trạng thái

```bash
git status
```

**Output ví dụ:**
```
On branch main
Changes to be committed:  (đang ở Staging Area - màu xanh)
        modified:   file3.txt

Untracked files:  (chưa được track - màu đỏ)
        file6.txt
```

### 3.6 Xem lịch sử commit

```bash
git log
```

**Output ví dụ:**
```
commit ef55348 (HEAD -> main)
Author: Tuyen Vo <email>
Date:   Sun May 17 21:28:09 2026 +0700
    feat: add file 6

commit 67db87c
    fix: update file 1
```

### 3.7 Kết nối và đẩy lên GitHub

```bash
git remote add origin <url>
git push origin main
```

### 3.8 Quy trình tổng kết

```
# Lần đầu tiên (setup)
git init
git config user.name / email   ← bỏ qua nếu đã dùng --global
git remote add origin <url>

# Mỗi khi có thay đổi
git add .
git commit -m "<message>"
git push origin main
```

---

## PHẦN 4: COMMIT CONVENTION (Quy ước đặt tên commit)

### Tại sao cần convention?

- Cả team làm việc theo **một quy tắc thống nhất**
- Nhìn vào commit biết ngay **nội dung thay đổi là gì**
- Code trông **gọn gàng, chuyên nghiệp** hơn

### Định dạng

```
<type>: <short_description>
```

| Type | Khi nào dùng | Ví dụ |
|------|-------------|-------|
| `feat` | Thêm tính năng mới, test case mới | `feat: add login test case` |
| `fix` | Sửa lỗi, sửa test bị fail | `fix: fix automation for case 1` |
| `chore` | Sửa nhỏ lẻ, xoá file thừa, sửa chính tả | `chore: remove unused file` |

**Lưu ý:**
- `short_description`: tối đa 50 ký tự
- Viết tiếng Anh hoặc tiếng Việt không dấu đều được

---

## PHẦN 5: JAVASCRIPT CƠ BẢN

### 5.1 JavaScript là gì?

- Là một **ngôn ngữ lập trinh**, ra đời năm 1995 bởi **Brendan Eich**
- Vai trò: HTML = xương, CSS = da, **JS = não bộ** (giúp trang web có logic)
- Chạy được nhờ **browser engine**: Chrome dùng V8, Firefox dùng SpiderMonkey
- Khi chạy trên máy tính (không có browser) → dùng **NodeJS**

### 5.2 Chạy file JavaScript với NodeJS

```bash
# Cài NodeJS qua NVM
nvm install v22.16.0
nvm use v22.16.0
node -v    # kiểm tra version

# Chạy file
node 01-hello.js              # file ở thư mục hiện tại
node folder/02-hello.js       # file trong subfolder
```

---

### 5.3 Hello World

```javascript
console.log("Hello world!");
console.log('Hello world!');    // dùng nháy đơn hoặc nháy kép đều được
```

---

### 5.4 Comment

```javascript
// Đây là comment 1 dòng — code sau dấu // sẽ không chạy

const name = "Playwright VN"  // Comment cuối dòng cũng được

/*
  Đây là comment nhiều dòng
  Tất cả code trong đây đều bị bỏ qua
  console.log("không chạy");
*/
```

> **Phím tắt:** `Ctrl + /` (Windows) hoặc `Cmd + /` (Mac) để comment/uncomment nhanh

---

### 5.5 Biến và Hằng

#### Khai báo biến

```javascript
var centerName = "Better Bytes Academy";   // cú pháp cũ, ít dùng
let isLovePlaywright = true;               // cú pháp hiện đại
```

#### Khai báo hằng

```javascript
const PI = 3.14;
const dollar = 26_000;    // dấu _ chỉ để dễ đọc, không ảnh hưởng giá trị
const slogan = "Học Kỹ - Hiểu Bản Chất - Mentor Tận Tình";
```

#### So sánh var vs let vs const

| | `var` | `let` | `const` |
|--|-------|-------|---------|
| Khai báo lại | Được | **Lỗi** | **Lỗi** |
| Thay đổi giá trị | Được | Được | **Lỗi** |
| Phạm vi (scope) | Function (rộng) | Block (an toàn) | Block (an toàn) |

```javascript
// var nguy hiểm: cho phép khai báo lại (dễ gây bug)
var x = 1;
var x = 2;  // OK nhưng nguy hiểm!

// let an toàn hơn
let y = 1;
let y = 2;  // LỖI: Identifier 'y' has already been declared

// Ví dụ block scope
{
    var a = 10;
    let b = 11;
}
console.log(a);  // 10 — var thoát ra ngoài block được
console.log(b);  // LỖI: b is not defined — let bị giới hạn trong block
```

> **Best practice:**
> - Mặc định dùng `const`
> - Chỉ dùng `let` khi **chắc chắn** cần gán lại giá trị
> - **Không dùng `var`**

---

### 5.6 Kiểu Dữ Liệu (Data Types)

Mỗi biến/hằng đều có một kiểu dữ liệu:

```javascript
// Number — số nguyên và số thực
const age = 25;
const price = 19.99;
const inf = Infinity;
const notNum = NaN;  // Not a Number

// String — chuỗi ký tự
const name = "John";           // nháy kép
const message = 'Hello';       // nháy đơn
const template = `Age: 10`;    // backtick (dấu huyền `)

// Boolean — giá trị logic
const isActive = true;
const isComplete = false;
```

**Kiểm tra kiểu dữ liệu:**

```javascript
typeof 25         // "number"
typeof "hello"    // "string"
typeof true       // "boolean"
```

---

### 5.7 Toán Tử So Sánh

```javascript
// == (Loose Equality) — so sánh sau khi chuyển đổi kiểu
5 == "5"      // true  (string "5" được chuyển thành number 5)
true == 1     // true  (true chuyển thành 1)
false == 0    // true  (false chuyển thành 0)

// === (Strict Equality) — so sánh cả giá trị VÀ kiểu — NÊN DÙNG
5 === "5"     // false (khác kiểu: number vs string)
true === 1    // false (khác kiểu)
5 === 5       // true  (cùng kiểu, cùng giá trị)

// != và !==
5 != "5"      // false (sau khi chuyển đổi thì bằng nhau)
5 !== "5"     // true  (khác kiểu)

// So sánh lớn hơn, nhỏ hơn
5 > 10    // false
5 >= 10   // false
5 < 10    // true
5 <= 10   // true
```

> **Luôn dùng `===` và `!==`** — chỉ dùng `==` khi có chủ đích muốn bỏ qua kiểu dữ liệu

---

### 5.8 Toán Tử Logic

```javascript
// && (AND) — cả 2 vế đúng mới đúng
const isMoreThan6 = true;
const isOfficial = true;
const receive14 = isMoreThan6 && isOfficial;  // true

// || (OR) — chỉ cần 1 vế đúng
const receive13 = isMoreThan6 || isOfficial;  // true
```

**Bảng chân lý:**

| A | B | A && B | A \|\| B |
|---|---|--------|---------|
| true | true | **true** | **true** |
| true | false | **false** | **true** |
| false | true | **false** | **true** |
| false | false | **false** | **false** |

---

### 5.9 Toán Tử Toán Học

```javascript
const a = 5;
const b = 10;

a + b   // 15 — cộng
a - b   // -5 — trừ
a * b   // 50 — nhân
a / b   // 0.5 — chia
a / 0   // Infinity — chia cho 0

// Toán tử gộp (compound assignment)
let sum = 0;
sum += 5;   // sum = sum + 5 = 5
sum += 3;   // sum = sum + 3 = 8
```

---

### 5.10 Toán Tử Một Ngôi (Increment / Decrement)

```javascript
let x = 5;
x++   // Postfix: trả về giá trị TRƯỚC rồi mới tăng
++x   // Prefix: tăng TRƯỚC rồi mới trả về
x--   // Postfix: trả về giá trị TRƯỚC rồi mới giảm
--x   // Prefix: giảm TRƯỚC rồi mới trả về
```

**Ví dụ cụ thể:**

```javascript
let a = 10;
let b = ++a;
// ++a: tăng a lên 11 TRƯỚC rồi gán cho b
console.log(a);  // 11
console.log(b);  // 11

let c = 10;
let d = c++;
// c++: gán giá trị 10 cho d TRƯỚC rồi mới tăng c
console.log(c);  // 11
console.log(d);  // 10
```

---

### 5.11 Câu Điều Kiện (if / else)

```javascript
// Cú pháp cơ bản
if (dieu_kien) {
    // chạy khi điều kiện ĐÚNG
}

// Ví dụ
let hour = 8;
if (hour <= 11) {
    console.log("Good morning");
}

// Kết hợp nhiều điều kiện
if (hour >= 6 && hour <= 11) {
    console.log("Good morning");
}

// If lồng nhau (nested if)
if (hour >= 6) {
    if (hour <= 11) {
        console.log("Good morning");
    }
}
```

Các loại câu điều kiện trong JavaScript:
- `if`
- `if...else`
- `if...else if...else`
- `switch...case`

---

### 5.12 Vòng Lặp (for loop)

```javascript
// Cú pháp
for (khoi_tao; dieu_kien_lap; cap_nhat) {
    // code lặp lại
}

// Ví dụ: in "Xin chào!" 5 lần
for (let i = 0; i < 5; i++) {
    console.log("Xin chào!");
}
```

**Giải thích từng vòng:**

| Vòng | i | Kiểm tra i < 5 | Kết quả |
|------|---|---------------|---------|
| 1 | 0 | 0 < 5 → đúng | Chạy → i++ → i=1 |
| 2 | 1 | 1 < 5 → đúng | Chạy → i++ → i=2 |
| 3 | 2 | 2 < 5 → đúng | Chạy → i++ → i=3 |
| 4 | 3 | 3 < 5 → đúng | Chạy → i++ → i=4 |
| 5 | 4 | 4 < 5 → đúng | Chạy → i++ → i=5 |
| 6 | 5 | 5 < 5 → **sai** | **Dừng** |

**Ví dụ tính tổng với `+=`:**

```javascript
let sum = 0;
for (let i = 1; i <= 10; i++) {
    sum += i;   // sum = sum + i
}
// i=1: sum=1, i=2: sum=3, i=3: sum=6, ... i=10: sum=55
console.log(sum);  // 55
```

Các loại vòng lặp trong JavaScript:
- `for (i)` — vòng lặp đếm
- `for (of)` — duyệt qua từng phần tử
- `forEach` — phương thức của array
- `for (in)` — duyệt key của object
- `while` — lặp khi điều kiện đúng
- `do...while` — chạy ít nhất 1 lần

---

## PHẦN 6: MẸO VS CODE

| Tác vụ | Windows | Mac |
|--------|---------|-----|
| Format code | `Alt + Shift + F` | `Option + Shift + F` |
| Comment / Uncomment | `Ctrl + /` | `Cmd + /` |

---

## TÓM TẮT TOÀN BỘ BÀI HỌC

```
Lesson 2
├── Version Control System
│   ├── Định nghĩa: hệ thống quản lý phiên bản
│   └── 3 loại: Local / Centralized / Distributed
│
├── Git
│   ├── Lịch sử: Linus Torvalds, 2005, 93.87% thị phần
│   ├── Git ≠ GitHub (phần mềm ≠ dịch vụ web)
│   ├── Three States: Working Dir → Staging → Repository
│   ├── Lệnh cơ bản: init, config, add, commit, status, log, push
│   └── Commit convention: feat / fix / chore
│
└── JavaScript Basic
    ├── Chạy file: node <file.js>
    ├── console.log(), comment (//, /* */)
    ├── Biến: var (cũ), let (hiện đại), const (hằng)
    ├── Kiểu dữ liệu: Number, String, Boolean
    ├── Toán tử: so sánh (===), logic (&&, ||), toán học (+,-,*,/)
    ├── Toán tử một ngôi: ++, -- (prefix vs postfix)
    ├── Câu điều kiện: if / else
    └── Vòng lặp: for(i)
```
