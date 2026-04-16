# Classement — Modèles locaux pour **RAG et embeddings**

> Cible matérielle : 16 GB VRAM + 128 GB RAM. Inférence via Ollama.
> Focus : embeddings, reranking, vector stores, pipelines RAG locaux complets.

---

## 🏗️ Architecture d'un pipeline RAG en 2026

```
┌────────────┐  ┌──────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────┐
│  Document  ├─▶│   Chunking   ├─▶│  Embedding  ├─▶│ Vector Store│  │  LLM   │
└────────────┘  └──────────────┘  └─────────────┘  └─────────────┘  └────────┘
                                                          │              ▲
                                                          ▼              │
                                                   ┌─────────────┐  ┌────┴───┐
              Query ─────────────────────────────▶ │  Retrieval  ├─▶│Rerank.│
                                                   │  (top 50)   │  │(top 5)│
                                                   └─────────────┘  └────────┘
```

**4 modèles** sont impliqués :

1. **Embedder** (vectorise document + query)
2. **Reranker** (cross-encoder, optionnel mais recommandé)
3. **LLM génératif** (synthétise la réponse à partir du contexte)
4. **Hybrid search BM25** (optionnel, pour les mots-clés exacts)

---

## 🏆 Top 5 — Modèles d'embeddings sur 16 GB VRAM

| #   | Modèle                       | Taille | MTEB      | Contexte   | Multilingue  |
| --- | ---------------------------- | ------ | --------- | ---------- | ------------ |
| 1   | **Qwen3-Embedding 8B**       | 8B     | **70.58** | 32k        | ⭐⭐⭐⭐⭐ (100+) |
| 2   | **BGE-M3**                   | 568M   | 63.0      | 8k         | ⭐⭐⭐⭐⭐ (100+) |
| 3   | **mxbai-embed-large**        | 335M   | 64.68     | **512** ⚠️ | ⭐⭐ (EN)      |
| 4   | **nomic-embed-text**         | 137M   | 62.39     | 8k         | ⭐⭐ (EN)      |
| 5   | **Snowflake Arctic Embed L** | 335M   | 55.98     | 2k         | ⭐⭐⭐          |

> **Repère** : `text-embedding-3-small` d'OpenAI = 62.3 MTEB, `text-embedding-3-large` = 64.6.
> Tous les modèles ci-dessus **égalent ou dépassent** OpenAI.

---

## 1. Qwen3-Embedding 8B — 🥇 le meilleur tout court

**Premier modèle local à dépasser tous les modèles commerciaux** (OpenAI, Cohere, Voyage). 70.58 MTEB.

### Forces

- **100+ langues** (FR/EN/CJK excellents)
- Output dimension **flexible** : 32 → 1024 (Matryoshka)
- Conçu aussi pour le **code** (CodeSearchNet)
- Tient en 16 GB VRAM en Q4 (~6 GB)

### Installation

```bash
ollama pull qwen3-embedding:8b
```

### Usage Python

```python
import ollama
emb = ollama.embeddings(model="qwen3-embedding:8b", prompt="Mon texte à vectoriser")
vector = emb["embedding"]  # 1024 dimensions par défaut
```

### Quand l'utiliser

- Production sérieuse, qualité avant tout
- Corpus multilingue (FR + EN + autres)
- RAG sur du code ET du texte mixte
- Projets où le coût Cohere/OpenAI devient prohibitif

---

## 2. BGE-M3 — 🥈 hybrid search en un seul modèle

568M params, **génère 3 types de représentations en une seule passe** :

- Dense (vecteur classique)
- Sparse (BM25-like, mots-clés)
- Multi-vector (ColBERT-like)

### Forces

- **Hybrid search natif** sans avoir à maintenir un index BM25 séparé
- 100+ langues
- 8k contexte
- Apache 2.0
- Excellent compromis qualité/vitesse

### Installation

```bash
ollama pull bge-m3
```

### Quand l'utiliser

- Recherche sur documents longs avec mix concept + mots-clés exacts
- Architecture simplifiée (un seul modèle pour dense + sparse)
- Multilingue avec qualité homogène

---

## 3. mxbai-embed-large — fast English

Excellent pour l'anglais pur, mais **contexte de 512 tokens seulement** → exige un chunking agressif.

### Installation

```bash
ollama pull mxbai-embed-large
```

### À utiliser uniquement si

