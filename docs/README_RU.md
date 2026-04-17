# Cloudflare Publisher

[English](../README.md) | [Chinese / 中文](README_ZH.md)

Превратите любой отчёт, анализ или документ в постоянную публичную ссылку за секунды — без ручного хостинга, без пастебинов, без истекающих URL. Просто скажите «опубликуй это» в [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

**Как это работает:** дайте Claude файл `.docx`, `.md`, `.txt`, `.html` или сгенерированный контент — скилл конвертирует его в стильную HTML-страницу и деплоит на Cloudflare Pages. Вы получаете постоянную ссылку `https://<name>.pages.dev`.

**Две темы, выбираются автоматически:**

- **Editorial (по умолчанию для markdown, если установлен `pandoc`)** — шрифты Fraunces + Inter, тёплая кремовая палитра с терракотовыми акцентами, журнальная типографика, стрелочные буллеты, жёлтый хайлайт под жирным. Полная поддержка GFM (списки, таблицы, ссылки, цитаты, код).
- **Legacy (fallback)** — собственный рендерер, синяя тема со светлым/тёмным авто-переключением. Без внешних зависимостей кроме `python-docx` для `.docx`. Включается, если `pandoc` не найден или передан `--legacy`. `.docx` всегда идёт через эту тему.

## Установка

Вставьте в Claude Code:

> Скачай файлы из github.com/rocketmandrey/cloudflare-publisher: скопируй папку `skills/cloudflare-pub/` в `~/.claude/plugins/local/cloudflare-pub/skills/cloudflare-pub/`. Создай директории если нужно.

Или вручную:

```bash
git clone https://github.com/rocketmandrey/cloudflare-publisher.git /tmp/cf-pub
mkdir -p ~/.claude/plugins/local/cloudflare-pub/
cp -r /tmp/cf-pub/skills ~/.claude/plugins/local/cloudflare-pub/
rm -rf /tmp/cf-pub
```

Запустите Claude Code с плагином:

```bash
claude --plugin-dir ~/.claude/plugins/local/cloudflare-pub/
```

### Настройка Cloudflare

1. Получите **Account ID** на [dash.cloudflare.com](https://dash.cloudflare.com) → Workers & Pages → Overview (правая панель)
2. Создайте **API Token** на [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens):

| Разрешение | Доступ |
|------------|--------|
| Account → Cloudflare Pages | **Edit** |
| Account → Workers Scripts | **Edit** |

Это минимальные scopes. Zone, DNS, R2 не нужны.

3. Сохраните credentials:

```bash
mkdir -p ~/.claude/cloudflare-pub
cat > ~/.claude/cloudflare-pub/.env << 'EOF'
CF_ACCOUNT_ID=ваш_account_id
CF_API_TOKEN=ваш_api_token
EOF
```

4. Установите wrangler: `npm i -g wrangler`
5. *(Рекомендуется для editorial-темы)* установите pandoc: `brew install pandoc` · `apt install pandoc` · `choco install pandoc`. Пропустите, если устраивает legacy-синяя тема.

Подробный гайд: [references/setup.md](../skills/cloudflare-pub/references/setup.md)

## Использование

Просто скажите в Claude Code:

- «опубликуй на cloudflare»
- «задеплой на pages»
- «сделай публичную ссылку»
- «выложи этот отчёт онлайн»

### Примеры

**Опубликовать результат анализа:**
> Проанализируй этот CSV и опубликуй отчёт на cloudflare

**Опубликовать документ:**
> Опубликуй report.docx на pages

**Сгенерировать и опубликовать:**
> Сделай лендинг для продукта X и задеплой на cloudflare

## Структура скилла

```
skills/cloudflare-pub/
├── SKILL.md                    ← Определение скилла (триггеры, workflow)
├── scripts/
│   ├── publish.py              ← Основной скрипт (парсинг → рендер → деплой)
│   ├── pretty.css              ← Стили для editorial-темы
│   └── pretty_template.html    ← pandoc-шаблон для editorial-темы
└── references/
    ├── setup.md                ← Настройка аккаунта и токена Cloudflare
    ├── html-features.md        ← Детали стилизации HTML
    └── troubleshooting.md      ← Типичные ошибки и решения
```

## Поддерживаемые форматы

| Формат | Тема | Обработка |
|--------|------|-----------|
| `.md` / `.txt` | Editorial если pandoc стоит, иначе legacy | Полный GFM (editorial) / заголовки+абзацы+tab-таблицы (legacy) |
| `.docx` | Legacy | Заголовки, абзацы, таблицы через `python-docx` |
| `.html` | — | Деплоится как есть |
| `stdin` | Editorial для текста если pandoc стоит | Авто-определяет HTML или текст |

## Флаги

| Флаг | Назначение |
|------|------------|
| `<file>` | Входной файл (.docx .md .txt .html) |
| `--stdin` | Читать из stdin |
| `--name` | Slug проекта → субдомен |
| `--title` | Заголовок `<title>` |
| `--favicon` | Эмодзи для фавикона, по умолчанию `📄` |
| `--legacy` | Форсить legacy-рендерер / синюю тему |
| `--html-only` | Сохранить HTML локально, не деплоить |

## Лимиты (Cloudflare Free)

- 500 деплоев в месяц
- 25 МБ на файл
- Безлимитный трафик
- Бессрочный хостинг

## Лицензия

MIT
