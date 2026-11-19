---
tags: [lesson, eval]
status: o'rganilmoqda
---
# Testlar va LLM-as-a-judge

Xarita: [[Eval (xarita)]] · [[Learning app]] · [[Home]]

**Muammo:** LLM javobi har safar biroz boshqacha. `assert answer == "..."` ishlamaydi. Shuning uchun testlar ikki qavatga bo'linadi.

| Qavat | Nimani tekshiradi | Qanday |
| --- | --- | --- |
| Oddiy unit test | Chunking, RRF, prompt yig'ish, JSON parse | `pytest`, LLM'siz, tez va tekin |
| Qoidaga asoslangan | Javobda manba raqami bormi, JSON sxemaga mosmi, "bilmayman" kerak joyda chiqdimi | Kod bilan, LLM javobi ustida |
| LLM-as-a-judge | Javob to'g'rimi, manbaga sodiqmi (faithfulness) | Boshqa LLM chaqiruvi baho beradi |

**LLM-judge misoli (structured output bilan):**

```python
class Verdict(BaseModel):
    faithful: bool          # javobdagi har bir da'vo manbada bormi
    reason: str

def judge(question, context, answer) -> Verdict:
    resp = client.responses.parse(
        model=MODEL,
        instructions=("Sen qat'iy tekshiruvchisan. JAVOBdagi har bir da'vo "
                      "MANBALARda bor-yo'qligini tekshir. Bitta asossiz da'vo bo'lsa ham faithful=false."),
        input=f"SAVOL: {question}\n\nMANBALAR:\n{context}\n\nJAVOB:\n{answer}",
        text_format=Verdict,
    )
    return resp.output_parsed
```

**Judge'ga ishonish qoidalari:**

- Bitta savolni bering ("sodiqmi?"), 1-10 ball emas. Ha/yo'q baholar barqarorroq.
- Avval 20-30 ta javobni o'zingiz qo'lda baholang, keyin judge bilan solishtiring. Ko'p hollarda mos kelsa, judge'ga ishonsa bo'ladi.
- Judge modelini va promptini o'zgartirsangiz, bu solishtiruvni qayta qiling.

**Qiyinligi:** oson-o'rta.

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
