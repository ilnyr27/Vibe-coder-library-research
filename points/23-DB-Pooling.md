---
tags: [database, pooling, supabase, prisma, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 23. Пулинг соединений к БД / DB Connection Pooling

← [[VAIBCODER-Index]] | → [[15-DB-Schema]] · [[Deploy-Vercel]]

## Проблема / Problem

Serverless-функции (Vercel, Edge) создают новое подключение к Postgres на каждый вызов. PostgreSQL по умолчанию держит ~100 соединений. При всплеске трафика — `too many connections` и 500-ки.

## Решение / Solution

### Supabase (встроенный Supavisor)

Supabase предоставляет два URL:
- **Direct** (`port 5432`) — прямое подключение, для миграций и CLI
- **Pooler** (`port 6543`) — через Supavisor, для приложения

```env
# .env.local
# Для приложения — pooler (transaction mode)
DATABASE_URL=postgresql://postgres.xxx:password@aws-0-eu-central-1.pooler.supabase.com:6543/postgres?pgbouncer=true

# Для миграций — direct
DIRECT_URL=postgresql://postgres.xxx:password@db.xxx.supabase.co:5432/postgres
```

### Prisma

```prisma
// schema.prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")       // pooler
  directUrl = env("DIRECT_URL")         // direct — для migrate
}
```

### Drizzle / raw pg

Используй pooler URL. Для `node-postgres` — задай `max: 1` в serverless:

```ts
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 1, // одно соединение на инвокацию
});
```

## Самохостинг (VPS)

Ставь **pgBouncer** перед Postgres:
- `pool_mode = transaction`
- `max_client_conn = 400`
- `default_pool_size = 20`

## Чек-лист

- [ ] Приложение использует pooler URL, не direct
- [ ] Миграции идут через direct URL
- [ ] В serverless `max` = 1 (или Prisma с pgbouncer=true)
- [ ] Мониторинг: `SELECT count(*) FROM pg_stat_activity`
