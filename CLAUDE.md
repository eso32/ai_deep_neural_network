# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A RAG (Retrieval-Augmented Generation) Q&A system over the Constitution of the Republic of Poland. Users ask questions in English; the system retrieves relevant constitutional articles from ChromaDB and generates grounded answers via a LangChain ReAct agent backed by `gpt-4o-mini`.

## Running the Notebook

```bash
# Activate the local venv
source .venv/bin/activate

# Launch Jupyter
jupyter notebook rag-constitution.ipynb
```

Dependencies are installed inside the notebook's first cell via `pip install -qq`. To install them into the venv without running the notebook:

```bash
pip install langchain langchain-openai langchain-community chromadb \
            sentence-transformers openai pypdf tiktoken python-dotenv
```

The `OPENAI_API_KEY` is loaded from `.env` via `python-dotenv`.

## Architecture

The notebook has two sequential pipelines:

**Indexing pipeline** (run once; output persisted to `./chroma_db2`):
1. Download `constitution.pdf` from sejm.gov.pl via `urllib`
2. Extract full text with `pypdf`
3. Split into per-article chunks using `re.split(r'(Art\. \d+\.)', raw_text)`
4. Embed with OpenAI `text-embedding-3-small` via `OpenAIEmbeddings`
5. Store in ChromaDB (`langchain_community.vectorstores.Chroma`)

**Query pipeline** (interactive, runs on each question):
1. A `@tool`-decorated function `retrieve_constitution_content` runs `vectorstore.similarity_search(query, k=1)`
2. The tool returns both a serialized string and the raw documents (using `response_format="content_and_artifact"`)
3. A LangChain ReAct agent (`create_agent`) wraps `gpt-4o-mini` with this tool and a strict system prompt that forbids hallucination outside retrieved context
4. `ask_lawyer(q)` invokes the agent and pretty-prints the result

## Key Constraints

- The system prompt instructs the LLM to answer **only** from retrieved context and always cite the article number. Do not relax this constraint.
- `k=1` in similarity search means only the single best-matching article is used as context per query.
- The ChromaDB persist directory is `./chroma_db2` (not `chroma_db`). Re-running the indexing cell appends to the existing collection; drop and recreate the directory to rebuild from scratch.
- `constitution.pdf` and `konstytucja.pdf` (Polish original) are excluded from git — download fresh or use the cached copy if present.
