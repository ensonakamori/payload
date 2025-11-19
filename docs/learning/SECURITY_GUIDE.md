# Security Guide

**Documented:** November 18, 2025

Security best practices for Payload CMS applications.

## Mental Model: Defense in Depth

**For React Developers:**
In client-side React, security often means XSS protection and input sanitization. In full-stack Payload, you need to think about multiple layers:

```
Client Security
    ↓
Network Security
    ↓
Server Security
    ↓
Database Security
    ↓
Infrastructure Security
```

**Key Principle:** Never trust the client. All security checks must happen server-side.

## Authentication & Authorization

### Authentication Setup

**Enable authentication on user collection:**

```typescript
{
  slug: 'users',
  auth: true, // Enables JWT authentication
  fields: [
    {
      name: 'email',
      type: 'email',
      required: true,
      unique: true
    },
    {
      name: 'role',
      type: 'select',
      options: ['admin', 'editor', 'user'],
      required: true,
      defaultValue: 'user'
    }
  ]
}
```

### Password Security

**Payload handles:**

- ✅ Automatic bcrypt hashing (cost factor: 10)
- ✅ Password complexity validation
- ✅ Secure password reset flow
- ✅ No plaintext password storage

**Enforce strong passwords:**

```typescript
{
  slug: 'users',
  auth: {
    verify: true, // Email verification
    maxLoginAttempts: 5, // Prevent brute force
    lockTime: 600000, // 10 minutes lockout
    tokenExpiration: 7200, // 2 hour token expiry
  },
  fields: [
    {
      name: 'password',
      type: 'text',
      required: true,
      validate: (value) => {
        // Enforce password complexity
        if (value.length < 12) {
          return 'Password must be at least 12 characters'
        }
        if (!/[A-Z]/.test(value)) {
          return 'Password must contain uppercase letter'
        }
        if (!/[a-z]/.test(value)) {
          return 'Password must contain lowercase letter'
        }
        if (!/[0-9]/.test(value)) {
          return 'Password must contain number'
        }
        if (!/[^A-Za-z0-9]/.test(value)) {
          return 'Password must contain special character'
        }
        return true
      }
    }
  ]
}
```

### JWT Security

**Configure secure JWT settings:**

```typescript
// payload.config.ts
export default buildConfig({
  secret: process.env.PAYLOAD_SECRET, // REQUIRED: 32+ char random string
  cookiePrefix: 'payload', // Cookie name prefix
  csrf: process.env.CSRF_DOMAINS?.split(',') || [], // CSRF protection
  collections: [
    {
      slug: 'users',
      auth: {
        tokenExpiration: 7200, // 2 hours (short = more secure)
        cookies: {
          secure: process.env.NODE_ENV === 'production', // HTTPS only
          sameSite: 'lax', // or 'strict' for more security
          domain: process.env.COOKIE_DOMAIN,
        },
      },
    },
  ],
})
```

**Generate secure PAYLOAD_SECRET:**

```bash
# Generate 32-byte random string
openssl rand -base64 32

# Add to .env.local
PAYLOAD_SECRET=<generated-secret>
```

**⚠️ CRITICAL:**

- **NEVER** commit `.env` files to git
- **NEVER** use default/example secrets in production
- **ROTATE** secrets if leaked

### Access Control

**Role-Based Access Control (RBAC):**

```typescript
{
  slug: 'posts',
  access: {
    // Who can create?
    create: ({ req }) => {
      // Public cannot create
      if (!req.user) return false

      // Admins and editors can create
      return ['admin', 'editor'].includes(req.user.role)
    },

    // Who can read?
    read: ({ req }) => {
      // Anyone can read published posts
      // Authenticated users can read their own drafts
      if (!req.user) {
        return {
          status: { equals: 'published' }
        }
      }

      return {
        or: [
          { status: { equals: 'published' } },
          { author: { equals: req.user.id } }
        ]
      }
    },

    // Who can update?
    update: ({ req, id }) => {
      if (!req.user) return false

      // Admins can update anything
      if (req.user.role === 'admin') return true

      // Authors can update their own posts
      return {
        author: { equals: req.user.id }
      }
    },

    // Who can delete?
    delete: ({ req }) => {
      if (!req.user) return false

      // Only admins can delete
      return req.user.role === 'admin'
    }
  }
}
```

