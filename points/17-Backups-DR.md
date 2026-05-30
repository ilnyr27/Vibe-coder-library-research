---
tags: [backups, dr, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 17. Бэкапы и восстановление / Backups & DR

← [[VAIBCODER-Index]] | → [[Deploy-VPS]]

- Supabase Pro = ежедневные бэкапы; PITR-аддон — гранулярность до секунд.
- Нативные бэкапы — снапшоты, **не** скачиваемые файлы. Для офсайта/переносимости — cron `pg_dump --format=custom` в S3/R2.
- Проверяй размер дампа и делай **тестовое восстановление раз в месяц**.
- Free-tier: 0 дней хранения + пауза проекта после ~недели простоя.
- Workflow: [[Deploy-VPS]] (`.github/workflows/backup.yml`).
