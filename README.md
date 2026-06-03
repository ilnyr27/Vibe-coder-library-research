# Vibe Coder Library / Библиотека Вайбкодера

Open-source knowledge base: **everything you need to check before launching a website.**
Открытая база знаний: **всё, что нужно проверить перед запуском сайта.**

**Stack 2026:** Next.js 15 · TypeScript · Tailwind v4 · shadcn/ui · Supabase · Vercel.

---

## What's inside / Что внутри: 28 points + extras / 28 пунктов + бонусы

### User & Access / Пользователь и доступ

| # | Topic | In one line | Одной строкой |
|---|-------|-------------|--------------|
| 01 | [Auth](points/01-Auth.md) | Registration, login, OAuth, magic link via Supabase Auth | Регистрация, вход, OAuth, magic link через Supabase Auth |
| 02 | [RLS](points/02-Authorization-RLS.md) | Roles and Row Level Security — only owner sees data | Роли и Row Level Security — данные видит только владелец |
| 03 | [152-FZ](points/03-Legal-152FZ.md) | PD consent, privacy policy, Russian data law | Согласие на ПДн, политика конфиденциальности, закон РФ |

### UI & UX / Внешний вид и UX

| # | Topic | In one line | Одной строкой |
|---|-------|-------------|--------------|
| 04 | [Theme](points/04-UI-Theme.md) | Light/dark theme without flash on load | Светлая/тёмная тема без мерцания при загрузке |
| 05 | [SEO](points/05-SEO.md) | sitemap, robots.txt, JSON-LD, meta tags | sitemap, robots.txt, JSON-LD, мета-теги для поиска |
| 06 | [Performance](points/06-Performance.md) | Core Web Vitals: LCP, CLS, INP — keep Google happy | Core Web Vitals: LCP, CLS, INP — чтобы Google не понижал |
| 07 | [Accessibility](points/07-Accessibility.md) | a11y: keyboard, screen reader, contrast, ARIA | a11y: клавиатура, скринридер, контраст, ARIA |
| 08 | [Forms](points/08-Forms.md) | Validation, Zod schemas, server actions, error UX | Валидация, Zod-схемы, серверные экшены, UX ошибок |
| 14 | [PWA & OG](points/14-PWA-Favicon-OG.md) | Icons, manifest, OG images for social sharing | Иконки, manifest, OG-картинки для соцсетей |
| 19 | [Onboarding](points/19-Search-Onboarding.md) | Search, empty states, first launch experience | Поиск, пустые состояния, первый запуск |

### Backend & Data / Бэкенд и данные

| # | Topic | In one line | Одной строкой |
|---|-------|-------------|--------------|
| 09 | [Errors](points/09-Errors.md) | error.tsx, not-found, global catch, fallback | error.tsx, not-found, глобальный перехват, fallback |
| 15 | [DB Schema](points/15-DB-Schema.md) | Tables, migrations, indexes, relations | Таблицы, миграции, индексы, связи |
| 16 | [API](points/16-API-RateLimit.md) | Rate limiting, endpoint protection, CORS | Rate limiting, защита эндпоинтов, CORS |
| 23 | [Pooling](points/23-DB-Pooling.md) | Supavisor/pgBouncer — so serverless won't kill Postgres | Supavisor/pgBouncer — чтобы serverless не убил Postgres |
| 25 | [Storage](points/25-Storage-RLS.md) | File uploads, private buckets, RLS on storage | Загрузка файлов, приватные бакеты, RLS на storage |
| 26 | [Cache & Rollback](points/26-Cache-Rollback.md) | ISR, revalidate, instant deploy rollback | ISR, revalidate, мгновенный rollback деплоя |

### Security & Secrets / Безопасность и секреты

| # | Topic | In one line | Одной строкой |
|---|-------|-------------|--------------|
| 12 | [Env](points/12-Env-Secrets.md) | .env.local, NEXT_PUBLIC, where to store keys | .env.local, NEXT_PUBLIC, где хранить ключи |
| 21 | [Gitleaks](points/21-Gitleaks-PreCommit.md) | Pre-commit hook: secrets won't reach the commit | Pre-commit хук: секрет не попадёт в коммит |
| 27 | [Admin Panel](points/27-Admin-Panel.md) | RBAC, MFA, audit log, CVE-2025-29927 fix | Админ-панель: роли, 2FA, аудит, CVE-2025-29927 |
| 30 | [Checklist](security/30-Security-Checklist.md) | 10 security checks before launch | 10 проверок безопасности перед запуском |

