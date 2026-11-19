---
tags: [lesson, llm-api, structured-output]
status: o'rganilmoqda
---

# Structured output to'liq qo'llanma

Xarita: [[LLM API]] · [[Learning app]] · [[Home]]

LLM javobini erkin matn emas, kod ishlata oladigan JSON qilib olish. Learning app v3: test savollari va flashcard'lar shu bilan yasaladi.

## 1. Nima va qachon kerak

Oddiy chaqiruvda model matn qaytaradi: "Mana 3 ta savol: 1) ...". Bu matnni kod bilan bo'laklarga ajratish qiyin va ishonchsiz. **Structured output** bilan siz oldindan shakl (sxema) berasiz, model esa aynan shu shakldagi JSON qaytaradi. Strict rejimda bu shakl **kafolatlanadi**: maydon tushib qolmaydi, turi o'zgarmaydi, ortiqcha maydon qo'shilmaydi.

| Erkin matn | Structured output |
| --- | --- |
| `"Savol: ... A) ... B) ..."` | `{"question": "...", "options": [...], "correct_index": 2}` |
| Regex bilan ajratasiz, ba'zan buziladi | `resp.output_parsed.correct_index` |
| Format har safar biroz boshqacha | Format har doim bir xil |

**Qachon ishlatiladi:**

- Natijani **kod** ishlatadi: bazaga yozish, UI'da ko'rsatish, keyingi qadamga uzatish.
- Matndan ma'lumot ajratish (extraction): hujjatdan atamalar, sanalar, ismlar.
- Tasniflash (classification): savol turi, qiyinlik darajasi.
- Bir nechta qismli javob: javob + manbalar + ishonch darajasi.

**Qachon kerak emas:** foydalanuvchi faqat o'qiydigan oddiy javob (chat). U yerda erkin matn yaxshiroq va streaming osonroq.

**Learning app'da:** test savollari, flashcard'lar, bob xulosasi (sarlavha + asosiy fikrlar + atamalar), RAG javobi manbalar bilan (`RagAnswer`), eval'dagi judge hukmi.

