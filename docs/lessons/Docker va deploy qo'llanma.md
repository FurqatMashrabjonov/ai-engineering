---
tags: [lesson, production]
status: o'rganilmoqda
---
# Docker va deploy

Xarita: [[Production (xarita)]] · [[Learning app]] · [[Home]]

**Nima:** Ilovani kompyuteringizdan internetga chiqarish. Docker'ni allaqachon `rag-db` uchun ishlatgansiz; endi ilovaning o'zini ham konteyner qilasiz va `docker compose` bilan ikkalasini birga ko'tarasiz.

**Dockerfile (uv bilan):**

```dockerfile
FROM python:3.13-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev
COPY src ./src
CMD ["uv", "run", "uvicorn", "ai_engineering.api:app", "--host", "0.0.0.0", "--port", "8000"]
```

**compose.yaml:**

```yaml
services:
  db:
    image: pgvector/pgvector:pg17
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: rag
    volumes: [pgdata:/var/lib/postgresql/data]
  api:
    build: .
    env_file: .env
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/rag
    ports: ["8000:8000"]
    depends_on: [db]
volumes:
  pgdata:
```

Ishga tushirish: `docker compose up --build`. E'tibor bering: konteyner ichida baza manzili `localhost:5433` emas, `db:5432`.

**Qayerga deploy:** Railway, Render, Fly.io (Docker'ni to'g'ridan oladi) yoki oddiy VPS + `docker compose`. Laravel'ni qayerga deploy qilgan bo'lsangiz, o'sha yerga ham bo'ladi.

**Tekshiruv ro'yxati:**

- `.env` image ichiga tushmasin (`.dockerignore`ga qo'shing), kalitlar platformaning secrets bo'limida.
- Healthcheck endpoint: `GET /health`.
- OpenAI uchun `timeout` va `max_retries`ni aniq qo'ying (LLM API qo'llanmasi, 7-bo'lim).

**Qiyinligi:** oson. Bu yerda siz allaqachon kuchlisiz.

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
