# Testing Guide

**Documented:** November 18, 2025

Comprehensive guide to testing in Payload CMS.

## Mental Model: Testing Pyramid

**For React Developers:**
In traditional React apps, you might test components in isolation. In Payload's full-stack environment, you test across multiple layers:

```
        E2E Tests (Playwright)
              /\
             /  \
            /    \
           /      \
     Integration Tests (Jest)
          /        \
         /          \
        /            \
    Unit Tests (Jest)
```

**Payload Testing Focus:**

- **Unit Tests** - Utilities, helpers, validators (10%)
- **Integration Tests** - Collections, hooks, access control (70%)
- **E2E Tests** - Full user workflows, admin UI (20%)

## Test Infrastructure

### Test Directory Structure

Each test suite in `test/` follows this pattern:

```
test/<feature-name>/
├── config.ts           # Lightweight Payload config
├── int.spec.ts         # Integration tests (Jest)
├── e2e.spec.ts         # End-to-end tests (Playwright)
├── payload-types.ts    # Generated types
├── seed/               # Seed data (optional)
└── shared.ts           # Shared test utilities
```

**Example test directories:**

- `test/fields/` - Field type tests
- `test/auth/` - Authentication tests
- `test/access-control/` - Permission tests
- `test/uploads/` - File upload tests
- `test/relationships/` - Relationship field tests

### Mental Model: Test Configs

**React Parallel:**
Like setting up a fresh Redux store for each test, Payload creates a minimal config for each test suite.

**Payload Approach:**

```typescript
// test/posts/config.ts
import { buildConfig } from 'payload'

export default buildConfig({
  collections: [
    {
      slug: 'posts',
      fields: [
        { name: 'title', type: 'text', required: true },
        { name: 'status', type: 'select', options: ['draft', 'published'] },
      ],
    },
  ],
  // Minimal config - only what's needed for tests
  typescript: {
    outputFile: path.resolve(__dirname, 'payload-types.ts'),
  },
})
```

## Integration Testing (Jest)

### Setup Pattern

**Every integration test follows this structure:**

```typescript
import { getPayload } from 'payload'
import config from './config'

describe('Posts Collection', () => {
  let payload: Payload

  beforeAll(async () => {
    // Initialize Payload with test config
    payload = await getPayload({ config })
  })

  afterAll(async () => {
    // Cleanup
    if (typeof payload.db.destroy === 'function') {
      await payload.db.destroy()
    }
  })

  // Tests go here
})
```

**Why this pattern?**

- `beforeAll` runs once before all tests (faster than `beforeEach`)
- `afterAll` cleans up database connections
- Each test suite is isolated with its own config

### Testing CRUD Operations

**Create:**

```typescript
it('should create a post', async () => {
  const post = await payload.create({
    collection: 'posts',
    data: {
      title: 'Test Post',
      status: 'draft',
    },
  })

  expect(post.title).toBe('Test Post')
  expect(post.status).toBe('draft')
  expect(post.id).toBeDefined()
  expect(post.createdAt).toBeDefined()
})
```

**Read:**

```typescript
it('should find posts', async () => {
  // Seed data
  await payload.create({
    collection: 'posts',
    data: { title: 'Post 1', status: 'published' },
  })
  await payload.create({
    collection: 'posts',
    data: { title: 'Post 2', status: 'draft' },
  })

  // Query
  const result = await payload.find({
    collection: 'posts',
    where: {
      status: { equals: 'published' },
    },
  })

  expect(result.docs).toHaveLength(1)
  expect(result.docs[0].title).toBe('Post 1')
  expect(result.totalDocs).toBe(1)
})
```

**Update:**

```typescript
it('should update a post', async () => {
  const created = await payload.create({
    collection: 'posts',
    data: { title: 'Draft Post', status: 'draft' },
  })

  const updated = await payload.update({
    collection: 'posts',
    id: created.id,
    data: { status: 'published' },
  })

  expect(updated.status).toBe('published')
  expect(updated.title).toBe('Draft Post') // Unchanged
  expect(updated.updatedAt).not.toBe(created.updatedAt)
})
```

**Delete:**

```typescript
it('should delete a post', async () => {
  const created = await payload.create({
    collection: 'posts',
    data: { title: 'To Delete', status: 'draft' },
  })

  await payload.delete({
    collection: 'posts',
    id: created.id,
  })

  // Verify deletion
  const result = await payload.find({
    collection: 'posts',
    where: { id: { equals: created.id } },
  })

  expect(result.docs).toHaveLength(0)
})
```

