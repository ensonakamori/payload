# Database Schema Guide

**Documented:** November 18, 2025

Understanding database schemas and design patterns in Payload CMS.

## Mental Model: Schema Generation

**For React Developers:**
In traditional MERN apps, you define Mongoose schemas manually. In Payload, **schemas are generated automatically** from your collection config:

```
Collection Config (TypeScript)
        ↓
Payload generates schema
        ↓
Database adapter applies it
        ↓
Database (MongoDB/Postgres/etc.)
```

**Key Insight:** You never write database schemas directly. Define fields in your collection config, and Payload handles the rest.

## Database Adapters

Payload supports multiple databases through adapters:

- **MongoDB** (via Mongoose)
- **PostgreSQL** (via Drizzle)
- **SQLite** (via Drizzle)
- **Vercel Postgres** (via Drizzle)
- **Cloudflare D1 SQLite** (via Drizzle)

**All use the same collection config.** Switch databases by changing the adapter:

```typescript
// MongoDB
import { mongooseAdapter } from '@payloadcms/db-mongodb'

export default buildConfig({
  db: mongooseAdapter({
    url: process.env.DATABASE_URI,
  }),
})

// PostgreSQL
import { postgresAdapter } from '@payloadcms/db-postgres'

export default buildConfig({
  db: postgresAdapter({
    pool: {
      connectionString: process.env.DATABASE_URI,
    },
  }),
})
```

## Schema Generation from Fields

### Basic Field Types

**Collection config:**

```typescript
{
  slug: 'posts',
  fields: [
    { name: 'title', type: 'text', required: true },
    { name: 'excerpt', type: 'textarea' },
    { name: 'views', type: 'number', defaultValue: 0 },
    { name: 'isPublished', type: 'checkbox' },
    { name: 'publishedAt', type: 'date' },
    { name: 'status', type: 'select', options: ['draft', 'published', 'archived'] }
  ]
}
```

**MongoDB schema (generated):**

```javascript
{
  _id: ObjectId,
  title: String (required),
  excerpt: String,
  views: Number (default: 0),
  isPublished: Boolean,
  publishedAt: Date,
  status: String (enum: ['draft', 'published', 'archived']),
  createdAt: Date,
  updatedAt: Date,
  __v: Number
}
```

**PostgreSQL schema (generated):**

