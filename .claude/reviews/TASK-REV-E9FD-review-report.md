# Review Report: TASK-REV-E9FD

## Executive Summary

The AutoBuild Benchmark Suite comprises 7 self-contained tasks (98 BDD scenarios) built on the FastAPI base project. The architecture is sound and well-scoped. Five design decisions must be resolved before implementation to avoid mid-stream refactoring. The most significant is a path collision between BENCH-001 and BENCH-003, and a middleware ordering constraint affecting BENCH-004 and BENCH-005.

**Architecture Score**: 81/100
**Status**: Approved with Recommendations
**Findings**: 5
**Recommendations**: 5

## Review Details

- **Mode**: Architectural Review
- **Depth**: Standard
- **Task**: TASK-REV-E9FD - Plan: AutoBuild Benchmark Suite
- **Reviewer**: architectural-reviewer agent

## Findings

### Finding 1: Path Ownership Collision (Severity: High)

BENCH-001 specifies `GET /v1/reference/{vocabulary_name}` for vocabulary lookups. BENCH-003 specifies `GET /v1/reference/document-types` for the paginated endpoint. These share the `/v1/reference/` prefix. FastAPI evaluates routes in declaration order within a router, so if the parametric route is declared first, `/document-types` matches as a `{vocabulary_name}` value rather than reaching the static route.

**Evidence**: bench-001 scenario "The reference vocabulary endpoint returns allowed values" and bench-003 scenario "The document types endpoint returns paginated results" both target `/v1/reference/`.

**Recommendation**: Register both endpoints in a single `src/reference/router.py` module with the static path `/document-types` declared before the parametric `/{vocabulary_name}` route.

### Finding 2: Shared Infrastructure Placement (Severity: Medium)

`PaginatedResponse[T]` (BENCH-003) and `ErrorResponse`/`ErrorCode` (BENCH-004) are consumed by BENCH-006 and BENCH-007. Placing these in feature-specific modules creates awkward cross-benchmark imports.

**Recommendation**:

| Artifact | Module |
|----------|--------|
| `PaginationParams`, `PaginatedResponse[T]`, `paginate()` | `src/pagination.py` |
| `ErrorCode`, `ErrorResponse`, exception handlers | `src/errors.py` |
| Vocabulary constants (`DOCUMENT_TYPES`, etc.) | `src/reference/constants.py` |
| `ClientStatus`, `Jurisdiction` enums | `src/clients/constants.py` |

### Finding 3: Middleware Registration Order (Severity: Medium)

Three middleware/handler layers interact:
1. Correlation ID middleware (base project) - sets context var
2. Request/Response Logging middleware (BENCH-005) - reads correlation ID
3. Global exception handlers (BENCH-004) - reads correlation ID for `request_id`

The logging middleware must read the correlation ID after the correlation middleware sets it. Wrong order = empty correlation IDs in log entries.

**Required registration order in app factory**:
1. Correlation ID middleware (innermost)
2. Logging middleware (wraps correlation middleware)
3. Exception handlers (FastAPI-level, registered separately)

**Recommendation**: Document this ordering as a comment in `src/main.py` before BENCH-005 implementation.

### Finding 4: Concurrency Safety for In-Memory Stores (Severity: Medium)

**BENCH-006**: Sequential reference counter (`CLT-{6 digits}`) tested with 10 concurrent creates. `counter += 1` is not atomic in asyncio.

**Recommendation**: Use `itertools.count(start=1)`. `next(counter)` is atomic at CPython level.

**BENCH-007**: "Simultaneous transitions - exactly one succeeds" scenario. Two concurrent approve/reject calls can both read `state == "under_review"` and both succeed.

**Recommendation**: Use `asyncio.Lock` per submission ID (`dict[str, asyncio.Lock]` on the store).

### Finding 5: TransitionEvent Design (Severity: Low)

Each transition requires different mandatory fields (submitted_by, assigned_to, decision_notes, rejection_reason+code). A single flat dataclass with all optional fields loses type safety.

**Recommendation**: Use a discriminated union with Pydantic:
```python
SubmitEvent(action="submit", submitted_by: str)
AssignEvent(action="assign", assigned_to: str)
ApproveEvent(action="approve", decision_notes: str)  # min 10 chars
RejectEvent(action="reject", rejection_reason: str, rejection_code: RejectionCode)
RecallEvent(action="recall")
TransitionEvent = Annotated[Union[...], Field(discriminator="action")]
```

## SOLID/DRY/YAGNI Assessment

| Principle | Score | Notes |
|-----------|-------|-------|
| Single Responsibility | 8/10 | Clean feature mapping; BENCH-004/005 belong in `src/core/` |
| Open/Closed | 7/10 | Fee schedule must be data-driven, not inline conditionals |
| Liskov Substitution | 10/10 | No inheritance hierarchies introduced |
| Interface Segregation | 6/10 | Store dependencies should be narrow, not full class |
| Dependency Inversion | 7/10 | Stores injected as FastAPI dependencies (correct pattern) |
| DRY | 21/25 | PaginatedResponse and ErrorResponse must be shared modules |
| YAGNI | 22/25 | Well-scoped; minor risk of over-generalising TransitionEvent |