### Monitoring & Analytics / Мониторинг и аналитика

| # | Topic | In one line | Одной строкой |
|---|-------|-------------|--------------|
| 10 | [Analytics](points/10-Analytics.md) | Yandex Metrika, goals, session replay | Яндекс Метрика, цели, вебвизор |
| 13 | [Logging](points/13-Logging-Sentry.md) | Sentry for errors, structured logs | Sentry для ошибок, структурированные логи |

### Communications & Payments / Коммуникации и деньги

| # | Topic | In one line | Одной строкой |
|---|-------|-------------|--------------|
| 11 | [Email](points/11-Email.md) | Transactional emails via Resend | Транзакционные письма через Resend |
| 20 | [Payments](points/20-Payments-YooKassa.md) | YooKassa: accepting payments, webhooks, receipts | ЮKassa: приём оплаты, вебхуки, чеки |

### Infrastructure / Инфраструктура

| # | Topic | In one line | Одной строкой |
|---|-------|-------------|--------------|
| 17 | [Backups](points/17-Backups-DR.md) | Auto DB backups, disaster recovery plan | Автобэкапы БД, план восстановления |
| 18 | [Testing](points/18-Testing.md) | Vitest, Playwright, what to test first | Vitest, Playwright, что тестировать первым |
| 22 | [CLAUDE.md](points/22-CLAUDE-MD.md) | AI context — so Claude understands the project | Контекст для AI — чтобы Claude понимал проект |
| 24 | [Staging](points/24-Staging.md) | Test environment, preview deploys, seed data | Тестовая среда, preview deploys, seed data |
| 28 | [AI Support](points/28-AI-Support.md) | AI chat: YandexGPT, Telegram alerts, prompt design | AI-поддержка: YandexGPT, Telegram, prompt design |

---

## Extras / Бонусы

| Section / Раздел | EN | RU |
|-------------------|----|----|
| [Deploy: Vercel](deploy/Deploy-Vercel.md) | Frontend deploy, env vars, domain | Деплой фронта, переменные окружения, домен |
| [Deploy: Timeweb](deploy/Deploy-Timeweb.md) | Russian hosting for 152-FZ compliance | Российский хостинг для 152-ФЗ |
| [Deploy: VPS](deploy/Deploy-VPS.md) | Own server: Docker, Nginx, SSL, CI/CD | Свой сервер: Docker, Nginx, SSL, CI/CD |
| [Legal templates](legal/) | Privacy policy, PD consent templates | Политика конфиденциальности, согласие на ПДн |
| [.gitignore](templates/gitignore-vaibcoder.md) | Reference template: 11 categories incl. AI configs | Эталонный шаблон: 11 категорий, включая AI-конфиги |
| [Boilerplates](templates/Templates.md) | Ready-to-fork starter repos | Готовые репозитории под форк |
| [Site Audit](templates/Audit-Prompt.md) | Full audit prompt: 10 roles, 80+ checks, copy & paste | Полный аудит сайта: 10 ролей, 80+ проверок, копируй и вставляй |

---

## Quick Start / Быстрый старт

1. Open / Открой **[00-system-check.md](00-system-check.md)** — 12 questions that activate the right points / 12 вопросов, которые включат нужные пункты
2. Go through the list above — each point = one specific task / Пройди по списку — каждый пункт = одна задача
3. Before launch check / Перед запуском проверь **[Security Checklist](security/30-Security-Checklist.md)**

Markdown files. Works as a GitHub repo and as an Obsidian vault.
Файлы Markdown. Работает как GitHub-репозиторий и как Obsidian vault.

---

## For AI Assistants / Для AI-ассистентов

> **You received a link to this library? Read AI-RULES.md — all 26 rules in one file:**
> **Получил ссылку на эту библиотеку? Прочитай AI-RULES.md — все 26 правил в одном файле:**
>
> ```
> https://raw.githubusercontent.com/ilnyr27/Vibe-coder-library-research/main/AI-RULES.md
> ```
>
> For details on a specific point / За деталями по конкретному пункту:
> ```
> https://raw.githubusercontent.com/ilnyr27/Vibe-coder-library-research/main/points/01-Auth.md
> ```
> Replace `01-Auth.md` with the needed file (01–26 + `security/30-Security-Checklist.md`).
>
> **Workflow:** read AI-RULES.md → apply rules → fetch points/ for details.

---

**License / Лицензия:** MIT — use, modify, share freely. / Бери, правь, используй.
