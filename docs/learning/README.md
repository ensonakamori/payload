# Payload CMS Learning Path

**Welcome to Payload CMS!** This comprehensive learning path will guide you from setup to confident contribution.

**Last Updated:** November 18, 2025
**Project Version:** Payload CMS v3.64.0
**Tech Stack Research:** [View Current Stack Analysis](./TECH_STACK_RESEARCH.md)

---

## 🎯 Who Is This For?

This learning path is designed for:

- **Mid-level frontend developers** (React, JavaScript/TypeScript)
- Developers new to **Next.js 15 App Router** and **React Server Components**
- Developers new to **monorepo** development
- Anyone wanting to understand **modern full-stack architecture** (2025)
- Contributors wanting to make meaningful contributions within 2-3 weeks

---

## 📚 Quick Navigation

### Start Here

1. [Technology Stack Research](./TECH_STACK_RESEARCH.md) - **READ THIS FIRST** to understand current vs. outdated patterns
2. [Getting Started](./GETTING_STARTED.md) - Setup and first steps
3. [Project Structure](./PROJECT_STRUCTURE.md) - Navigate the codebase

### Core Understanding

4. [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) - System design and patterns
5. [Tech Stack Guide](./TECH_STACK_GUIDE.md) - Deep dive into each technology
6. [Data Flow Guide](./DATA_FLOW_GUIDE.md) - How data moves through the system

### Deep Dives

