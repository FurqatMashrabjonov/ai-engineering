---
tags: [rag, tushuncha]
status: o'rganilmagan
---
# Vector search

Savol embedding'iga eng yaqin bo'laklarni topish. PostgreSQL + pgvector: `ORDER BY embedding <=> q LIMIT k`, HNSW indeksi bilan tezlashtiriladi. Filtr bilan ishlaganda `hnsw.iterative_scan` yoqiladi.

## Bog'liq
- [[Embedding]]
- [[pgvector]]
- [[Hybrid search]]
- [[Recall@k]]

## Manba
- [[RAG to'liq qo'llanma#6. PostgreSQL + pgvector]]
- [[RAG (xarita)]]
