---
tags: [freemium, monetization, limits, usage, point]
project: vaibcoder-library
type: point
updated: 2026-07-21
---

# 31. Freemium / Usage Limits

← [[VAIBCODER-Index]] | → [[29-AI-LLM]] · [[20-Payments-YooKassa]] · [[15-DB-Schema]]

**RU:** Freemium-модель: схема БД для отслеживания лимитов, атомарные счётчики, paywall, логика бесплатного тира.
**EN:** Freemium model: DB schema for usage tracking, atomic counters, paywall enforcement, free tier logic.

## Принцип / Core principle

**Лимиты проверяй и применяй ТОЛЬКО на сервере.** UI может показывать состояние, но решение принимает сервер. Пользователь всегда может вызвать API напрямую, минуя фронт.

**Enforce limits server-side ONLY. UI shows state, server decides. Users can always call the API directly.**

## Схема БД / DB schema

```sql
-- Планы и лимиты пользователя
CREATE TABLE user_plans (
  user_id              UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,

  -- счётчики сессий / session counters
  deepseek_sessions    INT  DEFAULT 0,
  claude_sessions      INT  DEFAULT 0,

  -- лимит сообщений на сессию / messages per session
  deepseek_msg_limit   INT  DEFAULT 40,
  claude_msg_limit     INT  DEFAULT 0,

  -- флаги одноразового бесплатного тира / one-time free tier flags
  free_session_used    BOOLEAN DEFAULT FALSE,
  free_analysis_used   BOOLEAN DEFAULT FALSE,

  -- дополнительные возможности / feature access
  has_report           BOOLEAN DEFAULT FALSE,

  -- подписка / subscription
  subscription_plan              TEXT CHECK (subscription_plan IN ('pro', 'max')),
  subscription_expires_at        TIMESTAMPTZ,
  subscription_payment_method_id TEXT,
  subscription_cancelled_at      TIMESTAMPTZ,

  updated_at           TIMESTAMPTZ DEFAULT NOW()
);

-- Активные сессии / Active chat sessions
CREATE TABLE chat_sessions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  provider        TEXT CHECK (provider IN ('deepseek', 'claude', 'free')),
  messages_used   INT  DEFAULT 0,
  messages_limit  INT  NOT NULL,
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX ON chat_sessions (user_id, is_active);
```

## Константы / Constants

```ts
// src/lib/payment/plans.ts
export const FREE_MSG_LIMIT = 12;   // сообщений в бесплатной сессии
export const DS_MSG_LIMIT   = 40;   // DeepSeek платная сессия
export const CL_MSG_LIMIT   = 40;   // Claude платная сессия

// При upsert бесплатной сессии — не перезаписывай лимит если он уже задан!
// (пользователь мог купить пакет с повышенным лимитом до использования бесплатной)
const msgLimit = plan?.deepseek_msg_limit ?? FREE_MSG_LIMIT;
```

## Логика старта сессии / Session start logic

```ts
// app/api/session/start/route.ts
export async function POST(request: Request) {
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return Response.json({ error: "Unauthorized" }, { status: 401 });

  const { data: plan } = await supabase.from("user_plans").select("*").eq("user_id", user.id).single();

  // Попытка бесплатной сессии
  if (sessionType === "free") {
    if (plan?.free_session_used) {
      return Response.json({ error: "Free session already used" }, { status: 403 });
    }
    // Upsert — сохрани уже купленный лимит, если есть
    await supabase.from("user_plans").upsert({
      user_id: user.id,
      free_session_used: true,
      deepseek_msg_limit: plan?.deepseek_msg_limit ?? FREE_MSG_LIMIT,
    }, { onConflict: "user_id" });

    const { data: session } = await supabase.from("chat_sessions").insert({
      user_id: user.id,
      provider: "free",
      messages_limit: plan?.deepseek_msg_limit ?? FREE_MSG_LIMIT,
    }).select().single();

    return Response.json({ sessionId: session.id });
  }

  // Платная сессия DeepSeek
  if (sessionType === "deepseek") {
    const remaining = plan?.deepseek_sessions ?? 0;
    if (remaining <= 0) return Response.json({ error: "No sessions left" }, { status: 403 });

    await supabase.from("user_plans").update({
      deepseek_sessions: remaining - 1,
      free_session_used: true,   // ← закрываем бесплатную сессию при старте ЛЮБОЙ платной
      updated_at: new Date().toISOString(),
    }).eq("user_id", user.id);

    const { data: session } = await supabase.from("chat_sessions").insert({
      user_id: user.id,
      provider: "deepseek",
      messages_limit: plan?.deepseek_msg_limit ?? DS_MSG_LIMIT,
    }).select().single();

    return Response.json({ sessionId: session.id });
  }
}
```

**Важно:** `free_session_used = true` устанавливается при старте **любой** сессии (платной или бесплатной). Иначе пользователь может сначала купить сессии, использовать их, а потом использовать бесплатную сверх лимита.

## Атомарный счётчик сообщений / Atomic message counter

