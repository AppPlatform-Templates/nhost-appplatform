# Environment Variables Configuration

This document describes all environment variables required to run Nhost on DigitalOcean App Platform.

## Required Variables

### Security & Authentication

#### GRAPHQL_ADMIN_SECRET
- **Description**: Admin secret for accessing Hasura GraphQL Engine
- **Required**: Yes
- **Example**: `my-super-secret-admin-key-change-this`
- **Generation**: Use a strong random string
  ```bash
  openssl rand -hex 32
  ```
- **Used By**: All services (GraphQL, Auth, Storage, Functions, Dashboard)

#### JWT_SECRET
- **Description**: JSON Web Token configuration for authentication
- **Required**: Yes
- **Format**: JSON string with type and key
- **Example**: `{"type":"HS256","key":"your-secret-key-at-least-32-characters-long"}`
- **Generation**:
  ```bash
  SECRET_KEY=$(openssl rand -hex 32)
  echo "{\"type\":\"HS256\",\"key\":\"$SECRET_KEY\"}"
  ```
- **Used By**: GraphQL, Auth, Functions

### DigitalOcean Spaces (Object Storage)

#### SPACES_REGION
- **Description**: DigitalOcean Spaces region
- **Required**: Yes
- **Example**: `nyc3`
- **Options**: `nyc3`, `ams3`, `sfo3`, `sgp1`, `fra1`, `syd1`
- **Used By**: Storage service