**Field-Level Access Control:**

```typescript
{
  slug: 'users',
  fields: [
    {
      name: 'role',
      type: 'select',
      options: ['admin', 'editor', 'user'],
      // Only admins can modify roles
      access: {
        create: ({ req }) => req.user?.role === 'admin',
        update: ({ req }) => req.user?.role === 'admin'
      }
    },
    {
      name: 'salary',
      type: 'number',
      // Only admin can see salary
      access: {
        read: ({ req }) => req.user?.role === 'admin',
        update: ({ req }) => req.user?.role === 'admin'
      }
    }
  ]
}
```

### Session Security

**Implement session timeout:**

```typescript
{
  slug: 'users',
  auth: {
    tokenExpiration: 3600, // 1 hour
    cookies: {
      secure: true,
      sameSite: 'strict'
    }
  },
  hooks: {
    beforeLogin: [
      ({ req }) => {
        // Log login attempts
        console.log(`Login attempt: ${req.data.email} from ${req.ip}`)
      }
    ],
    afterLogout: [
      ({ req }) => {
        // Log logouts
        console.log(`Logout: ${req.user.email}`)
      }
    ]
  }
}
```

**Detect concurrent sessions (optional):**

```typescript
{
  slug: 'users',
  fields: [
    {
      name: 'sessionToken',
      type: 'text',
      admin: { hidden: true }
    }
  ],
  hooks: {
    afterLogin: [
      async ({ user, req, token }) => {
        // Store current session token
        await req.payload.update({
          collection: 'users',
          id: user.id,
          data: {
            sessionToken: token
          }
        })
      }
    ],
    beforeRead: [
      async ({ req, doc }) => {
        // Invalidate if session token doesn't match
        if (req.user?.sessionToken !== doc.sessionToken) {
          throw new Error('Session expired. Please login again.')
        }
      }
    ]
  }
}
```

## Input Validation & Sanitization

### Server-Side Validation

**NEVER trust client input:**

```typescript
{
  slug: 'posts',
  fields: [
    {
      name: 'title',
      type: 'text',
      required: true,
      validate: (value, { req }) => {
        // Length check
        if (value.length > 200) {
          return 'Title too long (max 200 chars)'
        }

        // Disallow malicious patterns
        if (/<script|javascript:|on\w+=/i.test(value)) {
          return 'Invalid characters detected'
        }

        return true
      }
    },
    {
      name: 'slug',
      type: 'text',
      validate: (value) => {
        // Only allow safe URL characters
        if (!/^[a-z0-9-]+$/.test(value)) {
          return 'Slug must contain only lowercase letters, numbers, and hyphens'
        }
        return true
      }
    },
    {
      name: 'email',
      type: 'email',
      validate: (value) => {
        // Additional email validation
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        if (!emailRegex.test(value)) {
          return 'Invalid email format'
        }

        // Disallow disposable email domains
        const disposableDomains = ['tempmail.com', 'throwaway.email']
        const domain = value.split('@')[1]
        if (disposableDomains.includes(domain)) {
          return 'Disposable email addresses not allowed'
        }

        return true
      }
    }
  ]
}
```

### Sanitize Rich Text

**Payload automatically sanitizes rich text fields, but you can add custom rules:**

```typescript
import { sanitizeHtml } from 'payload'

{
  name: 'content',
  type: 'richText',
  editor: lexicalEditor({
    features: ({ defaultFeatures }) => [
      ...defaultFeatures,
      // Explicitly allow only safe tags
      {
        name: 'sanitize',
        sanitizeHtml: {
          allowedTags: ['p', 'strong', 'em', 'ul', 'ol', 'li', 'a'],
          allowedAttributes: {
            'a': ['href', 'title']
          },
          allowedSchemes: ['http', 'https', 'mailto']
        }
      }
    ]
  })
}
```

### File Upload Security

**Validate file types and sizes:**

