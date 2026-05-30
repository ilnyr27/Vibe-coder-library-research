---
tags: [staging, preview, environments, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 24. Staging vs Production / Среды разработки

← [[VAIBCODER-Index]] | → [[Deploy-Vercel]] · [[17-Backups-DR]]

## Зачем / Why

Без staging — тестируешь на проде. Сломанная миграция, случайный seed, debug-логирование в боевой БД — всё это реальные инциденты.

## Минимальная схема / Minimal Setup

```
local (.env.local)  →  staging (preview branch)  →  production (main)
     ↓                        ↓                          ↓
  Supabase local         Supabase staging project    Supabase prod project
  (supabase start)       (или branch)                (основной)
```

### 1. Локальная разработка

```bash
supabase start          # локальный Postgres + Auth + Storage
supabase db reset       # сброс + seed
```

### 2. Staging на Supabase

**Вариант A — отдельный проект** (надёжнее):
- Создай второй проект в Supabase Dashboard
- Отдельные ключи в `.env.staging`
- Миграции применяй: `supabase db push --linked`

**Вариант B — Supabase Branching** (Preview Branches):
- Включи в Dashboard → Settings → Branching
- Каждый PR получает изолированную БД
- Автоматически применяет миграции из `supabase/migrations/`

### 3. Preview Deployments (Vercel)

```json
// vercel.json — переменные для preview
{
  "env": {
    "NEXT_PUBLIC_SUPABASE_URL": "@staging-supabase-url",
    "NEXT_PUBLIC_SUPABASE_ANON_KEY": "@staging-supabase-anon-key"
  }
}
```

Каждый пуш в не-main ветку → preview deploy со staging-БД.

### 4. Seed Data

```sql
-- supabase/seed.sql
INSERT INTO profiles (id, email, role)
VALUES
  ('00000000-0000-0000-0000-000000000001', 'test@example.com', 'admin'),
  ('00000000-0000-0000-0000-000000000002', 'user@example.com', 'user');
```

**Никогда** не используй реальные данные в seed.

## Чек-лист

- [ ] Staging и prod — разные Supabase-проекты (или branches)
- [ ] `.env.local`, `.env.staging`, `.env.production` — разные ключи
- [ ] Миграции тестируются на staging до деплоя в прод
- [ ] Seed data — синтетические, не копия прода
- [ ] Preview deployments подключены к staging-БД
