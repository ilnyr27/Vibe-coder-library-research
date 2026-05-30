---
tags: [cache, isr, rollback, deploy, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 26. Кеширование и откат деплоя / Caching & Rollback

← [[VAIBCODER-Index]] | → [[06-Performance]] · [[Deploy-Vercel]]

## Кеширование в Next.js / Caching

### Статические страницы (ISR)

```ts
// app/blog/[slug]/page.tsx
export const revalidate = 3600; // перегенерация каждый час

// Или по событию (on-demand):
import { revalidatePath, revalidateTag } from 'next/cache';

// В server action после обновления поста:
revalidatePath('/blog');
revalidateTag('posts');
```

### fetch() с тегами

```ts
const posts = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'], revalidate: 3600 },
});

// Инвалидация:
revalidateTag('posts'); // сбросит кеш всех fetch с этим тегом
```

### Когда НЕ кешировать

- Данные, зависящие от `auth.uid()` (личные дашборды)
- Данные, меняющиеся чаще, чем раз в минуту
- Страницы с `cookies()` / `headers()` — dynamic по умолчанию

## Откат деплоя / Rollback

### Vercel

- **Instant Rollback**: Dashboard → Deployments → выбери предыдущий → Promote to Production
- Откат — мгновенный, без ребилда
- **Но**: откат не затрагивает БД. Если деплой включал миграцию — нужен `down`-скрипт

### VPS / Timeweb

```bash
# Через PM2 — сохраняй предыдущий билд
pm2 deploy production revert 1

# Или через Docker
docker tag myapp:latest myapp:rollback
# После деплоя, если плохо:
docker stop myapp && docker run myapp:rollback
```

### Миграции и откат

```bash
# Supabase — ручной down-скрипт
supabase migration repair --status reverted <version>

# Prisma
npx prisma migrate resolve --rolled-back <migration_name>
```

**Правило**: если миграция деструктивная (DROP COLUMN, изменение типа) — разбей на 2 деплоя:
1. Деплой кода, который работает и со старой, и с новой схемой
2. Деплой миграции

## Чек-лист

- [ ] ISR / revalidate настроен для публичных страниц
- [ ] revalidatePath/Tag вызывается при мутациях
- [ ] Знаешь, как откатить деплой (Vercel Promote / PM2 revert)
- [ ] Деструктивные миграции — в 2 этапа
- [ ] Есть down-скрипт для критичных миграций
