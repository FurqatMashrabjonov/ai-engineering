---
tags: [lesson, production]
status: o'rganilmoqda
---
# Observability: ichkarida nima bo'layotganini ko'rish

Xarita: [[Production (xarita)]] · [[Learning app]] · [[Home]]

**Nima:** Har bir so'rov uchun yozuv (trace): qaysi bo'laklar topildi, modelga qanday prompt ketdi, nima qaytdi, necha token, necha soniya, qancha pul. Foydalanuvchi "noto'g'ri javob berdi" desa, sababini shu yerdan topasiz. Laravel'dagi Telescope'ning LLM versiyasi deb tushuning.

**Haqiqat:** so'rovnomada 48% o'zi yozgan logging ishlatadi, 41%da umuman yo'q. Tayyor vositalardan eng mashhuri **Langfuse** (open source, o'zingiz host qilsangiz ham bo'ladi, bulutda tekin rejasi bor).

**Langfuse ulash:**

```python
# .env ga qo'shing:
# LANGFUSE_PUBLIC_KEY=pk-lf-...
# LANGFUSE_SECRET_KEY=sk-lf-...
# LANGFUSE_BASE_URL=https://cloud.langfuse.com

from langfuse.openai import OpenAI      # oddiy openai o'rniga, qolgani o'zgarmaydi
from langfuse import observe

client = OpenAI()

@observe()                               # funksiya trace bo'lib yoziladi
def answer(question: str) -> str:
    hits = search(question, k=5)          # ichki qadamlarni ham @observe qilsa bo'ladi
    ...
```

**Diqqat:** Langfuse hujjatlaridagi drop-in misollar `chat.completions` bilan ko'rsatilgan; Responses API'ni avtomatik yozishini o'zingiz tekshiring. Yozmasa, `@observe()` dekoratori bilan o'z funksiyalaringizni kuzating, bu baribir ishlaydi.

**Minimal o'zingiz qilsangiz:** har so'rovda Postgres'dagi `llm_logs` jadvaliga yozing: vaqt, model, input_tokens, output_tokens, cached_tokens, latency_ms, savol, topilgan chunk id'lari, javob. Bu ham observability.

**Nimani kuzatish kerak:** narx (kunlik), latency (p95), xatolar soni, "bilmayman" ulushi, foydalanuvchi bergan 👍/👎.

**Qiyinligi:** oson.

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
