# Production Deployment Guide

This guide covers production best practices, security hardening, and scaling for your Nhost deployment on DigitalOcean App Platform.

## Production Architecture

The production configuration (`.do/examples/production.yaml`) includes:

### Infrastructure
- **2+ instances per service** for high availability
- **Managed PostgreSQL** with standby node for failover
- **Larger instance sizes** for better performance
- **Console disabled** (schema managed via migrations)

### Cost Estimate
- **Services**: 9 instances × $6-12/month = ~$60/month
- **Database**: Managed PostgreSQL (2 nodes) = ~$30/month
- **Spaces**: $5/month
- **Total**: ~$95/month (scales with usage)

## Pre-Production Checklist

### 1. Security Configuration

#### Update Secrets
```bash
# Generate strong secrets
ADMIN_SECRET=$(openssl rand -hex 32)
JWT_KEY=$(openssl rand -hex 32)
JWT_SECRET="{\"type\":\"HS256\",\"key\":\"$JWT_KEY\"}"

echo "GRAPHQL_ADMIN_SECRET: $ADMIN_SECRET"
echo "JWT_SECRET: $JWT_SECRET"
```

#### Configure CORS
Set specific allowed origins instead of `*`:

```bash
# In App Platform settings, add environment variable:
CORS_ALLOWED_ORIGINS=https://myapp.com,https://www.myapp.com,https://app.myapp.com
```

#### Disable Dev Mode Features
In production config, ensure:
- `HASURA_GRAPHQL_DEV_MODE: "false"`
- `HASURA_GRAPHQL_ENABLE_CONSOLE: "false"`
- `AUTH_CONCEAL_ERRORS: "true"`
- `HASURA_GRAPHQL_ADMIN_INTERNAL_ERRORS: "false"`

### 2. Database Configuration

#### Upgrade to Managed Database

1. In App Platform console, go to your app
2. Navigate to **Database** component
3. Click **Edit** > **Upgrade to Production Database**
4. Choose:
   - **Size**: `db-s-2vcpu-4gb` (minimum for production)
   - **Nodes**: 2 (primary + standby)
   - **Region**: Same as your app

#### Enable Connection Pooling
For high-traffic apps, enable PgBouncer:

1. In DigitalOcean database settings
2. Go to **Connection Pools**
3. Create pool with mode: **Transaction**
4. Update `DATABASE_URL` to use pool connection string

#### Set Up Backups
- Managed databases auto-backup daily
- Configure backup retention (7-30 days)
- Test restore procedure before going live

### 3. Email Configuration

#### Production SMTP Setup

For production, use a reliable SMTP provider:

**SendGrid** (Recommended)
- Verify sender domain (not just email)
- Use dedicated IP for high volume
- Configure SPF/DKIM records
- Monitor deliverability

**AWS SES** (Cost-effective at scale)
- Move out of sandbox mode (requires request)
- Verify sending domain
- Set up bounce/complaint handling
- Configure SNS notifications

#### Email Verification
Enable email verification in production:
```bash
AUTH_EMAIL_SIGNIN_EMAIL_VERIFIED_REQUIRED=true
```

### 4. Storage Configuration

#### Spaces Security

1. **Enable CDN** for better performance:
   ```bash
   # Use CDN endpoint in configuration
   S3_ENDPOINT=https://<bucket>.nyc3.cdn.digitaloceanspaces.com
   ```

2. **Set CORS rules** on your Spaces bucket:
   ```json
   {
     "CORSRules": [
       {
         "AllowedOrigins": ["https://myapp.com"],
         "AllowedMethods": ["GET", "PUT", "POST"],
         "AllowedHeaders": ["*"],
         "ExposeHeaders": ["ETag"],
         "MaxAgeSeconds": 3000
       }
     ]
   }
   ```

3. **Configure lifecycle policies** for old files:
   - Auto-delete temporary files after 30 days
   - Archive old backups to Glacier-equivalent

