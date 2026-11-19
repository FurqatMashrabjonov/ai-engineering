---
tags: [lesson, production]
status: o'rganilmoqda
---
# FastAPI va streaming: ilovani API qilish (v4)

Xarita: [[Production (xarita)]] · [[Learning app]] · [[Home]]

**Nima:** Hozircha kodingiz notebook ichida. Ilova bo'lishi uchun uni HTTP API qilasiz: `POST /files` (fayl yuklash), `POST /ask` (savol). Laravel'dagi route + controller bilan bir xil g'oya. FastAPI Python'da LLM ilovalar uchun eng ko'p ishlatiladigan framework.

**Atamalar:**

| Atama | Ma'nosi | Laravel'dagi o'xshashi |
| --- | --- | --- |
| Endpoint | URL + method | Route |
| Pydantic model | So'rov/javob sxemasi | FormRequest + Resource |
| `async def` | Kutish paytida boshqa so'rovlarga xizmat qiladi | Queue/async'ga yaqin |
| SSE (Server-Sent Events) | Serverdan brauzerga matnni bo'lak-bo'lak yuborish | Broadcasting'ning soddasi |
| Uvicorn | Ilovani ishga tushiradigan server | `php artisan serve` |

```python
# src/ai_engineering/api.py
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
from openai import AsyncOpenAI

app = FastAPI()
client = AsyncOpenAI()

class AskIn(BaseModel):
    question: str

@app.post("/ask")
async def ask(body: AskIn):
    async def gen():
        stream = await client.responses.create(
            model=MODEL, input=body.question, stream=True
        )
        async for event in stream:
            if event.type == "response.output_text.delta":
                yield f"data: {event.delta}\n\n"
        yield "data: [DONE]\n\n"
    return StreamingResponse(gen(), media_type="text/event-stream")
```

Ishga tushirish: `uv add fastapi uvicorn` keyin `uv run uvicorn ai_engineering.api:app --reload`. `http://localhost:8000/docs` sahifasida Swagger avtomatik chiqadi.

**E'tibor beradigan joylar:**

- Endpoint ichida sinxron `OpenAI()` emas, `AsyncOpenAI()` ishlating. Aks holda bitta sekin so'rov butun serverni to'xtatib turadi.
- Fayl yuklash uchun `UploadFile` ishlatiladi; katta faylni indekslash so'rov ichida emas, fon vazifasida bo'lishi kerak (`BackgroundTasks`, keyinroq navbat).
- Streaming'da xato o'rtada chiqishi mumkin. Generator ichida `try/except` qilib, xatoni ham `data:` qatori qilib yuboring.

**Qiyinligi:** oson-o'rta. Laravel tajribangiz shu yerda eng ko'p yordam beradi.

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
