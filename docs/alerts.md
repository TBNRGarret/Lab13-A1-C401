# Alert Rules and Runbooks

This document provides step-by-step runbooks for every alert defined in `config/alert_rules.yaml`.

---

## 1. High Latency P95

- **Severity**: P2
- **Trigger**: `latency_p95_ms > 5000 for 30m`
- **Impact**: Tail latency breaches SLO — 0.5% of users experience slow responses

**Investigation Steps**:
1. Open top slow traces in the last 1 hour on Langfuse
2. Compare RAG span duration vs LLM span duration
3. Check if incident toggle `rag_slow` is enabled via `/admin/incidents`
4. Look for `latency_ms` > 5000 in structured logs: `grep '"latency_ms"' data/logs.jsonl | jq 'select(.latency_ms > 5000)'`

**Mitigation**:
- Truncate long queries before RAG retrieval
- Switch to a faster/smaller model for the generation step
- Lower the maximum prompt token limit
- Enable fallback retrieval source

**Escalate to P1 if**: latency_p95_ms > 10000 for more than 1 hour

---

## 2. High Error Rate

- **Severity**: P1
- **Trigger**: `error_rate_pct > 5 for 5m`
- **Impact**: Users receive failed responses — revenue and trust impact

**Investigation Steps**:
1. Group recent logs by `error_type`: `jq 'select(.level=="error") | .error_type' data/logs.jsonl | sort | uniq -c`
2. Inspect failed traces in Langfuse, filter by `status = error`
3. Determine whether failures are LLM-related, tool-related, or schema-related
4. Check for recent deployments or config changes

**Mitigation**:
- Rollback the latest deployment if correlated with error spike
- Disable the failing tool or RAG source temporarily
- Retry with a fallback LLM model
- Apply circuit breaker if downstream service is down

**Escalate if**: error_rate_pct > 20% — declare incident, notify stakeholders

---

## 3. Cost Budget Spike

- **Severity**: P2
- **Trigger**: `hourly_cost_usd > 2x_baseline for 15m`
- **Impact**: Burn rate exceeds budget — potential unexpected costs

**Investigation Steps**:
1. Split recent traces by `feature` and `model` in Langfuse
2. Compare `tokens_in` vs `tokens_out` to find unusually large completions
3. Check if `cost_spike` incident toggle is enabled
4. Identify top 5 most expensive requests by `cost_usd` in logs

**Mitigation**:
- Shorten prompts by reducing RAG context window
- Route easy/simple requests to a cheaper model
- Apply prompt caching for repeated queries
- Set a hard token limit per request

---

## 4. Quality Score Degraded

- **Severity**: P2
- **Trigger**: `quality_score_avg < 0.60 for 15m`
- **Impact**: Responses are incomplete or irrelevant — user experience degrades

**Investigation Steps**:
1. Inspect recent `quality_score` values in metrics: `GET /metrics` endpoint → `quality_avg`
2. Look for patterns: are low-quality requests from a specific feature or query type?
3. Check if the RAG retrieval is returning empty or irrelevant documents
4. Compare `tokens_out` — very short answers often correlate with low quality

**Mitigation**:
- Review and update RAG corpus / embeddings
- Improve the system prompt with more specific instructions
- Add query routing to handle edge cases differently
- If caused by an incident toggle, disable it via `/admin/incidents`

---

## 5. SLO Error Budget Warning

- **Severity**: P3
- **Trigger**: `error_budget_remaining_pct < 20`
- **Impact**: Less than 20% of the 28-day error budget remains — risky to deploy changes

**Investigation Steps**:
1. Review the SLO table in `config/slo.yaml` to identify which SLI is burning fastest
2. Check frequency of alerts in the past 7 days
3. Correlate budget burn with specific incidents or releases

**Mitigation**:
- Freeze non-critical deployments until budget recovers
- Focus engineering effort on reducing the #1 SLI burn source
- Consider a planned maintenance window to address root causes

---

## 6. Low Throughput

- **Severity**: P3
- **Trigger**: `requests_per_minute < 1 for 10m`
- **Impact**: No traffic — possible deployment failure, DNS issue, or load balancer problem

**Investigation Steps**:
1. Verify the application is running: `GET /health` should return `{"status": "ok"}`
2. Check recent deployments or infrastructure changes
3. Validate DNS and load balancer configuration
4. Look at server logs for startup errors

**Mitigation**:
- Restart the application service if unhealthy
- Rollback to the last known good deployment
- Escalate to infrastructure team if DNS/network is the cause
