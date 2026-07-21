---
tags: [payments, yookassa, point]
project: vaibcoder-library
type: point
updated: 2026-07-19
---

# 20. Платежи / Payments (ЮKassa)

← [[VAIBCODER-Index]] | → [[16-API-RateLimit]] · [[03-Legal-152FZ]]

Для РФ — **ЮKassa** (она же YooMoney Checkout — это одна компания, checkout идёт через `yoomoney.ru`, это нормально).

## Настройка / Setup

```
YUKASSA_SHOP_ID=ваш_shop_id
YUKASSA_SECRET_KEY=live_secret_ключ   ← НИКОГДА не в git → [[12-Env-Secrets]]
```

Базовый заголовок авторизации — HTTP Basic Auth: `Base64(shopId:secretKey)`.

```ts
// src/lib/payment/yukassa.ts
const AUTH = "Basic " + Buffer.from(`${SHOP_ID}:${SECRET_KEY}`).toString("base64");
```

## Флоу платежа / Payment flow

```
1. Клиент нажимает "Купить"
2. POST /api/payment/create  → создаём платёж в ЮKassa → получаем confirmation_url
3. Редирект на yoomoney.ru/checkout (это нормально — это и есть ЮKassa checkout)
4. Пользователь платит
5. ЮKassa → POST /api/payment/webhook (event: payment.succeeded)
6. Вебхук верифицируем → начисляем сессии/подписку
```

## Создание платежа / Create payment

```ts
const payment = await fetch("https://api.yookassa.ru/v3/payments", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: AUTH,
    "Idempotence-Key": crypto.randomUUID(), // ОБЯЗАТЕЛЬНО — защита от дублей
  },
  body: JSON.stringify({
    amount: { value: (kopecks / 100).toFixed(2), currency: "RUB" },
    capture: true,
    confirmation: { type: "redirect", return_url: returnUrl },
    description: "Описание для пользователя",
    metadata: {
      user_id: user.id,        // ← ОБЯЗАТЕЛЬНО для вебхука
      product_type: "pro_3",   // ← что именно куплено
      // любые свои поля — ЮКасса вернёт их в вебхуке
    },
    // save_payment_method: true  ← НЕ включай пока магазин не подключил рекуррентные!
  }),
});
```

Редиректь пользователя на `payment.confirmation.confirmation_url`.

## Вебхук / Webhook

### 🔴 КРИТИЧНО: верифицируй платёж перед начислением

ЮKassa **не предоставляет HMAC-подпись** для вебхуков по умолчанию. Любой может отправить фейковый POST с `payment.succeeded` и получить бесплатную подписку.

**Обязательная верификация:** перед обработкой — запроси платёж у ЮKassa и убедись, что он реально `succeeded`:

```ts
// app/api/payment/webhook/route.ts
export async function POST(request: NextRequest) {
  const body = await request.json();
  if (body.event !== "payment.succeeded") return Response.json({ ok: true });

  const yukassaId = body.object.id;
  const metadata = body.object.metadata || {};

  // ✅ ВЕРИФИКАЦИЯ — запрос к ЮКасса API
  const verified = await getPayment(yukassaId);
  if (verified.status !== "succeeded") {
    return Response.json({ ok: true }); // игнорируем, ЮКасса повторит
  }

  // только после верификации — начисляем
  const userId = metadata.user_id;
  // ...
}

// src/lib/payment/yukassa.ts
export async function getPayment(id: string) {
  const res = await fetch(`https://api.yookassa.ru/v3/payments/${id}`, {
    headers: { Authorization: AUTH },
  });
  return res.json();
}
```

### Шаблон вебхука / Webhook template

```ts
// Разбери metadata.product_type — определяет что начислять
if (metadata.product_type?.startsWith("subscription_")) {
  // подписка: обнови subscription_expires_at = now + 30 дней, добавь сессии
} else if (metadata.product_type === "report_addon") {
  // разовый отчёт: поставь has_report = true
} else {
  // пакет сессий: добавь sessions по metadata.sessions и metadata.msgs_per_session
}

