# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

University assignment (Deep Neural Networks course, SGGW) implementing a **RAG (Retrieval-Augmented Generation)** system for querying the Polish Constitution. The single deliverable is `sggw-zaliczenie-deep-neural-net.ipynb`.

The notebook is designed to run on **Google Colab or Kaggle** (note the `/kaggle/working/` path in the PDF reader cell). The local `.venv` is minimal — do not rely on it for running the notebook.

## Running the Notebook

Open and run on Google Colab:
1. Upload `sggw-zaliczenie-deep-neural-net.ipynb` to Colab
2. Run all cells top-to-bottom — the first cells install dependencies via `pip install -qq`
3. Set your OpenAI API key in the `API_KEY` variable (cell 13)

To run locally (requires manual dependency install):
```bash
pip install langchain langchain-community langchain-openai langchain-huggingface chromadb sentence-transformers pypdf tiktoken langgraph
jupyter notebook sggw-zaliczenie-deep-neural-net.ipynb
```

## Architecture

The notebook implements a two-stage pipeline:

**Indexing pipeline (cells 0–6):**
- Downloads `konstytucja.pdf` from sejm.gov.pl
- Extracts raw text with `pypdf`
- Splits into chunks per article using regex `re.split(r'(Art\. \d+\.)', raw_text)`
- Embeds with `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` (multilingual, no API needed)
- Persists to ChromaDB at `./chroma_db`

**Query pipeline (cells 9–17):**
- `retrieve_constitution_content` tool — LangChain `@tool` that runs `vectorstore.similarity_search(query, k=1)`, returns serialized text + raw docs
- LangGraph `create_react_agent` (from `langgraph.prebuilt`) creates the agent with `gpt-4o-mini` as LLM
- Agent is prompted to answer only from retrieved context, cite article numbers, and say "Nie wiem" if context is insufficient

## Known Issues / Watch Out For

- **`create_agent` does not exist** in `langchain.agents`. The correct import is `from langgraph.prebuilt import create_react_agent`. The parameter for the system prompt is `prompt` (a string or `SystemMessage`), not `system_prompt`.
- `vectorstore.persist()` is deprecated in newer ChromaDB versions — persistence is automatic when `persist_directory` is set.
- The PDF reader uses a hardcoded `/kaggle/working/` path — must be updated to `./konstytucja.pdf` for local or Colab runs.
- The OpenAI API key is hardcoded in cell 13 — replace with `os.getenv("OPENAI_API_KEY")` or a Colab secret.

## Assignment Requirements Checklist

- [ ] Screenshot 1: indexing pipeline (loading, chunking, embeddings, vector store save)
- [ ] Screenshot 2: query pipeline (search, prompt augmentation, answer generation)
- [ ] Screenshot 3: 3+ example Q&A with article citations
- [ ] Screenshot 4: off-topic question → system says "Nie wiem" (not hallucinating)
