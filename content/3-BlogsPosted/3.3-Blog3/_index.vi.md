---
title: "Blog 3: AWS Lambda Durable Functions"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# AWS Lambda Durable Functions: Xây dựng luồng công việc tin cậy không cần Step Functions

> **Chủ đề:** *Next-Generation Serverless Workflows*  
> **Tác giả:** Trần Quốc Khánh  
> **Cộng đồng đăng tải:** [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)  

*Tính năng mới nhất từ re:Invent 2025 giúp Lambda phá vỡ giới hạn 15 phút — Hướng dẫn cơ chế hoạt động và thời điểm nên áp dụng.*

---

Nếu bạn từng viết một hàm Lambda cần chờ con người phê duyệt, chờ phản hồi từ webhook trong nhiều giờ, hoặc điều phối một pipeline AI kéo dài cả ngày, bạn chắc chắn đã nếm trải giới hạn thời gian chạy 15 phút của Lambda. Giải pháp truyền thống luôn là AWS Step Functions — nhưng điều đó đồng nghĩa với việc bạn phải học ngôn ngữ Amazon States Language (ASL), tách biệt logic khỏi codebase quen thuộc và quản lý một tầng điều phối riêng biệt.

Tại **re:Invent 2025**, AWS đã ra mắt **Lambda Durable Functions** — một cách tiếp cận hoàn toàn mới: Xây dựng các luồng công việc nhiều bước (multi-step workflows) ngay bên trong Lambda bằng chính ngôn ngữ lập trình bạn đang dùng, không cần đến ASL.

---

### 1. Lambda Durable Functions là gì?

Về bản chất, một Durable Function vẫn là một hàm Lambda bình thường — giữ nguyên event handler và các tích hợp sẵn có. Điểm khác biệt là bạn tích hợp thêm thư viện mã nguồn mở **Durable Execution SDK** (hỗ trợ JavaScript/TypeScript, Python, và Java từ tháng 04/2026). Thư viện này cung cấp một đối tượng `DurableContext` với các thao tác đặc biệt:
* `step()`: Một đơn vị công việc độc lập.
* `wait()`: Tạm dừng theo thời gian.
* `callback()`: Chờ tín hiệu từ bên ngoài.
* `parallel()` / `map()`: Thực thi song song.
* Child contexts: Các sub-workflow con.

Nhờ mô hình này, một "durable execution" có thể kéo dài từ **vài phút đến 1 năm** qua nhiều lần gọi Lambda khác nhau mà **không phát sinh chi phí compute trong thời gian chờ**.

---

### 2. Cơ chế Checkpoint và Replay hoạt động như thế nào?

Đây là cơ chế kỹ thuật cốt lõi diễn ra qua 4 giai đoạn:

1. **Start:** Luồng chạy bắt đầu khi hàm Lambda được kích hoạt tính năng durable execution.
2. **Checkpoint:** Mỗi khi code gặp một thao tác như `step()`, SDK sẽ thực thi, đợi kết quả và lưu vết (checkpoint) kết quả vào một nhật ký (journal) do AWS quản lý trước khi chuyển sang bước tiếp theo.
3. **Suspend & Resume:** Khi gặp lệnh `wait()` hoặc `callback()`, lần thực thi Lambda hiện tại kết thúc hoàn toàn (dừng tính phí compute). Khi điều kiện được đáp ứng, một invocation mới sẽ được tự động kích hoạt.
4. **Replay:** Khi invocation mới bắt đầu, SDK chạy lại toàn bộ handler từ đầu. Tuy nhiên, đối với các step đã hoàn thành, SDK chỉ tải kết quả đã lưu trong journal chứ không chạy lại — giúp "tua nhanh" qua các bước cũ chỉ trong vài miligiây.

> **Yêu cầu bắt buộc — Tính Quyết định (Determinism):**  
> Do cơ chế Replay chạy lại handler, bất kỳ mã nguồn không quyết định nào (như tạo UUID ngẫu nhiên, lấy timestamp hiện tại, gọi API bên ngoài) đều phải được bọc bên trong một thao tác `step()`.

---