```ts
// app/api/ai/chat/route.ts — ПЕРЕД отправкой в AI

// 1. Читаем текущее состояние сессии
const { data: session } = await supabase
  .from("chat_sessions")
  .select("id, messages_used, messages_limit, is_active")
  .eq("id", sessionId)
  .eq("user_id", user.id)
  .single();

if (!session?.is_active) return Response.json({ error: "Session expired" }, { status: 403 });
if (session.messages_used >= session.messages_limit) {
  return Response.json({ error: "Message limit reached" }, { status: 403 });
}

// 2. Атомарный инкремент (оптимистичная блокировка)
// WHERE messages_used = <снапшот> — гарантирует что никто не изменил между read и write
const { data: incremented } = await supabase
  .from("chat_sessions")
  .update({ messages_used: session.messages_used + 1 })
  .eq("id", sessionId)
  .eq("messages_used", session.messages_used)   // ← атомарная проверка
  .select("id");

if (!incremented?.length) {
  // параллельный запрос успел обновить счётчик — просим retry
  return Response.json({ error: "Concurrent request, please retry" }, { status: 409 });
}

// 3. Только теперь идём в AI
const aiResponse = await streamFromAI(messages);
```

**Почему не fire-and-forget:** обновление после ответа AI не защищает от параллельных запросов. При быстром двойном нажатии оба запроса читают `messages_used = 5`, оба пишут `6`, счётчик не доходит до лимита.

## Проверка лимита на каждом сообщении / Per-message limit check

```ts
// Всегда проверяй лимит ПЕРЕД тем как идти в AI
// Не доверяй тому, что UI «уже проверил»

if (session.messages_used >= session.messages_limit) {
  return Response.json({
    error: "limit_reached",
    used: session.messages_used,
    limit: session.messages_limit,
  }, { status: 403 });
}
```

Клиент получает `error: "limit_reached"` → показывает paywall.

## Динамические лимиты / Dynamic limits (subscription-aware)

Если у пользователя подписка истекла — лимит должен упасть до базового. Проверяй на каждом сообщении:

```ts
function getEffectiveMsgLimit(plan: UserPlan): number {
  if (!plan.subscription_plan || !plan.subscription_expires_at) return DS_MSG_LIMIT;
  const isActive = new Date(plan.subscription_expires_at) > new Date();
  return isActive ? SUB_PLANS[plan.subscription_plan].msgLimit : DS_MSG_LIMIT;
}
```

## Paywall на фронте / Frontend paywall

```tsx
// После получения error: "limit_reached" от API
const [showPaywall, setShowPaywall] = useState(false);

// В обработчике ответа от /api/ai/chat:
if (res.status === 403) {
  const body = await res.json();
  if (body.error === "limit_reached") setShowPaywall(true);
}

// UI
{showPaywall && (
  <PaywallBanner
    used={session.messagesUsed}
    limit={session.messagesLimit}
    onUpgrade={() => router.push("/pricing")}
  />
)}
```

**Paywall должен показывать конкретно ЧТО пользователь получит после оплаты** — количество сообщений, функции.

## Начисление после оплаты / Credit after payment

```ts
// app/api/payment/webhook/route.ts — после верификации платежа

if (productType === "pack_ds_5") {
  await supabase.from("user_plans").upsert({
    user_id: userId,
    deepseek_sessions: (existing?.deepseek_sessions ?? 0) + 5,
    deepseek_msg_limit: 40,
  }, { onConflict: "user_id", ignoreDuplicates: false });
}

if (productType?.startsWith("subscription_")) {
  const plan = SUB_PLANS[productType]; // { deepseekSessions: 20, msgLimit: 60, ... }
  const currentExpiry = existing?.subscription_expires_at
    ? new Date(existing.subscription_expires_at)
    : new Date();
  // Продлеваем от текущего конца, не от сегодня (защита от раннего продления)
  const nextExpiry = new Date(Math.max(currentExpiry.getTime(), Date.now()));
  nextExpiry.setMonth(nextExpiry.getMonth() + 1);

  await supabase.from("user_plans").upsert({
    user_id: userId,
    subscription_plan: plan.name,
    subscription_expires_at: nextExpiry.toISOString(),
    deepseek_sessions: (existing?.deepseek_sessions ?? 0) + plan.deepseekSessions,
    deepseek_msg_limit: plan.msgLimit,
  }, { onConflict: "user_id" });
}
```

→ [[20-Payments-YooKassa]] для вебхук-верификации

## Грабли / Gotchas

### Upsert перезаписывает существующий лимит

```ts
// ❌ Уничтожает купленный лимит при бесплатном старте
await supabase.from("user_plans").upsert({
  user_id,
  free_session_used: true,
  deepseek_msg_limit: FREE_MSG_LIMIT, // ← затирает если пользователь купил 60-msg план
});

// ✅ Сохраняй существующее значение
await supabase.from("user_plans").upsert({
  user_id,
  free_session_used: true,
  deepseek_msg_limit: existingPlan?.deepseek_msg_limit ?? FREE_MSG_LIMIT,
});
```

### Бесплатная сессия доступна после использования платных

Сценарий: пользователь купил 5 сессий → использовал все 5 → использует бесплатную сверху.

Фикс: при старте **любой** платной сессии ставить `free_session_used = true`.

### Лимит проверяется только в UI

AI-роут без серверной проверки лимита → любой curl-запрос с валидным токеном использует AI бесплатно. Всегда проверяй `messages_used >= messages_limit` на сервере.

### Нет индекса на `(user_id, is_active)`

На каждое сообщение в чате идёт запрос к `chat_sessions`. Без индекса — seq scan на каждый запрос.

```sql
CREATE INDEX ON chat_sessions (user_id, is_active);
```
