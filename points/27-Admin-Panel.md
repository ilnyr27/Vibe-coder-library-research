---
tags: [admin, security, rbac, mfa, audit, point]
project: vaibcoder-library
type: point
updated: 2026-06-04
---

# 27. Админ-панель / Admin Panel Security

← [[VAIBCODER-Index]] | → [[02-Authorization-RLS]] · [[30-Security-Checklist]] · [[28-AI-Support]]

## Архитектура / Architecture

**Solo dev:** защищённый `/admin` route group внутри того же Next.js приложения. Авторизация — в **data layer** (RLS + серверные проверки роли), НЕ в middleware.
**Solo dev:** protected `/admin` route group inside the same Next.js app. Authorization in the **data layer** (RLS + server-side role checks), NOT in middleware alone.

**Когда выносить отдельно / When to separate:**
- Несколько админов или чувствительные ПДн → отдельный субдомен `admin.example.com` с IP/VPN-ограничением
- Multiple admins or sensitive PII → separate subdomain `admin.example.com` with IP/VPN restriction

**Off-the-shelf варианты:**
- **Refine** (refine.dev) — headless React, Supabase data provider, auth, access control, audit-log (MIT)
- **react-admin** — mature CRUD, official Supabase integration (~130K npm downloads/month)
- **AdminJS** — auto-generates UI from ORM models. **ОПАСНО по умолчанию**: показывает ВСЕ поля включая пароли/токены. Только с explicit allowlist / **DANGEROUS by default**: exposes ALL fields. Use only with explicit allowlists
- **Supabase Studio** — для dev/db ops, НЕ для production admin console

## CVE-2025-29927 — Обход middleware / Middleware Bypass

**CVSS 9.1.** Атакующий обходит middleware через заголовок `x-middleware-subrequest`. Affected: Next.js до 15.2.2.

**Защита / Defense:**
1. Обнови Next.js ≥ **15.2.3** / Patch Next.js ≥ 15.2.3
2. Убери заголовок на Nginx: `proxy_set_header x-middleware-subrequest "";` / Strip header at Nginx
3. **НИКОГДА не полагайся на middleware как единственный слой авторизации** — middleware = UX-редирект, НЕ security boundary / NEVER rely on middleware as the only authz layer

```ts
// middleware.ts — UX gate ONLY, NOT the security boundary
if (req.nextUrl.pathname.startsWith('/admin') && !user) {
  return NextResponse.redirect(new URL('/login', req.url))
}
```

```ts
// app/admin/actions.ts — REAL boundary: server-side role check
'use server'
const { data: { user } } = await supabase.auth.getUser()
if (!user) throw new Error('unauthenticated')

const { data: role } = await supabase.from('user_roles')
  .select('role').eq('user_id', user.id).single()
if (!role || !['superadmin','admin'].includes(role.role)) throw new Error('forbidden')

// verify MFA for sensitive actions
const { data: aal } = await supabase.auth.mfa.getAuthenticatorAssuranceLevel()
if (aal?.currentLevel !== 'aal2') throw new Error('mfa_required')
```

## MFA / 2FA (TOTP)

- Supabase Auth поддерживает TOTP MFA бесплатно / Supabase Auth supports TOTP MFA free on all projects
- `mfa.enroll()` → `mfa.challenge()` → `mfa.verify()` → сессия повышается до **AAL2**
- Enforce AAL2 для админов и в app-логике, и в RLS / Enforce AAL2 for admins in both app logic and RLS
- При верификации MFA — все другие сессии аккаунта завершаются / MFA verification logs out all other sessions

## RBAC — Роли / Roles

Роли: `superadmin` / `admin` / `moderator` / `support`

```sql
create table public.user_roles (
  user_id uuid primary key references auth.users(id) on delete cascade,
  role text not null check (role in ('superadmin','admin','moderator','support')),
  created_at timestamptz not null default now()
);
alter table public.user_roles enable row level security;

-- SECURITY DEFINER helper
create or replace function public.is_admin()
returns boolean language sql security definer stable set search_path = public as $$
  select exists (
    select 1 from public.user_roles
    where user_id = (select auth.uid())
      and role in ('superadmin','admin')
  );
$$;

-- Example: admins read conversations, MFA required
create policy "admins read conversations (MFA required)"
  on public.conversations as restrictive for select to authenticated
  using ( (select is_admin()) and (select auth.jwt()->>'aal') = 'aal2' );
```

**Критические правила / Critical rules:**
- Роли в **`app_metadata`**, НЕ `user_metadata` — users can modify `user_metadata`!
- `(select auth.uid())` вместо `auth.uid()` — до 100x быстрее / up to 100x faster
- `service_role` BYPASSRLS — **ТОЛЬКО на сервере** / server only, NEVER in browser
- `as restrictive` в RLS-политиках с MFA — overlapping permissive policies otherwise bypass it

## Audit Log — Журнал аудита

Append-only: только INSERT, блокировка UPDATE/DELETE.

```sql
create table public.audit_log (
  id bigint generated always as identity primary key,
  actor uuid,
  action text not null,
  table_name text,
  record_id text,
  old_data jsonb,
  new_data jsonb,
  at timestamptz not null default now()
);
alter table public.audit_log enable row level security;

-- Block mutations
create or replace function public.audit_no_mutate() returns trigger
  language plpgsql as $$ begin raise exception 'audit_log is append-only'; end; $$;
create trigger audit_immutable before update or delete on public.audit_log
  for each row execute function public.audit_no_mutate();
```

## Сессии и безопасность / Sessions & Security

- Короткие сессии для админов (Supabase Pro: Time-box + Inactivity timeout) / Short admin sessions
- Sudo mode: `supabase.auth.reauthenticate()` перед деструктивными операциями / before destructive ops
- Soft deletes (`deleted_at`) вместо hard DELETE / instead of hard DELETE
- Telegram-алерт при входе с нового устройства → [[28-AI-Support]] / Telegram alert on new-device login

## OWASP Top 10:2025 — Основные векторы / Main Vectors

| Вектор / Vector | OWASP | Защита / Defense |
|----------------|-------|-----------------|
| Broken access control | **A01** (100% apps affected) | Server-side authz, RLS, deny-by-default |
| Privilege escalation | A01 | Roles in `app_metadata`, re-check server-side |
| Security misconfiguration | **A02** | Security headers, no default creds |
| XSS in admin views | A05 | Escape/sanitize UGC, CSP with nonces |
| Brute-force admin login | A07 | Rate limit, MFA, HaveIBeenPwned |

## Infra Hardening (Timeweb VPS)

- Admin за отдельным Nginx server block / Admin behind its own Nginx block
- Security headers: CSP, HSTS, X-Frame-Options: DENY, nosniff, Referrer-Policy
- `X-Robots-Tag: noindex` + robots.txt disallow для админки / for admin
- fail2ban + firewall (только 443/SSH) / only 443/SSH
- Strip `x-middleware-subrequest` на Nginx / at Nginx
- SSH key-only, закрой все лишние порты / close all unnecessary ports

## Чек-лист админ-панели / Admin Panel Checklist

- [ ] Next.js ≥ 15.2.3 (CVE-2025-29927 patched)
- [ ] `service_role` / secret key — server-only
- [ ] RLS on every public table
- [ ] Role checks server-side on every mutation
- [ ] MFA/AAL2 enforced for admins
- [ ] Audit log append-only
- [ ] Soft deletes
- [ ] Security headers + noindex
- [ ] Rate limit + fail2ban
- [ ] Strip x-middleware-subrequest at Nginx
- [ ] Telegram alert on new-device admin login
