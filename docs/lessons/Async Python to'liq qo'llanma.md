---
tags: [lesson, python, async]
status: o'rganilmoqda
---

# Async Python to'liq qo'llanma

Xarita: [[LLM API]] · [[Production (xarita)]] · [[Home]]

LLM ilovalarida bir nechta so'rovni bir vaqtda yuborish uchun asyncio: nazariya, misollar va mashqlar. Workflow demo (07) va A3-topshiriqdan oldin o'qiladi.

## 1. Nima uchun kerak

LLM chaqiruvi 3-10 soniya davom etadi. Shu vaqtning **deyarli hammasida** kompyuteringiz hech narsa qilmaydi: so'rov internet orqali OpenAI serveriga ketdi, u yerda model ishlayapti, sizning dasturingiz esa shunchaki **javobni kutib turibdi**.

Bitta chaqiruv bo'lsa, bu muammo emas. Lekin:

- 30 ta bob uchun test tuzish kerak → 30 × 8 s = **4 daqiqa**
- 100 ta chunk'ni LLM bilan tekshirish → 100 × 5 s = **8 daqiqa**
- 20 ta foydalanuvchi bir vaqtda FastAPI serveringizga savol yubordi → oxirgisi 20 kishini kutadi

Bu so'rovlar **bir-biriga bog'liq emas**. 2-bobning testi 1-bobning testini kutishi shart emas. **Async** shu kutish vaqtlarini ustma-ust qo'yadi: hamma so'rovlarni birdan yuborasiz va hammasini birga kutasiz → 4 daqiqa o'rniga ~10 soniya.

**Muhim chegara: async faqat "kutish"ni tezlashtiradi.**

| Ish turi | Misol | Async yordam beradimi? |
| --- | --- | --- |
| **I/O-bound** (kutish) | API chaqiruvi, baza so'rovi, fayl yuklash, veb-sahifa | **Ha** — kutishlar ustma-ust tushadi |
| **CPU-bound** (hisoblash) | Lokal embedding (BGE-M3), katta matritsa, rasm qayta ishlash | **Yo'q** — protsessor band, kutish yo'q |

LLM ilovalarida ishning ko'p qismi I/O: OpenAI, baza, veb. Shuning uchun async bu sohada juda ko'p ishlatiladi.

## 2. Atamalar lug'ati

| Atama | Ma'nosi | Oshxona o'xshatishi |
| --- | --- | --- |
| **Sync (sinxron)** | Har bir ish oldingisi tugagandan keyin boshlanadi | Bitta choynak qaynaguncha yonida turasiz |
| **Async (asinxron)** | Kutish paytida boshqa ishga o'tiladi | Choynakni qo'yib, sabzavot to'g'raysiz |
| **I/O-bound** | Vaqtning ko'p qismi kutish (tarmoq, disk) | Suv qaynashini kutish |
| **CPU-bound** | Vaqtning ko'p qismi hisoblash | Piyozni o'zingiz to'g'rash — qo'lingiz band |
| **Coroutine** | `async def` bilan yozilgan funksiya. Chaqirilganda darhol ishlamaydi, "bajarilishi mumkin bo'lgan ish" obyektini qaytaradi | Retsept kartochkasi: hali pishirilmagan |
| **`await`** | "Shu yerda natijani kut, lekin kutayotganda boshqa coroutine'larga navbat ber" | "Choy qaynaguncha boshqa ishga o'taman" |
| **Event loop** | Hamma coroutine'larni boshqaradigan dispetcher: kim kutyapti, kim davom eta oladi | Oshpaz: qaysi qozonga qarash kerakligini hal qiladi |
| **Task** | Event loop'ga topshirilgan, fonda ishlayotgan coroutine | Plitaga qo'yilgan qozon |
| **`asyncio.gather`** | Bir nechta coroutine'ni birdan ishga tushirib, hammasi tugashini kutadi | Uchala choynakni birdan qo'yish |
| **`Semaphore`** | Bir vaqtda ishlaydigan coroutine'lar sonini cheklaydi | Plitada faqat 4 ta ko'z bor |
| **Blocking (bloklovchi)** | Event loop'ni to'xtatib qo'yadigan sinxron kod (`time.sleep`, sync `OpenAI()`) | Oshpaz bitta qozonga tikilib qolgan, boshqalari kuyyapti |

