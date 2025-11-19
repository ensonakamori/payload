# Frequently Asked Questions

**Documented:** November 18, 2025

Common questions and answers about Payload CMS development.

## Getting Started

### Q: What's the difference between Payload 2.x and 3.x?

**A:** Major architectural changes:

**Payload 3.x (Current):**

- Built on Next.js App Router
- React Server Components by default
- Drizzle ORM for SQL databases
- Lexical rich text editor
- TypeScript 5.7+
- Modern tooling (Turbopack, pnpm)

**Payload 2.x (Legacy):**

- Express-based server
- React Client Components
- Slate rich text editor
- No Server Components
- Webpack bundler

**Migration:** See [Payload migration guide](https://payloadcms.com/docs/beta/migrate-to-3.0)

### Q: Which database should I use?

**A:** Depends on your needs:

**MongoDB (Mongoose):**

- ✅ Flexible schema
- ✅ No migrations needed
- ✅ Good for rapid prototyping
- ❌ Less rigid data structure
- **Use for:** MVPs, flexible content

**PostgreSQL (Drizzle):**

- ✅ ACID compliant
- ✅ Better for relational data
- ✅ Advanced querying
- ❌ Requires migrations
- **Use for:** Enterprise apps, strict data models

**SQLite (Drizzle):**

- ✅ Zero configuration
- ✅ File-based database
- ✅ Great for development
- ❌ Not suitable for high traffic
- **Use for:** Development, small projects

**Recommendation:** Start with MongoDB for rapid development, migrate to PostgreSQL if you need strict schemas.

### Q: Can I use Payload with an existing Next.js app?

**A:** Yes! Payload installs directly into your Next.js `/app` directory.

```typescript
// payload.config.ts at root
export default buildConfig({
  // Your collections, globals, etc.
})

// app/(payload)/admin/[[...segments]]/page.tsx
import { RootPage, generatePageMetadata } from '@payloadcms/next/views'
import config from '@payload-config'

export const generateMetadata = () => generatePageMetadata({ config })
export default RootPage

// app/api/[...slug]/route.ts
import {
  REST_GET,
  REST_POST,
  REST_DELETE,
  REST_PATCH,
} from '@payloadcms/next/routes'
export {
  REST_GET as GET,
  REST_POST as POST,
  REST_DELETE as DELETE,
  REST_PATCH as PATCH,
}
```

See [GETTING_STARTED.md](./GETTING_STARTED.md) for full setup.

### Q: What Node version should I use?

**A:** Node.js **20.9.0** or higher recommended.

```bash
# Check your version
node --version

# Install via nvm
nvm install 20
nvm use 20
```

From `.nvmrc`: `v23.11.0` is project version, but 20+ works.

## Development

### Q: How do I run the dev server for a specific test config?

**A:** Use the directory name:

```bash
# Run default config (test/_community)
pnpm run dev

# Run specific test config
pnpm run dev fields        # test/fields/config.ts
pnpm run dev uploads       # test/uploads/config.ts
pnpm run dev auth          # test/auth/config.ts
```

See [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)

### Q: TypeScript errors after changing collection config?

**A:** Regenerate types:

```bash
# Generate types for specific config
pnpm run dev:generate-types <directory>

# Or just restart dev server (auto-generates)
pnpm run dev
```

### Q: How do I clear build artifacts?

**A:** Clean and rebuild:

```bash
# Clear build outputs
pnpm clean:build

# Reinstall dependencies
rm -rf node_modules pnpm-lock.yaml
pnpm install

# Rebuild
pnpm run build:core
```

### Q: Tests are failing locally. What do I do?

**A:** Common fixes:

```bash
# 1. Ensure MongoDB is running (if using MongoDB)
pnpm docker:start

# 2. Build packages first
pnpm run build:core

# 3. Run specific test
pnpm run test:int <directory>

# 4. Clear Jest cache
pnpm run test:int --clearCache

# 5. Check Node version
node --version  # Should be 20.9.0+
```

### Q: How do I debug server-side code?

**A:** Use VS Code debugger:

1. Create `.vscode/launch.json`:

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
      "console": "integratedTerminal"
    }
  ]
}
```

2. Set breakpoints in code
3. Press F5 or click "Run and Debug"

See [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)

## Collections & Fields

### Q: How do I make a field conditionally required?

**A:** Use `admin.condition`:

```typescript
{
  name: 'publishedAt',
  type: 'date',
  required: true,
  admin: {
    condition: (data) => data.status === 'published'
  }
}
```

### Q: How do I create a custom field component?

**A:** Create React component with `useField` hook:

```typescript
// fields/CustomField.tsx
'use client'
import { useField } from '@payloadcms/ui'

