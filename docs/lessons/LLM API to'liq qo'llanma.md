# LLM API: to'liq qo'llanma

Nov 11, 2026 · @write code that makes crud of users in laravel

OpenAI Python SDK va Responses API bilan LLM chaqirishning hamma asosiy jihatlari: har bo'limda tushuntirish va o'zingiz yozib ishga tushiradigan kod.

## 1. Tayyorgarlik

**API kalit.** ChatGPT obunasi API'ga kirmaydi. [platform.openai.com](https://platform.openai.com) saytida ro'yxatdan o'ting, balansga ozgina pul (masalan $5) tashlang va **API keys** bo'limida kalit yarating. Kalit faqat bir marta ko'rsatiladi.

**Loyiha papkasi:**

```bash
mkdir llm-darslar && cd llm-darslar
uv init
uv add openai python-dotenv pydantic tiktoken jupyter
```

**.env fayl** (loyiha papkasida):

```
OPENAI_API_KEY=sk-...
```

**.gitignore** ga albatta qo'shing:

```
.env
```

GitHub'ga tushib qolgan kalitni botlar bir necha daqiqada topib ishlatadi. Shunday bo'lsa, kalitni darhol o'chiring va yangisini yarating.

**Notebookni ishga tushirish:** `uv run jupyter notebook`. Birinchi katakka:

```python
from dotenv import load_dotenv
load_dotenv()  # .env dagi OPENAI_API_KEY ni o'qiydi

from openai import OpenAI
client = OpenAI()  # kalitni o'zi muhitdan oladi
```

**Qaysi model?** Model nomlari tez o'zgaradi: masalan `gpt-5-mini` 2026-yil 11-dekabrda o'chiriladi. Hozir mavjudlarini kodning o'zidan ko'ring:

```python
for m in client.models.list():
    print(m.id)
```

Hozir OpenAI'ning eng kuchli modellari GPT-6 Astra va GPT-6.1 Sol, arzon va tezlari GPT-6 Luna va GPT-5.4 Mini. O'rganish uchun arzonini oling: ro'yxatdan "luna" yoki "mini" so'zi bor aniq nomni topib, bir joyga yozib qo'ying. Pastdagi hamma misollar shu o'zgaruvchidan foydalanadi:

```python
MODEL = "..."  # ro'yxatdan tanlagan nomingiz
```

## 2. Atamalar lug'ati

| Atama | Ma'nosi |
| --- | --- |
| **LLM** | Katta til modeli. Matnni davom ettirishni o'rgangan model: GPT, Claude, Gemini |
| **API** | Modelni kodingizdan HTTP orqali chaqirish usuli |
| **SDK** | API'ni qulay chaqirish uchun tayyor kutubxona (`openai` paketi) |
| **Model** | Qaysi LLM ishlatilishi. Kichik model tez va arzon, katta model aqlliroq |
| **Token** | Model o'qiydigan matn bo'lagi. Inglizchada \~4 belgi, o'zbekchada ko'proq token ketadi. Narx tokenlar bo'yicha |
| **Input tokens** | Siz yuborgan matn: ko'rsatma, savol, tarix, hujjat |
| **Output tokens** | Model yozgan javob. Odatda input'dan bir necha baravar qimmat |
| **Reasoning tokens** | "Fikrlovchi" modellarning ichki mulohazasi. Ko'rinmaydi, lekin output sifatida pul to'lanadi |
| **Context window** | Model bir vaqtda ko'ra oladigan eng ko'p token: input + output birga |
| **Instructions (system prompt)** | Modelning doimiy roli va qoidalari |
| **Input (user)** | Foydalanuvchi xabari yoki savoli |
| **Assistant** | Modelning oldingi javoblari, suhbat tarixida |
| **Stateless** | API oldingi so'rovlarni eslamaydi, tarixni har safar o'zingiz yuborasiz |
| **Temperature** | Tasodiflik darajasi: 0 ga yaqin barqaror, yuqori bo'lsa xilma-xil javob |
| **max\_output\_tokens** | Javob uzunligining yuqori chegarasi |
| **Streaming** | Javobni oxirini kutmasdan bo'lak-bo'lak olish |
| **Structured output** | Javobni oldindan berilgan sxemaga mos JSON qilib olish |
| **Latency** | So'rovdan javobgacha o'tgan vaqt |
| **Rate limit** | Bir daqiqada ruxsat etilgan so'rov yoki token soni. Oshsa 429 xatosi qaytadi |
| **Retry / backoff** | Xatodan keyin qayta urinish, har safar kutish vaqtini oshirib |
| **Prompt caching** | Bir xil boshlanadigan uzun promptni provayder keshlaydi: arzonroq va tezroq |
| **Hallucination** | Model ishonch bilan noto'g'ri ma'lumot to'qishi |
| **Prompt injection** | Matn ichiga yashirilgan "oldingi ko'rsatmalarni unut" kabi zararli ko'rsatma |

