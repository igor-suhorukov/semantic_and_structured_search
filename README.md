# Semantic + Structured Text Search

A semantic + structured text search tool powered by **EmbeddingGemma 300M** via wllama and **DuckDB-WASM**. Runs entirely in your browser — no server required.

## What it does

Source articles are processed using an LLM to extract structured metadata (genres, topics, themes, sentiment, NSFW flags, etc.) and stored in a database. Embedding vectors are then computed from derived fields. The resulting collection is available for local search both by semantic proximity (vector distance) and by keywords or structured features.

## Installation

Clone the repository:

```bash
git clone https://github.com/igor-suhorukov/semantic_and_structured_search.git
cd semantic_and_structured_search
```

## Running

Start a local HTTP server:

```bash
python3 -m http.server 5000
```

Then open [http://localhost:5000](http://localhost:5000) in your browser.

## Architecture

| Component | Technology |
|---|---|
| Embedding model | EmbeddingGemma 300M (Q4_0) via wllama — runs client-side |
| Vector search | Cosine similarity over flat parquet partitions (`text_embeddings` view) |
| Structured data | 30-column `indexed_text` view loaded from a single parquet file (includes nested STRUCT, MAP, and ARRAY types) |
| Filters | DuckDB SQL WHERE-clause filters applied directly to structured columns |

### Semantic search workflow

1. Enter a free-text query.
2. The running LLM computes a 768-dim L2-normalized embedding vector with wllama.
3. DuckDB-WASM compares this vector against all stored embeddings using `list_cosine_similarity`.
4. Results are joined with `indexed_text` to display original text and metadata.

### Available embedded fields

Each record can produce embeddings for multiple sub-fields:

| Name | Source |
|---|---|
| `summary` | `s.summary` |
| `meaning` | `s.meaning` |
| `nsfwReason` | `s.nsfw_reasons[]` |
| `insight` | `s.key_insights[]` |
| `contradictory` | `s.contradictory_statements[]` |
| `theme` | `s.themes[]` |
| `topic` | `s.topics[]['topic_description']` |
| `audience` | `s.target_audience['audience_description']` |

### Filters

- **Single-value** — (Primary Genre, Sentiment Polarity, Completeness, Target Audience, Demagoguery Severity)
- **Multi-value** — (Secondary Genres, Keywords, Demagoguery Techniques)
- **Tristate** — Yes / No / All (Is NSFW, Has Advertising)

Multiple filters are AND-ed together in the SQL.