---
tags: [security, checklist, launch]
project: vaibcoder-library
type: checklist
updated: 2026-05-30
---

# 🔒 Чек-лист безопасности / Security Checklist

← [[VAIBCODER-Index]] | → [[02-Authorization-RLS]] · [[12-Env-Secrets]]

Пройти **перед запуском** / Pass before launch:

1. **HTTPS/SSL принудительно** — Certbot на VPS, авто на Vercel; редирект 80→443.
2. **Заголовки безопасности** — CSP (`frame-ancestors 'none'`), HSTS, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy`, `Permissions-Policy`.
3. **Секреты не в git** — `.env` в `.gitignore`; ротация утёкших → [[12-Env-Secrets]].
4. **SQL-инъекции / XSS** — параметризованные запросы (Supabase/Prisma), zod, без `dangerouslySetInnerHTML` с вводом.
5. **RLS включён и протестирован** на каждой публичной таблице → [[02-Authorization-RLS]].
6. **Rate limiting / антибрутфорс** на авторизации → [[16-API-RateLimit]].
7. **Сканирование зависимостей** — `npm audit`, Dependabot.
8. **CORS** — узко, только нужные origin.
9. **Безопасные cookie** — httpOnly, secure, sameSite (`@supabase/ssr` делает это для auth).
10. **Валидация/санитизация ввода** везде.

```js
// next.config.js
const securityHeaders = [
  { key:'Strict-Transport-Security', value:'max-age=63072000; includeSubDomains; preload' },
  { key:'X-Content-Type-Options', value:'nosniff' },
  { key:'X-Frame-Options', value:'DENY' },
  { key:'Referrer-Policy', value:'strict-origin-when-cross-origin' },
  { key:'Permissions-Policy', value:'camera=(), microphone=(), geolocation=()' },
];
module.exports = { async headers(){ return [{ source:'/:path*', headers: securityHeaders }]; } };
```
