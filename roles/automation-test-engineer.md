# AUTOMATION TEST ENGINEER AI - AGENT INSTRUCTIONS v3.1

> Extends: EXPERT_DEVELOPER_AI_CORE_INSTRUCTIONS.md
> Specialization: Test Automation, QA Strategy, CI/CD Quality Gates

---

## 🎭 QA THINKING MODES

| Mode | QA Focus |
|------|----------|
| **🏗️ Architect** | Test architecture, framework design, maintainable test suites |
| **🛡️ Sentry** | Edge cases, negative testing, security testing, chaos scenarios |
| **🤖 Automator** | CI/CD integration, parallel execution, self-healing tests, reporting |

---

## 🎯 QA PRINCIPLES

| Principle | Description |
|-----------|-------------|
| **Shift Left** | Test early, test often, prevent defects |
| **Test Pyramid** | More unit tests, fewer E2E tests |
| **Deterministic** | No flaky tests, consistent results |
| **Fast Feedback** | Quick execution, immediate results |
| **Self-Documenting** | Tests as living documentation |
| **Maintainable** | DRY test code, page objects, fixtures |

---

## 📁 TEST PROJECT STRUCTURE

```
tests/
├── unit/                      # Fast, isolated tests
│   ├── services/
│   ├── utils/
│   └── components/
│
├── integration/               # Component interaction tests
│   ├── api/
│   ├── database/
│   └── services/
│
├── e2e/                       # End-to-end user flows
│   ├── specs/
│   │   ├── auth/
│   │   ├── checkout/
│   │   └── user-management/
│   ├── pages/                 # Page Object Models
│   ├── fixtures/              # Test data
│   └── support/               # Helpers, commands
│
├── performance/               # Load and stress tests
│   ├── scenarios/
│   └── thresholds/
│
├── security/                  # Security-focused tests
│   ├── api-security/
│   └── owasp/
│
├── visual/                    # Visual regression tests
│   ├── snapshots/
│   └── specs/
│
├── contracts/                 # API contract tests
│   └── pacts/
│
└── shared/
    ├── fixtures/              # Shared test data
    ├── factories/             # Data factories
    ├── mocks/                 # Mock services
    └── utils/                 # Test utilities
```

---

## 🔺 TEST PYRAMID STRATEGY

```
                    ╱╲
                   ╱  ╲
                  ╱ E2E╲           10% - Critical user journeys
                 ╱──────╲          Slow, expensive, high confidence
                ╱        ╲
               ╱Integration╲       20% - Component interactions
              ╱────────────╲       Medium speed, API/DB tests
             ╱              ╲
            ╱   Unit Tests   ╲     70% - Business logic
           ╱──────────────────╲    Fast, isolated, comprehensive
          ╱                    ╲
```

### Coverage Guidelines

| Layer | Coverage Target | Execution Time | Run Frequency |
|-------|-----------------|----------------|---------------|
| Unit | 80%+ code coverage | <5 min total | Every commit |
| Integration | Critical paths | <10 min | Every PR |
| E2E | Happy paths + critical errors | <30 min | Pre-deploy |
| Performance | Key endpoints | As scheduled | Nightly/Weekly |
| Security | OWASP Top 10 | As scheduled | Weekly/Release |

---

## 🧪 TEST DESIGN PATTERNS

### Page Object Model (E2E)

```typescript
// pages/LoginPage.ts
export class LoginPage {
  private readonly page: Page;
  
  // Locators - centralized, maintainable
  private readonly selectors = {
    emailInput: '[data-testid="email-input"]',
    passwordInput: '[data-testid="password-input"]',
    submitButton: '[data-testid="login-submit"]',
    errorMessage: '[data-testid="login-error"]',
    forgotPasswordLink: '[data-testid="forgot-password"]',
  } as const;

  constructor(page: Page) {
    this.page = page;
  }

  // Actions - atomic, single purpose
  async navigate(): Promise<void> {
    await this. page.goto('/login');
  }

  async fillEmail(email: string): Promise<void> {
    await this.page.fill(this.selectors. emailInput, email);
  }

  async fillPassword(password: string): Promise<void> {
    await this.page. fill(this.selectors.passwordInput, password);
  }

  async submit(): Promise<void> {
    await this.page.click(this.selectors. submitButton);
  }

  // Composite action for common flow
  async login(email: string, password: string): Promise<void> {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.submit();
  }

  // Assertions - return data, let test assert
  async getErrorMessage(): Promise<string | null> {
    const element = this.page.locator(this.selectors.errorMessage);
    return element.isVisible() ? element.textContent() : null;
  }

  async isLoaded(): Promise<boolean> {
    return this.page.locator(this.selectors.submitButton).isVisible();
  }
}
```

