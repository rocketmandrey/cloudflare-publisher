# Cloudflare Publisher

[English](../README.md) | [Russian / Русский](README_RU.md)

将任何报告、分析或文档在几秒钟内转化为永久公开链接 — 无需手动托管、无需粘贴板、无需过期URL。只需在 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 中说「发布这个」。

**工作原理：** 将 `.docx`、`.md`、`.txt`、`.html` 文件或生成的内容交给 Claude — 技能会将其转换为精美的 HTML 页面（明暗主题、响应式表格），并部署到 Cloudflare Pages。您将获得永久链接 `https://<name>.pages.dev`。

**无外部 Python 依赖。** 单文件，仅使用标准库（`.docx` 需要 `python-docx`）。

## 安装

在 Claude Code 中粘贴：

> 从 github.com/rocketmandrey/cloudflare-publisher 下载文件：将 `skills/cloudflare-pub/` 目录复制到 `~/.claude/plugins/local/cloudflare-pub/skills/cloudflare-pub/`。如需要请创建目录。

或手动安装：

```bash
git clone https://github.com/rocketmandrey/cloudflare-publisher.git /tmp/cf-pub
mkdir -p ~/.claude/plugins/local/cloudflare-pub/
cp -r /tmp/cf-pub/skills ~/.claude/plugins/local/cloudflare-pub/
rm -rf /tmp/cf-pub
```

使用插件启动 Claude Code：

```bash
claude --plugin-dir ~/.claude/plugins/local/cloudflare-pub/
```

### 配置 Cloudflare

1. 在 [dash.cloudflare.com](https://dash.cloudflare.com) → Workers & Pages → Overview（右侧面板）获取 **Account ID**
2. 在 [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens) 创建 **API Token**：

| 权限 | 访问级别 |
|------|---------|
| Account → Cloudflare Pages | **Edit** |
| Account → Workers Scripts | **Edit** |

这是最低权限要求。不需要 Zone、DNS 或 R2 权限。

3. 保存凭证：

```bash
mkdir -p ~/.claude/cloudflare-pub
cat > ~/.claude/cloudflare-pub/.env << 'EOF'
CF_ACCOUNT_ID=你的account_id
CF_API_TOKEN=你的api_token
EOF
```

4. 安装 wrangler：`npm i -g wrangler`

详细指南：[references/setup.md](../skills/cloudflare-pub/references/setup.md)

## 使用方法

在 Claude Code 中说：

- "发布到Cloudflare"
- "部署到pages"
- "生成公开链接"
- "把这个部署到网上"

### 示例

**发布分析结果：**
> 分析这个CSV并将报告发布到cloudflare

**发布文档：**
> 将 report.docx 发布到 pages

**生成并发布：**
> 为产品X创建着陆页并部署到cloudflare

## 技能结构

```
skills/cloudflare-pub/
├── SKILL.md                    ← 技能定义（触发器、工作流）
├── scripts/
│   └── publish.py              ← 主脚本（解析 → 渲染 → 部署）
└── references/
    ├── setup.md                ← Cloudflare 账户和令牌设置
    ├── html-features.md        ← 生成的 HTML 样式详情
    └── troubleshooting.md      ← 常见错误和解决方案
```

## 支持的格式

| 格式 | 处理方式 |
|------|---------|
| `.docx` | 解析标题、段落、表格 → 精美 HTML |
| `.md` | Markdown 标题 + 表格 → 精美 HTML |
| `.txt` | 纯文本、制表符表格 → 精美 HTML |
| `.html` | 直接部署 |
| `stdin` | 自动检测 HTML 或文本 |

## 限制（Cloudflare 免费版）

- 每月 500 次部署
- 每文件 25 MB
- 无限流量
- 永久托管

## 许可证

MIT
