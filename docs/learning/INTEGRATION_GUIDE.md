# Integration Guide

**Documented:** November 18, 2025

How Payload integrates with external services and storage providers.

## Storage Integration

### AWS S3

```typescript
import { s3Storage } from '@payloadcms/storage-s3'

export default buildConfig({
  plugins: [
    s3Storage({
      collections: {
        'media': true,  // Enable for media collection
      },
      bucket: process.env.S3_BUCKET,
      config: {
        credentials: {
          accessKeyId: process.env.S3_ACCESS_KEY,
          secretAccessKey: process.env.S3_SECRET_KEY,
        },
        region: process.env.S3_REGION,
      },
    }),
  ],
})
```

### Vercel Blob

```typescript
import { vercelBlobStorage } from '@payloadcms/storage-vercel-blob'

export default buildConfig({
  plugins: [
    vercelBlobStorage({
      collections: {
        'media': true,
      },
      token: process.env.BLOB_READ_WRITE_TOKEN,
    }),
  ],
})
```

## Email Integration

### Nodemailer

```typescript
import { nodemailerAdapter } from '@payloadcms/email-nodemailer'

export default buildConfig({
  email: nodemailerAdapter({
    defaultFromAddress: 'noreply@example.com',
    defaultFromName: 'My App',
    transportOptions: {
      host: process.env.SMTP_HOST,
      port: 587,
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS,
      },
    },
  }),
})
```

### Resend

```typescript
import { resendAdapter } from '@payloadcms/email-resend'

export default buildConfig({
  email: resendAdapter({
    defaultFromAddress: 'noreply@example.com',
    defaultFromName: 'My App',
    apiKey: process.env.RESEND_API_KEY,
  }),
})
```

## Payment Integration (Stripe)

```typescript
import { stripePlugin } from '@payloadcms/plugin-stripe'

export default buildConfig({
  plugins: [
    stripePlugin({
      stripeSecretKey: process.env.STRIPE_SECRET_KEY,
      stripeWebhooksEndpointSecret: process.env.STRIPE_WEBHOOKS_SECRET,
      sync: [
        {
          collection: 'products',
          stripeResourceType: 'products',
          fields: [
            { fieldPath: 'name', stripeProperty: 'name' },
            { fieldPath: 'price', stripeProperty: 'default_price' },
          ],
        },
      ],
    }),
  ],
})
```

## Search Integration

### Plugin Search (Built-in)

```typescript
import { searchPlugin } from '@payloadcms/plugin-search'

export default buildConfig({
  plugins: [
    searchPlugin({
      collections: ['posts', 'pages'],
      searchOverrides: {
        fields: [
          {
            name: 'excerpt',
            type: 'textarea',
          },
        ],
      },
    }),
  ],
})
```

### External Search (Algolia, Elasticsearch)

Via hooks:

```typescript
{
  slug: 'posts',
  hooks: {
    afterChange: [
      async ({ doc }) => {
        // Index in external search
        await algolia.saveObject({
          objectID: doc.id,
          title: doc.title,
          content: doc.content,
        })
      },
    ],
  },
}
```

## Monitoring (Sentry)

```typescript
import { sentryPlugin } from '@payloadcms/plugin-sentry'

export default buildConfig({
  plugins: [
    sentryPlugin({
      dsn: process.env.SENTRY_DSN,
      environment: process.env.NODE_ENV,
    }),
  ],
})
```

## Authentication Providers

### OAuth (Custom)

```typescript
{
  slug: 'users',
  auth: {
    strategies: [
      {
        name: 'google',
        strategy: async ({ payload, req }) => {
          // Implement OAuth flow
          const user = await verifyGoogleToken(req.headers.authorization)
          return { user }
        },
      },
    ],
  },
}
```

## Webhooks

**Outgoing webhooks:**

```typescript
{
  slug: 'posts',
  hooks: {
    afterChange: [
      async ({ doc, operation }) => {
        // Trigger webhook
        await fetch('https://example.com/webhook', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ event: operation, data: doc }),
        })
      },
    ],
  },
}
```

**Incoming webhooks:**
Create custom endpoint in Next.js.

## Next Steps

- [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)
- [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)

---

*Integrate everything!*
