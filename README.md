# Vibe Coder Library / Библиотека Вайбкодера

Open-source knowledge base: **everything you need to check before launching a website.**
Открытая база знаний: **всё, что нужно проверить перед запуском сайта.**

**Stack 2026:** Next.js 15 · TypeScript · Tailwind v4 · shadcn/ui · Supabase · Vercel.

---

## Что внутри: 26 пунктов + бонусы

### Пользователь и доступ

| # | Тема | Одной строкой |
|---|------|--------------|
| 01 | [Auth](points/01-Auth.md) | Регистрация, вход, OAuth, magic link через Supabase Auth |
| 02 | [RLS](points/02-Authorization-RLS.md) | Роли и Row Level Security — данные видит только владелец |
| 03 | [152-ФЗ](points/03-Legal-152FZ.md) | Согласие на ПДн, политика конфиденциальности, закон РФ |

### Внешний вид и UX

| # | Тема | Одной строкой |
|---|------|--------------|
| 04 | [Тема](points/04-UI-Theme.md) | Светлая/тёмная тема без мерцания при загрузке |
| 05 | [SEO](points/05-SEO.md) | sitemap, robots.txt, JSON-LD, мета-теги для поиска |
| 06 | [Скорость](points/06-Performance.md) | Core Web Vitals: LCP, CLS, INP — чтобы Google не понижал |
| 07 | [Доступность](points/07-Accessibility.md) | a11y: клавиатура, скринридер, контраст, ARIA |
| 08 | [Формы](points/08-Forms.md) | Валидация, Zod-схемы, серверные экшены, UX ошибок |
| 14 | [PWA и OG](points/14-PWA-Favicon-OG.md) | Иконки, manifest, OG-картинки для соцсетей |
| 19 | [Онбординг](points/19-Search-Onboarding.md) | Поиск, пустые состояния, первый запуск |

### Бэкенд и данные

| # | Тема | Одной строкой |
|---|------|--------------|
| 09 | [Ошибки](points/09-Errors.md) | error.tsx, not-found, глобальный перехват, fallback |
| 15 | [Схема БД](points/15-DB-Schema.md) | Таблицы, миграции, индексы, связи |
| 16 | [API](points/16-API-RateLimit.md) | Rate limiting, защита эндпоинтов, CORS |
| 23 | [Пулинг](points/23-DB-Pooling.md) | Supavisor/pgBouncer — чтобы serverless не убил Postgres |
| 25 | [Storage](points/25-Storage-RLS.md) | Загрузка файлов, приватные бакеты, RLS на storage |
| 26 | [Кеш и откат](points/26-Cache-Rollback.md) | ISR, revalidate, мгновенный rollback деплоя |

### Безопасность и секреты

| # | Тема | Одной строкой |
|---|------|--------------|
| 12 | [Env](points/12-Env-Secrets.md) | .env.local, NEXT_PUBLIC, где хранить ключи |
| 21 | [Gitleaks](points/21-Gitleaks-PreCommit.md) | Pre-commit хук: секрет не попадёт в коммит |
| 30 | [Чек-лист](security/30-Security-Checklist.md) | 10 проверок безопасности перед запуском |

### Мониторинг и аналитика

| # | Тема | Одной строкой |
|---|------|--------------|
| 10 | [Аналитика](points/10-Analytics.md) | Яндекс Метрика, цели, вебвизор |
| 13 | [Логи](points/13-Logging-Sentry.md) | Sentry для ошибок, структурированные логи |

### Коммуникации и деньги

| # | Тема | Одной строкой |
|---|------|--------------|
| 11 | [Email](points/11-Email.md) | Транзакционные письма через Resend |
| 20 | [Платежи](points/20-Payments-YooKassa.md) | ЮKassa: приём оплаты, вебхуки, чеки |

### Инфраструктура

| # | Тема | Одной строкой |
|---|------|--------------|
| 17 | [Бэкапы](points/17-Backups-DR.md) | Автобэкапы БД, план восстановления |
| 18 | [Тесты](points/18-Testing.md) | Vitest, Playwright, что тестировать первым |
| 22 | [CLAUDE.md](points/22-CLAUDE-MD.md) | Контекст для AI — чтобы Claude понимал проект |
| 24 | [Staging](points/24-Staging.md) | Тестовая среда, preview deploys, seed data |