- Corpus 100 % anglais
- Documents courts (< 512 tokens) ou chunks fins
- Budget VRAM serré (~700 MB)

⚠️ **Ne pas utiliser pour du français** ou des documents longs.

---

## 4. nomic-embed-text — le plus auditable

**Seul modèle complètement open** : poids + code d'entraînement + dataset publics.

### Forces

- Reproductibilité totale (compliance, audit)
- 137M params, ultra-léger (~300 MB)
- 8k contexte
- Tourne sur CPU à latence acceptable

### Installation

```bash
ollama pull nomic-embed-text
```

### Quand l'utiliser

- Industries régulées (santé, finance, défense)
- Besoin de documenter chaque modèle de la stack
- Déploiement edge / CPU only
- ⚠️ Pas top en multilingue (~ 0 sur cross-lingual)

### Variante code

Nomic propose aussi **`nomic-embed-code` (7B)** qui bat Voyage Code 3 sur CodeSearchNet :

```bash
ollama pull nomic-embed-code
```

---

## 5. Snowflake Arctic Embed L — petit footprint

335M params, optimisé pour les contraintes hardware.

```bash
ollama pull snowflake-arctic-embed:large
```

**Niche** : edge deployments, applications mobiles, pipelines avec contraintes mémoire fortes.

---

## 🎯 Décision rapide — embedder

| Scénario                           | Modèle                                       |
| ---------------------------------- | -------------------------------------------- |
| Production RAG multilingue (FR+EN) | **Qwen3-Embedding 8B**                       |
| Hybrid search (dense + sparse)     | **BGE-M3**                                   |
| Anglais seulement, vitesse max     | **mxbai-embed-large**                        |
| Compliance / audit complet         | **nomic-embed-text**                         |
| RAG sur code source                | **nomic-embed-code** ou **Qwen3-Embedding**  |
| Edge / CPU only                    | **all-MiniLM-L6-v2** ou **nomic-embed-text** |
| Documents très longs (8k+)         | **BGE-M3** ou **Qwen3-Embedding**            |

---

## 🔄 Top 3 — Modèles de reranking (cross-encoders)

Les rerankers ajoutent **+15 à +40 % de précision** sur le retrieval. Pipeline standard :

```
Embeddings (top 50) → Reranker (top 5) → LLM
```

| #   | Modèle                           | Taille | Latence GPU | Multilingue  |
| --- | -------------------------------- | ------ | ----------- | ------------ |
| 1   | **BGE-reranker-v2-m3**           | 568M   | 50-100 ms   | ⭐⭐⭐⭐⭐ (100+) |
| 2   | **Jina Reranker v3**             | ~600M  | < 200 ms    | ⭐⭐⭐⭐⭐ (100+) |
| 3   | **GTE-reranker-modernbert-base** | 149M   | 80 ms       | ⭐⭐⭐ (EN+)    |

### 1. BGE-reranker-v2-m3 — 🥇 le standard

Apache 2.0, **égale Cohere Rerank gratuitement**. Premier choix par défaut.

```bash
# Pas dans Ollama — utiliser sentence-transformers ou FlagEmbedding
pip install -U FlagEmbedding
```

```python
from FlagEmbedding import FlagReranker
reranker = FlagReranker('BAAI/bge-reranker-v2-m3', use_fp16=True)
score = reranker.compute_score(['ma question', 'document candidat'])
```

### 2. Jina Reranker v3 — sub-200ms

**Le plus rapide** dans le top tier. Spécialement bon pour code et function-calling.

### 3. GTE-reranker-modernbert-base — léger

149M params seulement, latence excellente, qualité top sur l'anglais.

### Quand sauter le reranker ?

- Corpus < 1000 documents (BM25 + dense suffit)
- Latence critique (< 100 ms total)
- Embeddings déjà très spécialisés

---

## 🗄️ Vector stores locaux

| Outil        | Forces                                           | Faiblesses                |
| ------------ | ------------------------------------------------ | ------------------------- |
| **Qdrant** ⭐ | Performant, Rust, hybrid search natif, REST/gRPC | Complexité config         |
| **Chroma**   | Ultra-simple, parfait pour prototypage           | Moins scalable            |
| **pgvector** | Intégré PostgreSQL, ACID, joins SQL              | Moins rapide pure-vector  |
| **Weaviate** | Hybrid + GraphQL + multi-modal                   | Lourd                     |
| **LanceDB**  | Embedded, sans serveur, ultra-rapide             | Jeune écosystème          |
| **FAISS**    | Le plus rapide, mais bibliothèque pure           | Pas de persistence native |

