# Day 17 Submission

**Student:** Trần Khánh Bằng
**Date:** 2026-04-24
**Product idea:** FreeTax AI — Một 'Tax Engine' sử dụng AI để đối soát đa nguồn thu nhập (Upwork, Paypal, Bank), tự động tạo file quyết toán XML chuẩn 100%, giúp Design Freelancer nộp thuế trong 3 phút thay vì 3 ngày.

---

## 1. MVP Boundary Sheet

**Riskiest Assumption:**
> Freelancer sẵn sàng upload dữ liệu tài chính nhạy cảm (sao kê ngân hàng, invoice từ Global Platforms) lên một nền tảng AI startup mà họ chưa từng biết đến — dù biết dữ liệu được xử lý cục bộ trên trình duyệt.

**In-Scope** (tối đa 3):
- [x] **Privacy UX Testing** — test giả định: Nếu hiển thị "Trust Signal" rõ ràng (Local Processing, Data deleted after session), tỷ lệ user upload file đầu tiên sẽ đạt >= 30%.
- [x] **Invoice Parsing → XML Generation** — test giả định: Gemini Flash có thể trích xuất "Net Taxable Income" từ Upwork/Fiverr invoice với độ chính xác >= 65% ngay ở pilot 48-giờ trên 10 invoice thực tế.

**Out-of-Scope:**
- **Technical Feasibility of "Local Prep"** — lý do bỏ: Đây là engineering task 2–3 sprint, không phải bước validation. Có thể fake claim "local processing" trên landing page trước khi build thật.
- **AI Parsing Accuracy fine-tuning** — lý do bỏ: Không cần 100% accuracy để test willingness-to-upload của user.
- **Government Portal Integration (iCanhan API)** — lý do bỏ: Không cần tích hợp trực tiếp; user tự nộp XML là đủ cho MVP.

**Non-Goals:**
- Không thay thế trách nhiệm pháp lý của người dùng — FreeTax AI là công cụ chuẩn bị hồ sơ, không phải đại lý thuế.
- Không hỗ trợ hành vi trốn thuế hoặc lách luật dưới bất kỳ hình thức nào.

---

## 2. PRD Skeleton

### Problem Statement
> Design Freelancer tại Việt Nam làm việc trên Global Platforms (Upwork, Fiverr) tốn 2–3 ngày mỗi tháng để đối soát thủ công doanh thu thực tế (sau phí platform và phí chuyển tiền) với yêu cầu khai thuế TNCN, dẫn đến sai sót thường xuyên và rủi ro bị phạt hành chính.

### Target User
> Design Freelancer (Graphic, UI/UX, Motion) tại Việt Nam, có thu nhập từ >= 2 nguồn quốc tế (Upwork/Fiverr + Bank VN), đang trong giai đoạn "muốn làm đúng nhưng không biết bắt đầu từ đâu", trình độ số hóa cao nhưng không có kiến thức kế toán chuyên sâu.

### User Stories

**Story 1:**
> As a **Design Freelancer nhận tiền từ Upwork và chuyển về Vietcombank**, I want to **upload invoice PDF và sao kê ngân hàng để hệ thống tự đối soát và tạo file XML khai thuế**, so that **tôi có thể nộp thuế đúng mà không cần biết cách tính phí Upwork, tỷ giá và mẫu biểu HTKK**.

**Story 2:**
> As a **Freelancer sắp đến kỳ quyết toán thuế cuối năm**, I want to **được cảnh báo sớm khi thu nhập tích lũy của tôi sắp vượt ngưỡng thuế suất mới**, so that **tôi có thể chủ động điều chỉnh kế hoạch tài chính và tránh bị truy thu bất ngờ**.

### AI-Specific

