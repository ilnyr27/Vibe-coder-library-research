---
tags: [storage, supabase, rls, uploads, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 25. Supabase Storage + RLS / File Storage & Access Control

← [[VAIBCODER-Index]] | → [[02-Authorization-RLS]] · [[30-Security-Checklist]]

## Когда нужно / When

Аватары, документы, изображения, пользовательские загрузки — всё, что хранится в S3-совместимом storage Supabase.

## Бакеты / Buckets

| Тип | Доступ | Пример |
|-----|--------|--------|
| **Public** | Читает любой по URL | `avatars`, `og-images` |
| **Private** | Только через RLS | `documents`, `invoices` |

```sql
-- Создание приватного бакета
INSERT INTO storage.buckets (id, name, public)
VALUES ('documents', 'documents', false);
```

## RLS-политики / Policies

```sql
-- Пользователь видит только свои файлы
CREATE POLICY "Users read own files"
ON storage.objects FOR SELECT
USING (
  bucket_id = 'documents'
  AND auth.uid()::text = (storage.foldername(name))[1]
);

-- Пользователь загружает только в свою папку
CREATE POLICY "Users upload to own folder"
ON storage.objects FOR INSERT
WITH CHECK (
  bucket_id = 'documents'
  AND auth.uid()::text = (storage.foldername(name))[1]
);

-- Пользователь удаляет только свои файлы
CREATE POLICY "Users delete own files"
ON storage.objects FOR DELETE
USING (
  bucket_id = 'documents'
  AND auth.uid()::text = (storage.foldername(name))[1]
);
```

Структура папок: `documents/{user_id}/filename.pdf`

## Лимиты на клиенте и сервере / Limits

```ts
// Клиент — проверка перед загрузкой
const MAX_SIZE = 5 * 1024 * 1024; // 5 MB
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf'];

if (file.size > MAX_SIZE) throw new Error('File too large');
if (!ALLOWED_TYPES.includes(file.type)) throw new Error('Invalid file type');
```

```sql
-- Серверная проверка — в RLS или Database Function
-- Supabase storage config (Dashboard → Storage → Settings):
-- File size limit: 5MB
-- Allowed MIME types: image/jpeg, image/png, image/webp, application/pdf
```

## Загрузка / Upload

```ts
const { data, error } = await supabase.storage
  .from('documents')
  .upload(`${userId}/${file.name}`, file, {
    cacheControl: '3600',
    upsert: false,
  });
```

## Чек-лист

- [ ] Пользовательские файлы — в приватном бакете
- [ ] RLS на `storage.objects` — SELECT, INSERT, DELETE
- [ ] Лимит размера файла на клиенте и в Dashboard
- [ ] Allowed MIME types настроены
- [ ] Публичные бакеты — только для некритичных ассетов