### Recommandations 2026

| Cas d'usage                           | Vector store              |
| ------------------------------------- | ------------------------- |
| Production sérieuse, multi-tenant     | **Qdrant**                |
| Application Postgres existante        | **pgvector**              |
| Prototypage rapide                    | **Chroma** ou **LanceDB** |
| Recherche multimodale (texte + image) | **Weaviate**              |
| Embedded dans une app (sans serveur)  | **LanceDB**               |
| Performance pure (1M+ vecteurs)       | **Qdrant** ou **FAISS**   |

### Lancer Qdrant en local

```bash
docker run -p 6333:6333 -p 6334:6334 \
  -v $(pwd)/qdrant_storage:/qdrant/storage:z \
  qdrant/qdrant
```

### Lancer pgvector

```sql
-- Sur PostgreSQL 16+
CREATE EXTENSION vector;
CREATE TABLE docs (
  id bigserial PRIMARY KEY,
  content text,
  embedding vector(1024)
);
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
```

---

## ✂️ Stratégies de chunking

Le chunking conditionne **plus** la qualité du RAG que le choix du modèle d'embedding.

### Tailles recommandées par usage

| Type de doc               | Chunk size           | Overlap                  |
| ------------------------- | -------------------- | ------------------------ |
| Articles techniques       | 512-1024 tokens      | 100-200                  |
| Code source               | par fonction/classe  | 0 (avec contexte parent) |
| Conversation / chat logs  | par message          | 0                        |
| Documentation API         | par endpoint/section | 50                       |
| Livres / longs documents  | 1024-2048 tokens     | 200-400                  |
| Réglementaire / juridique | par article/clause   | 0                        |

### Stratégies par sophistication croissante

#### 1. Fixed-size (baseline)

```python
chunks = [text[i:i+1024] for i in range(0, len(text), 800)]  # overlap 224
```

Simple mais coupe au milieu des phrases.

#### 2. Sentence-aware

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1024,
    chunk_overlap=200,
    separators=["\n\n", "\n", ". ", " ", ""]
)
```

#### 3. Semantic chunking (recommandé 2026)

Découpe aux **vrais changements de sujet** (basé sur la similarité d'embedding entre phrases) :

```python
from langchain_experimental.text_splitter import SemanticChunker
chunker = SemanticChunker(embeddings, breakpoint_threshold_type="percentile")
```

#### 4. Document-aware

- Markdown → couper aux `#`, `##`
- Code → couper par AST (par fonction/classe avec `tree-sitter`)
- HTML → couper par balises sémantiques
- PDF → préserver la structure (tables, figures)

#### 5. Contextual chunking (Anthropic, 2024)

Préfixer chaque chunk par un résumé du document parent (généré par LLM). **Améliore le retrieval de 35-50 %**.

```python
for chunk in chunks:
    context = llm.generate(f"Résume en 2 phrases ce chunk dans son contexte global du doc: {full_doc}\n\nChunk: {chunk}")
    chunk.metadata["context"] = context
    chunk.text = f"{context}\n\n{chunk.text}"
```

---

## 🔍 Hybrid search (dense + sparse)

Combiner **embeddings (sémantique)** + **BM25 (mots-clés exacts)** améliore le retrieval de **20-40 %** sur les requêtes contenant des termes rares (noms propres, codes, jargon).

### Avec BGE-M3 (en une passe)

```python
from FlagEmbedding import BGEM3FlagModel
model = BGEM3FlagModel('BAAI/bge-m3', use_fp16=True)
out = model.encode(texts, return_dense=True, return_sparse=True, return_colbert_vecs=True)
```

### Avec Qdrant (vecteurs séparés)

```python
client.create_collection(
    collection_name="docs",
    vectors_config={
        "dense": VectorParams(size=1024, distance=Distance.COSINE),
    },
    sparse_vectors_config={
        "bm25": SparseVectorParams(),
    }
)
```

### Fusion des scores : Reciprocal Rank Fusion (RRF)

```python
def rrf(rankings: list[list[int]], k=60) -> list[int]:
    scores = {}
    for ranking in rankings:
        for rank, doc_id in enumerate(ranking):
            scores[doc_id] = scores.get(doc_id, 0) + 1 / (k + rank)
    return sorted(scores, key=scores.get, reverse=True)
```