---

## Бонусы

| Раздел | Что внутри |
|--------|-----------|
| [Deploy: Vercel](deploy/Deploy-Vercel.md) | Деплой фронта, переменные окружения, домен |
| [Deploy: Timeweb](deploy/Deploy-Timeweb.md) | Российский хостинг для 152-ФЗ |
| [Deploy: VPS](deploy/Deploy-VPS.md) | Свой сервер: Docker, Nginx, SSL, CI/CD |
| [Юр. шаблоны](legal/) | Политика конфиденциальности, согласие на ПДн |
| [.gitignore](templates/gitignore-vaibcoder.md) | Эталонный шаблон: 11 категорий, включая AI-конфиги |
| [Boilerplates](templates/Templates.md) | Готовые репозитории под форк |

---

## Быстрый старт

1. Открой **[00-system-check.md](00-system-check.md)** — 12 вопросов, которые включат нужные пункты
2. Пройди по списку выше — каждый пункт = одна конкретная задача
3. Перед запуском проверь **[Security Checklist](security/30-Security-Checklist.md)**

Файлы — обычный Markdown. Работает как GitHub-репозиторий и как Obsidian vault.

---

## Для AI-ассистентов / For AI Assistants

> **Если пользователь скинул тебе эту ссылку — это справочная библиотека.**
> Не клонируй её. Читай нужные файлы напрямую и применяй рекомендации к текущему проекту пользователя.

**Как читать файлы из этого репозитория:**

```
https://raw.githubusercontent.com/ilnyr27/Vibe-coder-library-research/main/points/01-Auth.md
```

Замени `01-Auth.md` на нужный файл. Полный список:

| Задача | Файл |
|--------|------|
| Аутентификация | `points/01-Auth.md` |
| Row Level Security | `points/02-Authorization-RLS.md` |
| Закон 152-ФЗ, ПДн | `points/03-Legal-152FZ.md` |
| Тёмная/светлая тема | `points/04-UI-Theme.md` |
| SEO, sitemap, meta | `points/05-SEO.md` |
| Core Web Vitals | `points/06-Performance.md` |
| Доступность (a11y) | `points/07-Accessibility.md` |
| Формы и валидация | `points/08-Forms.md` |
| Обработка ошибок | `points/09-Errors.md` |
| Аналитика | `points/10-Analytics.md` |
| Email (Resend) | `points/11-Email.md` |
| Env и секреты | `points/12-Env-Secrets.md` |
| Логи и Sentry | `points/13-Logging-Sentry.md` |
| PWA, favicon, OG | `points/14-PWA-Favicon-OG.md` |
| Схема БД, миграции | `points/15-DB-Schema.md` |
| API, rate limiting | `points/16-API-RateLimit.md` |
| Бэкапы | `points/17-Backups-DR.md` |
| Тестирование | `points/18-Testing.md` |
| Поиск, онбординг | `points/19-Search-Onboarding.md` |
| Платежи (ЮKassa) | `points/20-Payments-YooKassa.md` |
| Защита от утечек | `points/21-Gitleaks-PreCommit.md` |
| CLAUDE.md для AI | `points/22-CLAUDE-MD.md` |
| DB connection pooling | `points/23-DB-Pooling.md` |
| Staging vs Production | `points/24-Staging.md` |
| Supabase Storage + RLS | `points/25-Storage-RLS.md` |
| Кеш и rollback | `points/26-Cache-Rollback.md` |
| Security checklist | `security/30-Security-Checklist.md` |
| Деплой Vercel | `deploy/Deploy-Vercel.md` |
| Деплой Timeweb | `deploy/Deploy-Timeweb.md` |
| Деплой VPS | `deploy/Deploy-VPS.md` |
| .gitignore шаблон | `templates/gitignore-vaibcoder.md` |
| Boilerplate-репо | `templates/Templates.md` |
| Юр. шаблоны РФ | `legal/152fz.md` |

**Порядок работы:** определи, какие пункты относятся к задаче пользователя → прочитай эти файлы через raw URL → применяй как руководство.

---

**Лицензия:** MIT — бери, правь, используй.
