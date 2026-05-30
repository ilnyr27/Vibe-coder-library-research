---
tags: [api, ratelimit, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 16. API и rate limiting

← [[VAIBCODER-Index]] | → [[08-Forms]] · [[30-Security-Checklist]]

- Route Handlers в `app/api`.
- Ограничение частоты / защита от брутфорса на авторизацию и записи (Upstash Ratelimit или лимиты Supabase Auth).
- Валидируй вход `zod`; возвращай типизированные ошибки.
- CORS — узко, только нужные origin → [[30-Security-Checklist]].