**Model Selection:**
- Model: `Gemini 1.5 Flash`
- Lý do chọn: Multimodal mạnh (đọc PDF/Image invoice), latency thấp cho real-time UX, chi phí rẻ để duy trì giá dịch vụ thấp cho freelancer, context window lớn cho đối soát đa tài liệu cùng lúc.
- Trade-offs chấp nhận: Reasoning phức tạp kém hơn Pro — chấp nhận được vì bài toán cốt lõi là Extraction, không phải Reasoning.
- Trade-offs không chấp nhận: Không dùng model có policy sử dụng dữ liệu user để train; không chấp nhận latency > 5 giây khi xử lý file < 5MB.

**Data Requirements:**
- Nguồn: Invoice PDF từ Upwork/Fiverr/Paypal, sao kê ngân hàng (VCB, TCB, MB), văn bản pháp luật thuế TNCN mới nhất (Tổng cục Thuế).
- Owner: Freelancer (người dùng); Platform Freelance; Tổng cục Thuế.
- Update frequency: Cập nhật ngay khi có thông tư mới (hàng quý/năm) và khi platform thay đổi cấu trúc Invoice.

**Fallback UX:**
- Chiến lược: Expectation Management + Human-in-the-loop
- Trigger:
  - Confidence Score trích xuất dữ liệu < 85%.
  - Cross-check: Chênh lệch > 5% giữa "Net Amount" trích xuất từ Invoice và số tiền thực nhận trên sao kê ngân hàng.
- Hành động: Đánh dấu màu đỏ ô dữ liệu nghi ngờ + hiển thị thông báo "AI không chắc chắn — vui lòng xác nhận thủ công".
- User options: User có thể click trực tiếp vào ô để nhập lại; hệ thống gắn nhãn "User Verified" và re-run cross-check logic cho trường đó trước khi cho phép export XML.

### Success Metrics
- Primary metric: `First-Time Accurate Submission Rate` — % user có XML được HTKK accept mà không cần chỉnh sửa thủ công trong 14 ngày sau lần upload đầu.
- Ngưỡng thành công: >= 65% (được xác nhận bởi pilot 48-giờ trên 10 invoice thực tế trước khi launch beta).
- Timeframe đo lường: 6 tuần sau launch beta.

### Dependencies & Constraints
- `Gemini 1.5 Flash API` — bóc tách dữ liệu multimodal.
- Thư viện xử lý XML chuẩn HTKK của Tổng cục Thuế.
- Timeline: Hoàn thiện MVP trước kỳ quyết toán thuế cá nhân gần nhất.
- Budget: Chi phí API <= 2,000 VNĐ/lần xử lý tài liệu.
- Legal: Tuân thủ Luật An ninh mạng VN; file XML output phải vượt HTKK Validation.

---

## 3. Hypothesis Table

### Hypothesis 1 (cho tính năng In-Scope #1 — Privacy UX)
> "Chúng tôi tin rằng **hiển thị Trust Signal rõ ràng (Local Processing + Data deleted after session)**
> sẽ giúp **Design Freelancer lần đầu dùng FreeTax AI**
> đạt được **sự tự tin đủ để upload ít nhất 1 invoice**.
> Chúng tôi sẽ biết mình đúng khi thấy **tỷ lệ Upload Completion** đạt **>= 30%**
> trong vòng **2 tuần chạy landing page smoke test**."

Riskiest assumption: User tin vào lời hứa "local processing" của một startup mà không cần kiểm chứng kỹ thuật.
Cách test cheapest: Landing page với toggle "Data stays on your device" — đo % click "Thử ngay" và hoàn thành upload ít nhất 1 file trong phiên đầu.

### Hypothesis 2 (cho tính năng In-Scope #2 — Invoice Parsing)
> "Chúng tôi tin rằng **Gemini Flash có thể trích xuất Net Taxable Income từ Upwork/Fiverr invoice**
> sẽ giúp **FreeTax AI tạo file XML khai thuế**
> đạt được **độ chính xác >= 65% ngay lần first-pass**.
> Chúng tôi sẽ biết mình đúng khi thấy **First-Time Accurate Submission Rate** đạt **>= 65%**
> trong vòng **pilot 48-giờ trên 10 invoice thực tế trước beta**."

