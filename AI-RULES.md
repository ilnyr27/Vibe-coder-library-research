# AI-RULES — Vibe Coder Library

> **You received a link to this library from the user.**
> This is a reference guide for projects built with Next.js 15 + Supabase + Tailwind v4 + shadcn/ui.
> Apply these rules to the user's current project. Do not clone this repo — just read and follow.
> For details on a specific point, fetch the full note via raw URL:
> `https://raw.githubusercontent.com/ilnyr27/Vibe-coder-library-research/main/points/<file>`
>
> **Workflow:** read these rules → apply to the user's project → fetch `points/XX-Name.md` only when you need deeper context.

---

## 01. Auth
- Supabase Auth + `@supabase/ssr` (cookie-based сессии)
- ОДНО поле пароля + «показать/скрыть», БЕЗ «повторите пароль» (+56% конверсии)
- Правила пароля — сразу у поля, не после ошибки
- Защищённые роуты — проверка сессии в middleware
- Стандартный SMTP Supabase = 2 письма/час → подключи Resend

## 02. RLS
- RLS ВЫКЛЮЧЕН по умолчанию — включи на КАЖДОЙ таблице в public
- `ALTER TABLE <t> ENABLE ROW LEVEL SECURITY`
- Всегда `TO authenticated` в политиках
- Оборачивай `auth.uid()` в `(select auth.uid())` — до 100x быстрее
- UPDATE требует USING + WITH CHECK + SELECT-политику
- Тестируй из клиентского SDK, не из SQL-редактора (он обходит RLS)
- `service_role` обходит RLS — только на сервере, никогда в браузере

## 03. 152-ФЗ (Россия)
- В подвале 4 документа: политика, согласие на ПДн, cookie-политика, пользовательское соглашение
- Согласие на ПДн = ОТДЕЛЬНЫЙ документ с НЕотмеченной галочкой (156-ФЗ, штраф 700к)
- Первая запись ПДн гражданина РФ — в БД на территории РФ (23-ФЗ с 01.07.2025)
- Supabase Cloud для ПДн = нарушение → самохостинг на российском VPS
- Уведомление РКН до начала обработки через pd.rkn.gov.ru

## 04. Тема
- Tailwind v4 (CSS-first) + `next-themes` + shadcn
- `@custom-variant dark (&:is(.dark *))` + CSS-переменные для цветов
- `defaultTheme="system"` уважает prefers-color-scheme
- Тоггл защищай флагом `mounted` — иначе ошибка гидрации

## 05. SEO
- `export const metadata` / `generateMetadata` для мета-тегов
- `app/sitemap.ts` → `/sitemap.xml`, `app/robots.ts` → `/robots.txt`
- `metadataBase` + canonical через `alternates.canonical`
- JSON-LD через `<script type="application/ld+json">`
- `title.template` для уникальных заголовков
- Сабмит: Google Search Console + Яндекс.Вебмастер

## 06. Производительность
- `next/image` с резервированием размеров (контроль CLS)
- `next/font` — self-host шрифтов
- Минимизируй `"use client"` — больше Server Components
- Цель: зелёные LCP / INP / CLS в Lighthouse

## 07. Доступность
- Семантический HTML: `<nav>`, `<main>`, `<button>` — не div на всё
- Навигация клавиатурой + видимый фокус
- Контраст WCAG AA ≥ 4.5:1
- `alt` у изображений; ARIA только где нужно
- Примитивы shadcn/Radix доступны по умолчанию — не ломай

## 08. Формы
- `react-hook-form` + `zod` + `@hookform/resolvers/zod`
- Валидация `mode: "onBlur"` или на отправке, НЕ на каждое нажатие
- Одна zod-схема для клиента И сервера (Server Actions safeParse)
- Не очищай поля при ошибке валидации
- Компоненты Form shadcn подключают ошибки автоматически

## 09. Ошибки
- `error.tsx`, `global-error.tsx`, `not-found.tsx` в App Router
- Error boundaries вокруг рискованных секций
- Тосты (sonner) для нефатальных ошибок
- Понятные 404/500 с путём назад

## 10. Аналитика
- Для РФ — Яндекс Метрика (данные в ДЦ России)
- Подключай через `next/script`
- SPA-переходы: `ym(ID, "hit", url)` через usePathname
- Аналитику включай ПОСЛЕ cookie-согласия

## 11. Email
- Resend — рекомендованный Supabase SMTP-провайдер
- Вставь SMTP Resend в Supabase Dashboard (Authentication → SMTP) — снимет лимит 2/час
- Транзакционные письма: Resend API + `react-email`
- Подтверди домен (SPF/DKIM) — иначе спам

## 12. Env и секреты
- `.env.local` в `.gitignore` — НИКОГДА не коммить
- `NEXT_PUBLIC_*` — только безопасные для браузера значения
- `service_role`, ключи платёжек, SMTP — только сервер
- Секреты прода — в дашборде хоста + GitHub Actions Secrets
- Утёк ключ = немедленная ротация

