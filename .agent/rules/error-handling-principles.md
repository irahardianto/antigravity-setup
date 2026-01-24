---
trigger: model_decision
description: When implementing logging, working with loggers, or setting up observability for operations (API handlers, database queries, background jobs, external API calls)
---

## Error Handling Principles

**1. Never Fail Silently:**

- All errors must be handled explicitly (no empty catch blocks)  
- If you catch an error, do something with it (log, return, transform, retry)

**2. Fail Fast:**

- Detect and report errors as early as possible  
- Validate at system boundaries before processing  
- Don't process invalid data hoping it'll work out

**3. Provide Context:**

- Include error codes, correlation IDs, actionable messages  
- Enough information for debugging without exposing sensitive details  
- Example: "Database query failed (correlation-id: abc-123)" not "SELECT * FROM users WHERE..."

**4. Separate Concerns:**

- Different handlers for different error types  
- Business errors ≠ technical errors ≠ security errors

**5. Resource Cleanup:**

- Always clean up in error scenarios (close files, release connections, unlock resources)  
- Use language-appropriate patterns (defer, finally, RAII, context managers)

**6. No Information Leakage:**

- Sanitize error messages for external consumption  
- Don't expose stack traces, SQL queries, file paths, internal structure to users  
- Log full details internally, show generic message externally

### Application Error Object (Internal/Log Format)
```
{
  "status": "error",
  "code": "VALIDATION_ERROR",
  "message": "User-friendly error message",
  "correlationId": "uuid-for-tracking",
  "details": {
    "field": "email",
    "issue": "Invalid email format",
    "provided": "invalid-email",
    "expected": "valid email format (user@example.com)"
  }
}
```

### Error Handling Checklist

- [ ] Are all error paths explicitly handled (no empty catch blocks)?  
- [ ] Do errors include correlation IDs for debugging?  
- [ ] Are sensitive details sanitized before returning to client?  
- [ ] Are resources cleaned up in all error scenarios?  
- [ ] Are errors logged at appropriate levels (warn for 4xx, error for 5xx)?  
- [ ] Are error tests written (negative test cases)?  
- [ ] Is error handling consistent across application?

### Related Principles
- [API Design: Error Response Format](#api-design-principles) - JSON envelope structure
- [Logging: Correlation IDs](#logging-and-observability-principles) - Traceability
- [Security: No Information Leakage](#security-principles) - Sanitization
- [Testing: Negative Test Cases](#testing-strategy) - Error path coverage
- [Concurrency: Error in Thread Context](#concurrency-and-threading-principles) - Thread failures
