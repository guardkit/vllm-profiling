---
id: TASK-REV-E9FD
title: "Plan: AutoBuild Benchmark Suite"
status: review_complete
created: 2026-03-06T00:00:00Z
updated: 2026-03-06T00:00:00Z
priority: high
tags: [review, benchmark, autobuild, planning]
task_type: review
review_mode: architectural
review_depth: standard
complexity: 0
test_results:
  status: pending
  coverage: null
  last_run: null
review_results:
  mode: architectural
  depth: standard
  score: 81
  findings_count: 5
  recommendations_count: 5
  decision: implement
  report_path: .claude/reviews/TASK-REV-E9FD-review-report.md
  completed_at: 2026-03-06T00:00:00Z
---

# Task: Plan: AutoBuild Benchmark Suite

## Description
Review and plan the implementation of a suite of 7 self-contained benchmark tasks (BENCH-001 through BENCH-007) built on the FastAPI base project. The suite is designed to generate comparable `llm.call` instrumentation data across different AutoBuild prompt profiles.

Tasks span Tier A (complexity 3-4, primary tuning) and Tier B (complexity 5, validation), covering:
- BENCH-001: Reference Data Validator (14 scenarios)
- BENCH-002: Fee Calculator (13 scenarios)
- BENCH-003: Pagination Helper (16 scenarios)
- BENCH-004: Structured Error Responses (11 scenarios)
- BENCH-005: Request/Response Logging (11 scenarios)
- BENCH-006: Client Record API (21 scenarios)
- BENCH-007: Submission State Machine (22 scenarios)

Total: 98 scenarios (17 smoke, 33 key-example, 30 boundary, 22 negative, 23 edge-case)

## Context
- Feature spec: features/autobuild-benchmark-suite/autobuild-benchmark-suite_summary.md
- Base project: features/fastapi-base-project/fastapi-base-project_summary.md
- Assumptions: 4 total (all confirmed)

## Acceptance Criteria
- [x] Technical options analyzed for implementation approach
- [x] Architecture implications assessed
- [x] Task breakdown created with dependency analysis
- [x] Risk analysis completed
- [x] Implementation recommendation provided

## Implementation Notes
Review completed. See .claude/reviews/TASK-REV-E9FD-review-report.md for full report.

### Key Findings
1. **Path collision** between BENCH-001 and BENCH-003 on `/v1/reference/` - resolved by single router
2. **Shared infrastructure** (PaginatedResponse, ErrorResponse) belongs in shared modules, not feature modules
3. **Middleware ordering** is a hard constraint - correlation ID must be innermost
4. **Concurrency safety** needed: `itertools.count()` for BENCH-006, `asyncio.Lock` per submission for BENCH-007
5. **TransitionEvent design** should use discriminated union for type safety

### Execution Strategy: 3-Wave Parallel
- Wave 1: BENCH-001, BENCH-002, BENCH-003 (independent Tier A utilities)
- Wave 2: BENCH-004, BENCH-005 (global middleware)
- Wave 3: BENCH-006, BENCH-007 (Tier B, depends on Waves 1+2)
