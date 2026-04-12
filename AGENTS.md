# Cloudflare Publisher — Agent Rules

## Описание проекта

CLI-инструмент + Claude Code skill для публикации контента на Cloudflare Pages.
Один Python-файл (`publish.py`), stdlib only (кроме `python-docx` для `.docx`), zero pip requirements для основного пайплайна.

**Цель:** Позволить агенту Claude Code публиковать сгенерированный контент (отчёт, лендинг, таблицу, документацию) на Cloudflare Pages одной командой и возвращать постоянную ссылку.

**Контекст:** Пользователи Claude Code генерируют длинные артефакты, но не могут удобно поделиться ими. Скилл решает проблему: одна фраза → постоянная ссылка на `*.pages.dev`.

**Ограничения:**
- Только stdlib Python 3.10+ (кроме `python-docx` для `.docx`)
- Требуется `wrangler` CLI (npm)
- Требуется Cloudflare Account ID + API Token

## Ключевые файлы

- `publish.py` — main CLI script (parse → render HTML → wrangler deploy)
- `.claude/skills/cloudflare-pub/skill.md` — Claude Code skill с триггерами и workflow

## Conventions

- JSON / plain text output на stdout, диагностика на stderr
- Токены хранятся в `.env` (не в коде)
- Slug автоматически транслитерируется из кириллицы
- `.html` файлы деплоятся без конвертации
- wrangler запускается из `/tmp` чтобы избежать конфликтов с `wrangler.toml`
- Deploy URL vs Permanent URL — пользователю даём Permanent

## Универсальные правила

- Следуй существующим паттернам в коде
- Держи документацию краткой: не копируй код, ссылайся на файлы
- Не добавляй pip-зависимости без крайней необходимости

## Чеклист перед завершением

- [ ] Требования выполнены
- [ ] Скрипт работает: `python3 publish.py --help`
- [ ] `.env` не закоммичен
- [ ] Документация обновлена
