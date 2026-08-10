# Git Commit --amend (Sửa Commit Cuối)

---

## 1. `git commit --amend` là gì?

> Lệnh cho phép **sửa đổi commit gần nhất** — thay đổi message, thêm/bớt file, hoặc cả hai.
> Thay vì tạo commit mới, nó **viết lại** commit cuối cùng.

**Khi nào dùng:**
- Viết sai commit message (lỗi chính tả, thiếu thông tin)
- Quên thêm file vào commit
- Muốn bỏ bớt file khỏi commit cuối
- Muốn sửa nhỏ mà không tạo commit mới "rác"

> ⚠️ **Nguyên tắc vàng:** Chỉ amend commit **chưa push** lên remote.
> Nếu đã push → amend sẽ gây conflict cho người khác.

---

## 2. Các trường hợp sử dụng

### 2.1 Sửa commit message

```bash
git commit --amend -m "message mới chính xác hơn"
```

**Ví dụ thực tế:**
```bash
# Vừa commit với message sai
$ git log
# commit ...  "sai ten"

# Sửa lại message
$ git commit --amend -m "feat: add file 2"
# [main 87a69df] feat: add file 2  ← message đã được sửa
```

---

### 2.2 Thêm file bị quên (giữ nguyên message)

```bash
git add <file-bi-quen>
git commit --amend --no-edit    # --no-edit = giữ nguyên message cũ
```

**Ví dụ:**
```bash
$ git add file8.txt
$ git commit --amend --no-edit
# [lesson-03] edit add file 7 - su dung VIM
# 2 files changed  ← commit giờ có cả file7.txt và file8.txt
```

---

### 2.3 Vừa thêm file vừa sửa message

```bash
git add <file>
git commit --amend -m "message mới"
```

**Ví dụ:**
```bash
$ git add file9.txt
$ git commit --amend -m "feat: add file7, file8, file9"
# [lesson-03] feat: add file7, file8, file9
# 3 files changed  ← commit gộp cả 3 file
```

---

### 2.4 Bỏ file khỏi commit cuối

```bash
git reset HEAD~ -- <file-can-bo>
git commit --amend --no-edit
```

---

### 2.5 Sửa không dùng `-m` (mở Vim editor)

```bash
git commit --amend    # mở Vim để sửa message
```

**Cách dùng Vim:**

| Việc cần làm | Thao tác |
|---|---|
| Vào chế độ gõ chữ | Nhấn `i` (INSERT mode) |
| Lưu và thoát | Nhấn `Esc` → gõ `:wq` → `Enter` |
| Thoát không lưu | Nhấn `Esc` → gõ `:q!` → `Enter` |

---

## 3. Cơ chế hoạt động bên trong

Khi chạy `git commit --amend`, Git thực chất:
1. Lấy nội dung commit cuối + staging area hiện tại
2. Tạo một **commit mới với hash khác** thay thế commit cũ
3. Commit cũ vẫn còn trong `reflog` nhưng không hiện trên branch

```
Trước amend:   A — B — C
Sau amend:     A — B — C'   (C' thay C, hash khác nhau)
```

> Đó là lý do tại sao hash commit thay đổi sau khi amend.

---

## 4. Amend commit đã push (trường hợp bắt buộc)

```bash
git commit --amend -m "message sửa"
git push --force-with-lease     # an toàn hơn --force
```

> `--force-with-lease` kiểm tra xem remote có thay đổi từ người khác không trước khi ghi đè.

> ⚠️ **Chỉ làm trên branch cá nhân. Không bao giờ force push lên `main` / `develop` chung.**

---

## 5. Khôi phục nếu amend nhầm

```bash
git reflog                    # tìm hash commit cũ trước khi amend
git reset --hard <old-hash>   # quay lại commit đó
```

---

## 6. Tổng hợp tất cả lệnh

```bash
# Sửa message
git commit --amend -m "new message"

# Thêm file, giữ message
git add <file>
git commit --amend --no-edit

# Vừa thêm file vừa sửa message
git add <file>
git commit --amend -m "new message"

# Bỏ file khỏi commit
git reset HEAD~ -- <file>
git commit --amend --no-edit

# Sửa author
git commit --amend --author="Tên Mới <email@moi.com>" --no-edit

# Khôi phục nếu amend nhầm
git reflog
git reset --hard <old-hash>
```

---

## 7. So sánh: amend vs các cách khác

| Tình huống | Nên dùng |
|---|---|
| Sửa commit **cuối**, chưa push | `git commit --amend` ✅ |
| Sửa commit **cũ hơn** (không phải commit cuối) | `git rebase -i` |
| Commit đã push, làm việc **nhóm** | Tạo commit mới sửa lỗi |
| Commit đã push, branch **cá nhân** | `amend` + `push --force-with-lease` |
