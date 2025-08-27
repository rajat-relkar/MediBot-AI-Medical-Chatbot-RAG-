# MediBot — AI Medical Chatbot (RAG)

A smart medical chatbot built fully on open-source tooling with a simple Streamlit UI. It uses HuggingFace for embeddings, FAISS (CPU) for vector storage, and either Mistral (HuggingFace Inference) or a fast, free Groq-hosted Llama model for the conversational LLM. The retrieval pipeline follows a classic RAG pattern: load PDFs → chunk → embed → store → retrieve → answer.

> ❗️**Disclaimer:** This project is for educational use only and does **not** provide medical advice or diagnosis. Always consult qualified healthcare professionals.

---

## ✨ Features

- **RAG pipeline** over your own PDFs (medical notes, guidelines, etc.)
- **FAISS** local vector store on CPU for fast retrieval
- **Embeddings:** `sentence-transformers/all-MiniLM-L6-v2` (HuggingFace)
- **LLM options:**
  - **Mistral-7B-Instruct** via HuggingFace Inference (requires HF token)
  - **meta-llama/llama-4-maverick-17b-128e-instruct** via Groq (free, fast)
- **Streamlit chat UI** with session-based history and cached vector store
- **Grounded answers** with a strict custom prompt that avoids fabrication

---

## 🗺️ Architecture

**Phase 1 — Setup Memory for LLM (Vector DB)**  
1) Load raw PDF(s) → 2) Split into chunks → 3) Create embeddings → 4) Store in FAISS.

**Phase 2 — Connect Memory with LLM**  
Load LLM (Mistral or Llama), connect retriever over FAISS, build `RetrievalQA` chain.

**Phase 3 — Chatbot UI (Streamlit)**  
Cache FAISS store, run query through `RetrievalQA`, display answer + sources.

---

## 📁 Project Layout
```
.
├─ data/The_GALE_ENCYCLOPEDIA_of_MEDICINE_SECOND.pdf
├─ vectorstore/
│ └─ db_faiss/
|      └─ index.faiss
|      └─ index.pkl
|      
├─ create_memory_for_llm.py # Phase 1: build FAISS from PDFs
├─ connect_memory_with_llm.py # Phase 2: test RAG via console
├─ medibot.py # Phase 3: Streamlit chat UI
└─ README.md
```
---

## 🧰 Tech Stack

- **LangChain** (RAG orchestration)  
- **HuggingFace** (embeddings + optional Mistral endpoint)  
- **FAISS (CPU)** (vector database)  
- **Streamlit** (chat UI)  
- **Python** (core language)

---

## 🚀 Quickstart

### 1) Create & Activate Virtual Env (Pipenv)

```bash
pipenv install langchain langchain_community langchain_huggingface faiss-cpu pypdf
pipenv install huggingface_hub
pipenv install streamlit
pipenv shell
```

### 2) Add your PDFs

Place one or more `.pdf` files in `data/`. The loader scans `data/*.pdf`.

### 3) Configure API Keys

Create a `.env` or export environment variables before running:

```bash
# For HuggingFace Inference (Mistral)
export HF_TOKEN=your_hf_token_here

# For Groq (used by Streamlit app with Llama-4 Maverick)
export GROQ_API_KEY=your_groq_api_key_here
```

### 4) Build the Vector Store (Phase 1)

```bash
python create_memory_for_llm.py
```

What it does:
- Loads PDFs from `data/` with `DirectoryLoader` + `PyPDFLoader`  
- Splits into chunks of **500** tokens with **50** overlap  
- Embeds using **all-MiniLM-L6-v2**  
- Saves FAISS index at `vectorstore/db_faiss`

### 5a) Console Test (Mistral via HF Inference)

```bash
python connect_memory_with_llm.py
```

### 5b) Streamlit Chat UI (Groq Llama-4 Maverick)

```bash
streamlit run medibot.py
```

---

## 🧪 Prompting & Guardrails

The system prompt instructs the model to **only** answer from provided context and to say “I don’t know” if the answer isn’t found—this helps avoid unsafe or made-up medical claims.

---

## ⚙️ Configuration Notes

- **Retriever top-k:** 3 neighbors for focused context  
- **Chunking:** 500/50 (size/overlap) tuned for small medical PDFs  
- **Model choices:**  
  - HF Inference: `mistralai/Mistral-7B-Instruct-v0.3`  
  - Groq: `meta-llama/llama-4-maverick-17b-128e-instruct`

---

## 🔐 Security & Privacy

- **Local vectors:** FAISS index stays on disk (`vectorstore/db_faiss`).  
- **API keys:** Use environment variables; do not commit `.env` to source control.

---

## 🛠️ Troubleshooting

- **“Failed to load the vector store”** → Run `create_memory_for_llm.py` first  
- **Mistral console errors** → Ensure `HF_TOKEN` is set  
- **Groq chat errors** → Ensure `GROQ_API_KEY` is set

---

