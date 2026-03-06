---
id: TASK-ABS-002
title: "BENCH-002: Fee Calculator"
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

# TASK-ABS-002: Fee Calculator (BENCH-002)

## Description

Implement a fee calculator for document registration that computes fees based on document type, applicant category, and processing mode. Amounts are in pence (integer) with Decimal pounds conversion. Exempt applicants pay zero. Expedited mode doubles the base fee.

## Feature Spec

`features/autobuild-benchmark-suite/bench-002-fee-calculator.feature` (13 scenarios)

## Scope

### Service Layer (`src/fees/`)
- Fee schedule as data structure (dict/dataclass constant), NOT inline conditionals
- Document types: `standard` (8200 pence), `complex` (16700 pence)
- Applicant categories: `individual`, `organisation`, `exempt`
- Expedited mode doubles base fee
- Exempt overrides all (always 0 pence, regardless of type or mode)
- Amount in pounds uses `Decimal` for precision ("82.00")
- Result object with: amount_pence, amount_pounds, currency ("GBP"), is_exempt, is_expedited
- Custom `FeeCalculationError` for unknown types/categories

### API Layer
- `POST /v1/fees/calculate` - accepts document_type, applicant_category, expedited flag
- Returns fee result as JSON
- Returns structured validation error for missing/invalid fields
- Error messages list valid document types or applicant categories

### Design Note
- Fee calculation is stateless and self-contained (no dependencies on other benchmarks)
- Fee schedule must be data-driven for Open/Closed compliance

## Acceptance Criteria

- [ ] All 13 BDD scenarios pass
- [ ] All type/category/expedited combinations produce correct pence amounts
- [ ] Exempt applicants always pay 0 regardless of type or mode
- [ ] Amount in pounds is exact Decimal (not float)
- [ ] Unknown document type raises FeeCalculationError with valid types listed
- [ ] Unknown category raises FeeCalculationError with valid categories listed
- [ ] API endpoint returns validation errors for invalid/missing input
- [ ] Fee calculation is stateless and repeatable

## Test File

`tests/fees/test_bench_002_fee_calculator.py`
