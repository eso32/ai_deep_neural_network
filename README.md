# 🏛️ RAG — Polish Constitution Q&A System

A question-answering system built on the text of the **Constitution of the Republic of Poland**, using the RAG (Retrieval-Augmented Generation) technique.

---

## 📌 Overview

The user asks a question in Polish, and the system:
1. 🔍 Searches for relevant fragments of the Constitution in a vector database
2. 🧠 Generates an answer **strictly** based on the retrieved fragments
3. 📎 Cites the **source** (article number / chapter)

**Example:**
```
Question: Kto może być prezydentem Polski?

Answer: Według Art. 127 Konstytucji RP, na Prezydenta
Rzeczypospolitej może być wybrany obywatel polski, który
najpóźniej w dniu wyborów kończy 35 lat i korzysta z pełni
praw wyborczych do Sejmu.
```

---

## 🏗️ Architecture

### 📥 Indexing Pipeline

1. Download `konstytucja.pdf` from sejm.gov.pl
2. Extract raw text with `pypdf`
3. Split into chunks per article using regex `Art\. \d+\.`
4. Generate embeddings with `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`
5. Persist to ChromaDB at `./chroma_db`

### 💬 Query Pipeline

1. Accept a question from the user
2. Retrieve top-k nearest chunks via `similarity_search`
3. Build an augmented prompt with retrieved context
4. Generate an answer using `gpt-4o-mini` via a LangGraph ReAct agent

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| 🔗 Orchestration | [LangChain](https://www.langchain.com/) + [LangGraph](https://www.langchain.com/langgraph) |
| 🗄️ Vector Store | [ChromaDB](https://www.trychroma.com/) |
| 🔢 Embeddings | [sentence-transformers](https://www.sbert.net/) (`paraphrase-multilingual-MiniLM-L12-v2`) |
| 🤖 LLM | [OpenAI API](https://openai.com/api/) (`gpt-4o-mini`) |
| 📄 PDF Parsing | [pypdf](https://pypdf.readthedocs.io/) |

---

## 🚀 Running the Notebook

### ☁️ Google Colab / Kaggle (recommended)

1. Upload `sggw-zaliczenie-deep-neural-net.ipynb` to Colab
2. Run all cells top-to-bottom — the first cells install dependencies via `pip install -qq`
3. Set your OpenAI API key in the `API_KEY` variable (cell 13)

### 💻 Locally

```bash
pip install langchain langchain-community langchain-openai langchain-huggingface \
            chromadb sentence-transformers pypdf tiktoken langgraph

jupyter notebook sggw-zaliczenie-deep-neural-net.ipynb
```

---

## 🧩 System Prompt

```
Jesteś ekspertem od Konstytucji RP.
Odpowiadaj WYŁĄCZNIE na podstawie podanego kontekstu.
Zawsze podawaj numer artykułu.
Jeśli nie znajdziesz odpowiedzi — powiedz "Nie wiem".
```

---

## 🧪 Sample Test Questions

- 🗳️ *"Jakie prawa ma obywatel polski?"*
- 👔 *"Kto powołuje premiera?"*
- 🏛️ *"Ile trwa kadencja Sejmu?"*
- ⚠️ *"Kiedy można wprowadzić stan wyjątkowy?"*

---

## 📸 Required Screenshots

| # | Content |
|---|---------|
| 1️⃣ | **Indexing pipeline** — text loading, chunking, embeddings, saving to vector store |
| 2️⃣ | **Query pipeline** — retrieval, prompt augmentation, answer generation |
| 3️⃣ | **3+ example Q&As** with answers citing article numbers |
| 4️⃣ | **Off-topic question** — system responds "Nie wiem" instead of hallucinating |

---

## ⚠️ Known Issues

- `vectorstore.persist()` is deprecated in newer ChromaDB versions — persistence is automatic when `persist_directory` is set
- The PDF path defaults to `/kaggle/working/` — change to `./konstytucja.pdf` for Colab or local runs
- The OpenAI API key is hardcoded in cell 13 — replace with `os.getenv("OPENAI_API_KEY")` or a Colab secret
- `create_agent` does not exist in `langchain.agents` — use `from langgraph.prebuilt import create_react_agent` instead
