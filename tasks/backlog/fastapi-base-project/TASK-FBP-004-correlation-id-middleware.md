---
id: TASK-FBP-004
title: "Correlation ID middleware with ContextVar"
task_type: feature
parent_review: TASK-REV-01B0
feature_id: FEAT-FBP
status: pending
wave: 2
implementation_mode: task-work
complexity: 5
dependencies:
  - TASK-FBP-001
priority: high
tags: [middleware, correlation-id, tracing]
---

# Task: Correlation ID middleware with ContextVar

## Description

Implement a Starlette middleware that manages correlation IDs for request tracing. The middleware generates a UUID when no correlation ID header is provided, passes through caller-provided IDs, treats empty headers as absent, and uses a ContextVar for async-safe context isolation.

## Acceptance Criteria

- [ ] `src/core/middleware.py` contains `CorrelationIdMiddleware` class
- [ ] `correlation_id_ctx: ContextVar[str | None]` defined at module level and exported
- [ ] When no `X-Correlation-ID` header is present, generates a UUID4 and sets it as the response header
- [ ] When `X-Correlation-ID` header is present with a non-empty value, preserves it in the response
- [ ] When `X-Correlation-ID` header is present but empty, treats it as absent (generates new UUID)
- [ ] Correlation IDs up to 500 characters are preserved without truncation
- [ ] Special characters in correlation ID (e.g., `<script>alert(1)</script>`) are handled safely — passed through in header, no injection
- [ ] Each concurrent request gets its own ContextVar value — no leakage between requests
- [ ] Two consecutive requests without correlation IDs receive different UUIDs
- [ ] ContextVar is set before request processing and available for logging filter
- [ ] ContextVar is reset after each request completes

## BDD Scenarios Covered

- Key-example: A correlation ID is generated when none is provided
- Key-example: A caller-provided correlation ID is preserved
- Boundary: An empty correlation ID header is treated as absent
- Edge: A very long correlation ID is preserved without truncation
- Edge: Each request without a correlation ID receives a unique ID
- Edge: Correlation ID with special characters is handled safely
- Edge: Concurrent requests receive independent correlation IDs
- Edge: Correlation ID context does not leak between requests

## Implementation Notes

- Use `contextvars.ContextVar` (Python stdlib) for async-safe request isolation
- Middleware should be a Starlette `BaseHTTPMiddleware` subclass or raw ASGI middleware
- Header name: `X-Correlation-ID` (case-insensitive read, exact case write)
- UUID generation: `uuid.uuid4()` — standard, no external dependency
- ContextVar must be reset in a `finally` block to prevent leakage
- Consider raw ASGI middleware over BaseHTTPMiddleware for better performance and ContextVar reliability
