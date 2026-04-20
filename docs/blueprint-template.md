# Day 13 Observability Lab Report

> **Instruction**: Fill in all sections below. This report is designed to be parsed by an automated grading assistant. Ensure all tags (e.g., `[GROUP_NAME]`) are preserved.

## 1. Team Metadata
- **[GROUP_NAME]**: A1-C401
- **[REPO_URL]**: https://github.com/TBNRGarret/Lab13-A1-C401.git
- **[MEMBERS]**:
  - Member A: [Name] | Role: Logging & PII
  - Member B: [Đàm Lê Văn Toàn] | Role: Tracing & Enrichment
  - Member C: Vũ Lê Hoàng | Role: SLO & Alerts
  - Member D: [Nguyễn Quang Trường] | Role: Load Test + Incident Injection
  - Member E: [Name] | Role: Dashboard + Evidence
  - Member F: Phạm Tuấn Anh | Role: Blueprint + Demo lead

---

## 2. Group Performance (Auto-Verified)
- **[VALIDATE_LOGS_FINAL_SCORE]**: 80/100
- [TOTAL_TRACES_COUNT]: 
- [PII_LEAKS_FOUND]: 

---

## 3. Technical Evidence (Group)

### 3.1 Logging & Tracing
- [EVIDENCE_CORRELATION_ID_SCREENSHOT]: [Path to image]
- [EVIDENCE_PII_REDACTION_SCREENSHOT]: [Path to image]
- [EVIDENCE_TRACE_WATERFALL_SCREENSHOT]: [Path to image]
- [TRACE_WATERFALL_EXPLANATION]: (Briefly explain one interesting span in your trace)

### 3.2 Dashboard & SLOs
- [DASHBOARD_6_PANELS_SCREENSHOT]: [Path to image]
- [SLO_TABLE]:

| SLI | Target | Window | Current Value |
|---|---|---|---|
| Latency P95 | < 1000ms | 28d | 163.2ms |
| Error Rate | < 1% | 28d | 0% | 
| Cost Budget | < $5.00/day | 1d | $0.0021/req (~$2.10/day baseline) | 
| Quality Score Avg | > 0.70 | 28d | 0.8296 | 

