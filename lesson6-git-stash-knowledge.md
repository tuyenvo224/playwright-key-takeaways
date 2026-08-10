# Git Stash - Tổng hợp kiến thức đầy đủ

## 1. Khái niệm

`git stash` là lệnh lưu **tạm thời** các thay đổi chưa commit vào một "ngăn tạm" (stash stack), giúp working directory trở về trạng thái sạch (clean) để bạn có thể chuyển nhánh, pull code, hoặc làm việc khác mà không bị conflict.

**Khi nào cần dùng:**
- Đang làm dở feature A, sếp yêu cầu hotfix gấp trên nhánh khác
- Muốn pull code mới từ remote nhưng có thay đổi chưa commit
- Cần thử nghiệm gì đó mà không muốn mất code hiện tại

---

## 2. Stash hoạt động như thế nào

Stash sử dụng cơ chế **stack (LIFO - Last In First Out)**:

```
git stash        → đẩy vào stack (push)
git stash pop    → lấy ra khỏi stack + xóa (pop)
git stash apply  → lấy ra khỏi stack nhưng KHÔNG xóa
```

Cấu trúc stack:
```
stash@{0}  ← mới nhất (vừa stash)
stash@{1}
stash@{2}  ← cũ nhất
```

Sau mỗi lần pop/drop, index tự động cập nhật lại.

---

## 3. Bảng lệnh đầy đủ

| Lệnh | Mô tả |
|------|-------|
| `git stash` | Stash các file tracked đang modified/staged |
| `git stash -u` | Stash cả untracked files (file mới chưa `git add`) |
| `git stash -a` | Stash tất cả kể cả file bị ignore (.gitignore) |
| `git stash save "message"` | Stash kèm tên mô tả (cú pháp cũ) |
| `git stash push -m "message"` | Stash kèm tên mô tả (cú pháp mới, khuyến nghị) |
| `git stash push -u -m "message"` | Stash untracked + có message (cú pháp mới) |
| `git stash list` | Xem danh sách tất cả stash |
| `git stash show` | Xem tóm tắt thay đổi của stash mới nhất |
| `git stash show stash@{n}` | Xem tóm tắt thay đổi của stash theo index |
| `git stash show -p stash@{n}` | Xem chi tiết diff của stash |
| `git stash pop` | Lấy stash mới nhất ra và **xóa** khỏi stack |
| `git stash pop stash@{n}` | Lấy đúng stash theo index ra và xóa |
| `git stash apply` | Lấy stash mới nhất ra nhưng **giữ lại** trong stack |
| `git stash apply stash@{n}` | Lấy stash theo index, giữ lại trong stack |
| `git stash drop stash@{n}` | Xóa một stash khỏi stack |
| `git stash clear` | Xóa **toàn bộ** stash |
| `git stash branch <branch> stash@{n}` | Tạo nhánh mới từ stash |

---

## 4. Cú pháp đúng - Lỗi thường gặp

### Lỗi sai thứ tự flag và subcommand

```bash
# ❌ SAI - flag -u đặt trước subcommand save
git stash -u save "add new"
# fatal: subcommand wasn't specified; 'push' can't be assumed due to unexpected token 'save'

# ✅ ĐÚNG - cú pháp cũ: subcommand "save" phải đứng trước flag
git stash save -u "add new"

# ✅ ĐÚNG - cú pháp mới (khuyến nghị)
git stash push -u -m "add new"
```

### Tóm tắt cú pháp đúng

| Mục đích | Cú pháp cũ (deprecated) | Cú pháp mới (khuyến nghị) |
|----------|--------------------------|---------------------------|
| Stash đơn giản | `git stash` | `git stash push` |
| Stash có message | `git stash save "msg"` | `git stash push -m "msg"` |
| Stash + untracked | `git stash save -u "msg"` | `git stash push -u -m "msg"` |

---

## 5. Ví dụ chi tiết theo từng tình huống

### Tình huống 1: Stash cơ bản

