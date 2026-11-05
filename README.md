# HCM-KS26-CNTT1_NhapMonCNTT_Session20_Ex06
# BÀI 6: NGĂN CHẶN RÒ RỈ THÔNG TIN BÍ MẬT TRÊN GITHUB

**Họ và tên:** Phạm Thành Đạo  
**Lớp:** HCM-KS26-CNTT1  
**Môn học:** Nhập Môn CNTT  
**Tên repository:** `HCM-KS26-CNTT1_NhapMonCNTT_Session19_Ex06`

---

## 1. Phân tích vấn đề và mục tiêu bảo mật

### 1.1. Bối cảnh

Team phát triển chuẩn bị đưa mã nguồn dự án thương mại điện tử lên GitHub Public để đối tác kiểm tra. Trong quá trình phát triển, mã nguồn có thể chứa những thông tin nhạy cảm như:

- API Key của các dịch vụ bên thứ ba.
- Database Username và Password.
- Secret Key dùng để xác thực ứng dụng.
- Access Token và Refresh Token.
- Thông tin kết nối đến máy chủ và dịch vụ thanh toán.
- Khóa bí mật của các dịch vụ lưu trữ hoặc gửi email.

Nếu những thông tin này bị đưa lên repository Public, bất kỳ ai cũng có thể xem và khai thác chúng. Hacker có thể sử dụng API Key để truy cập dịch vụ, lấy dữ liệu khách hàng, truy cập cơ sở dữ liệu hoặc phát sinh chi phí trái phép.

### 1.2. Mục tiêu

Xây dựng quy trình bảo vệ thông tin bí mật trước khi đưa mã nguồn lên GitHub, bao gồm:

1. Xây dựng Pre-push Checklist để kiểm tra mã nguồn trước khi push.
2. Sử dụng biến môi trường thay cho việc hardcode API Key.
3. Tạo tệp `.env.example` làm mẫu cấu hình cho môi trường Dev, Test và Production.
4. Sử dụng `.gitignore` để ngăn các tệp chứa thông tin nhạy cảm bị đưa vào Git.
5. Sử dụng công cụ quét secret và Push Protection để phát hiện, ngăn chặn rò rỉ.
6. Bắt buộc thành viên tham gia repository sử dụng xác thực hai yếu tố (2FA).

## 2. Xây dựng Pre-push Checklist

### 2.1. Mục đích

Pre-push Checklist là danh sách các bước kiểm tra bắt buộc trước khi đưa mã nguồn lên GitHub.

Mục tiêu là phát hiện và loại bỏ những thông tin nhạy cảm trước khi chúng xuất hiện trong repository, đặc biệt là các API Key, mật khẩu và token bị hardcode.

### 2.2. Bảng kiểm tra trước khi Push

| STT | Nội dung kiểm tra | Yêu cầu |
|---|---|---|
| 1 | Kiểm tra API Key | Không được hardcode API Key trong mã nguồn |
| 2 | Kiểm tra mật khẩu | Không chứa Database Password hoặc tài khoản quản trị thật |
| 3 | Kiểm tra tệp `.env` | Không được đưa `.env` lên GitHub |
| 4 | Kiểm tra `.gitignore` | Đã khai báo các tệp và thư mục nhạy cảm |
| 5 | Kiểm tra `.env.example` | Chỉ chứa tên biến và giá trị mẫu, không chứa secret thật |
| 6 | Kiểm tra Git Staging | Không có tệp nhạy cảm nằm trong danh sách chuẩn bị commit |
| 7 | Quét mã nguồn | Sử dụng Gitleaks hoặc công cụ tương đương |
| 8 | Kiểm tra lịch sử Git | Không có secret từng được commit |
| 9 | Kiểm tra GitHub Security | Đã bật Secret Scanning và Push Protection nếu khả dụng |
| 10 | Kiểm tra xác thực | Thành viên tham gia repository đã bật 2FA |

### 2.3. Quy trình thực hiện

**Bước 1: Kiểm tra các tệp đang thay đổi**

