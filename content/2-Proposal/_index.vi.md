---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Hệ thống Quản lý Thông tin Sân bay
## Tự động hoá nhập liệu thẻ lên máy bay và thẻ hành lý bằng pipeline OCR serverless trên AWS

### 1. Tóm tắt

Mỗi ngày sân bay phải xử lý một lượng lớn tài liệu dạng hình ảnh — thẻ lên máy
bay, thẻ hành lý, hộ chiếu. Phần lớn dữ liệu này vẫn được nhân viên nhập tay vào
hệ thống nội bộ: chậm, dễ sai sót và không thể mở rộng trong giờ cao điểm.

Đề xuất này mô tả một hệ thống ba lớp trên AWS nhằm loại bỏ bước thủ công đó.
Hành khách hoặc nhân viên tải tài liệu lên; hệ thống trích xuất dữ liệu bằng
OCR, đối chiếu với lịch bay và tự động phát hành các thông báo tương ứng.

Thiết kế tuân theo AWS Well-Architected Framework: tầng public chứa bộ cân bằng
tải và VPC Endpoint, tầng private chứa container ứng dụng và cơ sở dữ liệu,
triển khai trên hai Availability Zone. Pipeline xử lý tài liệu hoạt động theo sự
kiện và được tách rời bằng hàng đợi, nên một đợt tải lên đột biến sẽ được hấp
thụ thay vì bị mất; tài liệu không đọc được sẽ bị cách ly để xem xét thay vì
thất bại âm thầm.

Một prototype hoạt động được của pipeline đã được xây dựng và kiểm chứng; phần 5
của báo cáo trình bày kiến trúc và phần 5.6 hướng dẫn cách chạy.

### 2. Vấn đề

#### Vấn đề là gì?

+ **Nhập liệu thủ công không thể mở rộng.** Mỗi thẻ lên máy bay xử lý tại quầy
  đều do con người gõ vào. Giờ cao điểm lại chính là lúc nhân viên ít thời gian
  nhất.
+ **Gõ tay sinh ra sai sót.** Một số ghế, cổng hay số hộ chiếu nhập sai sẽ lan
  sang các hệ thống phía sau và rất tốn công truy vết ngược.
+ **Hành khách được thông báo muộn, hoặc không được thông báo.** Thông tin trễ
  chuyến, đổi cổng, huỷ chuyến phụ thuộc vào việc ai đó nhớ phải gửi đi.
+ **Không có bản ghi thống nhất cho một tài liệu.** Ảnh đã tải lên, dữ liệu đã
  trích xuất và thẻ lên máy bay tương ứng nằm ở những nơi khác nhau, nên rất khó
  giải quyết khi có tranh chấp.

#### Giải pháp

Một pipeline hướng sự kiện, biến ảnh tải lên thành dữ liệu có cấu trúc và liên
kết với nhau mà không cần gõ phím:

1. API cấp **Presigned URL**; client tải thẳng lên Amazon S3, nên dữ liệu tài
   liệu không đi qua tầng ứng dụng.
2. Sự kiện `ObjectCreated` rơi vào hàng đợi **Amazon SQS**, tách rời việc tải
   lên khỏi việc xử lý.
3. Một hàm **Lambda** gọi **Amazon Textract** để trích xuất tên hành khách, số
   hiệu chuyến bay, giờ khởi hành và giờ đến, số ghế, cổng, giờ lên máy bay, số
   hộ chiếu và dữ liệu mã vạch.
4. Một hàm **Lambda** thứ hai ghi kết quả vào **Amazon RDS for PostgreSQL**, đối
   chiếu đúng chuyến bay và hành khách, rồi phát hành thẻ lên máy bay.
5. **Amazon SES** gửi email xác nhận cho hành khách; **Amazon EventBridge** kích
   hoạt bản tổng hợp định kỳ về các chuyến bay sắp tới.