#### File Upload Limits
Configure in Nhost Storage environment:
```bash
# Max file size (in bytes)
MAX_FILE_SIZE=52428800  # 50MB
```

### 5. Monitoring & Logging

#### Enable App Platform Insights

1. Go to app in console
2. Click **Insights** tab
3. Monitor:
   - Response times
   - Error rates
   - Memory/CPU usage
   - Request counts

#### Set Up Alerts

1. Go to **Settings** > **Alerts**
2. Configure alerts for:
   - High error rate (>1%)
   - High response time (>2s avg)
   - Memory usage (>85%)
   - Database connections (>80% of max)

#### Configure Structured Logging

Set appropriate log levels for production:
```bash
HASURA_GRAPHQL_LOG_LEVEL=warn
HASURA_GRAPHQL_ENABLED_LOG_TYPES=startup,http-log,webhook-log,websocket-log
```

### 6. Database Schema Management

#### Use Migrations (Not Console)

1. **Install Hasura CLI locally:**
   ```bash
   npm install -g hasura-cli
   ```

2. **Initialize migrations:**
   ```bash
   hasura init my-nhost-project
   cd my-nhost-project
   ```

3. **Connect to your database:**
   ```bash
   hasura console --endpoint https://your-app.ondigitalocean.app/graphql \
     --admin-secret your-admin-secret
   ```

4. **Create migrations:**
   - Make schema changes in the console
   - Migrations are auto-generated in `migrations/` folder
   - Commit these to git

5. **Apply migrations in production:**
   ```bash
   hasura migrate apply --endpoint https://your-app.ondigitalocean.app/graphql \
     --admin-secret your-admin-secret
   ```

### 7. Performance Optimization

#### Database Indexes

Add indexes for frequently queried columns:
```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_todos_user_id ON todos(user_id);
CREATE INDEX idx_todos_created_at ON todos(created_at);
```

#### Enable Query Caching

In Hasura, cache frequently accessed queries:
```graphql
query GetUser @cached(ttl: 60) {
  users_by_pk(id: "user-id") {
    id
    email
    displayName
  }
}
```

#### Connection Pool Settings

Optimize PostgreSQL connections:
```bash
HASURA_GRAPHQL_PG_CONNECTIONS=50  # Adjust based on workload
HASURA_GRAPHQL_PG_TIMEOUT=180
```

### 8. Rate Limiting

Auth service has built-in rate limiting. Adjust for production:
```bash
# Brute force protection
AUTH_RATE_LIMIT_BRUTE_FORCE_BURST=10
AUTH_RATE_LIMIT_BRUTE_FORCE_INTERVAL=5m

# Email rate limits
AUTH_RATE_LIMIT_EMAIL_BURST=10
AUTH_RATE_LIMIT_EMAIL_INTERVAL=1h

# Signup protection
AUTH_RATE_LIMIT_SIGNUPS_BURST=10
AUTH_RATE_LIMIT_SIGNUPS_INTERVAL=5m

# Global rate limit
AUTH_RATE_LIMIT_GLOBAL_BURST=100
AUTH_RATE_LIMIT_GLOBAL_INTERVAL=1m
```

### 9. Authentication Hardening

#### Token Expiration
Set appropriate token lifetimes:
```bash
AUTH_ACCESS_TOKEN_EXPIRES_IN=900        # 15 minutes
AUTH_REFRESH_TOKEN_EXPIRES_IN=2592000  # 30 days
```

#### Password Requirements
```bash
AUTH_PASSWORD_MIN_LENGTH=12
AUTH_PASSWORD_HIBP_ENABLED=true  # Check against breached passwords
```

#### MFA (Multi-Factor Authentication)
Enable for sensitive applications:
```bash
AUTH_MFA_ENABLED=true
AUTH_MFA_TOTP_ISSUER=YourApp
```

### 10. Backup Strategy

#### Database Backups
- **Automatic**: Managed database backs up daily
- **Manual**: Take backup before major changes
  ```bash
  doctl databases backup list <database-id>
  ```

