# Lesson 3 — Git & JavaScript Basic (tiếp theo)

> **Nội dung bài học:**
> - Git: Undo actions, Branching, .gitignore
> - JavaScript: Convention, console.log nâng cao, Object, Array, Function

---

## PHẦN 1: GIT — UNDO THINGS (Hoàn tác)

Có 3 tình huống cần hoàn tác:

```
1. Sửa commit message đã commit
2. Đưa file từ Staging Area → Working Directory (unstage)
3. Đưa file từ Repository → Working Directory (un-commit)
```

---

### 1.1 Sửa Commit Message

Dùng khi vừa commit xong nhưng message bị sai chính tả hoặc nội dung:

```bash
git commit --amend -m "message mới"
```

> Chỉ sửa được **commit gần nhất**.

---

### 1.2 Unstage — Đưa file từ Staging về Working Directory

```
Working Directory  --[git add]-->  Staging Area  --[git commit]-->  Repository
                   <-[git restore --staged]--
```

```bash
git restore --staged <tên_file>   # unstage 1 file cụ thể
git restore --staged .            # unstage toàn bộ files
```

**Ví dụ thực tế:**
```bash
$ git add .
$ git status
# Changes to be committed: new file: file7.txt // dòng này có màu xanh lá

$ git restore --staged .
$ git status
# Untracked files: file7.txt   ← file quay về working directory // dòng này có màu đỏ
```

---

### 1.3 Un-commit — Hoàn tác Commit

```bash
git reset HEAD~1    # hoàn tác 1 commit gần nhất
git reset HEAD~3    # hoàn tác 3 commit gần nhất
```

**Ví dụ:**
```bash
$ git log
# commit d1b759b  ← HEAD (commit "add file 7")
# commit a516bac  (commit "add file6")

$ git reset HEAD~1

$ git log
# commit a516bac  ← HEAD (commit "add file6")
# (commit "add file 7" đã bị xóa, không xuất hiện trong log)
```

> ⚠️ **Lưu ý quan trọng:**
> - **Không thể reset commit đầu tiên** (initial commit)
> - Nếu muốn reset commit đầu tiên → xóa thư mục `.git` rồi `git init` lại

---

### 1.4 Sơ đồ tổng hợp các lệnh Undo

```
Working Directory  ──[git add <file>]──►  Staging  ──[git commit -m"message"]──►  Repository
        ◄── git restore --staged <file> ──
        ◄── git restore --staged .      ──
        ◄──────────── git reset HEAD~N  ────────────────────────────────────
```

---

## PHẦN 2: GIT — BRANCHING (Nhánh)

### 2.1 Branch là gì?

Branch = nhánh làm việc riêng biệt — tạo bản sao của code để làm tính năng mới **mà không ảnh hưởng bản gốc**.

```
main ──── C1 ──── C2 ──── C3 ─────────── merge ──►
                    \                   /
lesson-03            C4 ──── C5 ──── C6
```

- Nhánh `main`: code chính, ổn định
- Nhánh `lesson-03`: làm việc riêng, khi xong thì merge về `main`

### 2.2 Nhánh mặc định

Khi chạy `git init`, cấu hình nhánh mặc định là `main`:

```bash
git config --global init.defaultBranch main
```

### 2.3 Các lệnh Branch

```bash
git branch                      # xem danh sách nhánh (dấu * = đang đứng ở nhánh nào), cần có ít nhất 1 commit mới hiện danh sách nhánh
git branch <nhánh_a>          # tạo nhánh mới (copy từ nhánh hiện tại), vd: tạo nhánh mới là <nhánh_a>
git checkout <nhánh_b>        # chuyển sang nhánh khác, vd: chuyển sang <nhánh_b>
git checkout -b <nhánh_c>     # vừa tạo mới vừa chuyển sang nhánh mới (cách nhanh), vd: vừa tạo mới <nhánh_c> vừa chuyển sang <nhánh_c>
git branch -D <nhánh_b>       # xóa nhánh, lưu ý: đứng ở nhánh khác trước khi xóa, vd: đứng ở <nhánh_c> và thực hiện xóa <nhánh_b>
```

