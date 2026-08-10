# Lesson 06-01 - Git: HTTPS vs SSH khi Clone/Push

## Mục lục
1. [Clone repo PUBLIC bằng HTTPS vs SSH](#1-clone-repo-public-bằng-https-vs-ssh)
2. [Clone repo PRIVATE bằng HTTPS vs SSH](#2-clone-repo-private-bằng-https-vs-ssh)
3. [Cơ chế Collaborator - Push lên repo của người khác](#3-cơ-chế-collaborator---push-lên-repo-của-người-khác)
4. [SSH Key là gì và hoạt động như thế nào](#4-ssh-key-là-gì-và-hoạt-động-như-thế-nào)

---

## 1. Clone repo PUBLIC bằng HTTPS vs SSH

### HTTPS
```bash
git clone https://github.com/user/repo.git
```

### SSH
```bash
git clone git@github.com:user/repo.git
```

### So sánh với repo PUBLIC

| Tính năng | HTTPS | SSH |
|---|---|---|
| **Clone** (chỉ đọc) | Không cần auth | Không cần auth |
| **Push** | Cần Personal Access Token (PAT) | Cần SSH key |
| **Setup** | Không cần setup gì | Phải tạo SSH key trước |
| **Firewall/Proxy** | Qua được hầu hết | Có thể bị block port 22 |

> **Kết luận:** Với repo public, clone bằng cả hai cách đều không cần xác thực. Sự khác biệt chỉ quan trọng khi **push code**.

---

## 2. Clone repo PRIVATE bằng HTTPS vs SSH

Với repo private, **clone lẫn push đều yêu cầu xác thực**.

### HTTPS — dùng Personal Access Token (PAT)

```bash
git clone https://github.com/user/private-repo.git
# GitHub sẽ hỏi username + PAT (không dùng password thường từ 2021)
```

Hoặc nhúng token thẳng vào URL (không khuyến khích vì dễ lộ token):
```bash
git clone https://<YOUR_TOKEN>@github.com/user/private-repo.git
```

**Cách tạo PAT:** GitHub → Settings → Developer settings → Personal access tokens

### SSH — dùng SSH Key pair

```bash
git clone git@github.com:user/private-repo.git
# Tự động xác thực qua SSH key, không hỏi password
```

**Cách setup SSH:**
```bash
# Bước 1: Tạo SSH key pair trên máy
ssh-keygen -t ed25519 -C "your_email@example.com"

# Bước 2: Copy public key
cat ~/.ssh/id_ed25519.pub

# Bước 3: Dán public key lên GitHub
# GitHub → Settings → SSH and GPG keys → New SSH key
```

### So sánh với repo PRIVATE

| Tính năng | HTTPS | SSH |
|---|---|---|
| **Clone** | Cần PAT | Cần SSH key |
| **Push** | Cần PAT | Cần SSH key |
| **Setup** | Tạo PAT trên GitHub | Tạo key pair + add public key lên GitHub |
| **Bảo mật** | Token có thể lộ nếu lưu sai chỗ | Private key không rời khỏi máy |
| **Tiện lợi hàng ngày** | Phải quản lý token (hết hạn, rotate) | Setup 1 lần, dùng mãi |
| **Token hết hạn** | PAT có thể hết hạn | SSH key không hết hạn |

> **Lưu ý quan trọng:** Có SSH key trên máy **chưa đủ** để clone repo private của người khác. SSH key chỉ xác minh danh tính bạn (tầng 1), nhưng GitHub còn kiểm tra quyền truy cập (tầng 2). Nếu chưa được owner add làm Collaborator, GitHub sẽ từ chối dù bạn có SSH key hợp lệ.

### Khi nào dùng cái nào?

| Trường hợp | Khuyến nghị |
|---|---|
| Dev cá nhân, làm việc hàng ngày | **SSH** — tiện hơn, setup 1 lần |
| CI/CD, automation, server | **HTTPS + PAT** — dễ quản lý, scope quyền chặt, rotate khi cần |

---

## 3. Cơ chế Collaborator - Push lên repo của người khác

### Tại sao có thể clone/push repo của người khác?

GitHub xác thực theo **2 tầng độc lập**:

```
Tầng 1 — Xác thực danh tính:
  "Mày là ai?" → SSH key hoặc PAT xác nhận bạn là account nào

Tầng 2 — Kiểm tra quyền:
  "Mày có quyền gì trên repo này?" → Collaborator settings quyết định
```

Khi owner add bạn làm Collaborator với quyền **Write**:
- GitHub lưu vào database: *"account X được phép đọc/ghi repo Y"*
- Từ lúc đó, mọi request từ account X đến repo Y đều được chấp thuận

### Flow hoàn chỉnh khi push

```
git push
    │
    ▼
SSH handshake
    │  Máy bạn ký bằng private key
    │  GitHub verify bằng public key của bạn (đã đăng ký)
    │
    ▼
GitHub biết: "Đây là account vuthaotester1122"
    │
    ▼
GitHub kiểm tra Collaborator list của repo k18-practice
    │
    ▼
"vuthaotester1122 có quyền Write" → Cho phép push ✓
```

### Các mức quyền Collaborator

| Quyền | Đọc | Push | Manage settings |
|---|---|---|---|
| **Read** | ✓ | ✗ | ✗ |
| **Write** | ✓ | ✓ | ✗ |
| **Admin** | ✓ | ✓ | ✓ |

---

## 4. SSH Key là gì và hoạt động như thế nào

### SSH Key là một cặp key (key pair)

```
┌─────────────────────────────┐     ┌──────────────────────────────┐
│       MÁY CỦA BẠN           │     │           GITHUB             │
│                             │     │                              │
│  Private Key (bí mật)       │     │  Public Key                  │
│  ~/.ssh/id_ed25519          │     │  (đăng ký tại               │
│                             │     │   Settings > SSH Keys)       │
│  KHÔNG bao giờ chia sẻ!     │     │  Ai cũng có thể biết         │
└─────────────────────────────┘     └──────────────────────────────┘
```

- **Private key**: Giữ bí mật tuyệt đối trên máy của bạn
- **Public key**: Upload lên GitHub, không cần giữ bí mật

### Cơ chế xác thực SSH (đơn giản hóa)

```
1. Bạn: git clone git@github.com:user/repo.git

2. GitHub: "Chứng minh mày là chủ của public key đã đăng ký đi"
           (gửi một bài toán mã hóa)

3. Máy bạn: Dùng private key để giải bài toán → gửi kết quả lại

4. GitHub: Dùng public key verify kết quả
           → Đúng! → "Đây là account của bạn" → Cho phép truy cập
```

> **Quan trọng:** SSH key của bạn là của **account GitHub của bạn**, không phải của repo. Một SSH key có thể dùng để truy cập nhiều repo khác nhau, miễn là account bạn có quyền trên các repo đó.

### SSH key trong bối cảnh Collaborator

Khi được add vào repo của người khác:
- Bạn dùng **SSH key của chính bạn** (không phải của owner)
- GitHub dùng SSH key đó để xác định đây là **account của bạn**
- Sau đó check xem account bạn có trong Collaborator list không

```
SSH Key của bạn ──→ Xác nhận danh tính bạn ──→ Check quyền trên repo ──→ Cho phép/Từ chối
```

---

## Tóm tắt nhanh

| Câu hỏi | Trả lời |
|---|---|
| Clone repo public cần auth không? | Không, cả HTTPS lẫn SSH |
| Clone repo private cần gì? | HTTPS: cần PAT / SSH: cần SSH key |
| Push repo public cần gì? | HTTPS: cần PAT / SSH: cần SSH key |
| Được add Collaborator thì làm được gì? | Tùy quyền (Read/Write/Admin) |
| SSH key khi là Collaborator là key của ai? | Của **bạn**, không phải của owner |
| Tại sao push được repo người khác? | Vì GitHub xác thực danh tính bạn → check quyền → có quyền Write → cho phép |
