---
tags: [auth, supabase, point]
project: vaibcoder-library
type: point
updated: 2026-07-19
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
   - Кнопка «показать/скрыть пароль» (Eye icon) — ОБЯЗАТЕЛЬНА / Show/hide toggle — MANDATORY
   - Ссылка «Нет аккаунта? Зарегистрируйся» → ведёт на Register / "No account? Sign up" → links to Register
   - Ссылка «Забыли пароль?» → ведёт на Reset / "Forgot password?" → links to Reset
   - OAuth-кнопки (Google, GitHub и т.д.) / OAuth buttons (optional)

2. **Register** (регистрация / sign up)
   - Email + **ДВА поля пароля**: пароль + подтверждение, **оба** с show/hide / Email + **TWO password fields**: password + confirm, **both** with show/hide
   - Если не совпадают → ошибка «Пароли не совпадают» / If mismatch → "Passwords don't match" error
   - Ссылка «Уже есть аккаунт? Войди» → ведёт на Login / "Already have an account? Sign in" → links to Login
   - Согласие на обработку ПДн (отдельная галочка, НЕотмеченная) → [[03-Legal-152FZ]]
   - После отправки → «Проверь почту для подтверждения» / "Check your email to confirm"
   - Если SMTP не настроен или домен не верифицирован → Supabase вернёт 500 (см. Грабли ниже)

3. **Forgot Password / Reset** (сброс пароля)
   - Поле email + кнопка «Отправить ссылку» / Email field + "Send reset link" button
   - `supabase.auth.resetPasswordForEmail(email, { redirectTo })`
   - После отправки → «Ссылка отправлена на почту» — **НЕ сообщай, существует ли аккаунт** (утечка информации) / "Link sent to email" — do NOT reveal if account exists
   - Ссылка «Вернуться ко входу» → Login / "Back to login" → Login
   - `redirectTo` должен вести на твой домен, не localhost / must point to your domain, not localhost

4. **Update Password** (установка нового пароля — по ссылке из письма)
   - Пользователь пришёл по ссылке из email / User arrived via email link
   - ДВА поля: новый пароль + подтверждение, **оба** с show/hide / TWO fields: new password + confirm, **both** with show/hide
   - Если не совпадают → ошибка «Пароли не совпадают» / If mismatch → "Passwords don't match" error
   - Минимальные требования к паролю — показывай сразу, не после ошибки
   - `supabase.auth.updateUser({ password })`
   - После успеха → редирект в приложение / After success → redirect to app

5. **Auth Callback** (`app/auth/callback/route.ts`) — **без него ничего не работает**
   - Обрабатывает code exchange для email-подтверждения, сброса пароля, OAuth
   - Без этого роута — auth-письма (подтверждение, сброс) не работают
   - Handles code exchange for email confirmation, password reset, OAuth

6. **Delete Account** (удаление аккаунта) — **обязательно по 152-ФЗ** → [[03-Legal-152FZ]]
   - Кнопка в настройках профиля, визуально второстепенная (деструктивное действие)
   - Обязательный confirm-диалог: «Это действие нельзя отменить. Все данные будут удалены.»
   - Маршрут `DELETE /api/user/delete` с service_role (обход RLS)
   - Порядок: сначала удаляем все дочерние таблицы → потом `auth.admin.deleteUser(userId)`
   - **Удалять ВСЕ связанные таблицы** — orphaned rows остаются в БД бессрочно
   - После: sign out + редирект на главную

```ts
// app/api/user/delete/route.ts — ШАБЛОН / TEMPLATE
export async function DELETE() {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return Response.json({ error: "Unauthorized" }, { status: 401 });

  const admin = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!,
    { cookies: { getAll: () => [], setAll: () => {} } }
  );

  // СПИСОК ТАБЛИЦ СОСТАВЛЯЙ ПРИ ПРОЕКТИРОВАНИИ БД — не забудь ни одну
  await Promise.all([
    admin.from("user_plans").delete().eq("user_id", user.id),
    admin.from("chat_sessions").delete().eq("user_id", user.id),   // ← часто забывают
    admin.from("payments").delete().eq("user_id", user.id),
    admin.from("user_reports").delete().eq("user_id", user.id),
    admin.from("test_results").delete().eq("user_id", user.id),
    // добавь все свои таблицы...
  ]);

  const { error } = await admin.auth.admin.deleteUser(user.id);
  if (error) return Response.json({ error: "Failed to delete account" }, { status: 500 });
  return Response.json({ ok: true });
}
```

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
if (!user && isProtectedRoute) return redirect('/login');

