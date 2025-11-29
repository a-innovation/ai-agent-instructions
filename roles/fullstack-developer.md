# FULLSTACK DEVELOPER AI - AGENT INSTRUCTIONS v3.1

> Extends: EXPERT_DEVELOPER_AI_CORE_INSTRUCTIONS.md
> Combines: Frontend + Backend guidance
> Focus: End-to-end feature development with type consistency

---

## 🎭 FULLSTACK THINKING MODES

| Mode | Fullstack Focus |
|------|-----------------|
| **🏗️ Architect** | API contracts, type sharing, data flow, monorepo structure |
| **🛡️ Sentry** | Consistent validation (client+server), auth flow, error propagation |
| **🤖 Automator** | E2E tests, contract tests, integration verification |

---

## 🎯 FULLSTACK PRINCIPLES

| Principle | Description |
|-----------|-------------|
| **Vertical Slicing** | Implement features end-to-end, not layer-by-layer |
| **Contract-First** | Define API types before implementation |
| **Type Sharing** | Single source of truth for types |
| **Consistent Validation** | Same rules on client and server |
| **Unified Errors** | Consistent error format across stack |

---

## 📁 MONOREPO STRUCTURE

```
project/
├── apps/
│   ├── web/                   # Frontend
│   └── api/                   # Backend
│
├── packages/
│   ├── shared-types/          # API types (single source of truth)
│   ├── shared-utils/          # Validation, formatting
│   ├── shared-constants/      # Error codes, config
│   └── ui/                    # Component library
│
├── package.json
└── turbo.json
```

---

## 🔗 TYPE SHARING

### Shared Types Package

```typescript
// packages/shared-types/src/api.types.ts

// Response envelope - used by both apps
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: ApiError;
  meta?: PaginationMeta;
}

export interface ApiError {
  code: string;
  message: string;
  details?: Record<string, unknown>;
}

// Domain types
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
}

export type UserRole = 'admin' | 'user' | 'guest';

// Request/Response types
export interface CreateUserRequest {
  email: string;
  password: string;
  name: string;
}
```

### Shared Validation

```typescript
// packages/shared-utils/src/validation. ts

// Same validation runs on frontend AND backend
export const ValidationRules = {
  email: {
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    message: 'Invalid email format',
  },
  password: {
    minLength: 8,
    maxLength: 100,
  },
} as const;

export function validateEmail(email: string): boolean {
  return ValidationRules.email. pattern.test(email);
}

export function validatePassword(password: string): { valid: boolean; error?: string } {
  if (password.length < ValidationRules.password. minLength) {
    return { valid: false, error: 'Password too short' };
  }
  return { valid: true };
}
```

---

## 🔄 FEATURE DEVELOPMENT WORKFLOW

### Vertical Slice Approach

```
Feature: User Registration

1.  TYPES (packages/shared-types)
   └── Define CreateUserRequest, User, ApiResponse<User>

2.  VALIDATION (packages/shared-utils)
   └── Create/reuse validateEmail, validatePassword

3. BACKEND (apps/api)
   ├── DTO using shared types
   ├── Controller → Service → Repository
   └── Tests

4. FRONTEND (apps/web)
   ├── API hook using shared types
   ├── Form with shared validation
   └── Tests

5. E2E TEST
   └── Full flow verification
```

### API Integration

```typescript
// apps/web/src/services/api.ts
import type { ApiResponse, User, CreateUserRequest } from '@project/shared-types';

class ApiService {
  async createUser(data: CreateUserRequest): Promise<ApiResponse<User>> {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    return response.json();
  }
}

// apps/web/src/hooks/useCreateUser. ts
export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: api.createUser,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['users'] }),
  });
}
```

---

## 🔐 CONSISTENT ERROR HANDLING

### Shared Error Codes

```typescript
// packages/shared-constants/src/errors.ts

export const ErrorCodes = {
  VALIDATION_ERROR: 'VALIDATION_ERROR',
  UNAUTHORIZED: 'UNAUTHORIZED',
  NOT_FOUND: 'NOT_FOUND',
  CONFLICT: 'CONFLICT',
  INTERNAL_ERROR: 'INTERNAL_ERROR',
} as const;

// User-friendly messages for frontend
export const ErrorMessages: Record<string, string> = {
  [ErrorCodes.VALIDATION_ERROR]: 'Please check your input.',
  [ErrorCodes.UNAUTHORIZED]: 'Please sign in to continue.',
  [ErrorCodes.NOT_FOUND]: 'Resource not found.',
};
```

### Frontend Error Handler

```typescript
// apps/web/src/utils/error-handler.ts
import { ErrorCodes, ErrorMessages } from '@project/shared-constants';
import type { ApiError } from '@project/shared-types';

export function handleApiError(error: ApiError): void {
  const message = ErrorMessages[error.code] ??  error.message;
  
  if (error.code === ErrorCodes.UNAUTHORIZED) {
    redirectToLogin();
    return;
  }
  
  showToast({ type: 'error', message });
}
```

---

## 🧪 FULLSTACK VERIFICATION PLAN

### End-to-End Feature Checklist

```markdown
### API Layer
- [ ] Types defined in shared-types
- [ ] Validation uses shared-utils
- [ ] Response matches type contract
- [ ] Errors use shared error codes
- [ ] Unit + integration tests pass

### Frontend Layer
- [ ] Uses shared types
- [ ] Validation matches backend
- [ ] Error handling uses shared codes
- [ ] Loading/error states handled
- [ ] Component tests pass

### Integration
- [ ] E2E test covers happy path
- [ ] E2E test covers error scenarios
- [ ] Types match across stack
- [ ] Validation consistent
```

### E2E Test Template

```typescript
describe('User Registration', () => {
  it('completes registration flow', async ({ page }) => {
    await page.goto('/register');
    await page.fill('[name="email"]', 'test@example. com');
    await page.fill('[name="password"]', 'securePassword123');
    await page.fill('[name="name"]', 'Test User');
    await page.click('button[type="submit"]');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Welcome, Test User')).toBeVisible();
  });

  it('shows validation error for invalid email', async ({ page }) => {
    await page.goto('/register');
    await page.fill('[name="email"]', 'invalid-email');
    await page.click('button[type="submit"]');
    
    await expect(page. getByText('Invalid email format')).toBeVisible();
  });
});
```

---

## 🚫 FULLSTACK ANTI-PATTERNS

| Avoid | Instead |
|-------|---------|
| Duplicating types in both apps | Shared types package |
| Different validation rules | Shared validation utils |
| Inconsistent error formats | Shared error codes |
| Testing layers in isolation only | E2E tests for critical paths |
| Hardcoded API URLs | Environment configuration |