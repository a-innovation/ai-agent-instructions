# Contributing to [Project Name]

Thank you for your interest in contributing!  🎉

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Pull Request Process](#pull-request-process)
- [Coding Standards](#coding-standards)
- [Testing Guidelines](#testing-guidelines)
- [Documentation](#documentation)

---

## 📜 Code of Conduct

Please read and follow our [Code of Conduct](CODE_OF_CONDUCT.md). 

---

## 🚀 Getting Started

### Prerequisites
- Node.js >= 20.x
- npm >= 10.x
- Git

### Setup
```bash
# Clone the repository
git clone https://github.com/org/repo.git
cd repo

# Install dependencies
npm install

# Set up environment
cp .env.example .env

# Run development server
npm run dev
```

---

## 🔄 Development Workflow

### Branch Naming
```
feature/[ticket-id]-short-description
bugfix/[ticket-id]-short-description
hotfix/[ticket-id]-short-description
docs/short-description
refactor/short-description
```

### Commit Messages
We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): description

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting
- `refactor`: Code restructuring
- `test`: Tests
- `chore`: Maintenance

**Example:**
```
feat(auth): add OAuth2 login support

- Implement Google OAuth provider
- Add token refresh mechanism
- Update user model with OAuth fields

Closes #123
```

---

## 🔀 Pull Request Process

1. **Create a branch** from `develop` (or `main`)
2. **Make your changes** following our coding standards
3. **Write/update tests** for your changes
4. **Update documentation** if needed
5. **Run all checks** locally:
   ```bash
   npm run lint
   npm run type-check
   npm run test
   ```
6. **Create a pull request** using our template
7. **Request review** from appropriate team members
8.  **Address feedback** and update as needed
9. **Merge** once approved and CI passes

### PR Requirements
- [ ] Follows coding standards
- [ ] Includes appropriate tests
- [ ] Documentation updated
- [ ] All CI checks pass
- [ ] Approved by at least 1 reviewer

---

## 📏 Coding Standards

### General
- Use TypeScript strict mode
- Maximum file length: 300 lines
- Maximum function length: 30 lines
- Use meaningful variable/function names

### Formatting
We use Prettier and ESLint.  Run before committing:
```bash
npm run format
npm run lint:fix
```

### File Structure
```
src/
├── components/     # UI components
├── hooks/          # Custom hooks
├── services/       # Business logic
├── utils/          # Utilities
├── types/          # TypeScript types
└── constants/      # Constants
```

---

## 🧪 Testing Guidelines

### Test Structure
```typescript
describe('ComponentName', () => {
  describe('methodName', () => {
    it('should [expected behavior] when [condition]', () => {
      // Arrange
      // Act
      // Assert
    });
  });
});
```

### Running Tests
```bash
# All tests
npm run test

# Watch mode
npm run test:watch

# Coverage
npm run test:coverage
```

---

## 📚 Documentation

- Update README.md for user-facing changes
- Add JSDoc comments for public APIs
- Update architecture docs for significant changes
- Include inline comments for complex logic

---

## ❓ Questions? 

- Check [GitHub Discussions](link)
- Join our [Slack/Discord](link)
- Email: dev@example.com

Thank you for contributing!  🙏