**Muhim ajratish:** structured output **shaklni** kafolatlaydi, **mazmunni** emas. `correct_index: 2` har doim butun son bo'ladi, lekin to'g'ri javob aslida 1-variant bo'lishi mumkin. Mazmunni siz tekshirasiz (10-bo'lim).

## 2. Atamalar lug'ati

| Atama | Ma'nosi | Misol |
| --- | --- | --- |
| **Sxema (schema)** | Javob shaklining ta'rifi: qaysi maydonlar, qaysi turda | `question: str`, `options: list[str]` |
| **JSON Schema** | Sxemani yozishning standart formati; API aslida shuni qabul qiladi | `{"type": "object", "properties": {...}}` |
| **Pydantic** | Python'da sxemani class qilib yozish kutubxonasi; SDK uni JSON Schema'ga o'zi aylantiradi | `class Quiz(BaseModel): ...` |
| **Strict rejim** | Model sxemadan chiqa olmaydigan rejim (constrained decoding) | `"strict": True` |
| **`text_format`** | `responses.parse`ga Pydantic class berish parametri | `text_format=Quiz` |
| **`output_parsed`** | Tayyor Pydantic obyekt ko'rinishidagi javob | `resp.output_parsed.questions[0]` |
| **Refusal** | Model xavfsizlik sababli javob bermaganda keladigan maxsus element | `item.type == "refusal"` |
| **Enum** | Faqat ruxsat etilgan qiymatlar ro'yxati | `Literal["oson", "o'rta", "qiyin"]` |
| **Optional maydon** | Bo'lmasligi mumkin bo'lgan maydon; strict'da `null` bilan beriladi | `hint: str \| None` |
| **`Field(description=...)`** | Maydon uchun izoh; model uni prompt sifatida o'qiydi | `Field(description="Manba raqami")` |
| **Validatsiya** | Kelgan ma'lumotni qoidalar bilan tekshirish | `correct_index < len(options)` |
| **Extraction** | Matndan tuzilgan ma'lumot ajratib olish | PDF'dan atamalar ro'yxati |
| **JSON mode** | Eski rejim: JSON bo'lishini kafolatlaydi, lekin sxemani emas | `{"type": "json_object"}` |

Bu atamalarning har biri vaultda alohida qayd sifatida bor: [[Structured output]], [[JSON Schema]], [[Pydantic]], [[Strict mode]], [[Refusal]], [[Enum]], [[Optional maydon]], [[Field description]], [[Validatsiya]], [[Extraction]], [[JSON mode]].

## 3. Birinchi structured chaqiruv

Yangi notebook oching: `notebooks/03_structured_output.ipynb`. Pydantic `openai` bilan birga o'rnatilgan, alohida o'rnatish shart emas.

```python
from openai import OpenAI
from pydantic import BaseModel

client = OpenAI(timeout=60, max_retries=3)
MODEL = "gpt-6-luna"

class Term(BaseModel):
    term: str
    definition: str

class TermList(BaseModel):
    terms: list[Term]

text = """Embedding - matnni sonlar vektoriga aylantirish.
Chunking - uzun hujjatni kichik bo'laklarga bo'lish.
Reranking - topilgan bo'laklarni qayta baholash."""

resp = client.responses.parse(
    model=MODEL,
    instructions="Matndagi barcha atamalarni ta'rifi bilan ajratib ol.",
    input=text,
    text_format=TermList,
)

result = resp.output_parsed          # TermList obyekti
for t in result.terms:
    print(t.term, "->", t.definition)
```

**Nima bo'ldi, qadam-baqadam:**

1. SDK `TermList` class'ini JSON Schema'ga aylantirdi va `strict: True` bilan yubordi.
2. Model javobni faqat shu sxemaga mos tokenlar bilan yozdi.
3. SDK kelgan JSON'ni `TermList` obyektiga aylantirib, `output_parsed`ga qo'ydi.

**`create` va `parse` farqi:**

|  | `responses.create` | `responses.parse` |
| --- | --- | --- |
| Natija | `output_text` (matn) | `output_parsed` (Pydantic obyekt) + `output_text` |
| Sxema | O'zingiz `text={"format": ...}` yozasiz | `text_format=Class` yetarli |
| Qachon | Oddiy matn, raw JSON Schema | Deyarli har doim structured uchun |

**Sinab ko'ring:** `print(resp.output_text)` ham qiling. U yerda xom JSON matnni ko'rasiz; `output_parsed` shu matndan yasalgan.

**Birinchi chaqiruv sekinroq:** yangi sxema birinchi marta ishlatilganda API uni qayta ishlaydi, shuning uchun biroz kechikish bo'ladi. Keyingi chaqiruvlar tezroq. Sxemani har safar o'zgartirib turmang.

## 4. Pydantic sxema yozish

Sxema — bu ham prompt. Maydon nomlari va izohlarini model o'qiydi va shunga qarab to'ldiradi. Shuning uchun nomlar aniq bo'lsin: `x` emas, `correct_index`.

**Python turi → JSON Schema:**

| Python | JSON Schema | Izoh |
| --- | --- | --- |
| `str` | `string` |  |
| `int` / `float` | `integer` / `number` |  |
| `bool` | `boolean` |  |
| `list[str]` | `array` of `string` |  |
| `Literal["a", "b"]` | `enum` | Faqat shu qiymatlar chiqadi |
| `str \| None` | `["string", "null"]` | Optional maydon |
| Boshqa `BaseModel` | ichma-ich `object` |  |

**To'liq misol: test savoli.**

```python
from typing import Literal
from pydantic import BaseModel, Field

class Question(BaseModel):
    question: str = Field(description="Savol matni, o'zbek tilida")
    options: list[str] = Field(description="Aynan 4 ta javob varianti")
    correct_index: int = Field(description="To'g'ri variant indeksi, 0 dan 3 gacha")
    explanation: str = Field(description="Nega bu javob to'g'ri, 1-2 gap")
    difficulty: Literal["oson", "o'rta", "qiyin"]
    source: int = Field(description="Savol olingan MANBA raqami")

class Quiz(BaseModel):
    topic: str
    questions: list[Question]
```

**Yaxshi sxema qoidalari:**

- **`description` yozing.** "0 dan 3 gacha", "o'zbek tilida", "1-2 gap" kabi cheklovlarni shu yerga yozing. Model ularni o'qiydi.
- **Erkin matn o'rniga `Literal`.** `difficulty: str` bo'lsa "medium", "O'rtacha", "2" chiqishi mumkin. `Literal` bilan faqat uchta qiymat chiqadi.
- **Maydonlar tartibi muhim.** Model maydonlarni sxemadagi tartibda yozadi. `explanation`ni `correct_index`dan **oldin** qo'ysangiz, model avval o'ylab, keyin javobni tanlaydi. Bu ko'pincha aniqlikni oshiradi.
- **Ichma-ich modelni haddan oshirmang.** 2-3 daraja yetadi. Murakkab sxema = modelga og'irroq vazifa.
- **Default qiymat bermang.** Strict rejimda hamma maydon majburiy (5-bo'lim). Bo'lmasligi mumkin bo'lsa, `str | None` qiling.

**Tartib haqida misol:**

```python
class ReasonedAnswer(BaseModel):
    reasoning: str        # avval fikrlash
    answer: str           # keyin javob
```

## 5. Strict rejim qoidalari va cheklovlar

Strict rejim JSON Schema'ning hammasini emas, bir qismini qo'llab-quvvatlaydi. Pydantic ishlatsangiz, SDK ko'p qoidani o'zi bajaradi. Lekin bilish kerak: xato chiqsa, sababi shu yerda bo'ladi.

**Qoidalar:**

| Qoida | Ma'nosi | Pydantic'da |
| --- | --- | --- |
| Ildiz obyekt bo'lsin | Eng tashqi tur `object`; ildizda `anyOf` bo'lmaydi | Har doim `BaseModel` bering, `list[...]` emas |
| Hamma maydon `required` | "Ixtiyoriy" maydon yo'q | SDK o'zi qo'shadi |
| Optional = `null` bilan | `"type": ["string", "null"]` | `str \| None` |
| `additionalProperties: false` | Har bir obyektda | SDK o'zi qo'shadi |
| Kalit tartibi saqlanadi | Javob sxemadagi tartibda chiqadi | class'dagi tartib |

**Qo'llab-quvvatlanadigan turlar:** string, number, integer, boolean, object, array, enum, anyOf. `$defs` + `$ref` va rekursiv sxema ham ishlaydi.

**Qo'shimcha kalit so'zlar:**

- string: `pattern`, `format` (`date-time`, `date`, `time`, `email`, `uuid`, `ipv4` va h.k.)
- number: `minimum`, `maximum`, `exclusiveMinimum`, `exclusiveMaximum`, `multipleOf`
- array: `minItems`, `maxItems`

**Ishlamaydigan kalit so'zlar:** `allOf`, `not`, `if/then/else`, `dependentRequired`, `dependentSchemas`.

**Cheklovlar:**

| Narsa | Limit |
| --- | --- |
| Jami obyekt maydonlari | 5 000 |
| Ichma-ich darajalar | 10 |
| Barcha nomlar + enum qiymatlari uzunligi | 120 000 belgi |
| Barcha enum qiymatlari soni | 1 000 |

Learning app'da bu limitlarga hech qachon yetmaysiz. Ular katta korporativ sxemalar uchun.

**Amaliy maslahat:** "aynan 4 ta variant" kabi talablarni ikki joyda bering: `Field(description=...)` ichida (model ko'rsin) va kodda tekshiring (10-bo'lim). `min_length=4, max_length=4` ham `minItems/maxItems` bo'lib ketadi, lekin tekshiruvni kodda qoldirish xavfsizroq.

## 6. Raw JSON Schema va JSON mode

Pydantic ko'pincha yetarli. Lekin sxemani qo'lda yozish ham kerak bo'ladi: sxema bazada saqlanganda, boshqa tildan kelganda yoki Pydantic'siz ishlaganda. Bunda `responses.create` va `text={"format": ...}` ishlatiladi.

```python
import json

schema = {
    "type": "object",
    "properties": {
        "front": {"type": "string"},
        "back": {"type": "string"},
        "hint": {"type": ["string", "null"]},
    },
    "required": ["front", "back", "hint"],     # hint ham required, lekin null bo'lishi mumkin
    "additionalProperties": False,
}

resp = client.responses.create(
    model=MODEL,
    input="Embedding haqida bitta flashcard tuz.",
    text={"format": {
        "type": "json_schema",
        "name": "flashcard",
        "schema": schema,
        "strict": True,
    }},
)
card = json.loads(resp.output_text)     # oddiy dict
```

**Pydantic'dan sxemani ko'rish:** `print(json.dumps(Quiz.model_json_schema(), indent=2))`. Bu SDK yuboradigan sxemaga juda yaqin (SDK ustiga strict qoidalarini qo'shadi).

**Uch xil rejim:**

| Rejim | Qanday yoqiladi | JSON kafolati | Sxema kafolati | Qachon |
| --- | --- | --- | --- | --- |
| Oddiy matn | hech narsa | Yo'q | Yo'q | Chat javobi |
| JSON mode | `{"type": "json_object"}` | Ha | **Yo'q** | Eski kod; ishlatmang |
| Structured output | `json_schema` + `strict: True` | Ha | **Ha** | Har doim shuni tanlang |

JSON mode faqat "to'g'ri JSON" beradi: maydonlar boshqacha nomlanishi yoki tushib qolishi mumkin. Rasmiy hujjat ham structured output'ni tavsiya qiladi.

**Tool calling bilan aloqasi:** tool'ning `parameters` qismi ham xuddi shu JSON Schema va `strict: True` qoidalari bilan yoziladi. Shu yerda o'rganganlaringiz [[Tool calling qo'llanma]]da to'g'ridan-to'g'ri ishlatiladi.

## 7. Xatolar: refusal, incomplete, validatsiya

Sxema kafolatlangan bo'lsa ham, `output_parsed` ba'zan `None` bo'ladi. Uchta holatni ushlash kerak:

| Holat | Nima bo'ldi | Qanday bilinadi |
| --- | --- | --- |
| **Refusal** | Model xavfsizlik sababli javob bermadi | `output` ichida `type == "refusal"` element |
| **Incomplete** | Javob token limitida kesildi, JSON chala | `resp.status == "incomplete"`, sabab `max_output_tokens` |
| **API xatosi** | Timeout, rate limit, noto'g'ri sxema | `openai` exception'lari |

Noto'g'ri sxema (masalan ildizda `list`) `BadRequestError` beradi: bu sizning kodingiz xatosi, retry yordam bermaydi.

**Hammasini ushlaydigan funksiya:**

```python
import openai

class StructuredError(Exception):
    pass

def parse_or_fail(text_format, **kwargs):
    try:
        resp = client.responses.parse(model=MODEL, text_format=text_format, **kwargs)
    except openai.BadRequestError as e:
        raise StructuredError(f"Sxema yoki so'rov xato: {e}") from e

    if resp.status == "incomplete":
        reason = resp.incomplete_details.reason
        raise StructuredError(f"Javob chala: {reason}. max_output_tokens'ni oshiring.")

    for item in resp.output:
        if item.type != "message":
            continue
        for part in item.content:
            if part.type == "refusal":
                raise StructuredError(f"Model rad etdi: {part.refusal}")

    if resp.output_parsed is None:
        raise StructuredError("Javob bo'sh keldi")
    return resp.output_parsed
```

**Kesilib qolishdan saqlanish:** 20 ta savol so'rasangiz, javob uzun bo'ladi. `max_output_tokens`ni yetarli qo'ying yoki savollarni 5 tadan bo'lib so'rang.

**"Yolg'on to'ldirish" xavfi:** agar matnda kerakli ma'lumot bo'lmasa, model sxemani to'ldirish uchun narsa **o'ylab topishi** mumkin. Bunga qarshi:

- Promptda yozing: "Manbada bo'lmasa, `questions` ro'yxatini bo'sh qoldir."
- Yoki sxemaga maxsus maydon qo'shing: `enough_info: bool`. Model avval shuni to'ldiradi.

## 8. Streaming

20 ta savol tuzish 10-20 soniya oladi. Foydalanuvchi bo'sh ekranga qarab turmasligi uchun javobni oqim (stream) qilib olish mumkin. Structured output uchun maxsus `responses.stream` bor.

```python
with client.responses.stream(
    model=MODEL,
    input="Chunking haqida 5 ta test savoli tuz.",
    text_format=Quiz,
) as stream:
    for event in stream:
        if event.type == "response.output_text.delta":
            print(event.delta, end="", flush=True)     # xom JSON bo'laklari
        elif event.type == "response.refusal.delta":
            print(event.delta, end="")
        elif event.type == "error":
            raise RuntimeError(event)
        elif event.type in ("response.failed", "response.incomplete"):
            raise RuntimeError(f"To'xtadi: {event.type}")

    final = stream.get_final_response()

quiz = final.output_parsed          # tayyor Quiz obyekti
```

**Muammo:** oqim davomida kelayotgan narsa chala JSON: `{"topic": "Chunk`. Uni foydalanuvchiga to'g'ridan ko'rsatib bo'lmaydi.

**Learning app uchun amaliy yechimlar:**

| Yondashuv | Qanday | Qachon |
| --- | --- | --- |
| Progress ko'rsatish | Delta'lar kelayotganda faqat "Tuzilyapti..." yoki belgilar soni | Eng oddiy, boshlash uchun |
| Bo'lib so'rash | 20 ta o'rniga 4 marta 5 tadan; har biri tayyor bo'lganda ko'rsatish | Ishonchli, tavsiya |
| Chala JSON'ni parse qilish | `partial-json-parser` kabi kutubxona | Keyinroq, kerak bo'lsa |

Boshlanishiga streaming'siz yozing. FastAPI bosqichida ([[FastAPI va streaming qo'llanma]]) qayting.

## 9. Learning app: test va flashcard (v3)

Endi RAG bilan birlashtiramiz: savollar **sizning hujjatlaringizdan** olinadi va har biri manbaga bog'lanadi. [[RAG to'liq qo'llanma]]dagi `search()` va `build_context()` funksiyalaridan foydalanamiz.

**Oqim:**

1. Mavzu bo'yicha qidiruv: `search(topic, k=8)` → 8 ta bo'lak.
2. Bo'laklarni raqamlab kontekst qilish: `[1] ...`, `[2] ...`.
3. `responses.parse` + `Quiz` yoki `Deck` sxemasi.
4. Mazmunni tekshirish (10-bo'lim).
5. Bazaga saqlash (11-bo'lim).

**Sxemalar:**

```python
from typing import Literal
from pydantic import BaseModel, Field

class Question(BaseModel):
    question: str = Field(description="Savol matni, o'zbek tilida")
    options: list[str] = Field(description="Aynan 4 ta variant")
    explanation: str = Field(description="To'g'ri javob nega to'g'ri, 1-2 gap")
    correct_index: int = Field(description="To'g'ri variant indeksi, 0-3")
    difficulty: Literal["oson", "o'rta", "qiyin"]
    source: int = Field(description="MANBALAR ichidagi raqam")

class Quiz(BaseModel):
    questions: list[Question]

class Flashcard(BaseModel):
    front: str = Field(description="Atama yoki qisqa savol")
    back: str = Field(description="Qisqa javob, 1-3 gap")
    source: int = Field(description="MANBALAR ichidagi raqam")

class Deck(BaseModel):
    cards: list[Flashcard]
```

E'tibor bering: `explanation` endi `correct_index`dan **oldin** (4-bo'limdagi tartib qoidasi).

**Generatsiya funksiyalari:**

```python
QUIZ_RULES = """Sen o'qituvchisan. Faqat MANBALAR asosida test tuz.
- Har savolda aynan 4 ta variant, faqat bittasi to'g'ri.
- Noto'g'ri variantlar ishonarli bo'lsin, lekin aniq noto'g'ri.
- source maydoniga savol olingan manba raqamini yoz.
- Manbada yetarli ma'lumot bo'lmasa, questions ro'yxatini bo'sh qoldir."""

def make_quiz(topic: str, n: int = 5) -> tuple[Quiz, list]:
    hits = search(topic, k=8)
    quiz = parse_or_fail(
        Quiz,
        instructions=QUIZ_RULES,
        input=f"MANBALAR:\n{build_context(hits)}\n\nMAVZU: {topic}\nSAVOLLAR SONI: {n}",
        max_output_tokens=4000,
    )
    return quiz, hits          # hits keyin tekshiruv va saqlash uchun kerak

def make_deck(topic: str, n: int = 10) -> tuple[Deck, list]:
    hits = search(topic, k=8)
    deck = parse_or_fail(
        Deck,
        instructions="Faqat MANBALAR asosida flashcard tuz. Har biriga manba raqamini yoz.",
        input=f"MANBALAR:\n{build_context(hits)}\n\nMAVZU: {topic}\nKARTALAR SONI: {n}",
    )
    return deck, hits

quiz, hits = make_quiz("pgvector indekslari")
for q in quiz.questions:
    print(q.question)
    for i, o in enumerate(q.options):
        print("  ", "✓" if i == q.correct_index else " ", o)
```

**Bonus — bob xulosasi:**

```python
class Summary(BaseModel):
    title: str
    key_points: list[str] = Field(description="3-7 ta asosiy fikr")
    terms: list[str] = Field(description="Bobdagi muhim atamalar")
```

`terms` ro'yxatidan keyin avtomatik flashcard yasash mumkin: har bir atama uchun `make_deck(term, n=1)`.

## 10. Mazmunni tekshirish

Sxema shaklni kafolatladi. Endi mazmunni kod bilan tekshiramiz. Ikki yo'l bor: Pydantic validator (parse paytida) yoki alohida funksiya (parse'dan keyin). Learning app uchun ikkinchisi qulayroq: yomon savolni tashlab, yaxshilarini saqlab qolasiz.

**Nimalarni tekshirish:**

| Tekshiruv | Nega |
| --- | --- |
| `len(options) == 4` | Model ba'zan 3 yoki 5 ta beradi |
| `0 <= correct_index < len(options)` | Indeks chegaradan chiqishi mumkin |
| Variantlar takrorlanmaydi | "A" va "A." ikki marta |
| `1 <= source <= len(hits)` | Mavjud bo'lmagan manbaga ishora |
| Savollar takrorlanmaydi | Bir xil savol ikki marta |

```python
def check_question(q: Question, n_sources: int) -> list[str]:
    errors = []
    if len(q.options) != 4:
        errors.append(f"{len(q.options)} ta variant")
    if not 0 <= q.correct_index < len(q.options):
        errors.append(f"correct_index={q.correct_index}")
    if len({o.strip().lower() for o in q.options}) != len(q.options):
        errors.append("takroriy variant")
    if not 1 <= q.source <= n_sources:
        errors.append(f"source={q.source} mavjud emas")
    return errors

def clean_quiz(quiz: Quiz, hits: list) -> list[Question]:
    good, seen = [], set()
    for q in quiz.questions:
        errs = check_question(q, len(hits))
        key = q.question.strip().lower()
        if errs or key in seen:
            print("Tashlandi:", q.question[:50], errs)
            continue
        seen.add(key)
        good.append(q)
    return good
```

**Pydantic validator varianti** (parse paytida xato beradi):

```python
from pydantic import field_validator

class StrictQuestion(Question):
    @field_validator("options")
    @classmethod
    def four_options(cls, v):
        if len(v) != 4:
            raise ValueError("aynan 4 ta variant kerak")
        return v
```

Bu holda xato bo'lsa butun `Quiz` tushib ketadi. Shuning uchun Learning app'da `clean_quiz` yondashuvi yaxshiroq.

**To'g'ri javob haqiqatan to'g'rimi?** Buni kod bilan tekshirib bo'lmaydi. Ikki yo'l:

1. **Ikkinchi chaqiruv (judge):** boshqa promptda savol + manbani berib, "qaysi variant to'g'ri?" deb so'rang. Javob `correct_index`ga mos kelmasa, savolni tashlang. Batafsil: [[LLM-as-a-judge qo'llanma]].
2. **Foydalanuvchi feedback'i:** har savol yonida "⚠️ xato" tugmasi. Bu eval uchun ham ma'lumot ([[Eval qo'llanma]]).

## 11. Postgres'ga saqlash

Test va flashcard'lar bir marta yasalib, keyin ko'p marta ishlatiladi. Ularni `rag-db` bazasiga, `chunks` jadvali yoniga saqlaymiz. `source` raqami (1, 2, 3...) faqat shu so'rov ichida ma'noli, shuning uchun saqlashda uni haqiqiy `chunk_id`ga aylantiramiz.

```sql
CREATE TABLE questions (
    id            bigserial PRIMARY KEY,
    topic         text NOT NULL,
    question      text NOT NULL,
    options       jsonb NOT NULL,          -- ["...", "...", "...", "..."]
    correct_index int  NOT NULL,
    explanation   text NOT NULL,
    difficulty    text NOT NULL,
    chunk_id      bigint REFERENCES chunks(id),
    created_at    timestamptz DEFAULT now()
);

CREATE TABLE flashcards (
    id         bigserial PRIMARY KEY,
    topic      text NOT NULL,
    front      text NOT NULL,
    back       text NOT NULL,
    chunk_id   bigint REFERENCES chunks(id),
    box        int  DEFAULT 1,             -- Leitner qutisi: 1 = har kuni, 5 = kam
    due_at     timestamptz DEFAULT now()
);
```

```python
import psycopg
from psycopg.types.json import Jsonb

DB = "postgresql://postgres:postgres@localhost:5433/rag"

def save_questions(topic: str, questions: list[Question], hits: list):
    with psycopg.connect(DB) as conn:
        for q in questions:
            chunk_id = hits[q.source - 1]["id"]     # 1-raqamli manba -> hits[0]
            conn.execute(
                """INSERT INTO questions
                   (topic, question, options, correct_index, explanation, difficulty, chunk_id)
                   VALUES (%s, %s, %s, %s, %s, %s, %s)""",
                (topic, q.question, Jsonb(q.options), q.correct_index,
                 q.explanation, q.difficulty, chunk_id),
            )

quiz, hits = make_quiz("pgvector indekslari")
good = clean_quiz(quiz, hits)
save_questions("pgvector indekslari", good, hits)
```

`hits[i]["id"]` — [[RAG to'liq qo'llanma]]dagi `search()` qaytaradigan bo'lak id'si. Sizning `search()` boshqacha qaytarsa (tuple, boshqa kalit), shu qatorni moslang.

**Nega `chunk_id` saqlanadi:** foydalanuvchi savolni ochganda "Manba: kitob.pdf, 12-bet" ko'rsatasiz; hujjat o'chirilsa, unga bog'liq savollarni ham topib o'chirasiz.

**Flashcard takrorlash (Leitner, eng sodda usul):** to'g'ri javob → `box + 1`, `due_at` uzoqroqqa (1, 2, 4, 8, 16 kun); xato → `box = 1`. Bugungi kartalar: `WHERE due_at <= now()`.

## 12. Mashqlar

Har birini `notebooks/03_structured_output.ipynb` ichida yozing. Tayyor kodni ko'chirmasdan, o'zingiz yozishga harakat qiling.

1. **Atama ajratish.** O'zingiz yuklagan PDF'dan bitta sahifa oling va `TermList` bilan barcha atamalarni ajrating. `output_text` va `output_parsed`ni solishtiring.
2. **Enum sinovi.** `difficulty: str` bilan 10 ta savol tuzing va qanday qiymatlar chiqqanini sanang. Keyin `Literal` qiling va farqni ko'ring.
3. **Tartib tajribasi.** `Question`da `explanation`ni `correct_index`dan oldin va keyin qo'yib, 10 tadan savol tuzing. Qaysi birida xato javob kamroq? (Qo'lda tekshiring.)
4. **Raw sxema.** `Flashcard`ni Pydantic'siz, qo'lda JSON Schema bilan yozing va `responses.create` orqali chaqiring. Ataylab `additionalProperties`ni olib tashlang: qanday xato chiqadi?
5. **Chala javob.** `max_output_tokens=200` bilan 20 ta savol so'rang. `parse_or_fail` nima deydi?
6. **Bo'sh manba.** Hujjatlaringizda **yo'q** mavzu (masalan, "futbol qoidalari") bo'yicha test so'rang. Model bo'sh ro'yxat qaytaradimi yoki o'ylab topadimi?
7. **To'liq zanjir.** `make_quiz` → `clean_quiz` → `save_questions`. DataGrip'da `questions` jadvalini oching va natijani ko'ring.
8. **Flashcard rejimi.** Bazadan bugungi kartalarni olib, terminalda ko'rsatadigan kichik sikl yozing: front → Enter → back → "bildim/bilmadim" → `box` va `due_at`ni yangilash.

Tugatgach, [[Learning app]] qaydidagi v3 bandini belgilang.

## 13. Manbalar

**Rasmiy:**

- [OpenAI: Structured model outputs](https://developers.openai.com/api/docs/guides/structured-outputs) — `responses.parse`, qoidalar, cheklovlar, refusal, streaming
- [OpenAI: Introducing Structured Outputs](https://openai.com/index/introducing-structured-outputs-in-the-api/) — strict rejim qanday ishlaydi (constrained decoding)
- [Pydantic: Models](https://docs.pydantic.dev/latest/concepts/models/) va [Validators](https://docs.pydantic.dev/latest/concepts/validators/)
- [JSON Schema: Understanding JSON Schema](https://json-schema.org/understanding-json-schema/)

**Vaultdagi bog'liq darslar:**

- &#91;\[LLM API to'liq qo'llanma]] — 9-bo'lim, structured output asoslari
- &#91;\[RAG to'liq qo'llanma]] — `search()`, `build_context()`, `RagAnswer`
- &#91;\[Tool calling qo'llanma]] — xuddi shu JSON Schema tool parametrlarida
- &#91;\[LLM-as-a-judge qo'llanma]] — to'g'ri javobni ikkinchi model bilan tekshirish
- &#91;\[Eval qo'llanma]] — test sifatini o'lchash

## Bog'liq tushunchalar

[[Structured output]] · [[Pydantic]] · [[JSON Schema]] · [[Strict mode]] · [[Optional maydon]] · [[Enum]] · [[Field description]] · [[Refusal]] · [[Validatsiya]] · [[Extraction]] · [[JSON mode]] · [[Flashcard]] · [[Leitner usuli]] · [[Hallucination]] · [[RAG]]
