# Nhost on DigitalOcean App Platform

Deploy your own Nhost Backend-as-a-Service on DigitalOcean App Platform with one click!

[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/AppPlatform-Templates/nhost-appplatform/tree/main)

## What is Nhost?

Nhost is an open-source **Firebase alternative** that provides a complete backend platform for building modern applications:

- **PostgreSQL Database** - Powerful relational database for your data
- **GraphQL API** - Instant, type-safe API powered by Hasura
- **Authentication** - Built-in user management with JWT tokens
- **Storage** - File upload/download with access control
- **Serverless Functions** - Custom backend logic in JavaScript/TypeScript
- **Dashboard** - Web interface for managing your backend

## Features

This deployment includes:

- **5 Microservices**:
  - GraphQL Engine (Hasura v2.46.0)
  - Auth Service (v0.40.2)
  - Storage Service (v0.7.2)
  - Functions Runtime (Node.js 22)
  - Admin Dashboard (v2.39.0)

- **Infrastructure**:
  - Managed PostgreSQL 16 database
  - DigitalOcean Spaces for file storage
  - Built-in load balancing
  - Automatic HTTPS/SSL

- **Two Deployment Tiers**:
  - **Starter**: Dev database, small instances, Hasura Console (~$42/month)
  - **Production**: Managed database with HA, scaled instances (~$80/month)

## Quick Start

### Prerequisites

Before deploying, you'll need:

1. **DigitalOcean Spaces Bucket** - For file storage
   - Create at: https://cloud.digitalocean.com/spaces
   - Note the bucket name and region

2. **Spaces API Keys** - For storage authentication
   - Generate at: https://cloud.digitalocean.com/account/api/spaces
   - Save access key and secret key

3. **SMTP Provider** - For sending emails (authentication, password resets)
   - Options: SendGrid (free tier), Mailgun, AWS SES
   - Get SMTP credentials from your provider

4. **Strong Secrets** - For security
   ```bash
   # Generate admin secret
   openssl rand -hex 32

   # Generate JWT secret
   JWT_KEY=$(openssl rand -hex 32)
   echo "{\"type\":\"HS256\",\"key\":\"$JWT_KEY\"}"
   ```

### Deploy

#### Option 1: Deploy to DO Button (Recommended for First-Time Users)

1. Click the **Deploy to DO** button above
2. Enter required environment variables (see [ENV_TEMPLATE.md](./ENV_TEMPLATE.md))
3. Choose your region
4. Click **Deploy** and wait 5-10 minutes

For detailed instructions, see [DEPLOY_TO_DO.md](./DEPLOY_TO_DO.md).

#### Option 2: Deploy via CLI (For Advanced Users)

```bash
# Clone this repository
git clone https://github.com/AppPlatform-Templates/nhost-appplatform.git
cd nhost-appplatform

# Edit .do/app.yaml and replace all <PLACEHOLDER> values with your actual values
# See ENV_TEMPLATE.md for guidance on each variable

# Deploy using doctl
doctl apps create --spec .do/app.yaml

# Or for production deployment
doctl apps create --spec .do/examples/production.yaml
```

## Documentation

- **[DEPLOY_TO_DO.md](./DEPLOY_TO_DO.md)** - Step-by-step deployment guide
- **[HOW_TO_USE_NHOST.md](./HOW_TO_USE_NHOST.md)** - Building apps with Nhost
- **[ENV_TEMPLATE.md](./ENV_TEMPLATE.md)** - All environment variables explained
- **[PRODUCTION.md](./PRODUCTION.md)** - Production best practices and scaling
- **[VERSION.md](./VERSION.md)** - Component versions and update guide

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  DigitalOcean App Platform               │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   Dashboard  │  │   GraphQL    │  │     Auth     │ │
│  │   (Admin)    │  │   (Hasura)   │  │   Service    │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │   Storage    │  │  Functions   │                    │
│  │   Service    │  │   Runtime    │                    │
│  └──────────────┘  └──────────────┘                    │
│                           │                             │
└───────────────────────────┼─────────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            │                               │
    ┌───────▼────────┐           ┌─────────▼──────────┐
    │   PostgreSQL   │           │  Spaces (Storage)  │
    │   Database     │           │   S3-Compatible    │
    └────────────────┘           └────────────────────┘
