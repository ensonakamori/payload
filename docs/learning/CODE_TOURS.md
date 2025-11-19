# Code Tours

**Documented:** November 18, 2025

Follow code execution through the system.

## Tour 1: Creating a Post

**Start:** User clicks "Create New" in admin panel

**Files visited:**
1. `/app/(payload)/admin/collections/posts/create/page.tsx`
2. `packages/ui/src/views/Create/index.tsx`
3. Server Action in form submission
4. `packages/payload/src/collections/operations/create.ts`
5. `packages/db-mongodb/src/create.ts` (or postgres)
6. Database

**Key steps:**
- Form renders with all fields
- User fills form
- Clicks "Save"
- Server Action handles submission
- Payload validates and runs hooks
- Database stores document
- Redirects to edit page

## Tour 2: REST API Query

**Start:** `GET /api/posts?limit=10`

**Files:**
1. `/app/api/posts/route.ts` (generated)
2. `packages/next/src/routes/rest/find.ts`
3. `packages/payload/src/collections/operations/find.ts`
4. Database adapter
5. Database

**Key transformations:**
- URL params → Payload query object
- Payload query → Database query
- Database results → PaginatedDocs
- PaginatedDocs → JSON response

## Tour 3: Authentication Flow

**Start:** User submits login form

**Files:**
1. Login form component
2. Server Action
3. `packages/payload/src/auth/operations/login.ts`
4. `packages/payload/src/auth/operations/local/login.ts`
5. Database (find user)
6. Password verification
7. JWT generation

**Result:** JWT token in httpOnly cookie

## Tour 4: File Upload

**Start:** User uploads image

**Files:**
1. Upload field component
2. Server Action
3. `packages/payload/src/uploads/uploadFile.ts`
4. Image processing (sharp)
5. Storage adapter (s3, local, etc.)
6. Database (save file metadata)

## Tour 5: Hook Execution

**Trace hook execution during create:**

```typescript
1. beforeValidate hooks
2. Field validation
3. beforeChange hooks
4. Database write
5. afterChange hooks
6. afterRead hooks (when retrieving)
```

**Files:**
- `packages/payload/src/collections/operations/create.ts`
- Hook execution in operation flow

---

*Follow the code, understand the flow!*
