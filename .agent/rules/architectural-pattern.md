---
trigger: always_on
---

## Architectural Patterns - Testability-First Design

### Core Principle
All code must be independently testable without running the full application or external infrastructure.

### Universal Architecture Rules

#### Rule 1: I/O Isolation
**Problem:** Tightly coupled I/O makes tests slow, flaky, and environment-dependent.

**Solution:** Abstract all I/O behind interfaces/contracts:
- Database queries
- HTTP calls (to external APIs)
- File system operations
- Time/randomness (for determinism)
- Message queues

**Implementation Discovery:**
1. Search for existing abstraction patterns: `find_symbol("Interface")`, `find_symbol("Mock")`, `find_symbol("Repository")`
2. Match the style (interface in Go, Protocol in Python, interface in TypeScript)
3. Implement production adapter AND test adapter

**Example (Go):**

```Go

// Contract
type UserStore interface {
  Create(ctx context.Context, user User) error
  GetByEmail(ctx context.Context, email string) (*User, error)
}

// Production adapter
type PostgresUserStore struct { /* ... */ }

// Test adapter
type MockUserStore struct { /* ... */ }
```

**Example (TypeScript/Vue):**
```typescript

// Contract (service layer)
export interface TaskAPI {
  createTask(title: string): Promise<Task>;
  getTasks(): Promise<Task[]>;
}

// Production adapter
export class BackendTaskAPI implements TaskAPI { /* ... */ }

// Test adapter (vi.mock or manual)
export class MockTaskAPI implements TaskAPI { /* ... */ }

```

#### Rule 2: Pure Business Logic
**Problem:** Business rules mixed with I/O are impossible to test without infrastructure.

**Solution:** Extract calculations, validations, transformations into pure functions:
- Input → Output, no side effects
- Deterministic: same input = same output
- No I/O inside business rules

**Examples:**
```

// ✅ Pure function - easy to test
func calculateDiscount(items []Item, coupon Coupon) (float64, error) {
// Pure calculation, returns value
}

// ❌ Impure - database call inside
func calculateDiscount(ctx context.Context, items []Item, coupon Coupon) (float64, error) {
validCoupon, err := db.GetCoupon(ctx, coupon.ID) // NO!
}

```

**Correct approach:**
```

// 1. Fetch dependencies first (in handler/service)
validCoupon, err := store.GetCoupon(ctx, coupon.ID)

// 2. Pass to pure logic
discount, err := calculateDiscount(items, validCoupon)

// 3. Persist result
err = store.SaveOrder(ctx, order)

```

#### Rule 3: Module Boundaries
**Problem:** Cross-module coupling makes changes ripple across codebase.

**Solution:** Feature-based organization with clear public interfaces:
- One feature = one directory
- Each module exposes a public API (exported functions/classes)
- Internal implementation details are private
- Cross-module calls only through public API

**Directory Structure (Language-Agnostic):**
```

/task

- public_api.{ext}      # Exported interface
- business.{ext}        # Pure logic
- store.{ext}           # I/O abstraction (interface)
- postgres.{ext}        # I/O implementation
- mock.{ext}            # Test implementation
- test.{ext}            # Unit tests (mocked I/O)
- integration.test.{ext} # Integration tests (real I/O)

```

**Go Example:**
```

/apps/backend/task

- task.go               # API endpoints (public)
- business.go           # Pure domain logic
- store.go              # interface UserStore
- postgres.go           # implements UserStore
- task_test.go          # Unit tests with MockStore
- task_integration_test.go # Integration with real DB

```

**Vue Example:**
```

/apps/frontend/src/features/task

- index.ts              # Public exports
- task.service.ts       # Business logic
- task.api.ts           # interface TaskAPI
- task.api.backend.ts    # implements TaskAPI
- task.store.ts         # Pinia store (uses TaskAPI)
- task.service.spec.ts  # Unit tests (mock API)

```

#### Rule 4: Dependency Direction
**Principle:** Dependencies point inward toward business logic.

```

┌─────────────────────────────────────┐
│  Infrastructure Layer               │
│  (DB, HTTP, Files, External APIs)   │
│                                     │
│  Depends on ↓                       │
└─────────────────────────────────────┘
↓
┌─────────────────────────────────────┐
│  Contracts/Interfaces Layer         │
│  (Abstract ports - no implementation)│
│                                     │
│  Depends on ↓                       │
└─────────────────────────────────────┘
↓
┌─────────────────────────────────────┐
│  Business Logic Layer               │
│  (Pure functions, domain rules)     │
│  NO dependencies on infrastructure  │
└─────────────────────────────────────┘

```

**Never:**
- Business logic imports database driver
- Domain entities import HTTP framework
- Core calculations import config files

**Always:**
- Infrastructure implements interfaces defined by business layer
- Business logic receives dependencies via injection

**Package Structure Philosophy:**

- **Organize by FEATURE, not by technical layer**  
- Each feature is a vertical slice
- Enables modular growth, clear boundaries, and independent deployability  

**Universal Rule: Context → Feature → Layer**

