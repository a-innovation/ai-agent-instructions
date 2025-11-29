<!-- 
  Thank you for your contribution! 🎉
  Please fill out this template to help reviewers understand your changes. 
-->

## 📋 Description

<!-- Provide a clear and concise description of what this PR does.  -->

### What does this PR do? 


### Why is this change needed? 


### Related Issues/Tickets
<!-- Link to related issues, Jira tickets, or documentation -->
- Closes #
- Related to #
- Jira: 

---

## 🔄 Type of Change

<!-- Mark the relevant option(s) with an "x" -->

- [ ] 🐛 **Bug fix** (non-breaking change that fixes an issue)
- [ ] ✨ **New feature** (non-breaking change that adds functionality)
- [ ] 💥 **Breaking change** (fix or feature that would cause existing functionality to change)
- [ ] 📝 **Documentation** (changes to documentation only)
- [ ] 🔧 **Configuration** (changes to configuration files, CI/CD, etc.)
- [ ] ♻️ **Refactor** (code changes that neither fix a bug nor add a feature)
- [ ] 🎨 **Style** (formatting, missing semicolons, etc.; no code logic change)
- [ ] ⚡ **Performance** (changes that improve performance)
- [ ] 🧪 **Test** (adding or updating tests)
- [ ] 🔒 **Security** (security-related changes)

---

## 🧪 Testing

### How has this been tested? 

<!-- Describe the tests you ran and how to reproduce them -->

- [ ] Unit tests
- [ ] Integration tests
- [ ] E2E tests
- [ ] Manual testing

### Test Configuration
- **Environment:**
- **Browser/Device (if applicable):**
- **Test data used:**

### Test Evidence
<!-- Screenshots, logs, or test output if applicable -->

<details>
<summary>📸 Screenshots / Test Output</summary>

<!-- Add screenshots or test output here -->

</details>

---

## ✅ Checklist

### Code Quality
- [ ] My code follows the project's coding standards and style guidelines
- [ ] I have performed a self-review of my own code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings or errors
- [ ] I have added tests that prove my fix is effective or my feature works
- [ ] New and existing unit tests pass locally with my changes

### Security (🛡️ Sentry Mode)
- [ ] I have checked for potential security vulnerabilities
- [ ] No sensitive data (API keys, passwords, PII) is exposed
- [ ] Input validation is implemented where necessary
- [ ] No SQL injection, XSS, or other injection vulnerabilities introduced

### Performance (⚡ Considerations)
- [ ] I have considered the performance impact of my changes
- [ ] No N+1 queries or performance anti-patterns introduced
- [ ] Large data sets are handled efficiently (pagination, lazy loading)

### Breaking Changes
- [ ] This PR does NOT contain breaking changes
- [ ] If breaking: I have documented migration steps below
- [ ] If breaking: I have communicated with affected teams

---

## 🔀 Migration Guide

<!-- If this PR contains breaking changes, describe how users should migrate -->

<details>
<summary>Migration Steps (if applicable)</summary>

### Before
```
// Old way of doing things
```

### After
```
// New way of doing things
```

### Steps to Migrate
1. 
2. 
3. 

</details>

---

## 📊 Impact Analysis

### Files Changed
<!-- List the key files changed and why -->

| File | Change Type | Description |
|------|-------------|-------------|
| `src/... ` | Modified | |
| `tests/...` | Added | |

### Dependencies
- [ ] No new dependencies added
- [ ] New dependencies: <!-- List them and justify -->

### Database Changes
- [ ] No database changes
- [ ] Migration required: <!-- Describe migration -->

### API Changes
- [ ] No API changes
- [ ] API changes: <!-- Describe endpoint changes -->

---

## 🚀 Deployment Notes

<!-- Any special deployment considerations?  -->

- [ ] Standard deployment - no special steps needed
- [ ] Requires environment variable changes
- [ ] Requires database migration
- [ ] Requires cache invalidation
- [ ] Requires coordination with other teams
- [ ] Feature flag required

### Deployment Steps (if non-standard)
1. 
2.  
3. 

### Rollback Plan
<!-- How to rollback if something goes wrong -->


---

## 📚 Additional Context

<!-- Add any other context about the PR here -->

### Design Decisions
<!-- Explain any significant design decisions and alternatives considered -->


### Known Limitations
<!-- Any known limitations or future improvements needed -->


### Follow-up Tasks
<!-- Any tasks that should be done after this PR is merged -->
- [ ] 
- [ ] 

---

## 👀 Reviewer Notes

<!-- Specific areas you'd like reviewers to focus on -->

### Areas to Focus
- 
- 

### Questions for Reviewers
- 
- 

---

<!-- 
  Reminder: PRs should be:
  - Small and focused (ideally <400 lines)
  - Well-tested
  - Self-documenting
  - Ready for review (not WIP)
-->