Sử dụng Terminal tại thư mục dự án:

```bash
git status
```

Lệnh này giúp xác định những tệp đã chỉnh sửa, những tệp đang được đưa vào vùng Staging và các tệp chưa được Git theo dõi.

**Bước 2: Kiểm tra nội dung chuẩn bị commit**

```bash
git diff --cached
```

Lệnh này hiển thị nội dung đã được đưa vào Staging, giúp phát hiện API Key hoặc mật khẩu vô tình xuất hiện trong mã nguồn.

**Bước 3: Kiểm tra các tệp được Git bỏ qua**

```bash
git check-ignore -v .env
```

Nếu `.env` được cấu hình đúng trong `.gitignore`, Git sẽ hiển thị quy tắc khiến tệp này bị bỏ qua.

Lưu ý: Nếu tệp `.env` đã được Git theo dõi từ trước, `.gitignore` không tự động loại bỏ tệp đó khỏi repository.

**Bước 4: Quét mã nguồn bằng Gitleaks**

Cài đặt Gitleaks trên macOS bằng Homebrew:

```bash
brew install gitleaks
```

Quét mã nguồn và lịch sử Git:

```bash
gitleaks git .
```

Quét các tệp trong thư mục hiện tại:

```bash
gitleaks dir .
```

Nếu phát hiện thông tin nhạy cảm, cần kiểm tra kết quả, loại bỏ secret khỏi mã nguồn và thay thế bằng biến môi trường.

**Bước 5: Kiểm tra lần cuối trước khi Push**

```bash
git status
git diff --cached
gitleaks git .
```

Chỉ thực hiện push khi đã kiểm tra và xử lý các cảnh báo hợp lệ.

### 2.4. Checklist xác nhận của nhóm

- [ ] Đã kiểm tra API Key và các thông tin đăng nhập.
- [ ] Không hardcode mật khẩu trong mã nguồn.
- [ ] Đã tạo `.gitignore`.
- [ ] Đã kiểm tra `.env` không bị Git theo dõi.
- [ ] Đã tạo `.env.example` không chứa thông tin thật.
- [ ] Đã kiểm tra nội dung bằng `git diff --cached`.
- [ ] Đã quét mã nguồn bằng Gitleaks.
- [ ] Đã kiểm tra lịch sử Git.
- [ ] Đã bật các tính năng bảo vệ secret trên GitHub nếu khả dụng.
- [ ] Thành viên tham gia đã bật 2FA.

## 3. Thiết lập cấu trúc tệp .env.example cho Dev, Test và Production

### 3.1. Vì sao cần sử dụng biến môi trường?

Không nên viết trực tiếp API Key hoặc mật khẩu vào mã nguồn.

Ví dụ không an toàn:

```javascript
const API_KEY = "sk_live_123456789";
const DB_PASSWORD = "MyDatabasePassword123";
```

Đây là cách làm nguy hiểm vì thông tin bí mật có thể bị đưa lên GitHub cùng với mã nguồn.

Cách xử lý an toàn hơn:

```javascript
const API_KEY = process.env.API_KEY;
const DB_PASSWORD = process.env.DB_PASSWORD;

if (!API_KEY || !DB_PASSWORD) {
    throw new Error("Missing required environment variables");
}
```

Mã nguồn chỉ đọc giá trị từ biến môi trường, không chứa thông tin bí mật thật.

### 3.2. Cấu trúc tệp .env.example

Tạo tệp `.env.example` tại thư mục gốc dự án:

```env
# ==========================================
# E-COMMERCE PROJECT - ENVIRONMENT TEMPLATE
# ==========================================

# Application
NODE_ENV=development
PORT=3000

# Database
DB_HOST=localhost
DB_PORT=3306
DB_NAME=YOUR_DATABASE_NAME
DB_USER=YOUR_DATABASE_USER
DB_PASSWORD=YOUR_DATABASE_PASSWORD

# API Configuration
API_KEY=YOUR_API_KEY
API_SECRET=YOUR_API_SECRET

# Authentication
JWT_SECRET=REPLACE_WITH_A_RANDOM_SECRET
JWT_EXPIRES_IN=1h

# Payment Gateway
PAYMENT_API_KEY=YOUR_PAYMENT_API_KEY
PAYMENT_WEBHOOK_SECRET=YOUR_WEBHOOK_SECRET

# Email Service
SMTP_HOST=YOUR_SMTP_HOST
SMTP_PORT=587
SMTP_USER=YOUR_SMTP_USERNAME
SMTP_PASSWORD=YOUR_SMTP_PASSWORD
```