Tài liệu không đọc được sẽ được thử lại, sau đó chuyển vào **Dead Letter Queue**
và kích hoạt cảnh báo **CloudWatch**, để lỗi được nhìn thấy chứ không bị bỏ sót.

#### Lợi ích và hiệu quả đầu tư

| Lợi ích | Tác động |
|---|---|
| Không còn nhập liệu thủ công | Thời gian nhân viên chuyển từ gõ máy sang phục vụ hành khách |
| Giảm sai sót nhập liệu | Việc trích xuất là tất định và kèm theo độ tin cậy |
| Thông báo trở nên tự động | Trễ chuyến, đổi cổng, lên máy bay, huỷ chuyến đều theo sự kiện |
| Vết dữ liệu đầy đủ | Ảnh, dữ liệu trích xuất và thẻ lên máy bay liên kết bằng khoá ngoại |
| Hấp thụ được tải cao điểm | Hàng đợi đệm các đợt đột biến; container mở rộng theo nhu cầu |
| Bảo vệ dữ liệu cá nhân | Số hộ chiếu được phát hiện và che trước khi lưu |

Hệ thống không tiêu tốn công suất cố định khi rảnh: Lambda, SQS, S3 và Textract
tính phí theo mức dùng, còn tầng container co giãn theo lưu lượng.

### 3. Kiến trúc giải pháp

![Kiến trúc](/images/5-Workshop/airport-architecture-overview.png)

Lưu lượng đi vào qua Route 53 và CloudFront, được AWS WAF lọc, rồi Application
Load Balancer định tuyến đến các container chạy trên ECS/Fargate trong private
subnet. Các container này truy cập S3, Textract và SES qua VPC Endpoint, nên
không có lưu lượng nào rời khỏi mạng AWS.

#### Các dịch vụ AWS sử dụng

| Tầng | Dịch vụ |
|---|---|
| **Biên & phân phối** | Route 53, CloudFront, AWS WAF |
| **Danh tính** | Amazon Cognito (JWT; nhóm phân tách hành khách và nhân viên) |
| **Tính toán** | Amazon ECS trên AWS Fargate, AWS Lambda |
| **Lưu trữ & dữ liệu** | Amazon S3, Amazon RDS for PostgreSQL (Multi-AZ), Amazon DynamoDB |
| **Tích hợp** | Amazon SQS (+ Dead Letter Queue), Amazon EventBridge |
| **Học máy** | Amazon Textract |
| **Gửi thông điệp** | Amazon SES |
| **Mạng** | Amazon VPC, Internet Gateway, VPC Endpoint (Gateway và Interface) |
| **Bảo mật & cấu hình** | AWS KMS, AWS Secrets Manager, AWS Systems Manager Parameter Store, Amazon Macie |
| **Giám sát** | Amazon CloudWatch và CloudWatch Logs |
| **Pipeline triển khai** | AWS CodeBuild, Amazon ECR, AWS CodeDeploy |

#### Thiết kế thành phần

+ **API nghiệp vụ (ECS/Fargate).** Quản lý chuyến bay, hành khách và thẻ lên máy
  bay; cấp Presigned URL. Không lưu trạng thái nên mở rộng ngang được. Xác thực
  token Cognito ở mọi request và giới hạn mỗi truy vấn theo người gọi.
+ **Hàm gửi job OCR (Lambda).** Đọc hàng đợi tài liệu, giành quyền xử lý object
  trong DynamoDB để message gửi lại không làm Textract tính phí hai lần, gọi
  Textract và đẩy kết quả đi.
+ **Hàm xử lý kết quả (Lambda).** Lưu dữ liệu trích xuất, che dữ liệu cá nhân
  trong văn bản thô, đối chiếu với hồ sơ chuyến bay và hành khách, rồi tạo thông
  báo.
+ **Hàm theo lịch (Lambda + EventBridge).** Gửi cho mỗi hành khách bản tổng hợp
  các chuyến bay sắp tới theo lịch hàng tháng.