export function CustomField({ path }) {
  const { value, setValue } = useField({ path })

  return (
    <input
      value={value || ''}
      onChange={(e) => setValue(e.target.value)}
    />
  )
}

// In collection config
{
  name: 'custom',
  type: 'text',
  admin: {
    components: {
      Field: '/fields/CustomField'
    }
  }
}
```

### Q: How do I validate across multiple fields?

**A:** Use collection-level `validate`:

```typescript
{
  slug: 'posts',
  fields: [...],
  validate: (data) => {
    if (data.status === 'published' && !data.publishedAt) {
      return {
        publishedAt: 'Published date required when status is published'
      }
    }
    return true
  }
}
```

### Q: Can I have relationships to multiple collections (polymorphic)?

**A:** Yes! Use array in `relationTo`:

```typescript
{
  name: 'relatedContent',
  type: 'relationship',
  relationTo: ['posts', 'pages', 'media'],
  hasMany: true
}

// Stored as:
// [
//   { relationTo: 'posts', value: 'post-id' },
//   { relationTo: 'pages', value: 'page-id' }
// ]
```

## Access Control

### Q: How do I make a collection completely public?

**A:**

```typescript
{
  slug: 'posts',
  access: {
    read: () => true,    // Anyone can read
    create: () => false, // No one can create via API
    update: () => false,
    delete: () => false
  }
}
```

### Q: How do I restrict based on user role?

**A:**

```typescript
{
  access: {
    create: ({ req }) => {
      // Only admins and editors
      return ['admin', 'editor'].includes(req.user?.role)
    },
    update: ({ req }) => {
      // Admins can update all
      if (req.user?.role === 'admin') return true

      // Others can only update their own
      return {
        author: { equals: req.user?.id }
      }
    }
  }
}
```

### Q: How do I hide fields based on user role?

**A:** Use field-level `access`:

```typescript
{
  name: 'internalNotes',
  type: 'textarea',
  access: {
    read: ({ req }) => req.user?.role === 'admin',
    update: ({ req }) => req.user?.role === 'admin'
  }
}
```

## Hooks

### Q: What's the difference between beforeValidate and beforeChange?

**A:** Hook execution order:

1. **beforeValidate** - Before validation runs
   - Use to: Transform data, set defaults
   - Data: May be incomplete

2. **Validation** - Field validation runs

3. **beforeChange** - After validation passes
   - Use to: Final transformations, async operations
   - Data: Validated and complete

4. **Database write**

5. **afterChange** - After save
   - Use to: Send notifications, update related docs
   - Data: Saved document with ID

```typescript
{
  hooks: {
    beforeValidate: [
      ({ data }) => {
        // Set slug before validation
        if (!data.slug) {
          data.slug = generateSlug(data.title)
        }
        return data
      }
    ],
    beforeChange: [
      async ({ data, req }) => {
        // Fetch related data after validation
        if (data.authorId) {
          data.authorName = await fetchAuthorName(data.authorId)
        }
        return data
      }
    ],
    afterChange: [
      async ({ doc }) => {
        // Send notification after save
        await sendNotification(`New post: ${doc.title}`)
        return doc
      }
    ]
  }
}
```

### Q: Can hooks modify data?

**A:** Yes, but you must return the modified data:

```typescript
// ✅ GOOD: Returns modified data
beforeChange: [
  ({ data }) => {
    data.slug = generateSlug(data.title)
    return data // Important!
  },
]

// ❌ BAD: Doesn't return (changes ignored)
beforeChange: [
  ({ data }) => {
    data.slug = generateSlug(data.title)
    // Missing return!
  },
]
```

### Q: How do I prevent infinite loops in hooks?

**A:** Disable hooks for nested operations:

```typescript
{
  hooks: {
    beforeChange: [
      async ({ data, req }) => {
        // Update related document
        await req.payload.update({
          collection: 'categories',
          id: data.categoryId,
          data: { postCount: await getPostCount() },
          hooks: false, // Prevent recursion
        })
        return data
      },
    ]
  }
}
```

## Authentication

### Q: How do I create an admin user?

**A:** Via seed script or admin UI:

**Seed script:**

```typescript
// seed.ts
import { getPayload } from 'payload'
import config from './payload.config'

async function seed() {
  const payload = await getPayload({ config })

  await payload.create({
    collection: 'users',
    data: {
      email: 'admin@example.com',
      password: 'secure-password',
      role: 'admin',
    },
  })
}

