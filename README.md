# 🧰 VAIBCODER — Библиотека Вайбкодера / The Vibe Coder Library

Двуязычная (RU/EN) база знаний для **быстрого и безошибочного запуска сайтов**.
A bilingual knowledge base for shipping production websites fast and error-free.

**Стек / Stack 2026:** Next.js 15 (App Router) · TypeScript · Tailwind CSS v4 · shadcn/ui · Motion · Supabase (Postgres + Auth + RLS).

## Как пользоваться / How to use
1. Открой **[VAIBCODER-Index.md](VAIBCODER-Index.md)** — это хаб со ссылками на всё.
2. На каждый новый проект сначала пройди **[00-system-check.md](00-system-check.md)** — 12 вопросов, которые включают нужное подмножество из 30 пунктов.
3. Файлы — обычный Markdown с Obsidian-wikilinks (`[[...]]`). Работает и как Obsidian-волт, и как GitHub-репозиторий.

## Структура / Structure
```
points/    — 30 главных пунктов любого сайта (заметки 01..20 + сводки)
security/  — чек-лист безопасности перед запуском
deploy/    — Vercel · Timeweb · свой VPS
legal/     — 152-ФЗ: шаблоны согласия и политики
templates/ — готовые boilerplate-репозитории под форк
```

## ⚠️ Главное для РФ / Critical for Russia
С 1 июля 2025 (23-ФЗ) первая запись ПДн гражданина РФ должна быть в БД на территории России → **Supabase Cloud для ПДн недопустим**, нужен самохостинг Supabase на российском VPS. С 1 сентября 2025 (156-ФЗ) согласие на обработку ПДн — **отдельный документ с НЕотмеченной галочкой**. Подробно: [legal/152fz.md](legal/152fz.md).

## Лицензия / License
MIT — правь под себя. / MIT — adapt freely.
