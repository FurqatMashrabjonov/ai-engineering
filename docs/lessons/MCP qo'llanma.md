---
tags: [lesson, agent]
status: o'rganilmoqda
---
# MCP: tool'larni standart qilib berish

Xarita: [[Agentlar (xarita)]] · [[Learning app]] · [[Home]]

**Nima:** Model Context Protocol. Tool'larni bitta standart formatda yozasiz, keyin ularni istalgan MCP mijozi (Claude Desktop, Cursor, ChatGPT, o'zingizning agentingiz) ulab ishlata oladi. "AI uchun USB-C" deb tushuntirishadi. [[Tool calling qo'llanma]] dagi tool calling bitta ilova ichida; MCP esa tool'ni alohida server qilib ajratadi.

**Atamalar:**

| Atama | Ma'nosi |
| --- | --- |
| MCP server | Tool, resource va prompt'larni taqdim etadigan dastur |
| MCP client | Serverga ulanadigan ilova (Claude Desktop va h.k.) |
| Tool | Model chaqiradigan funksiya |
| Resource | Model o'qiydigan ma'lumot (fayl, yozuv) |
| stdio / HTTP | Ulanish turi: lokal jarayon yoki tarmoq orqali |

**Python SDK v2 (hozirgi versiya).** Diqqat: eski darsliklarda `FastMCP` deb yoziladi. v2'da u `MCPServer` deb nomlangan, eski import yo'li olib tashlangan.

```python
# src/ai_engineering/mcp_server.py
from mcp.server import MCPServer

mcp = MCPServer("Learning app")

@mcp.tool()
def search_notes(query: str) -> str:
    """Foydalanuvchi yuklagan hujjatlardan qidiradi."""
    return build_context(search(query, k=5))

if __name__ == "__main__":
    mcp.run()          # standart: stdio
```

- O'rnatish: `uv add mcp`
- Sinash: `uv run mcp dev src/ai_engineering/mcp_server.py` (MCP Inspector ochiladi, Node/npx kerak)
- Claude Desktop'ga qo'shish: `uv run mcp install src/ai_engineering/mcp_server.py --name "Learning app"`

**Learning app'da:** hujjatlaringiz qidiruvini MCP server qilasiz. Shunda Claude Desktop yoki boshqa agent sizning bazangizdan qidira oladi. Portfolio uchun yaxshi "wow" nuqta.

**Xavfsizlik:** begona MCP serverni o'rnatish = begona kodni ishga tushirish. Faqat ishonchli manbadan oling ([[Guardrails va OWASP qo'llanma]], LLM03 Supply chain).

**Qiyinligi:** oson. Tool calling'ni tushungan bo'lsangiz, bir kechada bo'ladi.

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