### Testing Hooks

**beforeChange Hook:**

```typescript
// config.ts
{
  slug: 'posts',
  hooks: {
    beforeChange: [
      ({ data, operation }) => {
        if (operation === 'create') {
          data.slug = data.title.toLowerCase().replace(/\s+/g, '-')
        }
        return data
      }
    ]
  }
}

// int.spec.ts
it('should generate slug on create', async () => {
  const post = await payload.create({
    collection: 'posts',
    data: { title: 'My Test Post' }
  })

  expect(post.slug).toBe('my-test-post')
})
```

**afterChange Hook:**

```typescript
// Track afterChange executions
const afterChangeExecutions = []

// config.ts
{
  hooks: {
    afterChange: [
      ({ doc, operation }) => {
        afterChangeExecutions.push({ id: doc.id, operation })
        return doc
      },
    ]
  }
}

// int.spec.ts
it('should execute afterChange hook', async () => {
  afterChangeExecutions.length = 0 // Reset

  const post = await payload.create({
    collection: 'posts',
    data: { title: 'Test' },
  })

  expect(afterChangeExecutions).toHaveLength(1)
  expect(afterChangeExecutions[0]).toMatchObject({
    id: post.id,
    operation: 'create',
  })
})
```

### Testing Access Control

**Role-based access:**

```typescript
describe('Access Control', () => {
  it('should allow admin to create', async () => {
    const admin = await payload.create({
      collection: 'users',
      data: {
        email: 'admin@test.com',
        password: 'test',
        role: 'admin',
      },
    })

    // Login as admin
    const { token } = await payload.login({
      collection: 'users',
      data: {
        email: 'admin@test.com',
        password: 'test',
      },
    })

    // Create post as admin
    const post = await payload.create({
      collection: 'posts',
      data: { title: 'Admin Post' },
      overrideAccess: false,
      user: admin,
    })

    expect(post.title).toBe('Admin Post')
  })

  it('should deny public create', async () => {
    await expect(
      payload.create({
        collection: 'posts',
        data: { title: 'Public Post' },
        overrideAccess: false,
      }),
    ).rejects.toThrow('Forbidden')
  })
})
```

**Document-level access:**

```typescript
// config.ts
{
  access: {
    update: ({ req, data }) => {
      if (req.user?.role === 'admin') return true
      return req.user?.id === data.author
    }
  }
}

// int.spec.ts
it('should allow author to update own post', async () => {
  const author = await payload.create({
    collection: 'users',
    data: { email: 'author@test.com', password: 'test', role: 'user' },
  })

  const post = await payload.create({
    collection: 'posts',
    data: { title: 'My Post', author: author.id },
    user: author,
  })

  const updated = await payload.update({
    collection: 'posts',
    id: post.id,
    data: { title: 'Updated Title' },
    user: author,
  })

  expect(updated.title).toBe('Updated Title')
})

it('should deny non-author to update', async () => {
  const author = await payload.create({
    collection: 'users',
    data: { email: 'author@test.com', password: 'test' },
  })

  const otherUser = await payload.create({
    collection: 'users',
    data: { email: 'other@test.com', password: 'test' },
  })

  const post = await payload.create({
    collection: 'posts',
    data: { title: 'Author Post', author: author.id },
    user: author,
  })

  await expect(
    payload.update({
      collection: 'posts',
      id: post.id,
      data: { title: 'Hacked!' },
      user: otherUser,
      overrideAccess: false,
    }),
  ).rejects.toThrow()
})
```

### Testing Validation

**Required fields:**

```typescript
it('should require title', async () => {
  await expect(
    payload.create({
      collection: 'posts',
      data: { status: 'draft' }, // Missing title
    }),
  ).rejects.toThrow(/title.*required/i)
})
```

**Custom validation:**

```typescript
// config.ts
{
  name: 'email',
  type: 'email',
  validate: (value) => {
    if (!value.endsWith('@company.com')) {
      return 'Must be a company email'
    }
    return true
  }
}

// int.spec.ts
it('should validate company email', async () => {
  await expect(
    payload.create({
      collection: 'users',
      data: {
        email: 'user@gmail.com',
        password: 'test'
      }
    })
  ).rejects.toThrow('Must be a company email')

  // Valid email should work
  const user = await payload.create({
    collection: 'users',
    data: {
      email: 'user@company.com',
      password: 'test'
    }
  })

  expect(user.email).toBe('user@company.com')
})
```

### Testing Relationships

**HasOne relationship:**

