# Tổng hợp kiến thức Git Remote

## 1. Remote name là gì?

Remote name là một **alias (tên gọi tắt) cục bộ** trong repo local, dùng để trỏ tới một URL của repo trên GitHub (hoặc bất kỳ Git server nào).

```
Tên remote  →  URL thực tế
-----------     ----------------------------
origin      →  git@github.com:org/repo.git
personal    →  git@github.com:user/repo-a.git
personal2   →  git@github.com:user/repo-b.git
```

Thông tin này được lưu trong file `.git/config` của repo local:

```ini
[remote "origin"]
    url = git@github.com:better-bytes-academy/k18-practice.git
    fetch = +refs/heads/*:refs/remotes/origin/*

[remote "personal"]
    url = git@github.com:tuyenvo224/e101-practice-personal.git
    fetch = +refs/heads/*:refs/remotes/personal/*
```

---

## 2. Quy tắc về tên remote

### Tên remote phải là DUY NHẤT trong cùng 1 repo local

Ví dụ: nếu đã có `personal` rồi, thêm `personal` lần nữa sẽ báo lỗi:

```bash
$ git remote add personal git@github.com:tuyenvo224/e101-practice-personal.git
# ✅ Thành công

$ git remote add personal git@github.com:tuyenvo224/e101-practice-personal2.git
# ❌ error: remote personal already exists.
```

**Lý do:** Tên remote là key định danh trong `.git/config` — không thể có 2 key trùng nhau, dù URL đích khác nhau.

### Giải pháp khi cần thêm remote trỏ tới repo khác

**Cách 1:** Dùng tên khác

```bash
git remote add personal2 git@github.com:tuyenvo224/e101-practice-personal2.git
```

**Cách 2:** Cập nhật URL của remote hiện có (mất liên kết repo cũ)

```bash
git remote set-url personal git@github.com:tuyenvo224/e101-practice-personal2.git
```

---

## 3. `git clone` tự động tạo remote `origin`

Khi chạy `git clone <url>`, Git tự động thực hiện nhiều bước:

```
git clone git@github.com:better-bytes-academy/k18-practice.git
```

Tương đương với:

```bash
mkdir k18-practice
cd k18-practice
git init
git remote add origin git@github.com:better-bytes-academy/k18-practice.git  # ← tự động!
git fetch origin
git checkout main
```

**Kết quả:** Sau khi clone, repo local đã có sẵn remote tên `origin` trỏ về đúng URL bạn clone từ đó — **không cần** chạy `git remote add origin` thủ công.

### So sánh 2 luồng

| Luồng | Các bước |
|-------|----------|
| **Clone** (có sẵn repo trên GitHub) | `git clone <url>` → `origin` tự có → `git push origin <branch>` ✅ |
| **Init** (tạo repo mới từ đầu) | `git init` → `git remote add origin <url>` → `git push origin <branch>` ✅ |

---

## 4. Điều kiện để `git push` hoạt động

```bash
git push <tên_remote> <tên_nhánh>
```

**Tên remote phải tồn tại trong `.git/config`** — tức là phải đã được đăng ký bằng `git remote add` hoặc tự động tạo khi `git clone`.

### Kiểm tra danh sách remote hiện có

```bash
git remote -v
```

Ví dụ output:
```
origin    git@github.com:better-bytes-academy/k18-practice.git (fetch)
origin    git@github.com:better-bytes-academy/k18-practice.git (push)
personal  git@github.com:tuyenvo224/e101-practice-personal.git (fetch)
personal  git@github.com:tuyenvo224/e101-practice-personal.git (push)
personal2 git@github.com:tuyenvo224/e101-practice-personal2.git (fetch)
personal2 git@github.com:tuyenvo224/e101-practice-personal2.git (push)
```

Chỉ các tên xuất hiện ở đây mới dùng được với `git push`.

### Nếu push bằng tên remote chưa tồn tại → lỗi ngay

```bash
$ git push personal3 practice/thanh-tuyen
# ❌ fatal: 'personal3' does not appear to be a git repository
# ❌ fatal: Could not read from remote repository.
```

---

## 5. Push lên nhiều remote khác nhau

Khi đã có nhiều remote, mỗi lệnh push đi tới **đúng URL của tên remote đó** — Git không quan tâm bạn "định" push lên repo nào, chỉ tra tên và đẩy tới URL tương ứng.

```bash
git push personal  practice/thanh-tuyen  # → push tới e101-practice-personal.git  ✅
git push personal2 practice/thanh-tuyen  # → push tới e101-practice-personal2.git ✅
git push origin    practice/thanh-tuyen  # → push tới k18-practice.git            ✅
```

Cả 3 lệnh đều thành công, chỉ khác nhau đích đến.

### Push lại lần 2 lên cùng remote

Hoàn toàn bình thường:
- Nếu không có thay đổi mới → Git báo `Everything up-to-date`
- Nếu có commit mới → Git push thêm commit đó lên

---

## 6. Các lỗi phổ biến khi push

| Lỗi | Nguyên nhân |
|-----|-------------|
| `error: remote <tên> already exists` | Tên remote đã tồn tại, không thể `git remote add` với tên đó |
| `fatal: '<tên>' does not appear to be a git repository` | Tên remote chưa được đăng ký trong `.git/config` |
| `Repository not found` | Repo trên GitHub bị xóa hoặc URL sai |
| `Permission denied (publickey)` | SSH key chưa được thêm vào GitHub account |
| `rejected - non-fast-forward` | Remote có commit mới hơn local, cần `git pull` trước |

---

## 7. Các lệnh Git Remote thường dùng

```bash
# Xem danh sách remote
git remote -v

# Thêm remote mới
git remote add <tên> <url>

# Đổi URL của remote đã có
git remote set-url <tên> <url_mới>

# Xóa remote
git remote remove <tên>

# Đổi tên remote
git remote rename <tên_cũ> <tên_mới>

# Push lên remote cụ thể
git push <tên_remote> <tên_nhánh>

# Push và set upstream (lần sau chỉ cần git push)
git push -u <tên_remote> <tên_nhánh>
```

---

## 8. Tóm tắt nhanh

```
clone  = init + remote add origin + fetch + checkout
              ↓
         origin tự động có sẵn

remote name = alias CỤC BỘ, duy nhất trong 1 repo local
            ≠ liên quan tới GitHub account hay URL đích

git push <tên> <branch>
         ↑
         Phải tồn tại trong .git/config
         Nếu không → fatal error
         Nếu có → push tới đúng URL mà tên đó trỏ tới
```
