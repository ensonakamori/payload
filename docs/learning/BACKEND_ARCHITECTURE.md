# Backend Architecture Deep Dive

**Documented:** November 18, 2025

Complete guide to Payload's core backend logic.

## Core Concepts

### Collections

**Definition:** A collection is like a database table/model.

```typescript
{
  slug: 'posts',  // Unique identifier
  fields: [...],  // Schema definition
  access: {...},  // Permissions
  hooks: {...}    // Lifecycle hooks
}
```

**Auto-generates:**
- REST API endpoints (`/api/posts`)
- GraphQL types and resolvers
- TypeScript types
- Admin UI (list/edit/create views)

### Globals

**Single documents** (site settings, menus, etc.):

```typescript
{
  slug: 'settings',
  fields: [
    { name: 'siteName', type: 'text' },
    { name: 'logo', type: 'upload' }
  ]
}
```

**Access:**
```typescript
const settings = await payload.findGlobal({ slug: 'settings' })
```

## Field Types System

**Built-in field types:**
- **text** - Single-line text
- **textarea** - Multi-line text
- **richText** - WYSIWYG editor
- **number** - Numeric values
- **email** - Email with validation
- **select** - Dropdown options
- **radio** - Radio buttons
- **checkbox** - Boolean
- **date** - Date picker
- **upload** - File uploads
- **relationship** - References to other collections
- **array** - Repeatable groups
- **blocks** - Flexible content blocks
- **group** - Nested fields

**Custom fields:**
```typescript
const customField = {
  name: 'custom',
  type: 'text',
  admin: {
    components: {
      Field: '/fields/Custom'
    }
  }
}
```

## Access Control

**Granular permissions:**

```typescript
{
  slug: 'posts',
  access: {
    read: true,  // Public
    create: ({ req }) => !!req.user,  // Authenticated
    update: ({ req, data }) => {
      // Own posts or admin
      return req.user?.id === data.author || req.user?.role === 'admin'
    },
    delete: ({ req }) => req.user?.role === 'admin'
  }
}
```

**Field-level access:**
```typescript
{
  name: 'internalNotes',
  type: 'textarea',
  access: {
    read: ({ req }) => req.user?.role === 'admin'
  }
}
```

## Hooks System

**Lifecycle hooks:**

```typescript
{
  slug: 'posts',
  hooks: {
    beforeValidate: [
      ({ data }) => {
        // Run before validation
        return data
      }
    ],
    beforeChange: [
      ({ data, operation }) => {
        // Run before create/update
        if (operation === 'create') {
          data.author = req.user.id
        }
        return data
      }
    ],
    afterChange: [
      ({ doc, operation }) => {
        // Run after create/update
        // Send notification, clear cache, etc.
      }
    ],
    beforeRead: [
      ({ doc }) => {
        // Modify doc before returning
        return doc
      }
    ],
    afterRead: [
      ({ doc }) => {
        // Transform data
        doc.computedField = calculateValue(doc)
        return doc
      }
    ]
  }
}
```

## Authentication

**Auth-enabled collections:**

```typescript
{
  slug: 'users',
  auth: true,  // Enables authentication
  fields: [
    { name: 'name', type: 'text' },
    { name: 'role', type: 'select', options: ['user', 'admin'] }
  ]
}
```

**Auto-generates:**
- Login endpoint
- Password hashing
- JWT tokens
- Session management

**Strategies:**
- Local (email/password)
- OAuth (via plugins)
- API keys
- Custom strategies

## Validation

**Built-in validators:**

```typescript
{
  name: 'email',
  type: 'email',  // Auto-validates email format
  required: true,
  unique: true
}
```

**Custom validation:**

```typescript
{
  name: 'age',
  type: 'number',
  validate: (value) => {
    if (value < 18) {
      return 'Must be 18 or older'
    }
    return true
  }
}
```

## Operations

**All CRUD operations available:**

```typescript
// Create
const post = await payload.create({
  collection: 'posts',
  data: { title: 'Hello' }
})

// Read (many)
const posts = await payload.find({
  collection: 'posts',
  where: { status: { equals: 'published' } },
  limit: 10
})

// Read (one)
const post = await payload.findByID({
  collection: 'posts',
  id: '123'
})

// Update
await payload.update({
  collection: 'posts',
  id: '123',
  data: { title: 'Updated' }
})

// Delete
await payload.delete({
  collection: 'posts',
  id: '123'
})
```

## Query Language

**Where conditions:**

```typescript
{
  and: [
    { status: { equals: 'published' } },
    { title: { contains: 'Next.js' } },
    { createdAt: { greater_than: '2025-01-01' } }
  ]
}
```

**Operators:**
- `equals`, `not_equals`
- `greater_than`, `less_than`
- `contains`, `like`
- `in`, `not_in`
- `exists`

## Versioning

**Track document history:**

```typescript
{
  slug: 'posts',
  versions: {
    drafts: true,
    maxPerDoc: 50
  }
}
```

**Access versions:**
```typescript
const versions = await payload.findVersions({
  collection: 'posts',
  where: { parent: { equals: postId } }
})
```

## File Uploads

**Upload field type:**

```typescript
{
  slug: 'media',
  upload: {
    staticDir: 'media',
    imageSizes: [
      { name: 'thumbnail', width: 400 },
      { name: 'card', width: 768 },
      { name: 'full', width: 1920 }
    ]
  }
}
```

**Storage adapters:**
- Local filesystem (default)
- S3, Azure, GCS (via plugins)

## Email

**Configure email adapter:**

```typescript
import { nodemailerAdapter } from '@payloadcms/email-nodemailer'

export default buildConfig({
  email: nodemailerAdapter({
    transportOptions: {
      host: process.env.SMTP_HOST,
      port: 587,
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS
      }
    }
  })
})
```

**Send emails:**
```typescript
await payload.sendEmail({
  to: 'user@example.com',
  subject: 'Welcome!',
  html: '<h1>Welcome to our site!</h1>'
})
```

## GraphQL

**Auto-generated schema:**

```graphql
type Post {
  id: ID!
  title: String
  content: String
  status: PostStatus
  author: User
  createdAt: DateTime
  updatedAt: DateTime
}

type Query {
  Post(id: ID!): Post
  Posts(where: PostWhereInput, limit: Int): PostsConnection
}

type Mutation {
  createPost(data: PostInput!): Post
  updatePost(id: ID!, data: PostInput!): Post
  deletePost(id: ID!): Post
}
```

## Localization

**Multi-language support:**

```typescript
{
  slug: 'posts',
  fields: [
    {
      name: 'title',
      type: 'text',
      localized: true  // Supports multiple languages
    }
  ]
}
```

**Access localized content:**
```typescript
await payload.find({
  collection: 'posts',
  locale: 'es'  // Spanish
})
```

## Jobs/Background Tasks

**Scheduled tasks:**

```typescript
{
  slug: 'posts',
  hooks: {
    afterChange: [
      ({ doc }) => {
        // Queue background job
        queue.add('process-post', { postId: doc.id })
      }
    ]
  }
}
```

## Next Steps

- [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)
- [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)

---

*Master Payload's backend!*
