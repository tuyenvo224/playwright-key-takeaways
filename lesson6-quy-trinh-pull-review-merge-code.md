# Quy Trình Pull Code, Review Code và Merge Code

> Tài liệu này mô tả toàn bộ quy trình làm việc với Git/GitHub trong lớp: từ lúc tạo branch, push code, tạo PR, review code cho bạn, xử lý comment, cho đến khi merge code vào nhánh main.

---

## Tổng quan quy trình (4 giai đoạn)

```
[1] Tạo branch  →  [2] Tạo PR & gửi review request  →  [3] Review code cho bạn  →  [4] Fix comment
```

---

## Giai đoạn 1: Tạo branch

### Nguyên tắc bắt buộc khi tạo branch mới
- **Luôn luôn** chuyển về nhánh `main` trước khi tạo branch mới
- **Luôn luôn** pull code mới nhất từ remote về trước
- Tên branch phải theo **naming convention** của lớp (ví dụ: `feat/lesson-6-tuyenvo`, `practice/thanh-tuyen`)

### Các lệnh thực hiện

**Bước 1: Về nhánh main và pull code mới nhất**
```bash
git checkout main
git pull origin main
```

Kết quả mong đợi sau `git pull`:
```
remote: Enumerating objects: 98, done.
remote: Counting objects: 100% (97/97), done.
Unpacking objects: 100% (98/98), done.
Fast-forward
  ...danh sách file mới được cập nhật...
```

**Bước 2: Tạo branch mới từ main**
```bash
git checkout -b feat/lesson-6-tuyenvo
```

Kết quả:
```
Switched to a new branch 'feat/lesson-6-tuyenvo'
```

> Lúc này bạn đang ở branch mới, code của `main` đã được cập nhật đầy đủ.

---

## Giai đoạn 2: Code, Commit, Push và Tạo Pull Request

### Bước 1: Thực hiện code

Viết code / thêm file trên branch của bạn trong VSCode.

### Bước 2: Commit và push lên GitHub

```bash
# Thêm file vào staging
git add <tên-file>

# Kiểm tra trạng thái
git status

# Commit với message theo convention
git commit -m "feat: add new function"

# Push branch lên GitHub
git push origin practice/thanh-tuyen
```

Kết quả `git push` thành công:
```
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Writing objects: 100% (6/6), done.
To github.com:better-bytes-academy/k18-practice.git
  af1a753..ea1a96b  practice/thanh-tuyen -> practice/thanh-tuyen
```

### Bước 3: Tạo Pull Request (PR) trên GitHub

**Cách 1:** Vào tab **Pull requests** → Click **"New pull request"**

**Cách 2:** Vào tab **Pull requests** → GitHub sẽ hiện banner vàng thông báo branch vừa push → Click **"Compare & pull request"**

**Cách 3:** Vào `github.com/<repo>/compare` → Chọn branch của bạn ở ô `compare` → Click **"Create pull request"**

---

### Bước 4: Điền thông tin PR

Khi mở form tạo PR, điền đủ 4 thông tin sau:

| Trường | Mô tả | Ví dụ |
|--------|-------|-------|
| **Title** (①) | Tiêu đề ngắn gọn, rõ ràng | `feat: add new function` |
| **Description** (②) | Mô tả chi tiết PR làm gì, cách suy nghĩ/lập luận | `- Add file class khai báo Student` `- Add file txt ghi chú tạo nhánh` |
| **Reviewers** (③) | Thêm mentor vào để review | `Pham-Uyen`, `minhphong306` |
| **Assignees** (④) | Chọn tên **bạn** (người tạo PR) | `tuyenvo224` |

> **Lưu ý về Description:** Bạn có thể mô tả thêm về cách suy nghĩ, lập luận của mình. Ví dụ: *"Tôi muốn tạo trước 1 pull request cho 2 test case đầu. Sau đó, tôi sẽ bổ sung 1 PR khác."*

