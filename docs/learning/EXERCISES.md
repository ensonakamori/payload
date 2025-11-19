# Exercises

**Documented:** November 18, 2025

Hands-on exercises to master Payload CMS.

## How to Use This Guide

Each exercise builds on previous concepts. Work through them in order for the best learning experience.

**Before starting:**

1. Complete [GETTING_STARTED.md](./GETTING_STARTED.md)
2. Have dev server running: `pnpm run dev`
3. Create new test directory: `mkdir -p test/exercises`

## Exercise 1: Create Your First Collection

**Goal:** Understand collection basics and CRUD operations.

**Task:** Build a blog posts collection with all essential fields.

**Step 1: Create config**

```typescript
// test/exercises/config.ts
import { buildConfig } from 'payload'
import { mongooseAdapter } from '@payloadcms/db-mongodb'

export default buildConfig({
  secret: process.env.PAYLOAD_SECRET || 'test',
  db: mongooseAdapter({
    url: process.env.DATABASE_URI || 'mongodb://localhost:27017/payload',
  }),
  collections: [
    {
      slug: 'posts',
      admin: {
        useAsTitle: 'title',
      },
      fields: [
        {
          name: 'title',
          type: 'text',
          required: true,
        },
        {
          name: 'content',
          type: 'textarea',
          required: true,
        },
        {
          name: 'status',
          type: 'select',
          options: ['draft', 'published', 'archived'],
          defaultValue: 'draft',
          required: true,
        },
        {
          name: 'publishedAt',
          type: 'date',
        },
      ],
    },
  ],
  typescript: {
    outputFile: './test/exercises/payload-types.ts',
  },
})
```

**Step 2: Start dev server**

```bash
pnpm run dev exercises
```

**Step 3: Test in admin UI**

1. Go to http://localhost:3000/admin
2. Navigate to Posts
3. Create 3 posts:
   - One draft
   - One published (set publishedAt)
   - One archived

**Step 4: Test via API**

```bash
# Create post
curl -X POST http://localhost:3000/api/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "API Test Post",
    "content": "Created via API",
    "status": "published",
    "publishedAt": "2025-01-15T12:00:00.000Z"
  }'

# List posts
curl http://localhost:3000/api/posts

# Filter published posts
curl "http://localhost:3000/api/posts?where[status][equals]=published"
```

**Checkpoint:**

- ✅ Can create posts in admin UI
- ✅ Can see posts list
- ✅ Can edit posts
- ✅ Can filter by status
- ✅ API returns correct data

---

## Exercise 2: Add Relationships

**Goal:** Understand relationships between collections.

**Task:** Add authors and categories to posts.

**Step 1: Add collections**

```typescript
// test/exercises/config.ts
export default buildConfig({
  collections: [
    // Add BEFORE posts collection
    {
      slug: 'users',
      auth: true,
      admin: {
        useAsTitle: 'name',
      },
      fields: [
        {
          name: 'name',
          type: 'text',
          required: true,
        },
        {
          name: 'role',
          type: 'select',
          options: ['admin', 'editor', 'user'],
          defaultValue: 'user',
          required: true,
        },
      ],
    },
    {
      slug: 'categories',
      admin: {
        useAsTitle: 'name',
      },
      fields: [
        {
          name: 'name',
          type: 'text',
          required: true,
        },
        {
          name: 'slug',
          type: 'text',
          required: true,
          unique: true,
        },
      ],
    },
    {
      slug: 'posts',
      fields: [
        // ... existing fields ...
        {
          name: 'author',
          type: 'relationship',
          relationTo: 'users',
          required: true,
        },
        {
          name: 'categories',
          type: 'relationship',
          relationTo: 'categories',
          hasMany: true,
        },
      ],
    },
  ],
})
```

**Step 2: Create test data**

1. Create 2-3 categories (e.g., "JavaScript", "TypeScript", "React")
2. Create a user (auto-created on first login)
3. Create posts with author and categories

**Step 3: Test populated queries**

```typescript
// test/exercises/test.ts
import { getPayload } from 'payload'
import config from './config'

const test = async () => {
  const payload = await getPayload({ config })

  // Query with population (depth: 1)
  const posts = await payload.find({
    collection: 'posts',
    depth: 1, // Populates author and categories
  })

  console.log('Posts with relationships:')
  posts.docs.forEach((post) => {
    console.log(`- ${post.title}`)
    console.log(`  Author: ${post.author.name}`)
    console.log(
      `  Categories: ${post.categories.map((c) => c.name).join(', ')}`,
    )
  })
}

test()
```

