# Cloudflare Publisher

[English](../README.md) | [Chinese / 中文](README_ZH.md)

Превратите любой отчёт, анализ или документ в постоянную публичную ссылку за секунды — без ручного хостинга, без пастебинов, без истекающих URL. Просто скажите «опубликуй это» в [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

**Как это работает:** дайте Claude файл `.docx`, `.md`, `.txt`, `.html` или сгенерированный контент — скилл конвертирует его в стильную HTML-страницу (светлая/тёмная тема, адаптивные таблицы) и деплоит на Cloudflare Pages. Вы получаете постоянную ссылку `https://<name>.pages.dev`.

**Без внешних Python-зависимостей.** Один файл, только стандартная библиотека (`python-docx` нужен для `.docx`).

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
│   └── publish.py              ← Основной скрипт (парсинг → рендер → деплой)
└── references/
    ├── setup.md                ← Настройка аккаунта и токена Cloudflare
    ├── html-features.md        ← Детали стилизации HTML
    └── troubleshooting.md      ← Типичные ошибки и решения
```

## Поддерживаемые форматы

| Формат | Обработка |
|--------|-----------|
| `.docx` | Парсит заголовки, параграфы, таблицы → стильный HTML |
| `.md` | Markdown-заголовки + таблицы → стильный HTML |
| `.txt` | Обычный текст, tab-таблицы → стильный HTML |
| `.html` | Деплоит как есть |
| `stdin` | Авто-определяет HTML или текст |

## Лимиты (Cloudflare Free)

- 500 деплоев в месяц
- 25 МБ на файл
- Безлимитный трафик
- Бессрочный хостинг

## Лицензия

MIT
