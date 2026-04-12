# Troubleshooting

## Common errors

### `wrangler: command not found`

wrangler CLI is not installed. Fix:
```bash
npm i -g wrangler
```

### `Authentication error [code: 10000]`

Possible causes:
- Token lacks required scopes (`Cloudflare Pages: Edit` + `Workers Scripts: Edit`)
- Wrong `CF_ACCOUNT_ID`
- Token expired or revoked

Fix: verify `.env` contents and token permissions at dash.cloudflare.com/profile/api-tokens.

### `Project not found [code: 8000007]`

The script auto-creates projects since the `ensure_project` fix. If you still see this:
- Check that the token has `Cloudflare Pages: Edit` permission
- Try deploying again — the first attempt creates the project

### `Project name invalid`

Project names must be lowercase a-z, digits, and hyphens only. Fix:
- Let the script auto-transliterate by omitting `--name`
- Or provide a valid slug manually: `--name my-report`

### 404 after first deploy

Cloudflare DNS propagation takes ~30 seconds on first deploy of a new project. Wait and retry.

### `python-docx` import error

Only needed for `.docx` files. Fix:
```bash
pip install python-docx
```

## Deploy URLs

After deployment, the script shows two URLs:
- **Deploy URL** (`https://<hash>.<name>.pages.dev`) — immutable snapshot of this specific deploy
- **Permanent URL** (`https://<name>.pages.dev`) — always points to the latest deploy

Share the Permanent URL with users.

## .env search order

The script looks for `.env` in these locations (first match wins per key):
1. Same directory as `publish.py` (`scripts/`)
2. `~/.claude/cloudflare-pub/.env`
3. `~/Documents/personal_ai/.env`
4. Current working directory
