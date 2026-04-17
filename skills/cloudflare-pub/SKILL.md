---
name: cloudflare-pub
description: "Publish any content to Cloudflare Pages and return a permanent public URL. Use when user asks to publish, deploy, host, share online, make a public link / опубликовать, задеплоить, выложить на cloudflare, сделать публичную ссылку / 发布到Cloudflare, 部署页面, 生成公开链接, 分享到网上. Works with .docx, .md, .txt, .html and generated content. Even if the user says 'share this as a link', 'put this online', 'поделись ссылкой', '生成链接' — use this skill. Do NOT use for DNS, Workers, R2, or domain config — only static page deploys."
license: MIT
compatibility: "Requires wrangler CLI (npm i -g wrangler), Python 3.10+. python-docx only for .docx files. pandoc recommended for markdown — enables the editorial theme (Fraunces + Inter, cream palette); falls back to the legacy blue theme automatically when pandoc is absent."
metadata:
  author: rocketmandrey
  version: 1.1.0
  category: publishing
  tags: [cloudflare, pages, deploy, hosting, typography]
---

# Cloudflare Publisher

Publish content to Cloudflare Pages. Returns a permanent `https://<name>.pages.dev` link.

## Important

- Always verify wrangler is installed before attempting deploy
- Always give the user the **Permanent URL**, not the Deploy URL
- NEVER print, log, or echo .env contents or credentials
- If .docx fails with import error, check `python-docx` is installed before retrying

## Instructions

### Step 1: Verify prerequisites

Before first deploy in a session, verify once:

```bash
command -v wrangler || echo "MISSING: run npm i -g wrangler"
command -v pandoc   || echo "OPTIONAL: brew install pandoc (or apt/choco install pandoc) for the editorial theme"
```

`pandoc` is optional — without it the script silently falls back to the legacy renderer.

Credentials are loaded automatically from `.env` by the script. If deploy fails with auth error, tell the user:

> Create a `.env` file with `CF_ACCOUNT_ID` and `CF_API_TOKEN` next to the script or in `~/.claude/cloudflare-pub/.env`.

Consult `references/setup.md` for step-by-step token creation guide with required scopes.

### Step 2: Prepare content

| User situation | Action |
|----------------|--------|
| Provided a file (.docx, .md, .txt, .html) | Pass path as first argument |
| Asked to publish generated content | Write to temp file first, then pass path |
| Has raw HTML/text ready | Pipe via `--stdin` |

### Step 3: Choose project name

Name becomes the subdomain (`https://<name>.pages.dev`):
- Lowercase a-z, digits, hyphens only (max 63 chars)
- Auto-derived from filename if `--name` omitted
- Cyrillic auto-transliterated by the script
- Only ask user if truly ambiguous

### Step 4: Deploy

```bash
# From file
python3 scripts/publish.py "report.md" --name "weekly-report" --title "Weekly Report" --favicon "📊"

# From stdin (generated content)
echo "content" | python3 scripts/publish.py --stdin --name "my-page" --title "Title"

# Dry run — generate HTML without deploying
python3 scripts/publish.py "file.docx" --html-only
```

The `scripts/publish.py` is located in this skill's directory. The script auto-creates the Cloudflare Pages project on first deploy.

### Step 5: Return result

The script prints two URLs:
- **Deploy URL** (`https://<hash>.<name>.pages.dev`) — immutable snapshot
- **Permanent URL** (`https://<name>.pages.dev`) — always shows latest version

Always give the user the **Permanent URL**.

## Examples

### Example 1: Publish generated analysis (EN)
User says: "Analyze this CSV and publish the results as a link"

1. Generate the analysis as markdown
2. Write to temp file `/tmp/analysis.md`
3. Run `python3 scripts/publish.py /tmp/analysis.md --name "csv-analysis" --title "CSV Analysis"`

Result: User gets `https://csv-analysis.pages.dev` with styled report

### Example 2: Publish a .docx document (RU)
User says: "Опубликуй report.docx на cloudflare"

1. Run `python3 scripts/publish.py "report.docx" --name "report"`

Result: Document converted to styled HTML and deployed

### Example 3: Quick share generated content (ZH)
User says: "把这个分析结果发布成链接" (after generating content in conversation)

1. Take the content just generated
2. Pipe via stdin: `echo "content" | python3 scripts/publish.py --stdin --name "shared-content"`

Result: Instant shareable permanent link

## CLI Reference

| Flag | Purpose |
|------|---------|
| `<file>` | Input file path (.docx, .md, .txt, .html) |
| `--stdin` | Read from stdin instead of file |
| `--name` | Project slug → subdomain |
| `--title` | HTML page title |
| `--favicon` | Emoji for browser tab favicon (e.g. `🎨`) |
| `--legacy` | Force legacy renderer / blue theme instead of editorial pandoc theme |
| `--html-only` | Generate HTML locally, skip deploy |

## Themes

**Editorial (default for markdown when pandoc is present).** Fraunces + Inter, cream `#f6f2ea` / terracotta `#c8502a` palette, accent dot before each `h2`, yellow highlight under `<strong>`, arrow `→` bullets, editorial tables, callout blockquotes. Produced via `pandoc` + `scripts/pretty_template.html` + `scripts/pretty.css`. Supports the full GFM subset — lists, tables, links, blockquotes, inline code, fenced code.

**Legacy (fallback / `--legacy`).** Hand-rolled renderer, plain blue theme, light/dark automatic. Runs with Python stdlib only — no pandoc required. Triggered when pandoc is missing or when `--legacy` is set. `.docx` always uses this renderer (it goes through `python-docx`).

## Supported Formats

| Format | Theme | Handling |
|--------|-------|----------|
| `.md` / `.txt` | Editorial if pandoc present, else legacy | Full GFM (editorial) / heading+paragraph+tab-table (legacy) |
| `.docx` | Legacy | Headings, paragraphs, tables via `python-docx` |
| `.html` | — | Deployed as-is |
| `stdin` | Editorial for text if pandoc present | Auto-detects HTML vs text |

## Common Issues

- `wrangler: command not found` → Run `npm i -g wrangler`
- Auth error `[code: 10000]` → Check `.env` has valid `CF_ACCOUNT_ID` and `CF_API_TOKEN`. Consult `references/setup.md`
- 404 after first deploy → DNS propagation takes ~30s on new projects, wait and retry
- Project name invalid → Uppercase or non-ASCII in `--name`; omit flag to auto-transliterate

Consult `references/troubleshooting.md` for the full troubleshooting guide.

## References

- `references/setup.md` — Cloudflare account setup, API token creation, required scopes
- `references/html-features.md` — Generated HTML styling, theming, table rendering details
- `references/troubleshooting.md` — Common errors and fixes

## Limits (Cloudflare Free)

500 deploys/month · 25 MB/file · unlimited bandwidth · permanent hosting.