// Email пользователю (fire-and-forget)
sendPaymentConfirmation({ to: userEmail, productType: metadata.product_type })
  .catch(console.error);

return Response.json({ ok: true });
```

**ЮKassa ждёт ответ 200** — если вернёшь 4xx/5xx или долго не ответишь, она будет повторять вебхук. Возвращай `{ ok: true }` даже при неизвестном типе события.

## Грабли / Gotchas

### 🔴 `save_payment_method: true` → 403

**Симптом:** кнопка «Оформить подписку» → 403 от /api/payment/create. Кнопка «Купить сессии» работает.

**Причина:** `save_payment_method: true` отправлен в ЮКасса, но у магазина не подключена функция рекуррентных платежей → ЮКасса возвращает 403.

**Фикс:** `savePaymentMethod = false` пока магазин не подключил рекуррентные платежи. Заменить на `true` только после явного подтверждения от ЮКасса.

```ts
// ВСЕГДА false пока не включены рекуррентные платежи в магазине ЮКасса
savePaymentMethod = false;
// save_payment_method передаётся в ЮКасса только если true:
...(savePaymentMethod && { save_payment_method: true }),
```

### Разница YooMoney / ЮKassa

`yoomoney.ru/checkout/payments/v2/contract?orderId=...` — это **нормально**. ЮKassa и ЮMoney — одна компания. Checkout-страница всегда на yoomoney.ru.

### Сумма — всегда в копейках на сервере

Не доверяй сумме с клиента. Цены храни в копейках (int), переводи в рубли только при отправке в API:
```ts
const rubles = (amountKopecks / 100).toFixed(2); // "499.00"
```

### Idempotence-Key — обязателен

Без него повторный запрос (retry, сетевой сбой) создаст дубль платежа. Генерируй `crypto.randomUUID()` на каждый вызов.

### Race condition в начислении при параллельных запросах

Если пользователь может параллельно отправлять запросы (чат, счётчики сообщений) — используй атомарный UPDATE:

```ts
// НЕ делай так (race condition):
.update({ messages_used: session.messages_used + 1 }) // читает снапшот

// Делай так (оптимистичная блокировка):
const { data } = await supabase
  .from("chat_sessions")
  .update({ messages_used: session.messages_used + 1 })
  .eq("id", sessionId)
  .eq("messages_used", session.messages_used) // ← WHERE-условие атомизирует
  .select("id");

if (!data?.length) {
  return Response.json({ error: "Concurrent request, retry" }, { status: 409 });
}
```

### Чеки / Receipts

Если магазин на УСН/ОСН — нужен чек в ФНС. Самозанятые — чеки в «Мой налог» вручную. Настрой `receipt` в теле платежа согласно документации ЮKassa.

## Подписки / Subscriptions

ЮKassa не управляет подписками автоматически. Всё вручную:

1. При оплате: `subscription_expires_at = MAX(текущая дата, expiry) + 30 дней`
2. Авторенью (если включены рекуррентные): дневной cron → ищет подписки с `expires_at <= now + 2 days` → `chargeWithSavedMethod(paymentMethodId)`
3. Если `save_payment_method` не включён — пользователь продлевает вручную

```ts
// Продление: не сбрасывай, а добавляй к текущему сроку
const currentExpiry = existing?.subscription_expires_at
  ? new Date(existing.subscription_expires_at)
  : new Date();
const nextExpiry = new Date(Math.max(currentExpiry.getTime(), Date.now()));
nextExpiry.setMonth(nextExpiry.getMonth() + 1);
```

## Безопасность / Security

- `YUKASSA_SECRET_KEY` — только на сервере, никогда в браузере → [[12-Env-Secrets]]
- При ротации ключа (если засвечен) — смени в кабинете ЮKassa, обнови `.env`, перезапусти контейнер
- Верифицируй каждый вебхук через `getPayment()` перед начислением (см. выше)
- Логируй `[webhook]` события с `user_id` и `product_type` — удобно для разбора инцидентов
