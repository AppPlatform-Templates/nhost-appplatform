# Deploy to DigitalOcean

Deploy Nhost to DigitalOcean App Platform with one click!

[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/AppPlatform-Templates/nhost-appplatform/tree/main)

## Before You Deploy

### 1. Create a Spaces Bucket

Nhost requires DigitalOcean Spaces for file storage:

1. Go to [DigitalOcean Spaces](https://cloud.digitalocean.com/spaces)
2. Click **Create Spaces Bucket**
3. Choose a region (e.g., `nyc3`)
4. Enter a bucket name (e.g., `nhost-storage-prod`)
5. Set permissions to **Private** (recommended)
6. Click **Create Spaces Bucket**

### 2. Get Spaces API Keys

1. Go to [API Tokens](https://cloud.digitalocean.com/account/api/spaces)
2. Click **Generate New Key**
3. Enter a name (e.g., "Nhost Storage")
4. Save the **Access Key** and **Secret Key** securely

### 3. Choose an SMTP Provider

Nhost needs SMTP for sending authentication emails. Choose one:

#### Option A: SendGrid (Recommended)
- Free tier: 100 emails/day
- Sign up: https://sendgrid.com/
- Get API key from Settings > API Keys
- Configuration:
  ```
  SMTP_HOST: smtp.sendgrid.net
  SMTP_PORT: 587
  SMTP_USER: apikey
  SMTP_PASS: <your-sendgrid-api-key>
  SMTP_SECURE: false
  ```

#### Option B: Mailgun
- Free tier: 5,000 emails/month (first 3 months)
- Sign up: https://www.mailgun.com/
- Get SMTP credentials from Sending > Domain Settings > SMTP
- Configuration:
  ```
  SMTP_HOST: smtp.mailgun.org
  SMTP_PORT: 587
  SMTP_USER: <your-mailgun-smtp-username>
  SMTP_PASS: <your-mailgun-smtp-password>
  SMTP_SECURE: false
  ```

#### Option C: AWS SES
- Very affordable ($.10 per 1,000 emails)
- Requires AWS account and domain verification
- Configuration:
  ```
  SMTP_HOST: email-smtp.<region>.amazonaws.com
  SMTP_PORT: 587
  SMTP_USER: <your-ses-smtp-username>
  SMTP_PASS: <your-ses-smtp-password>
  SMTP_SECURE: false
  ```

### 4. Generate Secrets

Generate strong random secrets for security:

```bash
# Generate admin secret
openssl rand -hex 32

# Generate JWT secret key
JWT_KEY=$(openssl rand -hex 32)
echo "{\"type\":\"HS256\",\"key\":\"$JWT_KEY\"}"
```

Save these securely - you'll need them during deployment.

## Deployment Steps

### Step 1: Click Deploy to DO Button

Click the Deploy to DO button at the top of this page.

### Step 2: Configure Your App

You'll be prompted to enter environment variables:

#### Required Variables

1. **GRAPHQL_ADMIN_SECRET**
   - Paste the admin secret you generated
   - This protects your GraphQL API

2. **JWT_SECRET**
   - Paste the JWT secret JSON you generated
   - Format: `{"type":"HS256","key":"your-key-here"}`

3. **Spaces Configuration**
   - **SPACES_REGION**: Your bucket region (e.g., `nyc3`)
   - **SPACES_BUCKET**: Your bucket name
   - **SPACES_ACCESS_KEY**: Your Spaces access key
   - **SPACES_SECRET_KEY**: Your Spaces secret key

4. **SMTP Configuration**
   - **SMTP_HOST**: Your SMTP provider hostname
   - **SMTP_PORT**: Usually `587`
   - **SMTP_USER**: Your SMTP username/apikey
   - **SMTP_PASS**: Your SMTP password/API key
   - **SMTP_SENDER**: Email address for outgoing emails (e.g., `noreply@yourdomain.com`)
   - **SMTP_SECURE**: `false` for port 587, `true` for port 465

5. **FRONTEND_URL**
   - URL where your frontend app will be hosted
   - Can be another App Platform app or external domain
   - Example: `https://myapp.com` or `https://myapp-frontend.ondigitalocean.app`
   - Used for CORS and auth redirects

### Step 3: Review App Configuration

- **App Name**: Change if desired
- **Region**: Choose closest to your users
- **Database**: Dev database ($7/month) included by default

### Step 4: Deploy

Click **Deploy** and wait for the deployment to complete (5-10 minutes).

## After Deployment

### 1. Get Your App URL

Once deployed, find your app URL in the App Platform console:
- Format: `https://your-app-xyz.ondigitalocean.app`

### 2. Access the Dashboard

1. Open your app URL in a browser
2. You'll see the Nhost Dashboard
3. Click "GraphQL API" to access Hasura Console

### 3. Access Hasura Console

Navigate to: `https://your-app-xyz.ondigitalocean.app/graphql/console`

When prompted for admin secret:
- Enter the `GRAPHQL_ADMIN_SECRET` you set during deployment

### 4. Create Your First Table

1. In Hasura Console, go to **DATA** tab
2. Click **Create Table**
3. Name it (e.g., "todos")
4. Add columns:
   - `id` (UUID, Primary Key, default: `gen_random_uuid()`)
   - `title` (Text)
   - `completed` (Boolean, default: `false`)
   - `user_id` (UUID, Nullable)
   - `created_at` (Timestamp, default: `now()`)
5. Click **Add Table**

### 5. Set Up Permissions

1. Go to the **Permissions** tab of your table
2. For the `user` role:
   - **Insert**: Allow users to insert rows
   - **Select**: Allow users to query their own rows
   - **Update**: Allow users to update their own rows
   - **Delete**: Allow users to delete their own rows
3. Use row permissions: `{"user_id": {"_eq": "X-Hasura-User-Id"}}`

### 6. Test Authentication

Create a test user using the GraphQL API:

```graphql
mutation {
  insert_users_one(object: {
    email: "test@example.com",
    password_hash: "test-password"
  }) {
    id
    email
  }
}
```

Or use the Nhost SDK in your frontend (recommended).

## Costs

### Starter Deployment (Default)
- **5 Services**: 5 × $6/month = $30/month
- **Dev Database**: $7/month (PostgreSQL)
- **Spaces**: $5/month (250GB storage + 1TB transfer)
- **Total**: ~$42/month

### Production Deployment
See [PRODUCTION.md](./PRODUCTION.md) for production configuration (~$80/month).

## Common Issues

### SMTP Errors
- **"Authentication failed"**: Check SMTP username and password
- **"Connection refused"**: Verify SMTP_HOST and SMTP_PORT
- **"Sender not verified"**: Verify sender email with your SMTP provider

### Spaces Errors
- **"Access Denied"**: Check Spaces access key and secret
- **"Bucket not found"**: Verify bucket name and region match

### Database Connection Errors
- **"Could not connect"**: Wait a few minutes for database to fully provision
- **"SSL required"**: Database requires SSL (automatically configured)

## Next Steps

1. Read [HOW_TO_USE_NHOST.md](./HOW_TO_USE_NHOST.md) to learn how to build apps with Nhost
2. Review [ENV_TEMPLATE.md](./ENV_TEMPLATE.md) for all configuration options
3. See [PRODUCTION.md](./PRODUCTION.md) when ready to scale up
4. Check [VERSION.md](./VERSION.md) for update instructions

## Support

- [Nhost Documentation](https://docs.nhost.io/)
- [DigitalOcean App Platform Docs](https://docs.digitalocean.com/products/app-platform/)
- [DigitalOcean Community](https://www.digitalocean.com/community/tags/app-platform)
- [Nhost GitHub Discussions](https://github.com/nhost/nhost/discussions)

---

Happy building with Nhost on DigitalOcean!