```typescript
{
  slug: 'media',
  upload: {
    staticDir: 'media',
    adminThumbnail: 'thumbnail',
    mimeTypes: ['image/png', 'image/jpeg', 'image/gif'], // Whitelist only
    imageSizes: [
      {
        name: 'thumbnail',
        width: 400,
        height: 300,
        position: 'center'
      }
    ],
    limits: {
      fileSize: 5000000 // 5MB max
    }
  },
  hooks: {
    beforeChange: [
      ({ data, req }) => {
        // Additional file validation
        const dangerousExtensions = ['.exe', '.bat', '.sh', '.php']
        const ext = data.filename.toLowerCase().substring(data.filename.lastIndexOf('.'))

        if (dangerousExtensions.includes(ext)) {
          throw new Error('File type not allowed')
        }

        // Rename file to prevent path traversal
        data.filename = data.filename.replace(/[^a-z0-9.-]/gi, '_')

        return data
      }
    ]
  }
}
```

**Scan uploaded files (optional):**

```typescript
import ClamAV from 'clamav.js'

{
  hooks: {
    beforeChange: [
      async ({ data, req }) => {
        // Scan for viruses
        const isInfected = await ClamAV.scanFile(data.tempFilePath)
        if (isInfected) {
          throw new Error('File contains malware')
        }
        return data
      },
    ]
  }
}
```

## SQL Injection & NoSQL Injection

### Payload Protects You

**Payload uses parameterized queries automatically:**

```typescript
// ✅ SAFE: Payload sanitizes input
const posts = await payload.find({
  collection: 'posts',
  where: {
    title: { equals: userInput }, // Payload handles escaping
  },
})
```

**⚠️ DANGEROUS: Direct database queries**

```typescript
// ❌ NEVER DO THIS
const posts = await db.collection('posts').find({
  title: userInput, // Vulnerable to injection!
})

// ✅ DO THIS INSTEAD
const posts = await payload.find({
  collection: 'posts',
  where: { title: { equals: userInput } },
})
```

### MongoDB Injection Prevention

**Be cautious with `$where` operator:**

```typescript
// ❌ DANGEROUS
const result = await db.collection('posts').find({
  $where: `this.title === '${userInput}'`, // CODE INJECTION!
})

// ✅ SAFE: Use Payload's query language
const result = await payload.find({
  collection: 'posts',
  where: {
    title: { equals: userInput },
  },
})
```

## Cross-Site Scripting (XSS)

### Prevent XSS in React

**React escapes by default:**

```typescript
// ✅ SAFE: React escapes automatically
function PostTitle({ title }) {
  return <h1>{title}</h1> // <script> tags rendered as text
}
```

**⚠️ DANGEROUS: dangerouslySetInnerHTML**

```typescript
// ❌ NEVER DO THIS with user input
function PostContent({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />
}

// ✅ DO THIS: Use rich text field with sanitization
function PostContent({ richText }) {
  return <RichText content={richText} /> // Payload sanitizes
}
```

### Content Security Policy (CSP)

**Add CSP headers:**

```typescript
// next.config.js
const nextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: [
              "default-src 'self'",
              "script-src 'self' 'unsafe-inline' 'unsafe-eval'", // Needed for Next.js
              "style-src 'self' 'unsafe-inline'",
              "img-src 'self' data: https:",
              "font-src 'self'",
              "connect-src 'self' https://api.payloadcms.com",
              "frame-ancestors 'none'",
            ].join('; '),
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin',
          },
        ],
      },
    ]
  },
}
```

## Cross-Site Request Forgery (CSRF)

**Enable CSRF protection:**

```typescript
// payload.config.ts
export default buildConfig({
  csrf: [
    'https://yoursite.com',
    'https://www.yoursite.com'
  ],
  // Payload automatically validates Origin header
  collections: [...]
})
```

**In production:**

```bash
# .env
CSRF_DOMAINS=https://yoursite.com,https://www.yoursite.com
```

## Rate Limiting

**Prevent brute force and DoS attacks:**

```typescript
import rateLimit from 'express-rate-limit'

// In Next.js middleware or API route
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: 'Too many login attempts. Please try again later.',
  standardHeaders: true,
  legacyHeaders: false,
})

export async function POST(request: Request) {
  // Apply rate limiting
  await loginLimiter(request)

  // Proceed with login
  const payload = await getPayload({ config })
  // ...
}
```

