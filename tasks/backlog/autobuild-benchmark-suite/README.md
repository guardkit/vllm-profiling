# AutoBuild Benchmark Suite

**Feature ID**: FEAT-4A8A
**Parent Review**: [TASK-REV-E9FD](../TASK-REV-E9FD-plan-autobuild-benchmark-suite.md)
**Review Report**: [.claude/reviews/TASK-REV-E9FD-review-report.md](../../.claude/reviews/TASK-REV-E9FD-review-report.md)

## Problem Statement

AutoBuild prompt profiles need comparable `llm.call` instrumentation data to measure and tune performance. A standardised set of benchmark tasks with defined BDD scenarios provides controlled, repeatable workloads for profiling.

## Solution Approach

7 self-contained benchmark tasks built on the FastAPI base project (FEAT-1637), spanning two tiers:

- **Tier A** (BENCH-001 to BENCH-005): Complexity 3-4, primary tuning targets
- **Tier B** (BENCH-006 to BENCH-007): Complexity 5-6, validation targets

All tasks use in-memory stores (no database). Total: 98 BDD scenarios.

## Subtask Summary

| Task | Title | Complexity | Wave | Scenarios |
|------|-------|-----------|------|-----------|
| [TASK-ABS-001](TASK-ABS-001-reference-data-validator.md) | Reference Data Validator | 3 | 1 | 14 |
| [TASK-ABS-002](TASK-ABS-002-fee-calculator.md) | Fee Calculator | 3 | 1 | 13 |
| [TASK-ABS-003](TASK-ABS-003-pagination-helper.md) | Pagination Helper | 3 | 1 | 16 |
| [TASK-ABS-004](TASK-ABS-004-structured-error-responses.md) | Structured Error Responses | 4 | 2 | 11 |
| [TASK-ABS-005](TASK-ABS-005-request-response-logging.md) | Request/Response Logging | 4 | 2 | 11 |
| [TASK-ABS-006](TASK-ABS-006-client-record-api.md) | Client Record API | 5 | 3 | 21 |
| [TASK-ABS-007](TASK-ABS-007-submission-state-machine.md) | Submission State Machine | 6 | 3 | 22 |

## Pre-requisite

The FastAPI Base Project (FEAT-1637) must be implemented first. See [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md) for details.

## Quick Start

```bash
# Start with Wave 1 (after FEAT-1637 is complete)
/task-work TASK-ABS-001   # or parallel execution with Conductor
/task-work TASK-ABS-002
/task-work TASK-ABS-003

# Then Wave 2
/task-work TASK-ABS-004
/task-work TASK-ABS-005

# Finally Wave 3
/task-work TASK-ABS-006
/task-work TASK-ABS-007
```

See [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md) for full wave breakdown, integration contracts, and coordination notes.
