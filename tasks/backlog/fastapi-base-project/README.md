# Feature: FastAPI Base Project

## Problem Statement

We need a production-structured FastAPI application that serves as the foundation for building features. The base project must include health endpoints (health, liveness, readiness), structured JSON logging with correlation IDs for request tracing, Pydantic settings management with strict validation, and the app factory pattern for testability. The entire application must run and test with zero external dependencies.

## Solution Approach

**Option 1: Lean Feature-Based Structure** (selected)

A feature-based directory structure that follows the template's conventions while being minimal — only files needed for the 28 BDD scenarios. No database layers, no unused stubs.

Key design decisions:
- **App factory pattern**: `create_app(settings=None)` enables test settings injection without env patching
- **ContextVar for correlation IDs**: Python stdlib, async-safe, prevents context leakage between concurrent requests
- **Starlette middleware**: Handles correlation ID generation/passthrough at the request boundary
- **Pydantic Literal types**: Validates environment, log_level, log_format at construction time (not at runtime)

## Subtask Summary

| Task | Description | Complexity | Wave |
|------|------------|-----------|------|
| TASK-FBP-001 | Project scaffolding and directory structure | 3 | 1 |
| TASK-FBP-002 | Pydantic settings with validation | 4 | 2 |
| TASK-FBP-003 | Structured logging with JSON and text formats | 5 | 2 |
| TASK-FBP-004 | Correlation ID middleware with ContextVar | 5 | 2 |
| TASK-FBP-005 | Health endpoints and app factory | 4 | 3 |
| TASK-FBP-006 | Integration tests for all 28 BDD scenarios | 6 | 4 |
| TASK-FBP-007 | Quality gates: ruff, mypy, pytest-cov | 3 | 4 |

## Feature Spec

- **Source**: features/fastapi-base-project/fastapi-base-project.feature
- **Scenarios**: 28 total (6 smoke, 7 key-example, 5 boundary, 5 negative, 11 edge-case)
- **Assumptions**: 3 confirmed (correlation ID max 500 chars, empty treated as absent, 405 for unsupported methods)

## Review Reference

- **Review task**: TASK-REV-01B0
- **Decision**: Implement Option 1 (Lean Feature-Based Structure)
