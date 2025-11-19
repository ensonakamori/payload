# API Documentation

**Documented:** November 18, 2025

Complete reference for Payload's REST and GraphQL APIs.

## Mental Model: API Layers

**For React Developers:**
In React apps, you typically call external APIs with fetch(). In Payload, you work at multiple levels:

```
Local API (Node.js)
    ↓
REST API (HTTP)
    ↓
GraphQL API (HTTP)
```

**When to use each:**

- **Local API**: Server Components, Server Actions, API routes
- **REST API**: External clients, mobile apps, webhooks
- **GraphQL API**: Complex queries, specific field selection

## REST API

### Base URL

**Development:**

```
http://localhost:3000/api
```

**Production:**

```
https://yoursite.com/api
```

### Authentication

**Login:**

```http
POST /api/users/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**

```json
{
  "message": "Auth Passed",
  "user": {
    "id": "65f...",
    "email": "user@example.com",
    "role": "admin",
    "createdAt": "2025-01-15T10:30:00.000Z",
    "updatedAt": "2025-01-15T10:30:00.000Z"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "exp": 1705317000
}
```

**Cookie set:**

```
Set-Cookie: payload-token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...; HttpOnly; Secure; SameSite=Lax
```

**Using token in subsequent requests:**

```http
GET /api/posts
Cookie: payload-token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Or with Authorization header
GET /api/posts
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Logout:**

```http
POST /api/users/logout
```

**Check authentication:**

```http
GET /api/users/me
```

**Response:**

```json
{
  "user": {
    "id": "65f...",
    "email": "user@example.com",
    "role": "admin"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "exp": 1705317000
}
```

### Collections API

#### Find (List)

**Request:**

```http
GET /api/posts
GET /api/posts?limit=10&page=1
GET /api/posts?where[status][equals]=published
GET /api/posts?sort=-createdAt
GET /api/posts?depth=1
```

**Query Parameters:**

- `limit` - Number of docs to return (default: 10)
- `page` - Page number (default: 1)
- `where` - Filter conditions (see below)
- `sort` - Sort field (prefix `-` for descending)
- `depth` - Populate relationship depth (0-10)
- `locale` - Locale for localized fields
- `fallbackLocale` - Fallback locale

**Response:**

```json
{
  "docs": [
    {
      "id": "65f...",
      "title": "My Post",
      "status": "published",
      "author": {
        "id": "65e...",
        "name": "John Doe"
      },
      "createdAt": "2025-01-15T10:30:00.000Z",
      "updatedAt": "2025-01-15T10:30:00.000Z"
    }
  ],
  "totalDocs": 25,
  "limit": 10,
  "totalPages": 3,
  "page": 1,
  "pagingCounter": 1,
  "hasPrevPage": false,
  "hasNextPage": true,
  "prevPage": null,
  "nextPage": 2
}
```

#### Find by ID

**Request:**

```http
GET /api/posts/65f1234567890abcdef12345
GET /api/posts/65f1234567890abcdef12345?depth=2
```

**Response:**

```json
{
  "id": "65f...",
  "title": "My Post",
  "content": "Post content here...",
  "status": "published",
  "author": {
    "id": "65e...",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "createdAt": "2025-01-15T10:30:00.000Z",
  "updatedAt": "2025-01-15T10:30:00.000Z"
}
```

#### Create

**Request:**

```http
POST /api/posts
Content-Type: application/json

{
  "title": "New Post",
  "content": "This is my new post",
  "status": "draft",
  "author": "65e1234567890abcdef12345"
}
```

**Response:**

```json
{
  "message": "Post created successfully",
  "doc": {
    "id": "65f...",
    "title": "New Post",
    "content": "This is my new post",
    "status": "draft",
    "author": "65e...",
    "createdAt": "2025-01-15T10:30:00.000Z",
    "updatedAt": "2025-01-15T10:30:00.000Z"
  }
}
```

#### Update

**Request:**

```http
PATCH /api/posts/65f1234567890abcdef12345
Content-Type: application/json

{
  "title": "Updated Title",
  "status": "published"
}
```

**Response:**

```json
{
  "message": "Post updated successfully",
  "doc": {
    "id": "65f...",
    "title": "Updated Title",
    "status": "published",
    "updatedAt": "2025-01-15T11:00:00.000Z"
  }
}
```

#### Delete

**Request:**

```http
DELETE /api/posts/65f1234567890abcdef12345
```

**Response:**

```json
{
  "id": "65f...",
  "message": "Post deleted successfully"
}
```

### Where Queries

**Equals:**

```http
GET /api/posts?where[status][equals]=published
```

**Not equals:**

```http
GET /api/posts?where[status][not_equals]=draft
```

**Greater than:**

```http
GET /api/posts?where[views][greater_than]=100
```

**Less than:**

```http
GET /api/posts?where[views][less_than]=1000
```

**Contains:**

```http
GET /api/posts?where[title][contains]=React
```

**Like (case-insensitive):**

```http
GET /api/posts?where[title][like]=react
```

**In:**

```http
GET /api/posts?where[status][in][0]=draft&where[status][in][1]=published
```

**Not in:**

```http
GET /api/posts?where[status][not_in][0]=archived
```

**Exists:**

```http
GET /api/posts?where[featuredImage][exists]=true
```

**Near (geospatial):**

```http
GET /api/locations?where[point][near][0]=40.7128&where[point][near][1]=-74.0060&where[point][near][2]=5000
```

**AND conditions:**

```http
GET /api/posts?where[and][0][status][equals]=published&where[and][1][views][greater_than]=100
```

**OR conditions:**

```http
GET /api/posts?where[or][0][status][equals]=published&where[or][1][status][equals]=archived
```

**Complex nested:**

```http
GET /api/posts?where[and][0][or][0][status][equals]=published&where[and][0][or][1][status][equals]=archived&where[and][1][views][greater_than]=100
```

### File Uploads

**Upload image:**

```http
POST /api/media
Content-Type: multipart/form-data

{
  "file": <binary data>,
  "alt": "Image description",
  "caption": "Image caption"
}
```

**Response:**

```json
{
  "message": "Media created successfully",
  "doc": {
    "id": "65f...",
    "alt": "Image description",
    "filename": "image-1705317000.jpg",
    "mimeType": "image/jpeg",
    "filesize": 245678,
    "width": 1920,
    "height": 1080,
    "url": "/media/image-1705317000.jpg",
    "sizes": {
      "thumbnail": {
        "url": "/media/image-1705317000-thumbnail.jpg",
        "width": 400,
        "height": 300,
        "mimeType": "image/jpeg",
        "filesize": 45678,
        "filename": "image-1705317000-thumbnail.jpg"
      }
    }
  }
}
```

### Globals API

**Get global:**

```http
GET /api/globals/header
```

**Response:**

```json
{
  "id": "65f...",
  "logo": {
    "id": "65e...",
    "url": "/media/logo.png"
  },
  "navigation": [
    {
      "label": "Home",
      "url": "/"
    },
    {
      "label": "About",
      "url": "/about"
    }
  ],
  "createdAt": "2025-01-15T10:30:00.000Z",
  "updatedAt": "2025-01-15T10:30:00.000Z"
}
```

**Update global:**

```http
POST /api/globals/header
Content-Type: application/json

{
  "logo": "65e1234567890abcdef12345",
  "navigation": [
    {
      "label": "Home",
      "url": "/"
    }
  ]
}
```

### Preferences API

**Get preferences:**

```http
GET /api/_preferences/collection-posts
```

**Response:**

```json
{
  "key": "collection-posts",
  "value": {
    "columns": ["title", "status", "createdAt"],
    "limit": 25
  },
  "user": "65f..."
}
```

**Update preferences:**

```http
POST /api/_preferences/collection-posts
Content-Type: application/json

{
  "value": {
    "columns": ["title", "author", "status"],
    "limit": 50
  }
}
```

### Error Responses

**400 Bad Request:**

```json
{
  "errors": [
    {
      "message": "The following field is invalid: title",
      "field": "title"
    }
  ]
}
```

**401 Unauthorized:**

```json
{
  "errors": [
    {
      "message": "You must be logged in to access this resource"
    }
  ]
}
```

**403 Forbidden:**

```json
{
  "errors": [
    {
      "message": "You are not allowed to perform this action"
    }
  ]
}
```

**404 Not Found:**

```json
{
  "errors": [
    {
      "message": "The requested resource was not found"
    }
  ]
}
```

**500 Internal Server Error:**

```json
{
  "errors": [
    {
      "message": "An error occurred while processing your request"
    }
  ]
}
```

## GraphQL API

### GraphQL Endpoint

```
http://localhost:3000/api/graphql
```

**GraphQL Playground (development only):**

```
http://localhost:3000/api/graphql-playground
```

### Authentication

**HTTP header:**

```http
POST /api/graphql
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Or use cookie from REST login:**

```http
POST /api/graphql
Cookie: payload-token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Queries

**Find multiple:**

```graphql
query {
  Posts(
    limit: 10
    page: 1
    where: { status: { equals: published } }
    sort: "-createdAt"
  ) {
    docs {
      id
      title
      status
      author {
        id
        name
        email
      }
      createdAt
    }
    totalDocs
    limit
    totalPages
    page
    hasNextPage
    hasPrevPage
  }
}
```

**Response:**

```json
{
  "data": {
    "Posts": {
      "docs": [
        {
          "id": "65f...",
          "title": "My Post",
          "status": "published",
          "author": {
            "id": "65e...",
            "name": "John Doe",
            "email": "john@example.com"
          },
          "createdAt": "2025-01-15T10:30:00.000Z"
        }
      ],
      "totalDocs": 25,
      "limit": 10,
      "totalPages": 3,
      "page": 1,
      "hasNextPage": true,
      "hasPrevPage": false
    }
  }
}
```

**Find by ID:**

```graphql
query {
  Post(id: "65f1234567890abcdef12345") {
    id
    title
    content
    status
    author {
      id
      name
    }
    createdAt
    updatedAt
  }
}
```

**With variables:**

```graphql
query GetPost($id: String!) {
  Post(id: $id) {
    id
    title
    content
  }
}
```

**Variables:**

```json
{
  "id": "65f1234567890abcdef12345"
}
```

**Fragment usage:**

```graphql
fragment PostFields on Post {
  id
  title
  status
  createdAt
}

query {
  Posts(limit: 10) {
    docs {
      ...PostFields
      author {
        id
        name
      }
    }
  }
}
```

**Nested relationships:**

```graphql
query {
  Posts(limit: 5) {
    docs {
      id
      title
      author {
        id
        name
        posts {
          docs {
            id
            title
          }
        }
      }
      categories {
        id
        name
        posts {
          docs {
            id
            title
          }
        }
      }
    }
  }
}
```

### Mutations

**Create:**

```graphql
mutation {
  createPost(
    data: {
      title: "New Post"
      content: "Post content"
      status: draft
      author: "65e1234567890abcdef12345"
    }
  ) {
    id
    title
    status
    createdAt
  }
}
```

**With variables:**

```graphql
mutation CreatePost($data: mutationPostInput!) {
  createPost(data: $data) {
    id
    title
    status
  }
}
```

**Variables:**

```json
{
  "data": {
    "title": "New Post",
    "content": "Post content",
    "status": "draft",
    "author": "65e1234567890abcdef12345"
  }
}
```

**Update:**

```graphql
mutation {
  updatePost(
    id: "65f1234567890abcdef12345"
    data: { title: "Updated Title", status: published }
  ) {
    id
    title
    status
    updatedAt
  }
}
```

**Delete:**

```graphql
mutation {
  deletePost(id: "65f1234567890abcdef12345") {
    id
  }
}
```

**Login (authentication):**

```graphql
mutation {
  loginUser(email: "user@example.com", password: "password123") {
    user {
      id
      email
      role
    }
    token
    exp
  }
}
```

**Logout:**

```graphql
mutation {
  logoutUser
}
```

### GraphQL Where Queries

**Equals:**

```graphql
query {
  Posts(where: { status: { equals: published } }) {
    docs {
      id
      title
    }
  }
}
```

**Not equals:**

```graphql
query {
  Posts(where: { status: { not_equals: draft } }) {
    docs {
      id
      title
    }
  }
}
```

**Greater than:**

```graphql
query {
  Posts(where: { views: { greater_than: 100 } }) {
    docs {
      id
      title
      views
    }
  }
}
```

**Contains:**

```graphql
query {
  Posts(where: { title: { contains: "React" } }) {
    docs {
      id
      title
    }
  }
}
```

**In:**

```graphql
query {
  Posts(where: { status: { in: [draft, published] } }) {
    docs {
      id
      title
      status
    }
  }
}
```

**AND:**

```graphql
query {
  Posts(
    where: {
      and: [{ status: { equals: published } }, { views: { greater_than: 100 } }]
    }
  ) {
    docs {
      id
      title
    }
  }
}
```

**OR:**

```graphql
query {
  Posts(
    where: {
      or: [{ status: { equals: published } }, { status: { equals: archived } }]
    }
  ) {
    docs {
      id
      title
      status
    }
  }
}
```

### Introspection

**Get schema:**

```graphql
query {
  __schema {
    types {
      name
      kind
      description
    }
  }
}
```

**Get type details:**

```graphql
query {
  __type(name: "Post") {
    name
    kind
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

## Local API (Node.js)

**Use in Server Components, Server Actions, and API routes.**

### Get Payload Instance

```typescript
import { getPayload } from 'payload'
import config from '@payload-config'

const payload = await getPayload({ config })
```

### Find

```typescript
const posts = await payload.find({
  collection: 'posts',
  where: {
    status: { equals: 'published' },
  },
  limit: 10,
  page: 1,
  sort: '-createdAt',
  depth: 1,
})

console.log(posts.docs)
console.log(posts.totalDocs)
```

### Find by ID

```typescript
const post = await payload.findByID({
  collection: 'posts',
  id: '65f1234567890abcdef12345',
  depth: 2,
})
```

### Create

```typescript
const post = await payload.create({
  collection: 'posts',
  data: {
    title: 'New Post',
    content: 'Post content',
    status: 'draft',
    author: req.user.id,
  },
  user: req.user, // For access control
  locale: 'en', // Optional
})
```

### Update

```typescript
const post = await payload.update({
  collection: 'posts',
  id: '65f1234567890abcdef12345',
  data: {
    title: 'Updated Title',
    status: 'published',
  },
  user: req.user,
})
```

### Delete

```typescript
const post = await payload.delete({
  collection: 'posts',
  id: '65f1234567890abcdef12345',
  user: req.user,
})
```

### Count

```typescript
const count = await payload.count({
  collection: 'posts',
  where: {
    status: { equals: 'published' },
  },
})
```

### Find Global

```typescript
const header = await payload.findGlobal({
  slug: 'header',
  depth: 1,
})
```

### Update Global

```typescript
const header = await payload.updateGlobal({
  slug: 'header',
  data: {
    logo: '65e1234567890abcdef12345',
    navigation: [{ label: 'Home', url: '/' }],
  },
})
```

### Authentication

**Login:**

```typescript
const result = await payload.login({
  collection: 'users',
  data: {
    email: 'user@example.com',
    password: 'password123',
  },
})

console.log(result.user)
console.log(result.token)
```

**Logout:**

```typescript
await payload.logout({
  collection: 'users',
  req,
})
```

**Forgot password:**

```typescript
await payload.forgotPassword({
  collection: 'users',
  data: {
    email: 'user@example.com',
  },
})
```

**Reset password:**

```typescript
await payload.resetPassword({
  collection: 'users',
  data: {
    token: 'reset-token',
    password: 'newPassword123',
  },
})
```

### Advanced Options

**Override access control:**

```typescript
const post = await payload.create({
  collection: 'posts',
  data: { ... },
  overrideAccess: true // Skip access control (use carefully!)
})
```

**Disable hooks:**

```typescript
const post = await payload.update({
  collection: 'posts',
  id: '65f...',
  data: { ... },
  hooks: false // Skip all hooks
})
```

**Autosave (drafts):**

```typescript
const post = await payload.update({
  collection: 'posts',
  id: '65f...',
  data: { ... },
  autosave: true // Save as draft version
})
```

**Specify locale:**

```typescript
const post = await payload.findByID({
  collection: 'posts',
  id: '65f...',
  locale: 'es', // Spanish
  fallbackLocale: 'en', // Fallback to English
})
```

## Client-Side Usage

### REST with fetch

```typescript
// In Client Component
'use client'
export function PostsList() {
  const [posts, setPosts] = useState([])

  useEffect(() => {
    async function fetchPosts() {
      const res = await fetch('/api/posts?limit=10&where[status][equals]=published')
      const data = await res.json()
      setPosts(data.docs)
    }
    fetchPosts()
  }, [])

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### GraphQL with Apollo Client

```typescript
import { ApolloClient, InMemoryCache, gql, useQuery } from '@apollo/client'

const client = new ApolloClient({
  uri: '/api/graphql',
  cache: new InMemoryCache()
})

const GET_POSTS = gql`
  query GetPosts {
    Posts(limit: 10, where: { status: { equals: published } }) {
      docs {
        id
        title
        status
      }
    }
  }
`

export function PostsList() {
  const { loading, error, data } = useQuery(GET_POSTS)

  if (loading) return <p>Loading...</p>
  if (error) return <p>Error: {error.message}</p>

  return (
    <ul>
      {data.Posts.docs.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

## Rate Limits

**Default limits (configurable):**

- 100 requests per minute per IP
- 1000 requests per hour per IP

**Custom rate limiting:**
See [SECURITY_GUIDE.md](./SECURITY_GUIDE.md#rate-limiting)

## Versioning

**Get document versions:**

```http
GET /api/posts/65f.../versions
```

**Response:**

```json
{
  "docs": [
    {
      "id": "65g...",
      "parent": "65f...",
      "version": {
        "title": "Original Title",
        "status": "draft"
      },
      "createdAt": "2025-01-15T09:00:00.000Z"
    }
  ]
}
```

**Restore version:**

```http
POST /api/posts/65f.../versions/65g...
```

## Best Practices

### DO:

✅ Use Local API in Server Components (faster, no HTTP overhead)
✅ Use pagination for large datasets
✅ Specify `depth` to avoid over-fetching relationships
✅ Use GraphQL for complex, specific queries
✅ Cache responses when appropriate
✅ Handle errors gracefully
✅ Validate input data

### DON'T:

❌ Make API calls in Client Components when you can use Server Components
❌ Fetch without pagination (may timeout on large collections)
❌ Set `depth` too high (performance impact)
❌ Expose sensitive data in API responses
❌ Trust client-side data (always validate server-side)
❌ Make N+1 queries (use `depth` or GraphQL)

---

_APIs are the gateway to your data!_