---

## 🧠 LLM générateur — la phase finale

Une fois le contexte récupéré (top 5-10 chunks), un LLM génère la réponse. Pour ça :

| Cas d'usage RAG         | Modèle (voir docs dédiés)                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| RAG général (Q&A)       | **Qwen3 30B-A3B Instruct** ([analyse-resume](models-03-analyse-resume.md))                   |
| RAG sur code            | **Qwen3-Coder 30B-A3B** ([code-agentique](models-01-code-agentique.md))                      |
| RAG multilingue (FR)    | **Mistral Small 3.1** ([traduction](models-02-traduction.md))                                |
| RAG sur SQL/data        | **Qwen3-Coder** ou **DeepSeek-Coder V2** ([data-engineering](models-04-data-engineering.md)) |
| RAG analytique critique | **DeepSeek-R1** ([analyse-resume](models-03-analyse-resume.md))                              |

---

## 🧪 Évaluation d'un pipeline RAG

### Métriques essentielles

| Métrique              | Mesure                                        | Outil           |
| --------------------- | --------------------------------------------- | --------------- |
| **Hit@k**             | Le bon doc est-il dans le top k ?             | RAGAS, Trulens  |
| **MRR**               | À quel rang apparaît le bon doc ?             | RAGAS           |
| **Faithfulness**      | La réponse cite-t-elle vraiment le contexte ? | RAGAS, DeepEval |
| **Answer relevancy**  | La réponse répond-elle à la question ?        | RAGAS           |
| **Context precision** | Les chunks récupérés sont-ils pertinents ?    | RAGAS           |
| **Latency P95**       | 95e centile du temps total                    | Prometheus      |

### Frameworks d'évaluation

```bash
pip install ragas deepeval trulens-eval
```

**RAGAS** est le standard 2026 pour évaluer un pipeline RAG :

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision

result = evaluate(
    dataset=eval_dataset,
    metrics=[faithfulness, answer_relevancy, context_precision]
)
```

---

## 🛠️ Frameworks RAG complets

| Framework            | Forces                                         | Quand l'utiliser     |
| -------------------- | ---------------------------------------------- | -------------------- |
| **LlamaIndex**       | Le plus complet pour RAG, abstractions claires | Projets RAG sérieux  |
| **LangChain**        | Énorme écosystème, intégrations                | Multi-agents + RAG   |
| **Haystack**         | Production-ready, pipelines déclaratifs        | Recherche entreprise |
| **txtai**            | Tout-en-un (embeddings + vector + LLM)         | Simplicité           |
| **Verba** (Weaviate) | UI prête à l'emploi                            | Démos / POC          |

### Stack minimale 2026 recommandée

```
Langchain/LlamaIndex (orchestration)
  ├─ Qdrant (vector store)
  ├─ Ollama : qwen3-embedding:8b (embedder)
  ├─ FlagEmbedding : bge-reranker-v2-m3 (reranker)
  └─ Ollama : qwen3:30b-a3b-instruct-2507 (LLM)
```

---

## 🚀 Pipeline RAG complet — exemple Python

```python
import ollama, qdrant_client
from qdrant_client.models import VectorParams, Distance, PointStruct
from FlagEmbedding import FlagReranker

# 1. Setup
qdrant = qdrant_client.QdrantClient(url="http://localhost:6333")
qdrant.recreate_collection(
    collection_name="docs",
    vectors_config=VectorParams(size=1024, distance=Distance.COSINE)
)
reranker = FlagReranker('BAAI/bge-reranker-v2-m3', use_fp16=True)

EMB_MODEL = "qwen3-embedding:8b"
LLM_MODEL = "qwen3:30b-a3b-instruct-2507"

def embed(text: str) -> list[float]:
    return ollama.embeddings(model=EMB_MODEL, prompt=text)["embedding"]

# 2. Indexation
def index(docs: list[str]):
    points = [
        PointStruct(id=i, vector=embed(d), payload={"text": d})
        for i, d in enumerate(docs)
    ]
    qdrant.upsert(collection_name="docs", points=points)

