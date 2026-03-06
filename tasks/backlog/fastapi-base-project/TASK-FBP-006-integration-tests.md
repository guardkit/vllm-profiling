---
id: TASK-FBP-006
title: "Integration tests for all 28 BDD scenarios"
task_type: testing
parent_review: TASK-REV-01B0
feature_id: FEAT-FBP
status: pending
wave: 4
implementation_mode: task-work
complexity: 6
dependencies:
  - TASK-FBP-005
priority: high
tags: [testing, integration, bdd]
---

# Task: Integration tests for all 28 BDD scenarios

## Description

Implement the complete test suite covering all 28 BDD scenarios from the feature spec. Tests use httpx.AsyncClient with the app factory for integration testing through the HTTP boundary. Includes conftest.py fixtures, log capture utilities, and concurrency tests.

## Acceptance Criteria

- [ ] `tests/conftest.py` with AsyncClient fixture using `create_app()` with test settings
- [ ] `tests/conftest.py` with log capture fixture for JSON/text log verification
- [ ] `tests/health/test_router.py` — health endpoint smoke and integration tests
- [ ] `tests/core/test_config.py` — settings validation tests (valid values, invalid values, defaults)
- [ ] `tests/core/test_middleware.py` — correlation ID middleware tests (generation, passthrough, empty, long, special chars, concurrency)
- [ ] `tests/core/test_logging.py` — structured logging format tests (JSON valid, text readable, correlation ID in logs)
- [ ] `tests/test_app.py` — app factory tests (custom settings, default settings, edge cases)
- [ ] All 6 smoke scenarios pass
- [ ] All 7 key-example scenarios pass
- [ ] All 5 boundary scenarios pass
- [ ] All 5 negative scenarios pass
- [ ] All 11 edge-case scenarios pass
- [ ] Concurrency test: 10 concurrent requests get independent correlation IDs
- [ ] Concurrency test: correlation ID context does not leak between sequential requests
- [ ] Test coverage >= 80% line coverage, >= 75% branch coverage
- [ ] All tests pass with `pytest` and no environment variables set

## BDD Scenario Mapping

### Smoke Tests (6)
1. Health endpoint returns application status
2. Liveness probe indicates alive
3. Readiness probe indicates ready
4. Correlation ID generated when none provided
5. Caller-provided correlation ID preserved
6. Unknown route returns JSON error

### Key Examples (7)
7. Structured JSON logging includes correlation ID
8. Application created with custom settings
9. (covered by smoke 1-5 above, plus logging and factory)

### Boundary Tests (5)
10. Each valid environment setting reflected (development, staging, production)
11. Each valid log level starts successfully (DEBUG, INFO, WARNING, ERROR)
12. Log output matches configured format (json, text)
13. Empty correlation ID header treated as absent
14. Endpoints served under configured API prefix

### Negative Tests (5)
15. Invalid environment value rejected
16. Invalid log level rejected
17. Invalid log format rejected
18. Unknown route returns JSON not HTML
19. Health endpoint rejects unsupported HTTP methods (405)

### Edge Cases (11)
20. Each request gets unique correlation ID
21. 500-char correlation ID preserved
22. All tests pass with no env vars
23. Custom version reflected in health
24. Text log format produces human-readable output
25. Special characters in correlation ID handled safely
26. Malformed content-type handled gracefully
27. 10 concurrent requests get independent IDs
28. Correlation ID context no leakage between requests
29. JSON logging works when piped (no TTY)
30. App handles requests immediately after startup

## Implementation Notes

- Use `httpx.AsyncClient` with `ASGITransport` for async test client
- Use `create_app(settings=Settings(...))` for settings injection in tests
- Use `asyncio.gather()` for concurrency tests
- Use `caplog` or custom log handler for log capture tests
- For JSON log verification: capture log output, parse with json.loads()
- For "no env vars" test: create Settings with explicit defaults
- pytest-asyncio mode "auto" means no @pytest.mark.asyncio needed
- Trophy model: these are primarily integration tests (50% category)
