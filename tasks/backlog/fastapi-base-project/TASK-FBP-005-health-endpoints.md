---
id: TASK-FBP-005
title: Health endpoints and app factory
task_type: feature
parent_review: TASK-REV-01B0
feature_id: FEAT-FBP
status: in_review
wave: 3
implementation_mode: task-work
complexity: 4
dependencies:
- TASK-FBP-002
- TASK-FBP-003
- TASK-FBP-004
priority: high
tags:
- health
- endpoints
- app-factory
consumer_context:
- task: TASK-FBP-002
  consumes: SETTINGS
  framework: Pydantic BaseSettings
  driver: pydantic-settings
  format_note: Settings instance passed to create_app(); fields include environment,
    app_version, log_level, log_format, api_prefix
- task: TASK-FBP-003
  consumes: LOGGING_SETUP
  framework: Python stdlib logging
  driver: logging
  format_note: setup_logging(settings) must be called in create_app() before routes
    are registered
- task: TASK-FBP-004
  consumes: CORRELATION_ID_MIDDLEWARE
  framework: Starlette middleware
  driver: starlette
  format_note: CorrelationIdMiddleware must be added to app via app.add_middleware()
    in create_app()
autobuild_state:
  current_turn: 1
  max_turns: 30
  worktree_path: /Users/richardwoollcott/Projects/appmilla_github/vllm-profiling/.guardkit/worktrees/FEAT-1637
  base_branch: main
  started_at: '2026-03-07T12:37:00.219467'
  last_updated: '2026-03-07T12:42:29.964800'
  turns:
  - turn: 1
    decision: approve
    feedback: null
    timestamp: '2026-03-07T12:37:00.219467'
    player_summary: Implementation via task-work delegation
    player_success: true
    coach_success: true
---

# Task: Health endpoints and app factory

## Description

Implement the FastAPI application factory (`create_app()`) and health check endpoints (`/health`, `/live`, `/ready`). The factory accepts optional Settings for testability and wires up middleware, logging, and routers. Health endpoints return structured JSON responses with application status, version, and environment.

## Acceptance Criteria

- [ ] `src/main.py` contains `create_app(settings: Settings | None = None) -> FastAPI`
- [ ] When settings is None, creates default Settings() from environment
- [ ] App factory configures: CorrelationIdMiddleware, structured logging, health router
- [ ] `src/health/router.py` with APIRouter and three endpoints
- [ ] `GET /health` returns `{"status": "ok", "version": "<version>", "environment": "<env>"}`
- [ ] `GET /live` returns `{"alive": true}` (or equivalent boolean field)
- [ ] `GET /ready` returns `{"ready": true}` (or equivalent boolean field)
- [ ] `src/health/schemas.py` with Pydantic response models: HealthResponse, LiveResponse, ReadyResponse
- [ ] All health endpoints are served under the configured API prefix (e.g., `/v1/health`)
- [ ] POST/PUT/DELETE to health endpoints returns 405 Method Not Allowed
- [ ] Unknown routes return JSON error `{"detail": "Not Found"}`, not HTML
- [ ] Malformed content-type headers do not crash the application
- [ ] Application handles requests immediately after startup (no warm-up gap)
- [ ] Global exception handler returns JSON for unhandled exceptions

## BDD Scenarios Covered

- Key-example: Health endpoint returns application status
- Key-example: Liveness probe indicates the application is alive
- Key-example: Readiness probe indicates the application is ready
- Key-example: The application is created with custom settings
- Boundary: All endpoints are served under the configured API prefix
- Negative: An unknown route returns a JSON error
- Negative: Health endpoint rejects unsupported HTTP methods
- Edge: Malformed content-type header is handled gracefully
- Edge: Application handles requests immediately after startup

## Implementation Notes

- App factory pattern: `create_app()` is the single entry point for creating the FastAPI app
- Include `app = create_app()` at module level for uvicorn: `uvicorn src.main:app`
- Use `response_model` on routes for automatic schema validation
- Add global exception handler for unhandled errors returning JSON (not default HTML)
- Router prefix: `settings.api_prefix` (e.g., "/v1")
- Tags: `["health"]` for OpenAPI grouping

## Seam Tests

The following seam tests validate the integration contracts with producer tasks.

```python
"""Seam test: verify SETTINGS contract from TASK-FBP-002."""
import pytest


@pytest.mark.seam
@pytest.mark.integration_contract("SETTINGS")
def test_settings_fields_accessible():
    """Verify Settings has all fields needed by create_app().

    Contract: Settings instance with environment, app_version, log_level, log_format, api_prefix
    Producer: TASK-FBP-002
    """
    from src.core.config import Settings

    s = Settings()
    assert hasattr(s, "environment"), "Settings must have environment field"
    assert hasattr(s, "app_version"), "Settings must have app_version field"
    assert hasattr(s, "log_level"), "Settings must have log_level field"
    assert hasattr(s, "log_format"), "Settings must have log_format field"
    assert hasattr(s, "api_prefix"), "Settings must have api_prefix field"
```

```python
"""Seam test: verify LOGGING_SETUP contract from TASK-FBP-003."""
import pytest


@pytest.mark.seam
@pytest.mark.integration_contract("LOGGING_SETUP")
def test_setup_logging_callable():
    """Verify setup_logging() accepts Settings parameter.

    Contract: setup_logging(settings) callable before routes registered
    Producer: TASK-FBP-003
    """
    from src.core.logging import setup_logging
    from src.core.config import Settings

    s = Settings()
    # Should not raise
    setup_logging(s)
```

```python
"""Seam test: verify CORRELATION_ID_MIDDLEWARE contract from TASK-FBP-004."""
import pytest


@pytest.mark.seam
@pytest.mark.integration_contract("CORRELATION_ID_MIDDLEWARE")
def test_middleware_importable():
    """Verify CorrelationIdMiddleware is importable and can be added to app.

    Contract: CorrelationIdMiddleware added via app.add_middleware()
    Producer: TASK-FBP-004
    """
    from src.core.middleware import CorrelationIdMiddleware

    assert callable(CorrelationIdMiddleware), "Must be a callable class"
```