seed()
```

**First login:**
Navigate to `/admin` and create account on first visit (if no users exist).

### Q: How do I customize the login page?

**A:** Override login component:

```typescript
// components/Login.tsx
'use client'
export function CustomLogin() {
  return (
    <div>
      <h1>My Custom Login</h1>
      {/* Login form */}
    </div>
  )
}

// payload.config.ts
export default buildConfig({
  admin: {
    components: {
      views: {
        Login: '/components/Login'
      }
    }
  }
})
```

### Q: How do I add OAuth (Google, GitHub, etc.)?

**A:** Use custom auth strategy:

```typescript
{
  slug: 'users',
  auth: {
    strategies: [
      {
        name: 'google',
        strategy: async ({ headers, payload }) => {
          // Verify Google token
          const googleUser = await verifyGoogleToken(headers.authorization)

          // Find or create user
          let user = await payload.findOne({
            collection: 'users',
            where: { email: { equals: googleUser.email } }
          })

          if (!user) {
            user = await payload.create({
              collection: 'users',
              data: {
                email: googleUser.email,
                name: googleUser.name
              }
            })
          }

          return { user }
        }
      }
    ]
  }
}
```

## File Uploads

### Q: Where are uploaded files stored?

**A:** Depends on storage adapter:

**Local (default):**

```typescript
{
  slug: 'media',
  upload: {
    staticDir: './media'  // Files stored here
  }
}
```

**S3:**

```typescript
import { s3Storage } from '@payloadcms/storage-s3'

plugins: [
  s3Storage({
    collections: { media: true },
    bucket: process.env.S3_BUCKET,
    config: {
      credentials: {
        accessKeyId: process.env.S3_ACCESS_KEY,
        secretAccessKey: process.env.S3_SECRET_KEY,
      },
      region: process.env.S3_REGION,
    },
  }),
]
```

### Q: How do I limit file types?

**A:** Use `mimeTypes`:

```typescript
{
  slug: 'media',
  upload: {
    mimeTypes: ['image/*'],  // Only images
    // Or specific types:
    mimeTypes: ['image/png', 'image/jpeg', 'image/gif']
  }
}
```

### Q: How do I add custom image sizes?

**A:**

```typescript
{
  slug: 'media',
  upload: {
    imageSizes: [
      {
        name: 'thumbnail',
        width: 400,
        height: 300,
        position: 'center'
      },
      {
        name: 'og',  // Open Graph
        width: 1200,
        height: 630,
        position: 'center'
      }
    ]
  }
}

// Access sizes
const media = await payload.findByID({ collection: 'media', id })
console.log(media.url)  // Original
console.log(media.sizes.thumbnail.url)  // Thumbnail
console.log(media.sizes.og.url)  // OG image
```

## API & GraphQL

### Q: How do I enable GraphQL?

**A:** GraphQL is auto-enabled. Access at `/api/graphql`:

```bash
# Development playground
http://localhost:3000/api/graphql-playground

# Production endpoint
POST https://yoursite.com/api/graphql
```

### Q: Can I customize REST API routes?

**A:** Yes, via custom endpoints:

```typescript
// app/api/custom/route.ts
import { getPayload } from 'payload'
import config from '@payload-config'

export async function GET() {
  const payload = await getPayload({ config })

  const posts = await payload.find({
    collection: 'posts',
    where: { featured: { equals: true } },
  })

  return Response.json(posts)
}
```

### Q: How do I handle CORS?

**A:** Configure in `next.config.js`:

```javascript
const nextConfig = {
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          {
            key: 'Access-Control-Allow-Origin',
            value: process.env.ALLOWED_ORIGIN || '*',
          },
          {
            key: 'Access-Control-Allow-Methods',
            value: 'GET,POST,PUT,DELETE,OPTIONS',
          },
        ],
      },
    ]
  },
}
```

## Performance

### Q: My queries are slow. How do I optimize?

**A:** Several strategies:

**1. Add indexes:**

```typescript
{
  name: 'status',
  type: 'select',
  index: true  // Index for filtering
}
```

**2. Limit depth:**

```typescript
// ❌ BAD: Over-fetches
const posts = await payload.find({
  collection: 'posts',
  depth: 10, // Too deep!
})