## Dependency Graph

```
Base Project (correlation ID, logging, settings, health)
    |
    +---> Wave 1 (parallel, independent):
    |       BENCH-001 (vocabulary validator, /v1/reference/{name})
    |       BENCH-002 (fee calculator, /v1/fees/calculate)
    |       BENCH-003 (pagination utility, shared module extraction)
    |
    +---> Wave 2 (parallel, global middleware):
    |       BENCH-004 (structured errors, exception handlers)
    |       BENCH-005 (request/response logging middleware)
    |
    +---> Wave 3 (parallel, depend on Wave 1+2):
            BENCH-006 (client CRUD, uses pagination + errors)
            BENCH-007 (submission state machine, uses pagination + errors)
```

**Constraint**: Wave 2 must complete before Wave 3 starts. Wave 1 can overlap with Wave 2 except that BENCH-003's `PaginatedResponse` extraction to `src/pagination.py` must land before BENCH-006/007.

## Complexity Assessment

| Task | Spec | Revised | Scenarios | Key Risk |
|------|------|---------|-----------|----------|
| BENCH-001 | 3 | 3 | 14 | Path collision with BENCH-003 |
| BENCH-002 | 3 | 3 | 13 | Fee schedule must be data, not conditionals |
| BENCH-003 | 3 | 3 | 16 | Pydantic v2 generic type serialisation |
| BENCH-004 | 4 | 4 | 11 | Handler registration order in app factory |
| BENCH-005 | 4 | 4 | 11 | Middleware ordering (Finding 3) |
| BENCH-006 | 5 | 5 | 21 | Counter atomicity (Finding 4) |
| BENCH-007 | 5 | 6 | 22 | Per-submission locking + discriminated union |

## Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Route conflict `/v1/reference/` | High (will occur) | High | Single router, static route first |
| Logging middleware logs empty correlation ID | Medium | Medium | Enforce middleware registration order |
| Concurrent BENCH-007 transitions both succeed | High without lock | High | `asyncio.Lock` per submission ID |
| `PaginatedResponse[T]` generic fails Pydantic v2 | Low-Medium | Medium | Test round-trip serialisation first |
| Exception handler short-circuits middleware response path | Low | Low | Confirm ASGI flow |

## Implementation Recommendation

### Approach: 3-Wave Parallel Execution

**Wave 1** (3 tasks, parallel): BENCH-001, BENCH-002, BENCH-003
- All independent Tier A utilities
- BENCH-003 extracts `PaginatedResponse` to `src/pagination.py`
- BENCH-001 and BENCH-003 coordinate via single `src/reference/router.py`

**Wave 2** (2 tasks, parallel): BENCH-004, BENCH-005
- Both are global middleware additions to the app factory
- Must document middleware registration order
- Should integrate with base project's existing correlation ID middleware

**Wave 3** (2 tasks, parallel): BENCH-006, BENCH-007
- Full CRUD + state machine, Tier B validation
- Consume shared pagination and error modules from Waves 1-2
- Each has concurrency scenarios requiring specific patterns

### Pre-Implementation Decisions Required

Before any wave begins:

1. Confirm single `src/reference/` router for BENCH-001 + BENCH-003
2. Confirm `src/pagination.py` and `src/errors.py` as shared module locations
3. Confirm `itertools.count()` for BENCH-006 reference counter
4. Confirm `asyncio.Lock` per submission ID for BENCH-007
5. Confirm discriminated union vs flat dataclass for TransitionEvent

### Estimated Effort

| Wave | Tasks | Estimated Effort | Parallel Time |
|------|-------|-----------------|---------------|
| Wave 1 | BENCH-001, 002, 003 | 3h each | ~3h |
| Wave 2 | BENCH-004, 005 | 3h each | ~3h |
| Wave 3 | BENCH-006, 007 | 4-5h each | ~5h |
| **Total** | **7 tasks** | **~24h sequential** | **~11h parallel** |

## Appendix

### Base Project Dependency

The entire benchmark suite assumes the FastAPI base project (TASK-REV-01B0) is implemented first, providing:
- Health endpoints (`/v1/health`, `/v1/health/live`, `/v1/health/ready`)
- Correlation ID middleware
- Structured JSON logging
- Pydantic Settings
- App factory pattern (`create_app()`)

### Assumptions (All Confirmed)

- ASSUM-001: Health check paths for log suppression (`/v1/health/live`, `/v1/health/ready`)
- ASSUM-002: Rejection codes: `INCOMPLETE`, `INVALID`, `DUPLICATE`, `OUT_OF_SCOPE`
- ASSUM-003: Max correlation ID length: 500 characters
- ASSUM-004: Client reference uses sequential counter (`itertools.count`)
