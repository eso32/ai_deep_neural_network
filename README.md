# 🏛️ RAG — Polish Constitution Q&A System

A question-answering system built on the text of the **Constitution of the Republic of Poland**, using the RAG (Retrieval-Augmented Generation) technique.

---

## 📌 Overview

The user asks a question in English, and the system:

1. 🔍 Searches for relevant fragments of the Constitution in a vector database
2. 🧠 Generates an answer **strictly** based on the retrieved fragments
3. 📎 Cites the **source** (article number / chapter)

**Example:**

```
Question: Who appoints the Prime Minister?

Answer: The Prime Minister is appointed by the President of the Republic
of Poland, as stated in Article 154.
```

---

## 🏗️ Architecture

### 📥 Indexing Pipeline

1. Download `constitution.pdf` from sejm.gov.pl
2. Extract raw text with `pypdf`
3. Split into chunks per article using regex `Art\. \d+\.`
4. Generate embeddings with **OpenAI `text-embedding-3-small`**
5. Persist to ChromaDB at `./chroma_db2`

### 💬 Query Pipeline

1. Accept a question from the user
2. Retrieve the nearest chunk via `similarity_search` (k=1)
3. Build an augmented prompt with retrieved context
4. Generate an answer using `gpt-4o-mini` via a LangChain ReAct agent

---

## 🛠️ Tech Stack

| Component        | Technology                                                       |
| ---------------- | ---------------------------------------------------------------- |
| 🔗 Orchestration | [LangChain](https://www.langchain.com/)                          |
| 🗄️ Vector Store  | [ChromaDB](https://www.trychroma.com/)                           |
| 🔢 Embeddings    | [OpenAI API](https://openai.com/api/) (`text-embedding-3-small`) |
| 🤖 LLM           | [OpenAI API](https://openai.com/api/) (`gpt-4o-mini`)            |
| 📄 PDF Parsing   | [pypdf](https://pypdf.readthedocs.io/)                           |

---

## 🚀 Running the Notebook

### ☁️ Google Colab / Kaggle (recommended)

1. Upload `rag-constitution.ipynb` to Colab
2. Run all cells top-to-bottom — the first cells install dependencies via `pip install -qq`
3. Set your OpenAI API key as the `OPENAI_API_KEY` environment variable

### 💻 Locally

```bash
pip install langchain langchain-community langchain-openai langchain-huggingface \
            chromadb sentence-transformers pypdf tiktoken python-dotenv

jupyter notebook rag-constitution.ipynb
```

Set `OPENAI_API_KEY` in a `.env` file — the notebook loads it via `python-dotenv`.

---

## 🧩 System Prompt

```
You are an expert on the Constitution of the Republic of Poland.
Answer EXCLUSIVELY based on the provided context.
Always provide the article number.
If you cannot find the answer — say "I did not find the answer in the Constitution."
```

---

## 🧪 Sample Test Questions

- 👔 _"Who appoints the Prime Minister?"_
- 🏛️ _"How long is the term of the Sejm?"_
- ⚠️ _"When can a state of emergency be introduced?"_

---

## 📸 Required Screenshots

| #   | Content                                                                                                            |
| --- | ------------------------------------------------------------------------------------------------------------------ |
| 1️⃣  | **Indexing pipeline** — text loading, chunking, embeddings, saving to vector store                                 |
| 2️⃣  | **Query pipeline** — retrieval, prompt augmentation, answer generation                                             |
| 3️⃣  | **3+ example Q&As** with answers citing article numbers                                                            |
| 4️⃣  | **Off-topic question** — system responds "I did not find the answer in the Constitution." instead of hallucinating |