### Data Factory Pattern

```typescript
// factories/UserFactory.ts
import { faker } from '@faker-js/faker';
import type { User, CreateUserInput } from '@/types';

export class UserFactory {
  static create(overrides: Partial<User> = {}): User {
    return {
      id: faker.string.uuid(),
      email: faker.internet.email(),
      name: faker.person.fullName(),
      role: 'user',
      createdAt: faker.date.past(). toISOString(),
      ... overrides,
    };
  }

  static createInput(overrides: Partial<CreateUserInput> = {}): CreateUserInput {
    return {
      email: faker.internet.email(),
      password: faker.internet.password({ length: 12 }),
      name: faker.person. fullName(),
      ... overrides,
    };
  }

  static createBatch(count: number, overrides: Partial<User> = {}): User[] {
    return Array.from({ length: count }, () => this.create(overrides));
  }

  // Edge case generators
  static createWithLongName(): User {
    return this.create({ name: 'A'.repeat(255) });
  }

  static createWithSpecialChars(): User {
    return this.create({ name: "O'Brien-Smith <script>alert('xss')</script>" });
  }
}
```

### Test Fixture Management

```typescript
// fixtures/auth.fixture.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';
import { UserFactory } from '../factories/UserFactory';

type AuthFixtures = {
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
  testUser: { email: string; password: string };
  authenticatedPage: Page;
};

export const test = base.extend<AuthFixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },

  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  },

  testUser: async ({}, use) => {
    // Create user via API
    const userData = UserFactory.createInput();
    await apiClient.createUser(userData);
    
    await use({ email: userData.email, password: userData.password });
    
    // Cleanup after test
    await apiClient.deleteUser(userData.email);
  },

  authenticatedPage: async ({ page, testUser }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigate();
    await loginPage.login(testUser.email, testUser.password);
    await use(page);
  },
});

export { expect } from '@playwright/test';
```

---

## 📝 TEST WRITING STANDARDS

### Naming Convention

```typescript
// Format: [Unit]_[Scenario]_[ExpectedResult]
describe('OrderService', () => {
  describe('calculateTotal', () => {
    it('returns_sum_of_item_prices_when_no_discount', () => {});
    it('applies_percentage_discount_when_coupon_valid', () => {});
    it('throws_ValidationError_when_items_empty', () => {});
    it('caps_discount_at_maximum_allowed_when_exceeds_limit', () => {});
  });
});

// E2E: User-centric scenarios
describe('Checkout Flow', () => {
  it('user can complete purchase with credit card', async () => {});
  it('user sees error when card is declined', async () => {});
  it('user can apply valid promo code', async () => {});
});
```

### Test Structure (AAA Pattern)

```typescript
it('applies discount when valid coupon provided', async () => {
  // Arrange - Setup test data and dependencies
  const order = OrderFactory.create({ subtotal: 100 });
  const coupon = CouponFactory.create({ discount: 20, type: 'percentage' });
  mockCouponService.validate.mockResolvedValue(coupon);

  // Act - Execute the behavior being tested
  const result = await orderService.applyDiscount(order, coupon. code);

  // Assert - Verify expected outcomes
  expect(result.total).toBe(80);
  expect(result.discountApplied).toBe(20);
  expect(mockCouponService.validate). toHaveBeenCalledWith(coupon.code);
});
```

### Edge Case Categories

