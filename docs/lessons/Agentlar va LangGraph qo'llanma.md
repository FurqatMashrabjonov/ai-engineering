---
tags: [lesson, agent]
status: o'rganilmoqda
---
# Agentlar va LangGraph

Xarita: [[Agentlar (xarita)]] · [[Learning app]] · [[Home]]

**Agent nima:** tool calling'ni sikl ichida aylantirish. Model tool chaqiradi, natijani ko'radi, yana chaqiradi... vazifa tugaguncha. Anthropic'ning "Building effective agents" maqolasidagi asosiy fikr: avval eng oddiy yechimni (bitta LLM chaqiruvi, keyin oddiy workflow) sinang, agentga faqat kerak bo'lganda o'ting.

**Workflow va agent farqi:**

|  | Workflow | Agent |
| --- | --- | --- |
| Qadamlar | Kodda oldindan yozilgan | Model o'zi tanlaydi |
| Bashorat | Yuqori | Past |
| Qachon | Vazifa aniq (fayl → xulosa → test) | Qadamlar oldindan noma'lum (tadqiqot) |

**Avval o'zingiz yozing (30 qator):**

```python
def run_agent(question, max_steps=6):
    input_list = [{"role": "user", "content": question}]
    for _ in range(max_steps):
        resp = client.responses.create(model=MODEL, tools=tools, input=input_list)
        input_list += resp.output
        calls = [i for i in resp.output if i.type == "function_call"]
        if not calls:
            return resp.output_text          # tool so'ramadi = tugadi
        for c in calls:
            input_list.append({"type": "function_call_output", "call_id": c.call_id,
                               "output": run_tool(c.name, json.loads(c.arguments))})
    return "Qadamlar limiti tugadi."
```

`max_steps` majburiy: usiz agent cheksiz aylanib pul yeyishi mumkin.

**Keyin LangGraph:** agent jarayonini graf (tugunlar + o'tishlar) qilib yozadigan kutubxona. Agent framework'lari orasida eng ko'p yuklab olinadigani (oyiga ~34.5 mln; OpenAI Agents SDK ~10 mln, CrewAI ~5 mln). Sizga beradigan narsalari:

- **State va checkpointer:** suhbat holati saqlanadi, `thread_id` bo'yicha davom ettiriladi.
- **Human-in-the-loop:** `interrupt()` grafni to'xtatadi, odam tasdiqlagach `Command(resume=...)` bilan davom etadi.

```python
from langgraph.types import interrupt, Command
from langgraph.checkpoint.memory import InMemorySaver

def confirm_delete(state):
    ok = interrupt({"savol": f"{state['file']} o'chirilsinmi?"})
    return {"deleted": ok == "ha"}

graph = builder.compile(checkpointer=InMemorySaver())
cfg = {"configurable": {"thread_id": "user-1"}}
res = graph.invoke({"file": "kitob.pdf"}, cfg)   # res["__interrupt__"] da savol
graph.invoke(Command(resume="ha"), cfg)          # davom etadi
```

Muhim: davom etganda tugun **boshidan** qayta ishlaydi. Shuning uchun `interrupt()`dan oldingi qism ikki marta ishlasa ham zarari bo'lmasligi kerak.

**Learning app'da:** "O'qish rejasi tuz" agenti: hujjatlarni qidiradi, mavzularni ajratadi, har biriga test tuzadi, natijani saqlashdan oldin sizdan tasdiq so'raydi.

**Qiyinligi:** o'rta-qiyin. O'z siklingizni yozib tushunmasdan LangGraph'ga o'tmang.

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