### 3. Giới hạn cần lưu ý trước khi bắt đầu

* **Chỉ áp dụng khi tạo mới:** Bạn chỉ có thể bật durable execution khi tạo một hàm Lambda mới; không thể chuyển đổi một hàm hiện có.
* **Cần quyền IAM riêng:** Yêu cầu quyền `lambda:CheckpointDurableExecution` và `lambda:GetDurableExecutionState`.
* **Khu vực hỗ trợ:** Hiện đã hỗ trợ tại 15 AWS Regions (bao gồm US East/West, Châu Âu và Châu Á - Thái Bình Dương).

---

### 4. So sánh trực diện: Durable Functions vs. Step Functions

| Tiêu chí | Lambda Durable Functions | AWS Step Functions |
| :--- | :--- | :--- |
| **Cách định nghĩa Workflow** | Code thuần (JS/TS, Python, Java) | Amazon States Language (JSON/YAML) |
| **Tích hợp dịch vụ AWS khác** | Chủ yếu qua Lambda-to-Lambda | Tích hợp sẵn trực tiếp (SQS, DynamoDB, ECS, SNS) |
| **Khả năng quan sát (Observability)** | Qua CLI/SDK; chưa có giao diện trực quan | Visual workflow builder, theo dõi dạng cây dễ dàng |
| **Mô hình chi phí** | $8/triệu thao tác durable + chi phí Lambda | Tính phí theo lượt chuyển trạng thái (state transition) |
| **Thời gian chờ tối đa** | Lên đến 1 năm (không tốn phí compute khi chờ) | Không giới hạn (Standard Workflow, có phí lưu trữ) |
| **Phù hợp nhất cho** | Workflow chủ yếu xử lý bằng Lambda | Điều phối đa dịch vụ, workflow phức tạp nhiều team |

---

### 5. Bài toán chi phí: Rẻ hơn tới 4 lần

Durable Functions tính phí $8 cho mỗi 1 triệu thao tác (step, wait, callback), cộng với phí lưu dữ liệu checkpoint và chi phí compute thực tế của Lambda. Trong khi đó, Step Functions Standard tính phí ~$0.000025 mỗi state transition. 

Đối với các luồng công việc phê duyệt đơn giản (vài step, 1 lần chờ), các thử nghiệm thực tế cho thấy **Durable Functions có thể rẻ hơn tới 4 lần so với Step Functions Standard** do loại bỏ được chi phí quản lý chuyển trạng thái xếp chồng.

---

### 6. Khi nào nên chọn dịch vụ nào?

* **Nên chọn Lambda Durable Functions khi:**
  - Luồng công việc chủ yếu là "Lambda gọi Lambda" kèm theo các quãng nghỉ.
  - Bạn muốn giữ toàn bộ logic điều phối trong cùng một codebase bằng ngôn ngữ lập trình hiện tại.
  - Xây dựng pipeline AI nhiều bước (gọi model -> chờ kết quả -> xử lý tiếp).

* **Nên chọn AWS Step Functions khi:**
  - Cần điều phối trực tiếp nhiều dịch vụ AWS khác nhau mà không muốn viết các hàm Lambda trung gian.
  - Cần giao diện sơ đồ trực quan (Visual builder) cho nhiều team theo dõi.
  - Tổ chức đã đầu tư mạnh vào Step Functions và cần tính nhất quán vận hành.

---

### 7. Kết luận: Mô hình kết hợp (Hybrid Model)

Nhiều kiến trúc hiện đại kết hợp cả hai: Sử dụng **Step Functions** cho mức điều phối tổng thể cấp cao giữa các dịch vụ AWS, và sử dụng **Durable Functions** bên trong từng nhánh Lambda cụ thể để xử lý các logic phức tạp nhiều bước.

Quy tắc đơn giản: Nếu luồng công việc của bạn có thể mô tả là một hàm tuần tự kèm vài điểm dừng, hãy thử **Durable Functions**. Nếu đó là một sơ đồ điều phối đa dịch vụ cần giao diện trực quan, **Step Functions** vẫn là lựa chọn vững chắc nhất.

