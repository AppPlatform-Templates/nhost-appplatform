# How to Use Nhost

Nhost is an open-source Backend-as-a-Service (BaaS) platform that provides everything you need to build modern applications: **database**, **authentication**, **storage**, and **serverless functions** - all with a **GraphQL API**.

This guide will help you get started building applications with your deployed Nhost instance.

## What You Get

Your Nhost deployment includes:

1. **PostgreSQL Database** - Managed database for your data
2. **GraphQL API** (Hasura) - Instant GraphQL API for your database
3. **Authentication** - User management and JWT-based auth
4. **Storage** - File upload/download with access control
5. **Functions** - Serverless functions for custom backend logic
6. **Dashboard** - Web interface for managing your backend

## Accessing Your Nhost Services

After deployment, your services are available at:

- **Dashboard**: `https://your-app-xyz.ondigitalocean.app/` (main entry point)
- **GraphQL API**: `https://your-app-xyz.ondigitalocean.app/graphql/v1/graphql`
- **Auth API**: `https://your-app-xyz.ondigitalocean.app/auth/v1`
- **Storage API**: `https://your-app-xyz.ondigitalocean.app/storage/v1`
- **Functions**: `https://your-app-xyz.ondigitalocean.app/functions/v1`

Replace `your-app-xyz.ondigitalocean.app` with your actual App Platform URL.

## Getting Started

### 1. Access the Dashboard

1. Open your app URL in a browser
2. You'll see the Nhost Dashboard
3. Use your `GRAPHQL_ADMIN_SECRET` to access admin features

### 2. Set Up Your Database Schema

You can create your database schema in two ways:

#### Option A: Using Hasura Console (Starter Mode)

1. Navigate to: `https://your-app-xyz.ondigitalocean.app/graphql/console`
2. Go to the **DATA** tab
3. Click **Create Table**
4. Define your table structure
5. Set permissions for different user roles

#### Option B: Using Migrations (Production Recommended)