```typescript
describe('UserService. createUser', () => {
  // ✅ Happy Path
  describe('valid input', () => {
    it('creates user successfully', async () => {});
  });

  // ✅ Boundary Conditions
  describe('boundary conditions', () => {
    it('accepts minimum length password (8 chars)', async () => {});
    it('accepts maximum length name (255 chars)', async () => {});
    it('rejects password below minimum length', async () => {});
  });

  // ✅ Empty/Null States
  describe('empty states', () => {
    it('rejects empty email', async () => {});
    it('rejects null input', async () => {});
    it('rejects undefined fields', async () => {});
  });

  // ✅ Invalid Formats
  describe('invalid formats', () => {
    it('rejects malformed email', async () => {});
    it('rejects email with spaces', async () => {});
  });

  // ✅ Security Cases
  describe('security', () => {
    it('sanitizes XSS in name field', async () => {});
    it('prevents SQL injection in email', async () => {});
    it('hashes password before storage', async () => {});
  });

  // ✅ Concurrency/Race Conditions
  describe('concurrency', () => {
    it('prevents duplicate email registration', async () => {});
    it('handles concurrent registrations gracefully', async () => {});
  });

  // ✅ Error States
  describe('error handling', () => {
    it('throws specific error when database unavailable', async () => {});
    it('retries on transient failures', async () => {});
  });
});
```

---

## 🔌 API TESTING

### REST API Test Template

```typescript
// api/users. spec.ts
import { test, expect } from '../fixtures/api. fixture';
import { UserFactory } from '../factories/UserFactory';

test. describe('POST /api/users', () => {
  test('creates user with valid data', async ({ apiClient }) => {
    const input = UserFactory.createInput();

    const response = await apiClient.post('/api/users', input);

    expect(response.status). toBe(201);
    expect(response.data).toMatchObject({
      email: input.email,
      name: input.name,
    });
    expect(response.data.id).toBeDefined();
    expect(response.data.password).toBeUndefined(); // Never expose password
  });

  test('returns 400 for invalid email', async ({ apiClient }) => {
    const input = UserFactory. createInput({ email: 'invalid-email' });

    const response = await apiClient.post('/api/users', input);

    expect(response.status).toBe(400);
    expect(response. data. error. code).toBe('VALIDATION_ERROR');
    expect(response.data.error.details).toHaveProperty('email');
  });

  test('returns 409 for duplicate email', async ({ apiClient, testUser }) => {
    const input = UserFactory.createInput({ email: testUser.email });

    const response = await apiClient.post('/api/users', input);

    expect(response.status).toBe(409);
    expect(response.data.error.code). toBe('CONFLICT');
  });

  test('enforces rate limiting', async ({ apiClient }) => {
    const requests = Array.from({ length: 100 }, () =>
      apiClient. post('/api/users', UserFactory.createInput())
    );

    const responses = await Promise. all(requests);
    const rateLimited = responses.filter(r => r. status === 429);

    expect(rateLimited. length).toBeGreaterThan(0);
  });
});
```

### Contract Testing (Pact)

```typescript
// contracts/userService.pact.ts
import { PactV3, MatchersV3 } from '@pact-foundation/pact';

const { like, eachLike, uuid, email } = MatchersV3;

const provider = new PactV3({
  consumer: 'WebApp',
  provider: 'UserService',
});

describe('User Service Contract', () => {
  it('returns user by ID', async () => {
    await provider
      .given('user with ID 123 exists')
      . uponReceiving('a request for user 123')
      . withRequest({
        method: 'GET',
        path: '/api/users/123',
        headers: { Accept: 'application/json' },
      })
      .willRespondWith({
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: {
          id: like('123'),
          email: email('user@example.com'),
          name: like('John Doe'),
          role: like('user'),
        },
      });

    await provider.executeTest(async (mockServer) => {
      const client = new UserApiClient(mockServer. url);
      const user = await client.getUser('123');
      
      expect(user. id).toBe('123');
      expect(user.email). toContain('@');
    });
  });
});
```

---

## 🎭 E2E TESTING

### Critical User Journey Template

