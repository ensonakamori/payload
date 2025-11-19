# Database Architecture Deep Dive

**Documented:** November 18, 2025

Complete guide to Payload's database layer and adapters.

## Database Abstraction Layer

**Key Principle:** Payload doesn't depend on any specific database.

```
Payload Core
    ↓
BaseDatabaseAdapter (interface)
    ↓
┌──────────┬──────────┬──────────┐
MongoDB   Postgres   SQLite   Custom
```

## Drizzle ORM (PostgreSQL/SQLite)

**Why Drizzle?**
- Type-safe queries
- SQL-like syntax
- Lightweight
- Migration support

**Schema Example:**
```typescript
import { pgTable, serial, text, timestamp } from 'drizzle-orm/pg-core'

const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content'),
  createdAt: timestamp('created_at').defaultNow()
})
```

**Queries:**
```typescript
// Select
await db.select().from(posts).where(eq(posts.status, 'published'))

// Insert
await db.insert(posts).values({ title: 'Hello' })

// Update
await db.update(posts).set({ title: 'Updated' }).where(eq(posts.id, 1))

// Delete
await db.delete(posts).where(eq(posts.id, 1))
```

## MongoDB (Mongoose)

**Document-based:**
```typescript
const postSchema = new Schema({
  title: String,
  content: String,
  author: { type: ObjectId, ref: 'users' },
  createdAt: { type: Date, default: Date.now }
})
```

**Operations:**
```typescript
// Find
await Post.find({ status: 'published' })

// Create
await Post.create({ title: 'Hello' })

// Update
await Post.updateOne({ _id: id }, { title: 'Updated' })

// Delete
await Post.deleteOne({ _id: id })
```

## Schema Generation

Payload auto-generates database schemas from collections:

```typescript
// Collection config
{
  slug: 'posts',
  fields: [
    { name: 'title', type: 'text', required: true },
    { name: 'status', type: 'select', options: ['draft', 'published'] }
  ]
}

// ↓ Generates:

// MongoDB schema
{
  title: { type: String, required: true },
  status: { type: String, enum: ['draft', 'published'] },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
}

// PostgreSQL schema (Drizzle)
pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  status: text('status').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow()
})
```

## Migrations

**Generate migration:**
```bash
pnpm payload migrate:create
```

**Run migrations:**
```bash
pnpm payload migrate
```

**Migration file:**
```typescript
export async function up({ payload }) {
  // Run migration
  await payload.db.drizzle.execute(sql`
    ALTER TABLE posts ADD COLUMN featured BOOLEAN DEFAULT false
  `)
}

export async function down({ payload }) {
  // Rollback
  await payload.db.drizzle.execute(sql`
    ALTER TABLE posts DROP COLUMN featured
  `)
}
```

## Relationships

**One-to-Many:**
```typescript
{
  name: 'author',
  type: 'relationship',
  relationTo: 'users',
  required: true
}
```

**Many-to-Many:**
```typescript
{
  name: 'categories',
  type: 'relationship',
  relationTo: 'categories',
  hasMany: true
}
```

**Polymorphic:**
```typescript
{
  name: 'relatedDoc',
  type: 'relationship',
  relationTo: ['posts', 'pages']  // Can reference multiple collections
}
```

## Indexing

**Auto-indexes:**
- `_id` (primary key)
- Unique fields
- Relationship fields

**Custom indexes:**
```typescript
{
  slug: 'posts',
  indexes: [
    { fields: ['status', 'createdAt'] },
    { fields: ['author'], options: { unique: false } }
  ]
}
```

## Transactions

**PostgreSQL/SQLite:**
```typescript
await payload.db.beginTransaction()
try {
  await payload.create({ collection: 'posts', data: {...} })
  await payload.create({ collection: 'comments', data: {...} })
  await payload.db.commitTransaction()
} catch (error) {
  await payload.db.rollbackTransaction()
}
```

**MongoDB:**
Supports transactions in replica sets.

## Query Optimization

**1. Select only needed fields:**
```typescript
await payload.find({
  collection: 'posts',
  select: { title: true, createdAt: true }  // Only these fields
})
```

**2. Limit depth of population:**
```typescript
await payload.find({
  collection: 'posts',
  depth: 1  // Only populate one level
})
```

**3. Use pagination:**
```typescript
await payload.find({
  collection: 'posts',
  limit: 20,
  page: 1
})
```

**4. Add indexes:**
```typescript
{
  slug: 'posts',
  indexes: [
    { fields: ['status', '-createdAt'] }  // Compound index
  ]
}
```

## Database Comparison

| Feature | MongoDB | PostgreSQL | SQLite |
|---------|---------|------------|--------|
| **Type** | NoSQL | SQL | SQL |
| **Schema** | Flexible | Strict | Strict |
| **Scaling** | Horizontal | Vertical | Single file |
| **Transactions** | Yes (replica) | Yes | Yes |
| **Use Case** | Rapid dev | Complex queries | Small apps |

## Connection Management

**Pool configuration:**
```typescript
db: postgresAdapter({
  pool: {
    connectionString: process.env.DATABASE_URL,
    max: 20,  // Max connections
    idleTimeoutMillis: 30000
  }
})
```

## Next Steps

- [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)
- [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)

---

*Master database architecture!*