**Giải thích:**

- Đây là tệp mẫu để các thành viên biết dự án cần những biến môi trường nào.
- Các giá trị `YOUR_...` chỉ là placeholder, không phải thông tin đăng nhập thật.
- Không đưa API Key, Database Password hoặc JWT Secret thật vào tệp này.
- Khi tạo môi trường làm việc, mỗi thành viên tự thiết lập giá trị phù hợp.

### 3.3. Phân chia cấu hình theo môi trường

Dự án cần ba môi trường hoạt động:

| Tiêu chí | Development (Dev) | Testing (Test) | Production (Prod) |
|---|---|---|---|
| Mục đích | Phát triển tính năng | Kiểm thử ứng dụng | Vận hành thực tế |
| Database | Database local | Database kiểm thử riêng | Database chính thức |
| API Key | Khóa thử nghiệm | Khóa thử nghiệm riêng | Khóa Production |
| Dữ liệu | Dữ liệu giả lập | Dữ liệu kiểm thử | Dữ liệu thực tế |
| Quyền truy cập | Nhà phát triển | Nhóm kiểm thử | Người được phân quyền |
| Bảo mật | Không dùng secret thật | Tách biệt với Production | Kiểm soát nghiêm ngặt |

**Nguyên tắc quản lý:**

- Dev: sử dụng thông tin thử nghiệm trên máy cá nhân.
- Test: sử dụng tài khoản, database và API Key riêng cho kiểm thử.
- Production: sử dụng secret thật được cấp thông qua hệ thống quản lý bí mật như GitHub Actions Secrets hoặc dịch vụ quản lý secret chuyên dụng.
- Không sử dụng chung một API Key cho cả ba môi trường.

Tệp `.env.example` có thể dùng chung làm danh sách biến cần thiết. Các giá trị thật được cung cấp riêng cho từng môi trường, không đưa lên GitHub.

### 3.4. Cấu trúc thư mục đề xuất

```text
ecommerce-project/
│
├── src/
│   ├── config/
│   ├── controllers/
│   ├── models/
│   └── routes/
│
├── docs/
│   └── security-report.md
│
├── .env.example
├── .gitignore
├── package.json
├── package-lock.json
└── README.md
```

Các tệp `.env` chứa thông tin thật chỉ tồn tại trong môi trường triển khai hoặc máy cá nhân và không được đưa vào repository.

## 4. Thiết lập tệp .gitignore

### 4.1. Vai trò của .gitignore

`.gitignore` là tệp cấu hình giúp Git bỏ qua những tệp hoặc thư mục không cần đưa vào repository.

Đối với dự án thương mại điện tử, tệp này đặc biệt quan trọng vì có thể chứa các tệp cấu hình môi trường, dữ liệu nội bộ, log và thông tin xác thực.

### 4.2. Nội dung .gitignore đề xuất

Tạo tệp `.gitignore` tại thư mục gốc:

```gitignore
# Environment files
.env
.env.*
!.env.example

# Dependencies
node_modules/

# Build output
dist/
build/

# Logs
*.log
logs/

# Private keys and certificates
*.pem
*.key
*.p12
*.pfx

# Local database and backup files
*.sqlite
*.sqlite3
*.db
*.sql
*.bak

# Operating system files
.DS_Store
Thumbs.db

# IDE and editor
.vscode/
.idea/

# Temporary files
*.tmp
*.temp
```

**Giải thích các quy tắc quan trọng:**

