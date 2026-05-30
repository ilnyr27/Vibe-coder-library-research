---
tags: [email, resend, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 11. Email

← [[VAIBCODER-Index]] | → [[01-Auth]]

**Resend** — рекомендованный Supabase провайдер.

- Вставь SMTP-данные Resend в Supabase (Authentication → SMTP) — снимет лимит ~2/час для писем авторизации.
- Транзакционные письма — Resend API (или Edge Functions) + `react-email`.
- Подтверди домен отправки (SPF/DKIM) — иначе спам.
