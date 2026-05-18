# Insurellm RAG Chatbot

A Retrieval-Augmented Generation (RAG) chatbot for the fictional insurance company **Insurellm**. Ask questions about company products, employees, contracts, and culture — the chatbot retrieves relevant context from a knowledge base and answers accurately using GPT-4.1.

## How It Works

```
User Question
     │
     ▼
Query Rewriting (LLM)          ← refines the question for better KB search
     │
     ▼
Dual Retrieval (ChromaDB)      ← searches with both original + rewritten query
     │
     ▼
LLM Re-ranking                 ← picks the 10 most relevant chunks from ~40 candidates
     │
     ▼
Answer Generation (GPT-4.1)   ← answers grounded in retrieved context
     │
     ▼
Gradio UI (answer + context shown side by side)
```

## Features

- **Query rewriting** — LLM refines vague questions into precise KB queries before retrieval
- **Dual retrieval** — runs both the original and rewritten query, merges results
- **LLM re-ranking** — scores and reorders candidates to surface the most relevant chunks
- **Rich chunks** — each KB chunk contains an AI-generated headline, summary, and original text, improving semantic search quality
- **Side-by-side UI** — chat on the left, retrieved source context on the right
- **Evaluation dashboard** — measure retrieval (MRR, nDCG, keyword coverage) and answer quality (accuracy, completeness, relevance)

## Knowledge Base

The knowledge base covers four categories across 76 documents:

| Category | Contents |
|---|---|
| `company` | Overview, culture, careers, about |
| `products` | 8 insurance platforms (Carllm, Homellm, Lifellm, Bizllm, Claimllm, Healthllm, Markellm, Rellm) |
| `contracts` | Client contracts for each product |
| `employees` | Employee profiles |

## Setup

### Prerequisites

- [uv](https://docs.astral.sh/uv/) — Python package manager
- OpenAI API key

### Install

```bash
git clone git@github.com:viv-eak/IntelligentRagChatBot.git
cd IntelligentRagChatBot
uv sync
```

### Configure

Create a `.env` file in the project root:

```
OPENAI_API_KEY=sk-...
```

### Build the vector database

This step uses GPT to create semantically rich chunks — runs once, takes ~5 minutes.

```bash
uv run pro_implementation/ingest.py
```

### Run the chatbot

```bash
uv run app.py
```

Opens at `http://127.0.0.1:7860`

### Run the evaluation dashboard

```bash
uv run evaluator.py
```

## Project Structure

```
├── app.py                      # Gradio chatbot UI
├── evaluator.py                # Gradio evaluation dashboard
│
├── pro_implementation/
│   ├── answer.py               # Query rewriting, dual retrieval, re-ranking, RAG answer
│   └── ingest.py               # LLM-based chunking + embedding into preprocessed_db
│
├── implementation/             # Simpler baseline (rule-based chunking, single retrieval)
│   ├── answer.py
│   └── ingest.py
│
├── evaluation/
│   ├── eval.py                 # Retrieval + answer quality evaluation logic
│   ├── test.py                 # Test question loader
│   └── tests.jsonl             # Ground-truth QA pairs with keywords
│
├── knowledge-base/             # Source documents (Markdown)
│   ├── company/
│   ├── products/
│   ├── contracts/
│   └── employees/
│
├── pyproject.toml              # Dependencies (managed by uv)
└── uv.lock
```

## Tech Stack

| Component | Library |
|---|---|
| UI | Gradio |
| LLM calls | LiteLLM + OpenAI SDK |
| Embeddings | OpenAI `text-embedding-3-large` |
| Vector store | ChromaDB |
| Retries | Tenacity |
| Package management | uv |

## Evaluation Metrics

The evaluation dashboard measures two things:

**Retrieval quality** (does the right content get retrieved?)
- MRR (Mean Reciprocal Rank)
- nDCG (Normalized Discounted Cumulative Gain)
- Keyword coverage %

**Answer quality** (LLM-as-a-judge, scored 1–5)
- Accuracy
- Completeness
- Relevance
