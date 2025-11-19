# Patterns and Conventions

**Documented:** November 18, 2025

Code patterns and conventions used throughout Payload CMS.

## Code Style

**TypeScript:**
- Strict mode enabled
- Explicit return types preferred
- No `any` (use `unknown` if needed)

**Naming:**
- `camelCase` for variables and functions
- `PascalCase` for types and components
- `kebab-case` for file names

## Collection Patterns

**Slug naming:**
```typescript
{
  slug: 'posts',  // Plural, lowercase
  slug: 'media',  // No -items or -collection suffix
}
```

**Field naming:**
```typescript
{
  name: 'createdAt',  // camelCase
  name: 'isPublished',  // Boolean prefix: is, has, should
}
```

## Hook Patterns

✅ **CURRENT:** Return data from hooks

```typescript
beforeChange: [
  ({ data }) => {
    data.slug = generateSlug(data.title)
    return data  // Always return
  }
]
```

## Access Control Patterns

```typescript
access: {
  read: true,  // Simple boolean
  create: ({ req }) => !!req.user,  // Function for logic
  update: isAdminOrOwner,  // Reusable function
}
```

## Server Component Patterns

✅ **CURRENT (Nov 2025):**
```typescript
export async function Component() {
  const data = await payload.find({ collection: 'posts' })
  return <div>{data.docs.map(...)}</div>
}
```

⚠️ **OUTDATED:**
```typescript
'use client'
export function Component() {
  const [data, setData] = useState([])
  useEffect(() => { fetch('/api/posts').then(setData) }, [])
  return <div>{data.map(...)}</div>
}
```

## Error Handling

```typescript
try {
  await payload.create({ collection, data })
} catch (error) {
  if (error.name === 'ValidationError') {
    // Handle validation errors
  } else if (error.name === 'Forbidden') {
    // Handle permission errors
  }
  throw error
}
```

## Testing Patterns

**Integration tests:**
```typescript
describe('Posts Collection', () => {
  let payload: Payload

  beforeAll(async () => {
    payload = await getPayload({ config })
  })

  it('should create a post', async () => {
    const post = await payload.create({
      collection: 'posts',
      data: { title: 'Test' }
    })
    expect(post.title).toBe('Test')
  })
})
```

## Common Pitfalls

❌ **Don't:**
- Mutate hook data without returning
- Use `any` type
- Access `req.user` without checking if exists
- Forget to handle pagination in queries

✅ **Do:**
- Return data from hooks
- Use TypeScript types
- Check `req.user` existence
- Use `limit` and `page` parameters

---

*Follow patterns, write clean code!*
