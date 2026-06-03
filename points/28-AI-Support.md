---
tags: [ai, support, telegram, llm, security, point]
project: vaibcoder-library
type: point
updated: 2026-06-04
---

# 28. AI-поддержка и обработка возражений / AI-Powered Support

← [[VAIBCODER-Index]] | → [[27-Admin-Panel]] · [[16-API-RateLimit]] · [[03-Legal-152FZ]]

## Архитектура / Architecture

```
[Widget] → POST /api/chat (rate limit, validate)
         → INSERT user message (RF Postgres — FIRST!)
         → classify(text) → {category, sentiment, priority, escalate?}
         → if escalate/negative → Telegram alert
         → call LLM (YandexGPT/GigaChat in RF, or Claude w/ redaction)
         → INSERT AI message + confidence
         → return reply
```

**Главное правило: сохраняй КАЖДОЕ сообщение в РФ-Postgres ДО вызова AI.**
**Main rule: persist EVERY message to Russia-hosted Postgres BEFORE calling AI.**

## Схема БД / Database Schema

```sql
create table public.conversations (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id),
  status text not null default 'open'
    check (status in ('open','triaged','resolved')),
  created_at timestamptz not null default now()
);

create table public.messages (
  id bigint generated always as identity primary key,
  conversation_id uuid not null references public.conversations(id) on delete cascade,
  sender_role text not null check (sender_role in ('user','ai','human')),
  content text not null,
  ai_confidence numeric,
  escalated boolean default false,
  created_at timestamptz not null default now()
);

create table public.issues (
  id bigint generated always as identity primary key,
  conversation_id uuid references public.conversations(id),
  category text check (category in ('complaint','objection','bug','feature','question')),
  sentiment text check (sentiment in ('positive','neutral','negative')),
  priority text check (priority in ('low','medium','high','urgent')),
  status text not null default 'open' check (status in ('open','triaged','resolved')),
  summary text,
  created_at timestamptz not null default now()
);

-- RLS on all three tables!
alter table public.conversations enable row level security;
alter table public.messages enable row level security;
alter table public.issues enable row level security;
```

## Выбор AI-провайдера под 152-ФЗ / AI Provider Choice

| Провайдер / Provider | Данные / Data | 152-ФЗ |
|---------------------|-------------|--------|
| **YandexGPT** | Хранение в ДЦ РФ, ФСТЭК УЗ-3 / RF datacenters | Без трансграничной передачи / No cross-border |
| **GigaChat** (Sber) | Серверы РФ, OpenAI-compatible API / RF servers | Без трансграничной передачи / No cross-border |
| **Claude / OpenAI** | US servers | **Трансграничная передача** — уведомление РКН + согласие / Cross-border transfer — RKN notification + consent |

**Рекомендация: YandexGPT или GigaChat** — данные остаются в РФ, никаких юридических сложностей.
**Recommendation: YandexGPT or GigaChat** — data stays in RF, no legal headache.

**Если нужен Claude/OpenAI / If you must use Claude/OpenAI:**
- **Обезличь ПДн ДО отправки** — имена, ID, телефоны остаются в РФ Postgres, отправляй только деперсонализированный текст / Redact PII BEFORE sending
- Приказ Роскомнадзора №140 (с 01.09.2025): обезличивание должно делать реидентификацию **невозможной** без дополнительной информации + документированный анализ рисков реидентификации / Anonymization must make re-identification impossible + documented risk analysis
- Отдельное согласие на трансграничную передачу + уведомление РКН **до** начала / Separate consent + RKN notification before starting
- США не в списке «адекватной защиты» — ждать решения РКН до 10 рабочих дней / USA not on adequate-protection list — await RKN decision

## Supabase MCP — Смертельная триада / Lethal Trifecta

**6 июля 2025:** опубликована атака на Supabase MCP, когда атакующий внедряет инструкции в support-сообщение, и AI-агент с `service_role` выполняет вредоносный SQL.

**July 6, 2025:** published attack where attacker plants instructions in a support message, and AI agent with `service_role` executes malicious SQL.