**1. Level 1: Repository Scope (Conditional)**
   - **Scenario A (Monorepo/Full-Stack):** Root contains `apps/` grouping distinct applications (e.g., `apps/backend`, `apps/web`).
   - **Scenario B (Single Service):** Root **IS** the application. Do not create `apps/backend` wrapper. Start directly at Level 2.

**2. Level 2: Feature Organization**
   - **Rule:** Divide application into vertical business slices (e.g., `user/`, `order/`, `payment/`).
   - **Anti-Pattern:** Do NOT organize by technical layer (e.g., `controllers/`, `models/`, `services/`) at the top level.

#### Layout Examples

**A. Standard Single Service (Backend, Microservice or MVC)**
```
  apps/  
    task/                       # Feature: Task management    
      task.go                      # API handlers (public interface)
      task_test.go                 # Unit tests (mocked dependencies)
      business.go                  # Pure business logic
      business_test.go             # Unit tests (pure functions)
      store.go                     # interface TaskStore
      postgres.go                  # implements TaskStore
      postgres_integration_test.go # Integration tests (real DB)
      mock_store.go               # Test implementation
    migrations/
      001_create_tasks.up.sql
    order/                      # Feature: Order management  
      ...
```

**B. Monorepo Layout (Multi-Stack):**
**Use this structure when managing monolithic full-stack applications with backend, frontend, mobile in a single repository.*
*Clear Boundaries: Backend business logic is isolated from Frontend UI logic, even if they share the same repo*
```    
  apps/
    backend/                        # Backend application source code  
        task/                       # Feature: Task management  
          task.go                   # API handlers (public interface)
          ...   
        order/                      # Feature: Order management  
        ...
    frontend/                       # Frontend application source code
      assets/                       # Fonts, Images
      components/                   # Shared Component (Buttons, Inputs) - Dumb UI, No Domain Logic
        BaseButton.vue
        BaseInput.vue
      layouts/                      # App shells (Sidebar, Navbar wrappers)
      utils/                        # Date formatting, validation helpers
      features/                     # Business Features (Vertical Slices)
        task/                       # Feature: Task management
          TaskForm.vue              # Feature-specific components
          TaskListItem.vue          
          TaskFilters.vue           
          index.ts                  # Public exports
          task.service.ts           # Business logic
          task.api.ts               # interface TaskAPI
          task.api.backend.ts        # Production implementation
          task.store.ts             # Pinia store
        order/
      ...
```
> This Feature/Domain/UI/API structure is framework-agnostic. It applies equally to React, Vue, Svelte, and Mobile (React Native/Flutter). 'UI' always refers to the framework's native component format (.tsx, .vue, .svelte, .dart).

### Pattern Discovery Protocol

**Before implementing ANY feature:**

1. **Search existing patterns** (MANDATORY):
```

find_symbol("Interface") OR find_symbol("Repository") OR find_symbol("Service")

```

2. **Examine 3 existing modules** for consistency:
- How do they handle database access?
- Where are pure functions vs I/O operations?
- What testing patterns exist?

3. **Document pattern** (80%+ consistency required):
- "Following pattern from [task, user, auth] modules"
- "X/Y modules use interface-based stores"
- "All tests use [MockStore, vi.mock, TestingPinia] pattern"

4. **If consistency <80%**: STOP and report fragmentation to human.

### Testing Requirements

**Unit Tests (must run without infrastructure):**
- Mock all I/O dependencies
- Test business logic in isolation
- Fast (<100ms per test)
- 85%+ coverage of business paths

**Integration Tests (must test real infrastructure):**
- Use real database (Testcontainers, Firebase emulator)
- Test adapter implementations
- Verify contracts work end-to-end
- Cover all I/O adapters

**Test Organization:**
- Unit/Integration tests: Co-located with implementation
- E2E tests: Separate `/e2e` directory

### Language-Specific Idioms

**How to achieve testability in each ecosystem:**

| Language/Framework | Abstraction Pattern | Test Strategy |
|-------------------|---------------------|---------------|
| **Go** | Interface types, dependency injection | Table-driven tests, mock implementations |
| **TypeScript/Vue** | Interface types, service layer, Pinia stores | Vitest with `vi.mock`, `createTestingPinia` |
| **TypeScript/React** | Interface types, service layer, Context/hooks | Jest with mock factories, React Testing Library |
| **Python** | `typing.Protocol` or abstract base classes | pytest with fixtures, monkeypatch |
| **Rust** | Traits, dependency injection | Unit tests with mock implementations, `#[cfg(test)]` |
| **Flutter/Dart** | Abstract classes, dependency injection | `mockito` package, widget tests |

### Enforcement Checklist

Before marking code complete, verify:
- [ ] Can I run unit tests without starting database/external services?
- [ ] Are all I/O operations behind an abstraction?
- [ ] Is business logic pure (no side effects)?
- [ ] Do integration tests exist for all adapters?
- [ ] Does pattern match existing codebase (80%+ consistency)?

### Related Principles
- [SOLID: Dependency Inversion](#dependency-inversion-principle-dip) - Ports as abstractions
- [Testing: Mock Ports Strategy](#test-doubles-strategy) - Unit test isolation
- [Code Organization: Feature Packaging](#code-organization-principles) - Vertical slices
- [Dependency Management: Avoid Circular](#avoid-circular-dependencies) - Layer separation