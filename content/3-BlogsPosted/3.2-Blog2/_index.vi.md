---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Nghiên cứu Lambda và SQS: vì sao xử lý thành công vẫn có thể bị chạy lại?

Đăng ngày 05/11/2026 (giờ Việt Nam). [Bài đăng trên AWS Study Group VN](https://www.facebook.com/groups/660548818043427/posts/2326372234794402/).

*Các thử nghiệm trong bài là đề xuất và chưa có số liệu đo.*

Một hệ thống xử lý đơn hàng hoặc dữ liệu cảm biến thường cần tiếp nhận công việc nhanh rồi xử lý ở phía sau. Amazon SQS và AWS Lambda là một cách xây dựng luồng này. Tuy nhiên, điểm mình muốn tìm hiểu sâu hơn là điều gì xảy ra khi một phần dữ liệu xử lý thành công, phần còn lại thất bại.

Với SQS event source mapping, Lambda đọc message từ queue rồi gọi function theo batch. Khi xử lý thành công, các message tương ứng được xóa. Cơ chế xử lý là at least once, nên cùng một message có thể được xử lý nhiều lần. Ứng dụng phải tính đến khả năng này. [Cơ chế tích hợp Lambda và SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)

Giả sử một batch có 10 message và message thứ sáu gây lỗi. Theo hành vi mặc định, cả batch có thể xuất hiện lại sau visibility timeout, kể cả những message đã xử lý thành công. Nếu mỗi message làm tăng một bộ đếm hoặc tạo một giao dịch mới, việc chạy lại có thể tạo ra kết quả ngoài ý muốn.

AWS hỗ trợ partial batch response để function báo những message nào thất bại. Muốn dùng cơ chế này, cần cấu hình ReportBatchItemFailures và trả về danh sách message lỗi đúng định dạng. Chỉ sửa phần xử lý trong function mà chưa bật cấu hình ở event source mapping là chưa đủ. [Hướng dẫn xử lý lỗi theo từng message](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-errorhandling.html)

![Kết nối SQS với Lambda bằng event source mapping](/images/aurel/blog-sqs.png)
*Ảnh: AWS Documentation, bước cấu hình event source mapping trong hướng dẫn Lambda và SQS. [Nguồn ảnh và bài thực hành](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs-example.html).*

Partial batch response giúp giảm những lần thử lại không cần thiết, nhưng mình vẫn cần thiết kế idempotency. Có thể hiểu đây là khả năng nhận lại cùng một yêu cầu mà không tạo thêm tác động nghiệp vụ ngoài dự kiến.

Với một bài toán giả định về đơn hàng, mình sẽ dùng mã yêu cầu nghiệp vụ làm căn cứ nhận diện công việc. Phần lưu trạng thái phải xét cả trường hợp hai worker xử lý đồng thời và trường hợp worker dừng sau khi đã tạo tác động nhưng chưa ghi nhận hoàn tất. Một thao tác kiểm tra rồi ghi đơn giản chưa chắc xử lý được những tình huống đó.

Thử nghiệm đề xuất sẽ có dữ liệu hợp lệ, dữ liệu cố ý gây lỗi và các yêu cầu có cùng mã nghiệp vụ. Mình sẽ so sánh số lần function nhận message với số kết quả nghiệp vụ cuối cùng, đồng thời kiểm tra những message lỗi nhiều lần có được chuyển sang dead-letter queue theo cấu hình hay không. Visibility timeout, timeout của function và chính sách redrive cần được thiết kế cùng nhau. [Hướng dẫn cấu hình SQS cho Lambda](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-configure.html)

Về chi phí, message lỗi kéo dài có thể làm tăng số lần xử lý và lượng log. Về bảo mật, role của function nên giới hạn vào queue và nơi lưu kết quả cần dùng; nội dung nhạy cảm không nên được ghi nguyên vào log chỉ để thuận tiện gỡ lỗi.

Điều mình rút ra là hàng đợi giúp tách quá trình tiếp nhận và xử lý công việc, nhưng tính đúng đắn của kết quả vẫn phụ thuộc vào ứng dụng. Một hệ thống đáng tin cậy cần được kiểm tra cả khi thành công lẫn khi lỗi xảy ra giữa chừng.

#AWS #AWSStudyGroup #FCAJ #FirstCloudAIJourney #AWSLambda #AmazonSQS #Serverless #DistributedSystems
