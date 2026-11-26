---
title: "Tổng quan và tiêu chí hoàn thành"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Sản phẩm và người sử dụng

AUREL là thương hiệu minh họa được thiết kế riêng theo giao diện dark/gold. Khách có thể xem 100 món Việt và món bổ sung, xem ảnh không gian có ghi nguồn, gửi yêu cầu đặt bàn và lời nhắn. Nhân viên có tài khoản Cognito được cấp vào nhóm admin để quản lý dữ liệu.

Yêu cầu đặt bàn có trạng thái pending khi tiếp nhận. Nhân viên mới quyết định xác nhận hoặc hủy. Đây là sự khác biệt nghiệp vụ quan trọng: thông báo gửi form thành công không có nghĩa nhà hàng đã xác nhận bàn.

#### Công nghệ

Next.js 15.5.28, React 19, TypeScript, Tailwind CSS, Zod, AWS SDK v3, Node.js 22, pnpm 11.25.0. Lambda chạy arm64/256 MB/timeout 20 giây tại Singapore. Dữ liệu dùng DynamoDB on-demand, mã nguồn riêng trên CodeCommit, giao diện SSR trên Amplify WEB_COMPUTE.

#### Đầu ra của workshop

- Website gồm 7 trang và form hoạt động với backend.
- Thực đơn 100 món, ảnh WebP tự lưu trữ; nguồn/giấy phép rõ ràng.
- API công khai và admin có ranh giới quyền truy cập cụ thể.
- Transaction bảo vệ sức chứa trong một slot; UUID chống tạo yêu cầu trùng.
- Template hạ tầng, hướng dẫn build/deploy, kiểm thử, quan sát và cleanup.

#### Các giới hạn phải hiểu trước khi làm

SES vẫn ở sandbox; MFA admin chưa có giao diện xử lý; chưa có outbox/queue. Mô hình sức chứa chỉ áp dụng cho cùng giờ, chưa xét khoảng thời gian dùng bữa. Ảnh không gian là tư liệu từ nhiều địa điểm, không phải ảnh của nhà hàng AUREL có thật.

Các mục tiêu hiệu năng hoặc kiến trúc mở rộng được ghi là đề xuất, không phải tính năng đã triển khai. Workshop không kèm cam kết SLA, audit bảo mật hay kiểm thử toàn bộ thiết bị.
