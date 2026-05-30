---
tags: [deploy, vps, nginx, docker, ci, point]
project: vaibcoder-library
type: deploy
updated: 2026-05-30
---

# 🖥️ Деплой — Свой VPS (для новичка)

← [[VAIBCODER-Index]] | → [[17-Backups-DR]] · [[03-Legal-152FZ]]

**Когда:** дёшево держать все проекты на одном сервере; **обязателен** для самохостинга Supabase ради локализации ПДн → [[03-Legal-152FZ]].

## 1. Подготовка (Ubuntu 22.04)
```bash
adduser deploy && usermod -aG sudo deploy      # не работай под root
apt update && apt upgrade -y
apt install -y git curl nginx certbot python3-certbot-nginx
ufw allow OpenSSH && ufw allow 'Nginx Full' && ufw enable
```

## 2. Node + приложение + PM2
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
apt install -y nodejs
git clone <repo> /srv/app && cd /srv/app && npm ci && npm run build
npm i -g pm2
pm2 start npm --name app -- start
pm2 save && pm2 startup systemd
```

## 3. Nginx reverse proxy + SSL
`/etc/nginx/sites-available/app`:
```nginx
server {
  listen 80; server_name example.ru www.example.ru;
  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;
  }
}
```
```bash
ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
certbot --nginx -d example.ru -d www.example.ru --redirect -m you@mail.ru --agree-tos -n
```

## 4. Docker (переносимость Timeweb ↔ свой сервер)
Мульти-стейдж `Dockerfile` с `output: "standalone"` → `docker compose up -d`. Самохостинг Supabase: `git clone --depth 1 https://github.com/supabase/supabase && cd supabase/docker && cp .env.example .env && docker compose up -d`.

## 5. CI/CD, бэкапы, мониторинг
- `.github/workflows/deploy.yml` — push в `main` → build/test → SSH → `git pull && npm ci && npm run build && pm2 reload app` (секреты `VPS_HOST/VPS_USER/VPS_SSH_KEY`).
- Бэкапы: cron `pg_dump` в офсайт → [[17-Backups-DR]].
- Мониторинг: uptime-монитор + `pm2 logs`/`pm2 monit`; алерт при 80% RAM.

## Домены РФ / RU domains
`.ru`/`.рф` на reg.ru, webnames.ru, hoster.ru; A-запись на IP VPS. `.ru` ~189–225 ₽/год (акция), продление дороже. Whois `.ru` для физлица скрыть нельзя.
