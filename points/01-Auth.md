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

## Полный auth-flow / Complete auth flow

AI-ассистенты часто генерируют форму логина, но забывают про остальные экраны и переходы между ними. Вот полный flow — **каждый пункт должен быть реализован**:

**AI assistants often generate a login form but forget the other screens and transitions. Here is the complete flow — every point must be implemented:**

### Экраны / Screens

1. **Login** (вход / sign in)
   - Email + пароль / Email + password
   - Ссылка «Нет аккаунта? Зарегистрируйся» → ведёт на Register / "No account? Sign up" → links to Register
   - Ссылка «Забыли пароль?» → ведёт на Reset / "Forgot password?" → links to Reset
   - OAuth-кнопки (Google, GitHub и т.д.) / OAuth buttons

2. **Register** (регистрация / sign up)
   - Email + пароль (ОДНО поле + show/hide) / Email + password (ONE field + show/hide)
   - Ссылка «Уже есть аккаунт? Войди» → ведёт на Login / "Already have an account? Sign in" → links to Login
   - Согласие на обработку ПДн (отдельная галочка, НЕотмеченная) → [[03-Legal-152FZ]]
   - После отправки → «Проверь почту для подтверждения» / "Check your email to confirm"

3. **Forgot Password / Reset** (сброс пароля)
   - Поле email + кнопка «Отправить ссылку» / Email field + "Send reset link" button
   - `supabase.auth.resetPasswordForEmail(email, { redirectTo })`
   - После отправки → «Ссылка отправлена на почту» (НЕ говори, существует ли аккаунт — утечка информации) / "Link sent to email" (do NOT reveal if account exists)
   - Ссылка «Вернуться ко входу» → Login / "Back to login" → Login

4. **Update Password** (установка нового пароля)
   - Пользователь пришёл по ссылке из email / User arrived via email link
   - ДВА поля: новый пароль + подтверждение, оба скрыты / TWO fields: new password + confirm, both hidden
   - Если не совпадают → ошибка «Пароли не совпадают» / If mismatch → "Passwords don't match" error
   - `supabase.auth.updateUser({ password })` 
   - После успеха → редирект в приложение / After success → redirect to app

5. **Auth Callback** (`app/auth/callback/route.ts`)
   - Обрабатывает code exchange для email-подтверждения, сброса пароля, OAuth
   - Handles code exchange for email confirmation, password reset, OAuth
   - Без этого роута — auth-письма не работают / Without this route — auth emails don't work

### Переходы / Transitions

```
Login ←→ Register        (ссылки туда-обратно / links both ways)
Login  → Forgot Password (ссылка "Забыли пароль?" / "Forgot password?" link)
Forgot → Login           (ссылка "Вернуться" / "Back to login" link)
Email  → Auth Callback   (из письма подтверждения/сброса / from confirmation/reset email)
Callback → App           (после успешной верификации / after successful verification)
```

### Middleware

```ts
// middleware.ts — защита роутов / route protection
const { data: { user } } = await supabase.auth.getUser();

// Не аутентифицирован → редирект на /login
// Not authenticated → redirect to /login
if (!user && isProtectedRoute) return redirect('/login');

// Уже аутентифицирован → не показывай login/register
// Already authenticated → don't show login/register  
if (user && isAuthRoute) return redirect('/dashboard');
```

## UX-паттерн пароля / Password UX
**Кнопка «показать/скрыть» (глаз) — ОБЯЗАТЕЛЬНА на каждом поле пароля: вход, регистрация, сброс, смена.**
**Show/hide toggle (eye icon) — MANDATORY on every password field: login, register, reset, change.**

**Регистрация:** ОДНО поле пароля + показать/скрыть. БЕЗ «повторите пароль».
**Register:** ONE password field + show/hide. NO "confirm password".
Кейс Zuko: удаление confirm-password при регистрации дало **+56,3% конверсии**.

**Вход:** поле пароля + показать/скрыть.
**Login:** password field + show/hide.

**Сброс/смена пароля:** ДВА поля (новый + подтверждение), оба с кнопкой показать/скрыть. Ошибка в пароле = пользователь снова заблокирован.
**Reset/change password:** TWO fields (new + confirm), both with show/hide toggle. Typo = user locked out again.
- Правила пароля показывай **сразу** у поля, не после ошибки / Show password rules inline, not after error
- Валидируй на отправке, не на каждое нажатие → [[08-Forms]] / Validate on submit, not on every keystroke
- Ссылку «Забыли пароль?» — под полем пароля / "Forgot password?" link — below password field
- Защищённые роуты — проверка сессии в middleware / Protected routes — session check in middleware

## Грабли / Gotchas

**Зависимости / Dependencies:**
- `zod` должен быть УСТАНОВЛЕН (`npm i zod`), если используешь zod-схемы для форм. AI часто импортирует zod, но не добавляет в зависимости → страница падает при загрузке.
- `zod` must be INSTALLED (`npm i zod`) if you use zod schemas for forms. AI often imports zod but doesn't add it to dependencies → page crashes on load.
- Проверяй: `@supabase/supabase-js`, `@supabase/ssr`, `react-hook-form`, `@hookform/resolvers` — всё должно быть в package.json / Verify all deps are in package.json

**Email и SMTP:**
- Стандартный SMTP Supabase ≈ 2 письма/час → для прода подключи Resend → [[11-Email]] / Default SMTP = 2 emails/hr → connect Resend for prod
- Без настройки SMTP — письма подтверждения и сброса пароля не дойдут в проде / Without SMTP setup — confirmation and reset emails won't arrive in prod

**Безопасность / Security:**
- Никогда не клади `service_role` в браузер → [[12-Env-Secrets]] / Never put `service_role` in browser
- На странице сброса пароля НЕ сообщай, существует ли аккаунт — это утечка информации / On reset page do NOT reveal if account exists — information leak
- `redirectTo` в resetPasswordForEmail должен вести на твой домен, не на localhost / `redirectTo` must point to your domain, not localhost
