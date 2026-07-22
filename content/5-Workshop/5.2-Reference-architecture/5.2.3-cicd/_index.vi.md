---
title : "Pipeline CI/CD"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.2.3 </b> "
---

Quy trình tích hợp và triển khai liên tục (CI/CD) hoạt động **độc lập với luồng nghiệp vụ chính**, được kích hoạt mỗi khi có thay đổi mã nguồn:

1. **GitHub** lưu trữ mã nguồn của service backend; mỗi commit/merge kích hoạt pipeline.
2. **AWS CodeBuild** biên dịch mã nguồn, chạy test, và đóng gói thành Docker image.
3. **Amazon ECR** lưu trữ image đã build theo từng phiên bản (tag).
4. **AWS CodeDeploy** lấy image mới nhất từ ECR và triển khai (rolling deployment) lên ECS Cluster mà không gây gián đoạn dịch vụ.

```text
GitHub  →  CodeBuild  →  ECR  →  CodeDeploy  →  ECS Cluster
 commit     build+test    image    rolling        Fargate tasks
                          (tag)    deployment
```

{{% notice note %}}
Mô hình này giúp đội phát triển triển khai phiên bản mới nhanh chóng, an toàn và có thể rollback khi cần, đúng theo tinh thần DevOps hiện đại.
{{% /notice %}}