```bash
# Đang sửa file, chưa commit
$ git status
Changes not staged for commit:
  modified:   tests/data-test/tuyenvo224.txt

# Lưu tạm vào stash
$ git stash
Saved working directory and index state WIP on practice/thanh-tuyen: 6af3150 fix: fix bug

# Working directory đã sạch
$ git status
nothing to commit, working tree clean

# Làm xong việc khác, lấy lại thay đổi
$ git stash pop
On branch practice/thanh-tuyen
Changes not staged for commit:
  modified:   tests/data-test/tuyenvo224.txt
Dropped stash@{0}
```

---

### Tình huống 2: Stash có tên mô tả

```bash
# Đang làm feature login
$ git stash push -m "WIP: feature login - chua xong phan validate"

# Xem danh sách
$ git stash list
stash@{0}: On main: WIP: feature login - chua xong phan validate

# Lấy lại theo tên (phải dùng index)
$ git stash pop stash@{0}
```

---

### Tình huống 3: Stash kể cả untracked files (-u)

```bash
# Có file mới chưa git add
$ git status
Changes not staged for commit:
  modified:   tests/data-test/tuyenvo224.txt

Untracked files:
  tests/lesson6-tuyen/01-class.js
  tests/lesson6-tuyen/02-class-constructor.js
  tests/lesson6-tuyen/tuyen_note2.txt

# git stash thông thường sẽ BỎ QUA untracked files
$ git stash
Saved working directory...
$ git status
Untracked files:           ← file mới vẫn còn!
  tests/lesson6-tuyen/01-class.js

# Phải dùng -u để stash cả untracked
$ git stash -u
Saved working directory and index state WIP on practice/thanh-tuyen: 6af3150 fix: fix bug

$ git status
nothing to commit, working tree clean   ← sạch hoàn toàn
```

---

### Tình huống 4: Nhiều stash cùng lúc

```bash
# Lần 1: stash feature A
$ git stash push -m "feature A - dang lam do"
# stash@{0}: On main: feature A - dang lam do

# Chuyển sang sửa bug, stash lần 2
$ git stash push -m "hotfix login bug"
# stash@{0}: On main: hotfix login bug        ← mới nhất
# stash@{1}: On main: feature A - dang lam do

# Xem danh sách
$ git stash list
stash@{0}: On main: hotfix login bug
stash@{1}: On main: feature A - dang lam do

# Lấy đúng feature A (không phải stash mới nhất)
$ git stash pop stash@{1}
On branch main
Changes not staged for commit:
  modified: src/featureA.js
Dropped stash@{1}

# Stack còn lại
$ git stash list
stash@{0}: On main: hotfix login bug
```

---

### Tình huống 5: apply vs pop

```bash
$ git stash list
stash@{0}: On main: WIP feature login
stash@{1}: On main: WIP feature payment

# Dùng apply - stash VẪN CÒN trong stack
$ git stash apply stash@{1}
$ git stash list
stash@{0}: On main: WIP feature login   ← vẫn còn nguyên
stash@{1}: On main: WIP feature payment ← vẫn còn nguyên

# Dùng pop - stash BỊ XÓA khỏi stack
$ git stash pop stash@{0}
$ git stash list
stash@{0}: On main: WIP feature payment  ← index cập nhật lại
```

**Khi nào dùng apply thay pop:**
- Muốn apply cùng một stash lên nhiều nhánh khác nhau
- Chưa chắc chắn apply có conflict hay không, muốn giữ stash làm backup

---

### Tình huống 6: Xem nội dung stash trước khi apply

```bash
# Xem tóm tắt (tên file thay đổi)
$ git stash show stash@{0}
 tests/data-test/tuyenvo224.txt | 2 +-
 tests/lesson6-tuyen/02-class.js | 15 +++++++++------
 2 files changed, 10 insertions(+), 7 deletions(-)

# Xem chi tiết diff (như git diff)
$ git stash show -p stash@{0}
diff --git a/tests/data-test/tuyenvo224.txt b/tests/data-test/tuyenvo224.txt
index abc1234..def5678 100644
--- a/tests/data-test/tuyenvo224.txt
+++ b/tests/data-test/tuyenvo224.txt
@@ -1,3 +1,3 @@
-old content
+new content
```

---

### Tình huống 7: Tạo nhánh mới từ stash

