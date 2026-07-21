---
tags: [deploy, homeserver, docker, nginx, self-hosted, point]
project: vaibcoder-library
type: deploy
updated: 2026-07-21
---

# 🏠 Деплой — Домашний сервер

← [[VAIBCODER-Index]] | → [[Deploy-VPS]] · [[Deploy-Timeweb]] · [[Deploy-Vercel]]

**Когда:** основной хост для .ru проектов. Белый IP + Ubuntu + Docker + Nginx.
Это лучший вариант если Vercel нестабилен в РФ → [[Deploy-Vercel]] (раздел РКН).

---

## Инфраструктура

```
Роутер (проброс 80/443) → Ubuntu Server
  └── Nginx (единый вход по домену)
       ├── tracker27.ru → Docker :3003
       ├── poznaisebya27.ru → Docker :3002
       └── supabase.tracker27.ru → Supabase Kong :8001
```

**Требования:**
- Белый статический IP у провайдера (не CGNAT)
- Проброс портов 80/443 на сервер в роутере
- Домен с A-записью → твой IP (НЕ через Cloudflare-прокси)

---

## Схема: 1 проект = изолированный стек

```
~/projects/<name>/
  ├── app/           ← Next.js код
  ├── .env           ← секреты (не в git)
  ├── Dockerfile
  └── docker-compose.yml

~/projects/supabase-<name>/docker/
  └── docker-compose.yml  ← отдельный Supabase-инстанс
```

Каждый проект: **свой Docker-контейнер + свой Supabase + свой nginx-блок + свой SSL.**

---

## Dockerfile для Next.js (standalone)

```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app

# ⚠️ npm зеркало — npmjs.org за Cloudflare, душится РКН в Docker
RUN npm config set registry https://registry.npmmirror.com

COPY package*.json ./
RUN npm ci

# NEXT_PUBLIC_* вшиваются при сборке (не runtime!)
ARG NEXT_PUBLIC_SUPABASE_URL
ARG NEXT_PUBLIC_SUPABASE_ANON_KEY
ENV NEXT_PUBLIC_SUPABASE_URL=$NEXT_PUBLIC_SUPABASE_URL
ENV NEXT_PUBLIC_SUPABASE_ANON_KEY=$NEXT_PUBLIC_SUPABASE_ANON_KEY

COPY . .
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

В `next.config.ts` обязательно:
```ts
const nextConfig = { output: "standalone" };
```

---

## docker-compose.yml (приложение)

```yaml
services:
  app:
    build:
      context: .
      args:
        NEXT_PUBLIC_SUPABASE_URL: ${NEXT_PUBLIC_SUPABASE_URL}
        NEXT_PUBLIC_SUPABASE_ANON_KEY: ${NEXT_PUBLIC_SUPABASE_ANON_KEY}
    ports:
      - "127.0.0.1:3003:3000"   # только localhost, nginx проксирует
    env_file: .env
    restart: unless-stopped
    mem_limit: 512m
```

---

## Nginx блок

```nginx
server {
    server_name myproject.ru;

    location / {
        proxy_pass http://127.0.0.1:3003;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl;
    # SSL добавит certbot
}
server {
    listen 80;
    server_name myproject.ru;
    return 301 https://$host$request_uri;
}
```

---

## SSL — Let's Encrypt в РФ

LE (`acme-v02.api.letsencrypt.org`) интермиттентно душится РКН.
Использовать скрипт-ловец (ждёт пока LE доступен):

```bash
# ~/le-catch.sh domain1 domain2 ...
# Пример:
~/le-catch.sh myproject.ru supabase.myproject.ru
```

> **ZeroSSL** — блокирует .ru домены. Не использовать.
> **Google/BuyPass CA** — недоступны из РФ.

---

## Деплой нового проекта — чеклист

```bash
# На сервере
cd ~/projects/<name>
git clone <repo> app
cp .env.example .env && nano .env    # заполнить SUPABASE_URL, ANON_KEY и др.

# Поднять Docker (образ тянется через зеркало dockerhub.timeweb.cloud)
docker compose up -d --build

# Nginx
sudo nano /etc/nginx/sites-available/<name>
sudo ln -s /etc/nginx/sites-available/<name> /etc/nginx/sites-enabled/
sudo nginx -t && sudo nginx -s reload

# SSL
~/le-catch.sh <domain>
```

---

## Self-hosted Supabase — изоляция по проектам

```bash
# Создать новый Supabase-инстанс
~/new-supabase.sh NAME KONG_PORT KONGS_PORT PG_PORT POOLER_PORT PUBLIC_URL SITE_URL
# Пример:
~/new-supabase.sh tracker 8001 8444 5434 6544 https://supabase.tracker27.ru https://tracker27.ru
```

Каждый проект — **изолированный инстанс** (свои юзеры, свои таблицы, свои JWT_SECRET/ANON_KEY).
Секреты хранятся в `~/SUPABASE-<NAME>-SECRETS.txt`.

### ⚠️ NEXT_PUBLIC_* — вшиваются при сборке!

В Next.js переменные `NEXT_PUBLIC_*` встраиваются в JS-бандл во время `docker build`.
Изменение `.env` без пересборки образа — НЕ обновит URL/ключ в браузере.

```bash
# Обновить переменные → нужна пересборка
docker compose up -d --build
```

### ⚠️ SMTP по умолчанию = fake

Новый Supabase-инстанс использует `supabase-mail` (fake SMTP).
Без настройки реального SMTP:
- `ENABLE_EMAIL_AUTOCONFIRM=false` + `signup` → **500 Internal Server Error**
- Письма о сбросе пароля **не приходят**

Настройка Resend (рекомендуется):
```env
SMTP_HOST=smtp.resend.com
SMTP_PORT=465
SMTP_USER=resend
SMTP_PASS=re_<ключ из resend.com>
SMTP_SENDER_NAME=My App
SMTP_ADMIN_EMAIL=noreply@myproject.ru
ENABLE_EMAIL_AUTOCONFIRM=true   # или false если нужно подтверждение
```

После изменения `.env` — перезапустить auth контейнер:
```bash
cd ~/projects/supabase-<name>/docker && docker compose restart auth
```
