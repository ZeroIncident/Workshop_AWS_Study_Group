---
title: "COST — Dự toán và tối ưu chi phí"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

#### Kịch bản tải và phương pháp

Dự toán lập ngày 03/11/2026 cho Singapore, USD/tháng, chưa VAT, chưa credits/Free Tier/chiết khấu. Đây là mô hình dự toán độc lập từ bảng giá AWS, không phải hóa đơn và chưa phải bản xuất AWS Pricing Calculator.

Giả định: 10.000 lượt/tháng, 100.000 API requests, 100.000 SSR requests; SSR trung bình 200 ms và 0,5 GB, Lambda trung bình 200 ms và 0,25 GB; 20 GB nội dung frontend; 5 admin; 1.000 thư; bảng 0,1 GB; S3 2 GB; log ingest 0,25 GB; 4 alarm và 1 dashboard. Các số latency/memory SSR là giả định dự toán, không phải đo benchmark.

Số RRU 3,2 triệu đã dự phòng việc một GET menu trả 100 items và dùng đọc strong consistency. Không giả định một HTTP request chỉ tốn một RRU. 20.000 WRU bao gồm bộ đếm limiter và chi phí transaction; cần thay bằng ConsumedCapacity khi đo thực tế.

#### Bảng dự toán chi tiết

| Dịch vụ / Service | Lượng / Quantity | Đơn giá USD / Unit rate | USD/tháng / month |
|---|---|---|---|
| Amplify build | 30 minute | 0.01 | 0.3000 |
| Amplify storage | 1 GB-month | 0.023 | 0.0230 |
| Amplify transfer | 20 GB | 0.15 | 3.0000 |
| Amplify SSR requests | 1e+05 request | 0.0000003 | 0.0300 |
| Amplify SSR duration | 2.778 GB-hour | 0.20 | 0.5556 |
| Lambda arm64 compute | 5,000 GB-second | 0.0000133334 | 0.0667 |
| Lambda requests | 1e+05 request | 0.0000002 | 0.0200 |
| HTTP API requests | 1e+05 request | 0.00000125 | 0.1250 |
| DynamoDB strong reads | 3.2e+06 RRU | 0.0000001425 | 0.4560 |
| DynamoDB writes | 2e+04 WRU | 0.00000071 | 0.0142 |
| DynamoDB storage | 0.1 GB-month | 0.285 | 0.0285 |
| DynamoDB PITR | 0.1 GB-month | 0.228 | 0.0228 |
| S3 Standard storage | 2 GB-month | 0.025 | 0.0500 |
| S3 PUT | 1,000 request | 0.000005 | 0.0050 |
| S3 GET | 1e+04 request | 0.0000004 | 0.0040 |
| Cognito Essentials | 5 MAU | 0.015 | 0.0750 |
| SES Essentials outbound | 1,000 email | 0.00016 | 0.1600 |
| CloudWatch logs ingestion | 0.25 GB | 0.70 | 0.1750 |
| CloudWatch log storage | 0.1 GB-month | 0.03 | 0.0030 |
| CloudWatch standard alarms | 4 alarm-month | 0.10 | 0.4000 |
| CloudWatch dashboard budget | 1 dashboard-month | 3 | 3.0000 |
| Report Amplify transfer | 1 GB | 0.15 | 0.1500 |
| Report Amplify storage | 0.1 GB-month | 0.023 | 0.0023 |
| API/S3 internet transfer allowance | 1 budget allowance | 2 | 2.0000 |
| CodeCommit allowance | 1 budget allowance | 1 | 1.0000 |
| Report build and SSR allowance | 1 budget allowance | 0.5 | 0.5000 |
| Tổng / Total | | | 12.17 |
| Ngân sách +25% / Budget +25% | | | 15.21 |

Tài liệu đính kèm: [CSV có công thức đầu vào và nguồn từng dòng](/downloads/cost-model.csv) · [Kịch bản JSON](/downloads/cost-scenario.json)

Lambda: 100.000 × 0,2 giây × 0,25 GB = 5.000 GB-giây. SSR: 100.000 × 0,2 × 0,5 / 3.600 = 2,7778 GB-giờ. Các dòng allowance là khoản dự phòng, không phải đơn giá được AWS báo. Giá regional được đối chiếu AWS Price List; Amplify theo trang giá chính thức. Dashboard lập dự phòng 3 USD dù tài khoản có thể có quota miễn phí. SES dùng Essentials 0,16 USD/1.000 email theo plan đã ghi nhận. Cognito tính biên Essentials, cần đối chiếu tier thực tế. CodeCommit allowance cần thay theo user/storage/quota của tài khoản.

Khoản chưa định lượng riêng: attachments/email data, restore/export DynamoDB, tăng số build và bản lưu, query log lớn, hỗ trợ trả phí, custom domain/DNS, WAF, NAT, SQS và kiến trúc đề xuất. Băng thông API/S3 có khoản dự phòng 2 USD, không bảo đảm đủ nếu workload đổi. Không có multi-region hay SMS MFA trong dự toán.

#### Kết quả AWS Pricing Calculator đã tạo thật

Đã tạo workload estimate AUREL-FCAJ-20261103 bằng API chính thức của AWS Pricing Calculator trong Console. ID: 5c91cb04-0f41-436d-ae5f-72733aa95e31. AWS trả trạng thái VALID; đủ 20 dòng sử dụng đều VALID, không có lỗi nhập dòng. Rate type BEFORE_DISCOUNTS; timestamp bảng giá 03/11/2026. Đây là mô hình chi phí, không tạo thêm hạ tầng và không phải hóa đơn.


#### Bảng kết quả AWS Pricing Calculator

| Dịch vụ / Service | USD/tháng do AWS tính / AWS monthly USD |
|---|---|
| AWSAmplify | 4.419412 |
| AWSLambda | 0.086667 |
| AmazonApiGateway | 0.125000 |
| AmazonCloudWatch | 0.578000 |
| AmazonCognito | 0.075000 |
| AmazonDynamoDB | 0.493000 |
| AmazonS3 | 0.059000 |
| AmazonSES | 0.160000 |
| Tổng 20 dòng / Total for 20 lines | 5.996079 |

Kết quả khoảng 6,00 USD/tháng chỉ bao gồm đúng 20 dòng đã nhập, thuộc 8 dịch vụ. Không dùng số này như tổng vận hành hoàn chỉnh. Các dòng Amplify đã gộp report: build 60 phút, storage 1,1 GB, delivery 21 GB, SSR 110.000 requests và 11.000 GB-giây. Workload dùng usage type APS1 cho Singapore; API Calculator được gọi qua endpoint us-east-1.

Chênh lệch với ngân sách độc lập 12,17 USD được giải thích: 3 USD dự phòng dashboard, 2 USD transfer API/S3, 1 USD CodeCommit chưa đưa vào mô hình AWS; phần còn lại là đệm report, bậc storage và làm tròn. AWS tính DynamoDB storage 0,1 GB là 0 USD theo bậc giá hiện hành; bảng độc lập dùng đơn giá biên có phí để dự phòng. BEFORE_DISCOUNTS không có nghĩa mọi bậc giá 0 USD bị loại bỏ.
