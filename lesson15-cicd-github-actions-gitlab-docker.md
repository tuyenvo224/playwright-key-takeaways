# CI/CD với GitHub Actions, GitLab CI và Docker - Tổng Hợp Toàn Bộ Kiến Thức

> Tổng hợp từ buổi trao đổi trong session này. Ví dụ GitHub Actions lấy từ file thật trong repo:
> [tests/students-submission/ngochanh/lesson-8/.github/workflows/playwright.yml](tests/students-submission/ngochanh/lesson-8/.github/workflows/playwright.yml).
> Ví dụ GitLab CI/Docker lấy từ ảnh chụp thực tế màn hình được chia sẻ trong buổi trao đổi.

---

## Mục lục

1. [CI/CD là gì, vì sao mentor nói chạy trên đây "khỏe hơn"](#1-cicd-là-gì-vì-sao-mentor-nói-chạy-trên-đây-khỏe-hơn)
2. [GitHub Actions chạy trên máy nào? `runs-on: ubuntu-latest` là gì](#2-github-actions-chạy-trên-máy-nào-runs-on-ubuntu-latest-là-gì)
3. [Luồng chạy chi tiết của 1 workflow GitHub Actions](#3-luồng-chạy-chi-tiết-của-1-workflow-github-actions)
4. [Push file workflow lên repo của mình thì có chạy được không?](#4-push-file-workflow-lên-repo-của-mình-thì-có-chạy-được-không)
5. [Push vào repo người khác (có quyền) - ai bị charge phí khi vượt limit?](#5-push-vào-repo-người-khác-có-quyền---ai-bị-charge-phí-khi-vượt-limit)
6. [Mỗi lần push có chạy lại toàn bộ test không?](#6-mỗi-lần-push-có-chạy-lại-toàn-bộ-test-không)
7. [Nội dung file `playwright.yml` lấy từ đâu?](#7-nội-dung-file-playwrightyml-lấy-từ-đâu)
8. [Cơ chế trigger khi tạo Pull Request](#8-cơ-chế-trigger-khi-tạo-pull-request)
9. [Nhiều PR chạy song song có ảnh hưởng nhau không?](#9-nhiều-pr-chạy-song-song-có-ảnh-hưởng-nhau-không)
10. [Push thẳng vào `main` khác gì so với Pull Request?](#10-push-thẳng-vào-main-khác-gì-so-với-pull-request)
11. [GitLab CI - cấu trúc file `.gitlab-ci.yml`](#11-gitlab-ci---cấu-trúc-file-gitlab-ciyml)
12. [Nội dung file `.gitlab-ci.yml` lấy từ đâu? Vì sao khác với docs?](#12-nội-dung-file-gitlab-ciyml-lấy-từ-đâu-vì-sao-khác-với-docs)
13. [Nên dùng template GitLab CI nào cho project thật?](#13-nên-dùng-template-gitlab-ci-nào-cho-project-thật)
14. [Docker Image là gì? Tương đương gì bên GitHub Actions?](#14-docker-image-là-gì-tương-đương-gì-bên-github-actions)
15. [Docker không chỉ là Docker Image - các khái niệm liên quan](#15-docker-không-chỉ-là-docker-image---các-khái-niệm-liên-quan)
16. [Tổng kết bằng sơ đồ và bảng so sánh](#16-tổng-kết-bằng-sơ-đồ-và-bảng-so-sánh)

---

## 1. CI/CD là gì, vì sao mentor nói chạy trên đây "khỏe hơn"

**CI (Continuous Integration)** ở đây được thể hiện qua việc: mỗi khi có code mới (push hoặc tạo Pull Request), hệ thống **tự động** cài đặt môi trường và chạy toàn bộ test suite, để phát hiện lỗi sớm mà không cần ai đó tự tay bấm chạy test trên máy local.

Mentor nói chạy trên GitHub Actions/GitLab CI "khỏe hơn" chạy local vì:

- Máy chạy CI là **máy chủ (VM/container) riêng do nền tảng cấp**, không dùng tài nguyên (CPU/RAM) của máy cá nhân bạn — máy bạn có thể tiếp tục làm việc khác song song.
- Môi trường **luôn sạch và giống nhau mỗi lần chạy** — tránh tình trạng "chạy trên máy tôi thì pass, máy bạn thì fail" do khác cấu hình, khác version phần mềm đã cài.
- Không phụ thuộc máy cá nhân có đang yếu, đang treo, hay tắt máy hay không — CI chạy trên hạ tầng của GitHub/GitLab, ổn định hơn.

---

## 2. GitHub Actions chạy trên máy nào? `runs-on: ubuntu-latest` là gì

```yaml
runs-on: ubuntu-latest
```

Dòng này nghĩa là: GitHub sẽ **tự cấp phát một máy ảo (VM) hoàn toàn mới**, chạy hệ điều hành Ubuntu (bản mới nhất mà GitHub hỗ trợ) — gọi là **GitHub-hosted runner**. Đây **không phải máy của bạn**, mà là máy chủ do GitHub sở hữu và vận hành.

Đặc điểm quan trọng của máy ảo này:

| Đặc điểm | Giải thích |
|---|---|
| Cấu hình cố định | ~4 CPU core, 16GB RAM, 14GB SSD (miễn phí với repo public) |
| **Ephemeral** (dùng 1 lần rồi huỷ) | Mỗi lần workflow được trigger, GitHub tạo VM **mới hoàn toàn sạch**, chạy xong job thì **huỷ luôn** — không giữ lại gì giữa các lần chạy |
| Do đó | Mỗi lần chạy đều phải cài lại Node, cài lại dependencies, cài lại browser từ đầu — vì VM trước hoàn toàn không còn tồn tại |

---

## 3. Luồng chạy chi tiết của 1 workflow GitHub Actions

File tham khảo thực tế trong repo:

```yaml
name: k18 automation test
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: lts/*
      - name: Install dependencies
        run: npm ci
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
      - name: Run Playwright tests
        run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: ${{ !cancelled() }}
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

Luồng chạy tuần tự:

1. **`on: push/pull_request`** → khi có push hoặc tạo/update PR nhắm vào nhánh `main`/`master`, GitHub tự động trigger workflow.
2. GitHub cấp 1 VM Ubuntu trống, sạch hoàn toàn.
3. **`actions/checkout@v4`** → clone code vào VM đó.
4. **`actions/setup-node@v4`** → cài Node.js (bản LTS) vào VM.
5. **`npm ci`** → cài dependencies đúng theo `package-lock.json`.
6. **`npx playwright install --with-deps`** → cài trình duyệt (Chromium, Firefox, WebKit) + toàn bộ thư viện hệ thống cần thiết để chạy headless. Đây là bước tốn thời gian nhất (ví dụ thực tế ~2 phút 8 giây).
7. **`npx playwright test`** → chạy test thật trên VM đó.
8. **`actions/upload-artifact@v4`** → upload report (`playwright-report/`) để xem lại, lưu trong 30 ngày.

---

## 4. Push file workflow lên repo của mình thì có chạy được không?

**Có, chạy được ngay**, chỉ cần thoả 2 điều kiện:

- File nằm đúng đường dẫn `.github/workflows/playwright.yml` (bắt buộc đúng thư mục này để GitHub nhận diện).
- Repo đã bật GitHub Actions (mặc định luôn bật).
- Nếu repo **public** → miễn phí, không giới hạn phút chạy.
- Nếu repo **private** → tài khoản cần còn phút chạy miễn phí trong tháng (Free tier ~2.000 phút/tháng cho tài khoản cá nhân).

Không cần cấu hình gì thêm ngoài file yml — GitHub tự lo phần "cấp máy chạy".

---

## 5. Push vào repo người khác (có quyền) - ai bị charge phí khi vượt limit?

**Nguyên tắc cốt lõi**: GitHub Actions tính phí (billing) dựa trên **ai sở hữu repository**, **không phải ai là người push/trigger workflow**.

Nếu bạn được thêm làm **collaborator** vào repo cá nhân của mentor, và bạn push code trigger CI:

- Phút chạy đó bị trừ vào **quota của tài khoản mentor** (chủ repo), không liên quan đến tài khoản GitHub của bạn.

Chi tiết theo loại repo:

| Loại repo | Kết quả khi vượt limit |
|---|---|
| **Public** | Miễn phí không giới hạn phút (chỉ có giới hạn "fair use" chống lạm dụng) → hầu như không có gì phải lo |
| **Private**, mentor **chưa** set spending limit (mặc định `$0`) | Workflow **không chạy được nữa** (bị chặn), **không** tự động charge tiền |
| **Private**, mentor **đã** set spending limit > `$0` (có thẻ thanh toán) | Phần vượt bị **charge vào tài khoản/thẻ của mentor**, không liên quan tài khoản bạn |

→ Kết luận: bạn hoàn toàn không bị ảnh hưởng tài chính gì, kể cả khi vượt limit — rủi ro (nếu có) chỉ nằm ở phía chủ repo.

---

## 6. Mỗi lần push có chạy lại toàn bộ test không?

**Có** — với cấu hình mặc định (`npx playwright test` không kèm filter), **mọi lần push/PR** sẽ chạy **toàn bộ** test suite, bất kể bạn sửa 1 dòng nhỏ hay sửa hàng trăm file.

Lý do: lệnh `npx playwright test` mặc định quét và chạy tất cả file test tìm được trong project. GitHub Actions/Playwright **không tự biết** "commit này chỉ ảnh hưởng tới test A" trừ khi được cấu hình thêm logic riêng.

### Các cách tối ưu (tham khảo, không bắt buộc phải áp dụng ngay)

| Cách | Tác dụng |
|---|---|
| `paths` / `paths-ignore` trong `on:` | Bỏ qua trigger toàn bộ workflow nếu commit chỉ đổi file không liên quan (vd: chỉ sửa `.md`) |
| Cache `node_modules` và Playwright browsers (`actions/cache`) | Không giảm số test chạy, nhưng giảm thời gian **cài đặt** đáng kể |
| Test sharding (`--shard=1/3`) | Chia nhỏ suite chạy song song nhiều job → nhanh hơn, nhưng vẫn chạy full suite |
| Gắn tag + `grep` theo tag liên quan | Chỉ chạy đúng nhóm test bị ảnh hưởng — phức tạp, chỉ hợp lý khi project rất lớn (hàng nghìn test) |

Với quy mô nhỏ - vừa (dưới vài trăm test), chạy full suite mỗi lần là cách làm phổ biến và an toàn (tránh bỏ sót side-effect ở chỗ khác do thay đổi nhỏ gây ra).

---

## 7. Nội dung file `playwright.yml` lấy từ đâu?

File này **không phải do mentor tự viết từ đầu** — đây là **template mặc định do Playwright tự sinh ra**. Có 2 cách để lấy được nội dung này:

1. Khi chạy lệnh khởi tạo project:
   ```
   npm init playwright@latest
   ```
   CLI sẽ hỏi *"Add a GitHub Actions workflow?"* → chọn **Yes** → Playwright tự tạo `.github/workflows/playwright.yml` với đúng nội dung chuẩn.

2. Copy trực tiếp từ **trang tài liệu chính thức**: [playwright.dev/docs/ci](https://playwright.dev/docs/ci) → mục "Setting up CI" → có sẵn đoạn yml để copy-paste.

Dấu hiệu nhận biết đây là template gốc: cấu trúc chuẩn checkout → setup-node → `npm ci` → install browsers → run test → upload artifact, đúng thứ tự Playwright khuyến nghị chính thức.

---

## 8. Cơ chế trigger khi tạo Pull Request

Khi tạo PR (nhánh A → main), GitHub Actions **không đơn thuần lấy code ở đầu nhánh A**. Bước `actions/checkout@v4` với event `pull_request` mặc định checkout một ref đặc biệt:

```
refs/pull/<số_PR>/merge
```

Đây là **commit merge tạm thời** do GitHub **tự động tạo ra**, giả lập merge nhánh A vào `main` tại thời điểm hiện tại. Nói cách khác:

> Code được test = (code nhánh A) + (code nhánh main hiện tại), merge thử với nhau

### Vậy có "đụng" tới test của main không?

- Nếu từ lúc tách nhánh A, **main có thêm test case mới** (do PR khác đã merge vào main) → những test mới đó **sẽ có mặt** trong lần chạy CI của PR bạn, vì bản merge thử đã gộp cả main mới nhất.
- Nếu main không đổi gì → merge ref ≈ chính xác code nhánh A, không khác gì.

### Vì sao thiết kế như vậy?

Để test đúng trạng thái code **sau khi merge thật**, phát hiện sớm lỗi tích hợp (integration issues) hoặc xung đột logic giữa code bạn viết và main mới nhất, trước khi bấm merge.

> **Lưu ý**: nếu PR bị conflict (không tự merge được), GitHub không tạo được `refs/pull/.../merge`, một số workflow sẽ fallback về chạy trên chính nhánh A (head ref) tuỳ cấu hình checkout.

---

## 9. Nhiều PR chạy song song có ảnh hưởng nhau không?

**Không.** Mỗi PR chạy CI **hoàn toàn độc lập**, không đụng đến code hay test case của các PR khác đang mở.

Lý do:

- Bản merge thử của PR bạn = (nhánh của bạn) + (main hiện tại) — nhưng "main hiện tại" chỉ là những gì **đã thực sự merge vào main**, không bao gồm các PR khác còn đang mở, chưa merge.
- Mỗi PR chạy trên **một VM (runner) hoàn toàn riêng biệt**, dù chạy đồng thời cùng lúc.
- Nhánh của người khác (PR riêng, chưa merge) → **không tồn tại trong main** → **không có mặt** trong bản merge thử của bạn.

### Khi nào code người khác mới thực sự "chạm" vào PR của bạn?

Chỉ khi **PR của họ đã merge thật vào main** trước PR của bạn, và:

- Nếu bạn push thêm 1 commit mới → CI trigger lại → lần này merge thử sẽ gồm cả thay đổi mới từ main.
- Nếu bạn không push gì thêm, và branch protection không bắt buộc "Require branches to be up to date before merging" → CI cũ vẫn đứng yên, không tự chạy lại.

---

## 10. Push thẳng vào `main` khác gì so với Pull Request?

Về **cơ chế trigger** (cấp VM, cài đặt, chạy full test) thì **giống nhau**. Khác biệt nằm ở **code được lấy để test**:

| | `pull_request` | `push` (thẳng vào main) |
|---|---|---|
| Code checkout | Bản **merge giả lập** (nhánh bạn + main hiện tại) | **Chính xác** commit vừa được push, không giả lập gì |
| Lý do | Code chưa thực sự nằm trên main, cần "xem trước" kết quả merge | Code **đã thực sự nằm trên main**, không cần đoán |

### Điểm dễ nhầm: merge PR = một lần push

Khi bấm **"Merge pull request"**, hành động đó bản chất **tạo ra 1 commit mới trên main** → trigger `push` event, tách biệt hoàn toàn với lần chạy `pull_request` đã chạy trước đó lúc PR còn mở.

Với file workflow khai báo cả `on: push` và `on: pull_request` cùng nhắm vào `[main, master]`, một PR merge vào main thường khiến CI **chạy 2 lần**:

1. Lần `pull_request` — khi PR còn mở, kiểm tra trước khi merge (dùng merge giả lập).
2. Lần `push` — ngay sau khi merge thật, kiểm tra lại trạng thái main mới (dùng commit thật).

Đây là hành vi mặc định, không phải lỗi cấu hình.

---

## 11. GitLab CI - cấu trúc file `.gitlab-ci.yml`

Ví dụ thực tế:

```yaml
stages:
  - test

test-playwright:   # This job runs in the test stage.
  image: mcr.microsoft.com/playwright
  stage: test      # It only starts when the job in the build stage completes successfully.
  script:
    - npm ci
    - npx playwright install --with-deps
    - npx playwright test
  artifacts:
    when: always
    paths:
      - playwright-report/**
```

Giải thích từng phần:

- **`stages: [test]`** → định nghĩa các giai đoạn (stage) và thứ tự thực thi. Ở đây chỉ có 1 stage tên `test`.
- **`test-playwright:`** → tên job tuỳ ý đặt, chạy trong stage `test`.
- **`image: mcr.microsoft.com/playwright`** → đây là điểm khác biệt lớn nhất so với GitHub Actions: job chạy **trong 1 container** khởi tạo từ Docker image chính thức của Playwright, đã có sẵn Node + browser + toàn bộ dependencies (xem chi tiết ở mục 14).
- **`script:`** → các lệnh chạy tuần tự: cài deps → cài browser (để đảm bảo khớp version) → chạy test.
- **`artifacts:`** → tương đương `upload-artifact` bên GitHub, lưu lại report để xem sau khi job chạy xong (`when: always` = lưu report dù test pass hay fail).

---

## 12. Nội dung file `.gitlab-ci.yml` lấy từ đâu? Vì sao khác với docs?

Cũng giống GitHub Actions, đây là **template chính thức từ trang tài liệu Playwright**: [playwright.dev/docs/ci#gitlab-ci](https://playwright.dev/docs/ci#gitlab-ci).

Khi đối chiếu trực tiếp trang docs hiện tại với file mentor đưa, phát hiện vài khác biệt:

| | Trang docs hiện tại | File mentor đưa |
|---|---|---|
| Tên job | `tests` | `test-playwright` |
| Image | `mcr.microsoft.com/playwright:v1.61.0-noble` (có version cụ thể) | `mcr.microsoft.com/playwright` (không ghi version → dùng bản `latest`) |

Hai khả năng giải thích:

1. **Trang docs cập nhật theo thời gian** — hiện tại khuyến nghị **pin cứng version image** để đảm bảo version browser khớp chính xác với version `@playwright/test` trong `package.json`. File mentor đưa có thể được viết từ **thời điểm cũ hơn**, khi docs chưa khuyến nghị pin version.
2. Mentor có thể đã **tự sửa lại** (đổi tên job, bỏ version tag cho đơn giản) sau khi copy từ docs.

→ Kết luận: file mentu có **gốc là template chính thức**, nhưng **không phải bản copy-paste 100% khớp byte** với phiên bản docs ở hiện tại — là 1 biến thể.

---

## 13. Nên dùng template GitLab CI nào cho project thật?

**Khuyến nghị: pin đúng version image khớp với version `@playwright/test` đang cài**, không dùng bản `latest` mơ hồ.

Lý do: image không ghi version sẽ luôn kéo bản browser **mới nhất tại thời điểm CI chạy**. Nếu bản đó không khớp version `@playwright/test` trong code, dễ gặp lỗi version mismatch — Playwright rất nhạy với việc version code và version browser trong image phải khớp nhau.

Cách kiểm tra version đang dùng trong project:

```bash
node -e "console.log(require('./node_modules/@playwright/test/package.json').version)"
```

Template khuyến nghị (kết hợp ưu điểm cả 2 bên: pin version + có sẵn artifacts):

```yaml
stages:
  - test

test-playwright:
  stage: test
  image: mcr.microsoft.com/playwright:v1.59.0-noble   # sửa version theo đúng @playwright/test trong package.json
  script:
    - npm ci
    - npx playwright install --with-deps
    - npx playwright test
  artifacts:
    when: always
    paths:
      - playwright-report/**
    expire_in: 30 days
```

Lưu ý khi áp dụng:

1. **Version image phải khớp `@playwright/test`** trong `package.json` — nếu sau này `npm update`, phải sửa lại tag image tương ứng.
2. **`-noble`** là hậu tố chỉ bản Ubuntu nền (24.04 "Noble") mà image build trên — giữ nguyên theo docs.
3. Tên job không quan trọng về kỹ thuật, đặt tuỳ ý — chỉ cần nhất quán nếu có tham chiếu ở nơi khác.
4. Nên thêm `expire_in` để tránh artifact tồn mãi chiếm dung lượng lưu trữ.

---

## 14. Docker Image là gì? Tương đương gì bên GitHub Actions?

### Docker Image là gì

Một **"bản đóng gói sẵn"** gồm: hệ điều hành (thường Linux tối giản) + toàn bộ phần mềm/thư viện/cấu hình cần thiết, đã được cài sẵn và "đóng băng" thành 1 file duy nhất. Khi "chạy" image đó, nó tạo ra một **container** — giống máy ảo siêu nhẹ, khởi động trong vài giây, có sẵn mọi thứ bên trong.

### Docker image chính thức của Playwright (`mcr.microsoft.com/playwright`)

Bên trong đã có sẵn:

- Hệ điều hành Ubuntu (bản Noble 24.04)
- Node.js
- Cả 3 trình duyệt: Chromium, Firefox, WebKit
- Toàn bộ **system dependencies** (font, codec, thư viện đồ hoạ...) mà 3 trình duyệt cần để chạy headless
- Playwright test runner đã cấu hình sẵn tương thích đúng version browser

→ Toàn bộ việc mà `npx playwright install --with-deps` phải làm đã được làm sẵn và đóng gói cứng vào image.

### So sánh với GitHub Actions

| | GitLab (`image:`) | GitHub Actions (`runs-on: ubuntu-latest`) |
|---|---|---|
| Bạn nhận được | 1 **container** từ Docker image có sẵn mọi thứ | 1 **VM trần**, chỉ có Ubuntu gốc, chưa có gì cho Playwright |
| Cần cài thêm gì? | Không cần cài browser (đã có sẵn), chỉ cần `npm ci` | Phải chạy `npx playwright install --with-deps` **mỗi lần** |
| Thời gian install browser | ~0s (đã nằm sẵn trong image) | Vài phút mỗi lần chạy (thực tế ~2m8s) |

**Bản chất**: `runs-on: ubuntu-latest` + `npx playwright install --with-deps` chính là làm thủ công những gì Docker image làm sẵn.

### GitHub Actions có tương đương Docker image không?

**Có** — dùng key `container:` thay cho chạy trực tiếp trên VM trần:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: mcr.microsoft.com/playwright:v1.59.0-noble
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx playwright test    # không cần dòng install browsers nữa
```

Khi thêm `container:`, GitHub Actions: cấp VM `ubuntu-latest` như bình thường → nhưng **thay vì chạy lệnh trực tiếp trên VM**, nó pull Docker image về, khởi tạo container **bên trong** VM đó, rồi chạy toàn bộ `steps` **trong container** đó.

### Cơ chế hoạt động cụ thể (cả 2 nền tảng)

1. Runner nhận job → thấy khai báo `image`/`container`.
2. Pull Docker image về (nếu chưa có sẵn trong cache của runner) — lần đầu chậm, các lần sau nhanh hơn vì có cache layer.
3. Tạo container mới từ image, **mount code repo vào trong** container (qua bước checkout).
4. Chạy toàn bộ lệnh `script`/`steps` **bên trong** container — mọi lệnh `npm`/`npx` thực thi trong môi trường đã có sẵn browser.
5. Job xong → container bị **huỷ hoàn toàn**, không giữ trạng thái cho lần sau.

Lợi ích: môi trường **nhất quán** (đúng version browser khớp version code) và **nhanh hơn** vì bỏ qua bước cài đặt lặp lại.

---

## 15. Docker không chỉ là Docker Image - các khái niệm liên quan

"Docker" là khái niệm **tổng** (nền tảng công nghệ containerization); "Docker Image" chỉ là **một trong nhiều khái niệm con** bên trong nó.

| Khái niệm | Là gì |
|---|---|
| **Docker Image** | Bản đóng gói sẵn (blueprint), bất biến, chứa OS + phần mềm đã cài. Vd: `mcr.microsoft.com/playwright` |
| **Dockerfile** | File văn bản chứa **công thức/hướng dẫn từng bước** để tự **build ra** 1 image riêng |
| **Docker Container** | 1 **instance đang chạy** của 1 image |
| **Docker Compose** (`docker-compose.yml`) | Công cụ + file cấu hình để **chạy nhiều container cùng lúc**, phối hợp với nhau thành 1 hệ thống |

### Ví dụ áp vào 1 project thực tế (cấu trúc thư mục GitLab)

Quan sát 1 project thật với các folder: `nginx_docker_image/`, `php_docker_image/`, `docker/`, và file `docker-compose.yml`:

- **`nginx_docker_image/`, `php_docker_image/`** → mỗi folder gần như chắc chắn chứa 1 **Dockerfile** riêng, để **tự build image tuỳ chỉnh** cho từng service (Nginx web server, PHP backend) — khác với việc chỉ "kéo" 1 image có sẵn từ Docker Hub mà không sửa gì.
- **`docker/`** → folder chung chứa config/Dockerfile phụ trợ khác (vd: entrypoint script...).
- **`docker-compose.yml`** → định nghĩa **cả một dàn nhiều container chạy cùng lúc** (PHP + Nginx + Database + ... ghép lại thành 1 hệ thống hoàn chỉnh), thường dùng để chạy môi trường dev/local hoặc staging bằng 1 lệnh `docker compose up`.

### Trả lời trực tiếp: "Docker" có phải chỉ là "Docker Image"?

**Không.** Docker là cả một **bộ khái niệm** gồm: Dockerfile (công thức build) → Image (kết quả build ra) → Container (chạy image lên) → Compose (điều phối nhiều container). Ví dụ `mcr.microsoft.com/playwright` chỉ là 1 image **có sẵn, dùng thẳng không cần build**; còn 1 project công ty thường **tự build image riêng** (qua Dockerfile) cho từng phần rồi dùng Compose để ghép lại chạy chung.

---

## 16. Tổng kết bằng sơ đồ và bảng so sánh

### Sơ đồ luồng CI tổng quát (GitHub Actions)

```
Push code / Tạo PR
        │
        ▼
GitHub trigger workflow (on: push / pull_request)
        │
        ▼
Cấp 1 VM Ubuntu mới hoàn toàn (runs-on: ubuntu-latest)
        │
        ▼
checkout code ──► setup Node ──► npm ci ──► install browsers ──► npx playwright test
        │
        ▼
Upload artifact (playwright-report/)
        │
        ▼
VM bị huỷ hoàn toàn (ephemeral)
```

### So sánh GitHub Actions vs GitLab CI

| | GitHub Actions | GitLab CI |
|---|---|---|
| File cấu hình | `.github/workflows/*.yml` | `.gitlab-ci.yml` (gốc repo) |
| Máy chạy mặc định | VM trần (`runs-on: ubuntu-latest`) | Container từ Docker image (`image: ...`) |
| Cần cài browser mỗi lần? | Có (trừ khi thêm `container:`) | Không (đã có sẵn trong image) |
| Trigger tương tự | `on: push`, `on: pull_request` | `rules:`/`only:` (không đề cập chi tiết trong session này) |
| Lưu report | `actions/upload-artifact@v4` | `artifacts:` |

### Bảng khái niệm Docker

```
Dockerfile  (công thức)
    │  docker build
    ▼
Docker Image  (bản đóng gói sẵn, bất biến)
    │  docker run
    ▼
Docker Container  (instance đang chạy)

Docker Compose = điều phối NHIỀU container (từ nhiều image) chạy cùng lúc, thành 1 hệ thống
```

### Các điểm mấu chốt cần nhớ

1. CI luôn chạy trên máy chủ **do nền tảng cấp phát**, không phải máy cá nhân — mỗi lần chạy là 1 môi trường mới, sạch hoàn toàn.
2. Billing/quota tính theo **chủ sở hữu repo**, không tính theo người trigger.
3. Mặc định luôn chạy **toàn bộ** test suite mỗi lần trigger — trừ khi tự cấu hình thêm filter/cache/sharding.
4. Pull Request test trên **bản merge giả lập** (nhánh bạn + main hiện tại), không bị ảnh hưởng bởi các PR khác chưa merge.
5. Push thẳng vào main test trên **commit thật**, không cần merge giả lập — và merge PR cũng là 1 lần push, có thể khiến CI chạy 2 lần.
6. File workflow mẫu (GitHub Actions và GitLab CI) đều có gốc từ **template chính thức của Playwright** — nên đối chiếu với version Playwright thực tế đang dùng trong `package.json` trước khi áp dụng, tránh version mismatch.
7. Docker Image chỉ là 1 phần trong bộ khái niệm Docker — thực tế các project lớn thường tự build image riêng (Dockerfile) và dùng Compose để ghép nhiều service lại.