+ **Kho dữ liệu quan hệ (RDS PostgreSQL).** Sáu bảng chuẩn hoá — sân bay, chuyến
  bay, hành khách, tài liệu, thẻ lên máy bay, thông báo — trong đó
  `boarding_passes` là bảng nối chuyến bay, hành khách và tài liệu.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

| Giai đoạn | Phạm vi |
|---|---|
| 1 — Nền tảng | VPC với public/private subnet trên hai AZ, IAM role, security group, VPC Endpoint |
| 2 — Tầng dữ liệu | RDS PostgreSQL Multi-AZ, lược đồ và ràng buộc, thông tin đăng nhập trong Secrets Manager |
| 3 — Tầng ứng dụng | API đóng gói container, ECS service sau ALB, Cognito user pool và nhóm |
| 4 — Pipeline tài liệu | S3 bucket và thông báo sự kiện, hàng đợi SQS với redrive policy, hai hàm Lambda, tích hợp Textract |
| 5 — Thông báo | Định danh và mẫu SES, lịch EventBridge, lịch sử thông báo |
| 6 — Giám sát | Chỉ số CloudWatch, log tập trung, cảnh báo Dead Letter Queue |
| 7 — Triển khai | CodeBuild, ECR, CodeDeploy blue/green có rollback tự động |

**Yêu cầu kỹ thuật**

+ Tối thiểu hai Availability Zone cho bộ cân bằng tải, dịch vụ container và cơ
  sở dữ liệu.
+ Toàn bộ thành phần ứng dụng nằm trong private subnet; chỉ bộ cân bằng tải được
  truy cập công khai.
+ Truy cập S3, Textract và SES từ private subnet qua VPC Endpoint thay vì NAT
  Gateway — chi phí thấp hơn và bề mặt tấn công nhỏ hơn.
+ Mã hoá khi truyền trên toàn hệ thống, và mã hoá khi lưu cho S3, RDS và Secrets
  Manager bằng KMS.
+ Mọi request API đều phải xác thực; dữ liệu cá nhân chỉ chủ sở hữu hoặc nhân
  viên mới đọc được.
+ Xử lý tài liệu phải idempotent, vì SQS giao nhận theo cơ chế at-least-once.

### 5. Kế hoạch và các mốc thời gian

Bám theo mười ba tuần thực tập:

| Tuần | Mốc công việc |
|---|---|
| 1–2 | Quản trị tài khoản AWS, IAM, nền tảng mạng và tính toán |
| 3–4 | DNS, CDN, NoSQL, tự động hoá bằng CloudFormation và CDK, các dịch vụ bảo mật |
| 5 | Container, ECS, điều phối luồng công việc, phân tích chi phí |
| 6–7 | Microservice và thiết kế hướng sự kiện; ứng dụng serverless; chọn chủ đề workshop |
| 8 | Thực hành container mức production trên ECS và Fargate cùng pipeline CI/CD |
| 9–10 | Nền tảng phân tích dữ liệu và học máy; phác thảo kiến trúc |
| 11 | Hoàn thành các bài lab còn tồn; rà soát kiến trúc |
| 12 | Hoàn thiện workshop; cài đặt và kiểm chứng prototype đầu cuối |
| 13 | Viết và nộp báo cáo |

### 6. Dự toán chi phí

{{% notice warning %}}
Các con số dưới đây là ước tính của tôi cho một môi trường production nhỏ tại
`ap-southeast-1`, dựa trên bảng giá on-demand công bố. Đây là số liệu tham khảo,
không phải báo giá — hãy kiểm tra lại bằng AWS Pricing Calculator trước khi
quyết định, và thay ghi chú này bằng đường dẫn ước tính của chính bạn.
{{% /notice %}}

#### Chi phí hạ tầng

Giả định: xử lý 2.000 tài liệu mỗi tháng, 10.000 email thông báo, 50 GB lưu
lượng ra qua CloudFront, hai Fargate task chạy liên tục.