```sql
CREATE TABLE posts (
  id VARCHAR(255) PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  excerpt TEXT,
  views INTEGER DEFAULT 0,
  is_published BOOLEAN,
  published_at TIMESTAMP,
  status VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Relationship Fields

**HasOne relationship:**

```typescript
{
  slug: 'posts',
  fields: [
    {
      name: 'author',
      type: 'relationship',
      relationTo: 'users', // References users collection
      required: true
    }
  ]
}
```

**MongoDB:**

```javascript
{
  author: ObjectId (ref: 'users')
}
```

**PostgreSQL:**

```sql
author VARCHAR(255) REFERENCES users(id)
```

**HasMany relationship:**

```typescript
{
  name: 'categories',
  type: 'relationship',
  relationTo: 'categories',
  hasMany: true // Array of references
}
```

**MongoDB:**

```javascript
{
  categories: [ObjectId] (ref: 'categories')
}
```

**PostgreSQL (join table):**

```sql
CREATE TABLE posts_categories (
  id SERIAL PRIMARY KEY,
  post_id VARCHAR(255) REFERENCES posts(id),
  category_id VARCHAR(255) REFERENCES categories(id),
  order INTEGER
);
```

**Polymorphic relationships:**

```typescript
{
  name: 'related',
  type: 'relationship',
  relationTo: ['posts', 'pages'], // Can reference multiple collections
  hasMany: true
}
```

**MongoDB:**

```javascript
{
  related: [
    {
      relationTo: 'posts',
      value: ObjectId,
    },
    {
      relationTo: 'pages',
      value: ObjectId,
    },
  ]
}
```

### Array and Group Fields

**Array field:**

```typescript
{
  name: 'tags',
  type: 'array',
  fields: [
    { name: 'label', type: 'text' },
    { name: 'value', type: 'text' }
  ]
}
```

**MongoDB:**

```javascript
{
  tags: [
    {
      id: String,
      label: String,
      value: String,
    },
  ]
}
```

**PostgreSQL (JSON column):**

```sql
tags JSONB
```

**Group field:**

```typescript
{
  name: 'meta',
  type: 'group',
  fields: [
    { name: 'title', type: 'text' },
    { name: 'description', type: 'textarea' }
  ]
}
```

**MongoDB (nested object):**

```javascript
{
  meta: {
    title: String,
    description: String
  }
}
```

**PostgreSQL (separate columns):**

```sql
meta_title VARCHAR(255),
meta_description TEXT
```

### Rich Text Fields

**Lexical (default in Payload 3.x):**

```typescript
{
  name: 'content',
  type: 'richText'
}
```

**Stored as:**

```javascript
{
  content: {
    root: {
      type: 'root',
      format: '',
      indent: 0,
      version: 1,
      children: [
        {
          type: 'paragraph',
          format: '',
          indent: 0,
          version: 1,
          children: [
            {
              type: 'text',
              format: 0,
              text: 'Hello world',
              version: 1
            }
          ]
        }
      ]
    }
  }
}
```

**Database type:**

- MongoDB: Object
- PostgreSQL: JSONB

### Upload Fields

**Collection with upload:**

```typescript
{
  slug: 'media',
  upload: {
    staticDir: 'media',
    imageSizes: [
      { name: 'thumbnail', width: 400, height: 300 }
    ]
  },
  fields: [
    { name: 'alt', type: 'text' }
  ]
}
```

**MongoDB schema:**

```javascript
{
  _id: ObjectId,
  alt: String,
  filename: String,
  mimeType: String,
  filesize: Number,
  width: Number,
  height: Number,
  url: String,
  sizes: {
    thumbnail: {
      filename: String,
      mimeType: String,
      filesize: Number,
      width: Number,
      height: Number,
      url: String
    }
  },
  createdAt: Date,
  updatedAt: Date
}
```

### Localized Fields

**With localization:**

```typescript
{
  slug: 'posts',
  fields: [
    {
      name: 'title',
      type: 'text',
      localized: true // Available in multiple languages
    }
  ]
}
```

**MongoDB:**

```javascript
{
  title: {
    en: 'Hello',
    es: 'Hola',
    fr: 'Bonjour'
  }
}
```

**PostgreSQL (separate columns):**

```sql
title_en VARCHAR(255),
title_es VARCHAR(255),
title_fr VARCHAR(255)
```

## Indexes

### Automatic Indexes

**Payload automatically creates indexes for:**

- ID fields (unique)
- Email fields in auth collections (unique)
- Slug fields (unique if `unique: true`)
- Relationship fields
- `createdAt` and `updatedAt`

### Custom Indexes

**MongoDB:**

```typescript
{
  slug: 'posts',
  fields: [
    {
      name: 'slug',
      type: 'text',
      unique: true, // Creates unique index
      index: true   // Creates standard index
    },
    {
      name: 'status',
      type: 'select',
      options: ['draft', 'published'],
      index: true // Index for filtering
    }
  ]
}
```

**Compound indexes (MongoDB only):**

```typescript
// In database adapter config
import { mongooseAdapter } from '@payloadcms/db-mongodb'

export default buildConfig({
  db: mongooseAdapter({
    url: process.env.DATABASE_URI,
    schemaOptions: {
      // Add compound index
      posts: {
        indexes: [
          {
            fields: { status: 1, createdAt: -1 },
            name: 'status_createdAt_idx',
          },
        ],
      },
    },
  }),
})
```

**PostgreSQL indexes:**

```typescript
import { postgresAdapter } from '@payloadcms/db-postgres'

