---
name: cloudflare-pub
description: "Publish any content to Cloudflare Pages and return a permanent public URL. Use this skill whenever the user asks to publish, deploy, host, share online, make a public link, опубликовать, задеплоить, выложить на cloudflare, сделать публичную ссылку. Works with reports, landing pages, docs, tables, analyses, or any generated content. Converts .docx/.md/.txt to styled HTML with light/dark theme. Even if the user just says 'share this as a link' or 'put this online' after generating content — this is the right skill. Result: permanent https://<name>.pages.dev link."
---

# Cloudflare Publisher

Publish content to Cloudflare Pages. Returns a permanent `https://<name>.pages.dev` link.

## Prerequisites

Before first deploy in a session, verify once:

```bash
command -v wrangler || echo "MISSING: run npm i -g wrangler"
```

Credentials are loaded automatically from `.env` by the script. If deploy fails with auth error, tell the user:
> Create a `.env` file with `CF_ACCOUNT_ID` and `CF_API_TOKEN` next to the script or in `~/.claude/cloudflare-pub/.env`. See `references/setup.md` for step-by-step guide.

**NEVER print, log, or echo credentials.**

## Workflow

### 1. Prepare content

| User situation | Action |
|----------------|--------|
| Provided a file (.docx, .md, .txt, .html) | Pass path as first argument |
| Asked to publish generated content | Write to temp file first, then pass path |
| Has raw HTML/text ready | Pipe via `--stdin` |

### 2. Choose project name

Name becomes the subdomain (`https://<name>.pages.dev`):
- Lowercase a-z, digits, hyphens only (max 63 chars)
- Auto-derived from filename if `--name` omitted
- Cyrillic auto-transliterated by the script
- Only ask user if truly ambiguous

### 3. Deploy

```bash
# The script lives in this skill's scripts/ directory
SCRIPT="$(dirname "$(readlink -f "$0")")/scripts/publish.py"

# From file
python3 "$SCRIPT" "report.md" --name "weekly-report" --title "Weekly Report"

# From stdin (generated content)
echo "<content>" | python3 "$SCRIPT" --stdin --name "my-page" --title "Title"

# Dry run — generate HTML without deploying
python3 "$SCRIPT" "file.docx" --html-only
```

The script auto-creates the Cloudflare Pages project on first deploy.

### 4. Return result

The script prints two URLs:
- **Deploy URL** (`https://<hash>.<name>.pages.dev`) — immutable snapshot
- **Permanent URL** (`https://<name>.pages.dev`) — always shows latest version

Always give the user the **Permanent URL**.

## Quick CLI reference

| Flag | Purpose |
|------|---------|
| `<file>` | Input file path (.docx, .md, .txt, .html) |
| `--stdin` | Read from stdin instead of file |
| `--name` | Project slug → subdomain |
| `--title` | HTML page title |
| `--html-only` | Generate HTML locally, skip deploy |

## Supported formats

| Format | Handling |
|--------|---------|
| `.docx` | Headings, paragraphs, tables → styled HTML (requires `python-docx`) |
| `.md` / `.txt` | Markdown headings, tab-tables → styled HTML |
| `.html` | Deployed as-is |
| `stdin` | Auto-detects HTML vs text |

## When NOT to use

DNS management, Workers scripts, R2/KV storage, editing deployed sites, domain configuration. This skill only does static Pages deploys.

## References

- `references/setup.md` — Cloudflare account setup, API token creation, required scopes
- `references/html-features.md` — Generated HTML styling, theming, table rendering details
- `references/troubleshooting.md` — Common errors and fixes

## Limits (Cloudflare Free)

500 deploys/month · 25 MB/file · unlimited bandwidth · permanent hosting.
