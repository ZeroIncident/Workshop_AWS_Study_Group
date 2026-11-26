---
title: "Kiểm thử và xác thực"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Kết quả có bằng chứng

Lần rà soát ghi nhận 84 trang report Việt–Anh trả HTTP 200. Trang chủ AUREL trả 200, API công khai trả 100 món có ID duy nhất và API quản trị trả 401 khi thiếu token. Kết quả HTTP.

#### Bảng kết quả kiểm thử

| Nhóm | Kết quả đã ghi nhận | Phạm vi |
|---|---|---|
| Local | 30 tests; typecheck/build; 28 HTTP smoke assertions | Trước release ảnh không gian; transaction Dynamo dùng mock |
| Backend AWS | 14 checks thành công | Cognito/access token, từ chối ID token, lưu/đọc/status/idempotency, S3 PUT/GET thật, contact |
| Release thực đơn | API public/admin đều 100 món, 100 ảnh khớp bytes, 7 trang HTTP 200 | Kiểm tra HTTP/assets/auth, không tự động hóa trình duyệt |
| Release không gian | 12 thẻ, 8 ảnh không gian, 20 assets khớp bytes, menu vẫn 100 | HTTP và HTML render server |
| Monitoring | Xem evidence cập nhật tại phần 5.9 | Không suy ra đã có sự cố thật từ thao tác giả lập alarm |

#### Liên kết bằng chứng


Đã từng kiểm tra trình duyệt trên bản deploy đầu: login BFF, dashboard, logout và đặt bàn. Ảnh minh chứng đó là bản trước menu 100 món; không dùng để khẳng định mọi tương tác của gallery mới đã được thử.

#### Các request âm tính có thể tái lập

```bash
export API_URL=YOUR_API_URL
curl -i "$API_URL/admin/reservations"
curl -i -X POST "$API_URL/reservations" \
  -H 'Content-Type: application/json' --data '{}'
curl -i -X POST "$API_URL/reservations" \
  -H 'Content-Type: application/json' --data '{bad-json'
curl -i -X OPTIONS "$API_URL/menu" \
  -H 'Origin: https://untrusted.example' \
  -H 'Access-Control-Request-Method: GET'
```

Kỳ vọng lần lượt: 401, 400, 400; origin không tin cậy không nhận Access-Control-Allow-Origin cho origin đó. HTTP 200 cho OPTIONS không đồng nghĩa origin được phép. Không thử tải lớn trên demo công khai.

#### Kiểm thử có ghi dữ liệu — chỉ trên tài khoản lab

Dùng form với email đã verify hoặc SES mailbox simulator, ngày ở tương lai 1–90 ngày và trước ít nhất 30 phút. Ghi UUID được trả về, đăng nhập admin, xác nhận, tải lại rồi hủy. Đối chiếu bản ghi DynamoDB và bộ đếm slot. Gửi lại cùng UUID chỉ nhận cùng yêu cầu; muốn đổi dữ liệu phải dùng UUID mới.

Để thử cạnh tranh thực sự, chuẩn bị một slot riêng và gửi các yêu cầu đồng thời có tổng số khách vượt 24. Sau thử nghiệm, xác nhận tổng ghế không vượt 24 và không tồn tại bản ghi dang dở. Bài thử tải cạnh tranh này chưa được thực hiện trên DynamoDB thật trong bộ bằng chứng hiện tại.

Không xóa sạch bảng để dọn test. Chỉ xử lý đúng các UUID do bài thử tạo; hủy trước để hoàn ghế rồi mới xóa bản ghi test nếu cần. Lưu log trạng thái, không lưu token hay dữ liệu cá nhân vào báo cáo.

#### Kiểm tra UI còn lại

Cần trực tiếp thử Windows/Chrome/Edge, thiết bị iOS/Android, điều hướng bàn phím lightbox, screen reader, độ tương phản và form trên màn hình nhỏ. Không suy ra các mục này đã đạt chỉ từ việc HTTP trả 200 hoặc build thành công.