### 3.3 Alerts & Runbook
- [ALERT_RULES_SCREENSHOT]: [Path to image]
- [SAMPLE_RUNBOOK_LINK]: [docs/alerts.md#L...]

---

## 4. Incident Response (Group)

### Scenario 1: core_banking_fail
- **[SCENARIO_NAME]**: core_banking_fail
- [SYMPTOMS_OBSERVED]: Khách hàng gặp lỗi 500 khi kiểm tra số dư, Error Rate trên Dashboard tăng vọt, Alert high_error_rate (P1) được kích hoạt.
- [ROOT_CAUSE_PROVED_BY]: Hàm `mock_core_banking_api()` gặp lỗi 'Connection Timeout to Core Banking' (truy vết qua Langfuse Trace Waterfall).
- [FIX_ACTION]: Tắt toggle giả lập lỗi (disable incident), ứng dụng phục hồi và Error Rate giảm dần.
- [PREVENTIVE_MEASURE]: Triển khai cơ chế Circuit Breaker hoặc Fallback API để đảm bảo tính sẵn sàng khi Core Banking gặp sự cố.

### Scenario 2: account_lookup_slow
- **[SCENARIO_NAME]**: account_lookup_slow
- [SYMPTOMS_OBSERVED]: Khách hàng nhận thấy chatbot phản hồi chậm khi tra cứu tài khoản, Latency P95 tăng từ 789.3ms lên 13290.4ms (+1585%), Dashboard hiển thị spike latency, Alert latency_high được kích hoạt.
- [ROOT_CAUSE_PROVED_BY]: Hàm `mock_rag.py` có `time.sleep(2.5)` khi incident được bật, simulate database query chậm do overload hoặc index không tối ưu (truy vết qua Langfuse Trace Waterfall).
- [FIX_ACTION]: Tắt toggle incident, latency phục hồi về 789.3ms, khách hàng trải nghiệm bình thường trở lại.
- [PREVENTIVE_MEASURE]: Thêm database indexing, implement caching layer (Redis), optimize query, scale database horizontally.

### Scenario 3: credit_check_fail
- **[SCENARIO_NAME]**: credit_check_fail
- [SYMPTOMS_OBSERVED]: Khách hàng gặp lỗi HTTP 500 khi yêu cầu kiểm tra tín dụng, Error Rate tăng 100% (tất cả credit_inquiry requests bị lỗi), Alert error_rate_high được kích hoạt, feature credit_inquiry hoàn toàn không khả dụng.
- [ROOT_CAUSE_PROVED_BY]: Hàm `agent.py` raise `RuntimeError: Credit check service unavailable` khi incident được bật, simulate credit bureau API down hoàn toàn (truy vết qua Langfuse Trace Waterfall).
- [FIX_ACTION]: Tắt toggle incident, credit_inquiry hoạt động bình thường, error rate giảm về 0%, tất cả requests thành công.
- [PREVENTIVE_MEASURE]: Implement fallback to cached credit scores, add retry logic với exponential backoff, setup monitoring cho credit bureau API availability, implement circuit breaker pattern.

### Scenario 4: high_token_usage
- **[SCENARIO_NAME]**: high_token_usage
- [SYMPTOMS_OBSERVED]: Chi phí LLM API tăng từ $0.0021/request lên $0.0023/request (+9.5%), tokens_out tăng gấp đôi (từ 13873 → 27296), Cost Budget Alert được kích hoạt, có nguy cơ vượt budget hàng ngày.
- [ROOT_CAUSE_PROVED_BY]: Hàm `mock_llm.py` có `output_tokens *= 2` khi incident được bật, simulate LLM tạo response dài gấp đôi do prompt không tối ưu (truy vết qua Langfuse Trace Waterfall).
- [FIX_ACTION]: Tắt toggle incident, cost phục hồi về $0.0021/request, tokens_out trở về bình thường, budget được kiểm soát.
- [PREVENTIVE_MEASURE]: Optimize prompt engineering, add max_tokens constraint, implement response caching, monitor token usage per feature, setup cost alerts.

### Scenario 5: rate_limiter_triggered
- **[SCENARIO_NAME]**: rate_limiter_triggered
- [SYMPTOMS_OBSERVED]: Khi load test với 20 concurrent users, Latency P95 tăng từ 3290.1ms lên 3631.8ms (+10.4%), một số requests bị queue/delay, response time không đều (Min: 498ms, Max: 3632ms), Alert latency_high được kích hoạt.
- [ROOT_CAUSE_PROVED_BY]: Hệ thống có rate limit, khi 20 concurrent users × 27 queries = 540 requests/minute vượt quá capacity, requests bị queue và xử lý chậm (truy vết qua Langfuse Trace Waterfall).
- [FIX_ACTION]: Tắt toggle incident hoặc tăng rate limit threshold, requests được xử lý nhanh hơn, latency giảm.
- [PREVENTIVE_MEASURE]: Implement token bucket algorithm, add user-level rate limiting, setup queue system cho excess requests, scale API gateway horizontally, add load balancer.

---

## 5. Individual Contributions & Evidence

### [MEMBER_A_NAME]
- [TASKS_COMPLETED]: 
- [EVIDENCE_LINK]: (Link to specific commit or PR)

### Đàm Lê Văn Toàn - Tracing & Enrichment
- **[TASKS_COMPLETED]**: 
  1. Tích hợp Langfuse SDK, sử dụng `@observe(capture_input=False, capture_output=False)` để kiểm soát thủ công luồng dữ liệu trace thay vì tự động chụp, phòng tránh lộ lọt PII.
  2. Bổ sung Enrichment Metadata qua `update_current_trace`: Map Trace bằng `session_id` và `user_id` đã được băm (hash), gắn thêm các Custom Tags (`"banking"`, `"credit-card"`) để dễ phân loại trên giao diện Langfuse.
  3. Custom Observation bằng `update_current_observation`: Áp dụng làm sạch dữ liệu (`scrub_text`) vào input/output nội dung hội thoại trước khi trace, cấu hình metadata (`doc_count`, `query_preview`), bóc tách chi phí thông qua `usage_details`.
  4. Hỗ trợ tinh chỉnh PII logic (cho `passport`), bổ sung Alert Rule (`core_banking_500_error`) và dữ liệu đầu vào chứa các thông tin nhạy cảm định dạng CMND/Thẻ tín dụng để verify hệ thống.
  5. Triển khai cơ chế giả lập lỗi ngẫu nhiên 10% (`random_10_percent_error`) để kiểm tra khả năng chịu lỗi của hệ thống. Tối ưu hóa thứ tự Capture Trace (đưa lên đầu hàm `run`) để đảm bảo ngay cả khi xảy ra Exception, hệ thống vẫn thu thập đầy đủ Metadata (User ID, Session ID, Tags) phục vụ việc Debugging.
- **[EVIDENCE_LINK]**: https://github.com/TBNRGarret/Lab13-A1-C401/commit/985b12735977250a60b1c412a87317c16352b458

### [Vũ Lê Hoàng - SLO & Alerts]
- [TASKS_COMPLETED]: Add SLO and Alerts, add blueprint report
- [EVIDENCE_LINK]: https://github.com/TBNRGarret/Lab13-A1-C401/commit/0bc2380cf7d704bed079b067e18e7f7263c8e3bb

### Nguyễn Quang Trường - Load Test + Incident Injection
- **[TASKS_COMPLETED]**:
  1. Enhanced Load Test Script (`scripts/load_test.py`): Thêm class `LoadTestMetrics` để track metrics (latency P50/P95/P99, error rate, cost), feature performance breakdown (credit_inquiry, card_info, account_balance, loan_status, payment_schedule), error tracking, và comprehensive summary report.
  2. Banking-Specific Queries (`data/sample_queries.jsonl`): Thêm 15 câu hỏi banking realistic với PII (email, phone, CCCD, credit card) để test scrubbing và verify chatbot hiểu tiếng Việt.
  3. Banking-Specific Incidents (`app/incidents.py`, `app/mock_rag.py`, `app/mock_llm.py`, `app/agent.py`): Implement 5 incidents - `account_lookup_slow` (simulate database chậm 2.5s), `credit_check_fail` (simulate credit check API down), `high_token_usage` (simulate LLM verbose 2x tokens), `rate_limiter_triggered` (simulate rate limiting), `core_banking_fail` (simulate core banking sập).
  4. Incident Logic Implementation: Thêm logic simulate incidents trong mock_rag.py (time.sleep, RuntimeError), mock_llm.py (output_tokens *= 2), agent.py (raise RuntimeError cho credit_check_fail).
  5. Incident Status Endpoint (`app/main.py`): Thêm GET `/incidents/status` endpoint để check status tất cả incidents.
- **[EVIDENCE_LINK]**: https://github.com/TBNRGarret/Lab13-A1-C401/commit/c3ef40a

### [MEMBER_E_NAME]
- [TASKS_COMPLETED]: 
- [EVIDENCE_LINK]: 

### Member F: Phạm Tuấn Anh - Blueprint & Demo Lead
**[TASKS_COMPLETED]**:
1. Tổ chức và chuẩn hóa file báo cáo `blueprint-template.md`, thu thập Evidence (Tracing ảnh, bảng tính SLO, Error rate) từ các thành viên để hoàn thiện tài liệu nộp lab.
2. Viết toàn bộ Kịch bản Demo (Demo Script) bao gồm 3 phần: Chạy luồng chuẩn (Normal Flow) nhằm giới thiệu Regex che Credit Card, luồng sự cố (Incident Flow) giả lập lỗi Core Banking 500, và luồng Debugging tìm Root Cause qua Grafana/Langfuse.
3. Đóng vai trò Demo Lead trong buổi thuyết trình, kết nối và dẫn dắt giải thích cách hệ thống Log + Trace + Metrics phối hợp để đạt mục tiêu Observability theo chuẩn PCI-DSS (đối với dữ liệu tài chính).
- **[EVIDENCE_LINK]**: 
  - [4644c3b](https://github.com/TBNRGarret/Lab13-A1-C401/commit/4644c3b) - Update rule VIETNAMESE address
  - [4c3c927](https://github.com/TBNRGarret/Lab13-A1-C401/commit/4c3c927) - Update PII Rule
  - [36e9464](https://github.com/TBNRGarret/Lab13-A1-C401/commit/36e9464) - Implement logging correlation ID and PII redaction

---


## 6. Bonus Items (Optional)
- [BONUS_COST_OPTIMIZATION]: (Description + Evidence)
- [BONUS_AUDIT_LOGS]: (Description + Evidence)
- [BONUS_CUSTOM_METRIC]: (Description + Evidence)
