---
id: TASK-ABS-003
title: "BENCH-003: Pagination Helper"
status: pending
created: 2026-03-06T00:00:00Z
updated: 2026-03-06T00:00:00Z
priority: high
tags: [benchmark, tier-a, autobuild]
task_type: feature
complexity: 3
parent_review: TASK-REV-E9FD
feature_id: FEAT-4A8A
wave: 1
implementation_mode: task-work
dependencies: [FEAT-1637]
test_results:
  status: pending
  coverage: null
  last_run: null
---

# TASK-ABS-003: Pagination Helper (BENCH-003)

## Description

Implement a reusable pagination utility with `PaginatedResponse[T]` generic type and `PaginationParams` FastAPI dependency. Includes a paginated reference endpoint for document types as a demonstration.

## Feature Spec

`features/autobuild-benchmark-suite/bench-003-pagination-helper.feature` (16 scenarios)

## Scope

### Shared Module (`src/pagination.py`)
- `PaginationParams` - FastAPI dependency with `page` (default 1, min 1) and `page_size` (default 20, min 1, max 100)
- `PaginatedResponse[T]` - Generic Pydantic BaseModel with: items, total, page, page_size, total_pages, has_next, has_previous
- `paginate()` helper function that slices a collection and returns PaginatedResponse
- Offset property calculation: `(page - 1) * page_size`

### API Layer
- `GET /v1/reference/document-types` - paginated list of document type values
- **IMPORTANT**: This route must be declared BEFORE `/{vocabulary_name}` in `src/reference/router.py` to avoid path collision with BENCH-001

### Pydantic v2 Note
- `PaginatedResponse[T]` as `BaseModel, Generic[T]` works in Pydantic v2 when used as `response_model=PaginatedResponse[str]`
- Test round-trip serialisation with `PaginatedResponse[str]` first before building dependent features

### Integration Contract
- `PaginatedResponse` and `PaginationParams` are placed in `src/pagination.py` (shared module)
- BENCH-006 and BENCH-007 will import from this module
- Page beyond total returns empty items (not an error)

## Acceptance Criteria

- [ ] All 16 BDD scenarios pass
- [ ] Default pagination: page=1, page_size=20
- [ ] Correct total_pages, has_next, has_previous calculations
- [ ] Page beyond total returns empty items without error
- [ ] page=0 rejected with validation error
- [ ] page_size=0 rejected with validation error
- [ ] page_size=101 rejected with validation error
- [ ] page_size=100 accepted
- [ ] Offset correctly calculated for all test cases
- [ ] PaginatedResponse works with different item types (generic)
- [ ] Paginated document types endpoint returns correct results

## Test File

`tests/reference/test_bench_003_pagination_helper.py`
