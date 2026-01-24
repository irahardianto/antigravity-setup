---
trigger: always_on
---

## Logging and Observability Mandate

### Universal Requirement: All Operations Must Be Logged

**Every operation entry point MUST include logging. No exceptions.**

**What constitutes an "operation" (mandatory logging):**
- API endpoints and request handlers
- Background jobs and queue workers
- Event handlers and message consumers
- Scheduled tasks and cron jobs
- CLI commands
- External service calls (to third-party APIs)
- Database transactions

**What is NOT an operation (no direct logging):**
- Pure business logic functions (called within operations)
- Utility and helper functions
- Data transformations and validators

**Minimum logging requirement (3 points):**
1. **Operation start:** Log at entry with context (correlationId, userId, operation name)
2. **Operation success:** Log completion with duration and result identifiers
3. **Operation failure:** Log error with full context (correlationId, error details, stack trace)

**Mandatory context in all logs:**
- `correlationId`: UUID for tracing across services
- `operation`: Clear operation name (e.g., "create_order", "process_payment")
- `duration`: Execution time in milliseconds
- `userId`: Actor who triggered the operation (when applicable)
- `error`: Full error context on failures

**Enforcement strategy:**
Prefer middleware, decorators, or framework interceptors for automatic operation logging rather than manual logging in every handler. This ensures coverage without code duplication.

**When implementing any operation entry point, you MUST add logging before proceeding with implementation.**

### Implementation Guide

For detailed implementation patterns, see:
- **[Logging and Observability Principles](#logging-and-observability-principles)** - Complete implementation guide including:
  - Log levels and when to use them
  - Structured logging patterns
  - Language-specific implementations (Go, TypeScript, Python)
  - Security considerations (what never to log)
  - Performance best practices
  - Log patterns by operation type
  - Testing and monitoring integration

### Related Principles
- [Logging and Observability Principles (Implementation)](#logging-and-observability-principles) - Detailed implementation guide
- [Error Handling Principles](#error-handling-principles) - Error context requirements
- [API Design Principles](#api-design-principles) - API-specific observability patterns