export default buildConfig({
  db: postgresAdapter({
    pool: { connectionString: process.env.DATABASE_URI },
    // Indexes created via migrations
    push: false,
  }),
})
```

Then create migration:

```sql
CREATE INDEX posts_status_idx ON posts(status);
CREATE INDEX posts_created_at_idx ON posts(created_at DESC);
CREATE INDEX posts_status_created_at_idx ON posts(status, created_at DESC);
```

### Geospatial Indexes

**For location queries:**

```typescript
{
  name: 'location',
  type: 'point',
  index: true // Creates 2dsphere index in MongoDB
}
```

**MongoDB:**

```javascript
{
  location: {
    type: 'Point',
    coordinates: [longitude, latitude]
  }
}
```

**Index:**

```javascript
db.locations.createIndex({ location: '2dsphere' })
```

## Migrations

### MongoDB (Mongoose)

**Schema changes are automatic:**

- Add new fields → Existing docs unaffected (fields are `undefined` until set)
- Remove fields → Existing docs keep old data (ignored by Payload)
- Rename fields → Requires manual migration

**Manual migration (rename field):**

```typescript
// migration.ts
import { getPayloadHMR } from '@payloadcms/next/utilities'

const migrate = async () => {
  const payload = await getPayloadHMR({ config })

  const posts = await payload.find({
    collection: 'posts',
    limit: 0,
  })

  for (const post of posts.docs) {
    if (post.oldFieldName) {
      await payload.update({
        collection: 'posts',
        id: post.id,
        data: {
          newFieldName: post.oldFieldName,
          oldFieldName: null,
        },
      })
    }
  }

  console.log('Migration complete')
}

migrate()
```

### PostgreSQL/SQLite (Drizzle)

**Migrations are required for schema changes.**

**Generate migration:**

```bash
pnpm payload migrate:create
```

**Apply migrations:**

```bash
pnpm payload migrate
```

**Example migration (add column):**

```sql
-- migration_001.sql
ALTER TABLE posts ADD COLUMN views INTEGER DEFAULT 0;
```

**Example migration (add relationship table):**

```sql
-- migration_002.sql
CREATE TABLE posts_tags (
  id SERIAL PRIMARY KEY,
  post_id VARCHAR(255) REFERENCES posts(id) ON DELETE CASCADE,
  tag_id VARCHAR(255) REFERENCES tags(id) ON DELETE CASCADE,
  order INTEGER
);

CREATE INDEX posts_tags_post_id_idx ON posts_tags(post_id);
CREATE INDEX posts_tags_tag_id_idx ON posts_tags(tag_id);
```

## Schema Design Patterns

### Pattern 1: Normalized (SQL-style)

**Use separate collections for everything:**

```typescript
{
  slug: 'posts',
  fields: [
    { name: 'title', type: 'text' },
    {
      name: 'author',
      type: 'relationship',
      relationTo: 'users'
    },
    {
      name: 'categories',
      type: 'relationship',
      relationTo: 'categories',
      hasMany: true
    }
  ]
}

{
  slug: 'categories',
  fields: [
    { name: 'name', type: 'text' }
  ]
}
```

**Pros:**

- No data duplication
- Easy to update (change category name in one place)
- Follows SQL best practices

**Cons:**

- Requires `depth` or joins to get full data
- More database queries

### Pattern 2: Denormalized (MongoDB-style)

**Embed related data:**

```typescript
{
  slug: 'posts',
  fields: [
    { name: 'title', type: 'text' },
    {
      name: 'categories',
      type: 'array',
      fields: [
        { name: 'name', type: 'text' },
        { name: 'slug', type: 'text' }
      ]
    }
  ]
}
```

**Pros:**

- Single query gets all data
- Faster reads
- Works well with document databases

**Cons:**

- Data duplication
- Updates require changing multiple docs
- Can grow very large

### Pattern 3: Hybrid (Recommended)

**Use relationships for frequently updated data, arrays for static data:**

```typescript
{
  slug: 'posts',
  fields: [
    { name: 'title', type: 'text' },
    {
      name: 'author', // Relationship (users change)
      type: 'relationship',
      relationTo: 'users'
    },
    {
      name: 'tags', // Array (tags are static)
      type: 'array',
      fields: [
        { name: 'label', type: 'text' }
      ]
    }
  ]
}
```

## Versioning & Drafts

**Enable versions:**

```typescript
{
  slug: 'posts',
  versions: {
    drafts: true, // Enable draft versions
    maxPerDoc: 50 // Keep last 50 versions
  }
}
```

**Creates separate collection:**

**MongoDB:**

```javascript
// Main collection: posts
{
  _id: ObjectId,
  title: 'Published Title',
  status: 'published',
  _status: 'published'
}