| Dịch vụ | Cấu hình | Ước tính/tháng (USD) |
|---|---|---|
| Amazon Textract | 2.000 trang, phân tích biểu mẫu | ~100,00 |
| Amazon RDS PostgreSQL | db.t4g.micro, Multi-AZ, 20 GB | ~50,00 |
| Amazon ECS trên Fargate | 2 task, 0,5 vCPU / 1 GB | ~36,00 |
| Application Load Balancer | 1 ALB, mức LCU thấp | ~20,00 |
| VPC Interface Endpoint | 2 endpoint | ~14,00 |
| AWS WAF | 1 web ACL, managed rule group | ~10,00 |
| Amazon CloudFront | 50 GB lưu lượng ra | ~5,00 |
| Amazon CloudWatch | Chỉ số, log, cảnh báo | ~5,00 |
| Amazon S3 | 50 GB cùng số request | ~2,00 |
| SQS, DynamoDB, Lambda, SES, Route 53 | Lưu lượng thấp, phần lớn trong free tier | ~5,00 |
| **Tổng cộng** | | **~247,00** |

**Nhận xét về chi phí**

+ **Textract chiếm tỷ trọng lớn nhất.** Khoảng 40% tổng chi phí, nên số lượng
  tài liệu là yếu tố quyết định. Giới hạn phân tích đúng các trường thực sự cần
  là hướng tối ưu đầu tiên.
+ **VPC Endpoint rẻ hơn NAT Gateway** ở quy mô này, đồng thời loại bỏ một đường
  đi ra Internet.
+ **RDS chạy một AZ giúp giảm một nửa chi phí cơ sở dữ liệu** và chấp nhận được
  cho môi trường không phải production, đổi lại mất khả năng tự chuyển đổi dự
  phòng.
+ Nên cấu hình cảnh báo AWS Budgets trước khi triển khai, để chi phí được kiểm
  soát bằng chính sách chứ không phải nhờ tình cờ phát hiện.

### 7. Đánh giá rủi ro

#### Ma trận rủi ro

| Rủi ro | Khả năng xảy ra | Mức ảnh hưởng | Đánh giá |
|---|---|---|---|
| OCR đọc sai một trường, sinh dữ liệu hành khách sai | Trung bình | Cao | **Cao** |
| Chi phí Textract vượt dự toán khi lưu lượng tăng | Trung bình | Trung bình | **Trung bình** |
| Dữ liệu cá nhân bị lưu mà không che | Thấp | Cao | **Cao** |
| Xử lý trùng do cơ chế giao nhận at-least-once | Trung bình | Trung bình | **Trung bình** |
| Cơ sở dữ liệu không truy cập được | Thấp | Cao | **Cao** |
| Lộ thông tin đăng nhập trong cấu hình | Thấp | Cao | **Cao** |
| Lưu lượng tăng đột biến vượt khả năng pipeline | Trung bình | Thấp | **Thấp** |

#### Biện pháp giảm thiểu

+ **Độ chính xác trích xuất.** Textract trả về độ tin cậy cho từng trường và
  được lưu kèm kết quả. Các kết quả có độ tin cậy thấp có thể chuyển sang cho
  người kiểm tra thay vì ghi nhận âm thầm, và văn bản thô được giữ lại để đối
  chiếu mọi bản ghi với ảnh gốc.
+ **Kiểm soát chi phí.** Cảnh báo AWS Budgets đặt riêng cho khoản Textract; một
  chỉ số CloudWatch theo dõi số tài liệu đã xử lý, để mức tăng trưởng được nhìn
  thấy trước khi hoá đơn về.
+ **Dữ liệu cá nhân.** Tài liệu tải lên được quét tìm dữ liệu cá nhân; số hộ
  chiếu, ngày sinh, địa chỉ email và số điện thoại được che trong văn bản lưu
  trữ, trong khi các trường có cấu trúc mà nghiệp vụ cần vẫn được giữ. Amazon
  Macie phân loại các object trong S3 một cách độc lập.