// Уже аутентифицирован → не показывай login/register
if (user && isAuthRoute) return redirect('/dashboard');
```

## UX-паттерн пароля / Password UX

**Кнопка «показать/скрыть» (Eye icon) — ОБЯЗАТЕЛЬНА на каждом поле пароля: вход, регистрация, сброс, смена.**
**Show/hide toggle — MANDATORY on every password field: login, register, reset, change.**

| Экран | Полей пароля | show/hide | Подтверждение |
|-------|-------------|-----------|---------------|
| Login | 1 | ✅ | ❌ |
| Register | **2** | ✅ на каждом | ✅ |
| Reset (ввод нового) | **2** | ✅ на каждом | ✅ |

**Правило:** два поля пароля с show/hide — на регистрации И при сбросе пароля. Снижает число «забытых паролей сразу после регистрации».

**Остальные правила:**
- Требования к паролю — показывай сразу рядом с полем, не после ошибки
- Ссылку «Забыли пароль?» — под полем пароля, не в другом месте
- Валидируй форму на submit, не на каждое нажатие → [[08-Forms]]

## Грабли / Gotchas

### 🔥 SMTP + Resend: главная ловушка прода (реальный кейс)

**Симптом:** Пользователи нажимают «Зарегистрироваться» → Supabase возвращает HTTP 500. В логах `supabase-auth`:

```
"error": "gomail: could not send email 1: 550 You can only send testing emails
to your own email address (you@gmail.com). To send emails to other recipients,
please verify a domain at resend.com/domains"
```

**Причина:** Resend без верифицированного домена отправляет письма **только на email владельца Resend-аккаунта**. Любой другой пользователь → 550 → Supabase auth → 500 при регистрации.

**Как диагностировать:**
```bash
docker logs supabase-auth --tail=50 | grep -i "error\|smtp\|550"
```

**Решения:**

1. **Правильное** — верифицировать домен в Resend:
   - resend.com/domains → Add domain
   - Добавить DNS-записи (DKIM TXT, SPF MX + TXT, DMARC TXT) у регистратора
   - Домен `.ru` на REG.RU → reg.ru → Мои домены → DNS
   - После верификации обновить:
     ```bash
     # supabase/docker/.env
     SMTP_ADMIN_EMAIL=noreply@yourdomain.ru
     # приложение .env
     EMAIL_FROM=Проект <noreply@yourdomain.ru>
     # перезапустить auth
     docker compose up -d auth
     ```

2. **Временный обход** (пока домен не верифицирован):
   ```bash
   # supabase/docker/.env
   ENABLE_EMAIL_AUTOCONFIRM=true   # ← пользователи регистрируются без письма
   docker compose up -d auth
   ```
   ⚠️ `AUTOCONFIRM=true` отключает также письмо сброса пароля. Вернуть `false` после верификации.

### Зависимости

- `zod` — AI часто импортирует, но не ставит. Проверяй `package.json`
- Обязательно: `@supabase/supabase-js`, `@supabase/ssr`, `react-hook-form`, `@hookform/resolvers`

### Стандартный SMTP Supabase

≈ 2 письма/час — только для разработки. В проде **обязательно** Resend → [[11-Email]]

### Безопасность / Security

- `service_role` — только на сервере, никогда в браузере → [[12-Env-Secrets]]
- Сброс пароля: не раскрывай, существует ли аккаунт с этим email (information leak)
- Удаление аккаунта: проверяй `getUser()` в роуте, не из куки — защита от CSRF