## 3. Birinchi chaqiruv va javob tuzilishi

```python
resp = client.responses.create(
    model=MODEL,
    input="Python'da list va tuple farqini bir jumlada ayt.",
)
print(resp.output_text)
```

`resp` ichida faqat matn emas, ko'p foydali ma'lumot bor. Har birini alohida chiqarib ko'ring:

```python
print(resp.id)            # javob ID'si
print(resp.model)         # aslida qaysi model javob berdi
print(resp.status)        # "completed" yoki "incomplete"
print(resp.output_text)   # javob matni
print(resp.usage)         # token sarfi
```

`resp.usage` ichida:

- `input_tokens` — siz yuborgan tokenlar
- `output_tokens` — model yozgan tokenlar (reasoning tokenlar ham shu yerda)
- `output_tokens_details.reasoning_tokens` — ichki mulohazaga ketgani
- `input_tokens_details.cached_tokens` — keshdan olingan qism (11-bo'lim)

Butun javobni JSON ko'rinishida ko'rish uchun: `print(resp.model_dump_json(indent=2))`. Bir marta albatta qarab chiqing, `output` ro'yxatining tuzilishini tushunish keyin tool calling'da kerak bo'ladi.

**Mashq:** bitta savolni 3 marta yuboring. Javoblar bir xil chiqdimi? Token sonlari-chi?

## 4. Rollar va suhbat tarixi

**instructions** — modelning doimiy roli. Har so'rovda bir xil turadi:

```python
resp = client.responses.create(
    model=MODEL,
    instructions="Sen o'zbek tilida javob beradigan Python o'qituvchisisan. Javobing 3 jumladan oshmasin.",
    input="Decorator nima?",
)
print(resp.output_text)
```

**API hech narsani eslamaydi.** Buni tekshiring:

```python
client.responses.create(model=MODEL, input="Mening ismim Furqat.")
r = client.responses.create(model=MODEL, input="Mening ismim nima?")
print(r.output_text)  # model bilmaydi
```

Suhbatni davom ettirish uchun butun tarixni o'zingiz yuborasiz. `input` matn emas, xabarlar ro'yxati bo'lishi ham mumkin:

```python
history = []

def chat(user_text: str) -> str:
    history.append({"role": "user", "content": user_text})
    r = client.responses.create(
        model=MODEL,
        instructions="Sen yordamchisan.",
        input=history,
    )
    history.append({"role": "assistant", "content": r.output_text})
    return r.output_text

print(chat("Mening ismim Furqat."))
print(chat("Mening ismim nima?"))  # endi biladi
```

Har yangi xabarda `input_tokens` qanday o'sayotganini kuzating. Tarix uzaygan sari har so'rov qimmatlashadi. Shuning uchun real ilovalarda eski xabarlar qisqartiriladi yoki xulosaga aylantiriladi.

**OpenAI'ning qo'shimcha imkoniyati:** `previous_response_id=r.id` bersangiz, tarixni OpenAI o'zi saqlaydi. Qulay, lekin tarix ularning serverida qoladi va boshqa provayderga o'tish qiyinlashadi. O'rganish uchun avval qo'lda boshqaring.

## 5. Parametrlar va javob kesilishi

| Parametr | Nima qiladi | Qachon |
| --- | --- | --- |
| `max_output_tokens` | Javob uzunligi chegarasi | Doim qo'ying: narxni nazorat qiladi |
| `temperature` | Tasodiflik. 0–2 oralig'ida | Ma'lumot ajratish, RAG: past. Ijodiy matn: yuqoriroq |
| `reasoning={"effort": ...}` | Fikrlovchi modellar qancha "o'ylashi" | Oddiy vazifa: `"low"`. Murakkab: yuqoriroq. Qo'llanadigan qiymatlar modelga bog'liq |
| `timeout` | Javobni qancha kutish | Doim qo'ying (7-bo'lim) |

