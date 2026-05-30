---
tags: [testing, ci, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 18. Тестирование / Testing

← [[VAIBCODER-Index]] | → [[Deploy-VPS]]

Минимум перед прода:
- `tsc --noEmit` + `eslint` в CI.
- Несколько unit-тестов (Vitest/Jest).
- Один Playwright smoke-тест критического пути (логин → ключевое действие).
- Запуск в GitHub Actions **до** деплоя → [[Deploy-VPS]].
