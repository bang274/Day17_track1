# Day 17 Submission

**Student:** Trần Khánh Bằng
**Date:** 2026-04-24
**Product idea:** FreeTax AI — giúp freelancer tự động khai thuế từ invoice, không cần thuê kế toán.

---

## 1. MVP Boundary Sheet

**Riskiest Assumption:**
> Freelancer sẵn sàng upload sao kê ngân hàng và invoice lên một ứng dụng AI mà họ chưa dùng bao giờ.

**In-Scope** (tối đa 3):
- [ ] **Nhận diện invoice và điền form khai thuế tự động** — test giả định: AI có thể đọc invoice Upwork và xuất đúng số liệu để khai thuế.
- [ ] **Hiển thị Trust Signal** — test giả định: Nếu nói rõ "dữ liệu không lưu lên server", user sẽ sẵn lòng thử.

**Out-of-Scope:**
- **Tích hợp trực tiếp với cổng iCanhan** — chưa cần, user tự nộp file XML.
- **Hỗ trợ tất cả loại thuế** — chỉ tập trung vào thuế TNCN cá nhân trước.
- **App mobile** — làm web trước cho đơn giản.

**Non-Goals:**
- Không làm thay phần ký số và nộp hộ người dùng.
- Không tư vấn pháp lý chuyên sâu — chỉ hỗ trợ điền form đúng.

---

## 2. PRD Skeleton

### Problem Statement
> Freelancer ở Việt Nam tốn nhiều giờ mỗi tháng để tự tính toán và khai thuế từ nhiều nguồn thu nhập khác nhau, rất dễ sai và dễ bị phạt vì không hiểu luật thuế.

### Target User
> Designer freelancer ở Việt Nam, nhận tiền từ Upwork hoặc Fiverr, chuyển về ngân hàng Việt, mỗi tháng phải tự khai thuế nhưng không rành kế toán và sợ làm sai.

### User Stories

**Story 1:**
> As a **freelancer có thu nhập từ Upwork**, I want to **upload invoice PDF và nhận lại file khai thuế đã điền sẵn**, so that **tôi không cần tự tính toán mà vẫn nộp đúng thuế**.

**Story 2:**
> As a **freelancer bận rộn**, I want to **biết mình có đang vi phạm nghĩa vụ thuế không**, so that **tôi không bị phạt bất ngờ vào cuối năm**.

### AI-Specific

**Model Selection:**
- Model: Gemini 1.5 Flash
- Lý do chọn: Đọc được file PDF, nhanh, rẻ, đủ dùng cho bài toán này.
- Trade-offs chấp nhận: Không mạnh bằng GPT-4o về reasoning nhưng đủ để extract dữ liệu.
- Trade-offs không chấp nhận: Không dùng model lưu dữ liệu user để train lại.

**Data Requirements:**
- Nguồn: Invoice từ Upwork/Fiverr, sao kê ngân hàng, các mẫu biểu khai thuế của Tổng cục Thuế.

**Fallback UX:**
- Chiến lược: Human-in-the-loop
- Trigger: Khi AI không tự tin về 1 trường dữ liệu nào đó.
- Hành động: Highlight màu vàng, yêu cầu user xác nhận thủ công.
- User options: User có thể sửa trực tiếp trước khi xuất file.

### Success Metrics
- Primary metric: Tỷ lệ user tạo được file XML hợp lệ mà không cần hỗ trợ (Zero-touch completion rate).
- Ngưỡng thành công: >= 70% các trường quan trọng điền đúng ngay lần đầu.
- Timeframe đo lường: 4–6 tuần sau khi ra mắt beta.

### Dependencies & Constraints
- Cần Gemini Flash API và thư viện export XML theo chuẩn HTKK.
- Timeline: trước kỳ quyết toán thuế gần nhất.
- Budget: giữ chi phí AI < 2,000 VNĐ mỗi lần xử lý.

---

## 3. Hypothesis Table

### Hypothesis 1 (cho tính năng Invoice Parsing)
> "Chúng tôi tin rằng **AI đọc invoice và tự điền form khai thuế**
> sẽ giúp **freelancer nhận tiền từ Upwork/Fiverr**
> đạt được **khai thuế đúng mà không cần kế toán**.
> Chúng tôi sẽ biết mình đúng khi thấy **zero-touch completion rate** đạt **>= 70%**
> trong vòng **4–6 tuần sau beta**."

Riskiest assumption: AI đọc đúng số liệu từ invoice dù có nhiều loại phí trừ lẫn nhau.
Cách test cheapest: Lấy 10 invoice Upwork thật, chạy thử và so sánh kết quả với tính tay.

### Hypothesis 2 (cho tính năng Trust Signal)
> "Chúng tôi tin rằng **nói rõ 'dữ liệu xử lý ngay trên trình duyệt, không lưu server'**
> sẽ giúp **freelancer lần đầu dùng app**
> đạt được **sự tự tin để upload file tài chính cá nhân**.
> Chúng tôi sẽ biết mình đúng khi thấy **tỷ lệ upload thành công lần đầu** đạt **>= 30%**
> trong vòng **2 tuần chạy landing page**."

Riskiest assumption: User tin vào tuyên bố "local processing" mà không cần kiểm chứng kỹ thuật.
Cách test cheapest: Landing page đơn giản với badge "Xử lý cục bộ — không lưu server", đo tỷ lệ bấm upload.

---

## 4. PMF Scorecard

**Aha Moment:**
> Lần đầu tiên user upload invoice và nhìn thấy bảng tóm tắt "Thu nhập thực nhận / Thuế phải nộp / File XML: Sẵn sàng" hiện ra trong vòng 60 giây — thay thế cả buổi chiều ngồi tính tay.

**Actionable Metric:**
> `Aha Moment Rate` = % user tải file XML ngay trong phiên đầu tiên, trong vòng 3 phút sau khi upload invoice.

**PMF Method:**
> Aha Moment tracking — ngưỡng: 60% user đạt trong 7 ngày đầu.
> Retention Curve — D30 > 30%.

**Vanity Metrics tôi sẽ không dùng:**
- Số lượt đăng ký tài khoản.
- Số lượt tải app.

---

## 5. AI Critique Log

*(Sẽ điền sau khi nhận feedback từ AI review)*

**Điểm AI chỉ ra:**
1. [...] — Action: Accept / Reject / Partial — Lý do: [...]
2. [...] — Action: Accept / Reject / Partial — Lý do: [...]
3. [...] — Action: Accept / Reject / Partial — Lý do: [...]

**Thay đổi lớn nhất giữa Version A và Version B:**
> [Sẽ cập nhật sau khi có Version B]

---

## 6. Self-assessment

Mắt xích nào trong [MVP Boundary | PRD | Hypothesis | PMF] bạn đang yếu nhất?
> **Hypothesis** — con số 70% chưa có cơ sở thực tế, cần chạy thử trước khi launch để xác nhận có khả thi không.

Open questions bạn muốn giải đáp tiếp:
1. AI có đọc đúng số "thực nhận" sau khi trừ phí Upwork, phí Payoneer và chênh lệch tỷ giá không?
2. User có thật sự tin tưởng một startup mới để upload sao kê ngân hàng không?
