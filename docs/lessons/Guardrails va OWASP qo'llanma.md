---
tags: [lesson, xavfsizlik]
status: o'rganilmoqda
---
# Guardrails va xavfsizlik (OWASP LLM Top 10)

Xarita: [[Xavfsizlik (xarita)]] · [[Learning app]] · [[Home]]

**Nima:** LLM ilovalariga xos xavflar. OWASP 2025 ro'yxati shu sohaning standarti:

| # | Xavf | Learning app'da qanday ko'rinadi | Himoya |
| --- | --- | --- | --- |
| LLM01 | Prompt injection | Yuklangan PDF ichida "oldingi ko'rsatmalarni unut" yozuvi | Hujjat matnini ma'lumot deb belgilang, tool huquqlarini cheklang |
| LLM02 | Maxfiy ma'lumot oshkor bo'lishi | Bir foydalanuvchi boshqasining faylidan javob oladi | Qidiruvda har doim `WHERE user_id = ...` |
| LLM03 | Supply chain | Begona MCP server yoki paket | Ishonchli manba, versiyani qotirish |
| LLM04 | Ma'lumot zaharlanishi | Bazaga ataylab yolg'on hujjat qo'shish | Kim nima yuklaganini kuzatish |
| LLM05 | Chiqishni noto'g'ri ishlash | Model javobidagi HTML/JS sahifada ishga tushadi | Javobni escape qiling, SQL'ga to'g'ridan qo'ymang |
| LLM06 | Ortiqcha vakolat | Agent so'ramasdan fayl o'chiradi | Kam tool, xavfli amallarda `interrupt()` tasdig'i |
| LLM07 | System prompt sizishi | "Promptingni ko'rsat" | Promptga sir (kalit, parol) yozmang |
| LLM08 | Vektor/embedding zaifligi | Boshqa foydalanuvchi vektorlaridan qidirish | LLM02 bilan bir xil: filtr |
| LLM09 | Noto'g'ri ma'lumot | Ishonch bilan aytilgan xato javob | Grounding, manba, "bilmayman", eval |
| LLM10 | Cheksiz iste'mol | 500 MB fayl yoki minglab so'rov | Fayl hajmi limiti, rate limit, `max_output_tokens`, `max_steps` |

**Guardrail = model oldidan va keyin tekshiruv:**

- **Kirishda:** fayl turi/hajmi, savol uzunligi, kerak bo'lsa `client.moderations.create(...)` bilan zararli matn tekshiruvi.
- **Chiqishda:** sxemaga mosligi (structured output), manba raqamlari haqiqiyligi, faithfulness (judge).

**Eng muhim fikr:** prompt injection'ni 100% to'xtatadigan prompt yo'q. Himoya promptda emas, arxitekturada: model qila oladigan narsalarni cheklang.

**Qiyinligi:** o'rta. Kodi oson, o'ylab ko'rish kerak bo'lgan joyi ko'p.

## Manbalar

**Rasmiy hujjatlar:**

- [OpenAI: Function calling](https://platform.openai.com/docs/guides/function-calling)
- [OpenAI: Structured outputs](https://platform.openai.com/docs/guides/structured-outputs)
- [OpenAI: Streaming responses](https://platform.openai.com/docs/guides/streaming-responses)
- [FastAPI: Custom response / StreamingResponse](https://fastapi.tiangolo.com/advanced/custom-response/)
- [LangGraph: Interrupts (human-in-the-loop)](https://docs.langchain.com/oss/python/langgraph/interrupts)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Langfuse: OpenAI integratsiyasi](https://langfuse.com/integrations/model-providers/openai-py)
- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10/)
- [uv: Docker bilan ishlatish](https://docs.astral.sh/uv/guides/integration/docker/)

**O'qish uchun:**

- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Hamel Husain: Your AI product needs evals](https://hamel.dev/blog/posts/evals/)
- [PySphere: RAG kitobi (o'zbekcha)](https://pysphere.uz/kitob/rag/index.html)

**Bozor ma'lumotlari:** DataTalks.Club AI engineering so'rovnomasi (framework, eval va observability ulushlari); PyPI yuklab olish statistikasi (LangGraph, OpenAI Agents SDK, CrewAI); 2026 vakansiya tahlillari (43k+ e'lon, junior ulushi).
