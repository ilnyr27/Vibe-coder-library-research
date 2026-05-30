---
tags: [payments, yookassa, point]
project: vaibcoder-library
type: point
updated: 2026-05-30
---

# 20. Платежи / Payments (ЮKassa)

← [[VAIBCODER-Index]] | → [[16-API-RateLimit]] · [[03-Legal-152FZ]]

Для РФ — **ЮKassa**.
- Создание платежа: `POST /v3/payments` с заголовком **`Idempotence-Key`**.
- Вебхуки: `POST /v3/webhooks`, слушай `payment.succeeded` на свой URL уведомлений.
- Подтверждение/capture платежа.
- Node-SDK: пакет `yookassa`.
- Настрой систему налогообложения/НДС; самозанятые — чеки в «Мой налог».
- Не доверяй сумме с клиента — сверяй на сервере → [[16-API-RateLimit]].
