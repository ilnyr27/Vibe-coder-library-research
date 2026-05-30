---
tags: [seo, nextjs, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 5. SEO

← [[VAIBCODER-Index]] | → [[10-Analytics]] · [[14-PWA-Favicon-OG]]

Metadata API: статический `export const metadata` или `generateMetadata` (динамика). Файловые конвенции `app/sitemap.ts` → `/sitemap.xml`, `app/robots.ts` → `/robots.txt`.

```ts
// app/robots.ts
import type { MetadataRoute } from 'next'
export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/', disallow: ['/api', '/admin'] },
    sitemap: 'https://example.ru/sitemap.xml',
  }
}
```

- Задай `metadataBase` и canonical (`alternates.canonical`).
- Open Graph + Twitter-карточки → [[14-PWA-Favicon-OG]].
- JSON-LD через `<script type="application/ld+json">`.
- Уникальные title через `title.template`.
- Сабмит карты: Google Search Console + Яндекс.Вебмастер.