**Eng muhim g'oya:** async'da hamma narsa **bitta oqim**da ishlaydi, lekin kutishlar ustma-ust tushadi. Bu bir vaqtda o'nta oshpaz emas, balki kutish vaqtini bekor o'tkazmaydigan **bitta** oshpaz.

## 3. Sync va async: vaqt chizmasi

Uchta LLM chaqiruvi, har biri 3 soniya. `░` — kutish (tarmoq), `█` — sizning kodingiz ishlayapti (juda qisqa).

**Sync (ketma-ket) — 9 soniya:**

```
vaqt (s):   0     1     2     3     4     5     6     7     8     9
so'rov A:   █░░░░░░░░░░░░░░░█
so'rov B:                     █░░░░░░░░░░░░░░░█
so'rov C:                                         █░░░░░░░░░░░░░░░█
```

**Async (gather) — ~3 soniya:**

```
vaqt (s):   0     1     2     3
so'rov A:   █░░░░░░░░░░░░░░░█
so'rov B:   █░░░░░░░░░░░░░░░█
so'rov C:   █░░░░░░░░░░░░░░░█
```

Kodingiz har bir so'rovni **yuboradi**, `await` da "men kutyapman, boshqasiga navbat" deydi, event loop esa keyingi so'rovni yuboradi. Uchalasi tarmoqda **bir vaqtda** kutadi. Javoblar kelganda kod har birini qayta ishlaydi.

**Qoida:** async'da umumiy vaqt ≈ **eng uzoq** so'rov vaqti. Sync'da esa **barcha** so'rovlar vaqtining yig'indisi.

## 4. Birinchi async kod

Yangi notebook oching: `notebooks/08_async.ipynb`. Hozircha API yo'q: LLM chaqiruvini `asyncio.sleep` bilan taqlid qilamiz. Bu tekin va natijani darhol ko'rasiz.

```python
import asyncio, time

async def fake_llm(name: str, seconds: float) -> str:
    print(f"{name}: yuborildi")
    await asyncio.sleep(seconds)        # tarmoqni kutish (taqlid)
    print(f"{name}: javob keldi")
    return f"{name} natijasi"
```

**Uchta qoida:**

1. `async def` bilan yozilgan funksiya **coroutine** bo'ladi.
2. Coroutine'ni oddiy chaqirsangiz **ishlamaydi**: `fake_llm("A", 1)` faqat `<coroutine object ...>` qaytaradi. Uni ishga tushirish uchun `await` kerak.
3. `await` faqat `async def` ichida (yoki Jupyter katakchasida) yoziladi.

**Jupyter'da va oddiy `.py` faylda farq bor:**

```python
# Jupyter katakchasida — to'g'ridan-to'g'ri await:
result = await fake_llm("A", 1)

# Oddiy .py faylda — asyncio.run orqali:
async def main():
    result = await fake_llm("A", 1)
    print(result)

asyncio.run(main())
```

Jupyter'da event loop **allaqachon ishlab turibdi**, shuning uchun u yerda `asyncio.run()` xato beradi: *"asyncio.run() cannot be called from a running event loop"*. `.py` faylda esa event loop'ni `asyncio.run()` o'zi yaratadi.

**Sinab ko'ring:** quyidagisini yozib, natijani kuzating.

```python
x = fake_llm("A", 1)
print(type(x))          # nima chiqadi?
print(await x)          # endi-chi?
```

## 5. `asyncio.gather`: bir nechtasini birdan