#### Spaces Backups
- Enable versioning on Spaces bucket
- Set up cross-region replication for critical data
- Document restore procedures

#### Configuration Backups
- Store all `.yaml` configs in git
- Document environment variable values (encrypted)
- Keep runbook for disaster recovery

## Scaling Strategies

### Horizontal Scaling

Increase instance counts for specific services:

```bash
# Scale GraphQL for read-heavy workloads
doctl apps update <app-id> --spec .do/examples/production.yaml
# Edit graphql service: instance_count: 4
```

### Vertical Scaling

Upgrade instance sizes for CPU/memory-intensive services:

```bash
# Upgrade to larger instances
instance_size_slug: apps-s-2vcpu-4gb  # from apps-s-1vcpu-1gb
```

### Database Scaling

1. **Read Replicas**: Add read replicas for read-heavy workloads
2. **Larger Nodes**: Upgrade to `db-s-4vcpu-8gb` or higher
3. **Connection Pooling**: Use PgBouncer for high concurrency

### CDN Integration

For global applications:
1. Use Spaces CDN for static assets
2. Consider Cloudflare in front of App Platform
3. Cache GraphQL queries where appropriate

## Deployment Process

### 1. Staging Environment

Create a staging environment for testing:
```bash
# Deploy staging app
doctl apps create --spec .do/deploy.template.yaml
```

### 2. CI/CD Pipeline

Set up automated deployments:
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to App Platform
        run: |
          doctl apps update ${{ secrets.APP_ID }} \
            --spec .do/examples/production.yaml
```

### 3. Blue-Green Deployments

For zero-downtime deployments:
1. Deploy new version alongside old
2. Test new version
3. Switch traffic to new version
4. Keep old version for quick rollback

### 4. Rollback Plan

Document rollback procedure:
```bash
# Rollback to previous deployment
doctl apps list-deployments <app-id>
doctl apps create-deployment <app-id> --rollback <previous-deployment-id>
```

## Security Checklist

- [ ] Strong, unique secrets for all services
- [ ] CORS configured for specific domains
- [ ] Dev mode and console disabled
- [ ] Email verification enabled
- [ ] Rate limiting enabled
- [ ] HTTPS enforced (automatic with App Platform)
- [ ] Database backups configured
- [ ] Monitoring and alerts set up
- [ ] MFA enabled (if applicable)
- [ ] Password policies enforced
- [ ] Regular security updates scheduled
- [ ] Incident response plan documented

## Compliance Considerations

### GDPR
- Implement data export functionality
- Provide data deletion endpoints
- Log consent and privacy policy acceptance
- Set up data retention policies

### SOC 2
- Enable audit logging
- Implement access controls
- Document security procedures
- Regular security reviews

## Cost Optimization

### Right-Sizing
- Monitor resource usage via Insights
- Scale down underutilized services
- Use scheduled scaling for predictable traffic patterns

### Database Optimization
- Optimize queries to reduce database load
- Clean up unused data regularly
- Consider archiving old data

### Spaces Optimization
- Implement lifecycle policies
- Compress images before upload
- Use CDN caching effectively

## Support & Monitoring

### Health Checks
All services have health check endpoints:
- GraphQL: `/healthz`
- Auth: `/healthz`
- Functions: `/healthz`

Monitor these for service health.

### Performance Metrics
Track key metrics:
- P50/P95/P99 response times
- Error rates by service
- Database query performance
- API request rates

### On-Call Procedures
Document procedures for:
- Service outages
- Database issues
- Storage problems
- Security incidents

## Additional Resources

- [Hasura Production Checklist](https://hasura.io/docs/latest/deployment/production-checklist/)
- [PostgreSQL Performance Tuning](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [DigitalOcean Spaces Best Practices](https://docs.digitalocean.com/products/spaces/resources/best-practices/)
- [App Platform Scaling Guide](https://docs.digitalocean.com/products/app-platform/how-to/scale-app/)

---

Following these practices will ensure your Nhost deployment is production-ready, secure, and scalable.
