# Getting Started with Payload CMS

**Documented:** November 18, 2025
**Project Version:** Payload CMS v3.64.0
**Tech Stack:** [View Research](./TECH_STACK_RESEARCH.md)

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Initial Setup](#initial-setup)
- [Understanding the Setup](#understanding-the-setup)
- [Running the Development Server](#running-the-development-server)
- [Development Environment Options](#development-environment-options)
- [Verifying Your Setup](#verifying-your-setup)
- [Running Tests](#running-tests)
- [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)
- [Understanding What Just Happened](#understanding-what-just-happened)
- [Next Steps](#next-steps)

---

## Prerequisites

Before you begin, you'll need these tools installed on your machine.

### Required Tools

#### 1. Node.js (v20.9.0 or higher)

**Current Project Requirement:** `^18.20.2 || >=20.9.0` ([package.json:202](../../package.json#L202))
**Recommended for Nov 2025:** Node.js 22 LTS or 24 LTS

🧠 **Mental Model for React Developers:**
Think of Node.js as the "runtime" for JavaScript outside the browser - like how Chrome runs JS in the browser, Node runs it on your computer/server.

**Check your version:**

```bash
node --version
# Should output: v20.9.0 or higher (v22.x or v24.x recommended)
```

**Install/Update Node.js:**

- **Recommended:** Use [nvm](https://github.com/nvm-sh/nvm) (Node Version Manager)

  ```bash
  # Install nvm first (if you don't have it)
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

  # Install and use the correct Node version
  nvm install 22  # or 24
  nvm use 22
  ```

- **Direct Install:** Download from [nodejs.org](https://nodejs.org/)

🎯 **Why this version?**

- Node.js 22 is Active LTS (Long Term Support)
- Node.js 24 is the newest LTS (supported until April 2028)
- This project uses modern Node.js features like `import`/`export` (ESM)

---

#### 2. pnpm (v9.7.0 or higher)

**Current Project Requirement:** `^9.7.0` ([package.json:203](../../package.json#L203))
**Latest Available:** v10.18.1

🧠 **Mental Model:**

- **npm** = The default package manager (like App Store for JavaScript)
- **pnpm** = Faster, more efficient npm (saves disk space, faster installs)
- For monorepos like this, pnpm is **much faster** than npm

**Check your version:**

```bash
pnpm --version
# Should output: 9.7.0 or higher
```

**Install pnpm:**

```bash
# Using npm (ironic, but easiest way)
npm install -g pnpm

# OR using standalone script (recommended)
curl -fsSL https://get.pnpm.io/install.sh | sh -

# OR using Homebrew (macOS)
brew install pnpm
```

🌉 **Bridge from npm:**
| npm command | pnpm equivalent | What it does |
|-------------|-----------------|--------------|
| `npm install` | `pnpm install` | Install dependencies |
| `npm run dev` | `pnpm run dev` or `pnpm dev` | Run script |
| `npm install <pkg>` | `pnpm add <pkg>` | Add package |
| `npm install -g <pkg>` | `pnpm add -g <pkg>` | Global install |

💡 **Aha Moment:**
pnpm creates a single store of packages on your machine and uses hard links. If React 19.1.1 is installed in 5 projects, npm stores it 5 times, pnpm stores it once!

---

#### 3. Docker (Optional, but Recommended)

**Required for:** Running local databases (MongoDB, PostgreSQL, etc.) and storage services

**Check if installed:**

```bash
docker --version
docker compose version
```

**Install Docker:**

- **macOS/Windows:** [Docker Desktop](https://www.docker.com/products/docker-desktop)
- **Linux:** [Docker Engine](https://docs.docker.com/engine/install/)

🧠 **Mental Model:**
Docker = Virtual machines but lightweight. Think of it as "pre-configured computers" that run databases and services without cluttering your machine.

⚠️ **Optional but Highly Recommended:**
You CAN use in-memory MongoDB (no Docker needed), but Docker gives you the real experience:

- Real databases (MongoDB, PostgreSQL)
- Storage services (S3, Azure, GCS emulators)
- Redis for caching

---

#### 4. Git (Obviously!)

You're likely reading this from a git repo, so you have it. But just in case:

```bash
git --version
# Should output: 2.x or higher
```

---

### Recommended Tools

These aren't required but will make your life easier:

#### 1. VS Code (or your favorite IDE)

**Recommended Extensions for VS Code:**

- **ESLint** - Linting (the project uses ESLint)
- **Prettier** - Code formatting (configured in [.prettierrc.json](../../.prettierrc.json))
- **Error Lens** - Inline error messages
- **TypeScript + JavaScript Language Features** - Built-in, ensure it's enabled
- **Payload CMS** - Official Payload extension (if available)

#### 2. MongoDB Compass (Optional)

For visualizing MongoDB data:

- Download: [MongoDB Compass](https://www.mongodb.com/products/compass)

#### 3. Postman or Insomnia (Optional)

For testing REST/GraphQL APIs:

- [Postman](https://www.postman.com/)
- [Insomnia](https://insomnia.rest/)

---

## Initial Setup

Now that you have the prerequisites, let's set up the project!

### Step 1: Clone the Repository (If You Haven't)

```bash
# Clone the repository
git clone https://github.com/payloadcms/payload.git
cd payload

# OR if you already have it
cd /path/to/payload
```

---

### Step 2: Install Dependencies

This is where pnpm's speed shines. The project has **45 packages** in the monorepo!

```bash
pnpm install
```

**What's happening:**

1. pnpm reads [`package.json`](../../package.json) and [`pnpm-workspace.yaml`](../../pnpm-workspace.yaml)
2. It finds all packages in `/packages/*` and `/test/*`
3. It installs dependencies for ALL packages
4. It creates symlinks between local packages

**Time:** First install takes 2-5 minutes depending on your internet speed

**Output you'll see:**

```
Lockfile is up to date, resolution step is skipped
Packages: +2847
++++++++++++++++++++++++++++++++++++++++++++++++
Progress: resolved 2847, reused 2847, downloaded 0, added 2847, done
```

🎯 **Remember This:** `pnpm install` runs from the **root** of the monorepo and installs dependencies for ALL 45+ packages.

⚠️ **Common Pitfall:**
DON'T run `npm install` by accident! The project uses pnpm and has a `pnpm-lock.yaml` file. Using npm creates conflicts.

---

### Step 3: Build Core Packages

Before you can run the dev server, you need to build the core packages.

```bash
pnpm run build:core
```

**What's happening:**
This builds these essential packages:

- `payload` - Core CMS
- `@payloadcms/ui` - Admin UI components
- `@payloadcms/next` - Next.js integration
- `@payloadcms/db-mongodb` - MongoDB adapter
- `@payloadcms/db-postgres` - PostgreSQL adapter
- `@payloadcms/richtext-lexical` - Lexical editor
- `@payloadcms/translations` - i18n translations
- `@payloadcms/graphql` - GraphQL layer

**Excludes:** Plugins (`plugin-*`) and storage adapters (`storage-*`) - built separately if needed

**Time:** 3-8 minutes on first build

**Output:**

```
Tasks:    37 successful, 37 total
Cached:   0 cached, 37 total
Time:     4m32.157s
```

🧠 **Mental Model - Monorepo Builds:**
Think of the monorepo as a city:

- Each package is a building
- Some buildings depend on others (you can't build the roof before the foundation)
- Turborepo is the construction manager that builds in the right order

🌉 **Bridge from React Projects:**
In a typical React project: `npm run build` builds YOUR app
In this monorepo: `pnpm run build:core` builds the **libraries** that make up Payload CMS

💡 **Aha Moment:**
You're not building an app - you're building the **framework itself**! The test directories are like "example apps" that use the framework.

---

### Step 4: Start Docker Services (Optional)

If you installed Docker and want real databases:

```bash
pnpm docker:start
```

**What's happening:**
This starts services defined in [`test/docker-compose.yml`](../../test/docker-compose.yml#L1-L53):

- **LocalStack** (port 4563-4599, 8055) - S3-compatible storage emulator
- **Azurite** (ports 10000-10002) - Azure Storage emulator
- **Fake GCS Server** (port 4443) - Google Cloud Storage emulator

**You'll see:**

```
[+] Running 3/3
 ✔ Container localstack_demo    Started
 ✔ Container azure-storage       Started
 ✔ Container google-cloud-storage Started
```

**To stop services later:**

```bash
pnpm docker:stop
```

**To restart services:**

```bash
pnpm docker:restart
```

🎯 **Remember This:** Docker services are for **storage testing** (S3, Azure, GCS). The dev server can use **in-memory MongoDB** without Docker!

---

## Understanding the Setup

Before running the dev server, let's understand what we just set up.

### What Did `pnpm install` Do?

**Created directories:**

```
payload/
├── node_modules/           # Dependencies for root workspace
├── packages/
│   ├── payload/
│   │   └── node_modules/   # Dependencies for @payloadcms/payload
│   ├── ui/
│   │   └── node_modules/   # Dependencies for @payloadcms/ui
│   └── ... (43 more packages)
└── pnpm-lock.yaml          # Exact versions of all dependencies
```

**Key files created/updated:**

- `node_modules/` - All package dependencies
- `pnpm-lock.yaml` - Lockfile (like package-lock.json for npm)
- `.pnpm-store/` (hidden, outside repo) - Global pnpm store

### What Did `pnpm run build:core` Do?

Each package was compiled from TypeScript to JavaScript:

```
packages/payload/
├── src/                    # Source code (TypeScript)
│   └── index.ts
└── dist/                   # Built code (JavaScript) ← Created by build
    └── index.js
```

**Build tools used:**

- **SWC** ([.swcrc](../../.swcrc)) - Super-fast TypeScript compiler (replaces Babel)
- **TypeScript** (`tsc`) - Type checking and .d.ts generation
- **esbuild** (in some packages) - JavaScript bundler
- **Turborepo** - Orchestrates builds across all packages

🌉 **Bridge from React (Create React App / Vite):**
| Your React App | Payload Monorepo |
|----------------|------------------|
| `npm run build` → build **your app** | `pnpm run build:core` → build **the framework** |
| Output: `dist/` or `build/` | Output: Each package gets its own `dist/` |
| Takes 10-30s | Takes 3-8 minutes (building 20+ packages!) |

---

## Running the Development Server

Now for the fun part - let's start the dev server!

### Default Dev Server (In-Memory MongoDB)

```bash
pnpm run dev
```

**What's happening:**

1. Runs [`test/dev.ts`](../../test/dev.ts) script
2. Uses **in-memory MongoDB** (no Docker needed!)
3. Loads [`test/_community/config.ts`](../../test/_community/config.ts) (default test config)
4. Starts Next.js dev server on port 3000
5. Enables **Turbopack** (Next.js fast bundler) by default
6. Auto-login enabled: `dev@payloadcms.com` / `test`

**You'll see:**

```
Selected test suite: _community [Turbopack]
✓ Running on port 3000
▲ Next.js 15.4.7
- Local:        http://localhost:3000
- Environments: .env

✓ Starting...
✓ Ready in 2.3s
```

**Access the admin panel:**
Open your browser to: **http://localhost:3000/admin**

💡 **Aha Moment:**
You're running Payload CMS using the `_community` test configuration - this is a **minimal example** showing how to configure Payload!

### Check Out the Example Config

The dev server uses [`test/_community/config.ts`](../../test/_community/config.ts#L14-L46):

```typescript
export default buildConfigWithDefaults({
  collections: [PostsCollection, MediaCollection],
  editor: lexicalEditor({}),
  globals: [MenuGlobal],
  onInit: async (payload) => {
    // Auto-creates a user on startup
    await payload.create({
      collection: 'users',
      data: {
        email: devUser.email,
        password: devUser.password,
      },
    })
  },
})
```

🧠 **Mental Model:**

- **Collections** = Database tables/models (like "Posts", "Media")
- **Globals** = Single documents (like "Site Settings", "Menu")
- **Editor** = Rich text editor (Lexical in this case)
- **onInit** = Runs when Payload starts (creates seed data)

---

### Dev Server with PostgreSQL

If you prefer PostgreSQL over MongoDB:

```bash
pnpm run dev:postgres
```

**What's different:**

- Uses **PostgreSQL** instead of in-memory MongoDB
- Requires Docker or a local PostgreSQL instance
- Connection string from `DATABASE_URL` env variable

🎯 **Remember This:** Payload is **database-agnostic**! You can use MongoDB, PostgreSQL, SQLite, or others.

---

### Dev Server with Specific Test Config

The `test/` directory has **80+ test configurations** showcasing different features!

```bash
# Run with "fields" test configuration
pnpm run dev fields

# Run with "auth" test configuration
pnpm run dev auth

# Run with "uploads" test configuration
pnpm run dev uploads
```

**Explore test configs:**

```bash
ls test/
# You'll see: access-control, admin, auth, fields, uploads, etc.
```

Each directory has:

- `config.ts` - Payload configuration
- `int.spec.ts` - Integration tests
- `e2e.spec.ts` - E2E tests (Playwright)

💡 **Aha Moment:**
These test directories are **learning gold**! Each one demonstrates a specific Payload feature in isolation.

---

## Development Environment Options

### Environment Variables

The project uses `.env` files for configuration:

**Create your own `.env` file (optional):**

```bash
# In the root directory
touch .env
```

**Common variables:**

```bash
# Database (if not using in-memory)
DATABASE_URL=mongodb://localhost:27017/payload
# OR
DATABASE_URL=postgresql://user:password@localhost:5432/payload

# Disable auto-login
PAYLOAD_PUBLIC_DISABLE_AUTO_LOGIN=true

# Enable verbose logging
PAYLOAD_LOG_LEVEL=debug

# Use specific database adapter
PAYLOAD_DATABASE=postgres  # or 'mongodb', 'sqlite'
```

📖 **See:** [`.env.example`](../../.env.example) (if it exists) or check [`test/dev.ts`](../../test/dev.ts#L35) for how env vars are loaded

---

### Auto-Login

**Default credentials:**

- Email: `dev@payloadcms.com`
- Password: `test`

**Defined in:** [`test/credentials.ts`](../../test/credentials.ts)

**To disable auto-login:**

```bash
pnpm run dev --no-auto-login
# OR set env variable:
PAYLOAD_PUBLIC_DISABLE_AUTO_LOGIN=true pnpm run dev
```

🎯 **Remember This:** Auto-login is **ONLY for development**! Never enable in production.

---

### Turbopack vs Webpack

**Default:** Turbopack (faster, Next.js new bundler)

**Disable Turbopack (use Webpack):**

```bash
pnpm run dev --no-turbo
```

🌉 **Bridge from Create React App / Vite:**
| Tool | Bundler | Speed |
|------|---------|-------|
| Create React App | Webpack | Slower |
| Vite | esbuild + Rollup | Fast |
| Next.js 15 (default) | **Turbopack** | **Very Fast** |
| Next.js 15 (fallback) | Webpack | Slower |

✅ **CURRENT (Nov 2025):** Turbopack is **production-ready** (beta) in Next.js 15.5

---

### Memory Database vs Real Database

**In-memory MongoDB (default):**

```bash
pnpm run dev:memorydb
```

- **Pros:** No Docker/installation needed, fast startup
- **Cons:** Data lost on restart, not real MongoDB

**Real MongoDB (Docker):**

```bash
# Terminal 1: Start MongoDB
docker run -d -p 27017:27017 --name mongo mongo:latest

# Terminal 2: Run dev server
DATABASE_URL=mongodb://localhost:27017/payload pnpm run dev
```

**Real PostgreSQL:**

```bash
pnpm run dev:postgres
```

🎯 **Remember This:** Use **in-memory** for quick testing, **real database** for realistic development.

---

## Verifying Your Setup

### 1. Check the Admin Panel

1. Open http://localhost:3000/admin
2. You should see the Payload login screen
3. Auto-login should work (or use `dev@payloadcms.com` / `test`)
4. You should see:
   - **Collections:** Posts, Media, Users
   - **Globals:** Menu
   - Dashboard with "example post"

### 2. Check the API

**REST API:**

```bash
# Get all posts
curl http://localhost:3000/api/posts

# Should return JSON:
{
  "docs": [
    {
      "id": "...",
      "title": "example post",
      "createdAt": "...",
      "updatedAt": "..."
    }
  ],
  "totalDocs": 1,
  "limit": 10,
  "totalPages": 1,
  "page": 1
}
```

**GraphQL API:**

```bash
# GraphQL playground (if enabled)
open http://localhost:3000/api/graphql
```

✅ **If you see these, your setup is working!**

---

## Running Tests

Payload has comprehensive tests. Let's run some!

### Integration Tests (Recommended Start)

**Run all integration tests:**

```bash
pnpm test:int
```

⚠️ **Warning:** This runs **ALL** integration tests - takes 10-30 minutes!

**Run specific test suite:**

```bash
# Test just the "fields" functionality
pnpm test:int fields

# Test authentication
pnpm test:int auth

# Test file uploads
pnpm test:int uploads
```

**What's happening:**

1. Jest starts
2. Test creates a Payload instance with test config
3. Seeds test data
4. Runs API tests against MongoDB in-memory
5. Reports results

**Output:**

```
PASS test/fields/int.spec.ts
  ✓ Text Field (45 ms)
  ✓ Number Field (23 ms)
  ✓ Email Field (31 ms)

Test Suites: 1 passed, 1 total
Tests:       28 passed, 28 total
```

🧠 **Mental Model - Integration Tests:**
Think of integration tests as "API tests" - they test the **whole system** (database + API + validation) working together.

---

### E2E Tests (Playwright)

E2E tests run in a real browser!

**Run all E2E tests (headless):**

```bash
pnpm test:e2e
```

**Run with visible browser:**

```bash
pnpm test:e2e:headed
```

**Debug mode (opens Playwright Inspector):**

```bash
pnpm test:e2e:debug
```

**What's happening:**

1. Playwright launches Chrome/Firefox/Safari
2. Navigates to admin panel
3. Clicks buttons, fills forms, etc.
4. Asserts UI works correctly

💡 **Aha Moment:**

- **Integration tests** = Backend/API tests (no browser)
- **E2E tests** = Full user flow tests (with browser)

---

### Unit Tests

**Run unit tests:**

```bash
pnpm test:unit
```

These test individual functions/utilities in isolation.

---

### Component Tests

**Run React component tests:**

```bash
pnpm test:components
```

Tests React components using Jest + React Testing Library.

---

## Common Issues and Troubleshooting

### Issue 1: Port 3000 Already in Use

**Error:**

```
Error: listen EADDRINUSE: address already in use :::3000
```

**Solution:**

```bash
# Find what's using port 3000
lsof -i :3000  # macOS/Linux
netstat -ano | findstr :3000  # Windows

# Kill the process or use different port
PORT=3001 pnpm run dev
```

---

### Issue 2: pnpm Not Found

**Error:**

```
pnpm: command not found
```

**Solution:**

```bash
# Install pnpm globally
npm install -g pnpm

# Verify
pnpm --version
```

---

### Issue 3: Build Fails - TypeScript Errors

**Error:**

```
Error: TS2304: Cannot find name 'XYZ'
```

**Solution:**

```bash
# Clean and rebuild
pnpm clean:build
pnpm run build:core
```

---

### Issue 4: Module Not Found After Install

**Error:**

```
Cannot find module '@payloadcms/ui'
```

**Solution:**

```bash
# Reinstall dependencies
pnpm install

# If still broken, clean everything
pnpm clean:all
pnpm install
pnpm run build:core
```

---

### Issue 5: Docker Services Won't Start

**Error:**

```
Error: port is already allocated
```

**Solution:**

```bash
# Stop all containers
docker stop $(docker ps -aq)

# Remove stopped containers
docker rm $(docker ps -aq)

# Restart
pnpm docker:start
```

---

### Issue 6: Slow Build Times

**Symptoms:** `build:core` taking >10 minutes

**Solutions:**

```bash
# 1. Use Turborepo cache
# Already enabled by default

# 2. Build specific packages only
pnpm run build:payload
pnpm run build:ui
pnpm run build:next

# 3. Skip certain packages (be careful!)
pnpm run build:core --filter="!@payloadcms/plugin-*"
```

---

### Issue 7: Database Connection Errors

**Error (MongoDB):**

```
MongoNetworkError: connect ECONNREFUSED 127.0.0.1:27017
```

**Solution:**

```bash
# Use in-memory MongoDB instead
pnpm run dev:memorydb

# OR start MongoDB
docker run -d -p 27017:27017 mongo:latest
```

**Error (PostgreSQL):**

```
Connection refused: Is the server running?
```

**Solution:**

```bash
# Check PostgreSQL is running
docker ps | grep postgres

# Start PostgreSQL
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=password postgres:latest
```

---

## Understanding What Just Happened

Let's break down what you just set up from a React developer's perspective.

### The Monorepo Structure

🧠 **Mental Model:**
A monorepo is like a **multi-app repository**:

- Your typical project: 1 repo = 1 app
- Monorepo: 1 repo = many packages (45 in this case!)

**Why?**

- All Payload packages developed together
- Shared tooling (ESLint, TypeScript config)
- Easy to test interactions between packages
- Atomic commits across packages

🌉 **Bridge from React:**
You probably use packages like `react`, `react-dom` separately.
Payload builds ALL its packages (`@payloadcms/ui`, `@payloadcms/next`, etc.) in ONE repo!

---

### The Build Process

**What `pnpm run build:core` actually does:**

```bash
# From root package.json
"build:core": "turbo build --filter \"!@payloadcms/plugin-*\" --filter \"!@payloadcms/storage-*\" ..."
```

**Breakdown:**

1. **turbo build** - Turborepo CLI command
2. **--filter** - Build only matching packages
3. **"!@payloadcms/plugin-\*"** - Exclude plugins (not needed for basic dev)

**Turborepo's magic:**

- Builds packages in **dependency order**
- **Caches** builds (second build is instant if nothing changed!)
- Runs builds in **parallel** when possible

🎯 **Remember This:**
`turbo build` = smart `tsc` (TypeScript Compiler) that knows about package dependencies

---

### The Dev Server

**What `pnpm run dev` actually does:**

Looking at [`test/dev.ts`](../../test/dev.ts#L1-L100):

```typescript
// 1. Load environment variables
loadEnv()

// 2. Parse command line args
const {
  _: [testSuiteArg = '_community'],
} = minimist(process.argv)

// 3. Start memory database (if requested)
if (shouldStartMemoryDB) {
  await startMemoryDB()
}

// 4. Initialize Payload with test config
await runInit(testSuiteArg, true)

// 5. Start Next.js dev server
const app = next({ dev: true, dir: rootDir })
await app.prepare()
```

🧠 **Mental Model:**
The dev server is **Next.js** running with **Payload** as middleware.

**Flow:**

```
Browser Request
    ↓
Next.js Server (Port 3000)
    ↓
Payload Middleware
    ↓
Your Collections/Globals
    ↓
Database (MongoDB/PostgreSQL)
```

---

### The Auto-Login

Looking at the config's `onInit` hook ([`test/_community/config.ts:27-34`](../../test/_community/config.ts#L27-L34)):

```typescript
onInit: async (payload) => {
  await payload.create({
    collection: 'users',
    data: {
      email: devUser.email, // 'dev@payloadcms.com'
      password: devUser.password, // 'test'
    },
  })
}
```

This creates the user **every time** Payload starts (in dev mode).

🎯 **Remember This:** `onInit` is like React's `useEffect` but for Payload startup!

---

## Next Steps

Congratulations! You have a working Payload CMS development environment. 🎉

### Recommended Learning Path

1. **Explore the Admin Panel** (5-10 min)
   - Create a new Post
   - Upload an image to Media
   - Edit the Menu global
   - Observe the API calls in browser DevTools

2. **Read the Example Config** (10-15 min)
   - [`test/_community/config.ts`](../../test/_community/config.ts)
   - [`test/_community/collections/Posts/index.ts`](../../test/_community/collections/Posts/)
   - Understand the structure

3. **Try Another Test Config** (10 min)

   ```bash
   pnpm run dev fields
   ```

   - See how different field types work
   - Check [`test/fields/config.ts`](../../test/fields/config.ts)

4. **Run Some Tests** (15-20 min)

   ```bash
   pnpm test:int fields
   ```

   - See how integration tests work
   - Check [`test/fields/int.spec.ts`](../../test/fields/int.spec.ts)

5. **Read Next Documents**
   - [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md) - Understand the system design
   - [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) - Navigate the codebase
   - [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md) - Deep dive into technologies

---

## Quick Reference

### Essential Commands

```bash
# Install dependencies
pnpm install

# Build core packages
pnpm run build:core

# Start dev server (in-memory MongoDB)
pnpm run dev

# Start dev server (PostgreSQL)
pnpm run dev:postgres

# Start dev server (specific test config)
pnpm run dev <test-directory>

# Run tests
pnpm test:int <test-directory>
pnpm test:e2e

# Docker services
pnpm docker:start
pnpm docker:stop
pnpm docker:restart

# Clean and rebuild
pnpm clean:build
pnpm run build:core
```

### Important URLs

- **Admin Panel:** http://localhost:3000/admin
- **REST API:** http://localhost:3000/api/
- **GraphQL Playground:** http://localhost:3000/api/graphql

### Default Credentials

- **Email:** dev@payloadcms.com
- **Password:** test

---

## Questions?

- **FAQ:** [FAQ.md](./FAQ.md)
- **Troubleshooting:** [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)
- **Community:** [Payload Discord](https://discord.gg/payload)

---

_Happy coding! You're now ready to dive into Payload CMS development._ 🚀