# 3. Recherche + rerank + génération
def rag(query: str, top_k_search=20, top_k_rerank=5) -> str:
    # Retrieval
    hits = qdrant.search(
        collection_name="docs",
        query_vector=embed(query),
        limit=top_k_search
    )
    candidates = [(h.payload["text"], h.score) for h in hits]

    # Reranking
    pairs = [[query, c[0]] for c in candidates]
    rerank_scores = reranker.compute_score(pairs)
    ranked = sorted(zip(candidates, rerank_scores), key=lambda x: -x[1])
    top_chunks = [c[0][0] for c in ranked[:top_k_rerank]]

    # Génération
    context = "\n\n---\n\n".join(top_chunks)
    prompt = f"""Réponds à la question en t'appuyant UNIQUEMENT sur le contexte ci-dessous.
Si l'information n'est pas dans le contexte, dis-le.

Contexte :
{context}

Question : {query}

Réponse :"""
    return ollama.generate(model=LLM_MODEL, prompt=prompt)["response"]

# Usage
index(["doc 1...", "doc 2...", "doc 3..."])
print(rag("Quelle est la politique de remboursement ?"))
```

---

## ⚠️ Pièges fréquents

### 1. Embedder différent pour query et docs

**TOUJOURS** utiliser le même modèle d'embedding pour vectoriser les documents et les requêtes. Sinon les distances cosinus sont incohérentes.

### 2. Chunks trop petits

Chunks < 200 tokens → contexte insuffisant pour le LLM.
Chunks > 2048 tokens → bruit excessif, "lost in the middle".

### 3. Pas de reranker

Sauter le reranker = perdre 20-40 % de précision sur les requêtes complexes. Pour 50-100 ms de latence en plus, c'est presque toujours rentable.

### 4. Top-k trop petit avant reranking

Récupérer top 5 directement = manquer souvent le bon doc. Récupérer top 20-50 → reranker → top 5.

### 5. Pas d'évaluation

Sans set d'évaluation (50-100 questions avec réponses attendues), impossible de savoir si une modif améliore le pipeline. Toujours commencer par construire ce set.

### 6. Re-indexer à chaque modification

Garder un **versioning de l'index** : chaque changement d'embedder ou de chunking nécessite une réindexation complète. Prévoir des indexes par version (`docs_v1`, `docs_v2`).

### 7. Hallucinations malgré le RAG

Le LLM peut **ignorer le contexte** si le prompt n'est pas strict. Toujours préciser : *« Réponds UNIQUEMENT à partir du contexte. Si l'info n'y est pas, dis "Information non disponible". »*

---

## 🎯 Configuration finale recommandée (16 GB VRAM)

```
Embedder       : qwen3-embedding:8b    (~6 GB VRAM)
Reranker       : bge-reranker-v2-m3    (~1 GB VRAM)
LLM générateur : qwen3:30b-a3b-instruct-2507 (~6-8 GB VRAM)
Vector store   : Qdrant (Docker, RAM/disk seulement)
Frame-work     : LlamaIndex
```

Total VRAM : **~13-14 GB** → tient confortablement.

Avantages : tout local, pas de coût API, FR/EN excellent, < 2 s par requête.

---

## Sources

- [Best Embedding Models for RAG 2026 — Milvus](https://milvus.io/blog/choose-embedding-model-rag-2026.md)
- [Best Embedding Models for RAG 2026 — PremAI](https://blog.premai.io/best-embedding-models-for-rag-2026-ranked-by-mteb-score-cost-and-self-hosting/)
- [Ollama Embedding Models: Benchmarks — Morph](https://www.morphllm.com/ollama-embedding-models)
- [Best Open-Source Embedding Models 2026 — BentoML](https://www.bentoml.com/blog/a-guide-to-open-source-embedding-models)
- [Which Embedding Model 2026 — Cheney Zhang Benchmark](https://zc277584121.github.io/rag/2026/03/20/embedding-models-benchmark-2026.html)
- [Top 7 Rerankers for RAG — Analytics Vidhya](https://www.analyticsvidhya.com/blog/2025/06/top-rerankers-for-rag/)
- [Best Reranker Models 2026 — BSWEN](https://docs.bswen.com/blog/2026-02-25-best-reranker-models/)
- [Best Reranker for RAG — Agentset](https://agentset.ai/blog/best-reranker)
- [RAG Embeddings & Rerankers — LlamaIndex](https://www.llamaindex.ai/blog/boosting-rag-picking-the-best-embedding-reranker-models-42d079022e83)
- [Ultimate Guide to Reranking Models 2026 — ZeroEntropy](https://www.zeroentropy.dev/articles/ultimate-guide-to-choosing-the-best-reranking-model-in-2025)