```typescript
it('should populate relationship', async () => {
  const author = await payload.create({
    collection: 'users',
    data: { name: 'John Doe' },
  })

  const post = await payload.create({
    collection: 'posts',
    data: {
      title: 'Test Post',
      author: author.id,
    },
  })

  // Fetch with population
  const populated = await payload.findByID({
    collection: 'posts',
    id: post.id,
    depth: 1, // Populate relationships
  })

  expect(populated.author).toMatchObject({
    id: author.id,
    name: 'John Doe',
  })
})
```

**HasMany relationship:**

```typescript
it('should handle many relationships', async () => {
  const tag1 = await payload.create({
    collection: 'tags',
    data: { name: 'JavaScript' },
  })

  const tag2 = await payload.create({
    collection: 'tags',
    data: { name: 'TypeScript' },
  })

  const post = await payload.create({
    collection: 'posts',
    data: {
      title: 'TS Post',
      tags: [tag1.id, tag2.id],
    },
  })

  const populated = await payload.findByID({
    collection: 'posts',
    id: post.id,
    depth: 1,
  })

  expect(populated.tags).toHaveLength(2)
  expect(populated.tags[0].name).toBe('JavaScript')
  expect(populated.tags[1].name).toBe('TypeScript')
})
```

### Testing File Uploads

```typescript
import path from 'path'
import fs from 'fs'

it('should upload an image', async () => {
  const imagePath = path.resolve(__dirname, './seed/test-image.jpg')
  const imageBuffer = fs.readFileSync(imagePath)

  const upload = await payload.create({
    collection: 'media',
    data: {
      alt: 'Test Image',
    },
    filePath: imagePath,
  })

  expect(upload.filename).toBeDefined()
  expect(upload.mimeType).toBe('image/jpeg')
  expect(upload.filesize).toBeGreaterThan(0)
  expect(upload.width).toBeGreaterThan(0)
  expect(upload.height).toBeGreaterThan(0)

  // Check image was processed
  expect(upload.sizes).toBeDefined()
  expect(upload.sizes.thumbnail).toBeDefined()
})
```

### Testing Queries

**Where conditions:**

```typescript
describe('Query Tests', () => {
  beforeAll(async () => {
    // Seed data
    await payload.create({
      collection: 'posts',
      data: { title: 'Post 1', views: 100, status: 'published' },
    })
    await payload.create({
      collection: 'posts',
      data: { title: 'Post 2', views: 200, status: 'published' },
    })
    await payload.create({
      collection: 'posts',
      data: { title: 'Post 3', views: 50, status: 'draft' },
    })
  })

  it('should filter by equals', async () => {
    const result = await payload.find({
      collection: 'posts',
      where: {
        status: { equals: 'published' },
      },
    })

    expect(result.totalDocs).toBe(2)
  })

  it('should filter by greater than', async () => {
    const result = await payload.find({
      collection: 'posts',
      where: {
        views: { greater_than: 75 },
      },
    })

    expect(result.totalDocs).toBe(2)
    expect(result.docs.every((doc) => doc.views > 75)).toBe(true)
  })

  it('should filter by contains', async () => {
    const result = await payload.find({
      collection: 'posts',
      where: {
        title: { contains: '2' },
      },
    })

    expect(result.totalDocs).toBe(1)
    expect(result.docs[0].title).toBe('Post 2')
  })

  it('should combine AND conditions', async () => {
    const result = await payload.find({
      collection: 'posts',
      where: {
        and: [
          { status: { equals: 'published' } },
          { views: { greater_than: 150 } },
        ],
      },
    })

    expect(result.totalDocs).toBe(1)
    expect(result.docs[0].title).toBe('Post 2')
  })

  it('should combine OR conditions', async () => {
    const result = await payload.find({
      collection: 'posts',
      where: {
        or: [{ views: { less_than: 75 } }, { views: { greater_than: 150 } }],
      },
    })

    expect(result.totalDocs).toBe(2) // Post 2 (200) and Post 3 (50)
  })
})
```

**Pagination:**

```typescript
it('should paginate results', async () => {
  // Create 15 posts
  for (let i = 1; i <= 15; i++) {
    await payload.create({
      collection: 'posts',
      data: { title: `Post ${i}` },
    })
  }

  // Page 1
  const page1 = await payload.find({
    collection: 'posts',
    limit: 10,
    page: 1,
    sort: 'title',
  })

  expect(page1.docs).toHaveLength(10)
  expect(page1.totalDocs).toBe(15)
  expect(page1.totalPages).toBe(2)
  expect(page1.hasNextPage).toBe(true)
  expect(page1.hasPrevPage).toBe(false)

  // Page 2
  const page2 = await payload.find({
    collection: 'posts',
    limit: 10,
    page: 2,
  })

  expect(page2.docs).toHaveLength(5)
  expect(page2.hasNextPage).toBe(false)
  expect(page2.hasPrevPage).toBe(true)
})
```

