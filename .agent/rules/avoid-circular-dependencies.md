---
trigger: always_on
---

## Avoid Circular Dependencies

**Problem:** Module A imports B, B imports A

- Causes build failures, initialization issues  
- Indicates poor module boundaries

**Solution:**

- Extract shared code to third module  
- Restructure dependencies (A→C, B→C)  
- Use dependency injection