**Ví dụ thực tế:**
```bash
$ git branch
  lesson-03-quick2
  lesson-03
* main                    # đang ở main

$ git checkout lesson-03-quick2
Switched to branch 'lesson-03-quick2'

$ git branch
  lesson-03
* lesson-03-quick2               # đang ở lesson-03-quick2
  main
```

**Xóa nhánh:**
```bash
$ git checkout main               # phải sang nhánh khác trước
$ git branch -D lesson-03-quick2  # rồi mới xóa được
Deleted branch lesson-03-quick2
```

> ⚠️ Phải đứng ở **nhánh khác** trước khi xóa — không thể xóa nhánh đang đứng.

---

## PHẦN 3: GIT — .gitignore

### 3.1 .gitignore là gì?

File `.gitignore` chỉ định những file/thư mục Git **sẽ bỏ qua**, không theo dõi.

**Tại sao cần?** Vì dự án thường có nhiều file không cần commit:
- `node_modules/` — thư viện, rất nặng
- `.env`, `.env.prod` — chứa password, API key (bảo mật)
- `/test-results/`, `/playwright-report/` — kết quả test tạm thời
- `.DS_Store`, `Thumbs.db` — file của hệ điều hành
- `dist/`, `build/` — file build

> ⚠️ Lưu ý: tên file bắt đầu bằng dấu chấm: **`.gitignore`**

### 3.2 Cú pháp .gitignore

```gitignore
# Đây là comment

# Ignore file cụ thể
secret.txt

# Ignore tất cả file .log
*.log

# Ignore toàn bộ thư mục
node_modules/
build/

# Ignore file trong mọi thư mục con
**/*.tmp

# Ngoại lệ — KHÔNG ignore file này (dùng !)
!important.log

# Ví dụ từ dự án Playwright thực tế
node_modules/
/test-results/
/playwright-report/
/blob-report/
/playwright/.cache/
```

### 3.3 Thực hành

```bash
# Tạo các file .env
# Thêm vào .gitignore:
.env
.env.dev
.env.prod

# Kiểm tra — 3 file này sẽ không xuất hiện trong git status
git status
```

---

## PHẦN 4: JAVASCRIPT — CONVENTION (Quy ước đặt tên)

### 4.1 Convention là gì?

> **Convention = quy tắc** — giúp code đồng nhất, dễ đọc, dễ làm việc nhóm.

### 4.2 Các loại Convention phổ biến

| Convention | Hình dạng | Ví dụ |
|-----------|-----------|-------|
| `snake_case` | chữ thường, cách bởi `_` | `vo_thi_thanh_tuyen` |
| `kebab-case` | chữ thường, cách bởi `-` | `vo-thi-thanh-tuyen` |
| `camelCase` | chữ đầu thường, từ sau viết hoa đầu | `voThiThanhTuyen` |
| `PascalCase` | tất cả từ viết hoa đầu | `VoThiThanhTuyen` |
| `UPPER_CASE` | tất cả viết hoa, cách bởi `_` | `VO_THI_THANH_TUYEN` |

### 4.3 Áp dụng trong lớp học

| Đặt tên gì | Dùng convention nào |
|-----------|-------------------|
| Tên **file** và **folder** | `kebab-case` |
| Tên **biến** và **hàm** | `camelCase` |
| Tên **class** | `PascalCase` |

---

## PHẦN 5: JAVASCRIPT — CONSOLE.LOG NÂNG CAO

### 5.1 Template Literal (cách in chuỗi hiện đại)

```javascript
let myName = "Tuyen";
let queQuan = "Ho Chi Minh";

// Cách cũ — nối chuỗi bằng +
console.log("Ten toi la " + myName + ", toi den tu " + queQuan);

// Cách mới — template literal (dùng backtick ` và ${})
console.log(`Ten toi la ${myName}, toi den tu ${queQuan}`);

// Kết quả giống nhau: Ten toi la Tuyen, toi den tu Ho Chi Minh
```

> Template literal dùng ký tự backtick `` ` `` (phím bên trái số 1), không phải nháy đơn.

---

