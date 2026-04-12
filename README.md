# Cloudflare Publisher

Turn any report, analysis, or document into a permanent public link in seconds — no manual hosting, no pastebins, no expiring URLs. Just say "publish this" in [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

**How it works:** give Claude a `.docx`, `.md`, `.txt`, `.html` file or generated content — the skill converts it to a styled HTML page (light/dark theme, responsive tables) and deploys to Cloudflare Pages. You get back a permanent `https://<name>.pages.dev` link.

**No external Python dependencies.** Single file, stdlib only (`python-docx` needed for `.docx`).

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
├─��� scripts/
│   └── publish.py              ← Main script (parse → render → deploy)
└── references/
    ├── setup.md                ← Cloudflare account & token setup
    ├── html-features.md        ← Generated HTML styling details
    └── troubleshooting.md      ← Common errors and fixes
```

## Supported formats

| Format | Handling |
|--------|---------|
| `.docx` | Parses headings, paragraphs, tables → styled HTML |
| `.md` | Markdown headings + tables → styled HTML |
| `.txt` | Plain text, tab-tables → styled HTML |
| `.html` | Deployed as-is |
| `stdin` | Auto-detects HTML vs text |

## Limits (Cloudflare Free)

- 500 deploys/month
- 25 MB per file
- Unlimited bandwidth
- Permanent hosting

## License

MIT
