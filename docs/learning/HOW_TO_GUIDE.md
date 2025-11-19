# How-To Guide

**Documented:** November 18, 2025

Step-by-step guides for common tasks.

## Add a New Collection

1. Create collection config:
```typescript
// collections/Posts.ts
export const Posts = {
  slug: 'posts',
  fields: [
    { name: 'title', type: 'text', required: true },
    { name: 'content', type: 'richText' }
  ]
}
```

2. Add to payload config:
```typescript
export default buildConfig({
  collections: [Posts]
})
```

3. Restart dev server

## Add a New Field Type

1. Create field component:
```typescript
// fields/Custom/Component.tsx
'use client'
export function CustomField({ value, onChange }) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />
}
```

2. Use in collection:
```typescript
{
  name: 'custom',
  type: 'text',
  admin: {
    components: {
      Field: '/fields/Custom/Component'
    }
  }
}
```

## Add Custom API Endpoint

1. Create route handler:
```typescript
// app/api/custom/route.ts
import { getPayload } from 'payload'
import config from '@payload-config'

export async function GET() {
  const payload = await getPayload({ config })
  const data = await payload.find({ collection: 'posts' })
  return Response.json(data)
}
```

## Add Access Control

```typescript
{
  slug: 'posts',
  access: {
    create: ({ req }) => {
      return !!req.user
    },
    update: ({ req, data }) => {
      if (req.user?.role === 'admin') return true
      return req.user?.id === data.author
    }
  }
}
```

## Add Hooks

```typescript
{
  slug: 'posts',
  hooks: {
    beforeChange: [
      ({ data, operation }) => {
        if (operation === 'create') {
          data.author = req.user.id
          data.publishedAt = new Date()
        }
        return data
      }
    ]
  }
}
```

## Generate Types

```bash
pnpm payload generate:types
```

## Run Migrations

```bash
# Create migration
pnpm payload migrate:create

# Run migrations
pnpm payload migrate
```

## Customize Admin UI

```typescript
export default buildConfig({
  admin: {
    components: {
      beforeDashboard: ['/components/CustomWidget'],
      afterNavLinks: ['/components/CustomLink']
    }
  }
})
```

---

*Get things done!*