`await` ketma-ket yozilsa, kod baribir **ketma-ket** ishlaydi. Bu ko'p uchraydigan xato:

```python
t0 = time.perf_counter()
a = await fake_llm("A", 1)     # 1 s kutadi
b = await fake_llm("B", 1)     # keyin yana 1 s
c = await fake_llm("C", 1)     # keyin yana 1 s
print(f"{time.perf_counter() - t0:.1f} s")   # ~3.0 s
```

Bu yerda `await` faqat "kutayotganda boshqalarga navbat ber" deydi. Lekin "boshqalar" hali yaratilmagan, chunki B qatoriga A tugamaguncha yetib kelinmaydi.

**Parallel uchun `gather`:**

```python
t0 = time.perf_counter()
a, b, c = await asyncio.gather(
    fake_llm("A", 1),
    fake_llm("B", 1),
    fake_llm("C", 1),
)
print(f"{time.perf_counter() - t0:.1f} s")   # ~1.0 s
print(a, b, c)
```

**`gather` haqida bilish kerak bo'lgan narsalar:**

- Natijalar **siz bergan tartibda** qaytadi, qaysi biri birinchi tugaganidan qat'i nazar.
- Ro'yxat bilan ishlash uchun `*` ishlatiladi. Amalda ko'pincha shunday yoziladi:

```python
questions = ["savol 1", "savol 2", "savol 3", "savol 4"]
results = await asyncio.gather(*[fake_llm(q, 1) for q in questions])
```

`[fake_llm(q, 1) for q in questions]` coroutine'lar ro'yxatini yaratadi (ular hali ishlamaydi), `*` esa ro'yxatni alohida argumentlarga yoyadi.

**Sinab ko'ring:** `fake_llm("A", 3)`, `fake_llm("B", 1)`, `fake_llm("C", 2)` ni `gather` qiling. Print'lar qaysi tartibda chiqadi? Natijalar ro'yxati qaysi tartibda? Umumiy vaqt-chi?

## 6. `create_task` va `as_completed`

`gather` hamma narsa tugashini kutadi. Ba'zan esa boshqa narsa kerak bo'ladi.

**`asyncio.create_task`: fonda boshlash va keyinroq olish.** Coroutine'ni darhol ishga tushiradi, natijani esa keyin `await` qilasiz. Shu vaqt ichida boshqa ish qilish mumkin.

```python
task = asyncio.create_task(fake_llm("xulosa", 3))   # darhol boshlandi
print("Bu orada boshqa ish...")
await asyncio.sleep(1)
print("Hali ham boshqa ish...")
result = await task                                  # endi natijani olamiz (yana ~2 s)
```

**`asyncio.as_completed`: kim birinchi tugasa, o'sha birinchi.** Foydalanuvchiga natijalarni kelishi bilan ko'rsatish kerak bo'lganda qulay (masalan, 30 ta bob testidan tayyor bo'lganlarini ekranga chiqarib borish).

```python
coros = [fake_llm("A", 3), fake_llm("B", 1), fake_llm("C", 2)]
for next_done in asyncio.as_completed(coros):
    result = await next_done
    print("Tayyor:", result)       # B, C, A tartibida
```

|  | `gather` | `as_completed` |
| --- | --- | --- |
| Natija tartibi | Siz bergan tartibda | Tugash tartibida |
| Qachon olasiz | Hammasi tugagach, birdan | Har biri tugashi bilan |
| Qachon ishlatiladi | Hammasi kerak bo'lsa (workflow keyingi qadami) | Progress ko'rsatish, streaming UI |

Learning app'da: 30 ta bobga test → `as_completed` bilan foydalanuvchi birinchi testlarni 10 soniyada ko'radi, hammasini kutib o'tirmaydi.

## 7. `Semaphore`: bir vaqtda nechta?

