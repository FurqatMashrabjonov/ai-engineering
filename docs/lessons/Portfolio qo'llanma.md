---
tags: [lesson, karyera]
status: o'rganilmoqda
---
# Portfolio va bozor

Xarita: [[Production (xarita)]] · [[Learning app]] · [[Home]]

**Bozor (2026):** AQShda yanvar-oktyabrda 43 mingdan ortiq AI engineering vakansiyasi, 32% masofaviy. Lekin junior ulushi atigi 6-12%. Ya'ni "kurs tugatdim" yetmaydi; ishlaydigan, o'lchangan loyiha kerak. Sizning ustunligingiz: siz allaqachon backend dasturchisiz. Ko'p vakansiyalar aynan "LLM'ni bilgan software engineer"ni qidiradi.

**Vakansiyalarda eng ko'p uchraydigan narsalar va Learning app'dagi joyi:**

| Talab | Learning app'dagi isbot |
| --- | --- |
| RAG, vektor baza | v1-v2: pgvector, hybrid search, rerank |
| Structured output, tool calling | v3: test/flashcard, `search_docs` tool |
| Agentlar (LangGraph) | O'qish rejasi agenti, human-in-the-loop |
| API, production | FastAPI, streaming, Docker, deploy |
| Eval | Golden set + o'lchovlar jadvali |
| Observability | Langfuse yoki o'z log jadvali |
| MCP | Hujjat qidiruvi MCP server sifatida |

**README'da bo'lishi kerak:**

1. 1-2 daqiqalik demo video yoki GIF.
2. Arxitektura sxemasi (fayl → matn → bo'lak → embedding → pgvector → qidiruv → LLM).
3. **Eval natijalari jadvali** va qaysi o'zgarish nimani yaxshilagani. Bu eng kuchli qism: 31% kompaniyada eval umuman yo'q.
4. Qabul qilingan qarorlar va nima uchun ("nega pgvector, nega hybrid").
5. Narx: bitta savol o'rtacha necha sent.

**Qo'shimcha:** har versiyadan keyin qisqa post (LinkedIn, Medium yoki Telegram): "chunk hajmini o'zgartirdim, recall 0.72 dan 0.81 ga chiqdi". O'rganganingizni yozish ham o'rganishning bir qismi.

**Tartib (tavsiya):** [[Structured output to'liq qo'llanma]] → [[FastAPI va streaming qo'llanma]] → [[Tool calling qo'llanma]] → [[LLM-as-a-judge qo'llanma]] → [[Eval qo'llanma]] → [[Observability qo'llanma]] → [[Agentlar va LangGraph qo'llanma]] → [[MCP qo'llanma]] → [[Guardrails va OWASP qo'llanma]] → [[Docker va deploy qo'llanma]] → [[Portfolio qo'llanma]]. Eval'ni agentlardan oldin qiling: agentni o'lchamasdan yaxshilab bo'lmaydi.

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
