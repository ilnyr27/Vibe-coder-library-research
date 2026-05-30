---
tags: [database, schema, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 15. Схема БД / миграции / индексы

← [[VAIBCODER-Index]] | → [[02-Authorization-RLS]] · [[17-Backups-DR]]

- Миграции: `supabase migration new`, `supabase db push` (или Prisma в части проектов).
- **Индексируй** FK и любые колонки в RLS-политиках или частых фильтрах → [[02-Authorization-RLS]].
- Планируй схему до запуска; меняй только миграциями (не руками в проде).
- Внешние ключи + каскады — осознанно.
