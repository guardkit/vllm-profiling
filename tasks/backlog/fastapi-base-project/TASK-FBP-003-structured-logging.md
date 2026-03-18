---
id: TASK-FBP-003
title: Structured logging with JSON and text formats
task_type: feature
parent_review: TASK-REV-01B0
feature_id: FEAT-FBP
status: in_review
wave: 2
implementation_mode: task-work
complexity: 5
dependencies:
- TASK-FBP-001
- TASK-FBP-002
priority: high
tags:
- logging
- structured
- json
consumer_context:
- task: TASK-FBP-004
  consumes: CORRELATION_ID
  framework: Python contextvars.ContextVar
  driver: stdlib
  format_note: ContextVar[str | None] shared between middleware and logging filter;
    filter reads current value for log injection
autobuild_state:
  current_turn: 1
  max_turns: 30
  worktree_path: /Users/richardwoollcott/Projects/appmilla_github/vllm-profiling/.guardkit/worktrees/FEAT-1637
  base_branch: main
  started_at: '2026-03-07T12:32:00.355904'
  last_updated: '2026-03-07T12:37:00.197879'
  turns:
  - turn: 1
    decision: approve
    feedback: null
    timestamp: '2026-03-07T12:32:00.355904'
    player_summary: Implementation via task-work delegation
    player_success: true
    coach_success: true
---

# Task: Structured logging with JSON and text formats

## Description

Implement structured logging that supports both JSON and human-readable text formats. The logging system must inject correlation IDs from the ContextVar into every log entry and be configurable via the Settings log_format and log_level fields.

## Acceptance Criteria

- [ ] `src/core/logging.py` contains logging configuration setup
- [ ] JSON format produces valid JSON log output with fields: timestamp, level, message, correlation_id
- [ ] Text format produces human-readable log output (not JSON)
- [ ] Correlation ID is injected into log entries via a custom logging Filter that reads from ContextVar
- [ ] Logger is configurable via Settings.log_level and Settings.log_format
- [ ] `setup_logging(settings: Settings)` function configures the root/app logger
- [ ] JSON logging works when output is piped (no TTY dependency)
- [ ] Log output includes: timestamp (ISO 8601), log level, message, correlation_id, logger name

## BDD Scenarios Covered

- Key-example: Structured JSON logging includes the correlation ID
- Boundary: Log output matches the configured format (json/text)
- Edge: Text log format produces human-readable output
- Edge: JSON logging works when output is piped

## Implementation Notes

- Use Python stdlib `logging` module with custom Formatter classes
- JSON formatter: `json.dumps()` with structured fields dict
- Text formatter: standard `logging.Formatter` with correlation_id in format string
- CorrelationIdFilter: reads from `contextvars.ContextVar` set by middleware (TASK-FBP-004)
- The ContextVar will be defined in `src/core/middleware.py` and imported here
- Do NOT use structlog or other third-party logging libraries (zero external dependencies beyond FastAPI stack)

## Seam Tests

The following seam test validates the integration contract with the middleware task. Implement this test to verify the boundary before integration.

```python
"""Seam test: verify CORRELATION_ID contract from TASK-FBP-004."""
import pytest


@pytest.mark.seam
@pytest.mark.integration_contract("CORRELATION_ID")
def test_correlation_id_context_var_accessible():
    """Verify CORRELATION_ID ContextVar is readable by logging filter.

    Contract: ContextVar[str | None] shared between middleware and logging filter
    Producer: TASK-FBP-004
    """
    from src.core.middleware import correlation_id_ctx

    # Verify ContextVar exists and has correct default
    value = correlation_id_ctx.get(None)
    assert value is None, "Default should be None when no request active"

    # Set a value and verify filter can read it
    token = correlation_id_ctx.set("test-correlation-id")
    assert correlation_id_ctx.get() == "test-correlation-id"
    correlation_id_ctx.reset(token)
```