// ✅ GOOD: Only what you need
const posts = await payload.find({
  collection: 'posts',
  depth: 1,
})
```

**3. Use pagination:**

```typescript
// ✅ GOOD
const posts = await payload.find({
  collection: 'posts',
  limit: 50,
  page: 1,
})
```

**4. Select specific fields (GraphQL):**

```graphql
query {
  Posts {
    docs {
      id
      title
      # Only fields you need
    }
  }
}
```

### Q: How do I cache API responses?

**A:** Use Next.js caching:

```typescript
// app/api/posts/route.ts
export async function GET() {
  const posts = await payload.find({ collection: 'posts' })

  return Response.json(posts, {
    headers: {
      'Cache-Control': 'public, s-maxage=60, stale-while-revalidate=120',
    },
  })
}
```

## Deployment

### Q: How do I deploy to Vercel?

**A:**

1. Push to GitHub
2. Import project on Vercel
3. Add environment variables:
   - `PAYLOAD_SECRET`
   - `DATABASE_URI`
   - `NODE_ENV=production`
4. Deploy

**Vercel Postgres:**

```typescript
import { vercelPostgresAdapter } from '@payloadcms/db-vercel-postgres'

export default buildConfig({
  db: vercelPostgresAdapter({
    pool: {
      connectionString: process.env.DATABASE_URI,
    },
  }),
})
```

### Q: What environment variables do I need in production?

**A:** Required:

```bash
# Required
PAYLOAD_SECRET=<32+ character random string>
DATABASE_URI=<database connection string>
NODE_ENV=production

# Optional but recommended
CSRF_DOMAINS=https://yoursite.com
NEXT_PUBLIC_SERVER_URL=https://yoursite.com
```

Generate secret:

```bash
openssl rand -base64 32
```

### Q: How do I run database migrations on deploy?

**A:** For Postgres/SQLite:

```bash
# In build command (package.json)
"build": "pnpm payload migrate && next build"
```

MongoDB doesn't require migrations.

## Troubleshooting

### Q: Error: "Cannot find module '@payloadcms/...'"

**A:**

```bash
# Reinstall dependencies
pnpm install

# Build core packages
pnpm run build:core
```

### Q: Error: "Database connection failed"

**A:**

1. Check MongoDB/Postgres is running:

```bash
pnpm docker:start  # For local development
```

2. Verify `DATABASE_URI` in `.env.local`
3. Test connection:

```bash
# MongoDB
mongosh $DATABASE_URI

# Postgres
psql $DATABASE_URI
```

### Q: Admin UI not loading / blank screen

**A:**

1. Check browser console for errors
2. Clear `.next` cache:

```bash
rm -rf .next
pnpm run dev
```

3. Check Node version (20.9.0+)
4. Verify build succeeded:

```bash
pnpm run build:core
```

### Q: "Module not found" errors after update

**A:**

```bash
# Clear everything and rebuild
rm -rf node_modules pnpm-lock.yaml .next
pnpm install
pnpm run build:core
pnpm run dev
```

## Migration & Upgrades

### Q: How do I migrate from Payload 2.x to 3.x?

**A:** See official [migration guide](https://payloadcms.com/docs/beta/migrate-to-3.0).

Key changes:

- Express → Next.js App Router
- Slate → Lexical
- Update all field components to Server Components
- Update auth hooks
- Update GraphQL queries

### Q: How do I update Payload to the latest version?

**A:**

```bash
# Update all Payload packages
pnpm update @payloadcms/next @payloadcms/ui payload

# Or specific version
pnpm add payload@latest @payloadcms/next@latest @payloadcms/ui@latest

# Rebuild
pnpm run build:core
```

Check [changelog](https://github.com/payloadcms/payload/releases) for breaking changes.

## Learning Resources

### Q: Where can I learn more?

**A:**

**Official Docs:**

- [payloadcms.com/docs](https://payloadcms.com/docs)
- [LLMS.txt](https://payloadcms.com/llms.txt)

**This Learning Guide:**

- [README.md](./README.md) - Start here
- [GETTING_STARTED.md](./GETTING_STARTED.md)
- [EXERCISES.md](./EXERCISES.md)
- [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)

**Community:**

- [Discord](https://discord.com/invite/payload)
- [GitHub Discussions](https://github.com/payloadcms/payload/discussions)
- [YouTube](https://www.youtube.com/@payloadcms)

### Q: How do I contribute to Payload?

**A:** See [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)

**Quick start:**

1. Find `good-first-issue` on GitHub
2. Fork and clone repository
3. Make changes and add tests
4. Submit pull request

### Q: Where can I get help?

**A:**

**Discord (fastest):**

- `#help` - General questions
- `#troubleshooting` - Bug help
- `#development` - Technical discussions

**GitHub:**

- [Issues](https://github.com/payloadcms/payload/issues) - Bugs
- [Discussions](https://github.com/payloadcms/payload/discussions) - Questions

**This Guide:**

- [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)
- [FAQ.md](./FAQ.md) - You are here!

---

**Can't find your question?**
Ask in [Discord](https://discord.com/invite/payload) or [GitHub Discussions](https://github.com/payloadcms/payload/discussions)!

_Happy building!_ 🚀
