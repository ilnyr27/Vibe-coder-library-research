---
tags: [deploy, vercel, point]
project: vaibcoder-library
type: deploy
updated: 2026-05-30
---

# ☁️ Деплой — Vercel + домен

← [[VAIBCODER-Index]] | → [[Deploy-Timeweb]] · [[Deploy-VPS]]

**Когда:** глобальные проекты и всё **без ПДн граждан РФ** → [[03-Legal-152FZ]].

1. Push в GitHub → импорт репозитория в Vercel → авто-сборка Next.js.
2. Интеграция Supabase подставляет ключи автоматически.
3. Домен: Project → Settings → Domains. Либо A-запись/CNAME на Vercel, либо неймсерверы Vercel.
4. SSL выдаётся автоматически.

**Минус для РФ:** периодически проблемы доступа из России → запасной путь [[Deploy-Timeweb]].
