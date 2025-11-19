# Technology Stack Deep Dive

**Documented:** November 18, 2025  
**Full Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

Complete guide to every technology used in Payload CMS.

## React 19 - UI Library

**Version:** 19.1.1  
**Status:** ✅ CURRENT (Nov 2025)

### Key Features Used

**Server Components (RSC)**
- Default in Next.js 15
- Fetch data on server
- Zero JavaScript to client

**React Compiler**
- Automatic memoization
- Replaces useMemo/useCallback
- Project uses RC version (upgrade to v1.0 recommended)

**New Hooks**
- `useActionState` - Form state management
- `useFormStatus` - Form submission status  
- `useOptimistic` - Optimistic UI updates

🌉 **Bridge from React 18:**
- No more `useEffect` for data fetching
- Server Components are the default
- Actions replace API calls

## Next.js 15 - Framework

**Version:** 15.4.7  
**Status:** ✅ CURRENT

### App Router

Payload uses **App Router** exclusively (Pages Router is legacy).

**Key Features:**
- Server Components by default
- Server Actions for mutations
- Streaming and Suspense
- Route Handlers (not API routes)

### Caching Changes

**Breaking change from Next.js 14:**
- GET Route Handlers: Uncached by default
- Client Router Cache: Uncached by default

### Turbopack

**Status:** Production-ready (beta) in 15.5

Used by default in dev server for **2-5x faster** builds.

## TypeScript 5.7 - Language

**Version:** 5.7.3  
**Status:** ⚠️ Slightly behind (5.9 available)

### Features Used

- Strict mode enabled
- Path mappings (`@/*`)
- Declaration files (`.d.ts`)
- Type generation from schemas

## Drizzle ORM - Database

**Version:** 0.44.6  
**Status:** ✅ CURRENT

### Why Drizzle?

- **TypeScript-first** - Types inferred from schema
- **SQL-like** - Familiar if you know SQL
- **Lightweight** - Smaller than Prisma
- **Flexible** - Works with any database

🎯 **Mental Model:**
```typescript
// Drizzle schema = TypeScript + SQL
const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
})

// Types automatically inferred!
type Post = typeof posts.$inferSelect
```

## pnpm - Package Manager

**Version:** 9.7.1  
**Status:** ⚠️ Behind (10.x available)

**Why pnpm?**
- Faster than npm/yarn
- Disk space efficient (hard links)
- Perfect for monorepos

**Workspaces:**
Manages 45+ packages in one repo.

## Turborepo - Build Orchestration

**Version:** 2.5.4  
**Status:** ⚠️ Behind (2.6.1)

**Features:**
- Dependency-aware builds
- Caching (2nd build instant!)
- Parallel execution
- Remote caching (optional)

## Jest - Testing (Integration/Unit)

**Version:** 29.7.0  
**Status:** ⚠️ Consider Vitest for new code

Used for:
- Integration tests (API tests)
- Unit tests (utility functions)
- Component tests (React Testing Library)

## Playwright - E2E Testing

**Version:** 1.56.1  
**Status:** ✅ LATEST!

**New in 1.56:**
- Agentic testing support (AI-driven)
- IndexedDB storage state
- Better debugging tools

## MongoDB - Database (NoSQL)

**Via:** mongoose v8.15.1

**Document-based** storage:
```json
{
  "_id": "123",
  "title": "Post",
  "author": { "name": "John" }
}
```

## PostgreSQL - Database (SQL)

**Via:** Drizzle ORM + pg driver

**Relational** tables with foreign keys.

## GraphQL - API Query Language

**Version:** 16.8.1

Auto-generated from Payload collections.

**Endpoint:** `/api/graphql`

## Key Differences from Traditional Stacks

### vs MERN Stack

```
MERN: MongoDB + Express + React + Node
Payload: MongoDB/Postgres + Payload + React 19 + Next.js

Key Differences:
- Next.js instead of Express (RSC support)
- Server Components instead of client-only React
- Built-in admin panel
```

### vs Headless CMS (Strapi, Contentful)

```
Other CMS: Separate backend application
Payload: Integrated into your Next.js app

Benefits:
- Same codebase
- Shared authentication
- Type safety across stack
```

## Learning Resources

**Official Docs (Current - Nov 2025):**
- React: https://react.dev
- Next.js: https://nextjs.org/docs
- Drizzle: https://orm.drizzle.team
- Playwright: https://playwright.dev

**Payload:**
- Docs: https://payloadcms.com/docs
- Discord: https://discord.gg/payload

---

*Master the stack, master the system!*