```bash
# Stash đang có thay đổi lớn, muốn chuyển thành nhánh riêng
$ git stash list
stash@{0}: On main: WIP feature dashboard

# Tạo nhánh mới từ stash và tự động apply
$ git stash branch feature/dashboard stash@{0}
Switched to a new branch 'feature/dashboard'
On branch feature/dashboard
Changes not staged for commit:
  modified: src/dashboard.js
Dropped stash@{0}   ← stash tự động bị xóa sau khi tạo nhánh thành công
```

---

### Tình huống 8: Stash chỉ một số file cụ thể

```bash
# Chỉ stash file cụ thể (không stash toàn bộ)
$ git stash push -m "chi stash file nay" -- tests/data-test/tuyenvo224.txt

# Hoặc nhiều file
$ git stash push -m "stash 2 file" -- file1.js file2.js
```

---

### Tình huống 9: Xóa stash

```bash
# Xóa một stash cụ thể
$ git stash drop stash@{0}
Dropped stash@{0} (abc123...)

# Xóa toàn bộ stash (cẩn thận! không recover được)
$ git stash clear
```

---

## 6. Xử lý conflict khi pop/apply

```bash
$ git stash pop
Auto-merging src/login.js
CONFLICT (content): Merge conflict in src/login.js

# Stash KHÔNG bị xóa khi có conflict (vì pop chưa hoàn thành)
$ git stash list
stash@{0}: On main: WIP feature login  ← vẫn còn

# Giải quyết conflict như bình thường
# Mở file, sửa conflict markers <<<< ==== >>>>
# Sau đó:
$ git add src/login.js
$ git stash drop stash@{0}   # xóa thủ công sau khi đã resolve
```

---

## 7. Lưu ý quan trọng

- `git stash` mặc định **bỏ qua** untracked files → dùng `-u` để bao gồm
- `git stash` mặc định **bỏ qua** file trong `.gitignore` → dùng `-a` để bao gồm
- Stash là **local**, không được push lên remote
- Khi có **conflict** lúc pop, stash không tự xóa → phải `drop` thủ công
- `git stash clear` **không thể hoàn tác** → cẩn thận!
- Nên đặt **tên mô tả** (`-m`) để dễ quản lý khi có nhiều stash

---

## 8. Hành vi khi chuyển nhánh (git checkout) - Khi nào cần stash?

Đây là phần quan trọng nhất để hiểu **tại sao** cần stash.

---

### 8.1 Quy tắc tổng quát

Git cho phép chuyển nhánh **chỉ khi** việc chuyển nhánh không làm **mất hoặc ghi đè** thay đổi chưa commit của bạn.

---

### 8.2 Bảng hành vi theo trạng thái file

| Trạng thái file | Điều kiện | Khi checkout | File đi đâu? |
|----------------|-----------|--------------|--------------|
| Modified (`M`) | File tồn tại ở cả 2 nhánh, nội dung base giống nhau | ✅ Thành công | **Đi theo** sang nhánh mới |
| Modified (`M`) | File khác version giữa 2 nhánh (sẽ bị ghi đè) | ❌ Bị chặn | Phải stash/commit trước |
| Staged (`A`) | File chưa tồn tại ở nhánh đích | ✅ Thành công | **Đi theo** sang nhánh mới |
| Staged (`A`) | File đã tồn tại ở nhánh đích với nội dung khác | ❌ Bị chặn | Phải stash/commit trước |
| Untracked | File không tồn tại ở nhánh đích | ✅ Thành công | **Đi theo** sang nhánh mới |
| Untracked | File đã tồn tại ở nhánh đích (tracked) | ❌ Bị chặn | Phải stash/commit trước |
| Committed | Luôn an toàn | ✅ Thành công | Ở lại nhánh hiện tại |

---

### 8.3 Phân tích thực tế từ ví dụ

**Bối cảnh:**
- Đang có 2 nhánh: `feat/lesson-6-tuyenvo` và `feat/lesson-6-tuyenvo2`
- File `tests/data-test/tuyenvo224.txt` bị modified (`M`) ở cả 2 nhánh
- File `tests/lesson6-tuyen/06-class.js` chỉ được commit trên `feat/lesson-6-tuyenvo`

```
Nhánh feat/lesson-6-tuyenvo  :  có 06-class.js (đã commit) + đang sửa dở
Nhánh feat/lesson-6-tuyenvo2 :  KHÔNG có 06-class.js
```

