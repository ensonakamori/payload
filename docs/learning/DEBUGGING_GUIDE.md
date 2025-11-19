# Debugging Guide

**Documented:** November 18, 2025

Master troubleshooting and debugging in Payload CMS.

## Mental Model: Full-Stack Debugging

**For React Developers:**
In React apps, you primarily debug client-side in the browser. In Payload's full-stack environment, you debug across multiple layers:

```
Browser (Client)
    ↕ Network
Next.js (Server)
    ↕ Database Connection
Database
```

**Key Difference:**

- React: Single environment (browser)
- Payload: Multiple environments (browser, Node.js, database)

## Debugging Tools Overview

### Browser DevTools

**Use for:**

- Client components
- Network requests
- Console errors
- React component state
- Performance profiling

**Access:** `F12` or `Cmd+Opt+I` (Mac) / `Ctrl+Shift+I` (Windows)

### VS Code Debugger

**Use for:**

- Server components
- API routes
- Payload operations
- Breakpoints in backend code

**Setup:** `.vscode/launch.json` (see below)

### Payload Logger

**Use for:**

- Database queries
- Hook execution
- Access control checks
- Operation flow

**Enable:** `logger: true` in config

## Common Error Scenarios

### 1. "Cannot find module" Errors

**Error:**

```
Error: Cannot find module '@payloadcms/db-mongodb'
```

**Cause:** Missing dependencies or build artifacts

**Solution:**

```bash
# Install dependencies
pnpm install

# Build packages
pnpm run build:core

# Clear cache and reinstall
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### 2. Database Connection Errors

**Error:**

```
MongoServerError: bad auth: Authentication failed
```

**Debugging steps:**

1. **Check environment variables:**

```bash
# Print DATABASE_URI (without exposing password)
echo $DATABASE_URI | sed 's/:.*@/:****@/'
```

2. **Verify MongoDB is running:**

```bash
# Docker
pnpm docker:start

# Check MongoDB status
docker ps | grep mongo

# Test connection
mongosh "mongodb://localhost:27017/payload" --eval "db.runCommand({ ping: 1 })"
```

3. **Check Payload config:**

```typescript
// payload.config.ts
import { mongooseAdapter } from '@payloadcms/db-mongodb'

export default buildConfig({
  db: mongooseAdapter({
    url: process.env.DATABASE_URI,
    // Add debug logging
    connectOptions: {
      serverSelectionTimeoutMS: 5000,
    },
  }),
})
```

### 3. Type Errors

**Error:**

```
Property 'author' does not exist on type 'Post'
```

**Cause:** Payload types not generated or outdated

**Solution:**

```bash
# Generate types
pnpm run dev:generate-types <directory>

# Or start dev server (auto-generates types)
pnpm run dev
```

**Verify types:**

```typescript
import type { Post } from './payload-types'

