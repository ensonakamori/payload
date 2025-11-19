# Development Workflow

**Documented:** November 18, 2025

Daily development workflow and best practices.

## Starting Work

```bash
# 1. Pull latest changes
git pull origin main

# 2. Create feature branch
git checkout -b feature/my-feature

# 3. Install dependencies (if package.json changed)
pnpm install

# 4. Build packages (if needed)
pnpm run build:core

# 5. Start dev server
pnpm run dev
```

## Making Changes

**Workflow:**
1. Make changes to code
2. Test in browser (auto-reload)
3. Run relevant tests
4. Commit frequently

**Testing:**
```bash
# Run specific test suite
pnpm test:int posts

# Run E2E tests
pnpm test:e2e:headed
```

## Committing

**Format:** `type(scope): description`

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `refactor` - Code refactor
- `test` - Tests
- `chore` - Maintenance

**Examples:**
```bash
git commit -m "feat(db-mongodb): add transaction support"
git commit -m "fix(ui): resolve table sorting bug"
git commit -m "docs: update getting started guide"
```

## Creating PRs

```bash
# Push branch
git push -u origin feature/my-feature

# Create PR via GitHub UI or:
gh pr create --title "Add feature X" --body "Description"
```

## Code Review

**Before submitting:**
- [ ] Tests pass
- [ ] Code linted
- [ ] Types compile
- [ ] Documentation updated

## Debugging

**Server-side:**
```typescript
console.log(data)  // Simple logging
debugger  // Breakpoint in VS Code
```

**Client-side:**
- Chrome DevTools
- React DevTools
- Network tab for API calls

## Common Commands

```bash
# Build specific package
pnpm run build:payload

# Clean build artifacts
pnpm clean:build

# Type check
tsc --noEmit

# Lint
pnpm lint

# Format
pnpm format
```

---

*Ship features confidently!*
