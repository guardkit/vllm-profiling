# Implementation Guide: AutoBuild Benchmark Suite

**Feature ID**: FEAT-4A8A
**Parent Review**: TASK-REV-E9FD
**Tasks**: 7 (98 BDD scenarios total)
**Pre-requisite**: FastAPI Base Project (FEAT-1637) must be implemented first

## Pre-Implementation Decisions (Resolved)

These design decisions were resolved during the TASK-REV-E9FD review:

1. **Path collision**: BENCH-001 and BENCH-003 share `/v1/reference/` prefix. Both registered in single `src/reference/router.py` with static `/document-types` route BEFORE parametric `/{vocabulary_name}`.

2. **Shared modules**: `PaginatedResponse` in `src/pagination.py`, `ErrorResponse`/`ErrorCode` in `src/errors.py`. Not in feature modules.

3. **Middleware ordering**: Correlation ID middleware (innermost) -> Logging middleware -> Exception handlers. Documented in `src/main.py`.

4. **BENCH-006 counter**: `itertools.count(start=1)` for atomic sequential reference generation.

5. **BENCH-007 locking**: `asyncio.Lock` per submission ID for concurrent transition safety.

6. **BENCH-007 events**: Discriminated union (`TransitionEvent`) with per-action Pydantic models.

## Execution Strategy: 3 Waves

### Wave 1: Independent Tier A Utilities (Parallel)

| Task | Title | Complexity | Method | Workspace |
|------|-------|-----------|--------|-----------|
| TASK-ABS-001 | Reference Data Validator | 3 | task-work | abs-wave1-1 |
| TASK-ABS-002 | Fee Calculator | 3 | task-work | abs-wave1-2 |
| TASK-ABS-003 | Pagination Helper | 3 | task-work | abs-wave1-3 |

**Dependencies**: FEAT-1637 (base project) only
**Coordination**: TASK-ABS-001 and TASK-ABS-003 share `src/reference/router.py`. If running in parallel workspaces, merge carefully. Alternatively, run TASK-ABS-003 first to establish the router, then TASK-ABS-001 adds to it.

**Key deliverables**:
- `src/reference/router.py` - shared router (BENCH-001 + BENCH-003)
- `src/reference/validator.py` - ReferenceDataValidator class
- `src/reference/constants.py` - vocabulary constants
- `src/fees/` - fee calculator module
- `src/pagination.py` - shared PaginatedResponse[T], PaginationParams, paginate()

### Wave 2: Global Middleware (Parallel)

| Task | Title | Complexity | Method | Workspace |
|------|-------|-----------|--------|-----------|
| TASK-ABS-004 | Structured Error Responses | 4 | task-work | abs-wave2-1 |
| TASK-ABS-005 | Request/Response Logging | 4 | task-work | abs-wave2-2 |

**Dependencies**: FEAT-1637 (base project) only
**Coordination**: Both modify `src/main.py` to register handlers/middleware. If parallel, merge carefully. Both read the correlation ID context variable from the base project.

**Key deliverables**:
- `src/errors.py` - ErrorCode, ErrorResponse, exception handlers
- `src/middleware/request_logging.py` - logging middleware
- Updates to `src/main.py` - handler registration with documented ordering

### Wave 3: Tier B Features (Parallel)

| Task | Title | Complexity | Method | Workspace |
|------|-------|-----------|--------|-----------|
| TASK-ABS-006 | Client Record API | 5 | task-work | abs-wave3-1 |
| TASK-ABS-007 | Submission State Machine | 6 | task-work | abs-wave3-2 |

**Dependencies**: FEAT-1637, TASK-ABS-003 (pagination), TASK-ABS-004 (errors)
**Coordination**: Fully independent of each other. Can run in parallel without merge conflicts.

**Key deliverables**:
- `src/clients/` - client CRUD module with in-memory store
- `src/submissions/` - submission state machine module with in-memory store

## Integration Contracts

| Contract | Producer | Consumer | Interface |
|----------|----------|----------|-----------|
| `PaginatedResponse[T]` | TASK-ABS-003 | TASK-ABS-006, 007 | `from src.pagination import PaginatedResponse, PaginationParams, paginate` |
| `ErrorResponse` / `ErrorCode` | TASK-ABS-004 | TASK-ABS-006, 007 (test assertions) | `from src.errors import ErrorResponse, ErrorCode` |
| Correlation ID context var | FEAT-1637 | TASK-ABS-004, 005 | `from src.middleware.correlation import correlation_id_ctx` |
| Vocabulary constants | TASK-ABS-001 | - | Self-contained, not imported by other benchmarks |
| Reference router | TASK-ABS-001 + 003 | `src/main.py` | `from src.reference.router import router as reference_router` |

## File Conflict Matrix

Files that may be modified by multiple tasks (merge coordination required):

| File | Tasks | Resolution |
|------|-------|------------|
| `src/main.py` | ABS-004, ABS-005, ABS-006, ABS-007 | Each adds router/middleware registration. Sequential or careful merge. |
| `src/reference/router.py` | ABS-001, ABS-003 | Both add routes. Static before parametric. |

## Quality Gates

Each task must pass before marking complete:
- [ ] All BDD scenarios pass (`pytest tests/{module}/test_bench_XXX.py -v`)
- [ ] No ruff violations (`ruff check src/{module}/`)
- [ ] Type annotations complete (`mypy src/{module}/`)
- [ ] Coverage >= 80% for feature module

## Estimated Timeline

| Phase | Duration (parallel) | Cumulative |
|-------|-------------------|------------|
| Wave 1 | ~3 hours | 3h |
| Wave 2 | ~3 hours | 6h |
| Wave 3 | ~5 hours | 11h |
| Integration verification | ~1 hour | 12h |
