---
id: TASK-FBP-007
title: 'Quality gates: ruff, mypy, pytest-cov configuration'
task_type: scaffolding
parent_review: TASK-REV-01B0
feature_id: FEAT-FBP
status: in_review
wave: 4
implementation_mode: direct
complexity: 3
dependencies:
- TASK-FBP-006
priority: normal
tags:
- quality
- linting
- typing
- coverage
autobuild_state:
  current_turn: 1
  max_turns: 30
  worktree_path: /Users/richardwoollcott/Projects/appmilla_github/vllm-profiling/.guardkit/worktrees/FEAT-1637
  base_branch: main
  started_at: '2026-03-07T12:42:29.994752'
  last_updated: '2026-03-07T12:51:02.864566'
  turns:
  - turn: 1
    decision: approve
    feedback: null
    timestamp: '2026-03-07T12:42:29.994752'
    player_summary: '[RECOVERED via player_report] Original error: Cancelled: Cancelled
      via cancel scope 110844f50 by <Task pending name=''Task-148'' coro=<<async_generator_athrow
      without __name__>()>>'
    player_success: true
    coach_success: true
---

# Task: Quality gates: ruff, mypy, pytest-cov configuration

## Description

Configure and verify all quality gate tools: ruff linting and formatting, mypy strict type checking, and pytest-cov coverage thresholds. Ensure all code passes all quality gates with zero violations.

## Acceptance Criteria

- [ ] `pyproject.toml` ruff config: select recommended rules, target Python 3.11+, line-length 88
- [ ] `pyproject.toml` mypy config: strict mode, disallow_untyped_defs=true, warn_return_any=true
- [ ] `pyproject.toml` pytest-cov config: minimum line coverage 80%, minimum branch coverage 75%
- [ ] `ruff check src/ tests/` passes with zero violations
- [ ] `ruff format --check src/ tests/` passes (code is formatted)
- [ ] `mypy src/` passes with zero errors in strict mode
- [ ] `pytest --cov=src --cov-report=term --cov-fail-under=80` passes
- [ ] All type annotations are complete — no `Any` types unless explicitly justified
- [ ] CI-ready script or Makefile target for running all quality gates

## Implementation Notes

- Ruff rules: select = ["E", "F", "W", "I", "UP", "B", "SIM", "C4"]
- Ruff ignore: rules that conflict with formatter
- mypy: use strict mode, plugins for pydantic if needed
- Coverage: src/ package only, exclude tests from coverage measurement
- Consider a simple Makefile or scripts section in pyproject.toml for running all gates