**Collection-level rate limiting:**

```typescript
{
  slug: 'users',
  auth: {
    maxLoginAttempts: 5,
    lockTime: 600000 // 10 minutes
  }
}
```

## API Security

### Secure API Routes

**Validate authentication:**

```typescript
// app/api/admin/route.ts
import { getPayload } from 'payload'
import { headers } from 'next/headers'

export async function GET() {
  const payload = await getPayload({ config })
  const headersList = headers()

  // Check authentication
  const user = await payload.auth.check({ headers: headersList })

  if (!user || user.role !== 'admin') {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Proceed with admin action
  const data = await getAdminData()
  return Response.json(data)
}
```

### Prevent Mass Assignment

**Explicitly define allowed fields:**

```typescript
export async function POST(request: Request) {
  const payload = await getPayload({ config })
  const body = await request.json()

  // ❌ DANGEROUS: User can set any field
  const user = await payload.create({
    collection: 'users',
    data: body, // Could include { role: 'admin' }!
  })

  // ✅ SAFE: Whitelist allowed fields
  const allowedFields = ['name', 'email', 'password']
  const data = Object.keys(body)
    .filter((key) => allowedFields.includes(key))
    .reduce((obj, key) => {
      obj[key] = body[key]
      return obj
    }, {})

  const user = await payload.create({
    collection: 'users',
    data,
  })

  return Response.json(user)
}
```

### CORS Configuration

**Configure CORS securely:**

```typescript
// next.config.js
const nextConfig = {
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          {
            key: 'Access-Control-Allow-Origin',
            value: process.env.ALLOWED_ORIGIN || 'https://yoursite.com',
          },
          {
            key: 'Access-Control-Allow-Methods',
            value: 'GET,POST,PUT,DELETE,OPTIONS',
          },
          {
            key: 'Access-Control-Allow-Headers',
            value: 'Content-Type, Authorization',
          },
          {
            key: 'Access-Control-Allow-Credentials',
            value: 'true',
          },
        ],
      },
    ]
  },
}

// ⚠️ NEVER use '*' in production
// value: '*' // VULNERABLE!
```

## Database Security

### Connection Security

**Use encrypted connections:**

```typescript
// MongoDB with TLS
import { mongooseAdapter } from '@payloadcms/db-mongodb'

export default buildConfig({
  db: mongooseAdapter({
    url: process.env.DATABASE_URI,
    connectOptions: {
      ssl: true,
      sslValidate: true,
      sslCA: process.env.SSL_CERT_PATH,
    },
  }),
})

// PostgreSQL with SSL
import { postgresAdapter } from '@payloadcms/db-postgres'

export default buildConfig({
  db: postgresAdapter({
    pool: {
      connectionString: process.env.DATABASE_URI,
      ssl: {
        require: true,
        rejectUnauthorized: true,
        ca: process.env.SSL_CERT,
      },
    },
  }),
})
```

### Principle of Least Privilege

**Database user should have minimal permissions:**

```sql
-- MongoDB: Create user with only necessary permissions
use payload
db.createUser({
  user: "payloadapp",
  pwd: "strong-password",
  roles: [
    { role: "readWrite", db: "payload" }
  ]
})

-- PostgreSQL: Grant minimal permissions
CREATE USER payloadapp WITH PASSWORD 'strong-password';
GRANT CONNECT ON DATABASE payload TO payloadapp;
GRANT USAGE ON SCHEMA public TO payloadapp;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO payloadapp;
```

### Backup & Encryption

**Encrypt sensitive fields:**

```typescript
import crypto from 'crypto'

{
  slug: 'users',
  fields: [
    {
      name: 'ssn',
      type: 'text',
      hooks: {
        // Encrypt before saving
        beforeChange: [
          ({ value }) => {
            if (!value) return value
            const cipher = crypto.createCipher('aes-256-cbc', process.env.ENCRYPTION_KEY)
            let encrypted = cipher.update(value, 'utf8', 'hex')
            encrypted += cipher.final('hex')
            return encrypted
          }
        ],
        // Decrypt when reading
        afterRead: [
          ({ value }) => {
            if (!value) return value
            const decipher = crypto.createDecipher('aes-256-cbc', process.env.ENCRYPTION_KEY)
            let decrypted = decipher.update(value, 'hex', 'utf8')
            decrypted += decipher.final('utf8')
            return decrypted
          }
        ]
      }
    }
  ]
}
```

