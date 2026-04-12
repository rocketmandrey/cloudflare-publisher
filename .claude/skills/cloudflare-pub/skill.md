---
name: cloudflare-pub
description: "Publish content to Cloudflare Pages — docx, markdown, text, HTML. Permanent shareable link. ALWAYS read skill.md before first use."
triggers:
  - "cloudflare"
  - "pages.dev"
  - "задеплой"
  - "deploy"
  - "опубликуй на cloudflare"
  - "publish to cloudflare"
  - "публичную ссылку"
---

# Cloudflare Publisher

Script: `~/.claude/cloudflare-pub/publish.py` (stdlib only, wrangler CLI required).

## STOP — Read Before Acting

- **DO NOT** deploy without `CF_ACCOUNT_ID` and `CF_API_TOKEN` in `.env`
- **DO NOT** run wrangler from a directory that has `wrangler.toml` — use `/tmp` or temp dir
- **DO NOT** use non-latin characters in `--name` — the script transliterates automatically if you omit `--name`
- **DO NOT** forget to install `python-docx` before publishing `.docx` files

## Commands

| Action | Command |
|--------|---------|
| Publish file | `python3 ~/.claude/cloudflare-pub/publish.py "file.docx" --name slug` |
| Publish stdin | `echo "content" \| python3 ~/.claude/cloudflare-pub/publish.py --stdin --name slug` |
| HTML only | `python3 ~/.claude/cloudflare-pub/publish.py "file.md" --html-only` |

## Parameters

| Param | Required | Description |
|-------|----------|-------------|
| `file` | one of two | Path to file (.docx, .txt, .md, .html) |
| `--stdin` | one of two | Read content from stdin |
| `--name` | no | Project name for URL (auto from filename) |
| `--title` | no | HTML page title (auto from content) |
| `--html-only` | no | Save HTML locally, skip deploy |

## Format Detection

Format is auto-detected from file extension:
- `.docx` → parses headings, paragraphs, tables → HTML
- `.md` / `.txt` → parses markdown headings, tab-tables → HTML
- `.html` / `.htm` → deploys as-is, no conversion
- `stdin` → auto-detects HTML (starts with `<!doctype` or `<html>`) or treats as text

## Workflow

When user asks to "publish to cloudflare", "deploy to pages", "задеплой", or "сделай ссылку":

1. Prepare content as a file or generate inline
2. Call the script, capture output
3. Return the **Permanent URL** (`https://<name>.pages.dev`) to the user

### Publish a generated report

```bash
python3 ~/.claude/cloudflare-pub/publish.py /tmp/report.md --name weekly-report --title "Weekly Report"
```

### Publish from stdin (heredoc)

```bash
python3 ~/.claude/cloudflare-pub/publish.py --stdin --name analysis-q1 --title "Q1 Analysis" <<'CONTENT'
# Q1 Results

Revenue grew 15% quarter-over-quarter.

## Key Metrics

| Metric | Q4 | Q1 | Change |
|--------|----|----|--------|
| Revenue | $1.2M | $1.38M | +15% |
| Users | 12,400 | 15,800 | +27% |
CONTENT
```

### Publish existing HTML

```bash
python3 ~/.claude/cloudflare-pub/publish.py landing.html --name product-landing
```

## Generated HTML Features

- Responsive layout, max-width 1100px
- Light/dark theme (follows system `prefers-color-scheme`)
- Tables with horizontal scroll on mobile
- URLs auto-linked
- Bold "Label:" patterns at paragraph start
- Long inline lists (`" - "` separated) converted to `<ul>`
- Publication timestamp in footer

## Output

Plain text on stdout:
```
Parsing: report.docx
  12 blocks
Deploying → weekly-report

  Deploy:    https://abc123.weekly-report.pages.dev
  Permanent: https://weekly-report.pages.dev
```

Always give the user the **Permanent** URL.

## Config

Requires `.env` file with:
```
CF_ACCOUNT_ID=your_account_id
CF_API_TOKEN=your_api_token
```

The script searches for `.env` in order:
1. Same directory as `publish.py`
2. `~/.claude/cloudflare-pub/.env`
3. Current working directory

### Creating the API Token

Go to [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens) → Create Custom Token:

| Permission | Access |
|------------|--------|
| Account → Cloudflare Pages | **Edit** |
| Account → Workers Scripts | **Edit** |

Account Resources: your account. Zone Resources: not needed. These two scopes are the minimum required for `wrangler pages deploy`.

## Limits

- Cloudflare Free: 500 deploys/month, 25MB per file, unlimited bandwidth
- Project name: max 63 characters, lowercase a-z0-9 and hyphens
- `.docx` parsing requires `pip install python-docx`
- No image upload (images must be hosted externally or embedded as base64)
- No auto-split for large content (single page only)