Create migration files and apply them programmatically. See the [Hasura Migrations Guide](https://hasura.io/docs/latest/migrations-metadata-seeds/overview/).

### 3. Connect Your Frontend

Install the Nhost JavaScript SDK:

```bash
npm install @nhost/nhost-js
# or
yarn add @nhost/nhost-js
```

Initialize Nhost in your application:

```javascript
import { NhostClient } from '@nhost/nhost-js'

const nhost = new NhostClient({
  subdomain: 'https://your-app-xyz.ondigitalocean.app',
  region: '' // Leave empty when using custom URL
})
```

### 4. Implement Authentication

#### Sign Up a User

```javascript
const { session, error } = await nhost.auth.signUp({
  email: 'user@example.com',
  password: 'secure-password'
})

if (error) {
  console.error('Sign up error:', error)
} else {
  console.log('User signed up:', session)
}
```

#### Sign In

```javascript
const { session, error } = await nhost.auth.signIn({
  email: 'user@example.com',
  password: 'secure-password'
})

if (error) {
  console.error('Sign in error:', error)
} else {
  console.log('User signed in:', session)
}
```

#### Check Authentication State

```javascript
// Get current user
const user = nhost.auth.getUser()

// Listen for auth state changes
nhost.auth.onAuthStateChanged((event, session) => {
  console.log(`Auth state: ${event}`, session)
})
```

### 5. Query Data with GraphQL

#### Fetch Data

```javascript
const { data, error } = await nhost.graphql.request(`
  query {
    users {
      id
      email
      displayName
    }
  }
`)

if (error) {
  console.error('GraphQL error:', error)
} else {
  console.log('Users:', data.users)
}
```

#### Insert Data

```javascript
const { data, error } = await nhost.graphql.request(`
  mutation($object: users_insert_input!) {
    insert_users_one(object: $object) {
      id
      email
    }
  }
`, {
  object: {
    email: 'newuser@example.com',
    displayName: 'New User'
  }
})
```

### 6. Upload and Download Files

#### Upload a File

```javascript
const { fileMetadata, error } = await nhost.storage.upload({
  file: fileInputElement.files[0],
  bucketId: 'default'
})

if (error) {
  console.error('Upload error:', error)
} else {
  console.log('File uploaded:', fileMetadata)
}
```

#### Get File URL

```javascript
const url = nhost.storage.getPublicUrl({
  fileId: fileMetadata.id
})

// Use in <img src={url} />
```

#### Download a File

```javascript
const { file, error } = await nhost.storage.download({
  fileId: 'file-id-here'
})
```

### 7. Use Serverless Functions

You can create custom serverless functions by mounting a volume with your function code. Functions can be JavaScript or TypeScript.

Example function structure:
```
functions/
  hello-world.ts
  users/
    list.ts
```

Call functions from your frontend:

```javascript
const { data, error } = await nhost.functions.call('/hello-world', {
  method: 'POST',
  body: { name: 'World' }
})
```

## Framework-Specific SDKs

Nhost provides SDKs for popular frameworks:

### React
```bash
npm install @nhost/react
```

```jsx
import { NhostProvider, useAuthenticated } from '@nhost/react'

function App() {
  return (
    <NhostProvider nhost={nhost}>
      <YourApp />
    </NhostProvider>
  )
}
```

### Next.js
```bash
npm install @nhost/nextjs
```

### Vue
```bash
npm install @nhost/vue
```

### React Native
```bash
npm install @nhost/react-native
```

## Example: Building a Todo App

Here's a complete example of a simple todo app:

```javascript
import { NhostClient } from '@nhost/nhost-js'

const nhost = new NhostClient({
  subdomain: 'https://your-app-xyz.ondigitalocean.app',
  region: ''
})

// Sign in
await nhost.auth.signIn({
  email: 'user@example.com',
  password: 'password'
})

// Create a todo table in Hasura Console first with columns:
// - id (uuid, primary key, default: gen_random_uuid())
// - title (text)
// - completed (boolean, default: false)
// - user_id (uuid, foreign key to users.id)

// Insert a todo
const { data } = await nhost.graphql.request(`
  mutation($title: String!) {
    insert_todos_one(object: { title: $title }) {
      id
      title
      completed
    }
  }
`, { title: 'Buy groceries' })

// Fetch todos
const { data: todos } = await nhost.graphql.request(`
  query {
    todos(order_by: { created_at: desc }) {
      id
      title
      completed
    }
  }
`)

// Update todo
const { data: updated } = await nhost.graphql.request(`
  mutation($id: uuid!, $completed: Boolean!) {
    update_todos_by_pk(
      pk_columns: { id: $id }
      _set: { completed: $completed }
    ) {
      id
      completed
    }
  }
`, { id: todoId, completed: true })

// Delete todo
const { data: deleted } = await nhost.graphql.request(`
  mutation($id: uuid!) {
    delete_todos_by_pk(id: $id) {
      id
    }
  }
`, { id: todoId })
```

## Database Permissions

Remember to set up proper permissions in Hasura Console:

1. Go to **DATA** > **Your Table** > **Permissions**
2. For the **user** role:
   - **Insert**: `{ "user_id": { "_eq": "X-Hasura-User-Id" } }`
   - **Select**: `{ "user_id": { "_eq": "X-Hasura-User-Id" } }`
   - **Update**: `{ "user_id": { "_eq": "X-Hasura-User-Id" } }`
   - **Delete**: `{ "user_id": { "_eq": "X-Hasura-User-Id" } }`

This ensures users can only access their own data.

## Storage Permissions

Configure storage permissions in the Hasura Console:

1. Navigate to **DATA** > **storage** > **buckets**
2. Set permissions for file upload/download
3. Use rules to control access based on user roles

## Common Use Cases

### 1. SaaS Application
- Use auth for user management
- Store user data in PostgreSQL via GraphQL
- Upload user files to storage
- Use functions for custom business logic

### 2. Mobile App Backend
- Authenticate mobile users
- Sync data via GraphQL subscriptions
- Store media files in storage
- Push notifications via functions

### 3. Blog or CMS
- Manage posts and authors via GraphQL
- Upload images to storage
- Public read, authenticated write
- Use functions for custom routes

## Resources

- [Nhost Documentation](https://docs.nhost.io/)
- [Hasura Documentation](https://hasura.io/docs/)
- [Nhost Examples](https://github.com/nhost/nhost/tree/main/examples)
- [GraphQL Tutorial](https://graphql.org/learn/)

## Getting Help

- [Nhost GitHub Discussions](https://github.com/nhost/nhost/discussions)
- [Nhost Discord](https://discord.com/invite/9V7Qb2U)
- [DigitalOcean Community](https://www.digitalocean.com/community/tags/app-platform)

## Next Steps

1. Create your database schema using Hasura Console
2. Set up authentication in your frontend app
3. Build GraphQL queries for your data
4. Implement file upload for user content
5. Deploy your frontend app (can also use App Platform!)
6. See [PRODUCTION.md](./PRODUCTION.md) for production best practices

---

Now you're ready to build your application with Nhost on DigitalOcean App Platform!