## Environment Variables

### Secure Configuration

**.env.local (never commit!):**

```bash
# Required
PAYLOAD_SECRET=<32+ character random string>
DATABASE_URI=mongodb://localhost:27017/payload

# Optional but recommended
NODE_ENV=production
CSRF_DOMAINS=https://yoursite.com

# Third-party API keys (NEVER commit!)
S3_ACCESS_KEY=...
S3_SECRET_KEY=...
STRIPE_SECRET_KEY=...
```

**.env.example (commit this):**

```bash
# Copy to .env.local and fill in real values
PAYLOAD_SECRET=
DATABASE_URI=
S3_ACCESS_KEY=
S3_SECRET_KEY=
```

**.gitignore:**

```
.env
.env.local
.env*.local
```

### Production Checklist

**Before deploying:**

1. ✅ `PAYLOAD_SECRET` is strong random string (32+ chars)
2. ✅ `NODE_ENV=production`
3. ✅ Database uses TLS/SSL
4. ✅ CORS configured for your domain only
5. ✅ CSRF protection enabled
6. ✅ Rate limiting enabled
7. ✅ CSP headers configured
8. ✅ File upload limits set
9. ✅ Admin panel behind authentication
10. ✅ No console.logs with sensitive data
11. ✅ Dependencies up to date (`pnpm audit`)
12. ✅ Environment variables in secure storage (not in code)

## Monitoring & Logging

### Security Logging

**Log security events:**

```typescript
{
  slug: 'users',
  hooks: {
    beforeLogin: [
      ({ req }) => {
        console.log(`[AUTH] Login attempt: ${req.data.email} from ${req.ip}`)
      }
    ],
    afterLogin: [
      ({ user, req }) => {
        console.log(`[AUTH] Successful login: ${user.email} from ${req.ip}`)
      }
    ],
    beforeOperation: [
      ({ operation, req }) => {
        if (['delete', 'update'].includes(operation)) {
          console.log(`[AUDIT] ${operation} by ${req.user?.email || 'anonymous'}`)
        }
      }
    ]
  }
}
```

### Error Handling

**Don't leak sensitive information:**

```typescript
// ❌ BAD: Exposes internal details
try {
  await payload.create({ ... })
} catch (error) {
  return Response.json(
    { error: error.message }, // Might expose database structure!
    { status: 500 }
  )
}

// ✅ GOOD: Generic error message
try {
  await payload.create({ ... })
} catch (error) {
  console.error('[ERROR]', error) // Log internally
  return Response.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

## Third-Party Security

### Dependency Audits

**Regularly check for vulnerabilities:**

```bash
# Check for known vulnerabilities
pnpm audit

# Fix automatically (when possible)
pnpm audit fix

# Update dependencies
pnpm update
```

### Secure Package Installation

```bash
# Always verify package before installing
pnpm info <package-name>

# Check package reputation
# - npm downloads
# - GitHub stars
# - Recent updates
# - Known vulnerabilities
```

## Security Headers Summary

**Complete security headers configuration:**

```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on',
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload',
  },
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN',
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff',
  },
  {
    key: 'X-XSS-Protection',
    value: '1; mode=block',
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin',
  },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()',
  },
]

const nextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
    ]
  },
}
```

## Incident Response

**If a security breach occurs:**

1. **Isolate:** Take affected systems offline
2. **Investigate:** Review logs, identify entry point
3. **Contain:** Change all credentials, rotate secrets
4. **Notify:** Inform affected users if personal data exposed
5. **Remediate:** Fix vulnerability
6. **Document:** Record incident details and response
7. **Learn:** Update security practices

**Emergency contacts:**

- Database admin
- Hosting provider support
- Security team
- Legal team (for data breaches)

---

_Security is not a feature, it's a requirement!_