**Muammo:** 1 000 ta chunk'ni tekshirish uchun 1 000 ta so'rovni `gather` bilan birdan yuborsangiz, OpenAI **rate limit** xatosini beradi (429). Datadog ma'lumotiga ko'ra production'dagi LLM xatolarining ~60% aynan shu.

**Yechim:** bir vaqtda ishlayotgan so'rovlar sonini cheklash. `Semaphore(5)` bir vaqtda faqat 5 ta coroutine o'tishiga ruxsat beradi, qolganlari navbatda kutadi.

```python
sem = asyncio.Semaphore(3)              # bir vaqtda ko'pi bilan 3 ta

async def limited_llm(name: str) -> str:
    async with sem:                     # bo'sh joy bo'lmasa, shu yerda kutadi
        return await fake_llm(name, 1)

t0 = time.perf_counter()
results = await asyncio.gather(*[limited_llm(f"#{i}") for i in range(9)])
print(f"{time.perf_counter() - t0:.1f} s")   # ~3 s: 9 ta so'rov, 3 tadan 3 to'lqin
```

**Vaqt chizmasi (Semaphore(3), 9 ta so'rov):**

```
1-to'lqin:  #0 #1 #2   ░░░░░░  (0-1 s)
2-to'lqin:  #3 #4 #5          ░░░░░░  (1-2 s)
3-to'lqin:  #6 #7 #8                 ░░░░░░  (2-3 s)
```

**Qancha qo'yish kerak?** OpenAI hisobingizning tier'iga bog'liq (daqiqadagi so'rov va token limiti). Boshlash uchun 5-10 xavfsiz son. Rate limit xatosi chiqsa — kamaytiring.

## 8. Xatolar va timeout

**Muammo:** `gather` ichidagi 30 ta so'rovdan bittasi xato bersa, standart holatda **butun `gather` xato bilan tugaydi** va qolgan 29 ta natija yo'qoladi.

```python
async def flaky(name: str) -> str:
    await asyncio.sleep(1)
    if name == "B":
        raise ValueError("B ishlamadi")
    return f"{name} ok"

# Standart: bitta xato hammasini buzadi
try:
    results = await asyncio.gather(flaky("A"), flaky("B"), flaky("C"))
except ValueError as e:
    print("Hammasi yo'qoldi:", e)
```

**Yechim: `return_exceptions=True`.** Xatolar natija sifatida qaytadi, qolganlari saqlanadi:

```python
results = await asyncio.gather(flaky("A"), flaky("B"), flaky("C"), return_exceptions=True)
for r in results:
    if isinstance(r, Exception):
        print("xato:", r)
    else:
        print("ok:", r)
```

**Timeout: osilib qolgan so'rovni to'xtatish.** Ba'zan so'rov juda uzoq kutib qoladi. `asyncio.wait_for` vaqt chegarasini qo'yadi:

```python
try:
    r = await asyncio.wait_for(fake_llm("sekin", 10), timeout=2)
except asyncio.TimeoutError:
    print("2 soniyada javob kelmadi")
```

OpenAI SDK'da timeout va retry allaqachon bor: `AsyncOpenAI(timeout=60, max_retries=3)` — LLM API qo'llanmasidagi kabi. `wait_for` esa butun bir qadam (masalan, "test tuzish + tekshirish") uchun umumiy chegara qo'yishda kerak.

**Learning app uchun qoida:** ko'p so'rovli workflow'da har doim `return_exceptions=True` ishlating va xato bergan elementlarni alohida qayta ishlang (qayta urinish yoki tashlab yuborish). Bitta bobning xatosi butun kitobni buzmasin.

## 9. `AsyncOpenAI`: haqiqiy LLM bilan

`AsyncOpenAI` — oddiy `OpenAI` ning async versiyasi. Metodlar bir xil, faqat oldiga `await` yoziladi.

| Sync | Async |
| --- | --- |
| `client = OpenAI()` | `client = AsyncOpenAI()` |
| `resp = client.responses.create(...)` | `resp = await client.responses.create(...)` |
| `resp = client.responses.parse(...)` | `resp = await client.responses.parse(...)` |

