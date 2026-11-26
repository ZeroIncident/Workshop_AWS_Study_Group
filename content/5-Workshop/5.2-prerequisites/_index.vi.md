---
title: "Điều kiện tiên quyết"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Tài khoản và quyền

Dùng một tài khoản AWS có quyền tạo CloudFormation stack và các dịch vụ trong template. Ưu tiên IAM Identity Center/role tạm thời; không tạo access key root. Phiên triển khai đầu của dự án đã dùng root qua đăng nhập CLI tạm thời theo xác nhận của chủ tài khoản; đây là điểm cần cải thiện, không phải best practice vận hành.

Cần email thuộc quyền sở hữu để xác minh SES và email admin. Người làm lab tự dùng tài khoản/email của mình; không sử dụng tài nguyên hoặc tài khoản quản trị của bản demo công khai.

#### Công cụ

| Công cụ | Phiên bản / Mục đích |
|---|---|
| Node.js | 22.13 trở lên, khuyến nghị nhánh 22 |
| pnpm | 11.25.0 theo packageManager |
| AWS CLI | v2; cấu hình SSO hoặc phiên đăng nhập tạm thời |
| Python | 3.9+ cho đóng gói source |
| Git | Kết nối CodeCommit khi cần cập nhật |
| SAM CLI | Tùy chọn; hướng dẫn chính dùng CloudFormation package/deploy |
| Hugo | 0.134.3 để build đúng template report hiện tại |

Không cần Docker cho quy trình đóng gói bằng esbuild được dùng ở đây.

#### Lệnh kiểm tra và cấu hình

```bash
node --version
pnpm --version
aws --version
python3 --version
git --version
aws configure sso --profile aurel-lab
aws sso login --profile aurel-lab
export AWS_PROFILE=aurel-lab
export AWS_REGION=ap-southeast-1
aws sts get-caller-identity
```

Giữ kết quả danh tính trên máy; khi đưa vào báo cáo chỉ cần role và region đã che thông tin tài khoản. Không chụp access key, session token hoặc mật khẩu.

#### Kiểm tra quota và ngân sách

Tài khoản triển khai đã có concurrency Lambda là 10. Không đặt reserved concurrency bằng toàn bộ quota vì AWS còn yêu cầu phần unreserved. Throttle API mặc định 10 request/giây, burst 20 không tự bảo đảm Lambda không throttle. Trước lab, xem quota thực tế của tài khoản và đặt AWS Budget phù hợp; chưa có bằng chứng Budget đã được tạo trong dự án.

#### Liên kết tài liệu tham khảo

- [Tài liệu IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Đăng nhập SSO qua CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
