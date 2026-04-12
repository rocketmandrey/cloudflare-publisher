# Generated HTML Features

When `publish.py` converts `.docx`, `.md`, or `.txt` input, it produces a styled HTML page with these features.

## Layout

- Max-width 1100px, centered
- System font stack: `-apple-system, BlinkMacSystemFont, Segoe UI, Roboto, sans-serif`
- Line height 1.7 for readability
- Responsive padding (2rem horizontal, 1.5rem on mobile)

## Theming

Automatic light/dark theme based on system `prefers-color-scheme`:

| Element | Light | Dark |
|---------|-------|------|
| Background | `#fff` | `#0f172a` |
| Text | `#1a1a2e` | `#e2e8f0` |
| Accent (links, h2) | `#2563eb` | `#60a5fa` |
| Table header bg | `#f1f5f9` | `#1e293b` |
| Table stripe | `#f8fafc` | `#1e293b` |

## Tables

- Wrapped in `.table-wrap` with `overflow-x: auto` for horizontal scroll on mobile
- Rounded border (`8px`)
- Sticky header row
- Alternating row stripes
- Font size 0.82rem for density

## Typography

- `h1`: 1.8rem, bottom accent border
- `h2`: 1.3rem, accent color
- `h3`: 1.1rem
- Paragraphs: 0.8rem bottom margin
- "Label:" patterns auto-bolded at paragraph start

## Auto-processing

- URLs in text → clickable `<a>` links with `target="_blank"`
- Long dash-separated lists (150+ chars with ` - `) → converted to `<ul>` lists
- Footer with publication timestamp

## HTML files

`.html` input is deployed as-is — no conversion or styling applied.