```python
from openai import AsyncOpenAI

client = AsyncOpenAI(timeout=60, max_retries=3)
MODEL = "gpt-6-luna"
sem = asyncio.Semaphore(5)

async def summarize(text: str) -> str:
    async with sem:
        resp = await client.responses.create(
            model=MODEL,
            instructions="Matnni bitta gapda xulosa qil.",
            input=text,
        )
    return resp.output_text

texts = ["Python - dasturlash tili...", "Docker - konteyner...", "Postgres - baza..."]

t0 = time.perf_counter()
summaries = await asyncio.gather(*[summarize(t) for t in texts], return_exceptions=True)
print(f"{time.perf_counter() - t0:.1f} s")
for s in summaries:
    print("-", s)
```

Bu misolda bugungi barcha qismlar bor: `AsyncOpenAI`, `Semaphore`, `gather`, `return_exceptions`. Workflow demo (`07_workflow_demo.ipynb`) aynan shu naqshni test tuzish va tekshirishga qo'llaydi.

**FastAPI'da ham shu:** endpoint `async def` bo'lsa, ichida `AsyncOpenAI` ishlating. Shunda bitta sekin LLM chaqiruvi boshqa foydalanuvchilarning so'rovlarini to'xtatib qo'ymaydi.

## 10. Tez-tez uchraydigan xatolar

| Xato | Belgisi | Tuzatish |
| --- | --- | --- |
| **`await` unutildi** | Natija o'rniga `<coroutine object ...>`, ogohlantirish: *"coroutine was never awaited"* | Chaqiruv oldiga `await` qo'ying |
| **Ketma-ket `await`** | Kod ishlaydi, lekin tezlashmadi | `asyncio.gather` ishlating |
| **`async def` ichida `time.sleep`** | Hamma narsa to'xtab qoladi, parallel ishlamaydi | `await asyncio.sleep` |
| **Async ichida sync `OpenAI()`** | Kod ishlaydi, lekin parallel emas | `AsyncOpenAI` + `await` |
| **Jupyter'da `asyncio.run`** | *"cannot be called from a running event loop"* | Jupyter'da to'g'ridan-to'g'ri `await` |
| **Cheksiz `gather`** | 429 rate limit xatolari | `Semaphore` |
| **Bitta xato hammasini buzdi** | Bitta `Exception`, hamma natija yo'q | `return_exceptions=True` |
| **CPU ishni async qilish** | Lokal BGE-M3 embedding async bilan tezlashmadi | Async I/O uchun; CPU uchun batch yoki boshqa usul |

**Eng xavflisi — 3 va 4-qatorlar:** kod xato bermaydi, faqat sekin ishlaydi. Bitta **bloklovchi** chaqiruv butun event loop'ni to'xtatadi: oshpaz bitta qozonga tikilib qoladi va qolgan hamma qozonlar kutadi.

**Qanday tekshirish:** vaqtni o'lchang. 3 ta 1 soniyalik so'rov ~3 soniya olsa — parallel ishlamayapti, yuqoridagi xatolardan biri bor.

## 11. Laravel/PHP bilan solishtirish

PHP'da odatda har bir so'rov sinxron ishlaydi, shuning uchun bu tushuncha sizga yangi bo'lishi tabiiy. Lekin o'xshash narsalarni allaqachon ishlatgansiz:

| Laravel / PHP | Python async |
| --- | --- |
| `Http::get()` — javob kelguncha kutadi | `await client.get()` |
| `Http::pool(fn ($pool) => [...])` — bir nechta HTTP so'rov birdan | `asyncio.gather(...)` |
| Queue + job'lar — og'ir ishni fonga berish | `asyncio.create_task(...)` (bitta jarayon ichida) |
| Queue worker soni / rate limiting middleware | `Semaphore(n)` |
| JavaScript `Promise.all([...])` | `asyncio.gather(...)` |
| JavaScript `async/await` | Deyarli bir xil sintaksis |

