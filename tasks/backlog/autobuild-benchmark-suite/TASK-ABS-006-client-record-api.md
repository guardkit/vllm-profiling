---
id: TASK-ABS-006
title: "BENCH-006: Client Record API"
status: pending
created: 2026-03-06T00:00:00Z
updated: 2026-03-06T00:00:00Z
priority: high
tags: [benchmark, tier-b, autobuild]
task_type: feature
complexity: 5
parent_review: TASK-REV-E9FD
feature_id: FEAT-4A8A
wave: 3
implementation_mode: task-work
dependencies: [FEAT-1637, TASK-ABS-003, TASK-ABS-004]
test_results:
  status: pending
  coverage: null
  last_run: null
---

# TASK-ABS-006: Client Record API (BENCH-006)

## Description

Implement a full CRUD API for client records using an in-memory store. Includes auto-generated UUID IDs, sequential references (CLT-{6 digits}), name/email validation, status/jurisdiction enums, paginated listing with status filtering, and concurrency-safe reference generation.

## Feature Spec

`features/autobuild-benchmark-suite/bench-006-client-record-api.feature` (21 scenarios)

## Scope

### Feature Module (`src/clients/`)
- `router.py` - CRUD endpoints
- `schemas.py` - ClientCreate, ClientUpdate, ClientResponse
- `service.py` - Business logic, in-memory store
- `constants.py` - ClientStatus enum (active, suspended, pending_review, closed), Jurisdiction enum (ENG, WAL, SCO, NIR)

### API Endpoints
- `POST /v1/clients` - Create client (returns UUID ID, CLT-XXXXXX reference, timestamps)
- `GET /v1/clients/{id}` - Get client by UUID
- `PATCH /v1/clients/{id}` - Partial update (only provided fields change, updated_at advances)
- `GET /v1/clients` - Paginated list with optional `?status=` filter
- `DELETE /v1/clients/{id}` - Delete (204 No Content)

### Validation
- Name: 1-200 characters
- Email: valid email format (Pydantic EmailStr or regex)
- Status: must be valid ClientStatus enum value
- Jurisdiction: must be valid Jurisdiction enum value
- Non-existent client ID: 404 with ErrorResponse format (from BENCH-004)

### Concurrency Safety
- Sequential reference counter: use `itertools.count(start=1)` for atomic `next()` calls
- Reference format: `CLT-{counter:06d}` (e.g., CLT-000001)
- Scenario tests 10 simultaneous creates producing unique references

### Integration
- Imports `PaginatedResponse`, `PaginationParams` from `src/pagination.py` (BENCH-003)
- Error responses use `ErrorResponse` format from `src/errors.py` (BENCH-004)
- Empty update body changes nothing (returns current state)
- Path traversal in client ID parameter returns 404 safely
- XSS payload in name/email stored literally (no sanitisation, no execution)

## Acceptance Criteria

- [ ] All 21 BDD scenarios pass
- [ ] Create returns UUID ID, CLT-XXXXXX reference, timestamps
- [ ] Get by ID returns full record
- [ ] Partial update changes only specified fields, advances updated_at
- [ ] Paginated list with pagination metadata
- [ ] Status filter on list endpoint
- [ ] Delete returns 204, subsequent get returns 404
- [ ] Name boundary: 1 char accepted, 200 chars accepted, 201 chars rejected
- [ ] All status values accepted
- [ ] All jurisdiction values accepted
- [ ] Invalid email rejected
- [ ] Invalid jurisdiction rejected
- [ ] Non-existent ID returns 404 for get/update/delete
- [ ] Each client gets unique reference
- [ ] 10 simultaneous creates produce unique references
- [ ] XSS payload stored safely
- [ ] Path traversal handled safely

## Test File

`tests/clients/test_bench_006_client_record_api.py`
