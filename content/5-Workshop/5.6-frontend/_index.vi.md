---
title: "Bước 3 — Frontend và CI/CD"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Khởi tạo source và Amplify

Đặt API_URL, COGNITO_CLIENT_ID, USER_POOL_ID từ backend outputs trước khi chạy. Template tạo CodeCommit riêng, role chỉ đọc repository này và Amplify WEB_COMPUTE. Source ZIP phải có package.json ở gốc.



```bash
python3 scripts/package-source.py --output ../aurel-codecommit-source.zip
aws s3 cp ../aurel-codecommit-source.zip \
  "s3://$ARTIFACT_BUCKET/source/aurel-lab.zip"
aws cloudformation deploy --stack-name aurel-frontend \
  --template-file infra/amplify-codecommit.yaml \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides SourceBucket="$ARTIFACT_BUCKET" \
    SourceKey=source/aurel-lab.zip ApiUrl="$API_URL" \
    CognitoClientId="$COGNITO_CLIENT_ID" \
    CognitoUserPoolId="$USER_POOL_ID"
aws cloudformation describe-stacks --stack-name aurel-frontend \
  --query 'Stacks[0].Outputs' --output table
```

Lấy AppId và FrontendOrigin từ Outputs, đặt APP_ID/FRONTEND_ORIGIN. Sau đó cập nhật backend CORS và S3 CORS theo cùng origin không có dấu / cuối.

```bash
aws cloudformation deploy --stack-name aurel-backend \
  --template-file packaged-backend.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND \
  --parameter-overrides FrontendOrigin="$FRONTEND_ORIGIN" \
    SenderEmail="$SENDER_EMAIL"
aws amplify start-job --app-id "$APP_ID" --branch-name main --job-type RELEASE
aws amplify list-jobs --app-id "$APP_ID" --branch-name main --max-results 5
```

Chỉ coi deploy thành công khi job SUCCEED. Kiểm tra cả BUILD/DEPLOY/VERIFY, mở URL và đối chiếu API. Trong lần triển khai thực tế, job 5 đã thành công với commit ffe7deaa88b7090ad0f10ca60ed1f86f3cc16d8b.

#### Cấu hình môi trường

| Biến | Nguồn / Quy tắc |
|---|---|
| DATA_MODE | aws |
| API_BASE_URL, NEXT_PUBLIC_API_BASE_URL | ApiUrl, không slash cuối |
| COGNITO_CLIENT_ID, COGNITO_USER_POOL_ID | Backend outputs |
| APP_AWS_REGION | ap-southeast-1 |
| APP_ORIGIN | Origin frontend chính xác |

scripts/amplify-env.mjs chỉ ghi danh sách cấu hình được phép vào .env.production. Không đưa mật khẩu admin, SESSION_SECRET hay AWS key vào Amplify frontend variables. BFF dùng Cognito public client và API, không cần role truy cập trực tiếp database.

#### Cập nhật source

Code.S3 chỉ nạp source lúc tạo repository lần đầu. Thay ZIP trong S3 rồi update stack không cập nhật Git history. Những lần sau dùng Git/CodeCommit với phiên IAM tạm thời, push branch main; Amplify tự build. Không lưu token trong remote URL và không để Git dùng helper lưu mật khẩu ngoài ý muốn.

Nếu tài khoản không hỗ trợ kết nối CodeCommit, sử dụng phương án GitHub trong README ứng dụng và template amplify.yaml; không công khai repository cùng file .env. Đường thay thế chưa được thử trong tài khoản mới.

**Liên kết tham khảo:**
- [Amplify Next SSR](https://docs.aws.amazon.com/amplify/latest/userguide/deploy-nextjs-app.html)
- [SSR environment](https://docs.aws.amazon.com/amplify/latest/userguide/environment-variables.html)

#### Website report riêng trên nhánh report

Quota tài khoản không cho tạo app Amplify mới; CloudFront yêu cầu xác minh tài khoản nên chưa triển khai được phương án S3/CloudFront. Report được xuất từ Hugo sang host Next.js tối giản ở nhánh CodeCommit report, gắn vào nhánh Amplify report của app hiện có. Nhánh main và backend nhà hàng không thay đổi.

Host giữ nguyên HTML, CSS, JavaScript và font của template Hugo; route phục vụ HTML từ public, static assets đi qua Hosting. Cấu hình giữ trailing slash và đóng gói HTML trong server trace. File download lớn được tách khi lưu Git. Trang tải mã nguồn tải từng phần nhỏ, kiểm tra SHA-256 rồi ghép ZIP trong trình duyệt để tránh giới hạn phản hồi SSR.

```bash
hugo --baseURL https://report.YOUR_APP_ID.amplifyapp.com/ --minify --cleanDestinationDir
python3 scripts/check-report.py
python3 scripts/export-host.py --destination ../aurel-report-host
# Push exported source to the separate report branch first.
aws cloudformation deploy --stack-name aurel-report-branch \
  --template-file report-branch.json --parameter-overrides ExistingAppId=YOUR_APP_ID
aws amplify start-job --app-id YOUR_APP_ID --branch-name report --job-type RELEASE
```

README trong source report hướng dẫn cập nhật và dọn riêng nhánh, không xóa shared app.
