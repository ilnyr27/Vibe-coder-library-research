---
tags: [auth, supabase, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 1. Аутентификация / Authentication

← [[VAIBCODER-Index]] | → [[02-Authorization-RLS]] · [[11-Email]]

**RU:** Supabase Auth закрывает регистрацию, вход, сброс пароля, верификацию email, OAuth и сессии через `@supabase/ssr` (cookie-based).
**EN:** Supabase Auth handles registration, login, reset, email verification, OAuth, sessions via `@supabase/ssr`.

## UX-паттерн пароля / Password UX
**ОДНО поле пароля + «показать/скрыть». БЕЗ «повторите пароль».**
Кейс Zuko: удаление confirm-password дало **+56,3% конверсии** без роста сбросов. UX Movement: поле подтверждения давало больше четверти отказов при регистрации.
*Single password field with show/hide toggle, NOT confirm-password.*

- Правила пароля показывай **сразу** у поля, не после ошибки.
- Валидируй на отправке, не на каждое нажатие → [[08-Forms]].
- Ссылку «Забыли пароль?» — под полем пароля.
- Защищённые роуты — проверка сессии в middleware.

## Грабли / Gotchas
- Стандартный SMTP Supabase ≈ 2 письма/час → для прода подключи Resend → [[11-Email]].
- Никогда не клади `service_role` в браузер → [[12-Env-Secrets]].
