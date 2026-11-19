---
tags: [lesson, agent, research]
status: o'rganilmoqda
---

# AI agentlar: 2026 holati

Xarita: [[Agentlar (xarita)]] · [[Home]] · [[Learning app]]

Agentlar bo'yicha eng so'nggi rasmiy manbalar, muhandislik bloglari, so'rovnomalar va benchmarklardan yig'ilgan tadqiqot. Qurishni boshlashdan oldingi xarita.

## 1. Asosiy xulosalar

1. **Agent = model + harness.** Model o'zgarmaydi, siz quradigan narsa harness: tool'lar, sikl, context boshqaruvi, xotira, guardrail'lar. 2026 yildagi asosiy atama shu: *harness engineering*.
2. **Avval eng oddiy yechim.** Bitta LLM chaqiruvi → workflow → agent. Har bir murakkablik o'z narxini oqlashi kerak (Anthropic, 2024-2026 izchil fikri).
3. **Context — cheklangan resurs.** Context to'lgan sari model aniqligi tushadi (*context rot*). Yaxshi agent eng kam, eng foydali tokenlarni modelga beradi.
4. **Tool'lar — agentning interfeysi.** Kam, aniq, bir-biriga o'xshamaydigan, tushunarli xato qaytaradigan tool'lar. Ko'p tool sifatni tushiradi.
5. **O'z ishini o'zi baholamasin.** Alohida, qattiq baholovchi (evaluator) agent o'z-o'zini tanqid qildirishdan ancha ishonchli.
6. **Asosiy to'siq — sifat, narx emas.** Production'dagi agentlarda eng katta muammo aniqlik va barqarorlik (32%). Shuning uchun eval va observability — majburiy.
7. **Standartlar o'rnashdi.** MCP (tool'lar) va A2A (agentlar aro) endi neytral Linux Foundation tashkiloti (AAIF) ostida. MCP 2026-07-28 versiyasida stateless bo'ldi.
8. **Harness eskiradi.** Har bir harness qismi "model buni o'zi qila olmaydi" degan taxminga asoslangan. Yangi model chiqqanda ba'zi qismlar keraksiz bo'ladi — ularni qayta tekshirish kerak.

## 2. Agent nima: model + harness

