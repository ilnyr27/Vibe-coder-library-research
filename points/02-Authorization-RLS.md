---
tags: [rls, security, supabase, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 2. Авторизация / RLS

← [[VAIBCODER-Index]] | → [[01-Auth]] · [[30-Security-Checklist]]

**RU:** RLS **ВЫКЛЮЧЕН по умолчанию**. Любая таблица в `public` без RLS читается любым с anon-ключом — это ровно CVE-2025-48757 (май 2025): 10,3% проверенных Lovable-приложений имели публично читаемые таблицы.
**EN:** RLS is OFF by default; a public table without it is readable by anyone with the anon key.

```sql
ALTER TABLE todos ENABLE ROW LEVEL SECURITY;
CREATE POLICY "select own" ON todos FOR SELECT TO authenticated
  USING ((select auth.uid()) = user_id);
CREATE POLICY "insert own" ON todos FOR INSERT TO authenticated
  WITH CHECK ((select auth.uid()) = user_id);
```

## Best practices
- Всегда указывай `TO authenticated`.
- Оборачивай `auth.uid()` в `(select …)` — до **100× быстрее** на больших таблицах.
- Индексируй колонки из политик → [[15-DB-Schema]].
- UPDATE требует и `USING`, и `WITH CHECK`, и SELECT-политику.
- Тестируй из клиентского SDK (SQL-редактор обходит RLS).
- `service_role` обходит RLS — только сервер.
- Включи проектный тумблер «Enable RLS on new tables».
