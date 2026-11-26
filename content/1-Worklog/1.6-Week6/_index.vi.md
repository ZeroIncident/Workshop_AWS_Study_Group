---
title: "Nhật ký học tập Tuần 6"
date: 2026-10-19
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu Tuần 6:

* Tìm hiểu các mẫu độ tin cậy để bảo vệ dữ liệu, kết nối mạng, điều phối thông điệp và duy trì tính sẵn sàng của ứng dụng.
* Khám phá các phương pháp tính sẵn sàng cao cho Windows và SQL Server workloads chạy trên AWS.
* Phát triển kỹ năng container nhập môn với Docker, Amazon ECS và công cụ hạ tầng dưới dạng mã cho triển khai container.

### Nhiệm vụ trong tuần:

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | - Bảo vệ dữ liệu với AWS Backup<br>- Tích hợp mạng với VPC Peering<br>- Quản lý mạng tập trung với AWS Transit Gateway<br>- **Thực hành:** Tạo kế hoạch sao lưu và xem cài đặt VPC peering và Transit Gateway. | 19/10/2026 | 19/10/2026 | [Data Protection with AWS Backup](https://000013.awsfcaj.com/)<br>[VPC Peering](https://000019.awsfcaj.com/)<br>[Transit Gateway](https://000020.awsfcaj.com/) |
| 2 | - Hệ thống nhắn tin với Amazon SQS và SNS<br>- Lưu trữ dùng chung với Amazon EBS Multi-Attach<br>- **Thực hành:** Tạo SQS queue và SNS topic, sau đó xem cài đặt Multi-Attach. | 20/10/2026 | 20/10/2026 | [Messaging Systems](https://000077.awsfcaj.com/)<br>[EBS Multi-Attach](https://100000.awsfcaj.com/) |
| 3 | - Tính sẵn sàng cao cơ sở dữ liệu với EBS Multi-Attach và Systems Manager<br>- Windows Server Failover Clustering trên AWS<br>- **Thực hành:** Xem thiết kế failover cơ sở dữ liệu và yêu cầu Windows clustering. | 21/10/2026 | 21/10/2026 | [EBS Multi-Attach](https://100000.awsfcaj.com/)<br>[Windows Server Failover](https://100002.awsfcaj.com/) |
| 4 | - Tính sẵn sàng cao SQL Server trên AWS (2019)<br>- Tính sẵn sàng cao SQL Server trên AWS (2022)<br>- **Thực hành:** So sánh các tùy chọn triển khai sẵn sàng cao cho SQL Server. | 22/10/2026 | 22/10/2026 | [SQL Server High Availability on AWS (2019)](https://100003.awsfcaj.com/)<br>[SQL Server High Availability on AWS (2022)](https://100004.awsfcaj.com/) |
| 5 | - Container hóa với Docker<br>- Điều phối container với Amazon ECS<br>- Hạ tầng dưới dạng mã cho ECS với CDK<br>- **Thực hành:** Build Docker image, xem ECS task definition và kiểm tra ECS CDK stack. | 23/10/2026 | 23/10/2026 | [Containerization with Docker](https://000015.awsfcaj.com/)<br>[Container Orchestration](https://000016.awsfcaj.com/)<br>[Infrastructure as Code for ECS](https://000118.awsfcaj.com/) |

### Kết quả đạt được Tuần 6:

* Thực hành tạo cấu hình sao lưu và nhắn tin trong khi xem xét tùy chọn VPC peering, Transit Gateway và EBS Multi-Attach.
* Nâng cao kiến thức về các mẫu failover cơ sở dữ liệu và Windows, bao gồm các cân nhắc triển khai sẵn sàng cao SQL Server.
* Xây dựng kỹ năng Docker và ECS nhập môn bằng cách xem task definitions và hạ tầng ECS định nghĩa bằng AWS CDK.
