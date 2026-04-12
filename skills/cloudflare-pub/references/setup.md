# Cloudflare Setup Guide

## 1. Get Account ID

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com)
2. Select your account in the left sidebar
3. Navigate to **Workers & Pages** → **Overview**
4. Find **Account ID** on the right side — copy it

## 2. Create API Token

1. Go to [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)
2. Click **Create Token**
3. Choose **Create Custom Token** (at the bottom, "Get started")
4. Configure:

| Field | Value |
|-------|-------|
| **Token name** | `cloudflare-publisher` (or any name you like) |
| **Permissions** | |
| Account → Cloudflare Pages | **Edit** |
| Account → Workers Scripts | **Edit** |
| **Account Resources** | Include → select your account (or All accounts) |
| **Zone Resources** | Not needed — leave empty |
| **TTL** | Optional — token expiration |

5. Click **Continue to summary** → **Create Token**
6. Copy the token (**shown only once!**)

### Minimum required scopes

The script uses only `wrangler pages deploy` and `wrangler pages project create`, so two permissions are sufficient:
- **Cloudflare Pages: Edit** — create/update projects and deployments
- **Workers Scripts: Edit** — required by wrangler for the deploy pipeline

No Zone, DNS, R2, or other permissions are needed.

## 3. Save credentials

Create a `.env` file in one of these locations (checked in order):
1. Next to `publish.py` (in `scripts/`)
2. `~/.claude/cloudflare-pub/.env`
3. Current working directory

```bash
mkdir -p ~/.claude/cloudflare-pub

cat > ~/.claude/cloudflare-pub/.env << 'EOF'
CF_ACCOUNT_ID=your_account_id_here
CF_API_TOKEN=your_api_token_here
EOF
```

## 4. Install wrangler

```bash
npm i -g wrangler
```

## 5. Install python-docx (optional)

Only needed if you plan to publish `.docx` files:

```bash
pip install python-docx
```

## Token rotation

If a token is compromised:
1. Go to [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)
2. Find the token → click **Roll**
3. Update the new token in your `.env` file