- `.env`: Bỏ qua tệp cấu hình chứa thông tin môi trường.
- `.env.*`: Bỏ qua các tệp môi trường như `.env.development`, `.env.test`, `.env.production`.
- `!.env.example`: Cho phép đưa tệp mẫu `.env.example` lên GitHub.
- `node_modules/`: Không đưa thư viện phụ thuộc đã cài đặt vào repository.
- `*.pem`, `*.key`: Ngăn các tệp khóa riêng và chứng chỉ có khả năng bị đưa lên Git.

Lưu ý: Các quy tắc trên là cấu hình tham khảo. Nếu dự án cần quản lý chứng chỉ hoặc dữ liệu mẫu hợp lệ, phải xem xét từng tệp trước khi bỏ qua.

### 4.3. Xử lý khi tệp .env đã được Git theo dõi

Nếu tệp `.env` đã được đưa vào Git trước khi cấu hình `.gitignore`, chỉ thêm tên tệp vào `.gitignore` là chưa đủ.

Có thể sử dụng:

```bash
git rm --cached .env
```

Sau đó:

```bash
git add .gitignore
git commit -m "Remove env file from tracking"
```

Lệnh `git rm --cached` chỉ loại bỏ tệp khỏi vùng theo dõi của Git, không xóa tệp trên máy tính.

**Trường hợp nghiêm trọng:** Nếu API Key hoặc mật khẩu thật đã từng được push lên GitHub Public, cần:

1. Thu hồi hoặc vô hiệu hóa API Key đã lộ.
2. Tạo API Key mới và cập nhật vào hệ thống.
3. Đổi mật khẩu database hoặc thông tin xác thực liên quan.
4. Kiểm tra log truy cập để phát hiện hoạt động bất thường.
5. Nếu cần thiết, phối hợp xóa secret khỏi lịch sử Git bằng công cụ như `git-filter-repo`.
6. Thông báo cho các thành viên để xử lý những bản sao repository cũ.

Không được cho rằng chỉ xóa tệp trên commit mới nhất là đã giải quyết xong sự cố.

## 5. Kích hoạt xác thực hai yếu tố (2FA) cho mọi thành viên

### 5.1. Tại sao cần bắt buộc 2FA?

Nếu chỉ sử dụng mật khẩu, hacker có thể chiếm quyền tài khoản GitHub khi mật khẩu bị lộ qua Phishing, Brute Force hoặc Credential Stuffing.

Khi kích hoạt 2FA, người dùng cần cung cấp thêm yếu tố xác thực thứ hai bên cạnh mật khẩu.

Các phương thức nên ưu tiên:

- Passkeys.
- Security Keys (khóa bảo mật vật lý).
- Ứng dụng Authenticator sử dụng TOTP.
- GitHub Mobile.

SMS không nên là phương thức bảo vệ duy nhất nếu có thể sử dụng các phương thức an toàn hơn.

### 5.2. Hướng dẫn thành viên bật 2FA

**Bước 1:** Đăng nhập vào GitHub.

Truy cập:

https://github.com/settings/security

**Bước 2:** Tìm mục Two-factor authentication.

**Bước 3:** Chọn thiết lập phương thức xác thực phù hợp, ưu tiên ứng dụng Authenticator, Passkey hoặc Security Key.

**Bước 4:** Hoàn thành quá trình đăng ký và kiểm tra mã xác thực.

**Bước 5:** Tải và lưu Recovery Codes ở nơi an toàn, độc lập với thiết bị đăng nhập.

**Bước 6:** Đăng xuất và thử đăng nhập lại để xác nhận 2FA hoạt động bình thường.

### 5.3. Cách quản trị viên bắt buộc 2FA cho tổ chức

Để thực sự yêu cầu mọi thành viên và cộng tác viên phải bật 2FA, nên quản lý repository thông qua GitHub Organization thay vì chỉ sử dụng tài khoản cá nhân.

Quản trị viên thực hiện:

1. Truy cập GitHub và mở Organization của dự án.
2. Chọn mục **Settings**.
3. Trong phần Security, chọn **Authentication security**.
4. Tìm mục **Two-factor authentication**.
5. Bật tùy chọn **Require two-factor authentication for everyone in your organization**.
6. Xem thông báo ảnh hưởng đến những thành viên chưa bật 2FA.
7. Xác nhận để áp dụng chính sách.

Có thể bật thêm tùy chọn chỉ cho phép phương thức 2FA an toàn nếu gói và cấu hình tổ chức hỗ trợ.

**Lưu ý:** Tính năng bắt buộc 2FA ở cấp Organization áp dụng cho thành viên, outside collaborators và các đối tượng được GitHub hỗ trợ theo chính sách. Nếu repository thuộc tài khoản cá nhân, không có công tắc cấp repository để bắt buộc tất cả cộng tác viên phải bật 2FA. Vì vậy, nên chuyển dự án nhóm sang Organization để quản trị tập trung.

### 5.4. Quy định quản lý tài khoản

| Đối tượng | Yêu cầu bảo mật |
|---|---|
| Owner | Bắt buộc bật 2FA |
| Administrator | Bắt buộc bật 2FA |
| Developer | Bắt buộc bật 2FA |
| Tester | Bắt buộc bật 2FA |
| Outside Collaborator | Bắt buộc bật 2FA |
| Bot/Service Account | Có phương thức xác thực phù hợp và quản lý riêng |

Quản trị viên cần kiểm tra danh sách thành viên định kỳ, thu hồi quyền truy cập của tài khoản không còn tham gia dự án và không chia sẻ tài khoản GitHub giữa nhiều người.

## 6. Báo cáo đánh giá rủi ro nếu thiếu .gitignore

### 6.1. Các rủi ro có thể xảy ra

| STT | Rủi ro | Hậu quả |
|---|---|---|
| 1 | Tệp `.env` bị đưa lên GitHub Public | Hacker lấy được API Key và thông tin đăng nhập |
| 2 | Database Password bị lộ | Có thể bị truy cập trái phép hoặc đánh cắp dữ liệu khách hàng |
| 3 | Private Key bị công khai | Có nguy cơ mất quyền kiểm soát tài khoản hoặc dịch vụ liên quan |
| 4 | Log chứa thông tin nhạy cảm bị push | Lộ dữ liệu người dùng, token hoặc thông tin phiên đăng nhập |
| 5 | Tệp backup database bị công khai | Dữ liệu thương mại điện tử có thể bị tải xuống và khai thác |
| 6 | Tệp cấu hình nội bộ bị rò rỉ | Lộ kiến trúc hệ thống và thông tin triển khai |
| 7 | Thông tin bí mật xuất hiện trong lịch sử Git | Việc xóa ở commit mới không loại bỏ được bản ghi cũ |

### 6.2. Phân tích một tình huống thực tế

Giả sử lập trình viên đưa tệp `.env` lên GitHub Public với nội dung chứa:

```env
DB_HOST=database.example.com
DB_USER=admin
DB_PASSWORD=RealPasswordExample
PAYMENT_API_KEY=RealPaymentKeyExample
```

Khi hacker phát hiện repository, họ có thể:

1. Tải mã nguồn và đọc tệp `.env`.
2. Sử dụng thông tin database để thử truy cập cơ sở dữ liệu.
3. Sử dụng API Key để truy cập dịch vụ thanh toán.
4. Khai thác dữ liệu khách hàng hoặc thực hiện các yêu cầu trái phép.
5. Gây thiệt hại tài chính, ảnh hưởng đến uy tín và hoạt động kinh doanh của công ty.

Các giá trị trên chỉ là ví dụ minh họa, không phải thông tin đăng nhập thật.

### 6.3. Đánh giá mức độ rủi ro

| Tình huống | Mức độ rủi ro | Lý do |
|---|---|---|
| Không có `.gitignore` | Cao | Dễ đưa nhầm tệp nhạy cảm vào Git |
| Có `.gitignore` nhưng vẫn hardcode API Key | Cao | Secret vẫn có thể xuất hiện trong mã nguồn |
| Có `.gitignore` nhưng secret đã được commit trước đó | Rất cao | Secret vẫn nằm trong lịch sử Git |
| Có `.gitignore` và sử dụng biến môi trường | Thấp hơn | Giảm nguy cơ lộ thông tin trong mã nguồn |
| Có `.gitignore`, Gitleaks, Push Protection và 2FA | Tốt hơn | Có nhiều lớp phòng vệ và kiểm soát truy cập |

