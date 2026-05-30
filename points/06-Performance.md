---
tags: [performance, cwv, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 6. Производительность / Core Web Vitals

← [[VAIBCODER-Index]] | → [[04-UI-Theme]]

- `next/image` — резервируй размеры (контроль CLS), lazy-loading из коробки.
- `next/font` — self-host шрифтов, меньше layout shift.
- Кеширование, code-splitting, анализ бандла (`@next/bundle-analyzer`).
- Цель — зелёные **LCP / INP / CLS**; меряй Lighthouse + PageSpeed.
- Минимизируй клиентский JS: больше Server Components, `"use client"` точечно.