## PHẦN 6: JAVASCRIPT — OBJECT

### 6.1 Object là gì?

> **Object = đối tượng** — dùng để nhóm nhiều giá trị liên quan vào **một biến duy nhất**, theo dạng `key: value`.

### 6.2 Khai báo Object

```javascript
const/let <tên_biến> = {
    key1: value1,
    key2: value2,
};
```

**Ví dụ thực tế:**
```javascript
let user = {
    name: "Tuyen",
    age: 35,
    email: "tuyen@gmail.com"
};

const product = {
    name: "Laptop",
    price: 500,
    isWindow: true,
    manufacturer: {           // object lồng trong object
        name: "Acer",
        year: 2024
    }
};
```

### 6.3 Truy xuất giá trị

```javascript
// Cách 1: Dot notation (.)
console.log(user.name);                     // "Tuyen"
console.log(product.manufacturer.name);    // "Acer"

// Cách 2: Bracket notation ["key"]
console.log(product["price"]);                       // 500
console.log(product["manufacturer"]["name"]);        // "Acer"
```


### 6.4 Cập nhật giá trị

```javascript
user.age = 28;
product["manufacturer"]["year"] = 2025;
```

### 6.5 Ví dụ thực tế (chạy được)

```javascript
const myInfo ={
    "name": 'Tuyen',
    favoriteNumber: 22,
    address: "Ho Chi Minh",
    "my address 2": "Hue",
    myAdressThree: "Quan 12",
    'myAdressFour': "Trung My Tay",
    'my address 5': "khu pho 1",
    isLoveCoding: true,
    codingClass:{
        name: "Playwright",
        level: "Beginer to Junior"
    }
};

console.log(myInfo.name);               // Tuyen
console.log(myInfo.codingClass.name);   // Playwright
console.log(myInfo["codingClass"]["level"]); // Beginner to Junior
console.log(myInfo["name"]); // Tuyen
console.log(myInfo["codingClass"]["name"]); // Playwright
console.log(myInfo["codingClass"]); // { name: 'Playwright', level: 'Beginer to Junior' }
console.log(myInfo["my address 2"]); // Hue
```
> **Khi nào dùng bracket?** Khi key có dấu cách: `myInfo["my address 2"]`

---

## PHẦN 7: JAVASCRIPT — LOGICAL OPERATOR (Toán tử Logic)

```javascript
&&   // AND: cả 2 vế đúng → đúng
||   // OR:  1 trong 2 vế đúng → đúng
!    // NOT: đảo ngược giá trị
```

```javascript
const isMoreThan6 = true;
const isOfficial = false;

console.log(isMoreThan6 && isOfficial);  // false (1 vế sai)
console.log(isMoreThan6 || isOfficial);  // true  (1 vế đúng là đủ)
console.log(!isOfficial);                // true  (đảo ngược false → true)
```

---

## PHẦN 8: JAVASCRIPT — ARRAY (Mảng)

### 8.1 Array là gì?

> **Array = mảng** — danh sách các phần tử **cùng loại**, được đánh số thứ tự từ `0`.

### 8.2 Khai báo và truy xuất

```javascript
const arr = [25, 45, 10, 35];

console.log(arr.length);  // 4 — số phần tử, chiều dài mảng
console.log(arr[0]);      // 25 — phần tử đầu (index 0)
console.log(arr[1]);      // 45 - phần tử thứ 2 (index 1)
console.log(arr[3]);      // 35 — phần tử cuối (index 3)
```

> Index bắt đầu từ **0**, không phải 1.

### 8.3 Duyệt mảng với for loop

```javascript
const arr = [25, 45, 10, 35];

for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
}
// In ra: 25, 45, 10, 35
```

---

## PHẦN 9: JAVASCRIPT — FUNCTION (Hàm)

### 9.1 Function là gì?

> **Function = hàm** — đoạn code được đặt tên, **có thể gọi lại nhiều lần**, tránh viết code lặp lại.

### 9.2 Khai báo và gọi hàm

