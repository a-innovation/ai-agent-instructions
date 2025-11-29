# Code Review Checklist

> **Reviewer:** @
> **PR:** #
> **Date:** YYYY-MM-DD

---

## 🎭 Review Using Three Thinking Modes

### 🏗️ Architect Mode (Design & Structure)

#### Code Organization
- [ ] Code follows established project patterns and architecture
- [ ] Files are appropriately sized (functions <30 lines, files <300 lines)
- [ ] Single Responsibility Principle is maintained
- [ ] Dependencies are properly managed (injected, not hardcoded)
- [ ] No circular dependencies introduced

#### Reusability
- [ ] Code reuses existing utilities/components where possible
- [ ] New shared logic is extracted to appropriate locations
- [ ] Constants are centralized, not hardcoded
- [ ] DRY principle followed (no copy-paste code)

#### API Design (if applicable)
- [ ] API follows RESTful conventions
- [ ] Request/response schemas are well-defined
- [ ] Backward compatibility maintained (or breaking changes documented)
- [ ] Versioning strategy followed

#### Data Design (if applicable)
- [ ] Database schema is normalized appropriately
- [ ] Indexes are considered for query performance
- [ ] Migrations are reversible
- [ ] Data types are appropriate

---

### 🛡️ Sentry Mode (Security & Edge Cases)

#### Security
- [ ] No sensitive data exposed (API keys, passwords, PII)
- [ ] Input validation implemented at boundaries
- [ ] SQL injection prevented (parameterized queries)
- [ ] XSS prevention in place (output encoding)
- [ ] Authentication/authorization properly checked
- [ ] Rate limiting considered for public endpoints
- [ ] CORS configured correctly

#### Error Handling
- [ ] Errors are caught and handled appropriately
- [ ] No empty catch blocks
- [ ] Error messages are user-friendly (no stack traces exposed)
- [ ] Errors are logged with sufficient context
- [ ] Graceful degradation implemented

#### Edge Cases
- [ ] Null/undefined cases handled
- [ ] Empty collections handled
- [ ] Boundary conditions tested
- [ ] Concurrent access considered
- [ ] Network failures handled
- [ ] Timeout handling implemented

#### Data Validation
- [ ] All inputs are validated
- [ ] Type checking is thorough (no `any` without justification)
- [ ] Required fields are enforced
- [ ] Data formats are validated (email, phone, etc.)

---

### 🤖 Automator Mode (Testing & Automation)

#### Test Coverage
- [ ] Unit tests cover happy path
- [ ] Unit tests cover edge cases
- [ ] Unit tests cover error scenarios
- [ ] Integration tests added (if applicable)
- [ ] E2E tests updated (if applicable)
- [ ] Test names are descriptive

#### Test Quality
- [ ] Tests are independent (no shared state)
- [ ] Tests are deterministic (no flaky tests)
- [ ] Mocks/stubs are used appropriately
- [ ] Test data is realistic

#### CI/CD
- [ ] All CI checks pass
- [ ] No decrease in code coverage
- [ ] Linting errors resolved
- [ ] Build succeeds

---

## 📊 Code Quality Assessment

### Readability
| Aspect | Rating | Notes |
|--------|--------|-------|
| Naming clarity | ⬜ Excellent ⬜ Good ⬜ Needs Work | |
| Code comments | ⬜ Excellent ⬜ Good ⬜ Needs Work | |
| Function length | ⬜ Excellent ⬜ Good ⬜ Needs Work | |
| Complexity | ⬜ Excellent ⬜ Good ⬜ Needs Work | |

### Maintainability
| Aspect | Rating | Notes |
|--------|--------|-------|
| Modularity | ⬜ Excellent ⬜ Good ⬜ Needs Work | |
| Testability | ⬜ Excellent ⬜ Good ⬜ Needs Work | |
| Documentation | ⬜ Excellent ⬜ Good ⬜ Needs Work | |
| Extensibility | ⬜ Excellent ⬜ Good ⬜ Needs Work | |

---

## 🔍 Detailed Findings

### 🔴 Must Fix (Blocking)

<!-- Critical issues that must be resolved before merge -->

| Location | Issue | Suggestion |
|----------|-------|------------|
| `file:line` | | |

### 🟡 Should Fix (Recommended)

<!-- Important improvements that should be addressed -->

| Location | Issue | Suggestion |
|----------|-------|------------|
| `file:line` | | |

### 🟢 Nice to Have (Optional)

<!-- Minor suggestions for improvement -->

| Location | Issue | Suggestion |
|----------|-------|------------|
| `file:line` | | |

### 💡 Praise

<!-- Highlight good practices and well-written code -->

- 
- 

---

## 📝 Questions for Author

<!-- Questions or clarifications needed -->

1. 
2. 

---

## 🎯 Summary

### Overall Assessment
- [ ] ✅ **Approve** - Ready to merge
- [ ] 🔄 **Request Changes** - Needs modifications
- [ ] 💬 **Comment** - Feedback only, no blocking issues

### Key Takeaways
<!-- 2-3 sentence summary of the review -->


### Estimated Effort for Fixes
- [ ] Trivial (< 30 min)
- [ ] Small (< 2 hours)
- [ ] Medium (< 1 day)
- [ ] Large (> 1 day)

---

## 📚 Resources

<!-- Helpful links for the author -->

- [Coding Standards](link)
- [Architecture Guidelines](link)
- [Testing Best Practices](link)

---

<!-- 
Review Tips:
- Be kind and constructive
- Explain the "why" behind suggestions
- Offer solutions, not just criticism
- Acknowledge good work
- Focus on the code, not the person
-->