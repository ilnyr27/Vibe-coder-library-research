---
tags: [ai, llm, deepseek, claude, streaming, point]
project: vaibcoder-library
type: point
updated: 2026-07-21
---

# 29. AI / LLM интеграция

← [[VAIBCODER-Index]] | → [[31-Freemium]] · [[16-API-RateLimit]]

**RU:** Интеграция LLM в Next.js: выбор провайдера для РФ, SSE-стриминг, rate limiting, защита ключей.
**EN:** LLM integration in Next.js: provider choice for Russia, SSE streaming, rate limiting, key protection.

## Выбор провайдера для РФ / Provider selection for Russia

| Провайдер | Работает в РФ без VPN | Цена | Качество |
|-----------|----------------------|------|---------|
| **DeepSeek V3** | ✅ ДА | ~$0.14/1M input | Отличное, особенно EN |
| **DeepSeek R1** | ✅ ДА | ~$0.55/1M input | Лучшее для рассуждений |
| **Anthropic Claude** | ✅ ДА (API) | $3–15/1M input | Лучшее качество |
| OpenAI GPT-4 | ❌ Заблокирован (Cloudflare) | — | — |
| YandexGPT | ✅ ДА | По тарифам Яндекс | Слабее, но RU-контекст |
| GigaChat (Сбер) | ✅ ДА | По тарифам Сбер | API неудобный |

**Вывод для прода в РФ:** DeepSeek — основной провайдер (дёшево, работает), Claude — премиум-опция.

OpenAI в проде через RF-сервер **не работает стабильно** — не планируй его как основной.

## Абстракция провайдера / Provider abstraction

Не хардкодь провайдер — оберни в интерфейс. Это позволит переключить провайдера без переписывания всего кода.

```ts
// src/lib/ai/providers.ts
export interface AIMessage {
  role: "system" | "user" | "assistant";
  content: string;
}

interface AIProvider {
  streamChat(messages: AIMessage[]): Promise<ReadableStream<string>>;
  complete(messages: AIMessage[]): Promise<string>;
}

class DeepSeekProvider implements AIProvider {
  private client = new OpenAI({
    apiKey: process.env.DEEPSEEK_API_KEY!,
    baseURL: "https://api.deepseek.com",
  });

  async streamChat(messages: AIMessage[]): Promise<ReadableStream<string>> {
    const stream = await this.client.chat.completions.create({
      model: "deepseek-chat",
      messages,
      stream: true,
      max_tokens: 2000,
    });
    return toReadableStream(stream);
  }

  async complete(messages: AIMessage[]): Promise<string> {
    const res = await this.client.chat.completions.create({
      model: "deepseek-chat",
      messages,
      max_tokens: 4000,
    });
    return res.choices[0].message.content ?? "";
  }
}

export function createProvider(type: "deepseek" | "claude" = "deepseek"): AIProvider {
  return new DeepSeekProvider(); // расширяй по мере добавления провайдеров
}
```

DeepSeek API совместим с OpenAI SDK — ставишь `@openai/openai` и просто меняешь `baseURL`.

## SSE-стриминг / Server-Sent Events streaming

### API route (сервер)

```ts
// app/api/ai/chat/route.ts
export async function POST(request: Request) {
  // ... проверка сессии, лимитов ...

  const provider = createProvider("deepseek");
  const aiStream = await provider.streamChat(messages);

  const encoder = new TextEncoder();
  const readable = new ReadableStream({
    async start(controller) {
      const reader = aiStream.getReader();
      const decoder = new TextDecoder();

      try {
        while (true) {
          const { done, value } = await reader.read();
          if (done) break;
          const text = decoder.decode(value);
          controller.enqueue(encoder.encode(`data: ${JSON.stringify({ content: text })}\n\n`));
        }
        controller.enqueue(encoder.encode("data: [DONE]\n\n"));
      } finally {
        controller.close();
      }
    },
  });

  return new Response(readable, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      Connection: "keep-alive",
    },
  });
}

// Хелпер: openai stream → ReadableStream<string>
async function* streamToAsyncIterable(stream: AsyncIterable<any>) {
  for await (const chunk of stream) {
    yield chunk.choices[0]?.delta?.content ?? "";
  }
}
function toReadableStream(stream: AsyncIterable<any>): ReadableStream<string> {
  const iter = streamToAsyncIterable(stream);
  return new ReadableStream({
    async pull(controller) {
      const { value, done } = await iter.next();
      if (done) controller.close();
      else if (value) controller.enqueue(value);
    },
  });
}
```

### Клиент / Client-side consumption

