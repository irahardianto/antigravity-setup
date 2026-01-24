---
trigger: always_on
---

## Concurrency and Threading Mandate

### When to Use Concurrency

**I/O-Bound Operations (async/await, event loops):**

- Network requests, file I/O, database queries  
- Waiting for external responses dominates execution time  
- Use: Asynchronous I/O, event-driven concurrency, coroutines

**CPU-Bound Operations (threads, parallel processing):**

- Heavy computation, data processing, video encoding  
- CPU cycles dominate execution time  
- Use: OS threads, thread pools, parallel workers

**Don't Over-Use Concurrency:**

- Adds significant complexity (race conditions, deadlocks, debugging difficulty)  
- Use only when there's measurable performance benefit  
- Profile first, optimize second

### When NOT to Use Concurrency
- Simple synchronous operations
- No measurable performance benefit
- Avoid premature optimization