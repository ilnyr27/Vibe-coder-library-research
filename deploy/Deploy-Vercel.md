---
tags: [deploy, vercel, point]
project: vaibcoder-library
type: deploy
updated: 2026-07-21
---

# ☁️ Деплой — Vercel + домен

← [[VAIBCODER-Index]] | → [[Deploy-Timeweb]] · [[Deploy-VPS]] · [[Deploy-HomeServer]]

**Когда:** глобальные проекты и всё **без ПДн граждан РФ** → [[03-Legal-152FZ]].

1. Push в GitHub → импорт репозитория в Vercel → авто-сборка Next.js.
2. Интеграция Supabase подставляет ключи автоматически.
3. Домен: Project → Settings → Domains. Либо A-запись/CNAME на Vercel, либо неймсерверы Vercel.
4. SSL выдаётся автоматически.

---

## ⚠️ Vercel в России — НЕ РАБОТАЕТ без VPN

Vercel стоит за **Cloudflare** (IP-блок 76.76.21.x / 76.76.22.x).
РКН системно throttle-ит Cloudflare → сайты на Vercel недоступны или очень медленные для пользователей в РФ **без VPN**.

**Проверено на практике (2026-07):** проект `tracker27.ru` был задеплоен на Vercel → пользователь в РФ не мог зайти без AmneziaVPN.

### Симптомы
- `ERR_CONNECTION_TIMED_OUT` при открытии сайта
- Сайт открывается через VPN, но не без него
- Curl с российского IP: `curl -w "%{http_code} %{time_total}s"` → таймаут или `000`

### Что ещё за Cloudflare (тоже не работает напрямую)
| Сервис | Проблема |
|--------|---------|
| Supabase Cloud (`*.supabase.co`) | За Cloudflare → недоступен в РФ |
| Docker Hub (`hub.docker.com`) | `auth.docker.io` за Cloudflare → `docker pull` падает |
| npm registry (`registry.npmjs.org`) | За Cloudflare по IPv4 → `npm ci` в Docker зависает |
| Let's Encrypt (`acme-v02.api.letsencrypt.org`) | За Cloudflare → интермиттентно блокируется |

### Решения
- **Cloudflare**: DNS only (серое облако), НЕ проксировать трафик через CF
- **Supabase**: self-hosted на своём сервере → [[Deploy-HomeServer]]
- **Docker Hub**: зеркало `dockerhub.timeweb.cloud` в `/etc/docker/daemon.json`
- **npm в Docker**: `RUN npm config set registry https://registry.npmmirror.com`
- **Let's Encrypt**: скрипт `~/le-catch.sh` — ловит окно когда РКН не душит

### Вывод
Для .ru проектов с российской аудиторией:
- **Vercel** → только как preview/CI, не прод
- **Прод** → [[Deploy-HomeServer]] (домашний сервер) или [[Deploy-Timeweb]] (VPS в РФ)