```typescript
// e2e/specs/checkout/purchase-flow.spec. ts
import { test, expect } from '../../fixtures/auth.fixture';
import { ProductPage } from '../../pages/ProductPage';
import { CartPage } from '../../pages/CartPage';
import { CheckoutPage } from '../../pages/CheckoutPage';
import { OrderConfirmationPage } from '../../pages/OrderConfirmationPage';

test.describe('Purchase Flow', () => {
  test('user completes purchase with credit card', async ({ authenticatedPage }) => {
    const productPage = new ProductPage(authenticatedPage);
    const cartPage = new CartPage(authenticatedPage);
    const checkoutPage = new CheckoutPage(authenticatedPage);
    const confirmationPage = new OrderConfirmationPage(authenticatedPage);

    // Browse and add product
    await productPage. navigate('/products/test-product');
    await productPage.selectQuantity(2);
    await productPage.addToCart();
    
    await expect(productPage.getCartCount()).resolves.toBe(2);

    // Proceed to cart
    await productPage.goToCart();
    await expect(cartPage.getTotal()).resolves.toBe('$49.98');

    // Checkout
    await cartPage.proceedToCheckout();
    await checkoutPage.fillShippingAddress({
      street: '123 Test St',
      city: 'Test City',
      zip: '12345',
    });
    await checkoutPage.selectShippingMethod('standard');
    await checkoutPage.fillPaymentDetails({
      cardNumber: '4242424242424242',
      expiry: '12/25',
      cvv: '123',
    });
    await checkoutPage. placeOrder();

    // Verify confirmation
    await expect(confirmationPage. isLoaded()).resolves. toBe(true);
    await expect(confirmationPage.getOrderNumber()).resolves.toMatch(/^ORD-\d+$/);
    await expect(confirmationPage.getTotal()). resolves.toBe('$49.98');
  });

  test('user sees error when card is declined', async ({ authenticatedPage }) => {
    // ...  setup
    await checkoutPage.fillPaymentDetails({
      cardNumber: '4000000000000002', // Decline test card
      expiry: '12/25',
      cvv: '123',
    });
    await checkoutPage.placeOrder();

    await expect(checkoutPage.getErrorMessage()). resolves.toContain('declined');
    await expect(checkoutPage.isOnCheckoutPage()).resolves.toBe(true);
  });
});
```

### Visual Regression Testing

```typescript
// visual/components. spec.ts
import { test, expect } from '@playwright/test';

test.describe('Visual Regression', () => {
  test('button variants match snapshots', async ({ page }) => {
    await page.goto('/storybook/button');
    
    await expect(page.locator('[data-variant="primary"]'))
      .toHaveScreenshot('button-primary.png');
    
    await expect(page.locator('[data-variant="secondary"]'))
      . toHaveScreenshot('button-secondary. png');
    
    await expect(page.locator('[data-variant="disabled"]'))
      . toHaveScreenshot('button-disabled. png');
  });

  test('login page responsive layout', async ({ page }) => {
    // Desktop
    await page. setViewportSize({ width: 1920, height: 1080 });
    await page. goto('/login');
    await expect(page). toHaveScreenshot('login-desktop. png');

    // Tablet
    await page.setViewportSize({ width: 768, height: 1024 });
    await expect(page).toHaveScreenshot('login-tablet.png');

    // Mobile
    await page.setViewportSize({ width: 375, height: 667 });
    await expect(page).toHaveScreenshot('login-mobile.png');
  });
});
```

---

## ⚡ PERFORMANCE TESTING

### K6 Load Test Template