**Checkpoint:**

- ✅ Categories collection created
- ✅ Posts have author relationship
- ✅ Posts have multiple categories
- ✅ Relationships populate with `depth: 1`

---

## Exercise 3: Add Access Control

**Goal:** Secure your collections with role-based access control.

**Task:** Implement read/write permissions.

**Step 1: Add access control to posts**

```typescript
{
  slug: 'posts',
  access: {
    // Anyone can read published posts
    read: ({ req }) => {
      // Admin can read all
      if (req.user?.role === 'admin') return true

      // Others can only read published posts
      return {
        status: { equals: 'published' }
      }
    },

    // Only authenticated users can create
    create: ({ req }) => {
      return !!req.user
    },

    // Only admin or author can update
    update: ({ req }) => {
      if (!req.user) return false

      // Admin can update all
      if (req.user.role === 'admin') return true

      // Authors can update their own posts
      return {
        author: { equals: req.user.id }
      }
    },

    // Only admin can delete
    delete: ({ req }) => {
      return req.user?.role === 'admin'
    }
  },
  fields: [
    // ... existing fields ...
  ]
}
```

**Step 2: Test access control**

```typescript
// test/exercises/access-test.ts
const test = async () => {
  const payload = await getPayload({ config })

  // Test as anonymous user
  const publicPosts = await payload.find({
    collection: 'posts',
    overrideAccess: false, // Enforce access control
  })

  console.log('Public posts:', publicPosts.totalDocs)
  // Should only show published posts

  // Login as user
  const { user, token } = await payload.login({
    collection: 'users',
    data: {
      email: 'user@example.com',
      password: 'password',
    },
  })

  // Try to create post
  const post = await payload.create({
    collection: 'posts',
    data: {
      title: 'User Post',
      content: 'Created by user',
      status: 'draft',
      author: user.id,
    },
    user, // Pass authenticated user
  })

  console.log('Created post:', post.id)
}
```

**Checkpoint:**

- ✅ Anonymous users see only published posts
- ✅ Authenticated users can create posts
- ✅ Users can update their own posts
- ✅ Users cannot update others' posts
- ✅ Only admins can delete

---

## Exercise 4: Add Hooks

**Goal:** Understand hook lifecycle and data transformation.

**Task:** Auto-generate slug, track changes, send notifications.

**Step 1: Auto-generate slug**

```typescript
{
  slug: 'posts',
  fields: [
    // ... existing fields ...
    {
      name: 'slug',
      type: 'text',
      unique: true,
      admin: {
        position: 'sidebar'
      }
    }
  ],
  hooks: {
    beforeChange: [
      ({ data, operation }) => {
        // Generate slug from title on create
        if (operation === 'create' && !data.slug) {
          data.slug = data.title
            .toLowerCase()
            .replace(/[^a-z0-9]+/g, '-')
            .replace(/(^-|-$)/g, '')
        }
        return data
      }
    ]
  }
}
```

**Step 2: Track changes**

```typescript
{
  slug: 'posts',
  hooks: {
    afterChange: [
      ({ doc, operation, previousDoc }) => {
        if (operation === 'update') {
          console.log('Post updated:')
          console.log('- ID:', doc.id)
          console.log('- Previous status:', previousDoc.status)
          console.log('- New status:', doc.status)

          // Detect status change
          if (previousDoc.status !== doc.status) {
            console.log('Status changed!')
          }
        }
        return doc
      }
    ]
  }
}
```

**Step 3: Send notification on publish**

```typescript
{
  hooks: {
    afterChange: [
      async ({ doc, operation, previousDoc, req }) => {
        // Check if post was just published
        if (
          operation === 'update' &&
          previousDoc.status !== 'published' &&
          doc.status === 'published'
        ) {
          console.log(`Post "${doc.title}" was published!`)

          // In real app, send email notification:
          // await sendEmail({
          //   to: 'subscribers@example.com',
          //   subject: `New post: ${doc.title}`,
          //   body: doc.content
          // })
        }
        return doc
      },
    ]
  }
}
```

**Checkpoint:**

- ✅ Slug auto-generates from title
- ✅ Changes are tracked in console
- ✅ Publish notification works
- ✅ Hooks don't interfere with each other

---

## Exercise 5: File Uploads

**Goal:** Handle file uploads and image processing.

**Task:** Create media collection with image sizes.

**Step 1: Create media collection**