**Lần 1 - Checkout thành công, file đi theo:**
```bash
# Đang ở feat/lesson-6-tuyenvo2, có file tuyenvo224.txt đang modified
$ git checkout feat/lesson-6-tuyenvo
M       tests/data-test/tuyenvo224.txt   ← Git thông báo file "đi theo"
Switched to branch 'feat/lesson-6-tuyenvo'
```
→ `tuyenvo224.txt` đi theo vì cả 2 nhánh đều có file này với base giống nhau,
không có conflict → Git cho phép "mang" thay đổi sang nhánh mới.

**Lần 2 - Checkout bị chặn:**
```bash
# Đang ở feat/lesson-6-tuyenvo, vừa sửa 06-class.js (file chỉ có ở nhánh này)
$ git checkout feat/lesson-6-tuyenvo2
error: Your local changes to the following files would be overwritten by checkout:
        tests/lesson6-tuyen/06-class.js
Please commit your changes or stash them before you switch branches.
Aborting
```
→ Bị chặn vì `feat/lesson-6-tuyenvo2` **không có** `06-class.js`.
Nếu Git cho checkout, file này sẽ bị xóa → mất code đang làm dở.

**Giải pháp - Stash rồi checkout:**
```bash
$ git stash
Saved working directory and index state WIP on feat/lesson-6-tuyenvo: 95cba76 feat: add 2 files class

$ git checkout feat/lesson-6-tuyenvo2
Switched to branch 'feat/lesson-6-tuyenvo2'   ← thành công!
```

---

### 8.4 Sơ đồ quyết định khi muốn chuyển nhánh

```
Muốn git checkout <nhánh-khác>
           │
           ▼
   Có thay đổi chưa commit?
     │              │
    Không           Có
     │              │
     ▼              ▼
  Checkout      File thay đổi có tồn tại
  thành công    ở nhánh đích không?
                  │           │
            Có (cùng base)   Không / Khác version
                  │           │
                  ▼           ▼
             File đi theo  ❌ Bị chặn
             sang nhánh      │
             mới             ▼
                        Commit hoặc Stash
                             │
                             ▼
                        Checkout thành công
```

---

### 8.5 Checkout với `-f` (force) - NGUY HIỂM

```bash
# Bỏ qua cảnh báo, BUỘC checkout (mất toàn bộ thay đổi chưa commit!)
$ git checkout -f feat/lesson-6-tuyenvo2

# ⚠️ KHÔNG NÊN dùng trừ khi bạn chắc chắn muốn bỏ toàn bộ thay đổi
```

---

### 8.6 Tổng kết: File "đi theo" hay "bị ẩn"?

**File ĐI THEO sang nhánh mới khi:**
- File tồn tại ở cả 2 nhánh với nội dung base giống nhau (chỉ local thay đổi)
- File là untracked và không bị conflict với nhánh đích

**File BI ẨN (ở lại nhánh cũ trong stash) khi:**
- Bạn chủ động `git stash` trước khi checkout
- Stash lưu snapshot của nhánh + thay đổi → khi `pop` trên nhánh nào thì trả về đó

**Checkout BỊ CHẶN khi:**
- File đang sửa dở sẽ bị ghi đè hoặc xóa khi chuyển sang nhánh đích
- Git bảo vệ bạn khỏi mất code: bắt buộc commit hoặc stash trước

---

## 9. Workflow thực tế

```bash
# === Đang làm feature A ===
$ git status
modified: src/featureA.js
modified: src/featureA.test.js
Untracked: src/featureA-helper.js   ← file mới

# Sếp: "Fix bug login gấp!"
$ git stash push -u -m "WIP: feature A - dang lam phan validate"
Saved working directory and index state...

# Chuyển sang nhánh hotfix
$ git checkout hotfix/login-bug
# ... fix bug, commit, push ...

# Quay lại feature A
$ git checkout feature/A
$ git stash list
stash@{0}: On feature/A: WIP: feature A - dang lam phan validate

$ git stash pop
On branch feature/A
Changes not staged for commit:
  modified: src/featureA.js
  modified: src/featureA.test.js
Untracked files:
  src/featureA-helper.js
Dropped stash@{0}

# Tiếp tục làm feature A từ đúng chỗ đã dừng ✓
```
