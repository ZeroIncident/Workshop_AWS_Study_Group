---
title: "Dọn dẹp tài nguyên"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### Nguyên tắc và inventory

Đây là hướng dẫn cho tài khoản lab của người đọc. Không thực thi cleanup trên demo đang dùng nếu chưa quyết định ngừng dịch vụ. Xóa stack không có nghĩa mọi tài nguyên đã biến mất: bảng DynamoDB, bucket media/artifacts và CodeCommit repository có Retain.

Trước khi xóa, xuất danh sách resource vật lý và ghi nguồn sở hữu. Sao lưu dữ liệu cần giữ vào vị trí riêng tư được phép, kiểm tra có thể đọc bản sao. Chưa có restore drill thực tế để khẳng định RTO/RPO.

```bash
aws cloudformation list-stack-resources --stack-name aurel-backend
aws cloudformation list-stack-resources --stack-name aurel-frontend
aws cloudformation list-stack-resources --stack-name aurel-artifacts
aws cloudformation list-stack-resources --stack-name aurel-observability
```

#### Trình tự dọn lab

- Dừng pipeline/build mới; ghi AppId và resource IDs trước khi xóa.
- Lưu source/build report rồi xóa riêng stack aurel-report-branch và nhánh source report nếu không cần. Không xóa Amplify app dùng chung hoặc nhánh main của nhà hàng. Các bucket rỗng còn lại từ lần thử CloudFront cần kiểm kê riêng theo stack.
- Xóa stack frontend; chờ DELETE_COMPLETE. CodeCommit có Retain nên vẫn cần xử lý riêng.
- Xóa stack backend; chờ hoàn tất. Cognito users, SES identity và log group do stack sở hữu bị xóa; bảng và media bucket vẫn còn.
- Xóa stack observability để dọn alarms, dashboard và API access log. Chỉ xóa nếu log không cần giữ để điều tra.
- Với bucket Retain, kiểm tra versioning, toàn bộ versions/delete markers và multipart uploads. Sau sao lưu, empty bucket rồi xóa bucket. Lệnh rm –recursive không xóa hết versions của bucket có versioning.
- Xóa đúng bảng DynamoDB và repository Retain theo inventory; xem manual backup/export còn giữ và phí liên quan.
- Dọn artifact bucket, rồi xóa stack artifacts; xử lý bucket Retain nếu stack đã tách quyền quản lý.
- Kiểm tra log Amplify ngoài stack, SNS subscription/Budget nếu đã thêm sau này, và các tài nguyên ngoài region chính. Không xóa tài nguyên dự án khác.

Lệnh xóa stack mẫu, chỉ chạy sau khi đã quyết định kết thúc lab và sao lưu:

```bash
aws cloudformation delete-stack --stack-name aurel-frontend
aws cloudformation wait stack-delete-complete --stack-name aurel-frontend
aws cloudformation delete-stack --stack-name aurel-backend
aws cloudformation wait stack-delete-complete --stack-name aurel-backend
aws cloudformation delete-stack --stack-name aurel-observability
aws cloudformation wait stack-delete-complete --stack-name aurel-observability
```

Các bước xóa Retain không được tự động hóa bằng wildcard vì có nguy cơ xóa dữ liệu ngoài phạm vi. Đối chiếu trạng thái thật và kiểm tra Billing sau khi dữ liệu cập nhật; đóng trình duyệt không ngừng tính phí. Báo cáo này không tuyên bố đã dọn hạ tầng đang phục vụ demo.

**Liên kết:**
- [Retain trong CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-deletionpolicy.html)
- [Empty S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html)
