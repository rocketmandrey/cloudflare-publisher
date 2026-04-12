# Cloudflare Publisher

Скилл для [Claude Code](https://docs.anthropic.com/en/docs/claude-code), который публикует контент на Cloudflare Pages и возвращает постоянную ссылку.

Агент сгенерировал отчёт, таблицу, лендинг или документацию — а поделиться можно только копипастой? Этот скилл решает проблему: одна команда — и результат доступен по ссылке навсегда.

**Что умеет:**
- Принимает `.docx`, `.md`, `.txt`, `.html` и `stdin`
- Автоматически конвертирует в стильную HTML-страницу (light/dark theme)
- Таблицы — адаптивные, с горизонтальным скроллом
- URL в тексте автоматически становятся кликабельными ссылками
- Кириллические имена транслитерируются в URL автоматически
- Результат: постоянная ссылка `https://<name>.pages.dev`

**Без внешних Python-зависимостей.** Один файл, только стандартная библиотека (для `.docx` нужен `python-docx`).

## Установка

Вставьте этот промпт в Claude Code:

> Скачай файлы из репозитория github.com/YOUR_USERNAME/cloudflare-publisher-template: файл `publish.py` сохрани в `~/.claude/cloudflare-pub/publish.py`, а файл `.claude/skills/cloudflare-pub/skill.md` сохрани в `~/.claude/skills/cloudflare-pub/skill.md`. Создай директории, если их нет.

### Настройка Cloudflare

#### 1. Получите Account ID

1. Зайдите на [dash.cloudflare.com](https://dash.cloudflare.com)
2. В левом сайдбаре выберите ваш аккаунт
3. Перейдите в **Workers & Pages** → **Overview**
4. Справа на странице найдите **Account ID** — скопируйте его

#### 2. Создайте API Token

1. Перейдите на [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)
2. Нажмите **Create Token**
3. Выберите **Create Custom Token** (внизу, "Get started")
4. Настройте:

| Поле | Значение |
|------|---------|
| **Token name** | `cloudflare-publisher` (любое удобное имя) |
| **Permissions** | |
| Account → Cloudflare Pages | **Edit** |
| Account → Workers Scripts | **Edit** |
| **Account Resources** | Include → выберите ваш аккаунт (или All accounts) |
| **Zone Resources** | не нужно — оставьте пустым |
| **TTL** | опционально — срок действия токена |

5. Нажмите **Continue to summary** → **Create Token**
6. Скопируйте токен (**он показывается один раз!**)

> **Минимальные scopes:** скрипт использует только `wrangler pages deploy`, поэтому достаточно двух permissions: **Cloudflare Pages: Edit** (создание/обновление проектов) и **Workers Scripts: Edit** (требуется wrangler для деплоя). Не нужны Zone, DNS, R2 или другие права.

#### 3. Сохраните credentials

```bash
# Создайте файл ~/.claude/cloudflare-pub/.env
mkdir -p ~/.claude/cloudflare-pub

cat > ~/.claude/cloudflare-pub/.env << 'EOF'
CF_ACCOUNT_ID=your_account_id_here
CF_API_TOKEN=your_api_token_here
EOF
```

#### 4. Установите wrangler

```bash
npm i -g wrangler
```

#### 5. Проверьте

```bash
python3 ~/.claude/cloudflare-pub/publish.py --help
```

После этого скилл готов к работе.

## Как пользоваться

Просто напишите в Claude Code одну из фраз:

- «опубликуй на cloudflare»
- «задеплой на pages»
- «сделай публичную ссылку»
- «deploy to cloudflare»

Claude сам отформатирует контент и вернёт постоянную ссылку.

### Примеры

**Опубликовать результат анализа:**
> Проанализируй этот CSV и опубликуй отчёт на cloudflare

**Опубликовать документ:**
> Опубликуй report.docx на pages

**Создать лендинг из описания:**
> Сгенерируй лендинг для продукта X и задеплой на cloudflare

**С кастомным именем:**
> Опубликуй файл notes.md на cloudflare с именем weekly-report

## Поддерживаемые форматы

| Формат | Что происходит |
|--------|---------------|
| `.docx` | Парсит заголовки, параграфы, таблицы → HTML |
| `.md` | Парсит markdown-заголовки, таблицы → HTML |
| `.txt` | Парсит текст, tab-таблицы → HTML |
| `.html` | Деплоит как есть, без конвертации |
| `stdin` | Авто-определяет HTML или текст |

## CLI-интерфейс

```bash
# Из файла
python3 ~/.claude/cloudflare-pub/publish.py "report.docx"

# С кастомным именем и заголовком
python3 ~/.claude/cloudflare-pub/publish.py "report.docx" --name ai-report --title "AI Report"

# Готовый HTML
python3 ~/.claude/cloudflare-pub/publish.py "landing.html" --name my-landing

# Из stdin
echo "# Hello World" | python3 ~/.claude/cloudflare-pub/publish.py --stdin --name test

# Только сгенерировать HTML (без деплоя)
python3 ~/.claude/cloudflare-pub/publish.py "file.docx" --html-only
```

| Параметр | Описание |
|----------|---------|
| `file` | Путь к файлу (.docx, .txt, .md, .html) |
| `--stdin` | Читать контент из stdin вместо файла |
| `--name` | Имя проекта (= часть URL). Автоматически из имени файла |
| `--title` | Заголовок HTML-страницы. Автоматически из контента |
| `--html-only` | Сохранить HTML локально, без деплоя |

## Как это работает

```
Файл (.docx / .txt / .md)
  │
  ▼
Парсинг → блоки [(type, content), ...]
  │         type: h1, h2, h3, p, table
  ▼
Рендер → HTML страница
  │       - responsive layout
  │       - light/dark theme
  │       - адаптивные таблицы
  │       - кликабельные ссылки
  ▼
wrangler pages deploy
  │
  ▼
https://<name>.pages.dev  ← постоянная ссылка
```

## Ссылки после деплоя

- **Deploy URL** (`https://abc123.name.pages.dev`) — снэпшот конкретного деплоя, не меняется
- **Permanent URL** (`https://name.pages.dev`) — всегда показывает последнюю версию

Пользователям лучше давать Permanent URL.

## Лимиты Cloudflare Free

- 500 деплоев в месяц
- 20,000 файлов на проект
- 25 MB на файл
- Безлимитный трафик
- Бессрочный хостинг

## Лицензия

MIT
