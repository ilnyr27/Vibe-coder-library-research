---
tags: [security, gitleaks, precommit, gitignore, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 21. Защита от утечки секретов / Secret Leak Prevention

← [[VAIBCODER-Index]] | → [[12-Env-Secrets]] · [[30-Security-Checklist]]

## Проблема / Problem

`git add .` захватывает всё — `.env`, `.mcp.json`, конфиги IDE. Публичные репозитории сканируются ботами за минуты. Просто удалить файл новым коммитом **недостаточно** — он остаётся в истории.

**Реальный пример:** при создании этой библиотеки `git add .` затянул `.mcp.json` (конфиг MCP-серверов) и `.vscode/` в публичный репозиторий. Ключей внутри не было — повезло. Но если бы в `.mcp.json` лежал GitHub PAT или Supabase service_role — доступ к репозиториям и БД был бы скомпрометирован за минуты.

## Три уровня защиты / Three Layers of Defense

### Уровень 1: `.gitignore` — не дай файлу попасть в staging

Полный эталонный `.gitignore` с 11 категориями угроз: → [[gitignore-vaibcoder]]

**Особое внимание — AI-инструменты** (новая категория 2025–2026):

| Файл/папка | Инструмент | Что может утечь |
|------------|-----------|-----------------|
| `.mcp.json` | Claude Code / MCP | API-ключи серверов, токены GitHub, URL |
| `.claude/` | Claude Code | память, история чатов, настройки |
| `.cursor/` | Cursor | API-ключи, промпты, правила |
| `.aider*` | Aider | конфиг с ключами OpenAI/Anthropic |
| `.continue/` | Continue.dev | конфиг с провайдерами и ключами |
| `.codeium/` | Codeium | токен авторизации |
| `.codex/` | OpenAI Codex CLI | настройки, инструкции |
| `.bolt/` | Bolt.new | проектные конфиги |
| `.v0/` | Vercel v0 | проектные конфиги |
| `.windsurf/` | Windsurf | правила, память |

Ни один стандартный шаблон `.gitignore` (GitHub, gitignore.io) эти файлы **не включает**. Добавляй вручную.

### Уровень 2: Pre-commit хук — поймай секрет до коммита

```bash
# Установка
npm i -D husky
npx husky init

# Скачай gitleaks: https://github.com/gitleaks/gitleaks/releases
# Положи в PATH или в ./bin/

# .husky/pre-commit
gitleaks protect --staged --verbose
```

Каждый `git commit` автоматически сканирует staged-файлы на паттерны токенов, ключей, паролей. Даже если файл не в `.gitignore` — хук его поймает.

### Уровень 3: GitHub Secret Scanning — последний рубеж

- **Settings → Code security → Secret scanning** — включи для репо.
- GitHub уведомит, если в коммите найден известный формат ключа (AWS, Stripe, Supabase, GitHub PAT и т.д.).
- Работает и на приватных репозиториях (GitHub Advanced Security).

## `.env.example` — шаблон без значений

```env
# .env.example — коммитится в репо, показывает какие переменные нужны
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
RESEND_API_KEY=
YOOKASSA_SHOP_ID=
YOOKASSA_SECRET_KEY=
```

## Если секрет уже утёк / If a secret leaked

1. **Немедленно ротируй ключ** в соответствующем сервисе — это единственная надёжная мера.
2. Удали файл из истории (`git filter-repo`) или пересоздай репо, если 1–2 коммита.
3. Включи Secret Scanning, чтобы это не повторилось.
4. Проверь логи доступа в сервисе (Supabase Dashboard → Logs, GitHub → Security log).