Sau khi điền xong, click **"Create pull request"**.

---

### Bước 5: Tạo post trên group Facebook để xin review

Sau khi tạo PR, bạn cần đăng bài lên **group review code** và tag mentor + bạn học:

**Nội dung bài post mẫu:**
```
Git: https://github.com/playwrightvn/k9-practice/pull/75
Playwright: https://github.com/playwrightvn/k9-practice/pull/80

@Phạm Uyên @Do Minh Phong Pls help me review these PRs. Thanks
```

> Trong lớp sẽ có sắp xếp **các cặp comment chéo** — bạn review PR của bạn khác, và bạn đó review lại PR của bạn.

---

## Giai đoạn 3: Review Code cho người khác

Khi nhận được request review từ bạn học, hãy sắp xếp thời gian review — **bạn sẽ học được rất nhiều từ code của người khác**.

### Cách xem code thay đổi

Vào PR → Click tab **"Files changed"** → GitHub sẽ hiển thị toàn bộ thay đổi (dòng xanh = thêm mới, dòng đỏ = xóa).

---

### Trường hợp A: Code có vấn đề — Thêm comment

**Cách 1: Comment vào một dòng cụ thể**
1. Hover chuột vào dòng muốn comment → Xuất hiện icon **"+"** màu xanh ở đầu dòng
2. Click icon **"+"** → Hộp thoại comment mở ra
3. Điền nội dung comment
4. Click **"Add single comment"** (nếu chỉ có 1 comment) hoặc **"Start a review"** (nếu muốn gom nhiều comment)

**Cách 2: Comment vào cả file**
1. Click biểu tượng comment ở **góc phải bên trên** của mỗi file
2. Điền nội dung → Click **"Add single comment"**

**Ví dụ comment thực tế:**
- *"Biến `name` chưa được định nghĩa trong phạm vi phương thức `sayMyName`. Bạn cần sử dụng `this.name` để truy cập thuộc tính của lớp."*
- *"Format lại code"*
- *"Thêm 1 vài dòng note lưu ý thêm"*

> **Mẹo:** Reviewer có thể dùng tính năng **"Suggested change"** để đề xuất trực tiếp cách sửa — người được review có thể click "Apply suggestion" để áp dụng ngay.

---

### Cách Submit Review (gom nhiều comment lại)

Nếu bạn dùng **"Start a review"** (để gom nhiều comment), sau khi xong tất cả hãy:
1. Click **"Submit review"** ở góc trên phải
2. Điền tổng kết review (ví dụ: *"please check comment"*, *"Please fix my comment"*)
3. Chọn loại review:
   - **Comment** — Phản hồi chung, không approve hay yêu cầu sửa
   - **Approve** — Code ổn, đồng ý merge
   - **Request changes** — Yêu cầu sửa trước khi merge
4. Click **"Submit review"**

---

### Trường hợp B: Code không có vấn đề

Reply vào post review trên Facebook:

```
@<tên người request> Looks good to me
```

---

### Sau khi submit review — Thông báo cho người được review

Reply vào **post review** trên Facebook:

```
@<tên người request> Please check my comment
```

---

## Giai đoạn 4: Fix Code khi có Comment

### Trường hợp A: Comment hợp lý → Fix và resolve

**Bước 1:** Sửa code trong VSCode theo comment của reviewer

**Bước 2:** Commit và push lại lên cùng branch
```bash
git add tests/lesson6-tuyen/02-class.js
git commit -m "fix: fix bug"
git push origin practice/thanh-tuyen
```

**Bước 3:** Vào PR trên GitHub → Tìm comment đó → Click **"Resolve conversation"**

> Sau khi resolve, comment sẽ được thu gọn lại với trạng thái **"Resolved"**.

**Bước 4:** Comment lại vào thread review trên **Facebook**:
```
@<tên người review> Done
```

---

### Trường hợp B: Comment không hợp lý → Giải thích lại