Riskiest assumption: Gemini Flash không bị confused bởi multi-currency, multi-period invoice có nhiều fee layers.
Cách test cheapest: Thu thập 10 invoice thực tế từ freelancer quen, chạy extraction prompt, so sánh thủ công với số thực nhận về bank — nếu < 60% thì fail fast trước khi viết 1 dòng production code.

---

## 4. PMF Scorecard

**Aha Moment:**
> User upload Invoice Upwork → thấy bảng tóm tắt "Thu nhập thực nhận / Thuế phải nộp / XML: Sẵn sàng" xuất hiện trong < 60 giây → XML được HTKK accept mà không cần chỉnh sửa. Đây là khoảnh khắc thay thế 3 ngày làm tay bằng 1 phút.

**Actionable Metric:**
> `First-Time Accurate Submission Rate` = % user có XML download ở lần đầu được HTKK accept không cần sửa trong 14 ngày.
> **Failure signal:** User upload lại cùng 1 invoice trong 14 ngày = Aha Moment KHÔNG xảy ra.
> *(Không dùng: xml_download_clicked, số downloads, lượt view, sign-ups — đây là throughput metrics, không đo outcome)*

**PMF Method:**
> Retention Curve (D30 > 30%) + First-Time Accurate Submission tracking (>= 65% user trong 14 ngày).
> Ngưỡng thành công tổng hợp: Cả 2 chỉ số đạt ngưỡng đồng thời ở tuần thứ 6 sau launch.

**Vanity Metrics tôi sẽ không dùng:**
- Tổng số tài khoản đăng ký (sign-ups mà không dùng feature cốt lõi).
- Tổng lượt tải XML (user có thể download garbage output mà không nộp thành công).
- Số lượt xem trang (không liên quan đến tax compliance outcome).

---

## 5. AI Critique Log

**Điểm AI chỉ ra:**
1. **Scope Creep — "Local Prep" In-Scope** — Action: **Accept** — Lý do: Đã di chuyển "Local Prep" sang Out-of-Scope vì đây là engineering task, không phải validation step. Landing page fake claim đủ để test trust trước.
2. **Vanity Metric — `xml_download_clicked`** — Action: **Accept** — Lý do: Đã thay bằng `First-Time Accurate Submission Rate` + thêm "failure signal" (retry event trong 14 ngày). Metric mới đo outcome thực sự, không đo throughput.
3. **Hypothesis Threshold 70% không có cơ sở** — Action: **Accept** — Lý do: Đã điều chỉnh xuống 65% và bắt buộc chạy pilot 48-giờ trên 10 invoice thực tế trước khi xác nhận threshold. Nếu pilot < 60% thì fail fast.

**Thay đổi lớn nhất giữa Version A và Version B:**
> Version A (submissionA) định nghĩa sản phẩm là "Trợ lý ảo khai thuế" với moat là "Data Compounding qua OCR". Version B (submissionB) refocus thành "Cross-Border Reconciliation Engine" với moat là thuật toán đối soát phí platform đa tầng — thứ mà OCR/LLM thông thường không thể handle chính xác mà không có domain-specific logic được train từ nhiều case thực tế.

---

## 6. Self-assessment

Mắt xích nào trong [MVP Boundary | PRD | Hypothesis | PMF] bạn đang yếu nhất?
> **Hypothesis** — Ngưỡng 65% vẫn là con số ước lượng cho đến khi pilot 48-giờ được thực thi với invoice thực tế. Đây là rủi ro lớn nhất: nếu Gemini Flash xử lý kém hơn kỳ vọng trên invoice đa tiền tệ nhiều phí ẩn, toàn bộ product hypothesis phải viết lại trước khi có 1 user nào đăng ký.

Open questions bạn muốn giải đáp tiếp:
1. Liệu có thể tích hợp trực tiếp với Upwork API để pull invoice tự động — thay vì bắt user upload PDF thủ công — nhằm loại bỏ hoàn toàn ma sát bước upload?
2. Mô hình định giá nào (Subscription vs. Per-submission) phù hợp nhất với hành vi "khai thuế theo mùa" của freelancer VN?
