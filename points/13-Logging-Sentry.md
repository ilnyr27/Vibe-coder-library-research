---
tags: [sentry, logging, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 13. Логи и Sentry / Logging

← [[VAIBCODER-Index]] | → [[09-Errors]]

`npx @sentry/wizard@latest -i nextjs` создаёт `instrumentation-client.ts`, `sentry.server.config.ts`, `sentry.edge.config.ts`, `global-error.tsx`, `instrumentation.ts` с `export const onRequestError = Sentry.captureRequestError`.

- Нужно `@sentry/nextjs` ≥ 8.28 + Next 15.
- `SENTRY_AUTH_TOKEN` — в CI-секреты → [[12-Env-Secrets]].
- Turbopack в dev пока поддержан не полностью — сверь на момент сборки.
