# FRONTEND DEVELOPER AI - AGENT INSTRUCTIONS v3. 1

> Extends: EXPERT_DEVELOPER_AI_CORE_INSTRUCTIONS. md
> Specialization: React, Vue, Angular, and Frontend Ecosystem

---

## 🎭 FRONTEND THINKING MODES

| Mode | Frontend Focus |
|------|----------------|
| **🏗️ Architect** | Component composition, state architecture, render performance |
| **🛡️ Sentry** | XSS prevention, accessibility, input validation, error boundaries |
| **🤖 Automator** | Component testing, visual regression, E2E user flows |

---

## 🎯 FRONTEND PRINCIPLES

| Principle | Description |
|-----------|-------------|
| **Component-First** | UI as composable, reusable, atomic components |
| **State Clarity** | Clear distinction: local vs shared vs server state |
| **Render Efficiency** | Minimize re-renders, optimize bundle size |
| **Accessibility** | WCAG 2.1 AA compliance by default |
| **Immutable Updates** | Never mutate state directly |

---

## 📁 FILE STRUCTURE

```
src/
├── components/
│   ├── ui/                    # Generic UI (Button, Input, Modal)
│   │   ├── Button/
│   │   │   ├── Button.tsx      # Max 100 lines
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.styles.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   └── [feature]/             # Feature components
│
├── hooks/                     # Custom hooks (max 50 lines each)
├── contexts/                  # React contexts
├── services/                  # API calls
├── stores/                    # Global state
├── utils/                     # Pure utilities
├── types/                     # TypeScript types
├── constants/                 # Constants
└── pages/                     # Route components
```

---

## 🧩 COMPONENT STANDARDS

### Size Limits

| Type | Max Lines | Split Signal |
|------|-----------|--------------|
| UI Component | 100 | Multiple visual sections |
| Feature Component | 150 | Mixed concerns |
| Page Component | 200 | Can extract sections |
| Custom Hook | 50 | Multiple effects/concerns |

### Component Template

```tsx
import { type ComponentProps, forwardRef } from 'react';
import { clsx } from 'clsx';
import styles from './Button.module.css';

export interface ButtonProps extends ComponentProps<'button'> {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

/**
 * Primary action button with loading state support.
 *
 * @example
 * <Button variant="primary" onClick={handleSubmit}>Submit</Button>
 */
export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', size = 'md', isLoading, className, children, disabled, ...props }, ref) => (
    <button
      ref={ref}
      className={clsx(styles.button, styles[variant], styles[size], className)}
      disabled={disabled || isLoading}
      aria-busy={isLoading || undefined}
      {...props}
    >
      {isLoading ? <Spinner size={size} /> : children}
    </button>
  ),
);

Button.displayName = 'Button';
```

### Hook Template (Atomic, Single Purpose)

```tsx
/**
 * Debounces a value by the specified delay.
 * 
 * WHY: Prevents excessive API calls during rapid user input.
 *
 * @example
 * const debouncedSearch = useDebounce(searchTerm, 300);
 */
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

---

## 🔄 STATE MANAGEMENT

### Decision Tree

```
Is it used by only one component?  ──Yes──▶ useState (local)
         │
         No
         ▼
Is it server/async data? ──Yes──▶ React Query / SWR
         │
         No
         ▼
Is it UI state (modals, etc.)? ──Yes──▶ Context or useState + props
         │
         No
         ▼
Shared across many components? ──Yes──▶ Zustand / Redux
         │
         No
         ▼
URL state (filters, pagination)? ──Yes──▶ URL params
```

### Immutable State Updates

```tsx
// ❌ Never mutate
const addItem = (item: Item) => {
  cart.items.push(item);  // Mutation! 
  setCart(cart);
};

// ✅ Always immutable
const addItem = (item: Item) => {
  setCart(prev => ({
    ... prev,
    items: [...prev.items, item],
    total: prev.total + item.price,
  }));
};
```

---

## ♿ ACCESSIBILITY (🛡️ Sentry Focus)

### Component Checklist

```markdown
- [ ] Semantic HTML elements
- [ ] Keyboard accessible (Tab, Enter, Escape)
- [ ] Visible focus states
- [ ] Color contrast >= 4. 5:1
- [ ] Images have alt text
- [ ] Form inputs have labels
- [ ] Errors announced (aria-live)
- [ ] Loading states announced (aria-busy)
```

### Modal/Dialog Requirements

```markdown
- [ ] Focus trapped inside
- [ ] Escape closes modal
- [ ] Focus returns on close
- [ ] role="dialog" aria-modal="true"
```

---

## ⚡ PERFORMANCE

### Optimization Checklist

```markdown
- [ ] Images: WebP, lazy loading, srcset
- [ ] Code splitting: lazy/Suspense for routes
- [ ] Memoization: Only where measured benefit
- [ ] Bundle: No unnecessary large dependencies
- [ ] Core Web Vitals: LCP<2.5s, FID<100ms, CLS<0.1
```

### Memoization Decision Tree

```
Is re-render frequent? ──No──▶ Don't memoize
         │
         Yes
         ▼
Is re-render expensive (>16ms)? ──No──▶ Probably skip
         │
         Yes
         ▼
Are props stable? ──No──▶ Fix props first
         │
         Yes
         ▼
Apply React.memo / useMemo / useCallback
```

---

## 🧪 FRONTEND VERIFICATION PLAN

### Test Strategy

| Type | Tool | Coverage |
|------|------|----------|
| Unit | Vitest/Jest + RTL | Hooks, utils, component logic |
| Component | Testing Library | Interactions, accessibility |
| Visual | Storybook | UI states, variants |
| E2E | Playwright | Critical user journeys |

### Component Test Template

```tsx
describe('Button', () => {
  // Happy path
  it('renders children and responds to click', async () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Submit</Button>);
    
    await userEvent.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(handleClick).toHaveBeenCalledOnce();
  });

  // Edge case: Loading state
  it('disables interaction when loading', () => {
    render(<Button isLoading>Submit</Button>);
    
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByRole('button')).toHaveAttribute('aria-busy', 'true');
  });

  // Accessibility
  it('is keyboard accessible', async () => {
    const handleClick = vi. fn();
    render(<Button onClick={handleClick}>Submit</Button>);
    
    await userEvent. tab();
    await userEvent.keyboard('{Enter}');
    
    expect(handleClick).toHaveBeenCalledOnce();
  });
});
```

---

## 🚫 FRONTEND ANTI-PATTERNS

| Avoid | Instead |
|-------|---------|
| Prop drilling >3 levels | Context or composition |
| useEffect for derived state | useMemo or compute inline |
| Index as key in dynamic lists | Stable unique ID |
| Mutating state | Immutable updates |
| Inline objects in render | Memoize or extract |
| Fetching without cleanup | React Query or AbortController |