#### SPACES_BUCKET
- **Description**: Name of your Spaces bucket for file storage
- **Required**: Yes
- **Example**: `nhost-storage-prod`
- **Creation**: Create via [DigitalOcean Control Panel](https://cloud.digitalocean.com/spaces)
- **Used By**: Storage service

#### SPACES_ACCESS_KEY
- **Description**: Spaces API access key
- **Required**: Yes
- **Example**: `DO00EXAMPLE123456789`
- **Creation**: Generate in [API section of Spaces](https://cloud.digitalocean.com/account/api/spaces)
- **Used By**: Storage service

#### SPACES_SECRET_KEY
- **Description**: Spaces API secret key
- **Required**: Yes
- **Example**: `exampleSecretKey123456789ABCDEFGH`
- **Creation**: Generated with access key
- **Used By**: Storage service

### SMTP Configuration (Email)

#### SMTP_HOST
- **Description**: SMTP server hostname
- **Required**: Yes
- **Example**: `smtp.sendgrid.net`
- **Providers**: SendGrid, Mailgun, AWS SES, etc.
- **Used By**: Auth service (for sending verification emails, password resets)

#### SMTP_PORT
- **Description**: SMTP server port
- **Required**: Yes
- **Example**: `587` (TLS) or `465` (SSL)
- **Used By**: Auth service

#### SMTP_USER
- **Description**: SMTP authentication username
- **Required**: Yes
- **Example**: `apikey` (SendGrid) or your SMTP username
- **Used By**: Auth service

#### SMTP_PASS
- **Description**: SMTP authentication password/API key
- **Required**: Yes
- **Example**: `SG.abc123...` (SendGrid API key)
- **Used By**: Auth service

#### SMTP_SENDER
- **Description**: Email address for outgoing emails
- **Required**: Yes
- **Example**: `noreply@yourdomain.com`
- **Note**: Must be verified with your SMTP provider
- **Used By**: Auth service

#### SMTP_SECURE
- **Description**: Whether to use secure connection (TLS/SSL)
- **Required**: Yes
- **Example**: `true` (port 465) or `false` (port 587 with STARTTLS)
- **Used By**: Auth service

### Application URLs

#### FRONTEND_URL
- **Description**: URL of your frontend application
- **Required**: Yes
- **Example**: `https://myapp.com` or `https://myapp-frontend-xyz.ondigitalocean.app`
- **Purpose**: Used for CORS, redirects after authentication
- **Used By**: Auth service

## Optional Variables

### Production Configuration

#### CORS_ALLOWED_ORIGINS
- **Description**: Comma-separated list of allowed CORS origins
- **Required**: No (defaults to `*` in starter)
- **Example**: `https://myapp.com,https://www.myapp.com`
- **Used By**: GraphQL service (production only)
- **Note**: Set to specific domains in production for security

## Auto-Populated Variables

These variables are automatically populated by App Platform:

### APP_URL
- **Description**: Public URL of your deployed Nhost app
- **Auto-populated**: Yes
- **Format**: `https://your-app-xyz.ondigitalocean.app`
- **Used By**: Storage, Dashboard (for constructing service URLs)

### DATABASE_URL (via `${db.DATABASE_URL}`)
- **Description**: PostgreSQL connection string
- **Auto-populated**: Yes (from database component)
- **Format**: `postgresql://user:pass@host:port/dbname?sslmode=require`
- **Used By**: All services

## Environment Variable Setup

### Via Deploy-to-DO Button

When using the Deploy-to-DO button, you'll be prompted to enter all required variables.

### Via DigitalOcean Control Panel

1. Go to your app in the [App Platform console](https://cloud.digitalocean.com/apps)
2. Navigate to **Settings** > **App-Level Environment Variables**
3. Click **Edit** and add each variable
4. Save and redeploy

### Via doctl CLI

```bash
# Set environment variables
doctl apps update YOUR_APP_ID \
  --env "GRAPHQL_ADMIN_SECRET=your-secret" \
  --env "JWT_SECRET={\"type\":\"HS256\",\"key\":\"your-key\"}" \
  --env "SPACES_REGION=nyc3" \
  --env "SPACES_BUCKET=your-bucket" \
  --env "SPACES_ACCESS_KEY=your-key" \
  --env "SPACES_SECRET_KEY=your-secret" \
  --env "SMTP_HOST=smtp.sendgrid.net" \
  --env "SMTP_PORT=587" \
  --env "SMTP_USER=apikey" \
  --env "SMTP_PASS=your-api-key" \
  --env "SMTP_SENDER=noreply@yourdomain.com" \
  --env "SMTP_SECURE=false" \
  --env "FRONTEND_URL=https://yourapp.com"
```

## Security Best Practices

1. **Never commit secrets**: Don't add actual secrets to your repository
2. **Use strong secrets**: Generate random strings for GRAPHQL_ADMIN_SECRET and JWT keys
3. **Rotate secrets regularly**: Update secrets periodically
4. **Restrict CORS**: In production, set specific CORS_ALLOWED_ORIGINS
5. **Use environment-specific values**: Different secrets for dev/staging/production

## Quick Setup Checklist

- [ ] Generate `GRAPHQL_ADMIN_SECRET` (random 32+ char string)
- [ ] Generate `JWT_SECRET` (JSON format with HS256 key)
- [ ] Create DigitalOcean Spaces bucket
- [ ] Get Spaces access key and secret
- [ ] Configure SMTP provider (SendGrid, Mailgun, etc.)
- [ ] Set SMTP credentials
- [ ] Set frontend URL
- [ ] (Production) Configure CORS allowed origins

## SMTP Provider Setup Guides

### SendGrid
```bash
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASS=<your-sendgrid-api-key>
SMTP_SECURE=false
```

### Mailgun
```bash
SMTP_HOST=smtp.mailgun.org
SMTP_PORT=587
SMTP_USER=<your-mailgun-smtp-username>
SMTP_PASS=<your-mailgun-smtp-password>
SMTP_SECURE=false
```

### AWS SES
```bash
SMTP_HOST=email-smtp.us-east-1.amazonaws.com
SMTP_PORT=587
SMTP_USER=<your-ses-smtp-username>
SMTP_PASS=<your-ses-smtp-password>
SMTP_SECURE=false
```

## Troubleshooting

### JWT_SECRET format error
- Ensure JSON is properly escaped: `{\"type\":\"HS256\",\"key\":\"...\"}`
- Key should be at least 32 characters

### SMTP connection errors
- Verify SMTP credentials with your provider
- Check firewall rules allow outbound SMTP connections
- Ensure sender email is verified with provider

### Spaces connection errors
- Verify bucket exists in specified region
- Check access key has read/write permissions
- Ensure bucket name matches exactly (case-sensitive)
