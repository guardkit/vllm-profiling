---
id: TASK-ABS-007
title: "BENCH-007: Submission State Machine"
status: pending
created: 2026-03-06T00:00:00Z
updated: 2026-03-06T00:00:00Z
priority: high
tags: [benchmark, tier-b, autobuild]
task_type: feature
complexity: 6
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

# TASK-ABS-007: Submission State Machine (BENCH-007)

## Description

Implement a document submission lifecycle state machine with defined states, transitions, history tracking, and concurrency safety. Includes API endpoints for submission CRUD and transition operations.

## Feature Spec

`features/autobuild-benchmark-suite/bench-007-submission-state-machine.feature` (22 scenarios)

## Scope

### Feature Module (`src/submissions/`)
- `router.py` - Submission endpoints + transition endpoints
- `schemas.py` - SubmissionCreate, SubmissionResponse, transition event schemas
- `service.py` - State machine logic, in-memory store
- `constants.py` - SubmissionState enum, RejectionCode enum, transition rules

### State Machine
- **States**: `draft`, `submitted`, `under_review`, `approved`, `rejected`
- **Terminal states**: `approved`, `rejected` (no transitions allowed from these)
- **Transitions**:
  - `submit`: draft -> submitted (requires `submitted_by: str`, non-empty)
  - `assign`: submitted -> under_review (requires `assigned_to: str`)
  - `approve`: under_review -> approved (requires `decision_notes: str`, min 10 chars)
  - `reject`: under_review -> rejected (requires `rejection_reason: str` min 10 chars + `rejection_code: RejectionCode`)
  - `recall`: submitted|under_review -> draft (clears submitted_by, assigned_to)
- **Rejection codes**: `INCOMPLETE`, `INVALID`, `DUPLICATE`, `OUT_OF_SCOPE`

### Transition Event Design (Discriminated Union)
Use Pydantic discriminated union for type safety:
```python
SubmitEvent(action="submit", submitted_by: str)
AssignEvent(action="assign", assigned_to: str)
ApproveEvent(action="approve", decision_notes: str)  # min_length=10
RejectEvent(action="reject", rejection_reason: str, rejection_code: RejectionCode)  # reason min_length=10
RecallEvent(action="recall")
TransitionEvent = Annotated[Union[...], Field(discriminator="action")]
```

### History
- Each transition recorded with: from_state, to_state, action, actor, timestamp
- History returned with submission retrieval
- Chronologically ordered
- Failed transitions do NOT add history entries

### API Endpoints
- `POST /v1/submissions` - Create (starts in `draft`, no history entries)
- `GET /v1/submissions/{id}` - Get with full history
- `GET /v1/submissions` - Paginated list with `?state=` and `?client_id=` filters
- `POST /v1/submissions/{id}/transitions` - Execute transition (accepts TransitionEvent body)
- Invalid transitions return **409 Conflict** (not 400)

### Concurrency Safety
- `asyncio.Lock` per submission ID (`dict[str, asyncio.Lock]` on the store)
- Scenario: simultaneous approve + reject on same submission -> exactly one succeeds
- Failed transition leaves state completely unchanged (state, history, updated_at)

### Integration
- Imports `PaginatedResponse`, `PaginationParams` from `src/pagination.py` (BENCH-003)
- Error responses use `ErrorResponse` format from `src/errors.py` (BENCH-004)
- Injection characters in actor/notes fields stored literally
- Invalid rejection code "MADE_UP_CODE" fails validation

## Acceptance Criteria

- [ ] All 22 BDD scenarios pass
- [ ] New submissions start in draft with empty history
- [ ] Full lifecycle: draft -> submitted -> under_review -> approved (3 transitions in history)
- [ ] Rejection path works from under_review
- [ ] Recall from submitted and under_review returns to draft, clears assignment fields
- [ ] No transitions from approved or rejected (terminal states)
- [ ] Submit requires non-empty submitted_by
- [ ] Approve/reject require decision_notes/reason >= 10 chars
- [ ] Invalid rejection code fails
- [ ] Invalid transition via API returns 409 Conflict
- [ ] History entries in chronological order with correct fields
- [ ] Paginated list with state and client_id filters
- [ ] Simultaneous approve/reject: exactly one succeeds
- [ ] Failed transition leaves state completely unchanged
- [ ] Injection characters handled safely

## Test File

`tests/submissions/test_bench_007_submission_state_machine.py`
