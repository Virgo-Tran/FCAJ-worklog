---
title: "Blog 1: Tối ưu hóa chi phí Amazon EKS"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Tối ưu hóa chi phí Amazon EKS: Từ "Sốc hóa đơn" đến tiết kiệm 40-60%

> **Chủ đề:** *Container Infrastructure Cost Optimization*  
> **Tác giả:** Trần Quốc Khánh  
> **Cộng đồng đăng tải:** [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)  

*Tại sao hầu hết các cụm EKS hiện nay đều đang chi trả thừa chi phí, và lộ trình 7 bước giúp bạn kiểm soát lại ngân sách hạ tầng.*

---

Nếu bạn từng mở hóa đơn AWS vào đầu tháng và giật mình thấy chi phí cho EKS cao hơn nhiều so với dự kiến, bạn không hề đơn độc. Đây là phàn nàn phổ biến nhất trong cộng đồng DevOps: các đội ngũ kỹ thuật áp dụng EKS vì tính chuẩn hóa và sức mạnh của Kubernetes, để rồi vài tháng sau mới phát hiện ra phần lớn chi phí đang nuôi các worker node hầu như không hoạt động.

Một báo cáo tối ưu hóa Kubernetes cho thấy một con số đáng suy ngẫm: **trung bình một cụm cluster chỉ sử dụng khoảng 8% CPU và 20% Memory đã cấp phát**, trong khi tình trạng cấp phát thừa CPU (CPU over-provisioning) đã tăng lên 69% theo từng năm. Nói cách khác, phần lớn ngân sách EKS của bạn đang trả cho năng lực nhàn rỗi chứ không phải lượng traffic thực tế.

---

### 1. Cấu trúc chi phí thực sự của EKS — Không chỉ là $0.10/giờ

Nhiều người mới bắt đầu với EKS thường chỉ nhớ con số quen thuộc: **$0.10/giờ** cho mỗi cụm Control Plane (khoảng $73/tháng). Nhưng đó chỉ là phần nổi của tảng băng trôi. Chi phí thực sự của một cụm EKS tích lũy từ nhiều tầng:

* **Worker Node Compute:** Chi phí lớn nhất, đến từ các thể hiện EC2 chạy node hoặc chi phí vCPU/Memory khi chạy Pod trên AWS Fargate.
* **Phí Control Plane:** $0.10/giờ/cluster, cộng thêm phí hỗ trợ mở rộng (Extended Support) **$0.60/giờ** nếu cụm chạy phiên bản Kubernetes đã hết hạn hỗ trợ chuẩn — riêng khoản này có thể tăng thêm **$432/tháng** cho mỗi cụm chạy 24/7.
* **Phí EKS Auto Mode:** Nếu bật tính năng tự động quản lý compute, bạn sẽ trả thêm phí tính theo giây cho mỗi instance EC2 được cấp phát.
* **Chi phí Mạng (Networking):** Load Balancer, NAT Gateway, và đặc biệt là chi phí truyền dữ liệu (Data Transfer) giữa các Availability Zone hoặc ra Internet.
* **Lưu trữ (Storage):** EBS Volumes cho persistent storage và chi phí lưu trữ log trên CloudWatch.

---

### 2. Case Study: Bài học $10,000 từ một đội ngũ kỹ thuật

Một câu chuyện điển hình trong cộng đồng kỹ thuật: Một nhóm phát triển chuyển sang dùng EKS Auto Mode với mong muốn "không còn lo lắng về hạ tầng". Vài tuần sau, họ phát hiện một service đơn lẻ tiêu tốn khoảng **$240/tháng** — trong khi mức sử dụng thực tế chỉ cần một phần nhỏ con số đó. Tính trên toàn bộ hệ thống, chi phí lãng phí ước tính lên đến hơn **$1,100/tháng**, chỉ vì khai báo `requests` tài nguyên quá cao so với nhu cầu thực tế.

Sau khi điều chỉnh lại kích thước tài nguyên (right-sizing) dựa trên mức sử dụng thực, đội ngũ đã **cắt giảm được 40% hóa đơn EKS**.

---

### 3. Tại sao Karpenter không tự động giải quyết được vấn đề?

Karpenter là bộ tự động mở rộng (Autoscaler) thế hệ mới cho EKS thay thế cho Cluster Autoscaler truyền thống. Karpenter gọi trực tiếp EC2 API, chọn loại instance tối ưu nhất cho Pod đang chờ và có thể chuẩn bị một node mới chỉ trong 45-60 giây.