+ **Xử lý trùng.** Mỗi tài liệu được giành quyền xử lý trong DynamoDB bằng một
  lệnh ghi có điều kiện trước mọi lời gọi Textract, nên message gửi lại sẽ bị
  loại bỏ thay vì bị tính phí lần nữa.
+ **Tính sẵn sàng.** RDS chạy Multi-AZ có tự chuyển đổi dự phòng; dịch vụ
  container trải trên hai Availability Zone sau bộ cân bằng tải.
+ **Thông tin đăng nhập.** Thông tin đăng nhập cơ sở dữ liệu lấy từ Secrets
  Manager, mã hoá bằng KMS và nạp vào task lúc chạy — không bao giờ đưa vào
  repository hay ghi trong tệp môi trường.
+ **Tải cao.** Hàng đợi tách rời việc tải lên khỏi việc xử lý, nên đột biến chỉ
  làm hàng đợi dài thêm chứ không làm request thất bại.

#### Phương án dự phòng

+ **Tài liệu không xử lý được** sẽ thử lại ba lần, sau đó chuyển vào Dead Letter
  Queue và kích hoạt cảnh báo CloudWatch để xử lý thủ công. Bản ghi tài liệu lưu
  lại nguyên nhân lỗi.
+ **Bản triển khai lỗi** được tự động rollback: CodeDeploy chỉ chuyển lưu lượng
  sau khi health check đạt, và trả bộ cân bằng tải về task set trước đó nếu
  không đạt.
+ **Mất cơ sở dữ liệu** được bảo vệ bằng bản sao lưu tự động của RDS và khôi
  phục theo thời điểm; bản thân tài liệu vẫn nằm trong S3 và có thể xử lý lại.
+ **Textract gián đoạn** khiến tài liệu nằm chờ trong hàng đợi chứ không mất;
  việc xử lý tiếp tục khi dịch vụ hoạt động trở lại.

### 8. Kết quả mong đợi

#### Cải thiện về kỹ thuật

+ Loại bỏ việc nhập tay thẻ lên máy bay và thẻ hành lý đối với những tài liệu mà
  pipeline đọc được.
+ Thông báo chuyến bay trở thành thuộc tính của hệ thống, thay vì một việc ai đó
  phải nhớ làm.
+ Mỗi tài liệu tải lên đều có bản ghi bền vững và liên kết: ảnh trong S3, dữ
  liệu trích xuất trong PostgreSQL, và thẻ lên máy bay tham chiếu cả hai.
+ Lỗi trở nên nhìn thấy được. Tài liệu không đọc được sẽ kích hoạt cảnh báo thay
  vì biến mất.

#### Giá trị dài hạn

+ **Nền tảng để tự động hoá thêm.** Cùng một hình mẫu pipeline — Presigned URL,
  hàng đợi, trích xuất, đối chiếu, thông báo — có thể mở rộng cho hộ chiếu, thị
  thực và giấy tờ tuỳ thân mà không cần đổi kiến trúc.
+ **Tập dữ liệu đáng phân tích.** Các tài liệu đã xử lý tích luỹ thành nguồn dữ
  liệu phù hợp để phân tích độ chính xác trích xuất, lưu lượng giờ cao điểm và
  hiệu quả thông báo.
+ **Mức độ trưởng thành trong vận hành.** Idempotency, xử lý Dead Letter Queue,
  triển khai blue/green và phân quyền theo người gọi là những mẫu thiết kế có
  thể tái sử dụng, không phải tính năng dùng một lần của hệ thống này.
+ **Minh bạch chi phí.** Vì hệ thống hướng sự kiện, chi phí tăng theo số tài
  liệu được xử lý, giúp đo lường trực tiếp hiệu quả kinh tế của việc tự động hoá.