```typescript
{
  slug: 'media',
  upload: {
    staticDir: './test/exercises/media',
    mimeTypes: ['image/*'],
    imageSizes: [
      {
        name: 'thumbnail',
        width: 400,
        height: 300,
        position: 'center'
      },
      {
        name: 'card',
        width: 768,
        height: 432,
        position: 'center'
      },
      {
        name: 'featured',
        width: 1920,
        height: 1080,
        position: 'center'
      }
    ],
    limits: {
      fileSize: 5000000 // 5MB
    }
  },
  fields: [
    {
      name: 'alt',
      type: 'text',
      required: true
    },
    {
      name: 'caption',
      type: 'textarea'
    }
  ]
}
```

**Step 2: Add featured image to posts**

```typescript
{
  slug: 'posts',
  fields: [
    // ... existing fields ...
    {
      name: 'featuredImage',
      type: 'upload',
      relationTo: 'media',
      required: false
    }
  ]
}
```

**Step 3: Test upload**

1. Upload image in admin UI
2. Set alt text and caption
3. Add to post as featured image

**Step 4: Access image sizes**

```typescript
const post = await payload.findByID({
  collection: 'posts',
  id: postId,
  depth: 1,
})

console.log('Original:', post.featuredImage.url)
console.log('Thumbnail:', post.featuredImage.sizes.thumbnail.url)
console.log('Card:', post.featuredImage.sizes.card.url)
```

**Checkpoint:**

- ✅ Can upload images
- ✅ Thumbnails generated automatically
- ✅ Featured image appears in posts
- ✅ Different sizes accessible

---

## Exercise 6: Rich Text Editor

**Goal:** Use Lexical rich text editor with custom features.

**Task:** Add rich content field to posts.

**Step 1: Add rich text field**

```typescript
import { lexicalEditor } from '@payloadcms/richtext-lexical'

{
  slug: 'posts',
  fields: [
    {
      name: 'content',
      type: 'richText',
      editor: lexicalEditor({
        features: ({ defaultFeatures }) => [
          ...defaultFeatures,
          // Features are already included, just use defaults
        ]
      }),
      required: true
    }
  ]
}
```

**Step 2: Test in admin**

1. Create new post
2. Use rich text features:
   - Bold, italic, underline
   - Headings (H1, H2, H3)
   - Lists (ordered, unordered)
   - Links
   - Images
   - Code blocks

**Step 3: Query rich text**

```typescript
const post = await payload.findByID({
  collection: 'posts',
  id: postId,
})

console.log('Rich text content:', post.content)
// Returns Lexical JSON structure
```

**Checkpoint:**

- ✅ Rich text editor works in admin
- ✅ All formatting options available
- ✅ Content saved as JSON
- ✅ Can render in frontend

---

## Exercise 7: Localization

**Goal:** Make content available in multiple languages.

**Task:** Translate posts to Spanish and French.

**Step 1: Enable localization**

```typescript
export default buildConfig({
  localization: {
    locales: ['en', 'es', 'fr'],
    defaultLocale: 'en',
    fallback: true,
  },
  collections: [
    {
      slug: 'posts',
      fields: [
        {
          name: 'title',
          type: 'text',
          required: true,
          localized: true, // Available in all locales
        },
        {
          name: 'content',
          type: 'richText',
          required: true,
          localized: true,
        },
        {
          name: 'slug',
          type: 'text',
          unique: true,
          localized: true,
        },
        {
          name: 'status',
          type: 'select',
          options: ['draft', 'published'],
          required: true,
          // NOT localized - same status for all languages
        },
      ],
    },
  ],
})
```

**Step 2: Create localized content**

1. Create post in English
2. Switch locale to Spanish in admin (locale selector)
3. Translate title, content, slug
4. Switch to French and translate again

**Step 3: Query by locale**

```typescript
// Get English version
const postEN = await payload.findByID({
  collection: 'posts',
  id: postId,
  locale: 'en',
})

// Get Spanish version
const postES = await payload.findByID({
  collection: 'posts',
  id: postId,
  locale: 'es',
})

console.log('English title:', postEN.title)
console.log('Spanish title:', postES.title)
```

**Checkpoint:**

- ✅ Locale selector appears in admin
- ✅ Can create content in multiple languages
- ✅ API supports `locale` parameter
- ✅ Fallback to default locale works

---

## Exercise 8: Globals

**Goal:** Create singleton configuration collections.

**Task:** Build site header and footer globals.

**Step 1: Create header global**