const post: Post = {
  id: '123',
  title: 'Test',
  author: 'user-id', // Now TypeScript knows this exists
}
```

### 4. Infinite Loops / Recursion

**Error:**

```
RangeError: Maximum call stack size exceeded
```

**Common causes:**

**A) Hook calling itself:**

```typescript
// ❌ BAD: Causes infinite loop
{
  hooks: {
    beforeChange: [
      async ({ data, req }) => {
        await req.payload.update({
          collection: 'posts',
          id: data.id,
          data: { updatedAt: new Date() },
        })
        return data
      },
    ]
  }
}
```

**Fix:**

```typescript
// ✅ GOOD: Disable hooks
{
  hooks: {
    beforeChange: [
      async ({ data, req }) => {
        await req.payload.update({
          collection: 'posts',
          id: data.id,
          data: { updatedAt: new Date() },
          hooks: false, // Prevent recursion
        })
        return data
      },
    ]
  }
}
```

**B) Circular relationships with depth:**

```typescript
// Posts have authors, authors have posts
// ❌ BAD: depth: 10 causes infinite expansion
const post = await payload.findByID({
  collection: 'posts',
  id: postId,
  depth: 10, // Too deep!
})
```

**Fix:**

```typescript
// ✅ GOOD: Limit depth
const post = await payload.findByID({
  collection: 'posts',
  id: postId,
  depth: 1, // Just one level
})
```

### 5. Access Control Issues

**Error:**

```
Error: You are not allowed to perform this action
```

**Debug access control:**

**Enable verbose logging:**

```typescript
{
  slug: 'posts',
  access: {
    create: ({ req }) => {
      console.log('CREATE ACCESS CHECK:', {
        user: req.user,
        isAuthenticated: !!req.user
      })
      return !!req.user
    }
  }
}
```

**Check user in API:**

```typescript
// In Server Action or API route
export async function createPost(data: PostData) {
  const payload = await getPayload({ config })

  // Check current user
  const user = await payload.auth.check({ headers })
  console.log('Current user:', user)

  const post = await payload.create({
    collection: 'posts',
    data,
    // Temporarily bypass for debugging (REMOVE IN PRODUCTION!)
    overrideAccess: true,
  })
}
```

**Use authenticated requests in Postman:**

```bash
# Login first
POST /api/users/login
{
  "email": "dev@payloadcms.com",
  "password": "test"
}

# Copy the JWT cookie from response

# Then make request with cookie
POST /api/posts
Cookie: payload-token=<jwt-token>
```

### 6. Validation Errors

**Error:**

```
ValidationError: The following field is invalid: title
```

**Debug validation:**

```typescript
{
  name: 'title',
  type: 'text',
  validate: (value, { operation, data, req }) => {
    console.log('VALIDATING TITLE:', {
      value,
      operation,
      data,
      user: req.user
    })

    if (!value || value.length < 3) {
      return 'Title must be at least 3 characters'
    }

    return true
  }
}
```

**Catch and inspect:**

```typescript
try {
  await payload.create({
    collection: 'posts',
    data: { title: 'A' },
  })
} catch (error) {
  if (error.name === 'ValidationError') {
    console.error('Validation failed:', error.data)
    // error.data contains field-specific errors
  }
}
```

### 7. File Upload Errors

**Error:**

```
Error: File too large
```

**Debug uploads:**

```typescript
// payload.config.ts
{
  slug: 'media',
  upload: {
    limits: {
      fileSize: 5000000, // 5MB
    },
    imageSizes: [
      {
        name: 'thumbnail',
        width: 400,
        height: 300,
        // Debug image processing
        beforeCreate: async (file) => {
          console.log('Creating thumbnail:', {
            originalWidth: file.width,
            originalHeight: file.height
          })
        }
      }
    ]
  },
  hooks: {
    beforeChange: [
      ({ data, req }) => {
        console.log('UPLOAD DATA:', {
          filename: data.filename,
          mimeType: data.mimeType,
          filesize: data.filesize
        })
        return data
      }
    ]
  }
}
```

**Check file permissions:**

```bash
# Ensure upload directory is writable
ls -la /tmp/payload-uploads

# Create if missing
mkdir -p /tmp/payload-uploads
chmod 755 /tmp/payload-uploads
```

### 8. Next.js Cache Issues

**Problem:** Changes not appearing

**Solution:**

```bash
# Clear Next.js cache
rm -rf .next

# Clear Turbopack cache (if using Turbopack)
rm -rf .turbo

# Restart dev server
pnpm run dev
```

**Disable caching during development:**

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    // Disable caching for debugging
    staleTimes: {
      dynamic: 0,
      static: 0,
    },
  },
}
```

## Server-Side Debugging

### VS Code Setup

