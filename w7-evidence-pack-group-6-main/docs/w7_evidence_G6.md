# Evidence Pack — W7: Capstone Hackathon — Ship Production-Ready AI in 48 Hours

## Group 6 — HexaCode

---

## Section 1 — Cover

| Field                | Details                                                                                             |
| -------------------- | --------------------------------------------------------------------------------------------------- |
| **Group Number**     | Group 6                                                                                             |
| **Member Names**     | Minh Tuấn · Thành Vinh · Anh Hoàng · Hoàng Nhân · Mạnh Khang · Ngọc Thắng · Đình Thông · Thành Tâm  |
| **Link Repo**        | [GitHub repo URL](https://github.com/NguyenAnhHoangIT/W7-capstone-terraform.git)                          |
| **Live Demo URL**    | [CloudFront HTTPS URL](https://hexacode.codes)                                                      |
| **W6 Evidence Pack** | [W6_evidence](https://github.com/H1eu232/w7-evidence-pack-group-6/blob/main/docs/w7_evidence_G6.md) |

---

## Section 2 — Domain & Product Vision

### 2.1 Use Case & Target Users
StudyBot là một nền tảng SaaS EduTech sử dụng công nghệ RAG, được thiết kế cho sinh viên đại học, người tự học và các ứng viên chuẩn bị cho các kỳ thi chứng chỉ. Người dùng tải tài liệu (PDF, TXT, MD), hệ thống sẽ trích xuất, phân mảnh (chunk) và lưu trữ vector. Người dùng có thể chat hỏi đáp, tạo các bài quiz trắc nghiệm tương tác và học qua thẻ Flashcard 3D xoay.

### 2.2 Target Market & Parallels
- **Đối tượng:** Sinh viên đại học quản lý khối lượng lớn bài giảng, người học chứng chỉ chuyên môn cần phản hồi ôn tập nhanh.
- **Sản phẩm tương tự:** Google NotebookLM (hỏi đáp tài liệu), Quizlet AI (flashcard và quiz), Khanmigo (gia sư AI tương tác).

### 2.3 User Stories

| Vai trò | Câu chuyện người dùng (Hành động & Mục tiêu) | Chi tiết triển khai |
| :--- | :--- | :--- |
| **Học viên** | **Tải tài liệu:** Tải lên bài giảng (PDF/TXT) để hệ thống tự động chỉ mục. | S3 lưu trữ file, Lambda xử lý trích xuất văn bản và nạp vào Vector Store. |
| **Học viên** | **Dashboard:** Xem số liệu thống kê (tài liệu, câu hỏi, quiz) để theo dõi tiến độ. | Frontend lưu cache và hiển thị số liệu qua `localStorage`. |
| **Học viên** | **Chat Q&A:** Đặt câu hỏi và nhận câu trả lời có dẫn nguồn cụ thể. | Lambda gọi Bedrock Converse API, truy xuất chunk tương ứng và trả về Frontend. |
| **Học viên** | **Quiz trắc nghiệm:** Làm bài trắc nghiệm trích xuất từ tài liệu để kiểm tra kiến thức. | Frontend gửi yêu cầu cấu hình, nhận dữ liệu JSON của quiz từ Bedrock để hiển thị. |
| **Học viên** | **Flashcards:** Học từ vựng thông qua thẻ xoay 3D và tự đánh giá độ nhớ. | Frontend tạo thẻ học khái niệm bằng AI hoặc dữ liệu mock ngoại tuyến. |

---

## Section 3 — System Architecture & VPC Isolation

### 3.1 Sơ đồ kiến trúc hạ tầng

![Architecture Diagram](./images/Architecture-Diagram.png)

---

### 3.2 Network Isolation — Cấu hình Security Group (Mandatory #6)

![SG-inbound](./images/SG-VPC-inbound.png)

![SG-outbound](./images/SG-VPC-outbound.png)

![Lambda-outbound](./images/sg-lambda-outbound.png)

<sub>Note: Ảnh chụp cấu hình Inbound Rules của Security Group bảo vệ Database hoặc Private resources. Quy tắc chỉ cho phép truy cập từ Security Group của Compute Layer (Lambda/EC2 SG), không mở cổng ra internet (0.0.0.0/0).</sub>

---

### 3.3 VPC Endpoints — Kết nối nội bộ không qua NAT Gateway

![Bedrock Endpoints](./images/Bedrock-Endpoint.png)

<sub>Note: Ảnh chụp danh sách VPC Endpoints trong tài khoản (Gateway Endpoint cho S3/DynamoDB và Interface Endpoint cho Bedrock), giúp hệ thống hoạt động hoàn toàn trong mạng nội bộ VPC an toàn và tránh chi phí duy trì NAT Gateway (~1.5 USD/ngày).</sub>

![Bedrock Agent Endpoints](./images/Bedrock-Agent-Endpoint.png)

![Secret Manager](./images/Secret-Manager-Endpoint.png)

![S3 Endpoints](./images/S3-Endpoint.png)
---

### 3.4 IAM Least-Privilege Execution Role (Mandatory #7)

![Lambda Chatbot Policy](./images/Chatbot-policy.png)

![Lambda Extractor Policy](./images/Extractor-policy.png)

![Lambda Quiz Policy](./images/Quiz-policy.png)

<sub>Note: Cấu hình chính sách IAM (Policy) đính kèm vào Execution Role của Lambda compute. Chỉ cấp quyền đọc/ghi tối thiểu trên đúng ARN của S3 bucket, DynamoDB Table và Bedrock Model cụ thể, không sử dụng quyền Administrator hoặc tài nguyên đại diện `*`.</sub>

---

## Section 4 — MH-COST-V — Cost Visibility & Attribution

### 4.1 Tagging Schema áp dụng cho Capstone

![Tagging schema](./images/Tagging-schema.jpg)

<sub>Note: Ảnh chụp tab Properties/Tags của S3 Bucket chứa tài liệu bài học, hiển thị đầy đủ bộ 4 Tag key bắt buộc: Project, Team, Owner, và Environment.</sub>

---

### 4.2 Gắn nhãn tài nguyên thực tế (Tagging Verification)

![Tags on S3 Bucket](./images/Tagging-verification.jpg)

<sub>Note: Ảnh chụp tab Properties/Tags của S3 Bucket chứa tài liệu bài học, hiển thị đầy đủ bộ 4 Tag key bắt buộc: Project, Team, Owner, và Environment.</sub>

---

### 4.3 AWS Budget Alerts & Cost Anomaly Detection Setup

**Cấu hình ngân sách AWS Budgets:**

![AWS Budgets Config](./images/Budget-Alerts.jpg)

<sub>Note: Ngân sách Budget được thiết lập ở hạn mức $100 USD (cảnh báo ở mức 80% tức $80 USD) gửi cảnh báo về email nhóm thông qua cổng SNS. Trạng thái email đăng ký đã được xác nhận (Confirmed).</sub>

**Cấu hình Cost Anomaly Detection:**

![Cost Anomaly Detection Config](./images/AnomalyDetection.png)

<sub>Note: Monitor cảnh báo chi tiêu bất thường dựa trên thuật toán học máy (ML-based) đã được khởi tạo thành công để phát hiện sớm các sự cố chi phí.</sub>

---

### 4.4 Baseline Cost Breakdown (Theo dõi chi phí Cost Explorer)

**Cost Explorer Ngày 1 EOD:**

![Cost Explorer Day 1](./images/Cost-day-1.jpg)

<sub>Note: Ảnh chụp Cost Explorer lọc theo nhãn `Team=G6` vào cuối ngày thứ nhất sau khi khởi tạo toàn bộ tài nguyên nền tảng.</sub>

**Cost Explorer Ngày 2 EOD:**

![Cost Explorer Day 2](./images/Cost-day-2.jpg)

<sub>Note: Ảnh chụp Cost Explorer vào cuối ngày thứ hai sau khi tích hợp toàn bộ các tính năng AI Bedrock và thiết lập giám sát.</sub>

**Cost Explorer Sáng Thứ Sáu (Pre-Demo):**

![Cost Explorer Friday](./images/CostFriday.png)

<sub>Note: Chi phí cập nhật mới nhất vào sáng Thứ Sáu trước giờ thuyết trình dự án Capstone, cam kết nằm dưới hạn mức $100 USD.</sub>

---

### 4.5 Quan sát Top 3 Cost Driver:
1. **Amazon Bedrock (Claude 3.5 Haiku + Amazon Nova Pro + Cohere Embed Multilingual)**: Chi phí xử lý token chat, quiz generation và sync dữ liệu RAG.
2. **VPC Endpoints (Interface Endpoints)**: Chi phí duy trì kết nối private bảo mật sang dịch vụ Bedrock theo giờ.
3. **Amazon CloudFront**: Chi phí phân phối tài nguyên tĩnh và truyền dữ liệu mạng (Data Transfer Out).

---

## Section 5 — Security & Encryption Details

### 5.1 Root Account Multi-Factor Authentication (MFA)

![Root MFA Enabled](./images/MFA-root-acc.jpg)

<sub>Note: Ảnh chụp IAM Dashboard của tài khoản AWS cá nhân, xác nhận tài khoản Root đã được cấu hình xác thực đa yếu tố (MFA) thành công bằng ứng dụng bảo mật.</sub>

---

### 5.2 Amazon Bedrock Model Access

![Bedrock Model Access](./images/Model-Access.png)

<sub>Note: Ảnh chụp trang quản lý quyền truy cập mô hình trên Amazon Bedrock, chứng minh quyền sử dụng các dòng mô hình Claude 3.5 Haiku (Chat), Amazon Nova Pro (Quiz) và Cohere Embed Multilingual ở trạng thái "Access granted".</sub>

---

### 5.3 S3 Block Public Access & Default Encryption

![S3 Security Setup](./images/Bucket-Encryption.png)

<sub>Note: Ảnh chụp phần cấu hình bảo mật của S3 Bucket, hiển thị chế độ "Block all public access" đang được kích hoạt (ON) và mã hóa mặc định Server-side Encryption (SSE-S3 hoặc KMS CMK) được áp dụng.</sub>

![S3 Security Setup](./images/Block-public-access.png)

<sub>Note: Ảnh chụp phần cấu hình bảo mật của S3 Bucket, hiển thị chế độ "Block all public access" đang được kích hoạt (ON) và mã hóa mặc định Server-side Encryption (SSE-S3 hoặc KMS CMK) được áp dụng.</sub>

---

## Section 6 — Optional Capabilities (Nhóm chọn Option A)

### OPTION A — Full Observability (#8)

**CloudWatch Dashboard giám sát hệ thống:**

![CloudWatch Dashboard](./images/CloudWatch-Dashboard.png)

<sub>Note: Giao diện CloudWatch Dashboard "StudyBot-Dashboard" gồm 3 widgets: Invocations của 3 Lambda functions (W7Capstone-extractor, W7Capstone-chatbot, W7Capstone-quiz), biểu đồ lỗi 4xx/5xx của API Gateway, và chỉ số InputTokenCount/OutputTokenCount theo dõi lượng token sử dụng trên Bedrock.</sub>

**Cấu hình CloudWatch Alarm:**

![CloudWatch Alarm Config](./images/Cloudwatch-Alarm.png)

<sub>Note: Cảnh báo CloudWatch Alarm theo dõi tỷ lệ lỗi hoặc latency vượt ngưỡng, đang hoạt động ở trạng thái bình thường (OK) hoặc cảnh báo (ALARM).</sub>

**Lưu câu truy vấn Log Insights:**

![Saved Log Insights Query](./images/Log-Insight.png)

<sub>Note: Câu truy vấn CloudWatch Log Insights lọc log lỗi hệ thống trong 1 giờ qua đã được lưu thành công trên bảng điều khiển để truy cập nhanh khi vận hành.</sub>

---

## Section 7 — Bonus Paths

### 7.1 Custom Domain & HTTPS via Route 53 + ACM (Bonus Path A)

![ACM Config](./images/ACM.png)

<sub>Note: Ảnh chụp chứng chỉ SSL trạng thái "Issued" cấp bởi ACM cho tên miền tùy chỉnh của nhóm, được ánh xạ qua Route 53 trỏ tới CloudFront Distribution.</sub>

![Cloudfront Config](./images/Cloudfront-Distribution.png)

<sub>Note: Ảnh chụp chứng chỉ SSL trạng thái "Issued" cấp bởi ACM cho tên miền tùy chỉnh của nhóm, được ánh xạ qua Route 53 trỏ tới CloudFront Distribution.</sub>

![Route 53 Config](./images/Route53.png)

<sub>Note: Ảnh chụp chứng chỉ SSL trạng thái "Issued" cấp bởi ACM cho tên miền tùy chỉnh của nhóm, được ánh xạ qua Route 53 trỏ tới CloudFront Distribution.</sub>

### 7.3 Infrastructure as Code (IaC) Stack (Bonus Path C)

![Terraform Apply Success](./images/Terraform-Success.png)

<sub>Note: Ảnh chụp kết quả chạy lệnh Terraform apply thành công, xác nhận toàn bộ tài nguyên hạ tầng được triển khai tự động qua Infrastructure as Code.</sub>

### 7.4 Amazon Bedrock Guardrails Setup (Bonus Path D)

![Bedrock Guardrail Settings](./images/Guardrail.png)

<sub>Note: Cấu hình bộ lọc Guardrails trên Amazon Bedrock chặn lọc thông tin nhạy cảm PII và kiểm soát chất lượng từ ngữ đầu ra của mô hình.</sub>

---

## Section 8 — Customization & Originality (Quiz & Flashcards)

### 8.1 Trắc nghiệm tương tác Quiz Engine

![Interactive Quiz Demo](./images/Quiz-Engine.png)

<sub>Note: Giao diện Quiz Battle gamified hiển thị câu hỏi trắc nghiệm (4/10) được trích xuất từ tài liệu, với thanh Boss HP và Player HP mô phỏng trận đấu, người chơi chọn đáp án đúng để gây sát thương lên Boss.</sub>

![Interactive Quiz Demo](./images/Wrong-Ans.png)

<sub>Note: Minh họa trường hợp người chơi chọn sai đáp án (đáp án sai được highlight đỏ), hệ thống trừ -10 HP của Player và hiển thị hiệu ứng phản hồi trực quan ngay lập tức.</sub>

![Interactive Quiz Demo](./images/Correct-Ans.png)

<sub>Note: Minh họa trường hợp trả lời đúng, hệ thống gây -50 HP "Critical Attack" lên Boss kèm hiệu ứng hoạt hình, chứng minh cơ chế gamification tạo động lực học tập cho người dùng.</sub>

### 8.2 Thẻ ghi nhớ xoay Flashcards 3D

![Flashcard Flip Demo](./images/3D-FlashCard.png)

<sub>Note: Giao diện Flashcard mặt trước hiển thị câu hỏi "Tên dự án được đề xuất trong tài liệu là gì?" được AI trích xuất từ tài liệu đã tải lên, bên trái là Study Panel hiển thị nội dung tài liệu gốc để đối chiếu.</sub>

![Flashcard Flip Demo](./images/3D-FlashCard2.png)

<sub>Note: Flashcard mặt sau hiển thị đáp án kèm mức độ khó (EASY), hỗ trợ điều hướng giữa các thẻ (1/10) và nút chuyển thẻ trước/sau để người dùng tự đánh giá mức độ ghi nhớ.</sub>

![Flashcard Flip Demo](./images/3D-FlashCard3.png)

<sub>Note: Thẻ Flashcard 3D sau khi xoay hoàn tất (thẻ 2/10), minh họa hiệu ứng lật thẻ mượt mà và bố cục hiển thị đáp án rõ ràng trên nền trắng tương phản với giao diện tối.</sub>

---

## Section 9 — Measurement & Decisions (Mục §6.5 chống đối phó)

### Decision Block 1: Lựa chọn mô hình ngôn ngữ lớn (LLM) cho Chat RAG & Quiz Generation

![Decision Block 1](./images/Block-Decision-1.jpg)

```
DECISION: 
Sử dụng Claude 3.5 Haiku cho Chat RAG và Amazon Nova Pro cho Quiz Generation trên Bedrock, thay vì dùng một mô hình duy nhất cho cả hai tác vụ.

ALTERNATIVES CONSIDERED:
- Claude 3.5 Sonnet (cho cả hai): Loại bỏ vì chi phí token quá cao ($3.00/$15.00 per 1M tokens input/output) dẫn đến nguy cơ vượt ngân sách khi gọi test liên tục.
- Amazon Titan Text Express: Loại bỏ vì độ chính xác trong việc trích xuất thông tin tiếng Việt và lập luận câu hỏi trắc nghiệm dưới dạng cấu trúc JSON không ổn định.
- Claude 3.5 Haiku cho Quiz: Loại bỏ vì Nova Pro cho output JSON cấu trúc quiz ổn định hơn với chi phí thấp hơn.

MEASUREMENT:
- Chi phí Chat: Claude 3.5 Haiku tốn ~$0.25/$1.25 per 1M tokens, giảm thiểu 90% chi phí so với Sonnet.
- Chi phí Quiz: Nova Pro tốn ~$0.80/$3.20 per 1M tokens, cân bằng giữa chất lượng JSON output và chi phí.
- Độ trễ phản hồi trung bình: Haiku chat ~1.5 giây, Nova Pro quiz ~2.0 giây.

TRADE-OFF ACCEPTED:
- Chấp nhận quản lý hai mô hình riêng biệt để tối ưu chi phí và chất lượng cho từng tác vụ. Haiku phù hợp cho hỏi đáp RAG, Nova Pro phù hợp cho sinh câu hỏi trắc nghiệm cấu trúc.
```

### Decision Block 2: Công nghệ lưu trữ dữ liệu Database cho cấu hình Session Log

![Decision Block 2](./images/Block-Decision-2.1.jpg)

![Decision Block 2](./images/Block-Decision-2.2.jpg)

![Decision Block 2](./images/Block-Decision-2.3.jpg)

```
DECISION:
Sử dụng Amazon DynamoDB (On-Demand capacity) thay vì Amazon RDS PostgreSQL.

ALTERNATIVES CONSIDERED:
- Amazon RDS PostgreSQL db.t3.micro: Loại bỏ vì chi phí duy trì cố định (~$1.25/48h) và mất thời gian kết nối qua RDS Proxy để tránh nghẽn khi Lambda mở rộng đồng thời.

MEASUREMENT:
- Chi phí vận hành: DynamoDB tốn $0.00 cho nhu cầu hackathon (nằm trong 25GB free tier).
- Tốc độ truy vấn: Thời gian đọc ghi session log bằng DynamoDB SDK luôn dưới 10ms.

TRADE-OFF ACCEPTED:
- Chấp nhận việc không thực hiện được các câu lệnh SQL JOIN phức tạp hoặc phân tích dữ liệu chuyên sâu. Đổi lại, hệ thống cực kỳ dễ mở rộng và miễn phí hoàn toàn.
```

### Decision Block 3: Chiến lược trích xuất tài liệu kết hợp (Hybrid Extraction) cho StudyBot

```
DECISION:
StudyBot sử dụng chiến lược trích xuất kết hợp cho tài liệu học tập được tải lên. Với các tài liệu có thể trích xuất văn bản trực tiếp như TXT, MD và PDF đơn giản, Extractor Lambda sử dụng thư viện JavaScript để parse text. Với các PDF có mật độ text thấp, scan, nhiều bảng biểu hoặc bố cục phức tạp, hệ thống chuyển sang Amazon Textract.

Quyết định này phù hợp với use case EduTech vì tài liệu bài giảng không phải lúc nào cũng là text thuần. Một số file chứa bảng biểu, ảnh chụp slide, trang scan hoặc nội dung dạng hình ảnh. Phương pháp kết hợp giúp hệ thống tiết kiệm chi phí mà vẫn xử lý được tài liệu giáo dục phức tạp.

ALTERNATIVES CONSIDERED:
- Textract cho mọi file: Loại bỏ vì không phải file nào cũng cần OCR. TXT, MD và PDF có text sẵn có thể parse nhanh hơn và rẻ hơn bằng JS parser. Dùng Textract cho tất cả sẽ tăng chi phí và độ trễ cho các file đơn giản.
- Chỉ dùng JS parser: Loại bỏ vì parser đơn thuần có thể bỏ sót text trong các trang scan, bảng biểu hoặc bố cục slide phức tạp. Điều này làm giảm chất lượng RAG vì Knowledge Base nhận được nội dung không đầy đủ.
- Bedrock Vision cho mọi trang: Loại bỏ vì không cần thiết cho tài liệu text thông thường và sẽ tăng độ trễ/chi phí. Phù hợp hơn cho trang có nhiều hình ảnh/sơ đồ, không phải luồng xử lý mặc định.

TRADE-OFF ACCEPTED:
- Phương pháp kết hợp tăng độ phức tạp triển khai so với chỉ dùng một phương pháp trích xuất.
- Textract fallback chậm hơn parse trực tiếp, nhưng cải thiện độ tin cậy cho tài liệu bài giảng thực tế.
- Nhóm chấp nhận độ phức tạp này vì giá trị cốt lõi của StudyBot là trả lời từ nội dung tài liệu thực sự được tải lên, không chỉ từ file text thuần.
```

### Decision Block 4: Sử dụng S3 Vectors làm vector store cho Bedrock Knowledge Base

```
DECISION:
StudyBot sử dụng Amazon S3 Vectors làm vector store cho Bedrock Knowledge Base. Các chunk tài liệu được nhúng (embed) bằng Cohere Embed Multilingual và lưu trữ/truy xuất thông qua S3 Vectors cho hỏi đáp RAG.

Lựa chọn này phù hợp vì ứng dụng là SaaS hackathon ngắn hạn với lưu lượng demo thấp-trung bình, kiến trúc serverless và yêu cầu kiểm soát chi phí chặt chẽ. S3 Vectors phù hợp hơn so với việc vận hành một vector database riêng biệt trong thời gian demo.

ALTERNATIVES CONSIDERED:
- OpenSearch Serverless vector collection: Loại bỏ vì thêm hạ tầng always-on và chi phí. Mạnh mẽ cho tìm kiếm throughput cao nhưng quá mức cần thiết cho workload demo.
- Aurora PostgreSQL với pgvector: Loại bỏ vì phải thiết lập database quan hệ, quản lý kết nối và thêm overhead vận hành cho Lambda.
- Custom vector search trong DynamoDB/S3 JSON: Loại bỏ vì cần logic tìm kiếm tương đồng tùy chỉnh và không phải là luồng RAG managed sạch sẽ.

TRADE-OFF ACCEPTED:
- S3 Vectors tiết kiệm chi phí và serverless, nhưng không phải lựa chọn tốt nhất cho workload vector search độ trễ cực thấp, QPS cao.
- OpenSearch Serverless cung cấp khả năng kiểm soát search/index nâng cao hơn, nhưng thêm chi phí và gánh nặng vận hành.
- Với workload demo của StudyBot, kỷ luật chi phí và tích hợp Bedrock managed quan trọng hơn việc tinh chỉnh vector database nâng cao.
```

### Decision Block 5: Chiến lược chia mô hình cho Chatbot, Quiz, Flashcard và Embedding

```
DECISION:
StudyBot sử dụng các mô hình Bedrock khác nhau cho các tác vụ AI khác nhau thay vì dùng một mô hình cho tất cả:

- Claude 3.5 Haiku cho chatbot Q&A vì nhanh và tiết kiệm chi phí cho câu trả lời RAG hội thoại.
- Amazon Nova Pro cho quiz và flashcard generation vì các tác vụ này cần khả năng sinh cấu trúc mạnh hơn và định dạng nội dung giáo dục tốt hơn.
- Cohere Embed Multilingual cho embeddings vì tài liệu học tập và câu hỏi người dùng có thể bao gồm cả tiếng Việt và tiếng Anh.

Việc chia tách giúp chatbot phản hồi nhanh trong khi vẫn sử dụng mô hình mạnh hơn cho việc tạo nội dung học tập.

ALTERNATIVES CONSIDERED:
- Dùng Nova Pro cho tất cả tác vụ: Loại bỏ vì dùng mô hình mạnh cho mọi lượt chat sẽ tăng chi phí và không cần thiết cho Q&A đơn giản dựa trên ngữ cảnh.
- Dùng Claude 3.5 Haiku cho tất cả tác vụ: Loại bỏ vì quiz và flashcard generation cần output cấu trúc ổn định, bao gồm JSON trắc nghiệm, đáp án đúng, đáp án nhiễu và giải thích ngắn gọn.
- Dùng mô hình embedding chỉ tiếng Anh: Loại bỏ vì StudyBot nhắm đến nội dung học tập Việt/Anh, nên chất lượng truy xuất đa ngôn ngữ rất quan trọng.

MEASUREMENT:
- Chatbot model: Claude 3.5 Haiku
- Quiz/flashcard model: Amazon Nova Pro
- Embedding model: Cohere Embed Multilingual
- Độ trễ phản hồi chat trung bình: Haiku ~1.5 giây
- Độ trễ sinh quiz trung bình: Nova Pro ~2.0 giây
- Chi phí Chat: Haiku ~$0.25/$1.25 per 1M tokens
- Chi phí Quiz: Nova Pro ~$0.80/$3.20 per 1M tokens

TRADE-OFF ACCEPTED:
- Sử dụng nhiều mô hình tăng độ phức tạp cấu hình và thiết lập IAM/model-access.
- Backend phải xử lý các định dạng response khác nhau cho chatbot, quiz JSON và flashcard JSON.
- Nhóm chấp nhận điều này vì mỗi mô hình được dùng đúng chỗ: Haiku cho chat nhanh, Nova Pro cho sinh nội dung giáo dục phong phú, và Cohere Embed Multilingual cho truy xuất đa ngôn ngữ.
```

---

## Section 10 — Project Demonstration

### Giao diện chính của hệ thống hoạt động thực tế

![Application Interface](./images/Live-App-UI.png)

<sub>Note: Màn hình Dashboard tổng quan của StudyBot hiển thị các thông số hoạt động học tập thực tế và nhật ký.</sub>

![Application Interface Q&A](./images/RAG-Chat.png)

<sub>Note: Khung chat hỏi đáp AI RAG hiển thị bong bóng chat sinh động kèm liên kết dẫn nguồn chi tiết.</sub>

![Document Upload Ingestion](./images/Document-Upload.png)

<sub>Note: Khung chat hỏi đáp AI RAG hiển thị bong bóng chat sinh động kèm liên kết dẫn nguồn chi tiết.</sub>

---

## Section 11 — Teardown Plan & Checklist

### 11.1 Trình tự xóa tài nguyên (Teardown Order)
Để đảm bảo không bị lỗi phụ thuộc (dependency errors) khi xóa tài nguyên trên AWS, nhóm áp dụng trình tự xóa ngược lại với lúc triển khai:

# AWS Teardown Plan - W7Capstone Project

---

## THỨ TỰ ƯU TIÊN (Xóa ngay để dừng tính phí)
1.  **Amazon Bedrock Knowledge Base & OpenSearch Serverless** (Chi phí duy trì cao ~ $0.24/giờ).
2.  **AWS WAF** (Phí duy trì Web ACL hàng tháng).
3.  **NAT Gateway** (Nếu có cấu hình trong VPC - phí tính theo giờ rất cao).

---

## 🛠️ CHI TIẾT CÁC BƯỚC THỰC HIỆN

### Bước 1: Amazon Bedrock & Vector Store
*   **Knowledge Base (KB):** Vào **Bedrock Console** ➡️ **Knowledge bases** ➡️ Chọn KB của dự án ➡️ Bấm **Delete**.
*   **Vector Store (OpenSearch Serverless):** Vào **Amazon OpenSearch Service** ➡️ **Serverless** ➡️ **Collections** ➡️ Xóa Collection liên kết với KB.
*   **Guardrail:** Vào **Bedrock** ➡️ **Guardrails** ➡️ Chọn guardrail dự án và bấm **Delete**.

### Bước 2: CloudFront, WAF & Networking Ngoài
1.  **CloudFront Distribution:**
    *   Vào **CloudFront Console** ➡️ Chọn Distribution ➡️ Bấm **Disable**.
    *   Sau khi trạng thái là `Disabled`, chọn lại và bấm **Delete**.
2.  **AWS WAF:** Vào **WAF & Shield** ➡️ **Web ACLs** ➡️ Xóa Web ACL liên kết với CloudFront.
3.  **Route 53:** Xóa các bản ghi `A` hoặc `CNAME` trỏ về CloudFront (Không xóa NS và SOA).
4.  **ACM:** Vào **Certificate Manager** ➡️ Xóa SSL Certificate đã dùng.

### Bước 3: API Gateway & Cognito
*   **API Gateway:** Vào **API Gateway Console** ➡️ Chọn API dự án ➡️ Bấm **Delete**.
*   **Cognito User Pool:** 
    *   Vào **Cognito Console** ➡️ **User Pools**.
    *   Xóa **App Clients** trong tab App Integration trước.
    *   Sau đó quay lại trang chính của Pool và bấm **Delete**.

### Bước 4: Lambda Functions & IAM Roles

1.  **Lambda Functions:** Xóa lần lượt 3 functions:
    *   `W7Capstone-chatbot`
    *   `W7Capstone-quiz`
    *   `W7Capstone-extractor`
2.  **IAM Roles:** Vào **IAM Console** ➡️ **Roles** ➡️ Xóa các Role đã tạo cho dự án này.

### Bước 5: Làm rỗng và Xóa 4 S3 Buckets
*Thực hiện quy trình: Chọn Bucket ➡️ Bấm **Empty** ➡️ Nhập xác nhận ➡️ Quay lại bấm **Delete**.*
1.  `w7capstone-fe-host-*` (Frontend)
2.  `w7capstone-vector-store-*` (Vector)
3.  `w7capstone-raw-uploads-*` (Raw store)
4.  `w7capstone-clean-store-*`

### Bước 6: Database & Secrets
*   **DynamoDB Tables:** Xóa 4 bảng: `chat-history`, `quiz-rooms`, `quiz-questions`, `document-context`.
*   **Secrets Manager:** Chọn secret dự án ➡️ **Delete secret** ➡️ Chọn **Force deletion without recovery**.

### Bước 7: Giải phóng hạ tầng VPC 
1.  **VPC Endpoints:** Vào **VPC Console** ➡️ **Endpoints** ➡️ Xóa toàn bộ (S3, DynamoDB, Bedrock, Secrets Manager).
2.  **NAT Gateway:** (Nếu có) Xóa NAT Gateway và giải phóng Elastic IP.
3.  **Internet Gateway:** Chọn IGW ➡️ **Detach from VPC** ➡️ **Delete**.
4.  **VPC:** Vào **Your VPCs** ➡️ Chọn dự án VPC ➡️ **Actions** ➡️ **Delete VPC**.

---

*— End of W7 Evidence Pack —*
