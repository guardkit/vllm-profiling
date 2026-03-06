---
id: TASK-ABS-005
title: "BENCH-005: Request/Response Logging Middleware"
status: pending
created: 2026-03-06T00:00:00Z
updated: 2026-03-06T00:00:00Z
priority: high
tags: [benchmark, tier-a, autobuild]
task_type: feature
complexity: 4
parent_review: TASK-REV-E9FD
feature_id: FEAT-4A8A
wave: 2
implementation_mode: task-work
dependencies: [FEAT-1637]
test_results:
  status: pending
  coverage: null
  last_run: null
---

# TASK-ABS-005: Request/Response Logging Middleware (BENCH-005)

## Description

Implement ASGI middleware that emits structured log entries for every request (`request.received`) and response (`request.completed`) with correlation ID tracing, duration measurement, and configurable health check suppression.

## Feature Spec

`features/autobuild-benchmark-suite/bench-005-request-response-logging.feature` (11 scenarios)

## Scope

### Middleware (`src/middleware/request_logging.py`)
- `request.received` log entry: method, path, correlation_id, user_agent (null if absent)
- `request.completed` log entry: status_code, duration_ms (wall clock, >0, <5000), correlation_id, response_size_bytes
- Correlation ID read from context variable set by base project's correlation ID middleware
- JSON structured output when JSON log format configured
- Health check path suppression: `/v1/health/live` and `/v1/health/ready` suppressed when `LOG_HEALTH_CHECKS=false` (default)

### Configuration
- `LOG_HEALTH_CHECKS` setting (boolean, default false) added to Pydantic Settings
- Health check paths: `/v1/health/live`, `/v1/health/ready`

### Middleware Ordering (CRITICAL)
- This middleware MUST be registered AFTER the correlation ID middleware in the app factory
- In ASGI middleware stacks, last added = outermost, so correlation ID middleware must be added first (innermost) to ensure the context var is set before this middleware reads it
- Document ordering constraint as a comment in `src/main.py`

### Key Behaviours
- Each concurrent request gets independent correlation IDs in logs
- Missing optional fields (User-Agent) logged as null
- Duration measured as wall clock time (realistic, not zero)

## Acceptance Criteria

- [ ] All 11 BDD scenarios pass
- [ ] `request.received` contains method, path, correlation_id
- [ ] `request.completed` contains status_code, duration_ms, correlation_id, response_size_bytes
- [ ] Correlation ID consistent across request and response logs
- [ ] Health check paths suppressed when LOG_HEALTH_CHECKS=false
- [ ] Health check paths logged when LOG_HEALTH_CHECKS=true
- [ ] Log output is valid JSON when JSON format configured
- [ ] Missing User-Agent logged as null
- [ ] Duration is >0 and <5000
- [ ] Concurrent requests get independent correlation IDs

## Test File

`tests/middleware/test_bench_005_request_logging.py`
