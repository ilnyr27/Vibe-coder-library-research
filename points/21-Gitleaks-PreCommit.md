---
tags: [security, gitleaks, precommit, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 21. Защита от утечки секретов / Secret Leak Prevention

← [[VAIBCODER-Index]] | → [[12-Env-Secrets]] · [[30-Security-Checklist]]

## Проблема / Problem

`git add .` захватывает всё — `.env`, `.mcp.json`, конфиги IDE. Публичные репозитории сканируются ботами за минуты. Просто удалить файл новым коммитом **недостаточно** — он остаётся в истории.

## Решение / Solution

### 1. `.env.example` — шаблон без значений

```env
# .env.example — коммитится в репо
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
RESEND_API_KEY=
YOOKASSA_SHOP_ID=
YOOKASSA_SECRET_KEY=
```

### 2. Pre-commit хук с gitleaks

```bash
# Установка
npm i -D husky
npx husky init

# Скачай gitleaks: https://github.com/gitleaks/gitleaks/releases
# Положи в PATH или в ./bin/

# .husky/pre-commit
gitleaks protect --staged --verbose
```

Каждый `git commit` автоматически проверяет staged-файлы на токены, ключи, пароли.

### 3. GitHub Secret Scanning

- **Settings → Code security → Secret scanning** — включи для репо.
- GitHub сам уведомит, если в коммите найден известный формат ключа (AWS, Stripe, Supabase и т.д.).

### 4. `.gitignore` — минимум

```gitignore
.env
.env.local
.env*.local
.mcp.json
.vscode/
```

## Если секрет уже утёк / If a secret leaked

1. **Немедленно ротируй ключ** в соответствующем сервисе.
2. Удали файл из истории (`git filter-repo`) или пересоздай репо, если 1–2 коммита.
3. Включи Secret Scanning, чтобы это не повторилось.
