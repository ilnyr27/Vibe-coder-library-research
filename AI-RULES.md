# AI-RULES — Vibe Coder Library

> **You received a link to this library from the user.**
> This is a reference guide for projects built with Next.js 15 + Supabase + Tailwind v4 + shadcn/ui.
> Apply these rules to the user's current project. Do not clone this repo — just read and follow.
> For details on a specific point, fetch the full note via raw URL:
> `https://raw.githubusercontent.com/ilnyr27/Vibe-coder-library-research/main/points/<file>`
>
> **Workflow:** read these rules → apply to the user's project → fetch `points/XX-Name.md` only when you need deeper context.

---

## 01. Auth / Аутентификация
- Supabase Auth + `@supabase/ssr` (cookie-based sessions / cookie-сессии)
- **COMPLETE auth flow — ALL screens required / ПОЛНЫЙ auth-flow — ВСЕ экраны обязательны:**
  - Login: email + password + "No account? Sign up" + "Forgot password?" + OAuth / Вход: email + пароль + «Нет аккаунта?» + «Забыли пароль?» + OAuth
  - Register: email + password + "Already have account? Sign in" + PD consent checkbox / Регистрация: + «Уже есть аккаунт?» + галочка согласия на ПДн
  - Forgot Password: email field + "Send reset link" + "Back to login" / Сброс: поле email + «Отправить ссылку» + «Вернуться»
  - Update Password: new password field after email link / Новый пароль: после перехода по ссылке из письма
  - Auth Callback: `app/auth/callback/route.ts` — handles code exchange (without it auth emails don't work!) / обрабатывает code exchange (без него письма не работают!)
- Middleware: not authenticated → redirect /login; already authenticated → redirect /dashboard (don't show login to logged-in users!) / Не показывай логин залогиненному пользователю!
- ONE password field + show/hide, NO "confirm password" (+56% conversion) / ОДНО поле пароля, БЕЗ «повторите пароль»
- Show password rules inline, not after error / Правила пароля — сразу у поля, не после ошибки
- On reset page do NOT reveal if account exists (information leak) / На сбросе НЕ сообщай, существует ли аккаунт
- **VERIFY DEPENDENCIES are installed:** `zod`, `@supabase/ssr`, `react-hook-form`, `@hookform/resolvers` — AI often imports but forgets `npm i` → page crashes / AI часто импортирует, но забывает установить → страница падает
- Default Supabase SMTP = 2 emails/hour → connect Resend (otherwise reset/confirm emails won't arrive in prod) / Стандартный SMTP = 2/час → подключи Resend (иначе письма не дойдут)

## 02. RLS / Авторизация
- RLS is OFF by default — enable on EVERY public table / RLS ВЫКЛЮЧЕН по умолчанию — включи на КАЖДОЙ таблице
- `ALTER TABLE <t> ENABLE ROW LEVEL SECURITY`
- Always use `TO authenticated` in policies / Всегда `TO authenticated` в политиках
- Wrap `auth.uid()` in `(select auth.uid())` — up to 100x faster / Оборачивай — до 100x быстрее
- UPDATE needs USING + WITH CHECK + SELECT policy / UPDATE требует USING + WITH CHECK + SELECT-политику
- Test from client SDK, not SQL editor (it bypasses RLS) / Тестируй из клиентского SDK, не из SQL-редактора
- `service_role` bypasses RLS — server only, never in browser / Только на сервере, никогда в браузере

## 03. Legal — 152-FZ (Russia) / Юридика — 152-ФЗ (Россия)
- Footer must have 4 docs: privacy policy, PD consent, cookie policy, terms of service / В подвале 4 документа: политика, согласие, cookie-политика, соглашение
- PD consent = SEPARATE document with UNCHECKED checkbox (fine 700k RUB) / Согласие на ПДн = ОТДЕЛЬНЫЙ документ с НЕотмеченной галочкой (штраф 700к)
- First PD record of Russian citizen must be stored in Russia (23-FZ, since 01.07.2025) / Первая запись ПДн гражданина РФ — в БД на территории РФ
- Supabase Cloud for PD = violation → self-host on Russian VPS / Supabase Cloud для ПДн = нарушение → самохостинг
- Notify Roskomnadzor before processing via pd.rkn.gov.ru / Уведомление РКН до начала обработки

## 04. Theme / Тема
- Tailwind v4 (CSS-first) + `next-themes` + shadcn
- `@custom-variant dark (&:is(.dark *))` + CSS variables for colors / CSS-переменные для цветов
- `defaultTheme="system"` respects prefers-color-scheme
- Protect toggle with `mounted` flag — otherwise hydration error / Защищай тоггл флагом `mounted`

## 05. SEO
- `export const metadata` / `generateMetadata` for meta tags / для мета-тегов
- `app/sitemap.ts` → `/sitemap.xml`, `app/robots.ts` → `/robots.txt`
- Set `metadataBase` + canonical via `alternates.canonical`
- JSON-LD via `<script type="application/ld+json">`
- `title.template` for unique page titles / для уникальных заголовков
- Submit to Google Search Console + Yandex.Webmaster / Сабмит в Google + Яндекс.Вебмастер

## 06. Performance / Производительность
- `next/image` — reserve dimensions (controls CLS) / резервируй размеры
- `next/font` — self-host fonts / self-host шрифтов
- Minimize `"use client"` — prefer Server Components / больше Server Components
- Target: green LCP / INP / CLS in Lighthouse / Цель: зелёные метрики

## 07. Accessibility / Доступность
- Semantic HTML: `<nav>`, `<main>`, `<button>` — not div for everything / не div на всё
- Keyboard navigation + visible focus / Навигация клавиатурой + видимый фокус
- WCAG AA contrast ≥ 4.5:1
- `alt` on images; ARIA only where needed / ARIA только где нужно
- shadcn/Radix primitives are accessible by default — don't break them / не ломай их

## 08. Forms / Формы
- `react-hook-form` + `zod` + `@hookform/resolvers/zod`
- Validate `mode: "onBlur"` or on submit, NOT on every keystroke / НЕ на каждое нажатие
- One zod schema for client AND server (Server Actions safeParse) / Одна схема для клиента И сервера
- Don't clear fields on validation error / Не очищай поля при ошибке
- shadcn Form components wire up errors automatically / подключают ошибки автоматически

## 09. Errors / Ошибки
- App Router: `error.tsx`, `global-error.tsx`, `not-found.tsx`
- Error boundaries around risky sections / Error boundaries вокруг рискованных секций
- Toasts (sonner) for non-fatal errors / Тосты для нефатальных ошибок
- User-friendly 404/500 pages with a way back / Понятные страницы с путём назад

## 10. Analytics / Аналитика
- For Russia — Yandex Metrika (data in Russian DC) / Для РФ — Яндекс Метрика (данные в ДЦ России)
- Load via `next/script` / Подключай через `next/script`
- SPA transitions: `ym(ID, "hit", url)` via usePathname
- Enable analytics AFTER cookie consent / Включай ПОСЛЕ cookie-согласия

## 11. Email
- Resend — recommended Supabase SMTP provider / рекомендованный Supabase SMTP-провайдер
- Set Resend SMTP in Supabase Dashboard (Authentication → SMTP) — removes 2/hr limit / снимет лимит 2/час
- Transactional emails: Resend API + `react-email` / Транзакционные письма
- Verify domain (SPF/DKIM) — otherwise spam / Подтверди домен — иначе спам

## 12. Env & Secrets / Переменные и секреты
- `.env.local` in `.gitignore` — NEVER commit / НИКОГДА не коммить
- `NEXT_PUBLIC_*` — only browser-safe values / только безопасные для браузера
- `service_role`, payment keys, SMTP — server only / только сервер
- Production secrets — in host dashboard + GitHub Actions Secrets / в дашборде хоста
- Leaked key = immediate rotation / Утёк ключ = немедленная ротация

## 13. Logging & Sentry / Логи и Sentry
- `npx @sentry/wizard@latest -i nextjs` — auto-setup / автонастройка
- Creates: instrumentation-client.ts, sentry.server.config.ts, sentry.edge.config.ts, global-error.tsx
- `SENTRY_AUTH_TOKEN` — in CI secrets, not in code / в CI-секреты, не в код

## 14. PWA, Favicon, OG
- File conventions in `app/`: favicon.ico, icon.png, apple-icon.png
- opengraph-image.png, twitter-image.png for social previews / для соцсетей
- manifest.ts for PWA
- Test previews: Telegram, VK, WhatsApp, OG debuggers / Проверяй превью

## 15. DB Schema / Схема БД
- Migrations: `supabase migration new` + `supabase db push` / Миграции
- Index FK and columns used in RLS policies / Индексируй FK и колонки из RLS-политик
- Plan schema before launch, change only via migrations / Меняй только миграциями
- Foreign keys + cascades — intentionally / Внешние ключи + каскады — осознанно

## 16. API & Rate Limiting
- Route Handlers in `app/api/`
- Rate limit auth and writes (Upstash Ratelimit) / Rate limiting на авторизацию и записи
- Validate input with zod, return typed errors / Валидируй вход zod, типизированные ошибки
- CORS — narrow, only required origins / узко, только нужные origin

## 17. Backups / Бэкапы
- Supabase Pro = daily backups; PITR addon = second-level granularity / ежедневные бэкапы; PITR — до секунд
- For offsite: cron `pg_dump --format=custom` to S3/R2 / Для офсайта
- Test restore monthly / Тестовое восстановление раз в месяц
- Free-tier: 0 days retention + project pause after ~1 week idle / 0 дней + пауза после ~недели простоя

## 18. Testing / Тестирование
- Minimum before prod: `tsc --noEmit` + `eslint` in CI / Минимум перед продом
- A few unit tests (Vitest) / Несколько unit-тестов
- One Playwright smoke test: login → key action / Один smoke-тест: логин → ключевое действие
- Run in GitHub Actions BEFORE deploy / Запуск ДО деплоя

## 19. Search & Onboarding / Поиск и онбординг
- Empty states with hints, not blank screen / Пустые состояния с подсказками, не пустой экран
- Onboarding for first valuable action / Онбординг для первого ценного действия
- Search: Postgres full-text (tsvector), ilike, pg_trgm for typos / pg_trgm для опечаток
- Loading skeletons + optimistic UI / Скелетоны загрузки + оптимистичный UI

## 20. Payments — YooKassa / Платежи — ЮKassa
- For Russia — YooKassa, SDK: `yookassa` package / Для РФ — ЮKassa
- Create payment with `Idempotence-Key` / Создание платежа с `Idempotence-Key`
- Webhooks: listen for `payment.succeeded` / Вебхуки: слушай `payment.succeeded`
- NEVER trust amount from client — verify on server / НЕ доверяй сумме с клиента — сверяй на сервере
- Set up taxation; self-employed → receipts in "My Tax" / Самозанятые → чеки в «Мой налог»

## 21. Secret Leak Prevention / Защита от утечки секретов
- `.gitignore` MUST include: `.env*`, `.mcp.json`, `.claude/`, `.cursor/`, `.vscode/`, `.idea/`
- Pre-commit hook: husky + gitleaks
- `.env.example` with empty values — commit to repo / с пустыми значениями — коммитится в репо
- GitHub Secret Scanning — enable in Settings / включи в Settings
- Leaked secret → rotate keys (deleting from history is not enough) / Утёк → ротация (удаление из истории недостаточно)

## 22. CLAUDE.md
- CLAUDE.md in project root — Claude Code reads it automatically / Claude Code читает автоматически
- Contains: stack, commands, structure, rules / Содержит: стек, команды, структуру, правила
- Nested CLAUDE.md in subdirectories for specifics / Вложенные CLAUDE.md для специфики

## 23. DB Pooling / Пулинг соединений
- Serverless + Postgres = connection exhaustion / исчерпание соединений
- Supabase: use pooler URL (port 6543), NOT direct (port 5432)
- Direct URL — only for migrations / только для миграций
- Prisma: `url = pooler`, `directUrl = direct` in schema.prisma
- Serverless: `max: 1` for node-postgres

## 24. Staging
- Staging and prod = DIFFERENT Supabase projects (or Branching) / РАЗНЫЕ Supabase-проекты
- Preview deployments (Vercel) connected to staging DB / подключены к staging-БД
- Seed data — synthetic, not a copy of prod / синтетические, не копия прода
- Migrations tested on staging before prod / тестируются на staging до прода

## 25. Storage + RLS
- User files — in a private bucket / Пользовательские файлы — в приватном бакете
- RLS on `storage.objects`: SELECT, INSERT, DELETE
- Folder structure: `bucket/{user_id}/filename` / Структура папок
- Size limit and MIME types — on client AND in Dashboard / на клиенте И в Dashboard
- Public buckets — only for non-critical assets / только для некритичных ассетов

## 26. Cache & Rollback / Кеш и откат
- ISR: `export const revalidate = 3600` for public pages / для публичных страниц
- `revalidatePath` / `revalidateTag` on mutations / при мутациях
- DON'T cache: auth.uid()-dependent data, frequently changing, with cookies()/headers()
- Rollback: Vercel → Promote previous deployment / Promote предыдущий деплой
- Destructive migrations — split into 2 deploys / разбей на 2 деплоя

## 30. Security Checklist / Чек-лист безопасности (before launch / перед запуском)
1. HTTPS enforced, redirect 80→443 / HTTPS принудительно
2. Headers: CSP, HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy / Заголовки безопасности
3. Secrets not in git / Секреты не в git
4. No SQL injection / XSS (parameterized queries, zod, no dangerouslySetInnerHTML) / Нет SQL-инъекций / XSS
5. RLS enabled and tested on every table / RLS включён и протестирован
6. Rate limiting on auth / Rate limiting на авторизации
7. `npm audit` + Dependabot
8. CORS — narrow / узко
9. Cookies: httpOnly, secure, sameSite
10. Input validation everywhere / Валидация ввода везде