**Sorting:**

```typescript
it('should sort ascending', async () => {
  await payload.create({
    collection: 'posts',
    data: { title: 'C Post', createdAt: new Date('2025-01-01') },
  })
  await payload.create({
    collection: 'posts',
    data: { title: 'A Post', createdAt: new Date('2025-01-03') },
  })
  await payload.create({
    collection: 'posts',
    data: { title: 'B Post', createdAt: new Date('2025-01-02') },
  })

  const result = await payload.find({
    collection: 'posts',
    sort: 'title',
  })

  expect(result.docs[0].title).toBe('A Post')
  expect(result.docs[1].title).toBe('B Post')
  expect(result.docs[2].title).toBe('C Post')
})

it('should sort descending', async () => {
  const result = await payload.find({
    collection: 'posts',
    sort: '-createdAt', // - prefix for descending
  })

  const dates = result.docs.map((doc) => new Date(doc.createdAt).getTime())
  expect(dates).toEqual([...dates].sort((a, b) => b - a))
})
```

## E2E Testing (Playwright)

### Mental Model: Browser Automation

**React Testing Library Parallel:**
If you've used React Testing Library, Playwright is similar but for full browsers:

- **RTL**: `screen.getByRole('button')` → Click
- **Playwright**: `page.getByRole('button')` → Click

**Difference:**

- RTL tests React components in isolation
- Playwright tests full application in real browser

### Setup Pattern

```typescript
import { test, expect } from '@playwright/test'

test.describe('Posts Admin', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to admin
    await page.goto('/admin')

    // Login
    await page.fill('input[name="email"]', 'dev@payloadcms.com')
    await page.fill('input[name="password"]', 'test')
    await page.click('button[type="submit"]')

    // Wait for dashboard
    await page.waitForURL('/admin')
  })

  // Tests go here
})
```

### Testing Admin UI

**Navigate and create:**

```typescript
test('should create a new post', async ({ page }) => {
  // Go to posts collection
  await page.click('text=Posts')
  await page.waitForURL('/admin/collections/posts')

  // Click Create New
  await page.click('text=Create New')
  await page.waitForURL(/\/admin\/collections\/posts\/create/)

  // Fill form
  await page.fill('input[name="title"]', 'My New Post')
  await page.fill('textarea[name="content"]', 'Post content here')

  // Save
  await page.click('button:has-text("Save")')

  // Wait for redirect to edit page
  await page.waitForURL(/\/admin\/collections\/posts\/\d+/)

  // Verify success message
  await expect(page.locator('text=Successfully created')).toBeVisible()

  // Verify data
  await expect(page.locator('input[name="title"]')).toHaveValue('My New Post')
})
```

**Search and filter:**

```typescript
test('should filter posts', async ({ page }) => {
  await page.goto('/admin/collections/posts')

  // Use search
  await page.fill('input[placeholder*="Search"]', 'Test')
  await page.keyboard.press('Enter')

  // Wait for results
  await page.waitForTimeout(500)

  // Check filtered results
  const rows = page.locator('table tbody tr')
  await expect(rows).not.toHaveCount(0)

  // Each row should contain "Test"
  const count = await rows.count()
  for (let i = 0; i < count; i++) {
    const text = await rows.nth(i).textContent()
    expect(text.toLowerCase()).toContain('test')
  }
})
```

**Edit and update:**

```typescript
test('should edit a post', async ({ page }) => {
  await page.goto('/admin/collections/posts')

  // Click first row
  await page.click('table tbody tr:first-child')

  // Wait for edit page
  await page.waitForURL(/\/admin\/collections\/posts\/\d+/)

  // Change title
  const titleInput = page.locator('input[name="title"]')
  const originalTitle = await titleInput.inputValue()
  await titleInput.fill('Updated Title')

  // Save
  await page.click('button:has-text("Save")')

  // Wait for save confirmation
  await expect(page.locator('text=Successfully updated')).toBeVisible()

  // Verify change persisted
  await page.reload()
  await expect(titleInput).toHaveValue('Updated Title')
})
```

