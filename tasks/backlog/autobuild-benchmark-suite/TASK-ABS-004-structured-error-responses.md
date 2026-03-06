---
id: TASK-ABS-004
title: "BENCH-004: Structured Error Responses"
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

# TASK-ABS-004: Structured Error Responses (BENCH-004)

## Description

Implement a global structured error handling system that wraps all API errors (404, validation errors, unhandled exceptions) into a consistent `ErrorResponse` format with error codes, messages, and request IDs.

## Feature Spec

`features/autobuild-benchmark-suite/bench-004-structured-error-responses.feature` (11 scenarios)

## Scope

### Shared Module (`src/errors.py`)
- `ErrorCode` enum: `VALIDATION_ERROR`, `NOT_FOUND`, `CONFLICT`, `INTERNAL_ERROR`, `RATE_LIMITED`
- `ErrorResponse` Pydantic model: code (ErrorCode), message (str), request_id (str), detail (optional)
- Three exception handlers registered on the FastAPI app:
  1. `RequestValidationError` handler -> 422 with VALIDATION_ERROR code, field-level details
  2. `HTTPException` handler -> preserves status code, maps to appropriate ErrorCode
  3. Generic `Exception` handler -> 500 with INTERNAL_ERROR, safe message "An unexpected error occurred"

### Key Behaviours
- Correlation ID flows through to `request_id` field in error responses
- Generated request_id when no correlation ID header provided
- 500 handler MUST NOT leak exception class, message, or traceback
- Error response serialisation must never raise its own exception (handle non-serialisable data)
- Detail field is optional (absent/null for simple errors like 404)
- Existing endpoints (e.g., fee calculator) automatically use the new format

### Integration Contract
- Exception handlers registered in app factory (`src/main.py`)
- `ErrorResponse` and `ErrorCode` are imported by BENCH-006 and BENCH-007 test assertions
- Requires correlation ID context variable from base project middleware
- Test endpoints that raise specific exceptions may be needed (409 Conflict, RuntimeError, non-serialisable exception)

## Acceptance Criteria

- [ ] All 11 BDD scenarios pass
- [ ] Unknown route returns ErrorResponse with NOT_FOUND code
- [ ] Validation errors return field-level details with VALIDATION_ERROR code
- [ ] Unhandled exceptions return safe INTERNAL_ERROR (no leak)
- [ ] Correlation ID appears in error response request_id
- [ ] HTTPException status codes preserved (e.g., 409)
- [ ] All ErrorCode enum values defined
- [ ] Error serialisation handles non-serialisable exception data

## Test File

`tests/errors/test_bench_004_structured_errors.py`
