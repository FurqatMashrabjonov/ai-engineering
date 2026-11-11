---
tags: [llm-api, tushuncha]
status: o'rganilmoqda
---
# Retry va backoff

Vaqtinchalik xatodan (timeout, 429, 5xx) keyin qayta urinish, har safar kutishni ikki baravar oshirib (exponential backoff) va tasodifiy qo'shimcha (jitter) bilan. 400 va 401'da qayta urinilmaydi. SDK standart holatda 2 marta urinadi.

## Bog'liq
- [[Rate limit]]
- [[SDK]]

## Manba
- [[LLM API to'liq qo'llanma#7. Xatolar, timeout va retry]]
- [[LLM API]]