**Temperature hamma modelda ishlamaydi.** Fikrlovchi (reasoning) modellar uni qabul qilmasligi mumkin. Sinab ko'ring:

```python
from openai import BadRequestError

try:
    r = client.responses.create(model=MODEL, input="Salom", temperature=0.2)
    print("temperature ishladi:", r.output_text)
except BadRequestError as e:
    print("Bu model temperature qabul qilmaydi:", e.message)
```

**Javob kesilishi — eng ko'p e'tibordan chetda qoladigan xato.** Chegarani ataylab kichik qo'ying:

```python
r = client.responses.create(
    model=MODEL,
    input="Python tarixini batafsil yoz.",
    max_output_tokens=50,
)
print(r.status)              # "incomplete"
print(r.incomplete_details)  # reason='max_output_tokens'
print(repr(r.output_text))   # chala, hatto bo'sh bo'lishi mumkin
print(r.usage.output_tokens_details.reasoning_tokens)
```

Fikrlovchi modelda `output_text` umuman bo'sh chiqishi mumkin: hamma 50 token ichki mulohazaga ketib qoladi. Shuning uchun:

1. Bunday modellarda `max_output_tokens`ni kattaroq qo'ying (masalan 2000+).
2. Har javobdan keyin `status`ni tekshiring. `"incomplete"` bo'lsa, foydalanuvchiga chala javobni jim yubormang.

## 6. Token va narx

**Tokenni yuborishdan oldin sanash** (tiktoken):

```python
import tiktoken

enc = tiktoken.get_encoding("o200k_base")  # yangi OpenAI modellari ishlatadigan kodlash

for text in ["How are you?", "Qalaysiz, ishlaringiz yaxshimi?", "Как дела?"]:
    tokens = enc.encode(text)
    print(len(tokens), text)
    print([enc.decode([t]) for t in tokens])  # qanday bo'laklanganini ko'ring
```

O'zbekcha matn inglizchadan ko'proq token oladi. Demak o'zbekcha ilova bir xil hajmdagi matn uchun qimmatroq turadi.

**Narxni hisoblash.** Narxlar 1 million token uchun beriladi va tez-tez o'zgaradi. Tanlagan modelingiz narxini [OpenAI pricing](https://platform.openai.com/docs/pricing) sahifasidan olib, quyiga yozing:

```python
PRICE_IN = 0.10    # $ / 1M input token
PRICE_OUT = 0.50   # $ / 1M output token

def cost_usd(usage) -> float:
    return (usage.input_tokens * PRICE_IN + usage.output_tokens * PRICE_OUT) / 1_000_000

r = client.responses.create(model=MODEL, input="Rekursiya nima?")
print(r.usage.input_tokens, r.usage.output_tokens, f"${cost_usd(r.usage):.6f}")
```

**Mashq:** 1 000 ta foydalanuvchi kuniga 20 tadan savol bersa, oyiga qancha pul ketadi? O'rtacha token sonini o'zingiz o'lchab hisoblang. Bunday hisob loyiha README'sida juda kuchli ko'rinadi.

**Narxni kamaytirish yo'llari:** kichikroq model, qisqaroq instructions, tarixni qisqartirish, `max_output_tokens`, prompt caching (11-bo'lim).

## 7. Xatolar, timeout va retry

Production'dagi ishning katta qismi shu. Tashqi API har doim qachondir ishlamay qoladi.

| Xato (`openai` paketida) | Sababi | Qayta urinish kerakmi |
| --- | --- | --- |
| `APITimeoutError` | Javob belgilangan vaqtda kelmadi | Ha |
| `APIConnectionError` | Internet yoki tarmoq muammosi | Ha |
| `RateLimitError` (429) | Juda ko'p so'rov yoki balans tugagan | Ha, kutib. Balans tugagan bo'lsa yo'q |
| `InternalServerError` (5xx) | Provayder tomonida xato | Ha |
| `AuthenticationError` (401) | Kalit noto'g'ri | Yo'q, kalitni tuzating |
| `BadRequestError` (400) | So'rovda xato: parametr, model nomi | Yo'q, kodni tuzating |
| `NotFoundError` (404) | Bunday model yo'q | Yo'q |

**SDK'ning o'zidagi himoya** — eng oson yo'l. Standart holatda SDK 2 marta qayta urinadi (ulanish xatolari, 408, 409, 429 va 5xx'da), timeout esa 10 daqiqa. 10 daqiqa ilova uchun juda uzoq, shuning uchun ikkalasini o'zingiz belgilang:

```python
client = OpenAI(
    timeout=30.0,   # 30 soniyadan ortiq kutmaydi
    max_retries=3,  # 429, 5xx va ulanish xatolarida o'zi qayta urinadi
)
```

**Retry ichida nima bo'lishini tushunish uchun bir marta o'zingiz yozing:**

```python
import time, random
from openai import APITimeoutError, APIConnectionError, RateLimitError, InternalServerError

RETRYABLE = (APITimeoutError, APIConnectionError, RateLimitError, InternalServerError)
raw = OpenAI(max_retries=0, timeout=30.0)  # SDK retry'sini o'chiramiz

def call_with_retry(**kwargs):
    for attempt in range(4):
        try:
            return raw.responses.create(**kwargs)
        except RETRYABLE as e:
            if attempt == 3:
                raise
            wait = (2 ** attempt) + random.random()  # 1, 2, 4 soniya + tasodifiy qo'shimcha
            print(f"{type(e).__name__}, {wait:.1f}s kutamiz...")
            time.sleep(wait)

r = call_with_retry(model=MODEL, input="Salom")
print(r.output_text)
```

Kutish vaqti har safar ikki baravar oshadi (exponential backoff). Tasodifiy qo'shimcha (jitter) esa ko'p mijozlar bir vaqtda qayta urinib, serverni yana bosib qolmasligi uchun kerak.

**Xatoni ataylab chiqarib ko'ring:**

```python
from openai import AuthenticationError, NotFoundError

try:
    OpenAI(api_key="sk-notogri").responses.create(model=MODEL, input="Salom")
except AuthenticationError as e:
    print("401:", e.message)

try:
    client.responses.create(model="bunday-model-yoq", input="Salom")
except NotFoundError as e:
    print("404:", e.message)

try:
    OpenAI(timeout=0.01).responses.create(model=MODEL, input="Salom")
except APITimeoutError:
    print("Timeout")
```

## 8. Streaming

Oddiy chaqiruvda javob to'liq tayyor bo'lguncha hech narsa ko'rinmaydi. Streaming'da esa matn yozilish jarayonida bo'lak-bo'lak keladi. Foydalanuvchi uchun ilova ancha tezroq tuyuladi.

```python
stream = client.responses.create(
    model=MODEL,
    input="Python'ning 5 ta afzalligini sanab ber.",
    stream=True,
)

for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)   # yangi kelgan bo'lak
    elif event.type == "response.completed":
        print("\n\nTokenlar:", event.response.usage)   # oxirida to'liq javob va usage
```

Stream turli **event**'lardan iborat. Har biri `type` maydoniga ega. Hammasini ko'rish uchun bir marta `print(event.type)` qilib chiqing.

**Ikki muhim o'lchov:**

- **Time to first token** — birinchi bo'lak qancha vaqtda keldi. Foydalanuvchi aynan shuni sezadi.
- **Umumiy vaqt** — javob to'liq tugaguncha.

```python
import time

start = time.perf_counter()
first = None
for event in client.responses.create(model=MODEL, input="Hikoya yoz.", stream=True):
    if event.type == "response.output_text.delta" and first is None:
        first = time.perf_counter() - start
print(f"Birinchi token: {first:.2f}s, jami: {time.perf_counter() - start:.2f}s")
```

## 9. Structured output

Kodda ishlatish uchun erkin matn emas, aniq tuzilishga ega ma'lumot kerak bo'ladi. Pydantic model berasiz, model javobni shu sxemaga moslab qaytaradi. Learning app'dagi test savollari va flashcard'lar aynan shunday yasaladi.

```python
from pydantic import BaseModel, Field
from typing import Literal

class QuizQuestion(BaseModel):
    question: str
    options: list[str] = Field(description="4 ta variant")
    correct_index: int = Field(ge=0, le=3)
    explanation: str

class Quiz(BaseModel):
    topic: str
    difficulty: Literal["oson", "o'rta", "qiyin"]
    questions: list[QuizQuestion]

resp = client.responses.parse(
    model=MODEL,
    instructions="Berilgan matn asosida o'zbek tilida test tuz.",
    input="Python'da list o'zgaruvchan (mutable), tuple esa o'zgarmas (immutable). ...",
    text_format=Quiz,
)

quiz = resp.output_parsed        # oddiy matn emas, tayyor Quiz obyekti
print(quiz.topic, quiz.difficulty)
for q in quiz.questions:
    print(q.question, "->", q.options[q.correct_index])
```