// Versions collection: _posts_versions
{
  _id: ObjectId,
  parent: ObjectId (ref: 'posts'),
  version: {
    title: 'Draft Title',
    status: 'draft'
  },
  createdAt: Date
}
```

**PostgreSQL:**

```sql
-- Main table
CREATE TABLE posts (...);

-- Versions table
CREATE TABLE _posts_versions (
  id VARCHAR(255) PRIMARY KEY,
  parent VARCHAR(255) REFERENCES posts(id),
  version JSONB,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```

## Authentication Collections

**Users collection with auth:**

```typescript
{
  slug: 'users',
  auth: true,
  fields: [
    { name: 'name', type: 'text' },
    { name: 'role', type: 'select', options: ['admin', 'user'] }
  ]
}
```

**Generated schema:**

**MongoDB:**

```javascript
{
  _id: ObjectId,
  name: String,
  role: String,
  email: String (unique, required),
  password: String (bcrypt hashed),
  salt: String,
  hash: String,
  resetPasswordToken: String,
  resetPasswordExpiration: Date,
  emailVerified: Boolean,
  emailVerificationToken: String,
  loginAttempts: Number,
  lockUntil: Date,
  createdAt: Date,
  updatedAt: Date
}
```

**Key fields added by auth:**

- `email` - Unique email address
- `password` - Bcrypt hashed password
- `resetPasswordToken` - For password reset
- `loginAttempts` - Brute force protection
- `lockUntil` - Account lockout

## Globals

**Global singleton config:**

```typescript
{
  slug: 'header',
  fields: [
    { name: 'logo', type: 'upload', relationTo: 'media' },
    {
      name: 'navigation',
      type: 'array',
      fields: [
        { name: 'label', type: 'text' },
        { name: 'url', type: 'text' }
      ]
    }
  ]
}
```

**Schema (always single document):**

**MongoDB:**

```javascript
// Collection: globals
{
  _id: ObjectId,
  globalType: 'header',
  logo: ObjectId (ref: 'media'),
  navigation: [
    {
      id: String,
      label: String,
      url: String
    }
  ],
  createdAt: Date,
  updatedAt: Date
}
```

**PostgreSQL:**

```sql
CREATE TABLE _globals (
  id VARCHAR(255) PRIMARY KEY,
  global_type VARCHAR(255) UNIQUE,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

CREATE TABLE globals_header (
  id VARCHAR(255) PRIMARY KEY REFERENCES _globals(id),
  logo VARCHAR(255) REFERENCES media(id),
  navigation JSONB
);
```

## Query Performance

### Indexing Strategy

**Index fields you query frequently:**

```typescript
{
  slug: 'posts',
  fields: [
    {
      name: 'status',
      type: 'select',
      options: ['draft', 'published'],
      index: true // Frequently filtered
    },
    {
      name: 'author',
      type: 'relationship',
      relationTo: 'users'
      // Auto-indexed by Payload
    },
    {
      name: 'slug',
      type: 'text',
      unique: true // Unique index for lookups
    }
  ]
}
```

**Check index usage:**

**MongoDB:**

```javascript
// Explain query
db.posts.find({ status: 'published' }).explain('executionStats')

// Check if index used
{
  executionStats: {
    executionSuccess: true,
    executionTimeMillis: 2,
    totalKeysExamined: 150,
    totalDocsExamined: 150
  },
  winningPlan: {
    stage: 'FETCH',
    inputStage: {
      stage: 'IXSCAN', // ✅ Index scan (good!)
      indexName: 'status_1'
    }
  }
}
```

**PostgreSQL:**

```sql
EXPLAIN ANALYZE SELECT * FROM posts WHERE status = 'published';

-- Look for "Index Scan" not "Seq Scan"
Index Scan using posts_status_idx on posts  (cost=0.29..8.31 rows=1 width=...)
```

### Avoid N+1 Queries

**❌ BAD: N+1 problem**

```typescript
// Fetches posts (1 query)
const posts = await payload.find({ collection: 'posts', limit: 10 })

// Then fetches each author separately (10 more queries!)
for (const post of posts.docs) {
  const author = await payload.findByID({
    collection: 'users',
    id: post.author,
  })
}
```

**✅ GOOD: Use depth**

```typescript
// Single query with populated authors
const posts = await payload.find({
  collection: 'posts',
  limit: 10,
  depth: 1, // Populates author in same query
})

// Authors already populated
posts.docs.forEach((post) => {
  console.log(post.author.name)
})
```

### Pagination

**Always paginate large collections:**

```typescript
// ✅ GOOD
const posts = await payload.find({
  collection: 'posts',
  limit: 50,
  page: 1,
})

// ❌ BAD: Fetches all documents (may timeout!)
const posts = await payload.find({
  collection: 'posts',
  limit: 0, // 0 = no limit
})
```

## Backup Strategies

### MongoDB

**mongodump:**

```bash
# Full backup
mongodump --uri="mongodb://localhost:27017/payload" --out=/backups/$(date +%Y%m%d)

# Restore
mongorestore --uri="mongodb://localhost:27017/payload" /backups/20250115
```

**Automated backups:**

```bash
# Add to crontab (daily at 2am)
0 2 * * * mongodump --uri="$DATABASE_URI" --out=/backups/$(date +\%Y\%m\%d)
```

### PostgreSQL

**pg_dump:**

```bash
# Full backup
pg_dump $DATABASE_URI > /backups/payload_$(date +%Y%m%d).sql

# Restore
psql $DATABASE_URI < /backups/payload_20250115.sql
```

**Automated backups:**

```bash
# Add to crontab (daily at 2am)
0 2 * * * pg_dump $DATABASE_URI > /backups/payload_$(date +\%Y\%m\%d).sql
```

## Schema Inspection

### View Generated Schema

**MongoDB (Mongoose models):**

```typescript
import { getPayload } from 'payload'

const payload = await getPayload({ config })

// Access Mongoose model
const PostModel = payload.db.collections.posts

// View schema
console.log(PostModel.schema.obj)
console.log(PostModel.schema.indexes())
```

**PostgreSQL (Drizzle tables):**

```typescript
import { getPayload } from 'payload'

const payload = await getPayload({ config })

// Access Drizzle tables
const tables = payload.db.tables
console.log(tables.posts)
```

**Via database client:**

**MongoDB:**

```javascript
// Connect with mongosh
mongosh "mongodb://localhost:27017/payload"

// List collections
show collections

// View a document
db.posts.findOne()

// View indexes
db.posts.getIndexes()
```

**PostgreSQL:**

```sql
-- Connect with psql
psql $DATABASE_URI

-- List tables
\dt

-- Describe table
\d posts

-- View indexes
\di
```

## Best Practices

### DO:

✅ Let Payload generate schemas (don't create manually)
✅ Index frequently queried fields
✅ Use relationships for data that changes
✅ Use arrays for static nested data
✅ Set reasonable `limit` for queries
✅ Use `depth` to avoid N+1 queries
✅ Create regular backups
✅ Test migrations before production

### DON'T:

❌ Manually create/modify schema (use collection config)
❌ Over-index (slows writes)
❌ Denormalize everything (hard to update)
❌ Normalize everything (slow reads)
❌ Query without pagination
❌ Set `depth` too high (performance hit)
❌ Skip migrations (SQL databases)

---

_Let Payload manage your schema, focus on building features!_
