---
title: "Bảo mật và Well-Architected"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Danh tính và phân quyền

Khách dùng các route công khai cho thực đơn, ảnh món đang hiển thị, đặt bàn và liên hệ. Admin phải có token còn hạn, đúng issuer/client và nhóm admin. Token trong cookie HttpOnly giảm khả năng JavaScript đọc trực tiếp; Secure và SameSite=Strict kết hợp kiểm tra Origin cho BFF mutation. Không lưu token trong localStorage. Sign-out cố gắng GlobalSignOut nhưng JWT đã phát hành có thể vẫn được Gateway chấp nhận tới hạn; không tuyên bố thu hồi tức thì.

Cognito hiện MFA OFF. Không bật MFA REQUIRED khi UI chưa xử lý challenge vì sẽ làm admin không đăng nhập được. Ưu tiên tiếp theo là hỗ trợ TOTP/challenge, khôi phục tài khoản và kiểm thử đầy đủ trước khi bắt buộc MFA.

#### IAM runtime

Role Lambda chỉ truy cập bảng dự án, S3 uploads/*, menu-images/* và SES identity đã chỉ định. Tuy nhiên template đang dùng DynamoDBCrudPolicy, phạm vi action rộng hơn tập action runtime thực sự dùng. Cần thay bằng Query, GetItem, PutItem, DeleteItem, UpdateItem và quyền các thao tác dữ liệu trong TransactWriteItems, trên đúng table ARN; không cấp DynamoDB toàn tài khoản.

Ví dụ policy hẹp hơn để rà soát (chưa áp dụng vào stack hiện tại):

```json
{
  "Effect": "Allow",
  "Action": ["dynamodb:Query", "dynamodb:GetItem", "dynamodb:PutItem", "dynamodb:DeleteItem", "dynamodb:UpdateItem"],
  "Resource": "arn:aws:dynamodb:REGION:ACCOUNT:table/YOUR_TABLE"
}
```

TransactWriteItems được phân quyền theo các thao tác thành phần tương ứng; không thêm action IAM không tồn tại chỉ vì tên API là TransactWriteItems. Deployment role có nhu cầu khác runtime role và cần chính sách triển khai được rà soát riêng.

#### Dữ liệu và mạng

S3 bật cả bốn Block Public Access, SSE-S3, Object Ownership enforced và deny non-TLS. DynamoDB bật SSE/PITR. Không có public database password. URL ký không phải object public nhưng ai có URL còn hạn đều có thể dùng; không ghi URL này vào report hoặc log.

Public endpoint là cần thiết cho khách. Lambda không có public IP do khách tự truy cập; quyền invoke đến từ Gateway. Không có VPC tùy chỉnh trong bản triển khai. Chỉ thêm private subnet/endpoints khi có nhu cầu mạng phù hợp, đồng thời cân nhắc NAT/endpoint charges.

Log API chỉ ghi metadata, không ghi form, email, số điện thoại, Cookie hoặc Authorization. Bản ghi khách hiện chưa có retention/delete workflow hoàn chỉnh; cần chính sách dữ liệu trước vận hành thật.

#### Đối chiếu các trụ cột

| Trụ cột | Áp dụng | Hạn chế / Cải tiến |
|---|---|---|
| Operational Excellence | IaC, pipeline, tests, logs và runbook | Chuẩn hóa rollback và thay đổi nhỏ có kiểm chứng |
| Security | Cognito/JWT/group, cookie, private S3, không hard-code key | MFA, quyền deploy khỏi root, thu hẹp IAM actions |
| Reliability | DDB transaction/PITR, dịch vụ managed | Outbox/queue, restore drill, chưa có multi-region DR |
| Performance Efficiency | WebP, thumbnails, self-host fonts, arm64 | Chưa có load-test; dashboard cần API pagination |
| Cost Optimization | On-demand, lifecycle 30 ngày, log retention 14 ngày | Hoàn thành Calculator và Budget alert |
| Sustainability | Compute theo yêu cầu, giảm byte ảnh và tải font | Đo traffic và loại bỏ tài nguyên không cần thiết |

#### Liên kết tham khảo

- [Least privilege và temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM DynamoDB actions](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazondynamodb.html)
