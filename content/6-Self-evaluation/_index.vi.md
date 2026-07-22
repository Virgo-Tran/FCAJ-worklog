---
title: "Tự đánh giá"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

Trong thời gian thực tập tại **Công ty TNHH Amazon Web Services Việt Nam** từ
**05/05/2026** đến **30/07/2026** — mười ba tuần trong chương trình Workforce
Bootcamp, First Cloud AI Journey — tôi đã có cơ hội học hỏi, thực hành và áp
dụng kiến thức đã học ở trường vào môi trường làm việc thực tế.

Chương trình đi từ nền tảng quản trị tài khoản đến một sản phẩm hoàn chỉnh.
Những tuần đầu, tôi thiết lập quản trị tài khoản với AWS Budgets và IAM theo
nguyên tắc đặc quyền tối thiểu, sau đó xây dựng tầng mạng và tính toán cốt lõi:
VPC với public/private subnet, EC2, S3 và RDS. Tiếp đó là DNS và phân phối nội
dung với Route 53 và CloudFront, mô hình hoá dữ liệu NoSQL trên DynamoDB, cùng
bộ công cụ di trú và khôi phục sau thảm hoạ của AWS.

Giai đoạn giữa tập trung vào tự động hoá và vận hành — Lambda, CloudFormation và
AWS CDK để dựng hạ tầng lặp lại được, Systems Manager và Grafana cho công việc
vận hành hằng ngày — song song với các dịch vụ bảo mật cốt lõi: WAF, GuardDuty,
Security Hub, Macie và mã hoá bằng KMS. Sau đó tôi chuyển sang container và điều
phối luồng công việc với Docker, Amazon ECS và Step Functions, rồi đến phân rã
microservice, thiết kế hướng sự kiện và một ứng dụng CRUD serverless có truy vết
bằng AWS X-Ray. Các tuần cuối bổ sung kỹ năng container mức production trên ECS
và EKS, phân tích dữ liệu với Glue, Athena và QuickSight, cùng vòng đời học máy
trên Amazon SageMaker.

Sản phẩm chính của tôi là **Hệ thống Quản lý Thông tin Sân bay**, một kiến trúc
ba lớp trên AWS nhằm tự động hoá công việc mà các sân bay hiện vẫn làm thủ công:
đọc thẻ lên máy bay và thẻ hành lý. Tôi đã thiết kế kiến trúc, biên soạn thành
một workshop song ngữ và cài đặt một prototype hoạt động được của pipeline xử lý
tài liệu — tải lên S3 qua Presigned URL, hàng đợi hướng sự kiện, trích xuất OCR,
đối chiếu với lịch bay trong PostgreSQL và gửi thông báo tự động cho hành khách,
kèm Dead Letter Queue và cảnh báo CloudWatch cho luồng xử lý lỗi.

Qua công việc này, tôi đã nâng cao các kỹ năng về **thiết kế kiến trúc đám mây,
bảo mật mạng và danh tính, container hoá, phát triển hướng sự kiện và
serverless, mô hình hoá dữ liệu quan hệ, hạ tầng dưới dạng mã, viết tài liệu kỹ
thuật bằng hai ngôn ngữ, cùng kiểm thử và kiểm chứng có hệ thống**.

Để nhìn nhận khách quan về quá trình thực tập, tôi tự đánh giá theo các tiêu chí
sau:

| STT | Tiêu chí | Mô tả | Tốt | Khá | Trung bình |
| --- | -------- | ----- | --- | --- | ---------- |
| 1   | **Kiến thức & kỹ năng chuyên môn** | Hiểu biết về lĩnh vực, vận dụng kiến thức vào thực tế, sử dụng công cụ, chất lượng công việc | ☑ | ☐ | ☐ |
| 2   | **Khả năng học hỏi** | Khả năng tiếp thu kiến thức mới và học nhanh | ☑ | ☐ | ☐ |
| 3   | **Tính chủ động** | Chủ động tìm việc, không chờ được giao | ☑ | ☐ | ☐ |
| 4   | **Tinh thần trách nhiệm** | Hoàn thành công việc đúng hạn và đảm bảo chất lượng | ☑ | ☐ | ☐ |
| 5   | **Tính kỷ luật** | Tuân thủ thời gian, nội quy và quy trình làm việc | ☑ | ☐ | ☐ |
| 6   | **Tinh thần cầu tiến** | Sẵn sàng tiếp nhận góp ý và tự hoàn thiện | ☑ | ☐ | ☐ |
| 7   | **Kỹ năng giao tiếp** | Trình bày ý tưởng và báo cáo công việc rõ ràng | ☑ | ☐ | ☐ |
| 8   | **Làm việc nhóm** | Phối hợp hiệu quả với đồng nghiệp và tham gia hoạt động nhóm | ☑ | ☐ | ☐ |
| 9   | **Tác phong chuyên nghiệp** | Tôn trọng đồng nghiệp, đối tác và môi trường làm việc | ☑ | ☐ | ☐ |
| 10  | **Kỹ năng giải quyết vấn đề** | Nhận diện vấn đề, đề xuất giải pháp, thể hiện sự sáng tạo | ☑ | ☐ | ☐ |
| 11  | **Đóng góp cho dự án/nhóm** | Hiệu quả công việc, ý tưởng mới, được nhóm ghi nhận | ☑ | ☐ | ☐ |
| 12  | **Đánh giá chung** | Đánh giá tổng thể cả quá trình thực tập | ☑ | ☐ | ☐ |

{{% notice note %}}
Phần tự đánh giá trên đã được hoàn thiện với mức đánh giá **Tốt** cho tất cả các tiêu chí, thể hiện sự nỗ lực nghiêm túc và kết quả đạt được trong suốt kỳ thực tập.
{{% /notice %}}

### Điểm cần cải thiện

Những điểm dưới đây xuất phát từ chính thực trạng công việc của tôi, và là các
khoảng trống tôi dự định khắc phục tiếp theo:

* **Triển khai trên tài khoản AWS thật.** Prototype của tôi chạy hoàn toàn trên
  các dịch vụ giả lập. Tôi chưa vận hành kiến trúc này trên hạ tầng thật, nơi
  chi phí, chi tiết chính sách IAM và đặc thù từng region đều quan trọng theo
  cách mà môi trường cục bộ không thể hiện được.
* **Chạy pipeline triển khai thật.** Tôi đã viết các định nghĩa CodeBuild, ECR
  và CodeDeploy nhưng chưa từng thực thi chúng. Việc chứng kiến một bản triển
  khai blue/green tự rollback khi health check thất bại vẫn là điều tôi mới chỉ
  đọc chứ chưa tận mắt thấy.
* **Kiểm thử sớm hơn, bớt dựa vào đọc rà soát.** Hai lỗi đáng kể trong dự án —
  một luật tường lửa khiến tầng biên không khởi động được, và một endpoint trả
  về dữ liệu hành khách mà không cần token — đã vượt qua mọi bước rà soát tĩnh
  và chỉ lộ ra khi tôi chạy toàn bộ hệ thống lần đầu. Tôi cần dựng và chạy sớm
  hơn thay vì tin vào việc đọc kỹ.
