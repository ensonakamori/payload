# First Contributions

**Documented:** November 18, 2025

Your guide to contributing to Payload CMS.

## Why Contribute?

Contributing to open source:

- **Learn** from experienced developers
- **Build** your portfolio with real-world code
- **Connect** with the community
- **Shape** the future of Payload

**Payload welcomes contributions of all sizes!**

## Before You Start

**Prerequisites:**

- Completed [GETTING_STARTED.md](./GETTING_STARTED.md)
- Completed [EXERCISES.md](./EXERCISES.md) (recommended)
- Familiar with Git and GitHub
- Node.js 20+ and pnpm installed

**Helpful to read:**

- [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
- [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)
- [TESTING_GUIDE.md](./TESTING_GUIDE.md)

## Types of Contributions

### 1. Bug Reports

**Found a bug?**

1. Search [existing issues](https://github.com/payloadcms/payload/issues)
2. If not found, create new issue
3. Include:
   - Payload version
   - Database adapter
   - Node version
   - Steps to reproduce
   - Expected vs actual behavior
   - Error messages/screenshots

**Good bug report example:**

```markdown
## Bug Description

Upload field causes crash when file size exceeds limit

## To Reproduce

1. Create collection with upload field
2. Set fileSize limit to 1MB
3. Upload 2MB file
4. Server crashes with "RangeError: Maximum call stack size exceeded"

## Environment

- Payload: 3.64.0
- Database: MongoDB
- Node: 20.10.0
- OS: macOS Sonoma

## Expected Behavior

Should show validation error: "File too large"

## Actual Behavior

Server crashes, need to restart

## Error Log
```

RangeError: Maximum call stack size exceeded
at uploadFile (/packages/payload/src/uploads/uploadFile.ts:45)
...

```

```

### 2. Feature Requests

**Have an idea?**

1. Check [existing feature requests](https://github.com/payloadcms/payload/issues?q=is%3Aissue+label%3Aenhancement)
2. Discuss in [Discord #feature-requests](https://discord.com/invite/payload)
3. Create GitHub issue with:
   - Problem you're solving
   - Proposed solution
   - Alternatives considered
   - Impact on existing features

**Good feature request:**

```markdown
## Problem

Currently, there's no way to bulk-edit documents in the admin UI.
Editors need to update status on 100+ posts manually.

## Proposed Solution

Add "Bulk Actions" menu to list view:

- Select multiple documents (checkboxes)
- Choose action (Update, Delete, Publish)
- Apply to all selected

## Alternatives

- Use REST API (requires technical knowledge)
- Update via database directly (risky)

## Impact

- New UI component in list view
- New REST endpoint for bulk operations
- Backward compatible (opt-in feature)
```

### 3. Documentation

**Easiest way to contribute!**

**Types:**

- Fix typos/grammar
- Add examples
- Clarify confusing sections
- Create tutorials
- Update outdated content

**Process:**

1. Find documentation at `/docs`
2. Make changes locally
3. Test locally: `pnpm run dev:docs`
4. Submit PR (see below)

### 4. Code Contributions

**Bug fixes, features, improvements.**

**Good first issues:**

- Look for `good-first-issue` label
- Look for `help-wanted` label
- Simple bugs in specific packages

**Where to contribute:**

- Bug fixes (any size)
- Tests (improve coverage)
- TypeScript improvements
- Performance optimizations
- Refactoring

## Development Setup

### 1. Fork and Clone

```bash
# Fork on GitHub (click "Fork" button)

# Clone your fork
git clone https://github.com/YOUR-USERNAME/payload.git
cd payload

# Add upstream remote
git remote add upstream https://github.com/payloadcms/payload.git
```

### 2. Install Dependencies

```bash
# Install all dependencies
pnpm install

# Build core packages
pnpm run build:core
```

### 3. Create Branch

```bash
# Sync with upstream
git fetch upstream
git checkout main
git merge upstream/main

# Create feature branch
git checkout -b fix/upload-validation

# Or for feature:
git checkout -b feat/bulk-actions
```

**Branch naming:**

- `fix/` - Bug fixes
- `feat/` - New features
- `docs/` - Documentation
- `refactor/` - Code refactoring
- `test/` - Adding tests

## Making Changes

### 1. Make Your Edits

**Where to edit:**

- **Core logic**: `packages/payload/src`
- **UI components**: `packages/ui/src`
- **Database adapters**: `packages/db-*`
- **Plugins**: `packages/plugin-*`

**Example: Fix bug in upload validation**

Find the issue:

```bash
# Search for upload validation
grep -r "fileSize" packages/payload/src/uploads/
```

Make the fix:

```typescript
// packages/payload/src/uploads/uploadFile.ts
export async function uploadFile(args) {
  const { file, req, collection } = args

  // ❌ OLD: Causes crash
  if (file.size > collection.upload.limits.fileSize) {
    throw new RangeError('File too large')
  }

  // ✅ NEW: Proper validation error
  if (file.size > collection.upload.limits.fileSize) {
    throw new ValidationError([
      {
        field: 'file',
        message: `File size must be less than ${collection.upload.limits.fileSize} bytes`,
      },
    ])
  }

  // ...rest of upload logic
}
```

### 2. Write Tests

**Every code change needs tests!**

**Create test file:**

```typescript
// test/uploads/int.spec.ts
import { getPayload } from 'payload'
import config from './config'

describe('Upload Validation', () => {
  let payload

  beforeAll(async () => {
    payload = await getPayload({ config })
  })

  it('should validate file size limit', async () => {
    const largeFile = createMockFile(10000000) // 10MB

    await expect(
      payload.create({
        collection: 'media',
        data: { alt: 'Large file' },
        filePath: largeFile.path,
      }),
    ).rejects.toThrow(/File size must be less than/)
  })

  it('should allow file under limit', async () => {
    const smallFile = createMockFile(500000) // 500KB

    const upload = await payload.create({
      collection: 'media',
      data: { alt: 'Small file' },
      filePath: smallFile.path,
    })

    expect(upload.id).toBeDefined()
    expect(upload.filesize).toBeLessThan(5000000)
  })
})
```

**Run tests:**

```bash
# Run specific test
pnpm run test:int uploads

# Run all tests
pnpm run test:int

# Watch mode (during development)
pnpm run test:int:watch uploads
```

### 3. Update Documentation

**If you changed behavior:**

1. Update README if public API changed
2. Update TypeScript types
3. Add/update JSDoc comments
4. Update docs website (`/docs`)

**Example:**

````typescript
/**
 * Upload a file to the specified collection
 *
 * @param collection - Collection slug with upload enabled
 * @param data - Document data (without file)
 * @param filePath - Absolute path to file
 * @returns Uploaded document with file metadata
 * @throws {ValidationError} If file exceeds size limit
 *
 * @example
 * ```ts
 * const upload = await payload.create({
 *   collection: 'media',
 *   data: { alt: 'My image' },
 *   filePath: '/path/to/image.jpg'
 * })
 * ```
 */
export async function uploadFile(args: UploadArgs): Promise<Document> {
  // ...
}
````

### 4. Lint and Format

```bash
# Check linting
pnpm run lint

# Auto-fix linting issues
pnpm run lint:fix

# Format code (Prettier)
pnpm run format
```

### 5. Commit Changes

**Follow conventional commits:**

```bash
# Format: type(scope): description
git commit -m "fix(uploads): validate file size before processing"
git commit -m "feat(ui): add bulk actions to list view"
git commit -m "docs: update upload field documentation"
```

**Commit types:**

- `fix` - Bug fixes
- `feat` - New features
- `docs` - Documentation
- `test` - Adding tests
- `refactor` - Code refactoring
- `perf` - Performance improvements
- `chore` - Maintenance

**Scope:**
Match package name:

- `uploads`, `auth`, `collections`
- `db-mongodb`, `db-postgres`
- `ui`, `next`, `graphql`
- `plugin-*`, `richtext-*`, `storage-*`

## Creating a Pull Request

### 1. Push to Fork

```bash
git push origin fix/upload-validation
```

### 2. Open PR on GitHub

1. Go to [Payload repository](https://github.com/payloadcms/payload)
2. Click "Pull Requests" → "New Pull Request"
3. Click "compare across forks"
4. Select your fork and branch
5. Click "Create Pull Request"

### 3. Fill PR Template

**PR Title:**

```
fix(uploads): validate file size before processing
```

**PR Description:**

```markdown
## Summary

Fixes file upload crash when file exceeds size limit.

## Changes

- Added proper ValidationError for file size check
- Updated error message to show size limit
- Added integration tests

## Related Issue

Fixes #12345

## Test Plan

1. Create collection with upload limit: 1MB
2. Upload 2MB file
3. Should show validation error (not crash)
4. Upload 500KB file
5. Should succeed

## Checklist

- [x] Tests added/updated
- [x] Docs updated
- [x] Follows code style
- [x] No breaking changes
```

**Screenshots (if UI change):**
Include before/after screenshots.

### 4. Wait for Review

**What happens next:**

1. CI runs tests automatically
2. Maintainer reviews code
3. You may need to make changes
4. Once approved, it gets merged!

**Review process:**

- Be patient (maintainers are volunteers)
- Respond to feedback professionally
- Make requested changes
- Push updates to same branch

**Making changes:**

```bash
# Make requested changes
git add .
git commit -m "fix: address review feedback"
git push origin fix/upload-validation

# PR updates automatically
```

## PR Review Checklist

**Before submitting, verify:**

### Code Quality

- ✅ Follows existing code style
- ✅ No console.logs (unless intentional)
- ✅ No commented-out code
- ✅ Descriptive variable names
- ✅ Functions are small and focused
- ✅ No unnecessary dependencies added

### Testing

- ✅ Tests pass locally: `pnpm run test:int`
- ✅ New features have tests
- ✅ Bug fixes have regression tests
- ✅ Edge cases covered

### Documentation

- ✅ JSDoc comments for public APIs
- ✅ README updated (if needed)
- ✅ Types updated
- ✅ Breaking changes documented

### Git

- ✅ Commit messages follow convention
- ✅ Branch up to date with main
- ✅ No merge commits (rebase if needed)
- ✅ Single logical change per commit

## Common Mistakes to Avoid

### ❌ Don't

**1. Make unrelated changes**

```bash
# BAD: Mixing fixes
git commit -m "fix uploads and update deps and format files"

# GOOD: Separate commits
git commit -m "fix(uploads): validate file size"
git commit -m "chore: update dependencies"
```

**2. Skip tests**

```typescript
// ❌ BAD: No tests for new code
export function newFeature() {
  // Complex logic without tests
}

// ✅ GOOD: Tests included
describe('newFeature', () => {
  it('should handle edge case', () => {})
})
```

**3. Make breaking changes without discussion**

```typescript
// ❌ BAD: Breaking existing API
-export function find(args) { }
+export function findDocuments(args) { } // Renamed!

// ✅ GOOD: Deprecate first
export function find(args) {
  console.warn('find() is deprecated, use findDocuments()')
  return findDocuments(args)
}
```

**4. Ignore PR template**

```markdown
# ❌ BAD PR description

Fixed bug

# ✅ GOOD PR description

## Summary

Fixed file upload crash when file exceeds size limit

## Changes

- Added ValidationError
- Updated tests

## Related Issue

Fixes #12345
```

### ✅ Do

**1. Start small**

- Fix typos
- Add tests
- Improve error messages
- Refactor small functions

**2. Ask questions**

- Not sure how something works? Ask!
- Discord: #help-and-discussions
- GitHub: Comment on issue

**3. Read existing code**

- See how similar features work
- Follow established patterns
- Match coding style

**4. Be patient**

- Reviews take time
- Feedback is meant to help
- Stay professional

## Getting Help

**Stuck? Here's how to get help:**

### Discord

- [Join Payload Discord](https://discord.com/invite/payload)
- Channels:
  - `#help` - General questions
  - `#contributing` - Contribution help
  - `#development` - Technical discussions

### GitHub Discussions

- [Payload Discussions](https://github.com/payloadcms/payload/discussions)
- Ask questions
- Share ideas
- Get feedback

### Office Hours

- Payload team hosts regular office hours
- Check Discord for schedule
- Ask questions live

## After Your PR Merges

**Congratulations! Your code is now in Payload!**

### What's next?

**1. Celebrate!**

- Share on social media
- Add to your portfolio
- Thank reviewers

**2. Stay involved**

- Help review other PRs
- Answer questions in Discord
- Contribute more!

**3. Update your resume**

- "Contributed to Payload CMS"
- Link to your PRs
- Showcase your work

## Contribution Ideas

**Easy contributions:**

- Fix typos in documentation
- Add code examples
- Improve error messages
- Add unit tests
- Update outdated screenshots

**Medium contributions:**

- Fix bugs (good-first-issue label)
- Improve TypeScript types
- Add integration tests
- Refactor complex functions
- Optimize performance

**Advanced contributions:**

- New field types
- Database adapter improvements
- Admin UI features
- Plugin development
- Performance optimizations

## Code of Conduct

**Be respectful:**

- Welcome newcomers
- Assume good intent
- Provide constructive feedback
- No harassment or discrimination
- Follow [Code of Conduct](https://github.com/payloadcms/payload/blob/main/CODE_OF_CONDUCT.md)

## Resources

**Documentation:**

- [Payload Docs](https://payloadcms.com/docs)
- [LLMS.txt](https://payloadcms.com/llms.txt)
- [This learning guide](./README.md)

**Community:**

- [Discord](https://discord.com/invite/payload)
- [GitHub Discussions](https://github.com/payloadcms/payload/discussions)
- [Twitter/X](https://twitter.com/payloadcms)

**Tools:**

- [VSCode](https://code.visualstudio.com/)
- [GitHub Desktop](https://desktop.github.com/) (if new to Git)
- [Postman](https://www.postman.com/) (for API testing)

---

**You're ready to contribute!**

_Start small, learn lots, build community._

**Your first PR is waiting!** 🚀
