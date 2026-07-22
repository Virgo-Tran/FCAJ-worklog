---
title : "Dọn dẹp"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3 </b> "
---

#### Dừng hệ thống

```bash
cd TranQuocKhanh-Project
docker compose down
```

Lệnh này dừng và xoá các container nhưng vẫn giữ volume cơ sở dữ liệu, nên lần
`docker compose up` tiếp theo sẽ tiếp tục với dữ liệu cũ.

#### Xoá toàn bộ

Để xoá luôn volume PostgreSQL — lần khởi động sau sẽ tạo lại schema và nạp lại
dữ liệu mẫu:

```bash
docker compose down -v
```

Để giải phóng cả dung lượng đĩa của các image đã build:

```bash
docker compose down -v --rmi local
```

{{% notice note %}}
Không có thành phần nào của prototype chạy trên AWS, nên không có chi phí đám mây
cần dừng và không có tài nguyên AWS nào cần xoá. Nếu bạn đã chuyển sang
`OCR_PROVIDER=textract` hoặc `NOTIFIER=ses`, các lời gọi đó có đi đến tài khoản
thật — hãy kiểm tra mức sử dụng Textract và SES trong Cost Explorer.
{{% /notice %}}

#### Hướng phát triển tiếp theo

Prototype cố ý dừng lại trước khi hoàn thiện toàn bộ kiến trúc. Các phần còn
thiếu, xếp theo mức độ ưu tiên:

1. **Xác thực bằng Cognito.** Hiện API nhận `owner_id` như một tham số; hệ thống
   triển khai thật sẽ xác thực JWT và đọc claim `sub`. Hiện chưa có bất kỳ cơ chế
   phân quyền nào, vì vậy tuyệt đối không expose ra ngoài localhost.
2. **Tác vụ định kỳ hàng tháng qua EventBridge.** Bước 10 trong luồng dữ liệu —
   một Lambda chạy theo lịch, truy vấn RDS và gửi thông báo hàng loạt — chưa được
   cài đặt.
3. **Infrastructure as Code.** Một stack CDK hoặc Terraform để tạo đủ 18 dịch vụ thật.
4. **Giao diện người dùng.** Trang web tĩnh trên S3 + CloudFront mà hành khách và
   nhân viên thực sự sử dụng.
5. **Đối chiếu dữ liệu tốt hơn.** Hiện việc so khớp là chính xác tuyệt đối theo số
   hiệu chuyến bay và tên (không phân biệt hoa thường); dữ liệu thực tế cần cơ chế
   so khớp linh hoạt hơn và luồng để con người kiểm tra lại các kết quả có độ tin
   cậy thấp.
