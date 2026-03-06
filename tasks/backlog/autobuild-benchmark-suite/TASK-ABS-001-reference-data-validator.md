---
id: TASK-ABS-001
title: "BENCH-001: Reference Data Validator"
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

# TASK-ABS-001: Reference Data Validator (BENCH-001)

## Description

Implement a controlled vocabulary validator with a reference data API endpoint. The validator checks values against predefined domain vocabularies (document_types, client_statuses, jurisdiction_codes) with support for case-sensitive and case-insensitive modes.

## Feature Spec

`features/autobuild-benchmark-suite/bench-001-reference-data-validator.feature` (14 scenarios)

## Scope

### Service Layer (`src/reference/`)
- `ReferenceDataValidator` class with case-sensitive and case-insensitive modes
- Vocabulary constants: `DOCUMENT_TYPES` (4 values), `CLIENT_STATUSES` (4 values), `JURISDICTION_CODES` (4 values)
- Immutable after construction (frozen dataclass or `__setattr__` override)
- Validation returns result object with original value, normalised value, success/failure, error message

### API Layer
- `GET /v1/reference/{vocabulary_name}` - returns allowed values for a named vocabulary
- Returns 404 for unknown vocabulary names
- Handles injection characters safely in vocabulary name parameter

### Integration Contract
- **IMPORTANT**: This module shares the `/v1/reference/` prefix with BENCH-003 (pagination endpoint). Both must be registered in a single `src/reference/router.py` with the static `/document-types` route declared BEFORE the parametric `/{vocabulary_name}` route to avoid path collision.

## Acceptance Criteria

- [ ] All 14 BDD scenarios pass
- [ ] Vocabulary constants contain expected values and counts
- [ ] Case-insensitive mode normalises and accepts uppercase input
- [ ] Case-sensitive mode rejects mismatched case
- [ ] Empty vocabulary raises construction error
- [ ] Validator is immutable after construction
- [ ] Unknown vocabulary returns 404
- [ ] Injection characters in vocabulary name handled safely

## Test File

`tests/reference/test_bench_001_reference_data_validator.py`