```ts
// hooks/useAIStream.ts
export async function streamAIResponse(
  url: string,
  body: object,
  onChunk: (text: string) => void,
  onDone: () => void
) {
  const res = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
  });

  if (!res.ok) throw new Error(`AI error: ${res.status}`);

  const reader = res.body!.getReader();
  const decoder = new TextDecoder();
  let accumulated = "";

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value, { stream: true });
    const lines = chunk.split("\n").filter((l) => l.startsWith("data: "));

    for (const line of lines) {
      const data = line.slice(6); // убираем "data: "
      if (data === "[DONE]") { onDone(); return; }
      try {
        const parsed = JSON.parse(data);
        accumulated += parsed.content ?? "";
        onChunk(accumulated);
      } catch {}
    }
  }
  onDone();
}
```

## Rate limiting для AI-эндпоинтов / AI endpoint rate limiting

AI-запросы дорогие — rate limit должен быть жёстче, чем у обычных API:

```ts
// src/lib/ai/rateLimit.ts
const userRequestTimes = new Map<string, number[]>();

export function checkAIRateLimit(userId: string, maxPerMinute = 10): boolean {
  const now = Date.now();
  const windowMs = 60_000;
  const times = (userRequestTimes.get(userId) ?? []).filter(
    (t) => now - t < windowMs
  );
  if (times.length >= maxPerMinute) return false;
  times.push(now);
  userRequestTimes.set(userId, times);
  return true;
}

// В API route:
if (!checkAIRateLimit(user.id)) {
  return Response.json({ error: "Too many requests" }, { status: 429 });
}
```

⚠️ In-memory rate limit — только для одного инстанса. Для multi-instance → Redis → [[16-API-RateLimit]]

## Системный промпт / System prompt

```ts
// src/lib/ai/prompt-builder.ts
export function buildSystemPrompt(context: {
  userName?: string;
  language?: "ru" | "en";
  mode: "coach" | "analyst";
}): string {
  return `
You are a ${context.mode === "coach" ? "psychological coach" : "data analyst"}.
Always respond in ${context.language === "ru" ? "Russian" : "English"}.
${context.userName ? `The user's name is ${context.userName}.` : ""}

Rules:
- Ask clarifying questions before diving deep
- Don't give unsolicited medical advice
- Keep responses focused and concise
`.trim();
}
```

**Правила системного промпта:**
- Язык инструкции — первой строкой (AI следует ей точнее)
- Не передавай все данные сразу — строй контекст постепенно по ходу диалога
- Системный промпт не меняется от сообщения к сообщению — провайдеры кэшируют его
- Правила форматирования (markdown, длина) — тоже в промпт

## Управление историей / History management

```ts
// Не шли всю историю — обрезай по токенам
const MAX_HISTORY_MESSAGES = 20;
const messages = [
  { role: "system", content: systemPrompt },
  ...chatHistory.slice(-MAX_HISTORY_MESSAGES), // последние N сообщений
  { role: "user", content: userMessage },
];
```

Полная история чата → токены растут → стоимость растёт экспоненциально. Обрезай.

## Грабли / Gotchas

### 🔴 API-ключ никогда в браузере

```ts
// ❌ НИКОГДА
const client = new OpenAI({ apiKey: process.env.NEXT_PUBLIC_DEEPSEEK_KEY });

// ✅ Только в server-side коде (route.ts, server action)
const client = new OpenAI({ apiKey: process.env.DEEPSEEK_API_KEY });
```

`NEXT_PUBLIC_` → переменная попадает в JS-бандл → ключ виден всем.

### 🔴 Нет таймаута → зависание

AI может думать вечно. Всегда устанавливай таймаут:

```ts
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 30_000); // 30 сек

try {
  const stream = await client.chat.completions.create(
    { ...params, stream: true },
    { signal: controller.signal }
  );
  // ...
} finally {
  clearTimeout(timeout);
}
```

### 🔴 Streaming без обработки ошибок → фронт зависает

Если AI-роут крашнул после начала стриминга — клиент получит обрыв потока без ошибки. Всегда оборачивай в try/catch и шли маркер ошибки:

```ts
} catch (err) {
  controller.enqueue(encoder.encode(`data: ${JSON.stringify({ error: "AI error" })}\n\n`));
  controller.close();
}
```

### OpenAI SDK + DeepSeek = совместимо

DeepSeek реализует OpenAI-совместимый API. Меняешь только `baseURL`:

```ts
const client = new OpenAI({
  apiKey: process.env.DEEPSEEK_API_KEY,
  baseURL: "https://api.deepseek.com", // ← единственное изменение
});
```

Переход с OpenAI на DeepSeek занимает 2 минуты.

### Параллельные запросы и счётчики

Если пользователь жмёт «отправить» несколько раз подряд — несколько AI-запросов уходят одновременно. Счётчик сообщений надо инкрементировать атомарно — см. [[31-Freemium]].

## Переменные окружения / Environment variables

```bash
# .env.local
DEEPSEEK_API_KEY=sk-...       # никогда NEXT_PUBLIC_
ANTHROPIC_API_KEY=sk-ant-...  # никогда NEXT_PUBLIC_
```

→ [[12-Env-Secrets]]