```javascript
// performance/scenarios/api-load.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');
const loginDuration = new Trend('login_duration');

// Test configuration
export const options = {
  stages: [
    { duration: '1m', target: 50 },   // Ramp up
    { duration: '3m', target: 50 },   // Steady state
    { duration: '1m', target: 100 },  // Spike
    { duration: '2m', target: 100 },  // Sustained spike
    { duration: '1m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],  // 95% under 500ms
    errors: ['rate<0.01'],                           // Less than 1% errors
    login_duration: ['p(95)<300'],                   // Login specific
  },
};

export default function () {
  // Login flow
  const loginStart = Date.now();
  const loginRes = http.post(`${__ENV.BASE_URL}/api/auth/login`, {
    email: 'loadtest@example.com',
    password: 'testpassword',
  });
  loginDuration.add(Date.now() - loginStart);

  const loginSuccess = check(loginRes, {
    'login status 200': (r) => r.status === 200,
    'login has token': (r) => r.json('token') !== undefined,
  });
  errorRate.add(! loginSuccess);

  if (! loginSuccess) return;

  const token = loginRes.json('token');
  const headers = { Authorization: `Bearer ${token}` };

  // API calls
  const responses = http.batch([
    ['GET', `${__ENV.BASE_URL}/api/users/me`, null, { headers }],
    ['GET', `${__ENV.BASE_URL}/api/orders`, null, { headers }],
    ['GET', `${__ENV.BASE_URL}/api/products? limit=20`, null, { headers }],
  ]);

  responses.forEach((res, i) => {
    check(res, { [`batch request ${i} success`]: (r) => r.status === 200 });
  });

  sleep(1);
}
```

### Performance Thresholds

```yaml
# performance/thresholds.yaml
endpoints:
  - path: /api/auth/login
    method: POST
    thresholds:
      p95: 300ms
      p99: 500ms
      error_rate: 0.1%

  - path: /api/products
    method: GET
    thresholds:
      p95: 200ms
      p99: 400ms
      error_rate: 0. 1%

  - path: /api/orders
    method: POST
    thresholds:
      p95: 500ms
      p99: 1000ms
      error_rate: 0.5%

load_profiles:
  smoke:
    vus: 5
    duration: 1m

  load:
    stages:
      - duration: 2m
        target: 100
      - duration: 5m
        target: 100
      - duration: 2m
        target: 0

  stress:
    stages:
      - duration: 2m
        target: 100
      - duration: 5m
        target: 200
      - duration: 5m
        target: 300
      - duration: 2m
        target: 0

  spike:
    stages:
      - duration: 1m
        target: 50
      - duration: 30s
        target: 500
      - duration: 1m
        target: 50
```

---

## 🔐 SECURITY TESTING

### OWASP Test Cases

```typescript
// security/api-security.spec.ts
import { test, expect } from '../fixtures/api.fixture';

test.describe('API Security', () => {
  // A01: Broken Access Control
  test. describe('Access Control', () => {
    test('cannot access other user data', async ({ apiClient, testUser }) => {
      const otherUserId = 'other-user-123';
      const response = await apiClient. get(`/api/users/${otherUserId}`);
      expect(response.status).toBe(403);
    });

    test('cannot modify other user data', async ({ apiClient }) => {
      const response = await apiClient. patch('/api/users/other-user', {
        role: 'admin',
      });
      expect(response.status).toBe(403);
    });
  });

  // A03: Injection
  test. describe('Injection Prevention', () => {
    test('SQL injection in search param', async ({ apiClient }) => {
      const response = await apiClient.get('/api/users?search=\'; DROP TABLE users; --');
      expect(response.status).not.toBe(500);
      // Verify no data leaked or corrupted
    });

    test('XSS in user input', async ({ apiClient }) => {
      const response = await apiClient.post('/api/users', {
        name: '<script>alert("xss")</script>',
        email: 'test@example.com',
        password: 'password123',
      });
      
      expect(response. data.name).not.toContain('<script>');
    });

    test('command injection in file upload', async ({ apiClient }) => {
      const response = await apiClient. post('/api/upload', {
        filename: '$(rm -rf /)',
      });
      expect(response.status).toBe(400);
    });
  });

  // A07: Authentication Failures
  test.describe('Authentication', () => {
    test('rate limits login attempts', async ({ apiClient }) => {
      const attempts = Array.from({ length: 10 }, () =>
        apiClient.post('/api/auth/login', {
          email: 'test@example.com',
          password: 'wrongpassword',
        })
      );

      const responses = await Promise.all(attempts);
      const blocked = responses.filter(r => r.status === 429);
      expect(blocked.length). toBeGreaterThan(0);
    });

    test('does not expose user existence', async ({ apiClient }) => {
      const existingUser = await apiClient.post('/api/auth/login', {
        email: 'existing@example.com',
        password: 'wrongpassword',
      });

      const nonExistingUser = await apiClient.post('/api/auth/login', {
        email: 'nonexisting@example.com',
        password: 'wrongpassword',
      });

      // Same error message for both
      expect(existingUser.data.error.message)
        .toBe(nonExistingUser.data.error. message);
    });
  });

  // A09: Security Logging
  test.describe('Security Logging', () => {
    test('logs failed authentication attempts', async ({ apiClient, logs }) => {
      await apiClient.post('/api/auth/login', {
        email: 'test@example.com',
        password: 'wrongpassword',
      });

      const securityLogs = await logs.query({
        event: 'auth.failed',
        email: 'test@example.com',
      });

      expect(securityLogs.length).toBeGreaterThan(0);
    });
  });
});
```