**Delete:**

```typescript
test('should delete a post', async ({ page }) => {
  await page.goto('/admin/collections/posts')

  // Get initial count
  const initialCount = await page.locator('table tbody tr').count()

  // Click first row
  await page.click('table tbody tr:first-child')
  await page.waitForURL(/\/admin\/collections\/posts\/\d+/)

  // Click delete
  await page.click('button:has-text("Delete")')

  // Confirm deletion
  await page.click('button:has-text("Confirm")')

  // Wait for redirect to list
  await page.waitForURL('/admin/collections/posts')

  // Verify count decreased
  const newCount = await page.locator('table tbody tr').count()
  expect(newCount).toBe(initialCount - 1)
})
```

### Testing File Uploads

```typescript
test('should upload an image', async ({ page }) => {
  await page.goto('/admin/collections/media/create')

  // Upload file
  const fileInput = page.locator('input[type="file"]')
  await fileInput.setInputFiles(path.join(__dirname, 'test-image.jpg'))

  // Wait for upload to complete
  await page.waitForSelector('img[src*="test-image"]')

  // Fill alt text
  await page.fill('input[name="alt"]', 'Test image alt text')

  // Save
  await page.click('button:has-text("Save")')

  // Verify success
  await expect(page.locator('text=Successfully created')).toBeVisible()

  // Verify image preview
  const img = page.locator('img[src*="test-image"]')
  await expect(img).toBeVisible()
})
```

### Testing Authentication

```typescript
test('should logout and login', async ({ page }) => {
  // Assume we're logged in from beforeEach

  // Logout
  await page.click('button[aria-label="Account"]')
  await page.click('text=Logout')

  // Should redirect to login
  await page.waitForURL('/admin/login')

  // Login again
  await page.fill('input[name="email"]', 'dev@payloadcms.com')
  await page.fill('input[name="password"]', 'test')
  await page.click('button[type="submit"]')

  // Should redirect to dashboard
  await page.waitForURL('/admin')
  await expect(page.locator('text=Dashboard')).toBeVisible()
})

test('should deny access without login', async ({ page }) => {
  // Go to admin without logging in
  await page.goto('/admin/collections/posts')

  // Should redirect to login
  await page.waitForURL('/admin/login')
})
```

### Testing Rich Text Editor

```typescript
test('should use rich text editor', async ({ page }) => {
  await page.goto('/admin/collections/posts/create')

  // Find Lexical editor
  const editor = page.locator('[contenteditable="true"]').first()

  // Type text
  await editor.click()
  await editor.type('This is bold text')

  // Select text
  await page.keyboard.press('Control+A')

  // Make bold
  await page.click('button[aria-label="Bold"]')

  // Verify bold
  await expect(editor.locator('strong')).toHaveText('This is bold text')

  // Add link
  await page.click('button[aria-label="Link"]')
  await page.fill('input[name="url"]', 'https://example.com')
  await page.click('button:has-text("Add Link")')

  // Verify link
  await expect(editor.locator('a[href="https://example.com"]')).toBeVisible()
})
```

## Running Tests

### Integration Tests

```bash
# All integration tests (MongoDB)
pnpm run test:int

# Specific test suite
pnpm run test:int fields

# With Postgres
pnpm run test:int:postgres

# With SQLite
pnpm run test:int:sqlite

# Watch mode
pnpm run test:int:watch fields

# Coverage
pnpm run test:int:coverage
```

### E2E Tests

```bash
# Headless (CI mode)
pnpm run test:e2e

# Headed (see browser)
pnpm run test:e2e:headed

# Debug mode (pause on failure)
pnpm run test:e2e:debug

# Specific test file
pnpm run test:e2e fields/e2e.spec.ts

# UI mode (interactive)
pnpm run test:e2e:ui
```

### Component Tests

```bash
# UI component tests
pnpm run test:components

# Watch mode
pnpm run test:components:watch
```

## Test Utilities

### Seeding Data

**Create seed function:**

```typescript
// shared.ts
export async function seedPosts(payload: Payload, count: number) {
  const posts = []

  for (let i = 1; i <= count; i++) {
    const post = await payload.create({
      collection: 'posts',
      data: {
        title: `Post ${i}`,
        content: `Content for post ${i}`,
        status: i % 2 === 0 ? 'published' : 'draft',
      },
    })
    posts.push(post)
  }

  return posts
}
```

**Use in tests:**