**Farqi:** Laravel queue — alohida jarayonlar (worker'lar), ish Redis orqali uzatiladi va server qayta ishga tushsa ham saqlanadi. `asyncio` esa **bitta jarayon ichida**, xotirada ishlaydi: dastur to'xtasa, ishlar yo'qoladi. Uzoq va muhim ishlar uchun (masalan, 1 000 betlik PDF'ni indekslash) Python'da ham queue kerak bo'ladi (Celery, RQ, arq) — bu Docker va deploy bosqichida.

## 12. Mashqlar

`notebooks/08_async.ipynb` da bajaring. Har mashqda **avval natijani taxmin qilib yozing** (markdown katakchada), keyin run qiling va solishtiring. 1-6 mashqlar tekin — faqat `asyncio.sleep`.

1. **Coroutine obyekti.** `fake_llm("A", 1)` ni `await`siz chaqirib `type()` ini chiqaring. Keyin `await` bilan. Farqni bir gapda yozing.
2. **Tartib.** `gather(fake_llm("A", 3), fake_llm("B", 1), fake_llm("C", 2))`. Print'lar qaysi tartibda chiqadi? Qaytgan ro'yxat qaysi tartibda? Umumiy vaqt?
3. **Ketma-ket tuzog'i.** Uchta `await` ni ketma-ket yozing va vaqtni o'lchang. Keyin `gather` bilan. Necha barobar farq?
4. **Bloklovchi xato.** `fake_llm` ichidagi `await asyncio.sleep` ni `time.sleep` ga almashtiring va `gather` bilan 3 tasini ishga tushiring. Vaqt qancha bo'ldi? Nega?
5. **Semaphore.** 10 ta so'rov, har biri 1 s, `Semaphore(4)`. Umumiy vaqtni oldindan hisoblang, keyin tekshiring.
6. **Xato.** 5 ta `flaky` so'rov, bittasi xato beradi. Avval `return_exceptions` siz, keyin u bilan. Har holatda nechta natija qoldi?
7. **as_completed.** 5 ta har xil uzunlikdagi `fake_llm` ni `as_completed` bilan ishga tushirib, "N/5 tayyor" progress chiqaring.
8. **Haqiqiy LLM** (bir necha sent). `AsyncOpenAI` bilan 6 ta qisqa matnga bitta gaplik xulosa: avval sync `OpenAI` bilan ketma-ket, keyin async `gather` bilan. Ikkala vaqtni yozing.

Tugatgach — [[07_workflow_demo]] ni qayta o'qing: endi har bir `await` va `gather` nima qilayotgani tushunarli bo'lishi kerak. Keyin A3-topshiriq.

## 13. Manbalar

- [Python docs: asyncio — Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html) — `gather`, `create_task`, `as_completed`, `wait_for`, `TaskGroup`
- [Python docs: asyncio — Synchronization primitives](https://docs.python.org/3/library/asyncio-sync.html) — `Semaphore`
- [OpenAI Python SDK (GitHub)](https://github.com/openai/openai-python) — `AsyncOpenAI`, timeout va retry sozlamalari
- [FastAPI: Concurrency and async / await](https://fastapi.tiangolo.com/async/) — FastAPI muallifining async haqidagi mashhur tushuntirishi (burger misoli)

**Vaultdagi bog'liq darslar:** [[LLM API to'liq qo'llanma]] (timeout, retry, async bo'limi), [[FastAPI va streaming qo'llanma]], [[Structured output to'liq qo'llanma]].


## Bog'liq tushunchalar
[[Async]] · [[Coroutine]] · [[Event loop]] · [[asyncio.gather]] · [[Semaphore]] · [[I-O bound va CPU bound]] · [[Rate limit]] · [[Parallelization]]
