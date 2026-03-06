---
id: TASK-FBP-002
title: "Pydantic settings with validation"
task_type: feature
parent_review: TASK-REV-01B0
feature_id: FEAT-FBP
status: pending
wave: 2
implementation_mode: task-work
complexity: 4
dependencies:
  - TASK-FBP-001
priority: high
tags: [configuration, pydantic, validation]
---

# Task: Pydantic settings with validation

## Description

Implement the Pydantic BaseSettings class for application configuration with strict validation of environment, log_level, log_format, version, and api_prefix. Settings must reject invalid values at startup (ValidationError) and support .env file loading.

## Acceptance Criteria

- [ ] `src/core/config.py` contains `Settings(BaseSettings)` class
- [ ] `environment` field validates against Literal["development", "staging", "production"] with default "development"
- [ ] `log_level` field validates against Literal["DEBUG", "INFO", "WARNING", "ERROR"] with default "INFO"
- [ ] `log_format` field validates against Literal["json", "text"] with default "json"
- [ ] `app_version` field with default "0.1.0"
- [ ] `api_prefix` field with default "/v1"
- [ ] `app_name` field with default "FastAPI App"
- [ ] Invalid environment value raises ValidationError at construction time
- [ ] Invalid log_level value raises ValidationError at construction time
- [ ] Invalid log_format value raises ValidationError at construction time
- [ ] Settings loads from environment variables (env_prefix not needed, direct field names)
- [ ] Settings supports .env file loading via `model_config = SettingsConfigDict(env_file=".env")`

## BDD Scenarios Covered

- Boundary: Health endpoint reflects each valid environment setting (development, staging, production)
- Boundary: Application starts successfully with each valid log level (DEBUG, INFO, WARNING, ERROR)
- Negative: An invalid environment value is rejected
- Negative: An invalid log level is rejected
- Negative: An invalid log format is rejected
- Edge: Health endpoint reflects a custom application version
- Key-example: The application is created with custom settings

## Implementation Notes

- Use `pydantic_settings.BaseSettings` (Pydantic v2 style)
- Use `Literal` types for enum-like validation (not custom validators)
- Use `SettingsConfigDict` instead of inner `Config` class for Pydantic v2
- Case-insensitive env var matching via `case_sensitive = False` in model_config