**Защита / Defense:**
- **READ-ONLY mode** (флаг `read_only`) / read-only flag
- **Project-scoped** (`project_ref`) — ограничь одним проектом / limit to one project
- **Не подключай к production** — MCP для dev/test / Don't connect to production
- **Ручное подтверждение** каждого tool call в MCP-клиенте / Manual approval of every tool call
- RLS **НЕ спасает** — agent использует `service_role` / RLS does NOT save you — agent uses service_role

## Промпт для поддержки / Support System Prompt

Структурированный system prompt с ограничениями:

- **ROLE + COMPANY** — кто ты, на кого работаешь / who you are, who you work for
- **SOURCE POLICY:** отвечай ТОЛЬКО из KB/FAQ (RAG) / answer ONLY from provided KB/FAQ
- **UNKNOWN:** «Не могу ответить, передаю человеку» / "I don't have that; connecting you to a human"
- **CONFIDENCE < 85%** → эскалация на человека / escalate to human
- **ЗАПРЕЩЕНО:** придумывать фичи, цены, политики, сроки / FORBIDDEN: invent features, prices, policies, timelines
- **Обработка возражений:** признай → сочувствие → ответ из KB → предложи следующий шаг / acknowledge → empathize → answer from KB → offer next step
- **Эскалация:** низкая уверенность, негатив, повторное обращение, запрос «агента», возвраты/юридика/ПДн / Escalate: low confidence, negative sentiment, repeated contact, explicit agent request, refunds/legal/PII

## Telegram-алерты / Telegram Alerts

```ts
// lib/telegram.ts
export async function alertDev(text: string) {
  const url = `https://api.telegram.org/bot${process.env.TELEGRAM_BOT_TOKEN}/sendMessage`
  await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      chat_id: process.env.TELEGRAM_CHAT_ID,
      text,
      parse_mode: 'HTML',
      disable_web_page_preview: true,
    }),
  })
}

// Usage on escalation:
await alertDev(
  `🚨 <b>${category.toUpperCase()}</b> | sentiment: ${sentiment}\n` +
  `"${snippet}"\n` +
  `<a href="https://admin.example.com/conversations/${id}">open</a>`
)
```

- `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID` — server env vars / серверные переменные
- Создай бота через @BotFather / Create bot via @BotFather
- Можно также через DB trigger / Edge Function на INSERT в `issues` WHERE `priority='urgent'`

## Rate Limiting

```ts
// app/api/chat/route.ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'),
})

export async function POST(req: Request) {
  const ip = req.headers.get('x-forwarded-for')?.split(',')[0] ?? 'anon'
  const { success } = await ratelimit.limit(`chat:${ip}`)
  if (!success) return new Response('Too many requests', { status: 429 })
  // validate input length, then proceed...
}
```

## OWASP LLM Top 10:2025

- **LLM01 Prompt Injection** (#1) — нельзя полностью устранить; defense in depth / can't fully fix
- **LLM02 Sensitive Information Disclosure** (#2) — не давай модели доступ к чувствительным данным / don't give model access to sensitive data
- Input validation, output filtering, least privilege, human-in-the-loop

## Compliance / 152-ФЗ для AI-поддержки

- Сообщи пользователю что он говорит с AI и что сообщения сохраняются / Disclose AI + data storage
- Получи согласие / Obtain consent
- Публикуй политику хранения / Publish retention policy
- Сбор ПДн через чат → ты оператор ПДн → реестр РКН / Chat PD collection → you're a PD operator
- Удаляй/обезличивай по политике / Delete/anonymize per policy

## Чек-лист AI-поддержки / AI Support Checklist

- [ ] Rate limit per IP/user (Upstash)
- [ ] Input validation + token caps
- [ ] Persist to RF Postgres FIRST
- [ ] Prefer YandexGPT/GigaChat (no cross-border)
- [ ] Redact PII if using foreign LLM + document risk analysis
- [ ] System prompt with confidence threshold + escalate-to-human
- [ ] RAG/source-grounded, no commitments
- [ ] MCP read-only + project-scoped + manual approval, never on prod with write
- [ ] AI disclosure + consent + retention policy
- [ ] Telegram alert with category/sentiment/snippet/link
- [ ] Classify every conversation (complaint/objection/bug/feature/question)