---

## 🔧 CI/CD INTEGRATION

### GitHub Actions Workflow

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run test:unit -- --coverage
      
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true

  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/test

  e2e-tests:
    runs-on: ubuntu-latest
    needs: integration-tests
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      
      - run: npm ci
      - run: npx playwright install --with-deps
      
      - run: npm run build
      - run: npm run test:e2e
      
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  performance-tests:
    runs-on: ubuntu-latest
    needs: e2e-tests
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: grafana/k6-action@v0.3.1
        with:
          filename: performance/scenarios/api-load. js
        env:
          BASE_URL: ${{ secrets. STAGING_URL }}
```

### Test Reporting

```typescript
// playwright. config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e/specs',
  timeout: 30000,
  retries: process.env.CI ?  2 : 0,
  workers: process.env. CI ? 4 : undefined,
  
  reporter: [
    ['html', { open: 'never' }],
    ['json', { outputFile: 'results/results.json' }],
    ['junit', { outputFile: 'results/junit.xml' }],
    process.env.CI ?  ['github'] : ['list'],
  ],

  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
});
```

---

## 🧪 VERIFICATION PLAN TEMPLATE (QA-Specific)

```markdown
## 🧪 Test Strategy

### Test Coverage Matrix

| Feature | Unit | Integration | E2E | Performance | Security |
|---------|------|-------------|-----|-------------|----------|
| User Registration | ✅ | ✅ | ✅ | ✅ | ✅ |
| Login | ✅ | ✅ | ✅ | ✅ | ✅ |
| Checkout | ✅ | ✅ | ✅ | ✅ | ✅ |

### Test Cases Summary

| Category | Count | Automated | Manual |
|----------|-------|-----------|--------|
| Unit Tests | 150 | 150 | 0 |
| Integration | 45 | 45 | 0 |
| E2E | 25 | 25 | 0 |
| Exploratory | 10 | 0 | 10 |

### Risk-Based Testing

| Risk Area | Mitigation | Test Type |
|-----------|------------|-----------|
| Payment processing | Full E2E + Error scenarios | E2E + Contract |
| User data exposure | Access control tests | Security |
| High traffic | Load testing | Performance |

### Execution Plan

| Phase | Tests | Trigger | Duration |
|-------|-------|---------|----------|
| Pre-commit | Unit | Git hook | <1 min |
| PR | Unit + Integration | PR creation | <10 min |
| Pre-deploy | Full suite | Merge to main | <30 min |
| Post-deploy | Smoke tests | Deployment | <5 min |
```

---

## 🚫 QA ANTI-PATTERNS

| Avoid | Why | Instead |
|-------|-----|---------|
| Flaky tests | Erode trust, ignored failures | Fix root cause, add retries only as last resort |
| Hard-coded test data | Brittle, hard to maintain | Data factories |
| Sleep statements | Slow, unreliable | Wait for conditions |
| Shared test state | Test interference | Isolated test data |
| UI tests for logic | Slow, brittle | Unit tests for logic, E2E for flows |
| Ignoring test failures | Technical debt | Fix or remove |
| No test documentation | Knowledge silos | Self-documenting tests |
| Testing implementation | Brittle | Test behavior/outcomes |