# Flare Stack Blog - Deployment Guide

This guide covers deploying the flare-stack-blog to Cloudflare Workers.

## Prerequisites

1. **Cloudflare Account** with access to:
   - Workers & Pages
   - D1 Database
   - R2 Storage
   - KV Namespaces
   - Workflows
   - Queues
   - AI (optional)

2. **GitHub Account** for OAuth authentication

3. **Bun** installed locally (>= 1.3)

4. **Wrangler CLI** authenticated with Cloudflare

## Step 1: Cloudflare Resources Setup

### 1.1 Create D1 Database
```bash
wrangler d1 create flare-stack-blog-db
```
Note the database ID for later.

### 1.2 Create R2 Bucket
```bash
wrangler r2 bucket create flare-stack-blog-media
```

### 1.3 Create KV Namespaces
```bash
wrangler kv:namespace create "KV"
wrangler kv:namespace create "OAUTH_KV"
```
Note both namespace IDs.

### 1.4 Create Queue
```bash
wrangler queues create blog-queue
```

## Step 2: GitHub OAuth Setup

1. Go to GitHub Settings → Developer Settings → OAuth Apps
2. Create New OAuth App:
   - Application name: Your Blog Name
   - Homepage URL: `https://your-domain.com`
   - Authorization callback URL: `https://your-domain.com/api/auth/callback/github`
3. Save the Client ID and Client Secret

## Step 3: Configuration

### 3.1 Copy and Edit Configuration Files

```bash
# Copy example files
cp .env.example .env
cp .dev.vars.example .dev.vars
cp wrangler.example.jsonc wrangler.jsonc
```

### 3.2 Edit `.env` (Client-side variables)
```
CLOUDFLARE_ACCOUNT_ID=your-account-id
CLOUDFLARE_DATABASE_ID=your-d1-database-id
CLOUDFLARE_D1_TOKEN=your-d1-api-token
```

### 3.3 Edit `.dev.vars` (Server-side secrets)
```
ENVIRONMENT=production
BETTER_AUTH_SECRET=$(openssl rand -hex 32)
BETTER_AUTH_URL=https://your-domain.com
ADMIN_EMAIL=your-email@example.com
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
CLOUDFLARE_ZONE_ID=your-zone-id
CLOUDFLARE_PURGE_API_TOKEN=your-purge-api-token
DOMAIN=your-domain.com
PAGEVIEW_SALT=$(openssl rand -hex 16)
```

### 3.4 Edit `wrangler.jsonc`
Update the placeholders:
- `your-domain.com` → Your actual domain
- `your-d1-database-id` → D1 database ID from step 1.1
- `your-r2-bucket-name` → R2 bucket name from step 1.2
- `your-kv-namespace-id` → KV namespace IDs from step 1.3

## Step 4: Install Dependencies

```bash
bun install
```

## Step 5: Database Migration

```bash
# Run migrations on remote D1
bun db:migrate
```

## Step 6: Deploy

```bash
# Deploy to Cloudflare Workers
bun run deploy
```

Or use the deploy script which includes migration:
```bash
bun deploy
```

## Step 7: Post-Deployment

### 7.1 Set up Custom Domain (optional)
In Cloudflare Dashboard:
1. Go to Workers & Pages → Your Worker
2. Settings → Triggers → Custom Domains
3. Add your domain

### 7.2 Configure R2 Public Access (optional)
If you want direct R2 access:
1. Go to R2 → Your Bucket → Settings
2. Enable Public Access
3. Configure custom domain for R2

### 7.3 First Admin Setup
1. Visit `https://your-domain.com/register`
2. Register with the email set in `ADMIN_EMAIL`
3. Verify email (check console in dev mode, or email in production)
4. You'll automatically have admin access

## Environment Variables Reference

### Required
| Variable | Description |
|----------|-------------|
| `BETTER_AUTH_SECRET` | Session encryption key (generate with `openssl rand -hex 32`) |
| `BETTER_AUTH_URL` | Your blog URL |
| `ADMIN_EMAIL` | Admin email address |
| `GITHUB_CLIENT_ID` | GitHub OAuth App Client ID |
| `GITHUB_CLIENT_SECRET` | GitHub OAuth App Client Secret |
| `DOMAIN` | Your blog domain |

### Optional
| Variable | Description |
|----------|-------------|
| `TURNSTILE_SECRET_KEY` | Cloudflare Turnstile for spam protection |
| `VITE_TURNSTILE_SITE_KEY` | Turnstile site key (client-side) |
| `UMAMI_SRC` | Umami analytics source URL |
| `VITE_UMAMI_WEBSITE_ID` | Umami website ID |
| `GITHUB_TOKEN` | For version check API (avoids rate limits) |

## Troubleshooting

### Migration fails
```bash
# Check migration status
wrangler d1 migrations list DB --remote

# Apply migrations manually
wrangler d1 migrations apply DB --remote
```

### Worker fails to start
Check logs in Cloudflare Dashboard → Workers → Your Worker → Logs

### Database connection issues
Verify `database_id` in `wrangler.jsonc` matches your D1 database ID.

## Development

```bash
# Start dev server
bun dev

# Run tests
bun test

# Type check
bun typecheck

# Lint
bun lint
```

## Additional Resources

- [Official Deployment Guide (Chinese)](https://blog.dukda.com/post/flare-stack-blog%E9%83%A8%E7%BD%B2%E6%95%99%E7%A8%8B)
- [Video Tutorial (Bilibili)](https://www.bilibili.com/video/BV1R4fnBhEs4?p=2)
- [Theme Development Guide](./docs/theme-guide.md)