Tuy nhiên, Karpenter có một điểm hạn chế về thiết kế mà nhiều đội ngũ bỏ qua: **Karpenter hoàn toàn tin tưởng vào khai báo `resource requests` trên Pod**. Nếu Pod yêu cầu 4 vCPU nhưng thực tế chỉ dùng 0.4 vCPU, Karpenter vẫn sẽ tạo một node đáp ứng đủ 4 vCPU.Nói cách khác, Karpenter tối ưu hóa việc *cấp phát* (provisioning) chứ không tối ưu hóa việc *khai báo* (declaration).

---

### 4. Lộ trình 7 bước tối ưu hóa chi phí EKS

Thứ tự thực hiện chuẩn được khuyến nghị bởi các chuyên gia tối ưu hóa chi phí Kubernetes:

1. **Right-size tài nguyên Pod trước tiên:** Sử dụng CloudWatch Container Insights để đo lường mức sử dụng CPU/Memory thực tế, sau đó điều chỉnh `requests` phù hợp.
2. **Triển khai Karpenter cho Dynamic Node Provisioning:** Thay thế cho Auto Scaling Group cố định.
3. **Áp dụng Spot Instances:** Sử dụng Spot cho môi trường Non-Production, các stateless service hoặc queue job (tiết kiệm đến 90%).
4. **Chuyển sang kiến trúc Graviton (ARM):** Các instance Graviton thường rẻ hơn đáng kể so với x86 tương đương.
5. **Thêm VPC Endpoints:** Sử dụng VPC Endpoints cho các dịch vụ AWS thường dùng (S3, ECR) để loại bỏ phí NAT Gateway.
6. **Tối ưu hóa EBS Volumes:** Nâng cấp từ `gp2` sang `gp3` và xóa các volume mồ côi (orphaned volumes).
7. **Gộp Application Load Balancers:** Sử dụng chung một Ingress Controller thay vì mỗi service dùng một ALB riêng.

---

### 5. So sánh các mô hình mua Compute EKS

| Mô hình mua | Mức tiết kiệm | Phù hợp nhất cho | Rủi ro chính |
| :--- | :--- | :--- | :--- |
| **On-Demand** | 0% (Giá niêm yết) | Workload thử nghiệm, traffic không ổn định | Chi phí cao nhất, không cam kết |
| **Spot Instances** | Lên đến 90% | Queue jobs, batch processing, stateless apps | Có thể bị thu hồi bất kỳ lúc nào |
| **Compute Savings Plans** | Thường từ 30-50% | Workload ổn định, áp dụng cho EC2/Fargate/Lambda | Cam kết 1-3 năm |
| **Reserved Instances** | Tương đương Savings Plans | Workload cố định theo loại instance | Ít linh hoạt hơn Savings Plans |

---

### 6. So sánh chi phí: EKS vs. GKE

| Hạng mục | Amazon EKS | Google GKE |
| :--- | :--- | :--- |
| **Phí Control Plane** | ~$0.10/giờ (~$73/tháng) | Zonal cluster miễn phí; Regional cluster ~$0.10/giờ |
| **On-demand Node** | Cạnh tranh, rẻ hơn ở một số dòng instance | Tương đương, tùy thuộc vào loại instance |
| **Discounted Capacity** | Spot Instances (Giảm tới 90%) | Preemptible/Spot VMs (Giảm tới 80%) |
| **Chế độ tự động hóa** | EKS Auto Mode (Có phí quản lý riêng) | GKE Autopilot (Tính phí theo resource request) |

---

### 7. Con số tiết kiệm thực tế ở quy mô Production

Xét một ví dụ thực tế: Một cụm EKS đang tiêu tốn khoảng **$85,000/tháng**. Sau khi hoàn thành trọn vẹn lộ trình 7 bước ở trên, mức tiết kiệm hàng tháng ước tính đạt **$40,000 - $55,000/tháng** — giảm gần một nửa hóa đơn ban đầu.

---

### 8. Kết luận: Tối ưu hóa là một quy trình, không phải việc làm một lần

Chi phí EKS không tăng đột biến qua đêm — nó tích lũy dần theo thời gian khi tài nguyên được khai báo dư thừa "cho an toàn" và các môi trường dev bị bỏ quên. Hãy bắt đầu bằng việc đo lường thực tế, tự động hóa cấp phát, tận dụng năng lực giá rẻ và chỉ cam kết dài hạn khi đã tối ưu xong baseline.

