---
tags: [vaibcoder, system-check, intake]
project: vaibcoder-library
type: process
updated: 2026-05-30
---

# 🚦 Системный тест / System Check

← [[VAIBCODER-Index]]

**RU:** «Задаётся вопрос — определяем ключевые параметры — и погнали.» 12 вопросов; каждое ДА включает заметки.
**EN:** Run these 12 questions per project; each YES turns notes ON.

| # | Вопрос / Question | ДА → включить / YES → enable |
|---|---|---|
| 1 | Нужна авторизация? / Auth needed? | [[01-Auth]], [[02-Authorization-RLS]] |
| 2 | **Храните ПДн граждан РФ?** / Store RU citizens' PII? | [[03-Legal-152FZ]] + самохостинг Supabase в РФ / self-host in RF |
| 3 | Платежи? / Payments? | [[20-Payments-YooKassa]] |
| 4 | Нужен русский хостинг? / Russian hosting? | [[Deploy-Timeweb]] / [[Deploy-VPS]] |
| 5 | Роли/доступы? / Roles? | [[02-Authorization-RLS]] |
| 6 | Email-уведомления? / Email? | [[11-Email]] |
| 7 | Многоязычность? / i18n? | i18n note |
| 8 | Поиск? / Search? | [[19-Search-Onboarding]] |
| 9 | Тёмная тема? / Dark mode? | [[04-UI-Theme]] |
| 10 | Публичный SEO-трафик? / Public SEO? | [[05-SEO]] |
| 11 | Аналитика? / Analytics? | [[10-Analytics]] |
| 12 | Загрузки/UGC? / Uploads/UGC? | storage RLS + [[30-Security-Checklist]] |

## Всегда включено / Always-on
[[05-SEO]] · [[06-Performance]] · [[07-Accessibility]] · [[09-Errors]] · [[12-Env-Secrets]] · [[30-Security-Checklist]] · [[17-Backups-DR]]

## Порог решения по хостингу / Hosting decision gate
- Q2 = ДА → **обязательно** самохостинг Supabase на VPS в РФ + уведомление РКН.
- Q2 = НЕТ → по умолчанию **Vercel + Supabase Cloud** ради скорости.
- >3 активных проектов → консолидация на один VPS (дешевле). / >3 projects → consolidate on one VPS.
