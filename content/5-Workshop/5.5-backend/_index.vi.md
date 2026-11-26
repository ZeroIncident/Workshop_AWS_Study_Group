---
title: "Bước 2 — Triển khai backend AWS"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Tạo artifacts và backend

Các lệnh bên dưới dùng shell macOS/Linux. Trên Windows dùng WSL/Git Bash hoặc chuyển cú pháp biến sang PowerShell. Thay email trước khi chạy. Không đưa AWS key hoặc mật khẩu vào source. Tên stack dành cho tài khoản lab riêng; kiểm tra không trùng dự án khác.



```bash
export AWS_PROFILE=aurel-lab
export AWS_REGION=ap-southeast-1
export SENDER_EMAIL='YOUR_OWN_VERIFIED_EMAIL'
pnpm build:lambda
aws cloudformation deploy --stack-name aurel-artifacts \
  --template-file infra/artifacts.yaml
export ARTIFACT_BUCKET=$(aws cloudformation describe-stacks \
  --stack-name aurel-artifacts \
  --query 'Stacks[0].Outputs[?OutputKey==`BucketName`].OutputValue | [0]' \
  --output text)
aws cloudformation package --template-file infra/template.yaml \
  --s3-bucket "$ARTIFACT_BUCKET" --s3-prefix lambda \
  --output-template-file packaged-backend.yaml
aws cloudformation deploy --stack-name aurel-backend \
  --template-file packaged-backend.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND \
  --parameter-overrides FrontendOrigin=https://pending.invalid \
    SenderEmail="$SENDER_EMAIL"
aws cloudformation describe-stacks --stack-name aurel-backend \
  --query 'Stacks[0].Outputs' --output table
```

Sau khi tạo SES identity, mở thư xác minh AWS gửi đến email của bạn. Stack có thể chờ xác minh; nếu không hoàn thành, xem CloudFormation Events thay vì tạo stack mới liên tục. Origin pending.invalid là tham số khởi tạo, sẽ được thay bằng URL Amplify ở bước 3.

#### Nạp dữ liệu và tạo admin

Lấy TableName, ApiUrl, MediaBucket, UserPoolId, CognitoClientId từ Outputs. Dùng chính giá trị tài khoản lab, không sao chép IDs của bản demo.

```bash
export TABLE_NAME=YOUR_TABLE_OUTPUT
node dist/seed-aws.mjs
```

Seed chỉ thêm món chưa tồn tại, giữ món đã sửa và có thể chạy lại. Bảng mới sẽ có 100 món, không có dữ liệu đặt bàn của bản demo.

Dùng script trong bootstrap-admin.py: script yêu cầu mật khẩu bằng getpass, tạo user không gửi thư mời, thêm nhóm admin và đặt mật khẩu vĩnh viễn để phù hợp luồng BFF hiện tại. Không có tài khoản tự đăng ký. File JSON mật khẩu tạm có quyền hạn chế và được xóa sau lệnh; trên Windows cần chạy trong thư mục cá nhân có ACL phù hợp.

```bash
python3 bootstrap-admin.py --pool YOUR_USER_POOL_ID --email YOUR_ADMIN_EMAIL
```

#### Kiểm chứng ngay sau deploy

```bash
export API_URL=YOUR_API_URL
curl -i "$API_URL/menu"
curl -i "$API_URL/admin/reservations"
curl -i -X POST "$API_URL/reservations" \
  -H 'Content-Type: application/json' --data '{}'
```

Menu phải trả 200, route admin không token bị 401, body rỗng bị 400. Chưa thử form trình duyệt đến khi CORS cập nhật. CORS không thay thế authorisation và không chặn mọi client ngoài trình duyệt.

Giữ nguyên stack/backend khi tạo frontend; không tạo thêm bảng/SES identity riêng bằng tay. Thay đổi tham số origin bằng deploy lại cùng file package, không ghi đè dữ liệu.
