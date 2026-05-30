---
tags: [forms, validation, zod, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 8. Формы и валидация / Forms

← [[VAIBCODER-Index]] | → [[01-Auth]] · [[09-Errors]]

`react-hook-form` + `zod` (`@hookform/resolvers/zod`). Валидируй **на отправке** (или `mode:"onBlur"`), НЕ на каждое нажатие.

```tsx
const form = useForm<FormFields>({ resolver: zodResolver(schema), mode: "onBlur" });
<form onSubmit={form.handleSubmit(onSubmit)}>…</form>
```

- Одна zod-схема для клиента **и** сервера (Server Actions `safeParse`) → [[16-API-RateLimit]].
- Компоненты `Form` shadcn подключают ошибки автоматически.
- Тост по результату (sonner) → [[09-Errors]].
- Не очищай поля при ошибке валидации.
