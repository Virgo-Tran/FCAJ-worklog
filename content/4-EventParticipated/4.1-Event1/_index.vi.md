---
title: "FCAJ Community Day"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Báo cáo tổng kết: FCAJ Community Day

* **Thời gian:** 08:30 – 12:00, Thứ Bảy, ngày 23/05/2026  
* **Địa điểm:** Tầng 26, Bitexco Financial Tower, Quận 1, TP. Hồ Chí Minh  
* **Ban tổ chức:** AWS Study Group / FCAJ Organizers (Huỳnh Hoàng Long, Thiên Lư, Trần Đại Vĩ)  
* **Trang sự kiện:** [Luma - FCAJ Community Day](https://luma.com/ubaur0y5)  
* **Vai trò:** Người tham dự (Thực tập sinh FCAJ)  

---

### Mục tiêu sự kiện

* Giao lưu, kết nối cộng đồng thực tập sinh và kỹ sư thuộc chương trình **First Cloud AI Journey (FCAJ)** tại AWS Việt Nam.
* Cập nhật các xu hướng kỹ thuật mới nhất về AI/ML, Serverless Workflows, Multi-Agent Systems và Tối ưu hóa hạ tầng Cloud.
* Học hỏi kinh nghiệm thực chiến từ các chuyên gia hàng đầu và lắng nghe hành trình biến ý tưởng thành sản phẩm thực tế từ các đội thi Hackathon.

---

### Diễn giả & Chủ đề chia sẻ

1. **Tinh Truong** — *Context Is Everything: Making AI Actually Work for You*
2. **Anh Pham** — *Friendly AI Assistant with Amazon Q*
3. **Thinh Nguyen** — *From Edge To Origin: CloudFront as Your Foundation*
4. **Team VIB** — *36 hrs with LotusHacks – Building UTMorpho from Idea to Reality* *(Video Demo: [YouTube - UTMorpho Demo](https://www.youtube.com/watch?v=XjMCrcDRACQ))*
5. **Duc Dao** — *Non-Determinism of "Deterministic" LLM Settings*
6. **Vy Lam** — *Enterprise-Grade Multi-Agent System: The Case of Startup Credit Scoring*

---

### Nội dung nổi bật

#### 1. Context Is Everything: Making AI Actually Work for You (Diễn giả: Tinh Truong)
* Bài chia sẻ làm rõ lý do tại sao các mô hình AI thường thất bại nếu thiếu "Context" (ngữ cảnh) và định nghĩa bản chất thực sự của ngữ cảnh trong ứng dụng thực tế.
* Giới thiệu sự dịch chuyển từ kỹ thuật viết prompt (Prompt Engineering) sang quản lý bộ nhớ dài hạn — khái niệm **Second AI Brain**.
* Định hướng tư duy thực tế và các lời khuyên giá trị giúp sinh viên bắt đầu xây dựng sản phẩm AI hiệu quả ngay từ khi còn ở trường đại học.

#### 2. Friendly AI Assistant with Amazon Q (Diễn giả: Anh Pham)
* Giới thiệu hệ sinh thái trợ lý thông minh Amazon Q:
  * **Quick Chat Agent:** Trợ lý hỗ trợ khám phá và phân tích dữ liệu chuyên sâu.
  * **Quick Flows:** Khả năng tự động hóa quy trình làm việc thông minh bằng ngôn ngữ tự nhiên không cần lập trình.
  * **Quick Spaces:** Không gian làm việc cộng tác biến tri thức cá nhân thành tri thức tập thể.
  * **QuickSight:** Tạo dashboard và báo cáo trực quan hóa dữ liệu từ nguồn thô bằng câu lệnh tự nhiên.

#### 3. From Edge To Origin: CloudFront as Your Foundation (Diễn giả: Thinh Nguyen)
* Phân tích vai trò nền tảng của Amazon CloudFront cho mọi loại ứng dụng (web, media, API).
* Phương pháp tối ưu hóa chi phí truyền dữ liệu, tăng cường tính năng bảo mật (kết hợp WAF & Shield), nâng cao tốc độ phản hồi và độ tin cậy của hệ thống tại tầng biên (Edge).

#### 4. 36 hrs with LotusHacks – Building UTMorpho from Idea to Reality (Diễn giả: Team VIB)
* *Xem video trình chiếu & demo sản phẩm tại:* **[YouTube Video - UTMorpho Demo](https://www.youtube.com/watch?v=XjMCrcDRACQ)**
* Chia sẻ hành trình 36 giờ vượt qua thử thách tại LotusHacks Hackathon của Team VIB: từ khâu động não (brainstorming), xác định bài toán thực tế, đến giai đoạn lập trình liên tục dưới áp lực thời gian.
* Giới thiệu tổng quan sản phẩm **UTMorpho**, demo tính năng thực tế, các thất bại và điểm ngoặt quan trọng trong quá trình hoàn thiện sản phẩm.

#### 5. Non-Determinism of "Deterministic" LLM Settings (Diễn giả: Duc Dao)
* Phân tích sâu về cơ chế chọn Token tiếp theo của các mô hình ngôn ngữ lớn (LLM).
* Phá vỡ lầm tưởng: Cấu hình `Temperature = 0` không hoàn toàn đảm bảo tính quyết định (determinism 100%) trong thực tế do ảnh hưởng từ các thuật toán tối ưu hóa suy luận (Inference Optimizations) ở tầng hạ tầng GPU/Hardware.
* Đề xuất các chiến lược giảm thiểu rủi ro (Mitigation Strategies) cho các ứng dụng yêu cầu độ chính xác tuyệt đối.

#### 6. Enterprise-Grade Multi-Agent System: The Case of Startup Credit Scoring (Diễn giả: Vy Lam)
* Phân tích sự lệch pha cấu trúc dữ liệu giữa hệ thống chấm điểm tín dụng của ngân hàng truyền thống và dữ liệu của các doanh nghiệp khởi nghiệp (Startup).
* So sánh mô hình Single Agent vs. Multi-Agent Paradigm.
* Giới thiệu bản thiết kế **Hội đồng thẩm định tín dụng ảo (Virtual Credit Committee)** áp dụng kiến trúc Multi-Agent, tích hợp rào chắn an toàn (Guardrails & Compliance) và lộ trình tính toán ROI vận hành cho doanh nghiệp.

---

### Bài học rút ra

* **Hiểu sâu về AI & LLM:** Thấy được tầm quan trọng cốt lõi của "Context Management" và các thách thức ẩn về tính không quyết định (non-determinism) khi đưa LLM vào sản phẩm thực tế.
* **Tư duy thiết kế hạ tầng:** Thấy rõ sức mạnh của tầng biên (Amazon CloudFront) trong việc vừa tối ưu chi phí vừa gia tăng bảo mật cho ứng dụng.
* **Tinh thần Hackathon & Lập trình sản phẩm:** Bài học truyền cảm hứng từ Team VIB về tinh thần làm việc nhóm, khả năng chịu áp lực và biến ý tưởng thành sản phẩm demo chạy được chỉ trong 36 giờ.

---

### Áp dụng vào công việc & dự án thực tập

* Áp dụng kiến thức về **Context Management** và **Amazon Q** vào việc cải thiện quy trình tra cứu tài liệu và phát triển các tính năng hỗ trợ tự động.
* Vận dụng tư duy kiến trúc tầng biên (CloudFront & Edge Security) để tối ưu hóa hiệu năng truyền dữ liệu cho dự án Workshop Sân bay (AIMS).
* Học hỏi mô hình **Multi-Agent System** để mở rộng tư duy thiết kế các luồng xử lý tự động phức tạp trong tương lai.

---

### Trải nghiệm sự kiện

Sự kiện FCAJ Community Day được tổ chức vô cùng chu đáo và chuyên nghiệp tại văn phòng Bitexco. Không khí trao đổi cởi mở giữa các diễn giả, cộng đồng kỹ sư và thực tập sinh đã mang lại cho tôi rất nhiều năng lượng tích cực và tri thức thực tiễn giá trị.
