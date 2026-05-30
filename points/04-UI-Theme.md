---
tags: [ui, theme, tailwind, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 4. Светлая/Тёмная тема / Theme

← [[VAIBCODER-Index]] | → [[06-Performance]]

Tailwind v4 (CSS-first, без `tailwind.config.ts`) + `next-themes` + shadcn.

```css
@import "tailwindcss";
@custom-variant dark (&:is(.dark *));
:root { --background: hsl(0 0% 100%); --foreground: hsl(0 0% 3.9%); }
.dark { --background: hsl(0 0% 3.9%); --foreground: hsl(0 0% 98%); }
@theme inline { --color-background: var(--background); --color-foreground: var(--foreground); }
```

```tsx
"use client";
import { ThemeProvider } from "next-themes";
export function Providers({ children }: { children: React.ReactNode }) {
  return <ThemeProvider attribute="class" defaultTheme="system" enableSystem>{children}</ThemeProvider>;
}
```

- Тоггл защищай флагом `mounted` (иначе ошибка гидрации).
- `defaultTheme="system"` уважает `prefers-color-scheme`.
- Выбор сохраняется автоматически.
