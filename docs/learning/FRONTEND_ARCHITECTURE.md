# Frontend Architecture Deep Dive

**Documented:** November 18, 2025

Complete guide to Payload's React-based admin panel and UI architecture.

## Overview

Payload's admin panel is built with:
- **React 19** (Server Components)
- **Next.js 15** (App Router)
- **React Compiler** (Automatic optimization)
- **SCSS** (Styling)

🧠 **Mental Model:**
The admin panel is a **Next.js application** that uses Payload as a data layer.

## Server Components vs Client Components

### Server Components (Default)

**Run on server, send HTML:**
```typescript
// packages/ui/src/views/List/index.tsx
export async function ListView({ collectionSlug }) {
  const docs = await payload.find({ collection: collectionSlug })
  return <Table docs={docs} />
}
```

**Benefits:**
- No JavaScript sent to client
- Direct database access
- Secure (API keys, secrets safe)

### Client Components

**Run in browser, interactive:**
```typescript
'use client'
export function TableRow({ doc }) {
  const [expanded, setExpanded] = useState(false)
  return <tr onClick={() => setExpanded(!expanded)}>...</tr>
}
```

**Use when:**
- Interactivity needed (onClick, onChange)
- Browser APIs (localStorage, window)
- React hooks (useState, useEffect)

## Component Architecture

```
Admin Panel
├── Views (Server Components)
│   ├── List View
│   ├── Edit View  
│   └── Create View
├── Forms (Mixed)
│   ├── Form (Server)
│   └── Field Components (Client)
├── Elements (Client)
│   ├── Button
│   ├── Modal
│   └── Table
└── Providers (Client)
    ├── Auth Provider
    ├── Preferences Provider
    └── Theme Provider
```

## Key Patterns

### 1. Form State Management

Uses **Server Actions** for mutations:

```typescript
'use server'
export async function handleSubmit(formData: FormData) {
  await payload.create({
    collection: 'posts',
    data: Object.fromEntries(formData)
  })
  revalidatePath('/admin/collections/posts')
  redirect(`/admin/collections/posts/${result.id}`)
}
```

### 2. Field Components

Each field type has its own component:

```
packages/ui/src/fields/
├── Text/
│   ├── Input.tsx (Client Component)
│   └── Cell.tsx (Display in table)
├── RichText/
├── Upload/
└── Relationship/
```

### 3. Real-time Updates

WebSocket connection for:
- Document locking (prevent concurrent edits)
- Live preview
- Collaborative editing

### 4. Optimistic UI

```typescript
const [optimisticDocs, addOptimistic] = useOptimistic(docs)

async function deleteDoc(id) {
  addOptimistic(docs.filter(d => d.id !== id))
  await payload.delete({ collection, id })
}
```

## Styling Architecture

**SCSS Modules:**
```scss
// packages/ui/src/elements/Button/index.module.scss
.button {
  background: var(--theme-bg);
  color: var(--theme-text);
}
```

**CSS Variables for theming:**
- Light/dark mode support
- Customizable via Payload config

## Performance Optimizations

1. **React Compiler** - Automatic memoization
2. **Code Splitting** - Dynamic imports
3. **Lazy Loading** - Fields load on demand
4. **Streaming** - Progressive rendering

## Custom Components

Extend admin UI:

```typescript
// payload.config.ts
export default buildConfig({
  admin: {
    components: {
      beforeDashboard: ['/components/CustomWidget'],
      views: {
        CustomView: '/views/Custom'
      }
    }
  }
})
```

## Next Steps

- [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)
- [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md)

---

*Master React Server Components!*