Nếu bạn không đồng ý với comment của reviewer, **đừng im lặng** — hãy reply lại trong chính PR đó để giải thích:

1. Trong PR, tìm comment cần phản hồi
2. Click **"Write a reply"** → Điền giải thích của bạn
3. Click **"Add single comment"**

**Ví dụ:**
- Reviewer hỏi: *"Tại sao thêm file không có nội dung?"*
- Bạn trả lời: *"File này là file template nên để trống"*

---

## Giai đoạn 5: Merge Code

Khi tất cả comment đã được resolve và reviewer đã Approve:

**Bước 1:** Vào PR → Kiểm tra trạng thái **"Ready to merge"** (màu xanh lá)

**Bước 2:** Click **"Merge pull request"** (hoặc **"Squash and merge"** / **"Rebase and merge"** tùy quy ước lớp)

**Bước 3:** Điền commit message (GitHub tự điền sẵn, có thể giữ nguyên):
```
Merge pull request #566 from better-bytes-academy/practice/thanh-tuyen
feat: add new function
```

**Bước 4:** Click **"Confirm merge"**

> Sau khi merge thành công, code của bạn đã được đưa vào nhánh `main`. Bạn có thể kiểm tra trên GitHub tại tab **Code** — folder của bạn sẽ hiển thị commit message mới nhất.

---

## Tóm tắt toàn bộ quy trình

```
┌─────────────────────────────────────────────────────────────────────┐
│  NGƯỜI TẠO PR                          NGƯỜI REVIEW                 │
├─────────────────────────────────────────────────────────────────────┤
│  1. git checkout main                                                │
│  2. git pull origin main                                             │
│  3. git checkout -b <branch-name>                                    │
│  4. [Viết code]                                                      │
│  5. git add / git commit / git push                                  │
│  6. Tạo PR trên GitHub                                               │
│     - Điền title, description                                        │
│     - Thêm reviewer, assignee                                        │
│  7. Đăng post Facebook, tag reviewer  ──────────────────────────►   │
│                                        8. Nhận notification review  │
│                                        9. Mở tab "Files changed"    │
│                                       10. Comment vào dòng / file   │
│                                       11. Submit review              │
│                                       12. Reply Facebook: "Please   │
│  ◄──────────────────────────────────      check my comment"         │
│  13. Xem comment trên PR                                             │
│      a) Hợp lý: fix code, commit,                                    │
│         push, resolve conversation                                   │
│      b) Không hợp lý: reply giải thích                               │
│  14. Reply Facebook: "@reviewer Done"  ──────────────────────────►  │
│                                       15. Kiểm tra lại              │
│                                       16. Approve PR                │
│  ◄──────────────────────────────────                                │
│  17. Merge pull request                                              │
│  18. Code vào main ✓                                                 │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Các lệnh Git hay dùng — Bảng tra nhanh

| Mục đích | Lệnh |
|---------|------|
| Chuyển về main | `git checkout main` |
| Lấy code mới nhất | `git pull origin main` |
| Tạo branch mới | `git checkout -b <tên-branch>` |
| Xem trạng thái file | `git status` |
| Thêm file vào staging | `git add <tên-file>` |
| Commit | `git commit -m "feat: mô tả"` |
| Push lên GitHub | `git push origin <tên-branch>` |

---

## Lưu ý quan trọng

- **Trước khi tạo branch mới:** Luôn `git checkout main` + `git pull` để tránh branch bị lỗi thời so với main.
- **Khi viết title PR:** Dùng prefix `feat:`, `fix:`, `chore:` theo convention của team.
- **Khi review:** Hãy nhớ dùng **"Start a review"** thay vì comment riêng lẻ nếu có nhiều điểm cần góp ý — giúp gom tất cả lại trước khi notify người kia một lần.
- **Khi resolve comment:** Chỉ người được review (author) mới nên click "Resolve" — đừng để reviewer tự resolve comment của mình.
- **Không bao giờ merge trực tiếp vào main** mà không có PR và review.