**Nhận xét:** `.gitignore` là một lớp phòng vệ cần thiết nhưng không phải giải pháp bảo mật tuyệt đối. Nó không thể phát hiện mọi secret được hardcode trong mã nguồn và cũng không tự xóa những tệp đã được Git theo dõi.

## 7. Tăng cường bảo mật bằng GitHub Secret Scanning và Push Protection

Ngoài các biện pháp trên, nhóm nên sử dụng tính năng bảo mật của GitHub.

### 7.1. Secret Scanning

Secret Scanning giúp phát hiện các thông tin xác thực có nguy cơ bị lộ trong repository.

Khi phát hiện secret, GitHub có thể tạo cảnh báo để quản trị viên kiểm tra và xử lý.

### 7.2. Push Protection

Push Protection giúp ngăn các lần push có chứa những loại secret được hỗ trợ trước khi chúng được đưa vào repository.

Nếu GitHub phát hiện một secret, hệ thống có thể chặn thao tác push và yêu cầu người dùng kiểm tra, xử lý.

### 7.3. Quy trình xử lý khi Push Protection cảnh báo

1. Dừng thao tác push.
2. Xác định tệp và dòng mã bị cảnh báo.
3. Xóa secret khỏi mã nguồn.
4. Thay thế bằng biến môi trường.
5. Kiểm tra lại bằng Gitleaks.
6. Thử push lại sau khi xử lý.

Không được bỏ qua cảnh báo chỉ để hoàn thành thao tác push nhanh hơn.

## 8. Kết luận

Việc đưa mã nguồn dự án thương mại điện tử lên GitHub Public có thể gây ra rủi ro nghiêm trọng nếu không kiểm soát thông tin bí mật.

Để giảm thiểu nguy cơ rò rỉ dữ liệu, nhóm cần thực hiện đồng thời các biện pháp:

- Sử dụng Pre-push Checklist trước mỗi lần push.
- Không hardcode API Key và mật khẩu trong mã nguồn.
- Sử dụng `.env.example` để hướng dẫn cấu hình môi trường.
- Thiết lập `.gitignore` để loại bỏ các tệp chứa thông tin nhạy cảm.
- Quét mã nguồn bằng Gitleaks.
- Bật Secret Scanning và Push Protection nếu khả dụng.
- Bắt buộc thành viên và cộng tác viên sử dụng 2FA thông qua chính sách Organization.

**Kết luận quan trọng:** Bảo mật thông tin trên GitHub không chỉ là việc tạo một tệp `.gitignore`, mà cần xây dựng quy trình kiểm tra, phát hiện, ngăn chặn và xử lý rò rỉ thông tin trong toàn bộ vòng đời phát triển phần mềm.

---


### Nội dung README.md đề xuất

```markdown
# E-Commerce Security Repository

## Mục tiêu
Ngăn chặn rò rỉ thông tin bí mật khi đưa mã nguồn lên GitHub.

## Các biện pháp bảo mật
- Sử dụng .gitignore.
- Sử dụng .env.example.
- Không hardcode API Key.
- Kiểm tra bằng Gitleaks.
- Sử dụng GitHub Secret Scanning.
- Kích hoạt Push Protection.
- Bắt buộc 2FA cho thành viên.

## Hướng dẫn cài đặt

1. Clone repository.
2. Sao chép .env.example thành .env.
3. Điền cấu hình môi trường hợp lệ.
4. Cài đặt dependencies.
5. Chạy ứng dụng.

## Kiểm tra bảo mật

gitleaks git .

## Quy tắc
Không commit hoặc push:
- .env
- API Key thật
- Database Password thật
- Private Key
- Recovery Codes
```