7. [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - React Server Components, UI patterns
8. [Backend Architecture](./BACKEND_ARCHITECTURE.md) - Payload core, API layer
9. [Database Architecture](./DATABASE_ARCHITECTURE.md) - Drizzle ORM, adapters
10. [Integration Guide](./INTEGRATION_GUIDE.md) - Plugins, storage, email

### Practical Application

11. [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md) - Code style, current patterns
12. [How-To Guide](./HOW_TO_GUIDE.md) - Step-by-step tasks
13. [Code Tours](./CODE_TOURS.md) - Follow real code flows
14. [Development Workflow](./DEVELOPMENT_WORKFLOW.md) - Daily workflow

### Quality & Reference

15. [Testing Guide](./TESTING_GUIDE.md) - Jest, Playwright, testing patterns
16. [Debugging Guide](./DEBUGGING_GUIDE.md) - Troubleshooting and debugging
17. [Security Guide](./SECURITY_GUIDE.md) - Security best practices
18. [API Documentation](./API_DOCUMENTATION.md) - REST and GraphQL APIs
19. [Database Schema](./DATABASE_SCHEMA.md) - Schema design and migrations

### Practice & Contribution

20. [Exercises](./EXERCISES.md) - Hands-on coding exercises
21. [First Contributions](./FIRST_CONTRIBUTIONS.md) - How to contribute
22. [FAQ](./FAQ.md) - Common questions answered

---

## 🗺️ Learning Paths

Choose your path based on your background and goals:

### Path 1: Frontend Developer → Full-Stack (Recommended)

**Time:** 3-4 weeks
**Goal:** Understand the full stack with emphasis on backend concepts

1. ✅ [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) - **Critical!** Understand current (2025) patterns
2. 🚀 [GETTING_STARTED.md](./GETTING_STARTED.md) - Get environment running
3. 📊 [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md) - See the big picture
4. 🎨 [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - Start with familiar territory
5. 🔧 [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md) - Understand request/response
6. 🗄️ [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Learn backend patterns
7. 💾 [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md) - Database concepts
8. 🧪 [CODE_TOURS.md](./CODE_TOURS.md) - Follow real code flows
9. ✍️ [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Practice tasks
10. 🎯 [EXERCISES.md](./EXERCISES.md) - Hands-on coding
11. 🤝 [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md) - Start contributing!

### Path 2: Backend Developer → Full-Stack

**Time:** 2-3 weeks
**Goal:** Learn modern frontend (React 19, Next.js 15, Server Components)

1. ✅ [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) - React 19 is **very different** from React 18!
2. 🚀 [GETTING_STARTED.md](./GETTING_STARTED.md)
3. 📊 [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
4. 🗄️ [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Familiar concepts first
5. 💾 [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)
6. 🎨 [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - **Focus here** - Server Components are new!
7. 🔧 [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md)
8. 📖 [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md) - React deep dive
9. 🧪 [CODE_TOURS.md](./CODE_TOURS.md)
10. 🤝 [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)

### Path 3: Quick Contributor (1 week intensive)

**Time:** 1 week
**Goal:** Make first contribution quickly

1. ✅ [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) - **Skim** for current patterns
2. 🚀 [GETTING_STARTED.md](./GETTING_STARTED.md)
3. 📁 [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md)
4. 📖 [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md)
5. 🧪 [CODE_TOURS.md](./CODE_TOURS.md) - Pick relevant tours
6. 🔧 [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)
7. 🤝 [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)
8. ⚠️ [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md) - Keep this handy!

### Path 4: Deep Technical Understanding

**Time:** 4-6 weeks
**Goal:** Master the entire system architecture

📚 **Read everything in order** (documents 1-22)

---

## 🧠 Key Concepts to Master

### 1. Modern React (2025)

- **React 19** with Server Components (RSC)
- **React Compiler** (automatic memoization)
- Actions API (async operations)
- New hooks: `useActionState`, `useFormStatus`, `useOptimistic`
- **Critical:** React 19 is very different from React 18!

### 2. Next.js 15 App Router

- **Server Components** by default
- App Router vs Pages Router (Pages Router is legacy)
- Server Actions
- Route Handlers
- **Critical:** Caching behavior changed in Next.js 15!

### 3. TypeScript Patterns

- Type-safe schemas with Drizzle
- Payload type generation
- Strict TypeScript configuration
- Advanced type patterns

### 4. Monorepo Development

- pnpm workspaces
- Turborepo build orchestration
- Package interdependencies
- Local package linking

### 5. Database Abstraction

- Drizzle ORM (TypeScript-first)
- Database adapters (MongoDB, Postgres, SQLite)
- Schema-driven development
- Migrations

### 6. Plugin Architecture

- Payload plugin system
- Composable functionality
- Storage adapters
- Rich text editors

---

## ⚠️ Important: Current vs. Outdated Patterns (Nov 2025)

**CRITICAL:** Read [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) to understand what's current!

### ✅ Current Patterns (Use These)

- React Server Components (RSC)
- React Compiler for automatic memoization
- Next.js App Router
- Drizzle ORM with identity columns
- Playwright for E2E testing
- pnpm workspaces + Turborepo

### ⚠️ Outdated Patterns (Still Work, But...)

- Manual `useMemo`/`useCallback` everywhere (React Compiler handles this now)
- Jest for new web projects (consider Vitest)
- `useEffect` for data fetching (use Server Components)
- Next.js Pages Router (use App Router)
- `serial` types in Postgres (use identity columns)

### 🚨 Deprecated (Don't Use)

- Class components in React
- Next.js AMP support (removed in Next.js 16)

---

## 🏗️ Project Architecture at a Glance

```
Payload CMS (Monorepo)
│
├─ Frontend (React 19 + Next.js 15)
│  ├─ Server Components (default)
│  ├─ Client Components (when needed)
│  └─ Admin UI (@payloadcms/ui)
│
├─ Backend (Payload Core)
│  ├─ Collections & Globals
│  ├─ Field Types
│  ├─ Access Control
│  ├─ Hooks & Lifecycle
│  └─ API Layer (REST + GraphQL)
│
├─ Database Layer
│  ├─ Drizzle ORM
│  ├─ Adapters (MongoDB, Postgres, SQLite, etc.)
│  └─ Migrations
│
└─ Plugin System
   ├─ Storage (S3, Azure, GCS, etc.)
   ├─ Rich Text (Lexical, Slate)
   ├─ Email (Nodemailer, Resend)
   └─ Features (SEO, Search, Stripe, etc.)
```

---

## 📦 Technology Stack (Nov 2025)

| Category            | Technology | Version | Status                             |
| ------------------- | ---------- | ------- | ---------------------------------- |
| **Frontend**        | React      | 19.1.1  | ✅ Current                         |
| **Framework**       | Next.js    | 15.4.7  | ✅ Current                         |
| **Language**        | TypeScript | 5.7.3   | ⚠️ Slightly behind (5.9 available) |
| **ORM**             | Drizzle    | 0.44.6  | ✅ Current                         |
| **Testing (Unit)**  | Jest       | 29.7.0  | ⚠️ Consider Vitest                 |
| **Testing (E2E)**   | Playwright | 1.56.1  | ✅ Latest!                         |
| **Package Manager** | pnpm       | 9.7.1   | ⚠️ Behind (10.x available)         |
| **Monorepo**        | Turborepo  | 2.5.4   | ⚠️ Slightly behind (2.6.1)         |

**Full analysis:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

---

## 🚀 Quick Start

```bash
# 1. Prerequisites
node --version  # Should be >= 20.9.0
pnpm --version  # Should be >= 9.7.0

# 2. Install dependencies
pnpm install

# 3. Start Docker services (optional, for databases)
pnpm docker:start

# 4. Build core packages
pnpm run build:core

# 5. Start dev server (MongoDB default)
pnpm run dev

# OR with Postgres
pnpm run dev:postgres

# 6. Access admin panel
# http://localhost:3000/admin
# Auto-login: dev@payloadcms.com / test
```

**Full setup guide:** [GETTING_STARTED.md](./GETTING_STARTED.md)

---

## 🧪 Testing Quick Reference

```bash
# Run all tests
pnpm test

# Integration tests (recommended)
pnpm test:int

# Specific test suite
pnpm test:int fields

# E2E tests
pnpm test:e2e

# E2E with UI
pnpm test:e2e:headed
```

**Full testing guide:** [TESTING_GUIDE.md](./TESTING_GUIDE.md)

---

## 🔑 Key Files to Understand

| File/Directory      | Purpose                      | Read First?  |
| ------------------- | ---------------------------- | ------------ |
| `/CLAUDE.md`        | Project structure, commands  | ✅ Yes       |
| `/package.json`     | Root workspace configuration | ✅ Yes       |
| `/packages/payload` | Core CMS logic               | After basics |
| `/packages/ui`      | Admin UI components          | After basics |
| `/packages/next`    | Next.js integration          | After basics |
| `/test/_community`  | Example configuration        | ✅ Yes       |
| `/docs`             | Official documentation       | Reference    |

---

## 💡 Learning Tips

### For Frontend Developers

**Bridges to Backend:**

- **React Components** → **Server Components** (like components, but run on server)
- **useEffect + fetch** → **Server Components** (just fetch directly, no useEffect)
- **Client State** → **Server State** (data lives on server by default)
- **REST API** → **Server Actions** (like RPC, simpler than REST)

**New Mental Models:**

- **RSC:** Think "server-side components" not "server-side rendering"
- **Database:** Think "structured data store" not "backend magic"
- **ORM:** Think "TypeScript for databases" (Drizzle)

### For Backend Developers

**Bridges to Frontend:**

- **Templates** → **Components** (reusable UI pieces)
- **Server Rendering** → **Server Components** (but with React)
- **API Routes** → **Route Handlers** (similar, but in app/ directory)
- **Database Models** → **Drizzle Schemas** (similar, but TypeScript-first)

**New Mental Models:**

- **React:** Think "composable UI functions" not "templates"
- **Server Components:** Like server rendering, but compositional
- **Client Components:** Run in browser (like regular JS)

---

## 🎯 Success Milestones

Track your progress:

- [ ] Environment set up and dev server running
- [ ] Understand monorepo structure (45 packages)
- [ ] Can navigate codebase confidently
- [ ] Understand React Server Components
- [ ] Understand Payload collections/globals
- [ ] Understand database adapters
- [ ] Can run and write tests
- [ ] Made first code contribution
- [ ] Can debug common issues independently
- [ ] Understand full request/response flow

---

## 🆘 Getting Help

1. **Check FAQ:** [FAQ.md](./FAQ.md)
2. **Debugging Guide:** [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)
3. **Search Issues:** [GitHub Issues](https://github.com/payloadcms/payload/issues)
4. **Community:** [Payload Discord](https://discord.gg/payload)
5. **Documentation:** [Official Docs](https://payloadcms.com/docs)

---

## 📖 External Resources

### Official Documentation (Current - Nov 2025)

- React 19: https://react.dev/
- Next.js 15: https://nextjs.org/docs
- TypeScript: https://www.typescriptlang.org/docs/
- Drizzle ORM: https://orm.drizzle.team/
- Playwright: https://playwright.dev/
- Payload CMS: https://payloadcms.com/docs

### Key Announcements (2025)

- [React Compiler v1.0](https://react.dev/blog/2025/10/07/react-compiler-1)
- [React 19.2](https://react.dev/blog/2025/10/01/react-19-2)
- [Next.js 15.5](https://nextjs.org/blog/next-15-5)
- [Playwright 1.56](https://playwright.dev/docs/release-notes#version-156)

---

## 🤝 Contributing

Ready to contribute? Start here:

1. Read [CONTRIBUTING.md](../../CONTRIBUTING.md) in the root
2. Review [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)
3. Check [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)
4. Find a [good first issue](https://github.com/payloadcms/payload/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)

---

## 📊 Documentation Status

| Phase   | Status         | Documents              |
| ------- | -------------- | ---------------------- |
| Phase 0 | ✅ Complete    | TECH_STACK_RESEARCH.md |
| Phase 1 | ✅ Complete    | EXECUTION_PLAN.md      |
| Phase 2 | ✅ Complete    | README.md (this file)  |
| Phase 3 | 🚧 In Progress | Foundation docs (5)    |
| Phase 4 | ⏳ Pending     | Deep dives (4)         |
| Phase 5 | ⏳ Pending     | Practical guides (4)   |
| Phase 6 | ⏳ Pending     | Reference docs (5)     |
| Phase 7 | ⏳ Pending     | Exercises (2)          |
| Phase 8 | ⏳ Pending     | Finalization           |

**Total Progress:** 3/23 documents complete

---

## 🏃 Start Learning!

**Ready to begin?**

1. 📚 **First:** Read [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) to understand current (2025) patterns
2. 🚀 **Then:** Follow [GETTING_STARTED.md](./GETTING_STARTED.md) to set up your environment
3. 🗺️ **Choose:** Pick your learning path above based on your background

**Welcome to Payload CMS! Let's build something amazing together.** 🎉

---

_Last updated: November 18, 2025_
_For questions or improvements to this learning path, please open an issue._
