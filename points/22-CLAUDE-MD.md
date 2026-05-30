---
tags: [ai, claude, workflow, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 22. Контекст для AI / CLAUDE.md Setup

← [[VAIBCODER-Index]] | → [[00-system-check]]

## Зачем / Why

`CLAUDE.md` в корне проекта — файл, который Claude Code читает автоматически при старте. Без него ИИ каждый раз тратит токены на изучение структуры, стека и правил проекта.

## Минимальная структура / Minimal Structure

```markdown
# Project Name

## Stack
- Next.js 15 (App Router), TypeScript, Tailwind v4, shadcn/ui
- Supabase (Auth + Postgres + RLS + Storage)
- Deploy: Vercel (frontend), Railway (backend)

## Commands
- `npm run dev` — dev server (port 3000)
- `npm run build` — production build
- `npm run lint` — ESLint + Prettier
- `npm run test` — Vitest

## Project Structure
- `src/app/` — pages (App Router)
- `src/components/` — UI components
- `src/lib/` — utilities, Supabase client
- `supabase/migrations/` — SQL migrations

## Rules
- Language: RU for UI, EN for code/comments
- i18n: next-intl, always add both locales
- Never commit .env files
- Use server actions for mutations, not API routes
- RLS on every table — no exceptions
```

## Вложенные CLAUDE.md / Nested Files

В поддиректориях можно размещать дополнительные `CLAUDE.md`:
- `supabase/CLAUDE.md` — правила миграций, RLS-паттерны
- `src/components/CLAUDE.md` — конвенции компонентов

## Связь с Obsidian vault

Глобальные правила для всех проектов — в `CLAUDE.md` пользователя (`~/.claude/CLAUDE.md`).
Онбординг нового проекта — `_vault/02 - Знания/ONBOARDING-Claude.md`.