**Create `.vscode/launch.json`:**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Dev Server",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "pnpm",
      "runtimeArgs": ["run", "dev"],
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"],
      "env": {
        "NODE_OPTIONS": "--inspect"
      }
    },
    {
      "name": "Debug Integration Tests",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": ["--runInBand", "--no-cache", "${file}"],
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

### Using Breakpoints

**1. Set breakpoint in VS Code:**

- Click in the gutter next to line number
- Red dot appears

**2. Start debugger:**

- Press `F5` or click "Run and Debug"

**3. Code pauses at breakpoint:**

- Inspect variables
- Step through code (F10 = step over, F11 = step into)
- Evaluate expressions in Debug Console

**Example:**

```typescript
// packages/payload/src/collections/operations/create.ts
export async function create({
  collection,
  data,
  req
}: CreateArgs) {
  // Set breakpoint here
  debugger // Or add this line

  // Inspect in Debug Console:
  // > collection.slug
  // 'posts'
  // > data
  // { title: 'Test Post' }

  const result = await executeCreate({ ... })

  return result
}
```

### Console Debugging

**Strategic logging:**

```typescript
// Log with context
console.log('[CREATE POST]', {
  slug: collection.slug,
  data,
  userId: req.user?.id,
  timestamp: new Date().toISOString(),
})

// Use console.table for arrays
console.table(
  posts.map((p) => ({
    id: p.id,
    title: p.title,
    status: p.status,
  })),
)

// Use console.group for nested logs
console.group('Creating post')
console.log('Data:', data)
console.log('User:', req.user)
console.groupEnd()
```

### Payload Logger

**Enable in config:**

```typescript
export default buildConfig({
  logger: {
    level: 'debug', // 'error' | 'warn' | 'info' | 'debug'
    options: {
      // Custom logging options
    },
  },
})
```

**Output:**

```
[Payload] DEBUG: Executing create operation on collection: posts
[Payload] DEBUG: Running beforeValidate hooks
[Payload] DEBUG: Validating fields
[Payload] DEBUG: Running beforeChange hooks
[Payload] DEBUG: Inserting document
[Payload] DEBUG: Running afterChange hooks
```

## Client-Side Debugging

### React DevTools

**Install:**

- [Chrome Extension](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
- [Firefox Add-on](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/)

**Use for:**

- Inspect component tree
- View props and state
- Identify which components re-render
- Profile performance

**Example:**

1. Open DevTools → "Components" tab
2. Select component in tree
3. View props, state, hooks
4. Right-click → "Copy to clipboard" for debugging

### Browser Console

**Debug Server Components:**

```typescript
// Server Component
export async function PostsList() {
  const payload = await getPayload({ config })
  const posts = await payload.find({ collection: 'posts' })

  // Server logs appear in terminal
  console.log('[SERVER] Posts found:', posts.totalDocs)

  return <PostsListClient posts={posts.docs} />
}

// Client Component
'use client'
export function PostsListClient({ posts }) {
  // Client logs appear in browser console
  console.log('[CLIENT] Rendering posts:', posts.length)

  return <ul>{posts.map(...)}</ul>
}
```

### Network Tab

**Debug API calls:**

1. Open DevTools → "Network" tab
2. Filter: "Fetch/XHR"
3. Make request (e.g., save form)
4. Click request to see:
   - **Headers**: Check authentication cookies
   - **Payload**: Request body
   - **Preview/Response**: Response data
   - **Timing**: Performance metrics

**Example debugging flow:**

```
POST /api/posts
Status: 403 Forbidden

1. Check Headers tab:
   - Cookie: payload-token=... ← Is JWT present?

2. Check Payload tab:
   - { "title": "Test", "status": "draft" } ← Valid?

3. Check Response tab:
   - { "errors": [{ "message": "You are not allowed..." }] }
   - Now you know it's an access control issue
```

## Database Debugging

### MongoDB

**Enable query logging:**

```typescript
import { mongooseAdapter } from '@payloadcms/db-mongodb'

export default buildConfig({
  db: mongooseAdapter({
    url: process.env.DATABASE_URI,
    connectOptions: {
      // Log all queries
      debug: true,
    },
  }),
})
```

**Inspect database directly:**

```bash
# Connect to MongoDB
mongosh "mongodb://localhost:27017/payload"

# List collections
show collections

# Find documents
db.posts.find().pretty()

# Find by ID
db.posts.findOne({ _id: ObjectId("...") })

# Count documents
db.posts.countDocuments()

# Check indexes
db.posts.getIndexes()
```

### PostgreSQL

**Enable query logging:**

```typescript
import { postgresAdapter } from '@payloadcms/db-postgres'

export default buildConfig({
  db: postgresAdapter({
    pool: {
      connectionString: process.env.DATABASE_URI,
    },
    // Enable Drizzle query logging
    logger: true,
  }),
})
```

**Inspect database:**

```bash
# Connect to PostgreSQL
psql $DATABASE_URI

# List tables
\dt

# Describe table
\d posts

# Query posts
SELECT * FROM posts LIMIT 10;

# Check indexes
\di
```

### Drizzle Studio

**Visual database explorer:**

```bash
# Install Drizzle Kit
pnpm add -D drizzle-kit

# Start Drizzle Studio
pnpm drizzle-kit studio

# Opens at https://local.drizzle.studio
```

**Features:**

- Browse all tables
- View relationships
- Edit data visually
- Run custom queries

## Performance Debugging

### Slow API Responses

**Measure operation time:**

```typescript
export async function GET() {
  const start = performance.now()

  const payload = await getPayload({ config })
  const posts = await payload.find({
    collection: 'posts',
    limit: 100,
  })

  const duration = performance.now() - start
  console.log(`Query took ${duration.toFixed(2)}ms`)

  return Response.json(posts)
}
```

**Profile with Chrome DevTools:**

1. DevTools → "Performance" tab
2. Click record
3. Perform slow operation
4. Stop recording
5. Analyze flame chart

**Optimize queries:**

```typescript
// ❌ BAD: N+1 query problem
const posts = await payload.find({ collection: 'posts' })
for (const post of posts.docs) {
  // Makes separate query for each author!
  const author = await payload.findByID({
    collection: 'users',
    id: post.author,
  })
}

// ✅ GOOD: Use depth to populate
const posts = await payload.find({
  collection: 'posts',
  depth: 1, // Populates author in single query
})
```

### Memory Leaks

**Monitor memory:**

```bash
# Start dev server with memory tracking
NODE_OPTIONS="--inspect --max-old-space-size=4096" pnpm run dev
```

**Profile in Chrome:**

1. Open `chrome://inspect`
2. Click "inspect" under target
3. DevTools → "Memory" tab
4. Take heap snapshot
5. Perform actions
6. Take another snapshot
7. Compare snapshots

**Common causes:**

- Event listeners not cleaned up
- Timers not cleared
- Large objects in global scope
- Circular references

### React Performance

**Use React DevTools Profiler:**

1. DevTools → "Profiler" tab
2. Click record
3. Interact with app
4. Stop recording
5. Analyze render times

**Identify unnecessary re-renders:**

```typescript
'use client'
import { memo } from 'react'

// ❌ BAD: Re-renders every time parent renders
export function PostItem({ post }) {
  console.log('Rendering:', post.id)
  return <div>{post.title}</div>
}

// ✅ GOOD: Only re-renders when post changes
export const PostItem = memo(function PostItem({ post }) {
  console.log('Rendering:', post.id)
  return <div>{post.title}</div>
})
```

## Debugging Strategies

### 1. Rubber Duck Debugging

**Explain the problem out loud:**

- Describe what should happen
- Describe what actually happens
- Often you'll spot the issue while explaining

### 2. Binary Search

**For "it worked yesterday" bugs:**

```bash
# Find the commit that broke it
git bisect start
git bisect bad  # Current commit is broken
git bisect good <commit-hash>  # This old commit worked

# Git will checkout middle commit
# Test if bug exists
git bisect good  # or 'bad'

# Repeat until Git finds the breaking commit
```

### 3. Isolation

**Create minimal reproduction:**

```typescript
// Create test/debug/config.ts with minimal config
export default buildConfig({
  collections: [
    {
      slug: 'posts',
      fields: [
        { name: 'title', type: 'text' },
        // Only the fields needed to reproduce bug
      ],
    },
  ],
})

// Run dev server with this config
// pnpm run dev debug
```

### 4. Hypothesis Testing

**Scientific method:**

1. Observe the bug
2. Form hypothesis (e.g., "The hook is not running")
3. Test hypothesis (add `console.log` in hook)
4. If confirmed → fix
5. If not → new hypothesis

### 5. Read the Error Message

**Example error:**

```
Error: Cannot read property 'title' of undefined
    at PostsList (/app/posts/page.tsx:12:15)
    at renderComponent
```

**What it tells you:**

- **What**: "Cannot read property 'title' of undefined"
- **Where**: `/app/posts/page.tsx:12:15` (line 12, column 15)
- **Why**: Something is `undefined` when you expected an object

**Solution:**

```typescript
// Line 12 in posts/page.tsx
const title = post.title // post is undefined

// Add null check
const title = post?.title ?? 'Untitled'
```

## Common Gotchas

### 1. Environment Variables

**Problem:** `process.env.MY_VAR` is `undefined`

**Checklist:**

- ✅ Variable in `.env.local`?
- ✅ Prefixed with `NEXT_PUBLIC_` for client?
- ✅ Restarted dev server after changing `.env`?
- ✅ Not accidentally in `.env.example`?

**Verify:**

```typescript
// Server-side
console.log('DATABASE_URI exists:', !!process.env.DATABASE_URI)

// Client-side (must use NEXT_PUBLIC_ prefix)
console.log('API URL:', process.env.NEXT_PUBLIC_API_URL)
```

### 2. Async/Await

**Problem:** Data not available

```typescript
// ❌ BAD: Forgot await
const posts = payload.find({ collection: 'posts' })
console.log(posts.docs) // undefined - it's a Promise!

// ✅ GOOD: Use await
const posts = await payload.find({ collection: 'posts' })
console.log(posts.docs) // [...]
```

### 3. Server vs Client Components

**Problem:** Hooks error in Server Component

```typescript
// ❌ BAD: Can't use useState in Server Component
export default async function Page() {
  const [count, setCount] = useState(0) // Error!
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}

// ✅ GOOD: Split into Server + Client
// page.tsx (Server Component)
export default async function Page() {
  const data = await fetchData()
  return <Counter initialData={data} />
}

// Counter.tsx (Client Component)
'use client'
export function Counter({ initialData }) {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

### 4. Stale Imports

**Problem:** Changes not reflecting

```typescript
// ❌ BAD: Importing from wrong location
import { Post } from '../old-types/payload-types'

// ✅ GOOD: Use generated types
import { Post } from '@/payload-types'
```

**Fix:**

```bash
# Regenerate types
pnpm run dev:generate-types <directory>

# Clear cache
rm -rf .next node_modules/.cache
```

## Debugging Checklist

**When stuck, go through this list:**

1. ✅ Read the error message carefully
2. ✅ Check browser console AND terminal
3. ✅ Verify environment variables
4. ✅ Restart dev server
5. ✅ Clear caches (`.next`, `node_modules/.cache`)
6. ✅ Check database connection
7. ✅ Regenerate types
8. ✅ Check git diff (what changed recently?)
9. ✅ Search GitHub issues
10. ✅ Create minimal reproduction
11. ✅ Ask for help (Discord, GitHub Discussions)

## Getting Help

### GitHub Issues

**Search existing issues:**

```
https://github.com/payloadcms/payload/issues?q=<your+error>
```

**Create new issue:**
Include:

- Payload version
- Database adapter
- Node version
- Minimal reproduction
- Error message
- What you've tried

### Payload Discord

**Join:** https://discord.com/invite/payload

**Channels:**

- `#help` - General questions
- `#troubleshooting` - Debugging help
- `#database` - Database issues

**Prepare:**

- Describe what you're trying to do
- Share relevant code (use code blocks)
- Share error messages
- Mention what you've tried

---

_Debug systematically, solve efficiently!_
