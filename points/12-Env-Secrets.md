---
tags: [env, secrets, security, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 12. Переменные и секреты / Env & Secrets

← [[VAIBCODER-Index]] | → [[30-Security-Checklist]]

- `.env.local` в `.gitignore` — **никогда** не коммить.
- `NEXT_PUBLIC_*` — только безопасные для браузера значения.
- `service_role`, ключи платёжек, SMTP — только сервер.
- Секреты прода — в дашборде хоста (Vercel/Timeweb) и в **GitHub Actions Secrets** → [[Deploy-VPS]].
- Утёк ключ — немедленно ротация.
