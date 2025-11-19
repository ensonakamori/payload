# Architecture Overview

**Documented:** November 18, 2025
**Project Version:** Payload CMS v3.64.0
**Tech Stack:** [View Research](./TECH_STACK_RESEARCH.md)

---

## Table of Contents

- [Introduction](#introduction)
- [High-Level Architecture](#high-level-architecture)
- [System Layers](#system-layers)
- [Request/Response Flow](#requestresponse-flow)
- [Monorepo Architecture](#monorepo-architecture)
- [Plugin System](#plugin-system)
- [Database Abstraction](#database-abstraction)
- [Server Components Architecture](#server-components-architecture)
- [API Layer Architecture](#api-layer-architecture)
- [Admin Panel Architecture](#admin-panel-architecture)
- [Build and Deployment Architecture](#build-and-deployment-architecture)
- [Key Architectural Patterns](#key-architectural-patterns)
- [Scalability Considerations](#scalability-considerations)
- [Next Steps](#next-steps)

---

## Introduction

Payload CMS is a **headless CMS** built as a **Next.js native application** using modern **React Server Components**. Unlike traditional CMSs that are separate applications, Payload integrates directly into your Next.js app.

### What Makes Payload Unique?

🧠 **Mental Model for React Developers:**

Think of Payload as:

- **Not a separate app** (like WordPress running on Apache)
- **A Next.js package** (like `next-auth` or `next-intl`)
- **Middleware + UI** that lives inside your Next.js `/app` directory

**Traditional CMS:**

```
Your App (React)  →  API calls  →  WordPress (separate server)
```

**Payload CMS:**

```
Your Next.js App
├── /app/your-pages    ← Your frontend
└── /app/(payload)     ← Admin panel (built-in)
    └── API routes     ← REST + GraphQL APIs
```

💡 **Aha Moment:**
Payload is **embedded** in your Next.js app, not a separate service!

---

## High-Level Architecture

### The 30,000-Foot View

```
┌─────────────────────────────────────────────────────────────┐
│                     PAYLOAD CMS ECOSYSTEM                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────┐         ┌────────────────┐              │
│  │  Admin Panel   │         │  Your Frontend │              │
│  │  (React 19)    │         │  (Next.js 15)  │              │
│  └────────┬───────┘         └────────┬───────┘              │
│           │                          │                       │
│           └──────────┬───────────────┘                       │
│                      │                                       │
│           ┌──────────▼──────────┐                            │
│           │   Next.js Server    │                            │
│           │   (App Router)      │                            │
│           └──────────┬──────────┘                            │
│                      │                                       │
│           ┌──────────▼──────────┐                            │
│           │  Payload Middleware │                            │
│           │  (Core Logic)       │                            │
│           └──────────┬──────────┘                            │
│                      │                                       │
│        ┌─────────────┼─────────────┐                         │
│        │             │             │                         │
│   ┌────▼────┐   ┌────▼────┐  ┌────▼────┐                    │
│   │REST API │   │GraphQL  │  │Webhooks │                    │
│   └────┬────┘   └────┬────┘  └────┬────┘                    │
│        │             │            │                          │
│        └─────────────┼────────────┘                          │
│                      │                                       │
│           ┌──────────▼──────────┐                            │
│           │ Database Adapter    │                            │
│           │ (Abstract Layer)    │                            │
│           └──────────┬──────────┘                            │
│                      │                                       │
│        ┌─────────────┼─────────────┐                         │
│        │             │             │                         │
│   ┌────▼────┐   ┌────▼────┐  ┌────▼────┐                    │
│   │MongoDB  │   │Postgres │  │SQLite   │                    │
│   └─────────┘   └─────────┘  └─────────┘                    │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

🎯 **Remember This:**
**Client → Next.js → Payload → Database Adapter → Database**

---

## System Layers

Payload follows a **layered architecture**. Understanding these layers is key to becoming a full-stack developer.

### Layer 1: Presentation Layer (UI)

**Location:** [`packages/ui/src/`](../../packages/ui/src/)

**Technologies:**

- React 19 (with Server Components)
- React Compiler for optimization
- SCSS for styling
- Next.js App Router for routing

**Components:**

```
@payloadcms/ui
├── elements/           ← Reusable UI components
│   ├── Button/
│   ├── Table/
│   └── Modal/
├── forms/              ← Form components
│   ├── Form/
│   ├── FieldTypes/
│   └── RenderFields/
├── fields/             ← Field-specific components
│   ├── Text/
│   ├── RichText/
│   └── Upload/
└── views/              ← Page-level views
    ├── List/
    ├── Edit/
    └── Account/
```

🧠 **Mental Model:**
The UI layer is like **React components** you're familiar with, but:

- Many are **Server Components** (run on server, not client)
- Optimized by **React Compiler** (automatic memoization)
- Use **Server Actions** instead of REST calls

🌉 **Bridge from React:**

```typescript
// ❌ Old pattern (Client Component with useEffect)
'use client'
export function PostsList() {
  const [posts, setPosts] = useState([])
  useEffect(() => {
    fetch('/api/posts').then(setPosts)
  }, [])
  return <div>{posts.map(p => <Post {...p} />)}</div>
}

// ✅ New pattern (Server Component)
export async function PostsList() {
  const posts = await payload.find({ collection: 'posts' })
  return <div>{posts.docs.map(p => <Post {...p} />)}</div>
}
```

**Key Files:**

- [`packages/ui/src/exports/client/index.ts`](../../packages/ui/src/exports/client/index.ts#L1-L50) - Client-side exports
- [`packages/ui/src/exports/rsc/index.ts`](../../packages/ui/src/exports/rsc/) - Server Component exports

---

### Layer 2: Application Layer (Core Logic)

**Location:** [`packages/payload/src/`](../../packages/payload/src/)

**Responsibilities:**

- Business logic
- Data validation
- Access control
- Lifecycle hooks
- Authentication

**Core Modules:**

```
payload
├── collections/        ← Collection operations
│   ├── operations/
│   │   ├── find.ts
│   │   ├── create.ts
│   │   ├── update.ts
│   │   └── delete.ts
│   └── config/
├── globals/            ← Global operations
├── auth/               ← Authentication
│   ├── operations/
│   │   ├── login.ts
│   │   ├── logout.ts
│   │   └── forgotPassword.ts
│   └── strategies/
├── fields/             ← Field type system
│   ├── validations/
│   └── hooks/
└── admin/              ← Admin functionality
```

🧠 **Mental Model:**
The Application Layer is like **your backend API routes**, but abstracted:

- Collections = Models/Controllers combined
- Fields = Schema + Validation + UI
- Hooks = Middleware/Lifecycle events

🎯 **Remember This - CRUD Operations:**
**C**reate, **R**ead, **U**pdate, **D**elete

Every collection has these operations built-in!

**Example from code ([`packages/payload/src/index.ts`](../../packages/payload/src/index.ts#L1-L50)):**

```typescript
// The Payload class exposes all operations
export class Payload {
  create(args: CreateArgs) {} // ← Create
  find(args: FindArgs) {} // ← Read (many)
  findByID(args) {} // ← Read (one)
  update(args) {} // ← Update
  delete(args) {} // ← Delete
  // ... and many more
}
```

---

### Layer 3: Data Access Layer (Database Adapter)

**Location:** [`packages/db-*`](../../packages/)

**Available Adapters:**

- `@payloadcms/db-mongodb` - MongoDB adapter
- `@payloadcms/db-postgres` - PostgreSQL adapter (uses Drizzle ORM)
- `@payloadcms/db-sqlite` - SQLite adapter
- `@payloadcms/db-vercel-postgres` - Vercel Postgres
- `@payloadcms/db-d1-sqlite` - Cloudflare D1 SQLite

🧠 **Mental Model - Adapter Pattern:**

```
Payload Core (doesn't know about databases)
        ↓
Database Adapter Interface (contract)
        ↓
Specific Implementation (MongoDB, Postgres, etc.)
```

This is the **Adapter Pattern** - one of the most important design patterns!

**Why?**

- Payload code is **database-agnostic**
- You can **switch databases** without changing Payload code
- Each adapter implements the same **interface**

🌉 **Bridge from React:**
Think of it like React's **Renderer** pattern:

- `react` package (core) doesn't know about DOM
- `react-dom` renders to browser
- `react-native` renders to mobile
- `react-three-fiber` renders to 3D

Similarly:

- `payload` package (core) doesn't know about databases
- `db-mongodb` "renders" to MongoDB
- `db-postgres` "renders" to PostgreSQL

**Interface Example:**

```typescript
// All adapters implement this
interface DatabaseAdapter {
  create(collection: string, data: any): Promise<any>
  find(collection: string, query: any): Promise<any>
  update(collection: string, id: string, data: any): Promise<any>
  delete(collection: string, id: string): Promise<any>
  // ... more methods
}
```

---

### Layer 4: Infrastructure Layer (Database)

**Not part of Payload code** - you bring your own database!

**Options:**

- **MongoDB** (Document database) - NoSQL
- **PostgreSQL** (Relational database) - SQL
- **SQLite** (File-based database) - SQL
- **Others** via custom adapters

🧠 **Mental Model - SQL vs NoSQL:**

**NoSQL (MongoDB):**

```json
// Document = JavaScript object
{
  "_id": "123",
  "title": "Hello World",
  "author": {
    "name": "John",
    "email": "john@example.com"
  }
}
```

**SQL (PostgreSQL):**

```sql
-- Normalized tables
posts: id | title        | author_id
       123 | Hello World | 456

users: id  | name | email
       456 | John | john@example.com
```

🎯 **Remember This:**

- **MongoDB** = Flexible, JavaScript-friendly, denormalized
- **PostgreSQL** = Structured, powerful queries, normalized

**Which to choose?**

- **MongoDB:** Rapid development, flexible schema
- **PostgreSQL:** Complex queries, data integrity, relations

---

## Request/Response Flow

Let's trace a request through the entire system. This is **crucial** for full-stack understanding!

### Flow 1: Admin Panel - Create a Post

**User Action:** Click "Create New Post" in admin panel

```
1. Browser (http://localhost:3000/admin/collections/posts/create)
   │
   ↓
2. Next.js Router (App Router)
   │ Routes to: /app/(payload)/admin/collections/[slug]/create
   ↓
3. Server Component: CreateView
   │ Fetches: - Collection config
   │          - Field schemas
   │          - User permissions
   ↓
4. Render: Form with all fields (Server Component)
   │ Sends HTML to browser
   ↓
5. Browser: User fills form, clicks "Save"
   │ Submits: Form data via Server Action
   ↓
6. Server Action: handleCreate
   │ Calls: payload.create({ collection: 'posts', data: {...} })
   ↓
7. Payload Core
   │ 1. Validates data (field validations)
   │ 2. Runs beforeValidate hooks
   │ 3. Runs beforeChange hooks
   │ 4. Checks access control
   │ 5. Calls database adapter
   ↓
8. Database Adapter (MongoDB example)
   │ Transforms Payload data → MongoDB document
   │ Calls: db.collection('posts').insertOne(...)
   ↓
9. MongoDB Database
   │ Stores document
   │ Returns: Inserted document with _id
   ↓
10. Response flows back up
    │ Database → Adapter → Payload → Server Action
    ↓
11. Next.js: Revalidates cache, redirects to edit page
    ↓
12. Browser: Shows success message, navigates to /admin/collections/posts/{id}
```

🎯 **Remember This Flow:**
**Browser → Next.js → Payload → Adapter → Database**

⚠️ **Common Pitfall:**
In Next.js 15, **Server Actions replace API routes** for mutations! Don't look for `/api/posts` endpoints.

---

### Flow 2: REST API - Get All Posts

**External Request:** `GET http://localhost:3000/api/posts`

```
1. HTTP GET /api/posts
   │
   ↓
2. Next.js: Route Handler
   │ File: /app/api/posts/route.ts (generated by Payload)
   ↓
3. Payload REST Handler
   │ Parses: - Query parameters (?limit=10&page=1)
   │         - Headers (Authorization)
   │         - Cookies (session)
   ↓
4. Payload Core: find() operation
   │ 1. Authenticates user (if auth collection)
   │ 2. Checks read access control
   │ 3. Runs beforeRead hooks
   │ 4. Builds query from params
   ↓
5. Database Adapter
   │ Transforms: Payload query → Database query
   │ Example: { limit: 10, page: 1 } → MongoDB .find().limit(10).skip(0)
   ↓
6. Database
   │ Executes query
   │ Returns: Array of documents
   ↓
7. Response flows back
   │ Database → Adapter → Payload Core
   │ Runs: afterRead hooks
   │ Formats: PaginatedDocs structure
   ↓
8. REST Handler
   │ Converts to JSON
   │ Sets headers (Content-Type, Cache-Control)
   ↓
9. HTTP Response
   {
     "docs": [...],      ← The actual posts
     "totalDocs": 42,    ← Total count
     "limit": 10,        ← Items per page
     "totalPages": 5,    ← Total pages
     "page": 1,          ← Current page
     "pagingCounter": 1,
     "hasPrevPage": false,
     "hasNextPage": true,
     "prevPage": null,
     "nextPage": 2
   }
```

🧠 **Mental Model - Pagination:**
Think of it like **Instagram feed**:

- `limit` = How many posts per scroll
- `page` = Which "page" you're on
- `totalDocs` = Total posts available
- `hasNextPage` = Can scroll more?

---

### Flow 3: GraphQL Query

**GraphQL Query:**

```graphql
query {
  Posts(limit: 5) {
    docs {
      id
      title
      author {
        name
      }
    }
  }
}
```

```
1. POST /api/graphql
   │ Body: { query: "query { Posts { ... } }" }
   ↓
2. GraphQL Server (graphql-http)
   │ Parses query
   │ Validates against schema
   ↓
3. GraphQL Resolvers (Payload-generated)
   │ Posts resolver calls: payload.find({ collection: 'posts', limit: 5 })
   ↓
4. Payload Core (same as REST flow)
   │ Access control → beforeRead → Database
   ↓
5. Database → Returns docs
   ↓
6. Payload → Runs afterRead hooks
   ↓
7. GraphQL Resolver
   │ Returns data in GraphQL format
   │ Includes relationships (populates author)
   ↓
8. Response
   {
     "data": {
       "Posts": {
         "docs": [
           { "id": "1", "title": "...", "author": { "name": "John" } }
         ]
       }
     }
   }
```

🌉 **Bridge from REST:**

```
REST:  GET /api/posts?limit=5&depth=1
       ↓ Returns everything

GraphQL:  You specify exactly what fields you want
          ↓ Returns only requested fields (more efficient!)
```

---

## Monorepo Architecture

Payload uses a **monorepo** to manage 45+ packages in one repository.

### Why Monorepo?

🧠 **Mental Model:**

```
Monorepo = City (one connected system)
Multi-repo = Separate towns (independent systems)
```

**Benefits:**

1. **Shared Code** - DRY (Don't Repeat Yourself)
2. **Atomic Changes** - Change multiple packages in one commit
3. **Easier Testing** - Test interactions between packages
4. **Single Source of Truth** - All code in one place
5. **Consistent Tooling** - ESLint, TypeScript, tests configured once

### Monorepo Tools Used

**pnpm Workspaces** ([`pnpm-workspace.yaml`](../../pnpm-workspace.yaml))

```yaml
packages:
  - 'packages/*'
  - 'test/*'
```

This tells pnpm: "These directories contain packages!"

**Turborepo** ([`turbo.json`](../../turbo.json))

- Builds packages in correct order
- Caches builds (second build is instant!)
- Runs tasks in parallel

🎯 **Remember This:**

- **pnpm** = Package manager (installs dependencies)
- **Turborepo** = Build orchestrator (builds packages)

### Package Dependencies

Packages depend on each other:

```
@payloadcms/next
├── depends on → payload
└── depends on → @payloadcms/ui

@payloadcms/ui
└── depends on → payload

@payloadcms/db-mongodb
└── depends on → payload

@payloadcms/richtext-lexical
└── depends on → payload
```

🧠 **Mental Model - Dependency Graph:**

```
             payload (core)
                 ↑
        ┌────────┼────────┐
        │        │        │
       ui     db-*    richtext-*
        │
     ┌──┴──┐
  next   templates
```

**Build order:**

1. `payload` (has no dependencies)
2. `ui`, `db-*`, `richtext-*` (depend on payload)
3. `next` (depends on ui)

Turborepo figures this out automatically!

---

## Plugin System

Payload's **plugin system** is one of its most powerful features.

### What is a Plugin?

🧠 **Mental Model:**
A plugin is a **function that modifies Payload config** before initialization.

```typescript
// Simplified plugin
const myPlugin = (incomingConfig) => {
  // Modify config
  incomingConfig.collections.push(newCollection)
  return incomingConfig
}

// Usage
export default buildConfig({
  plugins: [myPlugin()],
  collections: [...]
})
```

### Plugin Architecture

```
User Config
    ↓
Plugin 1 (transforms config)
    ↓
Plugin 2 (transforms config)
    ↓
Plugin 3 (transforms config)
    ↓
Final Config → Payload Initialization
```

**Example: SEO Plugin**

```typescript
// packages/plugin-seo/src/index.ts
export const seoPlugin = (pluginOptions) => (config) => {
  // Adds SEO fields to collections
  return {
    ...config,
    collections: config.collections.map((collection) => ({
      ...collection,
      fields: [
        ...collection.fields,
        {
          name: 'meta',
          type: 'group',
          fields: [
            { name: 'title', type: 'text' },
            { name: 'description', type: 'textarea' },
            { name: 'image', type: 'upload' },
          ],
        },
      ],
    })),
  }
}
```

🎯 **Remember This:**
Plugins = **Config transformers** (not runtime code)

### Built-In Plugins

**Storage Plugins:**

- `@payloadcms/storage-s3` - AWS S3
- `@payloadcms/storage-azure` - Azure Blob Storage
- `@payloadcms/storage-gcs` - Google Cloud Storage
- `@payloadcms/storage-uploadthing` - Uploadthing
- `@payloadcms/storage-vercel-blob` - Vercel Blob

**Feature Plugins:**

- `@payloadcms/plugin-seo` - SEO meta fields
- `@payloadcms/plugin-search` - Full-text search
- `@payloadcms/plugin-stripe` - Stripe integration
- `@payloadcms/plugin-form-builder` - Form builder
- `@payloadcms/plugin-redirects` - URL redirects
- `@payloadcms/plugin-nested-docs` - Nested document structure
- `@payloadcms/plugin-sentry` - Error tracking

---

## Database Abstraction

The database abstraction layer is **key to understanding** Payload's flexibility.

### The BaseDatabaseAdapter Interface

All database adapters implement this interface:

```typescript
// Simplified from packages/payload/src/database/types.ts
interface BaseDatabaseAdapter {
  // Connection
  connect(): Promise<void>

  // CRUD operations
  create(args: {
    collection: string
    data: Record<string, unknown>
  }): Promise<any>

  find(args: {
    collection: string
    where: Where
    limit: number
    page: number
  }): Promise<PaginatedDocs>

  findOne(args: { collection: string; where: Where }): Promise<any>

  update(args: {
    collection: string
    where: Where
    data: Record<string, unknown>
  }): Promise<any>

  delete(args: { collection: string; where: Where }): Promise<void>

  // Transactions
  beginTransaction(): Promise<void>
  commitTransaction(): Promise<void>
  rollbackTransaction(): Promise<void>

  // Schema
  createMigration(): Promise<void>
  migrate(): Promise<void>

  // More methods...
}
```

🧠 **Mental Model - Interface:**
An interface is like a **contract** or **blueprint**.

All adapters must implement these methods, but **how** they do it is up to them!

### MongoDB Adapter

**Uses:** Native MongoDB driver

```typescript
// Simplified from packages/db-mongodb/src/index.ts
class MongoAdapter implements BaseDatabaseAdapter {
  async create({ collection, data }) {
    const result = await this.db
      .collection(collection)
      .insertOne(data)
    return result
  }

  async find({ collection, where, limit, page }) {
    const query = this.buildMongoQuery(where)
    const docs = await this.db
      .collection(collection)
      .find(query)
      .limit(limit)
      .skip((page - 1) * limit)
      .toArray()
    return { docs, totalDocs: ... }
  }
}
```

### PostgreSQL Adapter

**Uses:** Drizzle ORM

```typescript
// Simplified from packages/db-postgres/src/index.ts
class PostgresAdapter implements BaseDatabaseAdapter {
  async create({ collection, data }) {
    const result = await this.drizzle
      .insert(tables[collection])
      .values(data)
      .returning()
    return result[0]
  }

  async find({ collection, where, limit, page }) {
    const query = this.buildDrizzleQuery(where)
    const docs = await this.drizzle
      .select()
      .from(tables[collection])
      .where(query)
      .limit(limit)
      .offset((page - 1) * limit)
    return { docs, totalDocs: ... }
  }
}
```

🌉 **Bridge - Why This Matters:**

```typescript
// Payload code doesn't know about databases!
await payload.find({ collection: 'posts' })

// ↓ Adapter translates to:

// MongoDB
db.collection('posts').find()

// OR PostgreSQL
drizzle.select().from(posts)
```

**Same Payload code, different databases!**

---

## Server Components Architecture

React Server Components (RSC) are **fundamental** to Payload's architecture.

### What are Server Components?

🧠 **Mental Model:**

```
Client Component (old way):
1. Send JavaScript to browser
2. Fetch data in browser
3. Render in browser

Server Component (new way):
1. Fetch data on server
2. Render on server
3. Send HTML to browser (no JavaScript!)
```

### RSC in Payload

**Admin Panel is mostly Server Components!**

```typescript
// packages/ui/src/views/List/index.tsx
// This is a Server Component (default in Next.js 15)

export async function ListView({ collectionSlug }) {
  // ✅ Fetch data on server
  const docs = await payload.find({
    collection: collectionSlug,
    limit: 10
  })

  // ✅ Render on server
  return (
    <Table>
      {docs.map(doc => <Row key={doc.id} {...doc} />)}
    </Table>
  )
}
```

🎯 **Remember This:**

- **Server Component** = Runs on server, sends HTML
- **Client Component** = Runs in browser, sends JS

**When to use each:**

```typescript
// ✅ Server Component (default)
// - Fetch data
// - Access database
// - Access file system
// - No interactivity needed

// ✅ Client Component ('use client')
// - Interactivity (onClick, onChange)
// - Browser APIs (localStorage, window)
// - State (useState, useReducer)
// - Effects (useEffect)
```

### Server Actions

**Server Actions** replace API route handlers:

```typescript
// Old way (API route)
// /app/api/posts/route.ts
export async function POST(req) {
  const data = await req.json()
  const post = await payload.create({
    collection: 'posts',
    data
  })
  return Response.json(post)
}

// ✅ New way (Server Action)
'use server'
export async function createPost(formData) {
  const post = await payload.create({
    collection: 'posts',
    data: Object.fromEntries(formData)
  })
  revalidatePath('/posts')
  return post
}

// Used in component
<form action={createPost}>
  <input name="title" />
  <button>Save</button>
</form>
```

💡 **Aha Moment:**
Server Actions = Functions that run on server, called from client **without API routes**!

---

## API Layer Architecture

Payload provides **two APIs** out of the box:

### 1. REST API

**Auto-generated endpoints:**

```
Collections:
GET    /api/{collection}           - List all
POST   /api/{collection}           - Create
GET    /api/{collection}/:id       - Get by ID
PATCH  /api/{collection}/:id       - Update
DELETE /api/{collection}/:id       - Delete

Globals:
GET    /api/globals/{global}       - Get global
POST   /api/globals/{global}       - Update global

Auth (for auth-enabled collections):
POST   /api/{collection}/login     - Login
POST   /api/{collection}/logout    - Logout
POST   /api/{collection}/refresh   - Refresh token
POST   /api/{collection}/me        - Get current user
POST   /api/{collection}/forgot-password
POST   /api/{collection}/reset-password
```

**Example:**

```bash
# Get all posts
GET /api/posts?limit=10&page=1&sort=-createdAt

# Create post
POST /api/posts
{
  "title": "Hello World",
  "content": "..."
}

# Update post
PATCH /api/posts/123
{
  "title": "Updated Title"
}
```

### 2. GraphQL API

**Single endpoint:**

```
POST /api/graphql
```

**Auto-generated schema** from your collections!

```graphql
# Query
query {
  Posts(limit: 10, where: { status: { equals: "published" } }) {
    docs {
      id
      title
      author {
        name
        email
      }
    }
    totalDocs
  }
}

# Mutation
mutation {
  createPost(data: { title: "New Post", content: "Content here" }) {
    id
    title
  }
}
```

🌉 **Bridge - REST vs GraphQL:**

```
REST: Multiple endpoints, fixed responses
  GET /posts → Returns ALL fields
  GET /users → Separate request

GraphQL: One endpoint, flexible queries
  POST /graphql
  {
    posts { id, title }    ← Only these fields
    users { name }         ← In same request!
  }
```

---

## Admin Panel Architecture

The admin panel is a **Next.js App Router application** built with Server Components.

### Admin Routes

```
/admin
├── /dashboard                    ← Dashboard view
├── /collections
│   ├── /{collectionSlug}         ← List view
│   ├── /{collectionSlug}/create  ← Create view
│   └── /{collectionSlug}/{id}    ← Edit view
├── /globals
│   └── /{globalSlug}             ← Global edit view
└── /account                      ← User account
```

### View Components

**List View** ([`packages/ui/src/views/List/`](../../packages/ui/src/views/List/))

- Displays table of documents
- Pagination, sorting, filtering
- Bulk actions

**Edit View** ([`packages/ui/src/views/Edit/`](../../packages/ui/src/views/Edit/))

- Form with all fields
- Save, auto-save
- Version history

**Create View**

- Same as Edit but for new documents

🧠 **Mental Model - Views:**

```
Views = Pages in admin panel
Each view is a Server Component that:
1. Fetches data
2. Renders UI
3. Handles actions via Server Actions
```

---

## Build and Deployment Architecture

### Development Build

```bash
pnpm run build:core
```

**What happens:**

1. **Turborepo** reads `turbo.json`
2. Determines build order (dependency graph)
3. For each package:
   - **SWC** compiles TypeScript → JavaScript
   - **TypeScript** generates `.d.ts` files
   - **esbuild** bundles (for some packages)
4. Outputs to each package's `dist/` directory

### Production Build

```bash
# Build Payload packages
pnpm run build:all

# Build your Next.js app
next build
```

**Deployment:**

- Deploy to Vercel, Netlify, or any Node.js host
- Database connection string via environment variables
- File uploads to S3/storage service

---

## Key Architectural Patterns

### 1. Adapter Pattern

**Used for:** Database abstraction

```
Client Code → Interface → Concrete Implementation
Payload     → BaseAdapter → MongoAdapter | PostgresAdapter
```

### 2. Plugin Pattern

**Used for:** Extensibility

```
Config → Plugin 1 → Plugin 2 → Final Config
```

### 3. Middleware Pattern

**Used for:** Request processing

```
Request → Middleware 1 → Middleware 2 → Handler
```

### 4. Repository Pattern

**Used for:** Data access

```
Business Logic → Repository → Database Adapter → Database
```

### 5. Factory Pattern

**Used for:** Creating instances

```typescript
// Field factory creates field instances
fieldFactory.create({ type: 'text', name: 'title' })
```

---

## Scalability Considerations

### Horizontal Scaling

Payload can run on multiple servers:

```
          Load Balancer
         /      |      \
    Server 1  Server 2  Server 3
         \      |      /
          Database (shared)
```

### Caching Strategies

1. **Next.js Cache** - Page and data caching
2. **Redis** (optional) - Session and data caching
3. **CDN** - Static assets and media

### Performance Optimizations

- **React Compiler** - Automatic memoization
- **Server Components** - Reduced JavaScript
- **Streaming** - Progressive rendering
- **Code Splitting** - Load only what's needed

---

## Next Steps

Now that you understand the architecture:

1. **Read:** [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) - Navigate the codebase
2. **Read:** [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md) - Trace requests in detail
3. **Read:** [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - Deep dive into UI
4. **Read:** [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Deep dive into Core
5. **Explore:** Pick a feature and trace it through all layers

---

**Congratulations!** You now understand Payload's architecture. 🎉

_You're well on your way to becoming a full-stack architect!_
