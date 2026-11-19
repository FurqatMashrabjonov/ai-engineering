---
tags: [lesson, eval]
status: o'rganilmoqda
---
# Eval: o'zgarish yaxshilandimi yoki buzildimi

Xarita: [[Eval (xarita)]] · [[Learning app]] · [[Home]]

**Nima:** O'zgarish (yangi chunk hajmi, boshqa model, yangi prompt) qilganda, "yaxshi bo'ldi shekilli" deb emas, raqam bilan bilish. Eval = savollar to'plami + o'lchovlar + har o'zgarishda qayta ishga tushirish.

**Haqiqat:** sanoatda ham buni kam qilishadi. DataTalks so'rovnomasida: avtomatik eval 17%, qo'lda 38%, umuman yo'q 31%. Ya'ni eval qilsangiz, ko'pchilikdan oldinga o'tasiz. Ish suhbatlarida eng ko'p so'raladigan mavzulardan biri.

**Learning app uchun eval to'plami:**

1. **Golden set:** o'z hujjatlaringizdan 30-50 ta savol. Har birida: savol, to'g'ri javob, to'g'ri manba (fayl + sahifa). Bir qismi hujjatda **yo'q** narsalar haqida bo'lsin ("bilmayman" testi).
2. **O'lchovlar:**

| O'lchov | Nimani o'lchaydi | Qanday |
| --- | --- | --- |
| Recall@k | To'g'ri bo'lak top-k ichidami | RAG qo'llanmasidagi `eval_retrieval()` |
| Faithfulness | Javob manbaga sodiqmi | [[LLM-as-a-judge qo'llanma]] dagi `judge()` |
| Correctness | Javob to'g'ri javobga mosmi | Judge yoki qo'lda |
| Refusal | Yo'q narsaga "bilmayman" dedimi | Qoida bilan |
| Narx va tezlik | So'rov uchun token/soniya | `usage` va vaqt |

3. **Natijani jadvalda saqlang:** sana, o'zgarish, har bir o'lchov. Masalan: "chunk 800 → 500: recall@5 0.72 → 0.81, faithfulness o'zgarmadi".

```python
def run_eval(golden, answer_fn):
    rows = []
    for g in golden:
        out = answer_fn(g["question"])
        v = judge(g["question"], out["context"], out["text"])
        rows.append({"id": g["id"], "faithful": v.faithful,
                     "refused_ok": (g["answer"] is None) == ("bilmayman" in out["text"].lower())})
    n = len(rows)
    return {"faithfulness": sum(r["faithful"] for r in rows) / n,
            "refusal_acc": sum(r["refused_ok"] for r in rows) / n}
```

**Tayyor kutubxonalar:** Ragas, DeepEval, promptfoo. Avval o'zingiz 50 qatorda yozing, keyin kutubxona nima qilayotganini tushunasiz.

**Qiyinligi:** o'rta. Kod oson; qiyini yaxshi golden set tuzish.

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