```

## Use Cases

### SaaS Applications
Build multi-tenant applications with user authentication, data isolation, and file storage.

### Mobile App Backends
GraphQL API provides efficient data fetching for mobile apps with built-in auth and offline sync.

### Internal Tools & Dashboards
Rapid backend development for admin panels, dashboards, and internal tools.

### E-commerce Platforms
Product catalog, user accounts, order management, and file uploads out of the box.

## Pricing

### Starter Tier (Default)
- **Services**: 5 × $6/month = $30/month
- **Dev Database**: $7/month (PostgreSQL)
- **Spaces**: $5/month
- **Total**: ~**$42/month**

### Production Tier
- **Services**: ~$60/month (scaled instances)
- **Managed DB**: ~$30/month (with HA)
- **Spaces**: $5/month
- **Total**: ~**$95/month**

See [PRODUCTION.md](./PRODUCTION.md) for production configuration.

## Example: Building a Todo App

```javascript
import { NhostClient } from '@nhost/nhost-js'

// Initialize Nhost
const nhost = new NhostClient({
  subdomain: 'https://your-app.ondigitalocean.app',
  region: ''
})

// Sign up user
await nhost.auth.signUp({
  email: 'user@example.com',
  password: 'secure-password'
})

// Sign in
await nhost.auth.signIn({
  email: 'user@example.com',
  password: 'secure-password'
})

// Insert data
const { data } = await nhost.graphql.request(`
  mutation($title: String!) {
    insert_todos_one(object: { title: $title }) {
      id
      title
    }
  }
`, { title: 'Buy groceries' })

// Query data
const { data: todos } = await nhost.graphql.request(`
  query {
    todos {
      id
      title
      completed
    }
  }
`)
```

For more examples, see [HOW_TO_USE_NHOST.md](./HOW_TO_USE_NHOST.md).

## Framework Support

Nhost works with any JavaScript/TypeScript framework:

- **React** - `@nhost/react`
- **Next.js** - `@nhost/nextjs`
- **Vue** - `@nhost/vue`
- **Svelte** - `@nhost/svelte`
- **React Native** - `@nhost/react-native`
- **Flutter/Dart** - `nhost_sdk`

See the [Nhost documentation](https://docs.nhost.io/) for framework-specific guides.

## Security

This deployment includes:

- **JWT-based authentication** with secure token handling
- **Row-level security** via Hasura permissions
- **HTTPS/SSL** enforced by App Platform
- **Environment-based secrets** management
- **Rate limiting** on auth endpoints
- **CORS configuration** for frontend apps

For production security, see [PRODUCTION.md](./PRODUCTION.md).

## Monitoring

Access logs and metrics in the App Platform console:

1. Go to your app in [App Platform](https://cloud.digitalocean.com/apps)
2. Click **Insights** for metrics (CPU, memory, requests)
3. Click **Runtime Logs** for application logs
4. Set up alerts for error rates and performance

## Updating

To update to the latest Nhost versions:

1. Check [VERSION.md](./VERSION.md) for current versions
2. Update image tags in `.do/deploy.template.yaml`
3. Redeploy your app
4. Review [Nhost CHANGELOG](https://github.com/nhost/nhost/blob/main/CHANGELOG.md)

## Troubleshooting

### Common Issues

**Database connection errors**
- Wait 5-10 minutes for database to provision
- Check database is running in App Platform console

**SMTP/Email errors**
- Verify SMTP credentials with your provider
- Check sender email is verified
- Review [ENV_TEMPLATE.md](./ENV_TEMPLATE.md) for SMTP configuration

**Spaces/Storage errors**
- Verify bucket name and region match
- Check Spaces access key has read/write permissions
- Ensure bucket exists in DigitalOcean Spaces

**Authentication errors**
- Verify JWT_SECRET is in correct JSON format
- Check GRAPHQL_ADMIN_SECRET is set
- Review [DEPLOY_TO_DO.md](./DEPLOY_TO_DO.md) for secret generation

## Support & Resources

- **Nhost Documentation**: https://docs.nhost.io/
- **Hasura Documentation**: https://hasura.io/docs/
- **DigitalOcean App Platform**: https://docs.digitalocean.com/products/app-platform/
- **Nhost GitHub**: https://github.com/nhost/nhost
- **Nhost Discord**: https://discord.com/invite/9V7Qb2U
- **DigitalOcean Community**: https://www.digitalocean.com/community/tags/app-platform

## Contributing

Found an issue or have a suggestion? Please open an issue on the [GitHub repository](https://github.com/AppPlatform-Templates/nhost-appplatform).

## License

This template is MIT licensed. Nhost components maintain their respective open-source licenses.

## Acknowledgments

Built with:
- [Nhost](https://nhost.io/) - Open source Backend-as-a-Service
- [Hasura](https://hasura.io/) - GraphQL Engine
- [PostgreSQL](https://www.postgresql.org/) - Database
- [DigitalOcean App Platform](https://www.digitalocean.com/products/app-platform) - Hosting

---

**Ready to build?** Click the Deploy to DO button and start building in minutes!

[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/AppPlatform-Templates/nhost-appplatform/tree/main)
