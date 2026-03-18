---
id: TASK-FBP-001
title: Project scaffolding and directory structure
task_type: scaffolding
parent_review: TASK-REV-01B0
feature_id: FEAT-FBP
status: in_review
wave: 1
implementation_mode: task-work
complexity: 3
dependencies: []
priority: high
tags:
- scaffolding
- setup
autobuild_state:
  current_turn: 1
  max_turns: 30
  worktree_path: /Users/richardwoollcott/Projects/appmilla_github/vllm-profiling/.guardkit/worktrees/FEAT-1637
  base_branch: main
  started_at: '2026-03-07T12:23:25.141045'
  last_updated: '2026-03-07T12:27:47.096128'
  turns:
  - turn: 1
    decision: approve
    feedback: null
    timestamp: '2026-03-07T12:23:25.141045'
    player_summary: Implementation via task-work delegation
    player_success: true
    coach_success: true
---

# Task: Project scaffolding and directory structure

## Description

Create the project directory structure, pyproject.toml, requirements files, and all __init__.py files for the FastAPI base project. This is the foundation task that all other tasks depend on.

## Acceptance Criteria

- [ ] Directory structure created: `src/`, `src/health/`, `src/core/`, `tests/`, `tests/health/`, `tests/core/`, `requirements/`
- [ ] `pyproject.toml` configured with project metadata, ruff (linter + formatter), mypy (strict mode), pytest (asyncio_mode = "auto"), and pytest-cov settings
- [ ] `requirements/base.txt` contains: fastapi>=0.104.0, uvicorn[standard], pydantic>=2.0.0, pydantic-settings>=2.0.0
- [ ] `requirements/dev.txt` contains: pytest>=7.4.0, pytest-asyncio>=0.21.0, httpx>=0.25.0, pytest-cov, ruff, mypy
- [ ] All `__init__.py` files created for packages: `src/`, `src/health/`, `src/core/`
- [ ] `.env.example` with documented environment variables (APP_ENVIRONMENT, LOG_LEVEL, LOG_FORMAT, APP_VERSION, API_PREFIX)
- [ ] Project can be installed in development mode (`pip install -e ".[dev]"` or `pip install -r requirements/dev.txt`)

## Implementation Notes

- Use pyproject.toml as the single source of project configuration (no setup.py/setup.cfg)
- Configure ruff with recommended rules + FastAPI best practices
- Configure mypy in strict mode with 100% annotation coverage target
- Configure pytest with asyncio_mode = "auto" to avoid @pytest.mark.asyncio on every test
- No database dependencies (zero external dependencies requirement)