```typescript
export default buildConfig({
  globals: [
    {
      slug: 'header',
      fields: [
        {
          name: 'logo',
          type: 'upload',
          relationTo: 'media',
          required: true,
        },
        {
          name: 'navigation',
          type: 'array',
          fields: [
            {
              name: 'label',
              type: 'text',
              required: true,
            },
            {
              name: 'url',
              type: 'text',
              required: true,
            },
          ],
        },
      ],
    },
    {
      slug: 'footer',
      fields: [
        {
          name: 'copyright',
          type: 'text',
          required: true,
        },
        {
          name: 'socialLinks',
          type: 'array',
          fields: [
            {
              name: 'platform',
              type: 'select',
              options: ['twitter', 'github', 'linkedin'],
              required: true,
            },
            {
              name: 'url',
              type: 'text',
              required: true,
            },
          ],
        },
      ],
    },
  ],
})
```

**Step 2: Configure globals**

1. Go to Header in admin
2. Upload logo
3. Add navigation links
4. Save

**Step 3: Use in frontend**

```typescript
const header = await payload.findGlobal({
  slug: 'header',
  depth: 1,
})

console.log('Logo:', header.logo.url)
console.log('Nav links:', header.navigation)
```

**Checkpoint:**

- ✅ Header and footer globals created
- ✅ Can update global settings
- ✅ Globals accessible via API
- ✅ No duplicates (singleton pattern)

---

## Exercise 9: Validation

**Goal:** Add custom validation logic.

**Task:** Validate post content, email format, slug uniqueness.

**Step 1: Add custom validation**

```typescript
{
  slug: 'posts',
  fields: [
    {
      name: 'title',
      type: 'text',
      required: true,
      validate: (value) => {
        if (value.length < 5) {
          return 'Title must be at least 5 characters'
        }
        if (value.length > 100) {
          return 'Title must be less than 100 characters'
        }
        return true
      }
    },
    {
      name: 'slug',
      type: 'text',
      unique: true,
      validate: (value) => {
        // Only allow lowercase letters, numbers, and hyphens
        if (!/^[a-z0-9-]+$/.test(value)) {
          return 'Slug must contain only lowercase letters, numbers, and hyphens'
        }
        return true
      }
    },
    {
      name: 'readTime',
      type: 'number',
      validate: (value) => {
        if (value < 1) {
          return 'Read time must be at least 1 minute'
        }
        if (value > 120) {
          return 'Read time must be less than 120 minutes'
        }
        return true
      }
    }
  ]
}
```

**Step 2: Test validation**
Try to create posts with:

- Short title (< 5 chars) → Should fail
- Invalid slug (contains spaces) → Should fail
- Negative read time → Should fail
- Valid data → Should succeed

**Checkpoint:**

- ✅ Validation errors show in admin UI
- ✅ API returns validation errors
- ✅ Valid data passes validation
- ✅ Error messages are clear

---

## Exercise 10: Build a Full Blog

**Goal:** Combine all concepts into a working blog.

**Final config structure:**

```typescript
export default buildConfig({
  localization: {
    locales: ['en', 'es'],
    defaultLocale: 'en'
  },
  collections: [
    // Users (authentication)
    {
      slug: 'users',
      auth: true,
      fields: [...]
    },

    // Categories
    {
      slug: 'categories',
      fields: [...]
    },

    // Media (uploads)
    {
      slug: 'media',
      upload: {...},
      fields: [...]
    },

    // Posts (main content)
    {
      slug: 'posts',
      access: {...},
      hooks: {...},
      fields: [
        // All field types
        // Relationships
        // Rich text
        // Localized content
        // Validation
      ]
    }
  ],
  globals: [
    // Header
    { slug: 'header', fields: [...] },

    // Footer
    { slug: 'footer', fields: [...] }
  ]
})
```

**Requirements:**

- ✅ User authentication
- ✅ Post CRUD with access control
- ✅ Categories and relationships
- ✅ File uploads
- ✅ Rich text content
- ✅ Auto-generated slugs
- ✅ Localization (EN/ES)
- ✅ Site-wide header/footer
- ✅ Custom validation

**Test checklist:**

1. Create user and login
2. Create categories
3. Upload featured images
4. Create posts with all fields
5. Test access control (try as different users)
6. Translate content to Spanish
7. Configure header/footer
8. Query via API
9. Test validation

---

## Next Steps

**After completing exercises:**

1. Read [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)
2. Explore [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)
3. Check [SECURITY_GUIDE.md](./SECURITY_GUIDE.md)
4. Build your own project!

**Project ideas:**

- E-commerce store
- Portfolio site
- Job board
- Event management
- Knowledge base
- Multi-tenant SaaS

---

_Practice makes perfect. Keep building!_
