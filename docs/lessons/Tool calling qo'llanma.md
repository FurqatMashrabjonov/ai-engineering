---
tags: [lesson, agent]
status: o'rganilmoqda
---
# Tool calling: model funksiyangizni chaqiradi

Xarita: [[Agentlar (xarita)]] · [[Learning app]] · [[Home]]

**Nima:** Siz modelga funksiyalar ro'yxatini (nomi, tavsifi, parametrlari) berasiz. Model o'zi kodni ishlatmaydi: "`search_docs`ni shu argument bilan chaqir" deb javob qaytaradi. Funksiyani **siz** ishlatasiz va natijani modelga qaytarasiz. Agentlarning hammasi shu mexanizm ustiga qurilgan.

**Oqim (Responses API):**

1. `tools=[...]` bilan so'rov yuborasiz.
2. Javobda `type == "function_call"` bo'lgan element keladi: `name`, `arguments` (JSON matn), `call_id`.
3. Funksiyani ishlatasiz.
4. `response.output`ni va `{"type": "function_call_output", "call_id": ..., "output": "..."}`ni input ro'yxatiga qo'shib, qayta so'rov yuborasiz.
5. Model yakuniy matnni beradi (yoki yana tool chaqiradi).

```python
import json

tools = [{
    "type": "function",
    "name": "search_docs",
    "description": "Foydalanuvchi yuklagan hujjatlardan qidiradi.",
    "parameters": {
        "type": "object",
        "properties": {"query": {"type": "string"}},
        "required": ["query"],
        "additionalProperties": False,
    },
    "strict": True,
}]

def run_tool(name, args):
    if name == "search_docs":
        return build_context(search(args["query"], k=5))
    return f"Noma'lum tool: {name}"

input_list = [{"role": "user", "content": "Kitobda pgvector haqida nima deyilgan?"}]
resp = client.responses.create(model=MODEL, tools=tools, input=input_list)
input_list += resp.output

for item in resp.output:
    if item.type == "function_call":
        result = run_tool(item.name, json.loads(item.arguments))
        input_list.append({"type": "function_call_output",
                           "call_id": item.call_id, "output": result})

resp = client.responses.create(model=MODEL, tools=tools, input=input_list)
print(resp.output_text)
```

**E'tibor beradigan joylar:**

- `strict: True` bo'lsa, argumentlar sxemaga aniq mos keladi. Buning uchun `additionalProperties: False` va barcha maydonlar `required`da bo'lishi shart.
- `description` prompt hisoblanadi: model qaysi tool'ni qachon chaqirishni shundan biladi. Aniq yozing.
- Modelga ishonmang: argumentlarni tekshiring. Masalan, foydalanuvchi faqat o'z fayllarini qidira olishi kerak (`user_id`ni modeldan emas, sessiyadan oling).
- RAG'ni tool qilish "agentic RAG" deyiladi: model qachon va nima qidirishni o'zi hal qiladi.

**Qiyinligi:** o'rta. Lekin RAG'dan osonroq: yangi g'oya bitta, qolgani for-sikl.

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
