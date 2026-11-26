---
title: "Xử lý sự cố và bài học kỹ thuật"
date: 2024-01-01
weight: 13
chapter: false
pre: " <b> 5.13. </b> "
---

#### Các tình huống đã gặp trong dự án

| Tình huống | Phân tích | Cách xử lý và bằng chứng |
| :--- | :--- | :--- |
| Source ZIP có thư mục bọc | Build không tìm package.json đúng gốc | Script package-source tạo gói sạch ở root |
| Reserved concurrency 10 xung đột quota 10 | Không chừa unreserved capacity | Bỏ reserved concurrency, giữ API throttling và ghi giới hạn tài khoản |
| Đổi cấu hình origin gây lỗi CORS/BFF | Public API và BFF có cách gọi khác nhau | Cập nhật CORS API/S3 cùng origin, APP_ORIGIN và rebuild env public |
| SES sandbox | Gửi được không đồng nghĩa gửi cho mọi địa chỉ | Verify sender, thử mailbox simulator, lưu yêu cầu trước và ghi rõ sandbox |
| Commit tài liệu vẫn kích hoạt build | Pipeline dựa branch, không chắc hiểu skip marker | Dừng job chỉ đổi docs; tách report repository khỏi app |
| Seed cũ và menu 100 món | Không được xóa món người dùng tự thêm | Migration backup và nhận diện đúng món mẫu, bảo toàn nội dung khác |
| Muốn thêm ảnh thật nhưng khác địa điểm | Rủi ro gây hiểu nhầm về thương hiệu | Ghi ảnh tư liệu/inspiration, nguồn và giấy phép theo ảnh |



#### Bài học có thể áp dụng

Kiểm tra theo đường đi dữ liệu giúp xác định lỗi nằm ở trình duyệt, BFF, Gateway, Lambda hay database. Thành công ở build không đủ chứng minh form ghi dữ liệu; HTTP 200 không chứng minh responsive hoặc accessibility. Một thông báo SES lỗi cũng không có nghĩa transaction đặt bàn thất bại.

Least privilege cần xét từng action và resource, không chỉ tên policy. Tài nguyên được quản lý không có nghĩa tự động phù hợp mọi yêu cầu bảo mật. Retain bảo vệ dữ liệu nhưng tạo nghĩa vụ dọn dẹp và kiểm tra phí.

#### Phát triển tiếp theo theo thứ tự

- Chuyển tài khoản triển khai khỏi root, thu hẹp policy, hỗ trợ và kiểm thử MFA.
- Hoàn thiện thông báo ngoài Console, tạo Budget theo ngưỡng đã thống nhất.
- Outbox/SQS/DLQ và xử lý message trùng; đối soát các outbox chưa phát.
- API pagination, expiration pending có quy tắc, mô hình bàn/thời lượng dùng bữa.
- Load test có kiểm soát, restore drill và các phép đo p95/chi phí thực.
- Kiểm thử Windows/mobile/accessibility, bổ sung video và sơ đồ cá nhân tự chỉnh.

Đây là roadmap, không phải danh sách đã triển khai. Không thêm WAF bằng cách gắn trực tiếp vào HTTP API; cần thiết kế đường bảo vệ tương thích trước.
