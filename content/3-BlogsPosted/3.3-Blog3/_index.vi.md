---
title: "Blog 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Nghiên cứu tối ưu Athena: đổi cách lưu dữ liệu có thể hiệu quả hơn sửa câu SQL

Đăng ngày 08/11/2026 (giờ Việt Nam). [Bài đăng trên AWS Study Group VN](https://www.facebook.com/groups/660548818043427/posts/2327599394671686/).

*Các thử nghiệm trong bài là đề xuất và chưa có số liệu đo.*

Khi phân tích dữ liệu trên S3 bằng Amazon Athena, mình muốn tìm hiểu một câu hỏi khá thực tế: cùng một kết quả truy vấn, cách tổ chức dữ liệu có thể làm thay đổi bao nhiêu lượng dữ liệu phải đọc?

Trong bài này, mình xét truy vấn Athena SQL theo cơ chế tính phí dựa trên dữ liệu quét. Với phạm vi đó, giảm dữ liệu phải đọc có thể hỗ trợ cả hiệu năng lẫn chi phí. Tuy nhiên, Athena có nhiều lựa chọn tính phí, nên khi lập dự toán phải đối chiếu đúng chế độ đang sử dụng. [Thông tin giá Athena](https://aws.amazon.com/athena/pricing/)

Hai hướng nghiên cứu là định dạng lưu trữ và partition.

Parquet là định dạng theo cột. Với truy vấn chỉ cần một vài trường, cách lưu này cho phép tránh đọc những cột không liên quan. Parquet còn có metadata giúp bỏ qua một số khối dữ liệu khi điều kiện truy vấn phù hợp. Đây là cơ sở để kỳ vọng lợi ích so với dữ liệu văn bản dạng hàng, chứ chưa phải bảo đảm rằng mọi truy vấn đều nhanh hơn. [Tài liệu định dạng cột trong Athena](https://docs.aws.amazon.com/athena/latest/ug/columnar-storage.html)

Partition giải quyết một vấn đề khác: giới hạn phần dữ liệu cần đọc. Ví dụ, với log được chia theo ngày, truy vấn một khoảng thời gian có điều kiện phù hợp trên partition key có thể chỉ đọc các partition liên quan. Việc tạo thư mục theo ngày trên S3 cần đi cùng cấu hình bảng và metadata phù hợp trong Athena. [Tài liệu partition](https://docs.aws.amazon.com/athena/latest/ug/partitions.html)

![Ví dụ pipeline chuyển đổi và phân vùng log để truy vấn bằng Athena](/images/aurel/blog-athena.jpg)
*Ảnh: AWS Big Data Blog, ví dụ pipeline phân tích CloudFront log sử dụng chuyển đổi định dạng và partition. Đây là kiến trúc tham khảo của AWS, không phải kết quả thử nghiệm của bài viết. [Nguồn ảnh](https://aws.amazon.com/blogs/big-data/analyze-your-amazon-cloudfront-access-logs-at-scale/).*

Mình đề xuất so sánh ba cách lưu cùng một bộ dữ liệu: CSV chưa chia partition, Parquet chưa chia partition và Parquet chia partition theo ngày. Các bảng phải có cùng nội dung và kiểu dữ liệu tương đương để tránh so sánh sai.

Bộ truy vấn nên có trường hợp đọc vài cột trong một ngày, tổng hợp trong một tháng và đọc phần lớn dữ liệu. Với mỗi truy vấn, cần ghi lại lượng dữ liệu quét, thời gian thực thi và kết quả trả về. Nên kiểm soát việc tái sử dụng kết quả truy vấn, chạy nhiều lần và so sánh kết quả trước khi kết luận về tốc độ.

Một rủi ro là chia partition quá nhỏ hoặc tạo quá nhiều tệp nhỏ. Khi đó, chi phí xử lý metadata và mở tệp có thể làm giảm lợi ích. AWS cũng lưu ý lựa chọn partition key phải xuất phát từ cách dữ liệu được truy vấn. [Phân tích tối ưu Athena của AWS](https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/)

Ngoài chi phí truy vấn, cần tính công chuyển đổi dữ liệu, lưu trữ bản đã tối ưu và việc duy trì pipeline. Khi làm lab, mình sẽ dùng dữ liệu công khai, giới hạn quyền vào bucket thử nghiệm và quản lý cả nơi lưu kết quả truy vấn.

Với mình, đây là bài toán phù hợp để luyện tư duy thực nghiệm. Thay vì nói Parquet luôn tốt hơn CSV, mình cần chỉ ra nó tốt hơn trong loại truy vấn nào, đổi lại điều gì và được chứng minh bằng số liệu nào.

#AWS #AWSStudyGroup #FCAJ #FirstCloudAIJourney #AmazonAthena #AmazonS3 #DataEngineering #CostOptimization