```typescript
import { seedPosts } from './shared'

it('should handle multiple posts', async () => {
  await seedPosts(payload, 10)

  const result = await payload.find({
    collection: 'posts',
  })

  expect(result.totalDocs).toBe(10)
})
```

### Custom Matchers

```typescript
// jest.setup.ts
expect.extend({
  toHaveSuccessStatus(response) {
    const pass = response.status >= 200 && response.status < 300
    return {
      pass,
      message: () => `Expected status ${response.status} to be successful`,
    }
  },
})

// In tests
const response = await fetch('/api/posts')
expect(response).toHaveSuccessStatus()
```

### Mocking External Services

**Mock S3 uploads:**

```typescript
import { S3Client } from '@aws-sdk/client-s3'

jest.mock('@aws-sdk/client-s3', () => ({
  S3Client: jest.fn().mockImplementation(() => ({
    send: jest.fn().mockResolvedValue({
      ETag: '"mock-etag"',
      Location: 'https://mock-bucket.s3.amazonaws.com/mock-file.jpg',
    }),
  })),
}))
```

**Mock email sending:**

```typescript
const mockSendMail = jest.fn().mockResolvedValue({ messageId: 'mock-id' })

jest.mock('nodemailer', () => ({
  createTransport: jest.fn().mockReturnValue({
    sendMail: mockSendMail,
  }),
}))

// In test
it('should send email', async () => {
  await payload.create({
    collection: 'users',
    data: { email: 'test@example.com', password: 'test' },
  })

  expect(mockSendMail).toHaveBeenCalledWith(
    expect.objectContaining({
      to: 'test@example.com',
    }),
  )
})
```

## Testing Best Practices

### DO:

✅ Test business logic, not implementation details
✅ Use descriptive test names: `should create post with valid data`
✅ Arrange-Act-Assert pattern
✅ Clean up resources in `afterAll`
✅ Use `beforeAll` for setup (faster)
✅ Test error cases and edge cases
✅ Mock external services (S3, email, etc.)
✅ Use type-safe matchers

### DON'T:

❌ Test Payload internals (trust the framework)
❌ Test third-party libraries
❌ Use `beforeEach` unless necessary (slower)
❌ Make tests depend on each other
❌ Use hardcoded IDs (use created IDs)
❌ Skip cleanup (`afterAll`)
❌ Test UI implementation details (test behavior)

### Test Organization

**Group related tests:**

```typescript
describe('Posts Collection', () => {
  describe('Create', () => {
    it('should create with valid data', async () => {})
    it('should reject without title', async () => {})
    it('should auto-generate slug', async () => {})
  })

  describe('Access Control', () => {
    it('should allow admin', async () => {})
    it('should deny public', async () => {})
  })
})
```

**Use helper functions:**

```typescript
async function createUser(role: 'admin' | 'user') {
  return await payload.create({
    collection: 'users',
    data: {
      email: `${role}@test.com`,
      password: 'test',
      role,
    },
  })
}

it('should test admin', async () => {
  const admin = await createUser('admin')
  // Test with admin
})
```

## Debugging Tests

### Jest Debugging

**Add breakpoints:**

```typescript
it('should create post', async () => {
  debugger // Will pause here in VS Code debugger

  const post = await payload.create({
    collection: 'posts',
    data: { title: 'Test' },
  })
})
```

**Run specific test:**

```bash
# Only this test
pnpm run test:int fields -- -t "should validate required field"
```

**Increase timeout:**

```typescript
it('should handle long operation', async () => {
  // Increase timeout for this test
  jest.setTimeout(10000)

  await longRunningOperation()
}, 10000) // Or set here
```

### Playwright Debugging

**Slow down execution:**

```typescript
test.use({ launchOptions: { slowMo: 500 } })
```

**Pause on failure:**

```bash
pnpm run test:e2e:debug
```

**Visual debugging:**

```typescript
test('should create post', async ({ page }) => {
  await page.pause() // Opens Playwright Inspector

  await page.click('text=Create New')
})
```

**Screenshots on failure:**

```typescript
test.afterEach(async ({ page }, testInfo) => {
  if (testInfo.status !== testInfo.expectedStatus) {
    await page.screenshot({
      path: `screenshots/${testInfo.title}.png`,
    })
  }
})
```

## Continuous Integration

**GitHub Actions example:**

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mongodb:
        image: mongo:7
        ports:
          - 27017:27017

    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'pnpm'

      - run: pnpm install
      - run: pnpm run build:core
      - run: pnpm run test:int
      - run: pnpm run test:e2e
```

---

_Test comprehensively, ship confidently!_
