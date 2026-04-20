# Day 13 Observability Lab Report

> **Instruction**: Fill in all sections below. This report is designed to be parsed by an automated grading assistant. Ensure all tags (e.g., `[GROUP_NAME]`) are preserved.

## 1. Team Metadata
- GROUP_NAME: A1-C401
- REPO_URL: https://github.com/TBNRGarret/Lab13-A1-C401.git
- MEMBERS:
  - Member A: [Name] | Role: Logging & PII
  - Member B: [Đàm Lê Văn Toàn] | Role: Tracing & Enrichment
  - Member C: Vũ Lê Hoàng | Role: SLO & Alerts
  - Member D: [Name] | Role: Load Test + Incident Injection
  - Member E: [Name] | Role: Dashboard + Evidence
  - Member F: Phạm Tuấn Anh | Role: Blueprint + Demo lead

---

## 2. Group Performance (Auto-Verified)
- [VALIDATE_LOGS_FINAL_SCORE]: 80/100
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
- SLO_TABLE:

| SLI | Target | Window | Current Value |
|---|---:|---|---:|
| Latency P95 | < 1000ms | 28d | 151ms |
| Error Rate | < 1% | 28d | 0% |
| Cost Budget | < $5.00/day | 1d | ~$2.10/day baseline |
| Quality Score Avg | > 0.70 | 28d | 0.88 |

### 3.3 Alerts & Runbook
- [ALERT_RULES_SCREENSHOT]: [Path to image]
- [SAMPLE_RUNBOOK_LINK]: [docs/alerts.md#L...]

---

## 4. Incident Response (Group)
- [SCENARIO_NAME]: core_banking_fail
- [SYMPTOMS_OBSERVED]: Khách hàng gặp lỗi 500 khi kiểm tra số dư, Error Rate trên Dashboard tăng vọt, Alert high_error_rate (P1) được kích hoạt.
- [ROOT_CAUSE_PROVED_BY]: Hàm `mock_core_banking_api()` gặp lỗi 'Connection Timeout to Core Banking' (truy vết qua Langfuse Trace Waterfall).
- [FIX_ACTION]: Tắt toggle giả lập lỗi (disable incident), ứng dụng phục hồi và Error Rate giảm dần.
- [PREVENTIVE_MEASURE]: Triển khai cơ chế Circuit Breaker hoặc Fallback API để đảm bảo tính sẵn sàng khi Core Banking gặp sự cố.

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

### Vũ Lê Hoàng - SLO & Alerts
- **TASKS_COMPLETED**:
  1. Xây dựng bộ SLO cho hệ thống quan sát gồm 4 chỉ số chính: `Latency P95 < 1000ms`, `Error Rate < 1%`, `Cost Budget < $5.00/day`, và `Quality Score Avg > 0.70`, đồng thời cập nhật bảng `[SLO_TABLE]` để nhóm theo dõi được ngưỡng, cửa sổ đo và giá trị hiện tại.
  2. Cấu hình bộ Alert Rules trong `config/alert_rules.yaml` theo mức độ ưu tiên P1/P2/P3, bao phủ các tình huống `high_error_rate`, `high_latency_p95`, `cost_budget_spike`, `low_quality_score`, `token_budget_exceeded`, và `core_banking_500_error` để hệ thống có thể cảnh báo sớm cả lỗi dịch vụ lẫn vượt ngân sách vận hành.
  3. Viết runbook tại `docs/alerts.md` theo quy trình `Detect -> Triage -> Mitigate -> Prevent`, liên kết giữa Dashboard, `/metrics`, logs và Langfuse traces, giúp On-call engineer nhanh chóng xác định nguyên nhân và hướng xử lý khi alert được kích hoạt.
  4. Hoàn thiện phần báo cáo cá nhân trong blueprint, tổng hợp lại mục tiêu SLO, cơ chế alert và bằng chứng commit để thể hiện rõ đóng góp của vai trò `SLO & Alerts` trong bài lab Observability.
- **[EVIDENCE_LINK]**: https://github.com/TBNRGarret/Lab13-A1-C401/commit/0bc2380cf7d704bed079b067e18e7f7263c8e3bb
- **[EVIDENCE_LINK]**: https://github.com/TBNRGarret/Lab13-A1-C401/commit/02f7ff3e09b9b95638d531b8510b0c8cdc69577b
- **[EVIDENCE_LINK]**: https://github.com/TBNRGarret/Lab13-A1-C401/commit/335ebe2f71a8872c72eb05469d9fb71dd09e54dc

### [MEMBER_D_NAME]
- [TASKS_COMPLETED]: 
- [EVIDENCE_LINK]: 

### [MEMBER_E_NAME]
- [TASKS_COMPLETED]: 
- [EVIDENCE_LINK]: 

### Member F: Phạm Tuấn Anh - Blueprint & Demo Lead
**[TASKS_COMPLETED]**:
1. Tổ chức và chuẩn hóa file báo cáo `blueprint-template.md`, thu thập Evidence (Tracing ảnh, bảng tính SLO, Error rate) từ các thành viên để hoàn thiện tài liệu nộp lab.
2. Viết toàn bộ Kịch bản Demo (Demo Script) bao gồm 3 phần: Chạy luồng chuẩn (Normal Flow) nhằm giới thiệu Regex che Credit Card, luồng sự cố (Incident Flow) giả lập lỗi Core Banking 500, và luồng Debugging tìm Root Cause qua Grafana/Langfuse.
3. Đóng vai trò Demo Lead trong buổi thuyết trình, kết nối và dẫn dắt giải thích cách hệ thống Log + Trace + Metrics phối hợp để đạt mục tiêu Observability theo chuẩn PCI-DSS (đối với dữ liệu tài chính).
- **[EVIDENCE_LINK]**: 
  - [7a7a338](https://github.com/TBNRGarret/Lab13-A1-C401/commit/7a7a338) - Finalize Lab 13 Observability Report and Demo Script
  - [8783e9a](https://github.com/TBNRGarret/Lab13-A1-C401/commit/8783e9a) - Fill Member F individual report and incident response

---


## 6. Bonus Items (Optional)
- [BONUS_COST_OPTIMIZATION]: (Description + Evidence)
- [BONUS_AUDIT_LOGS]: (Description + Evidence)
- [BONUS_CUSTOM_METRIC]: (Description + Evidence)