E'tibor bering: `create` emas, **`parse`** metodi, va `output_text` emas, **`output_parsed`**. Internetdagi ko'p misollarda shu joyda xato bor.

**Baribir tekshirish kerak:**

- `resp.output_parsed` `None` bo'lishi mumkin: model javob berishdan bosh tortgan yoki javob kesilgan. Avval `resp.status`ni tekshiring.
- Sxema to'g'ri bo'lsa ham, mazmun noto'g'ri bo'lishi mumkin: masalan `correct_index` noto'g'ri variantni ko'rsatadi. Bu endi eval'ning ishi.
- `Field(description=...)` modelga yo'riqnoma vazifasini bajaradi. Maydon nomlari va izohlarni aniq yozing.

**Mashq:** `Flashcard(front: str, back: str)` modelini yarating va istalgan maqoladan 10 ta flashcard oling.

## 10. Async va parallel chaqiruvlar

LLM chaqiruvi bir necha soniya oladi. 10 ta hujjatga ketma-ket xulosa so'rasangiz, 10 baravar kutasiz. Async bilan ular bir vaqtda yuboriladi.

```python
import asyncio, time
from openai import AsyncOpenAI

aclient = AsyncOpenAI(timeout=30.0, max_retries=3)

async def summarize(topic: str) -> str:
    r = await aclient.responses.create(
        model=MODEL,
        input=f"{topic} haqida bir jumla yoz.",
    )
    return r.output_text

topics = ["list", "dict", "set", "tuple", "generator"]

# ketma-ket
start = time.perf_counter()
for t in topics:
    await summarize(t)
print(f"Ketma-ket: {time.perf_counter() - start:.1f}s")

# parallel
start = time.perf_counter()
results = await asyncio.gather(*(summarize(t) for t in topics))
print(f"Parallel: {time.perf_counter() - start:.1f}s")
```

Jupyter'da `await`ni katakda to'g'ridan-to'g'ri yozish mumkin. Oddiy `.py` faylda esa kodni `async def main()` ichiga olib, `asyncio.run(main())` bilan ishga tushirasiz.

**Ehtiyot bo'ling:** 500 ta so'rovni birdan yuborsangiz, rate limit (429) xatosiga urilasiz. Bir vaqtdagi so'rovlar sonini cheklang:

```python
sem = asyncio.Semaphore(5)  # bir vaqtda ko'pi bilan 5 ta so'rov

async def limited(topic: str) -> str:
    async with sem:
        return await summarize(topic)

results = await asyncio.gather(*(limited(t) for t in topics * 10))
```

**Client'ni bir marta yarating** va qayta ishlating. Har chaqiruvda yangi client yaratish ulanishlarni isrof qiladi.

## 11. Prompt caching

Learning app'da bitta katta hujjatga ko'p savol berasiz va har safar o'sha hujjat qaytadan yuboriladi. Qo'llab-quvvatlanadigan modellarda prompt caching standart holatda yoqilgan: promptning **boshlanishi** oldingi so'rov bilan bir xil bo'lsa, o'sha qism keshdan olinadi. Bu kamida 1 024 tokenlik promptlarda ishlaydi. OpenAI ma'lumotiga ko'ra keshlangan input 95 foizgacha arzonroq va javob tezroq keladi. Yangi modellarda keshga yozish oddiy input'dan biroz qimmatroq, shuning uchun kesh bir matn qayta-qayta ishlatilgandagina foyda beradi.

**Asosiy qoida: o'zgarmaydigan qism boshida, o'zgaradigan qism oxirida.**

```python
hujjat = open("python_kitob.txt", encoding="utf-8").read()  # uzun matn

def ask_doc(question: str):
    r = client.responses.create(
        model=MODEL,
        instructions="Faqat berilgan hujjat asosida javob ber.",
        input=[
            {"role": "user", "content": f"HUJJAT:\n{hujjat}"},   # o'zgarmaydi -> keshlanadi
            {"role": "user", "content": f"SAVOL: {question}"},  # har safar boshqa
        ],
    )
    u = r.usage
    print(f"input={u.input_tokens}, keshdan={u.input_tokens_details.cached_tokens}")
    return r.output_text

ask_doc("Decorator nima?")
ask_doc("Generator nima?")   # ikkinchi chaqiruvda cached_tokens > 0 bo'lishi kerak
```

