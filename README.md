# Cloudflare Publisher

[Russian / Русский](docs/README_RU.md) | [Chinese / 中文](docs/README_ZH.md)

Turn any report, analysis, or document into a permanent public link in seconds — no manual hosting, no pastebins, no expiring URLs. Just say "publish this" in [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

**How it works:** give Claude a `.docx`, `.md`, `.txt`, `.html` file or generated content — the skill converts it to a styled HTML page and deploys to Cloudflare Pages. You get back a permanent `https://<name>.pages.dev` link.

**Two themes, picked automatically:**

- **Editorial (default for markdown when `pandoc` is installed)** — Fraunces + Inter, warm cream palette with terracotta accents, magazine-style typography, arrow bullets, yellow-highlighted bolds. Full GFM support (lists, tables, links, blockquotes, code).
- **Legacy (fallback)** — hand-rolled renderer, clean blue theme with light/dark auto-switch. No external dependencies beyond `python-docx` for `.docx`. Kicks in when `pandoc` is missing or `--legacy` is passed. `.docx` always uses this theme.

## Installation

Paste this into Claude Code:

> Download files from github.com/rocketmandrey/cloudflare-publisher: copy the entire `skills/cloudflare-pub/` directory to `~/.claude/plugins/local/cloudflare-pub/skills/cloudflare-pub/`. Create directories if needed.

Or clone manually:

```bash
git clone https://github.com/rocketmandrey/cloudflare-publisher.git /tmp/cf-pub
mkdir -p ~/.claude/plugins/local/cloudflare-pub/
cp -r /tmp/cf-pub/skills ~/.claude/plugins/local/cloudflare-pub/
rm -rf /tmp/cf-pub
```

Then launch Claude Code with the plugin:

```bash
claude --plugin-dir ~/.claude/plugins/local/cloudflare-pub/
```

### Cloudflare credentials

1. Get your **Account ID** at [dash.cloudflare.com](https://dash.cloudflare.com) → Workers & Pages → Overview (right panel)
2. Create an **API Token** at [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens):

| Permission | Access |
|------------|--------|
| Account → Cloudflare Pages | **Edit** |
| Account → Workers Scripts | **Edit** |

These two scopes are the minimum required. No Zone, DNS, or R2 permissions needed.

3. Save credentials:

```bash
mkdir -p ~/.claude/cloudflare-pub
cat > ~/.claude/cloudflare-pub/.env << 'EOF'
CF_ACCOUNT_ID=your_account_id
CF_API_TOKEN=your_api_token
EOF
```

4. Install wrangler: `npm i -g wrangler`
5. *(Recommended for the editorial theme)* install pandoc: `brew install pandoc` · `apt install pandoc` · `choco install pandoc`. Skip if you prefer the legacy blue theme.

See [references/setup.md](skills/cloudflare-pub/references/setup.md) for the full step-by-step guide.

## Usage

Just say in Claude Code:

**English:**
- "deploy to cloudflare"
- "share this as a link"
- "publish this report online"

**Русский:**
- «опубликуй на cloudflare»
- «задеплой на pages»
- «сделай публичную ссылку»

**中文:**
- "发布到Cloudflare"
- "生成公开链接"
- "把这个部署到网上"

Claude will format the content and return a permanent link.

### Examples

**EN — Publish analysis results:**
> Analyze this CSV and publish the report to cloudflare

**RU — Опубликовать документ:**
> Опубликуй report.docx на pages

**ZH — 生成并发布:**
> 创建一个产品X的着陆页并部署到cloudflare

## Skill structure

```
skills/cloudflare-pub/
├── SKILL.md                    ← Skill definition (triggers, workflow)
├── scripts/
│   ├── publish.py              ← Main script (parse → render → deploy)
│   ├── pretty.css              ← Editorial theme stylesheet
│   └── pretty_template.html    ← Editorial theme pandoc template
└── references/
    ├── setup.md                ← Cloudflare account & token setup
    ├── html-features.md        ← Generated HTML styling details
    └── troubleshooting.md      ← Common errors and fixes
```

## Supported formats

| Format | Theme | Handling |
|--------|-------|----------|
| `.md` / `.txt` | Editorial if pandoc installed, else legacy | Full GFM (editorial) / heading+paragraph+tab-table (legacy) |
| `.docx` | Legacy | Headings, paragraphs, tables via `python-docx` |
| `.html` | — | Deployed as-is |
| `stdin` | Editorial for text if pandoc installed | Auto-detects HTML vs text |

## Flags

| Flag | Purpose |
|------|---------|
| `<file>` | Input file (.docx .md .txt .html) |
| `--stdin` | Read from stdin |
| `--name` | Project slug → subdomain |
| `--title` | Page `<title>` |
| `--favicon` | Emoji favicon, default `📄` |
| `--legacy` | Force the legacy renderer and blue theme |
| `--html-only` | Save HTML locally, skip deploy |

## Limits (Cloudflare Free)

- 500 deploys/month
- 25 MB per file
- Unlimited bandwidth
- Permanent hosting

## License

MIT
