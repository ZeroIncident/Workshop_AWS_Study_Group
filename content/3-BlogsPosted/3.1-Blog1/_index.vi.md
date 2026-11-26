---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Nghiên cứu Managed Spot Training trên SageMaker AI: tiết kiệm chi phí cần đi cùng khả năng khôi phục

Đăng ngày 03/11/2026 (giờ Việt Nam). [Bài đăng trên AWS Study Group VN](https://www.facebook.com/groups/660548818043427/posts/2324996344931991/).

*Các thử nghiệm trong bài là đề xuất và chưa có số liệu đo.*

Trong một dự án machine learning, chi phí không chỉ đến từ lần huấn luyện cuối cùng. Quá trình thử mô hình, điều chỉnh siêu tham số và chạy lại thí nghiệm có thể sử dụng nhiều tài nguyên hơn dự kiến. Vì vậy, mình tìm hiểu Managed Spot Training của Amazon SageMaker AI dưới góc nhìn chi phí và khả năng hoàn thành công việc.

Managed Spot Training sử dụng Spot capacity để chạy training job. Đánh đổi chính là tài nguyên có thể bị gián đoạn, khiến thời gian bắt đầu hoặc hoàn thành khó dự đoán hơn. Theo tài liệu AWS, job sử dụng chế độ này không tự chuyển sang On-Demand khi thiếu Spot capacity. [Tài liệu Managed Spot Training](https://docs.aws.amazon.com/sagemaker/latest/dg/model-managed-spot-training.html)

Điều đó dẫn đến một câu hỏi quan trọng: sau gián đoạn, chương trình sẽ tiếp tục từ đâu?

Checkpoint giúp lưu trạng thái trong quá trình huấn luyện. SageMaker AI hỗ trợ đồng bộ checkpoint từ thư mục trong container lên S3 và đưa checkpoint trở lại môi trường huấn luyện khi khởi động. Với script hoặc container tùy chỉnh, người phát triển vẫn cần viết đúng phần lưu và nạp trạng thái. [Checkpoint trong SageMaker AI](https://docs.aws.amazon.com/sagemaker/latest/dg/model-checkpoints.html)

Trong một chương trình deep learning, mình sẽ kiểm tra checkpoint có đủ thông tin để tiếp tục quá trình tối ưu hay chưa. Ngoài trọng số mô hình, tùy framework và cách huấn luyện, có thể cần lưu trạng thái optimizer, learning rate scheduler và bước huấn luyện hiện tại. Đây là phần cần kiểm chứng bằng chương trình cụ thể, không nên suy ra chỉ từ việc thấy một tệp checkpoint xuất hiện.

![Khôi phục huấn luyện từ checkpoint trong S3](/images/aurel/blog-spot.png)
*Ảnh: AWS Documentation, minh họa việc đồng bộ và khôi phục checkpoint. [Nguồn ảnh](https://docs.aws.amazon.com/sagemaker/latest/dg/model-checkpoints-resume.html).*

Một thiết kế thử nghiệm phù hợp là chạy cùng mô hình, dữ liệu và cấu hình trên On-Demand và Managed Spot Training. Trước đó, cần kiểm tra riêng khả năng nạp checkpoint để bảo đảm chương trình thực sự tiếp tục từ trạng thái đã lưu.

Các chỉ số mình muốn ghi nhận gồm thời gian hoàn thành theo đồng hồ, thời gian tính phí, số lần gián đoạn và chất lượng mô hình cuối cùng. SageMaker cung cấp thông tin trạng thái job và các giai đoạn như Starting, Training hay Interrupted để theo dõi tiến trình. [Vòng đời Managed Spot Training](https://docs.aws.amazon.com/sagemaker/latest/dg/model-managed-spot-training-status.html)

Tần suất checkpoint cũng là một biến đáng nghiên cứu. Lưu quá thưa có thể làm mất nhiều công việc khi gián đoạn. Lưu quá thường xuyên lại tăng thời gian ghi và đồng bộ. Mình sẽ thử một vài khoảng lưu trên cùng workload rồi so sánh tổng thời gian cùng lượng công việc phải thực hiện lại.

Về vận hành, dữ liệu đầu vào, checkpoint và model artifact nên có vị trí lưu rõ ràng. Role của training job chỉ nên được cấp quyền phù hợp với những vị trí cần sử dụng. Khi kết thúc thử nghiệm, cần rà soát các tài nguyên phụ trợ và dữ liệu trung gian còn giữ lại.

Nhận định của mình là Spot phù hợp để cân nhắc cho các thí nghiệm chấp nhận thời gian chờ và có khả năng khôi phục tốt. Với công việc có thời hạn hoàn thành nghiêm ngặt, mức giảm chi phí phải được đặt cạnh rủi ro chậm tiến độ. Chưa chạy đối chứng thì chưa thể kết luận tỷ lệ tiết kiệm thực tế.

#AWS #AWSStudyGroup #FCAJ #FirstCloudAIJourney #AmazonSageMaker #ManagedSpotTraining #MachineLearning #CostOptimization
