# Project Structure Guide

**Documented:** November 18, 2025  
**Tech Stack:** [View Research](./TECH_STACK_RESEARCH.md)

This guide provides a comprehensive directory-by-directory breakdown of the Payload CMS monorepo.

## Quick Navigation

- [Root Directory](#root-directory)
- [Packages Directory](#packages-directory)
- [Test Directory](#test-directory)
- [Configuration Files](#configuration-files)

## Root Directory

```
payload/
├── #claude/              # Claude AI agent configuration
├── .github/              # GitHub Actions, templates
├── .husky/               # Git hooks (pre-commit, pre-push)
├── app/                  # Development Next.js app
├── docs/                 # Documentation (deployed to payloadcms.com)
├── examples/             # Example implementations
├── node_modules/         # Root dependencies
├── packages/             # Monorepo packages (45 packages)
├── public/               # Static assets
├── scripts/              # Build and utility scripts
├── templates/            # Production-ready templates
├── test/                 # Test suites (80+ directories)
├── tools/                # Monorepo tooling
└── package.json          # Root workspace configuration
```

## Packages Directory

All publishable packages live here. Each follows the structure:

```
packages/{package-name}/
├── src/                  # TypeScript source code
├── dist/                 # Compiled JavaScript (after build)
├── package.json          # Package configuration
├── README.md             # Package documentation
└── tsconfig.json         # TypeScript configuration
```

### Core Packages

**`packages/payload`** - Core CMS engine
- Collections, Globals, Fields
- Authentication, Access Control
- Operations (CRUD)
- Hooks, Validation

**`packages/ui`** - Admin panel UI
- React Server Components
- Form components
- Field-specific UIs
- Views (List, Edit, Create)

**`packages/next`** - Next.js integration
- Layouts, Routes
- API handlers
- Admin panel integration

### Database Adapters

**`packages/db-mongodb`** - MongoDB adapter  
**`packages/db-postgres`** - PostgreSQL adapter (Drizzle)  
**`packages/db-sqlite`** - SQLite adapter  
**`packages/db-vercel-postgres`** - Vercel Postgres  
**`packages/db-d1-sqlite`** - Cloudflare D1  

### Other Core Packages

**`packages/drizzle`** - Drizzle ORM integration  
**`packages/graphql`** - GraphQL API layer  
**`packages/translations`** - i18n translations  

### Rich Text Editors

**`packages/richtext-lexical`** - Lexical editor (current)  
**`packages/richtext-slate`** - Slate editor (legacy)  

### Storage Adapters

**`packages/storage-s3`** - AWS S3  
**`packages/storage-azure`** - Azure Blob  
**`packages/storage-gcs`** - Google Cloud Storage  
**`packages/storage-uploadthing`** - Uploadthing  
**`packages/storage-vercel-blob`** - Vercel Blob  

### Email Adapters

**`packages/email-nodemailer`** - Nodemailer  
**`packages/email-resend`** - Resend  

### Plugins

**Feature Plugins:**
- `plugin-seo` - SEO meta fields
- `plugin-search` - Full-text search
- `plugin-form-builder` - Form builder
- `plugin-stripe` - Stripe payments
- `plugin-redirects` - URL redirects
- `plugin-nested-docs` - Nested structure
- `plugin-sentry` - Error tracking

### Other Packages

**`packages/create-payload-app`** - CLI for creating new apps  
**`packages/sdk`** - JavaScript SDK for Payload API  

## Test Directory

Each test directory contains a minimal Payload configuration demonstrating specific features:

```
test/{feature}/
├── config.ts              # Payload configuration
├── int.spec.ts            # Integration tests (Jest)
├── e2e.spec.ts            # E2E tests (Playwright)
├── collections/           # Collection configs
├── globals/               # Global configs
└── payload-types.ts       # Generated types
```

**Key test directories:**
- `_community/` - Default example configuration
- `fields/` - All field types
- `auth/` - Authentication
- `uploads/` - File uploads
- `access-control/` - Permissions
- `admin/` - Admin customization

## Configuration Files

**TypeScript:**
- `tsconfig.base.json` - Base TypeScript config
- `tsconfig.json` - Root config

**Build Tools:**
- `turbo.json` - Turborepo configuration
- `pnpm-workspace.yaml` - Workspace definition

**Code Quality:**
- `.eslintrc.js` - ESLint rules
- `.prettierrc.json` - Prettier formatting
- `.swcrc` - SWC compiler config

**Next Steps:**
- [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md) - Deep dive into each technology
- [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md) - How data flows through the system

---

*Navigate the codebase confidently!*