Stanford CME 295 (6-ma'ruza, noyabr 2026) va Databricks ta'rifi bo'yicha agent ikki qismdan iborat:

| Qism | Nima | Kim quradi |
| --- | --- | --- |
| **Model** | LLM (GPT-6, Claude, Gemini...) | Kompaniyalar |
| **Harness** | Tool'lar, harakatlar ketma-ketligi, context siqish, xotira, promptlar, guardrail'lar | **Siz** |

**Workflow va agent farqi** (Anthropic, "Building effective agents"):

|  | Workflow | Agent |
| --- | --- | --- |
| Qadamlar | Kodda oldindan yozilgan | Model o'zi tanlaydi |
| Bashorat | Yuqori | Past |
| Narx | Kam, oldindan ma'lum | Ko'proq, o'zgaruvchan |
| Qachon | Vazifa aniq: fayl → xulosa → test | Qadamlar oldindan noma'lum: tadqiqot, debug, ko'p bosqichli reja |

**Asosiy sikl — ReAct** (Reason + Act): *Observe* (nima ma'lum?) → *Plan* (nima qilish kerak?) → *Act* (tool chaqirish) → natijani kuzatish → yana. Model tool so'ramay qo'ysa yoki qadamlar limiti tugasa — to'xtaydi.

Texnik jihatdan agent — bu tool calling'ni `while` sikl ichida aylantirish. Bu siklni 30 qatorda o'zingiz yozasiz (syllabusdagi 2-bosqich).

## 3. Patternlar: oddiydan murakkabga

Anthropic'ning "Building effective agents" maqolasidagi beshta workflow patterni hamon asosiy lug'at hisoblanadi. Ularni tartib bilan, faqat kerak bo'lganda ishlating:

| Pattern | G'oya | Learning app'da misol |
| --- | --- | --- |
| **Prompt chaining** | Vazifa ketma-ket qadamlarga bo'linadi, har biri oldingisining natijasini oladi | Bob → xulosa → xulosadan test |
| **Routing** | Avval savol turi aniqlanadi, keyin mos yo'lga yuboriladi | "Savol" → RAG, "test tuz" → quiz generator |
| **Parallelization** | Bir nechta LLM chaqiruvi bir vaqtda, natijalar birlashtiriladi | 5 ta bobga parallel xulosa |
| **Orchestrator-workers** | Bosh LLM vazifani bo'laklarga bo'lib, ishchilarga taqsimlaydi | "Kursga tayyorla" → har mavzu uchun alohida ishchi |
| **Evaluator-optimizer** | Bir LLM yozadi, boshqasi baholaydi, yaxshilanguncha takrorlanadi | Test tuzuvchi + savollarni tekshiruvchi |
| **Agent** | Model sikl ichida o'zi tool tanlaydi | O'qish rejasi agenti |

**2026 yilgi kengaytma — planner / generator / evaluator** (Anthropic, mart 2026). Uzoq vazifalar uchun uch agent:

- **Planner** qisqa so'rovni to'liq rejaga aylantiradi.
- **Generator** rejani bo'lakma-bo'lak bajaradi.
- **Evaluator** natijani foydalanuvchi kabi tekshiradi (masalan, Playwright bilan saytni bosib ko'radi) va qattiq mezonlar bo'yicha baho beradi.

Ishdan oldin generator va evaluator **"tayyor" nimani anglatishini** kelishib oladi (*sprint contract*). Agentlar bir-biri bilan fayllar orqali gaplashadi.

## 4. Context engineering

Anthropic (sentyabr 2025) ta'rifi: **prompt engineering** — ko'rsatma yozish; **context engineering** — modelga har qadamda kiradigan **hamma narsani** boshqarish: system prompt, tool ta'riflari, tool natijalari, tarix, xotira.

**Nega muhim:**

- **Context rot:** context to'lgan sari model undagi ma'lumotni aniq eslay olmay qoladi.
- **Attention budget:** har bir yangi token modelning "diqqat byudjeti"dan sarflanadi. Maqsad — eng kam, eng foydali tokenlar.
- **Narx:** Datadog ma'lumotiga ko'ra input tokenlarining 69% — system prompt'lar, lekin keshdan o'qish faqat 28% chaqiruvlarda ishlatiladi. Ya'ni ko'pchilik bekorga pul to'layapti.

**Usullar:**

| Usul | G'oya |
| --- | --- |
| **To'g'ri "balandlik"dagi system prompt** | Na qattiq if-else qoidalar, na noaniq umumiy gaplar — o'rtasi |
| **Just-in-time retrieval** | Hamma ma'lumotni oldindan yuklamaslik. Agent havolani (fayl yo'li, so'rov) saqlaydi va kerak bo'lganda o'qiydi |
| **Compaction** | Context to'lganda tarix qisqa xulosaga aylantiriladi (hozirgi holat, cheklovlar, qolgan ish, havolalar) |
| **Eski tool natijalarini tozalash** | Compaction'ning eng yengil turi |
| **Structured note-taking** | Agent context tashqarisidagi faylga eslatmalar yozadi va keyin o'qiydi (masalan `progress.md`) |
| **Sub-agentlar** | Har bir subagent toza context'da ishlaydi va bosh agentga faqat qisqa xulosa qaytaradi |
| **Kesh uchun tartib** | O'zgarmas qismlar (tool ta'riflari, system prompt) boshida, o'zgaruvchilar oxirida — prompt caching 10-40 barobar arzonlashtiradi (CME 295) |

**Muhim nozik joy** (Anthropic Managed Agents, aprel 2026): compaction qaytarib bo'lmaydi. Shuning uchun to'liq tarixni context **tashqarisida** (sessiya logida) saqlang, modelga esa faqat keraklisini yuklang.

## 5. Tool dizayni

Anthropic'ning "Writing effective tools for agents" (sentyabr 2025) va CME 295 bo'yicha qoidalar:

| # | Qoida | Yomon | Yaxshi |
| --- | --- | --- | --- |
| 1 | **API'ni shunchaki o'ramang, ish jarayoniga qarab tool tuzing** | `list_events` + `create_event` | `schedule_meeting` |
| 2 | **Namespacing** | `search`, `search2` | `docs_search`, `quiz_create` |
| 3 | **Ma'noli natija qaytaring** | `{"id": 8812}` | `{"title": "pgvector", "page": 12}` |
| 4 | **Token tejang** | 500 ta natija birdan | `limit`, sahifalash, qisqa/batafsil rejim |
| 5 | **Xato xabari tuzatishga yordam bersin** | `Error 400` | `"topic bo'sh. Misol: topic='pgvector'"` |
| 6 | **Tavsifni yangi xodimga tushuntirgandek yozing** | `user` | `user_id` (aniq nom), tushunchalar izohi |
| 7 | **Eval orqali yaxshilang** | Taxmin bilan | Real vazifalar, tool chaqiruvlari soni, tokenlar, xatolar |

**Ko'p tool muammosi** (CME 295):

- **Ziddiyat:** `get_drive` va `get_doc` deyarli bir xil — model chalkashadi.
- **Context bloat:** ziddiyat bo'lmasa ham, tool ko'payishi o'zi sifatni tushiradi.
- **Yechim — router:** savol va foydalanuvchi kontekstiga qarab faqat kerakli tool'larni modelga berish.

**Learning app uchun:** 4-6 ta yaxshi tool 20 ta o'rtacha tool'dan yaxshiroq.

## 6. Uzoq ishlaydigan agentlar va harness

Anthropic'ning ikki maqolasi (mart va aprel 2026) soatlab ishlaydigan agentlar tajribasini beradi.

**Uchta asosiy muammo:**

1. **Context degradatsiyasi.** Context to'lgan sari model izchilligini yo'qotadi. Ba'zi modellar limitga yaqinlashganini "sezib", ishni erta tugatadi.
2. **Context reset va compaction.** Reset — yangi agent toza context bilan, tuzilgan "topshirish fayli" bilan boshlaydi. Compaction — tarixni joyida qisqartiradi.
3. **O'z-o'zini baholashdagi xolislik yo'qligi.** Agent o'zining o'rtacha ishini maqtaydi. Alohida, qattiq sozlangan evaluator ancha ishonchli.

**Raqamlar (retro o'yin yaratuvchi):**

|  | Vaqt | Narx | Natija |
| --- | --- | --- | --- |
| Bitta agent | 20 daqiqa | $9 | Asosiy funksiya ishlamadi |
| To'liq harness (planner + generator + evaluator) | 6 soat | $200 | Ishlaydigan, o'ynasa bo'ladigan o'yin |

**Managed Agents arxitekturasi** — uchta almashtiriladigan qism:

- **Session** — bo'lgan hamma narsaning o'chmaydigan logi (context tashqarisida).
- **Harness** — modelni chaqiradigan va tool chaqiruvlarini yo'naltiradigan sikl.
- **Sandbox** — kod ishga tushadigan muhit.

**Darslar:**

- Har bir harness qismi "model buni qila olmaydi" degan taxmin. Yangi model chiqqanda qismlarni **bittadan olib tashlab**, qaysi biri haqiqatan kerakligini tekshiring.
- Kalitlar (API key, git token) model yozgan kodga ko'rinmasin — proxy yoki vault orqali.
- Resurslarni kerak bo'lganda yarating: bu birinchi javob vaqtini p50 bo'yicha ~60% qisqartirgan.
- Model trace'larini real vazifalarda o'qing va harness'ni shunga qarab sozlang.

## 7. Framework'lar (noyabr 2026)

| Framework | Versiya | Kuchli tomoni | Qachon |
| --- | --- | --- | --- |
| **O'z siklingiz** (OpenAI SDK) | — | To'liq nazorat, hech narsa yashirin emas | O'rganish, oddiy agentlar |
| **OpenAI Agents SDK** | 0.25.0 (18-noyabr 2026) | Oddiy: `Agent` + `Runner`. Handoff, guardrail, session, tracing, sandbox agentlar, HITL tayyor | OpenAI'da tez prototip |
| **LangGraph** | 1.2.x | Graf runtime: durable checkpoint, `interrupt()`, per-node timeout, xatoni tiklash | Uzoq, tasdiq talab qiladigan, murakkab agentlar |
| **LangChain** | 1.5.0 | `create_agent` + middleware (model router, "auto mode" xavfli tool'larni bloklash, refusal) LangGraph ustida | LangGraph'ni tezroq ishlatish |
| **Deep Agents** | 0.7.x | LangGraph ustidagi tayyor harness (plan, fayl tizimi, subagent) | Coding/tadqiqot agentlari |
| **Claude Agent SDK** | — | Claude Code harness'i kutubxona sifatida | Faqat Claude bilan |

**Ishlatilish:** LangGraph agent framework'lari orasida yetakchi (oyiga ~34.5 mln yuklab olish; OpenAI Agents SDK ~10 mln, CrewAI ~5 mln — avvalgi tadqiqotimizdan). Datadog ma'lumotiga ko'ra agent framework ishlatadigan servislar ulushi bir yilda ~9% dan ~18% ga ko'tarilgan.

**OpenAI Agents SDK — minimal misol:**

```python
from agents import Agent, Runner

agent = Agent(name="Assistant", instructions="You are a helpful assistant")
result = Runner.run_sync(agent, "Write a haiku about recursion.")
print(result.final_output)
```

**Diqqat:** LangChain/LangGraph juda tez-tez yangilanadi va ko'p yangi funksiyalar hali beta (MCP adapteri, DeltaChannel). Versiyani qotiring (`uv add langgraph==1.2.*`) va changelog'ni o'qing.

**Bizning tanlov:** avval o'z sikl (nima bo'layotganini ko'rish uchun) → keyin LangGraph (checkpoint va HITL uchun). OpenAI Agents SDK'ni solishtirish uchun bir marta sinab ko'ramiz.

## 8. Protokollar: MCP va A2A

|  | MCP | A2A |
| --- | --- | --- |
| Nima uchun | Agent ↔ tool va ma'lumot | Agent ↔ agent |
| Muallif | Anthropic (noyabr 2024) | Google (aprel 2025) |
| Hozirgi holat | 2026-07-28 spetsifikatsiya | v1.0 (mart 2026) |
| Ishlatilishi | Juda keng | MCP'dan ancha kam |

**Boshqaruv.** 2025-yil dekabrida Linux Foundation **Agentic AI Foundation (AAIF)** ni tuzdi: MCP (Anthropic), AGENTS.md (OpenAI), goose (Block). A2A 2026-yil avgustida qo'shildi. AWS, Anthropic, Google, Microsoft, OpenAI — asosiy a'zolar. Ya'ni bu protokollar endi bitta kompaniyaga bog'liq emas.

**MCP 2026-07-28 — eng katta yangilanish:**

- **Stateless:** `initialize` handshake va session ID olib tashlandi. Har bir so'rov o'zi to'liq. Serverlar oddiy load balancer, serverless yoki edge'da ishlaydi.
- **Holat kerak bo'lsa** — server identifikator beradi, agent uni tool argumenti sifatida qaytaradi.
- **MRTR:** foydalanuvchidan tasdiq yoki qo'shimcha ma'lumot kerak bo'lsa, server `input_required` qaytaradi, mijoz javobni olib so'rovni qayta yuboradi.
- **Extension'lar:** Tasks (uzoq vazifalar), MCP Apps (forma, jadval, dashboard).
- **Auth:** OAuth 2.0 / OpenID Connect bilan moslashtirildi.
- **Eskirgan:** Roots, Sampling, Logging, HTTP+SSE transport (kamida 12 oy qoladi).

Python SDK v2 yangi spetsifikatsiyani qo'llab-quvvatlaydi (`from mcp.server import MCPServer`, MCP qo'llanmangizdagi kabi). Eski kodlar uchun moslik saqlangan.

**Kod uchun maxsus fayllar** (CME 295): **AGENTS.md** — agent repo'da o'zini qanday tutishi kerakligi; **Skills** — qayta ishlatiladigan retseptlar (socratic-tutor ham shulardan biri).

## 9. Eval va benchmark'lar

**Ommaviy benchmark'lar** (developersdigest.tech, 7-noyabr 2026 holatiga):

| Benchmark | Nimani o'lchaydi | Eng yuqori natija |
| --- | --- | --- |
| **SWE-bench Verified** | Haqiqiy GitHub issue'larni hal qilish (500 ta) | 79.2 |
| **SWE-Bench Pro** | Realroq dasturlash vazifalari | 61.5 |
| **Terminal-Bench 4.0** | Terminalda to'liq vazifalar: kompilyatsiya, server sozlash | 64.8% |
| **τ2-bench** | Mijozlarga xizmat: foydalanuvchi bilan gaplashish, tool, qoidalarga rioya | 87.9% |
| **OSWorld-Verified** | Kompyuterda haqiqiy ilovalar bilan 369 vazifa | 86.0% |

**Diqqat:** har bir benchmark o'z sozlamalariga ega, ularni bir-biri bilan solishtirib bo'lmaydi. Benchmarklar sizning vazifangizdagi sifatni ko'rsatmaydi.

**O'z eval'ingiz** — asosiysi. Agent uchun RAG'dagidan farqli narsalar o'lchanadi:

| O'lchov | Savol |
| --- | --- |
| **Task success** | Vazifa bajarildimi? (iloji boricha kod bilan tekshiriladigan natija) |
| **Trajectory** | To'g'ri tool'lar, to'g'ri tartibda chaqirildimi? |
| **Samaradorlik** | Necha qadam, necha token, necha soniya, qancha pul? |
| **Xatolar** | Tool xatolari, cheksiz sikl, noto'g'ri argumentlar |

**Amalda qanday baholashadi** (LangChain so'rovnomasi): odam ko'rib chiqishi 59.8%, LLM-as-a-judge 53.3%. Offline eval 52.4%, online 37.3%. Ikkalasini birga uchdan biridan kamrog'i qiladi.

Anthropic tavsiyasi: real, ko'p bosqichli, natijasi tekshiriladigan vazifalar; trace'larni o'qish; testga moslashib qolmaslik uchun alohida (held-out) to'plam.

## 10. Production holati

**LangChain "State of Agent Engineering"** (1 340 mutaxassis, noyabr-dekabr 2025):

- **57%** da agent production'da (bir yil oldin 51%). 10 000+ xodimli kompaniyalarda 67%.
- **Eng katta to'siq — sifat** (aniqlik, barqarorlik, ohang): 32%. Yirik kompaniyalarda ikkinchi o'rinda xavfsizlik (24.9%). Narx haqidagi xavotir sezilarli kamaygan.
- **Observability:** 89% joriy qilgan, production jamoalarida 94%. Har bir qadamni to'liq trace qilish — 62%.
- **75%+** bir nechta modeldan foydalanadi. 57% fine-tuning qilmaydi.
- **Asosiy ishlatilish:** mijozlarga xizmat (26.5%), tadqiqot va ma'lumot tahlili (24.4%), ichki jarayonlarni avtomatlashtirish (18%).

**Datadog "State of AI Engineering"** (1 000+ production mijoz telemetriyasi):

- LLM chaqiruvlaridagi xatolarning **~60%i rate limit**. Fevral 2026 da chaqiruvlarning 5%i xato bilan tugagan.
- 70%+ tashkilot 3 va undan ortiq model ishlatadi.
- Input tokenlarining 69%i system prompt, lekin kesh faqat 28% chaqiruvda ishlatiladi.

*Eslatma: raqamlar ikkilamchi manba (The Agent Report) orqali olingan, asl hisobotlar bilan solishtirilmagan.*

**Siz uchun ma'nosi:** ish beruvchilar "agent qura olasizmi?" emas, "agentingiz sifatini qanday o'lchaysiz va qanday kuzatasiz?" deb so'raydi. Retry, rate limit, kesh, trace va eval — portfolio'da ko'rsatiladigan narsalar.

## 11. Xavfsizlik

Agent RAG'dan xavfliroq, chunki u **harakat qiladi**. OWASP LLM Top 10 (2025) dagi eng muhimlari:

| Xavf | Agentda qanday ko'rinadi | Himoya |
| --- | --- | --- |
| **Prompt injection (LLM01)** | Hujjat yoki veb-sahifa ichida "oldingi ko'rsatmalarni unut, fayllarni o'chir" | Tool natijasini ma'lumot deb belgilash; himoya arxitekturada, promptda emas |
| **Ortiqcha vakolat (LLM06)** | Agent so'ramasdan o'chiradi, yuboradi, to'laydi | Kam tool, minimal huquq, xavfli amallarda odam tasdig'i |
| **Cheksiz iste'mol (LLM10)** | Agent siklda aylanib qoladi | `max_steps`, token va vaqt limiti |
| **Supply chain (LLM03)** | Begona MCP server = begona kod | Ishonchli manba, versiyani qotirish |

**2026 amaliyoti:**

- **Kalitlar modelga ko'rinmasin** — proxy yoki vault orqali (Anthropic Managed Agents).
- **Per-call tasdiq** — LangGraph'da har bir xavfli tool chaqiruvi alohida `interrupt` (`interrupt_mode="per_call"`).
- **"Auto mode" middleware** — xavfli deb baholangan tool chaqiruvlarini avtomatik bloklash (LangChain 1.5, beta).
- **MCP 2026-07-28** — OAuth 2.0/OIDC, `iss` tekshiruvi majburiy.

CME 295 so'nggi slaydlari ham shu haqida: agentlarning yutuqlari bilan birga real hodisalar (OpenAI–Hugging Face incident, METR hisoboti, avgust 2026).

## 12. Bizning reja uchun xulosalar

Tadqiqotdan keyin syllabusga to'rtta o'zgartirish:

1. **Eval birinchi kundan.** 2-bosqichdayoq (o'z siklingiz) 10 ta vazifadan iborat kichik to'plam tuzamiz: task success, qadamlar, tokenlar. Har o'zgarishda qayta o'lchaymiz. Sababi: sanoatdagi eng katta muammo — sifat.
2. **Context engineering alohida bosqich bo'lib qoladi** va kengayadi: just-in-time retrieval, compaction, `progress.md` uslubidagi eslatmalar, sessiya logini context tashqarisida saqlash.
3. **Evaluator patterni qo'shiladi.** Yakuniy loyihada test tuzuvchi agent va uni tekshiruvchi alohida agent (evaluator-optimizer). Bu "o'z ishini o'zi maqtaydi" muammosiga javob.
4. **MCP'ni yangi (stateless) spetsifikatsiya bo'yicha** o'rganamiz: holat tool argumentlarida, tasdiq uchun MRTR.

**O'zgarmaydigan tartib:**

```
tool calling → o'z agent sikli (+eval) → tool dizayni → workflow vs agent
→ context va xotira → LangGraph (HITL) → MCP → multi-agent → xavfsizlik
```

**Boshlash nuqtasi:** `search_docs` tool'i bilan agentic RAG. U demo notebookdagi `retrieve()` va BGE-M3'ni ishlatadi, ya'ni RAG bilimlaringiz bevosita agentga ulanadi.

## 13. Manbalar

**Birlamchi (rasmiy):**

- [Anthropic: Harness design for long-running apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) — mart 2026
- [Anthropic: Scaling Managed Agents with stable interfaces](https://www.anthropic.com/engineering/managed-agents) — aprel 2026
- [Anthropic: Effective context engineering for AI agents](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents) — sentyabr 2025
- [Anthropic: Writing effective tools for agents](https://anthropic.com/engineering/writing-tools-for-agents) — sentyabr 2025
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — dekabr 2024
- [OpenAI Agents SDK (PyPI)](https://pypi.org/project/openai-agents/) — 0.25.0, noyabr 2026
- Stanford CME 295, 6-ma'ruza (A. va S. Amidi, noyabr 2026) — vaultingizdagi PDF

**Ikkilamchi (sharh va statistika):**

- [LangChain va LangGraph late 2026](https://www.developersdigest.tech/blog/langchain-langgraph-whats-new-2026)
- [State of Agent Engineering 2026 (LangChain + Datadog sharhi)](https://the-agent-report.com/2026/05/state-of-agent-engineering-2026-langchain-datadog/) — asl: [LangChain](https://www.langchain.com/state-of-agent-engineering), [Datadog](https://www.datadoghq.com/state-of-ai-engineering)
- [MCP 2026-07-28 o'zgarishlari (GIGAZINE)](https://gigazine.net/gsc_news/en/20260729-model-context-protocol-2026-07-28/)
- [MCP va A2A AAIF ostida](https://happyrock.cloud/blog/2026-09-01_b_en/)
- [Agent2Agent (Wikipedia)](https://en.wikipedia.org/wiki/Agent2Agent)
- [AI benchmark leaderboards](https://www.developersdigest.tech/benchmarks) — 7-noyabr 2026 holati

*Ikkilamchi manbalardagi raqamlar asl hisobotlar bilan to'liq solishtirilmagan.*


## Bog'liq
[[Agentlar]] · [[Tool calling]] · [[Workflow]] · [[LangGraph]] · [[MCP]] · [[Human-in-the-loop]] · [[Eval]] · [[Observability]] · [[Guardrails]] · [[OWASP LLM Top 10]] · [[Agentlar va LangGraph qo'llanma]] · [[Tool calling qo'llanma]] · [[MCP qo'llanma]]