Agar savolni hujjatdan **oldin** qo'ysangiz, prompt boshi har safar o'zgaradi va kesh ishlamaydi. Buni ham sinab, `cached_tokens`ni solishtiring.

Kesh cheklangan muddat saqlanadi: GPT-5.6 va undan yangi modellarda standart 30 daqiqa, eskiroq modellarda faolsiz holatda taxminan 5–10 daqiqa. Shuning uchun u bir hujjatga ketma-ket savol berilganda eng ko'p foyda beradi.

Ilg'or variant: yangi modellarda `prompt_cache_options` orqali keshlanadigan joylarni o'zingiz belgilashingiz (explicit mode) va kesh muddatini uzaytirishingiz mumkin. Boshlanishiga standart holat yetarli.

## 12. Hammasini birlashtirgan ask() funksiyasi

Oldingi bo'limlardagi hamma narsa bitta funksiyada: timeout, retry, kesilishni tekshirish, log va narx. Learning app'ingizdagi birinchi haqiqiy kod shu bo'ladi.

```python
import logging, time
from dataclasses import dataclass
from openai import OpenAI

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("llm")

client = OpenAI(timeout=30.0, max_retries=3)

@dataclass
class LLMResult:
    text: str
    input_tokens: int
    output_tokens: int
    cached_tokens: int
    latency_ms: float
    cost_usd: float
    complete: bool

def ask(question: str, instructions: str = "Sen yordamchisan.",
        history: list | None = None, max_output_tokens: int = 2000) -> LLMResult:
    messages = (history or []) + [{"role": "user", "content": question}]
    start = time.perf_counter()
    r = client.responses.create(
        model=MODEL,
        instructions=instructions,
        input=messages,
        max_output_tokens=max_output_tokens,
    )
    ms = (time.perf_counter() - start) * 1000
    u = r.usage
    result = LLMResult(
        text=r.output_text,
        input_tokens=u.input_tokens,
        output_tokens=u.output_tokens,
        cached_tokens=u.input_tokens_details.cached_tokens,
        latency_ms=ms,
        cost_usd=cost_usd(u),          # 6-bo'limdagi funksiya
        complete=(r.status == "completed"),
    )
    log.info("model=%s in=%d out=%d cached=%d %.0fms $%.5f status=%s",
             MODEL, result.input_tokens, result.output_tokens,
             result.cached_tokens, ms, result.cost_usd, r.status)
    if not result.complete:
        log.warning("Javob to'liq emas: %s", r.incomplete_details)
    return result

res = ask("Python'da generator nima?", instructions="Qisqa va misol bilan tushuntir.")
print(res.text)
```

Keyinchalik bu funksiya `llm/` papkasiga ko'chadi va `LLMProvider` interfeysi ortiga yashiriladi. Shunda Claude yoki boshqa provayderni qo'shish bitta yangi fayllik ish bo'ladi.

## 13. Mashqlar

Har birini kodga qaramasdan, o'zingiz yozing. Tiqilib qolsangiz, tegishli bo'limni qayta o'qing.

- [x] Bitta savolni 3 marta yuborib, javoblar va token sonlarini solishtiring
- [x] Terminalda ishlaydigan chat yozing: `while True` sikli, suhbat tarixi bilan, `/reset` buyrug'i tarixni tozalasin
- [x] Chatga har javobdan keyin tokenlar va narxni chiqarishni qo'shing
- [x] `max_output_tokens=50` bilan javob kesilishini chiqarib, foydalanuvchiga "javob to'liq emas" deb ogohlantiring
- [x] O'zbekcha va inglizcha bir xil matnning token sonini solishtiring
- [ ] Retry funksiyasini o'zingiz yozing va `timeout=0.01` bilan sinab ko'ring
- [ ] Chatni streaming'ga o'tkazing va time to first token'ni o'lchang
- [ ] Istalgan maqoladan 5 ta test savoli (`Quiz` modeli) va 10 ta flashcard oling
- [ ] 10 ta mavzuga xulosani ketma-ket va parallel olib, vaqtni solishtiring
- [ ] Uzun matnga 2 marta savol berib, `cached_tokens` o'sganini ko'ring
- [ ] `ask()` funksiyasini alohida `llm.py` faylga ko'chiring va chatni shu funksiya orqali ishlating

Oxirgi mashqdagi chat — learning app'ning v0 versiyasi. Uni GitHub'ga push qiling.