```javascript
// Khai báo
function <tênHàm>(thamSo1, thamSo2) {
    // code xử lý
    return kết_quả;   // trả về giá trị (nếu cần)
}

// Gọi hàm
tênHàm(giaTri1, giaTri2);
```

### 9.3 Ví dụ — Hàm không có return

```javascript
function xinChaoBaLan() {
    console.log("Xin chao");
    console.log("Xin chao");
    console.log("Xin chao");
}

xinChaoBaLan();  // In 3 dòng "Xin chao"
xinChaoBaLan();  // Gọi lại, in thêm 3 dòng nữa
```

### 9.4 Ví dụ — Hàm có tham số và return
#### Có return
```javascript
// Tính diện tích hình chữ nhật
function tinhDienTich(dai, rong) {
    const dienTich = dai * rong;
    return dienTich;
}

console.log(tinhDienTich(5, 10));  // 50
console.log(tinhDienTich(2, 11));  // 22
```
#### Không có return
```javascript
function tinhDienTich (dai, rong){
const dienTich= dai*rong;
console.log(`Dien tich hinh chu nhat (${dai}X${rong})= ${dienTich}`);
}

tinhDienTich(5,10);
tinhDienTich(2,11);
```

### 9.5 Ví dụ — Hàm có điều kiện

```javascript
function kiemTraChanLe(number) {
    if (number % 2 === 0) {
        return "Đây là số chẵn";
    }
    if (number % 2 !== 0) {
        return "lẻ";
    }
}

console.log(kiemTraChanLe(10));  // "Đây là số chẵn"
console.log(kiemTraChanLe(11));  // "lẻ"
```

### 9.6 So sánh: Không dùng function vs Dùng function

```javascript
// ❌ Không dùng function — code lặp, khó bảo trì
const dai1 = 5;  const rong1 = 10;
console.log(`Dien tich (${dai1}x${rong1}) = ${dai1 * rong1}`);

const dai2 = 2;  const rong2 = 11;
console.log(`Dien tich (${dai2}x${rong2}) = ${dai2 * rong2}`);

// ✅ Dùng function — gọn, tái sử dụng
function tinhDienTich(dai, rong) {
    return dai * rong;
}
console.log(`Dien tich (5x10) = ${tinhDienTich(5, 10)}`); // Dien tich (5x10) = 50
console.log(`Dien tich (2x11) = ${tinhDienTich(2, 11)}`); // Dien tich (2x11) = 22
console.log(tinhDienTich(5, 10)); // 50
console.log(tinhDienTich(2, 11)); // 22
```

---

## PHẦN 10: Ký hiệu trạng thái file trong VS Code / Git

| Ký hiệu | Tên | Ý nghĩa |
|---------|-----|---------|
| `U` | Untracked | File mới, chưa được Git theo dõi lần nào |
| `A` | Added | Đã `git add`, đang ở Staging Area |
| `M` | Modified | File đã bị sửa sau lần commit trước |

---

## TÓM TẮT TOÀN BỘ BÀI HỌC

```
Lesson 3
├── Git Undo
│   ├── Sửa commit message : git commit --amend -m"..."
│   ├── Unstage file       : git restore --staged <file> / .
│   └── Un-commit          : git reset HEAD~N
│
├── Git Branch
│   ├── Xem nhánh          : git branch
│   ├── Tạo nhánh          : git branch <tên>
│   ├── Chuyển nhánh       : git checkout <tên>
│   ├── Tạo + chuyển       : git checkout -b <tên>
│   └── Xóa nhánh          : git branch -D <tên>  (đứng ở nhánh khác trước)
│
├── Git .gitignore
│   └── Khai báo file/folder không cần commit (node_modules, .env, ...)
│
└── JavaScript
    ├── Convention         : kebab-case (file/folder), camelCase (biến/hàm), PascalCase (class)
    ├── console.log        : template literal `${biến}` thay vì nối chuỗi +
    ├── Object             : { key: value }, truy xuất bằng . hoặc ["key"]
    ├── Logical operator   : && (AND), || (OR), ! (NOT)
    ├── Array              : [1, 2, 3], truy xuất bằng [index], .length
    └── Function           : khai báo, tham số, return, tái sử dụng
```
