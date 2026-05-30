---
tags: [analytics, metrika, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 10. Аналитика / Analytics

← [[VAIBCODER-Index]] | → [[03-Legal-152FZ]]

**Для РФ — Яндекс Метрика** (данные в ДЦ России, под 152-ФЗ). Vercel Analytics — privacy-friendly для не-РФ.

```tsx
"use client";
import Script from "next/script";
// init: ym(ID, "init", { defer:true, webvisor:true, clickmap:true,
//   trackLinks:true, accurateTrackBounce:true })
// смена роута: ym(ID, "hit", url)  // через usePathname/useSearchParams
```

- Неосновную аналитику включай **после** cookie-согласия → [[03-Legal-152FZ]].
- Webvisor пишет действия пользователя — отрази это в политике.