## 13. Логи и Sentry
- `npx @sentry/wizard@latest -i nextjs` — автонастройка
- Создаёт: instrumentation-client.ts, sentry.server.config.ts, sentry.edge.config.ts, global-error.tsx
- `SENTRY_AUTH_TOKEN` — в CI-секреты, не в код

## 14. PWA, favicon, OG
- Файловые конвенции в `app/`: favicon.ico, icon.png, apple-icon.png
- opengraph-image.png, twitter-image.png для соцсетей
- manifest.ts для PWA
- Проверяй превью: Telegram, VK, WhatsApp, OG-дебаггеры

## 15. Схема БД
- Миграции: `supabase migration new` + `supabase db push`
- Индексируй FK и колонки из RLS-политик
- Планируй схему до запуска, меняй только миграциями
- Внешние ключи + каскады — осознанно

## 16. API и rate limiting
- Route Handlers в `app/api/`
- Rate limiting на авторизацию и записи (Upstash Ratelimit)
- Валидируй вход zod, возвращай типизированные ошибки
- CORS — узко, только нужные origin

## 17. Бэкапы
- Supabase Pro = ежедневные бэкапы; PITR — до секунд
- Для офсайта: cron `pg_dump --format=custom` в S3/R2
- Тестовое восстановление раз в месяц
- Free-tier: 0 дней хранения + пауза после ~недели простоя

## 18. Тестирование
- Минимум перед продом: `tsc --noEmit` + `eslint` в CI
- Несколько unit-тестов (Vitest)
- Один Playwright smoke-тест: логин → ключевое действие
- Запуск в GitHub Actions ДО деплоя

## 19. Поиск и онбординг
- Пустые состояния с подсказками, не пустой экран
- Онбординг для первого ценного действия
- Поиск: Postgres full-text (tsvector), ilike, pg_trgm для опечаток
- Скелетоны загрузки + оптимистичный UI

## 20. Платежи (ЮKassa)
- Для РФ — ЮKassa, SDK: пакет `yookassa`
- Создание платежа с `Idempotence-Key`
- Вебхуки: слушай `payment.succeeded`
- НЕ доверяй сумме с клиента — сверяй на сервере
- Настрой налогообложение; самозанятые → чеки в «Мой налог»

## 21. Защита от утечки секретов
- `.gitignore` ОБЯЗАТЕЛЬНО: `.env*`, `.mcp.json`, `.claude/`, `.cursor/`, `.vscode/`, `.idea/`
- Pre-commit хук: husky + gitleaks
- `.env.example` с пустыми значениями — коммитится в репо
- GitHub Secret Scanning — включи в Settings
- Утёк секрет → ротация ключей (удаление из истории недостаточно)

## 22. CLAUDE.md
- CLAUDE.md в корне проекта — Claude Code читает автоматически
- Содержит: стек, команды, структуру, правила
- Вложенные CLAUDE.md в поддиректориях для специфики

## 23. DB Pooling
- Serverless + Postgres = исчерпание соединений
- Supabase: используй pooler URL (port 6543), НЕ direct (port 5432)
- Direct URL — только для миграций
- Prisma: `url = pooler`, `directUrl = direct` в schema.prisma
- Serverless: `max: 1` для node-postgres

## 24. Staging
- Staging и prod — РАЗНЫЕ Supabase-проекты (или Branching)
- Preview deployments (Vercel) подключены к staging-БД
- Seed data — синтетические, не копия прода
- Миграции тестируются на staging до прода

## 25. Storage + RLS
- Пользовательские файлы — в приватном бакете
- RLS на `storage.objects`: SELECT, INSERT, DELETE
- Структура папок: `bucket/{user_id}/filename`
- Лимит размера и MIME types — на клиенте И в Dashboard
- Публичные бакеты — только для некритичных ассетов

## 26. Кеш и откат
- ISR: `export const revalidate = 3600` для публичных страниц
- `revalidatePath` / `revalidateTag` при мутациях
- НЕ кешируй: данные по auth.uid(), часто меняющиеся, с cookies()/headers()
- Rollback: Vercel → Promote предыдущий деплой
- Деструктивные миграции — разбей на 2 деплоя

## 30. Security Checklist (перед запуском)
1. HTTPS принудительно, редирект 80→443
2. Заголовки: CSP, HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy
3. Секреты не в git
4. Нет SQL-инъекций / XSS (параметризованные запросы, zod, без dangerouslySetInnerHTML)
5. RLS включён и протестирован на каждой таблице
6. Rate limiting на авторизации
7. `npm audit` + Dependabot
8. CORS — узко
9. Cookie: httpOnly, secure, sameSite
10. Валидация ввода везде
