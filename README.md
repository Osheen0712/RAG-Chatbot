# 📚 RAG Chatbot

A Retrieval-Augmented Generation (RAG) chatbot that allows users to
upload PDF documents and ask questions about their content using
natural language.

The application combines **Anthropic Claude** for generation,
**Voyage AI** for embeddings, and **ChromaDB** for vector storage.
It provides a **FastAPI backend** and a **Streamlit frontend**.


---

## ✨ Features

- 📄 **PDF ingestion** — Upload PDF documents and automatically extract
  and chunk their content.
- 🔎 **Semantic retrieval** — Retrieve the most relevant document
  chunks using vector similarity search.
- 💬 **Context-aware Q&A** — Generate answers using retrieved document
  context.
- 🧠 **Conversation memory** — Maintain session-based conversation
  history for multi-turn interactions.
- 🗂️ **Multi-document support** — Store and manage multiple documents
  independently using unique document IDs.
- 🚀 **REST API** — FastAPI backend with automatically generated
  Swagger documentation.
- 🖥️ **Interactive UI** — Streamlit-based chat interface for document
  upload and question answering.

---

## 🏗️ Architecture

```
┌─────────────────────┐         HTTP          ┌──────────────────────────┐
│   Streamlit (app.py)│  ──────────────────►  │  FastAPI (main.py)       │
│   localhost:8501    │                        │  localhost:8000          │
└─────────────────────┘                        └──────────┬───────────────┘
                                                          │
                    ┌─────────────────────────────────────┼──────────────────────┐
                    │                                     │                      │
             ┌──────▼──────┐                    ┌─────────▼──────┐   ┌──────────▼──────┐
             │ pdf_loader  │                    │  vector_store  │   │    chatbot.py    │
             │ (pypdf +    │                    │  (ChromaDB +   │   │  (LangChain +    │
             │  splitter)  │                    │   Voyage AI)   │   │   Claude LLM)   │
             └─────────────┘                    └────────────────┘   └─────────────────┘
```

**Request flow for a chat query:**

1. User sends a question via the Streamlit UI.
2. FastAPI `/chat` endpoint receives it.
3. `vector_store.py` retrieves the top-K most relevant chunks from ChromaDB using Voyage AI embeddings.
4. `chatbot.py` builds a prompt with the retrieved context + session history and calls Claude.
5. The answer is returned and displayed in the chat window.

---

## 📁 Project Structure

```
RAG-Chatbot/
├── app.py              # Streamlit frontend
├── main.py             # FastAPI application and API routes
├── chatbot.py          # LLM initialization and chat logic
├── vector_store.py     # ChromaDB setup and retrieval operations
├── pdf_loader.py       # PDF parsing and text chunking
├── history.py          # Session-based conversation history
├── config.py           # Settings loaded from .env (Pydantic)
├── schemas.py          # Pydantic request/response models
├── utils.py            # Logging, session ID generation, validators
├── run.py              # Script for running the application
├── requirements.txt    # Python dependencies
└── .gitignore
```

---

## 🔧 Prerequisites

- Python **3.10+**
- An **Anthropic API key** — [get one here](https://console.anthropic.com/)
- A **Voyage AI API key** — [get one here](https://www.voyageai.com/)

---

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/Osheen0712/RAG-Chatbot.git
cd RAG-Chatbot
```

### 2. Create and activate a virtual environment

```bash
python -m venv venv
# Linux / macOS
source venv/bin/activate
# Windows
venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file in the project root:

```env
ANTHROPIC_API_KEY=your_anthropic_api_key_here
VOYAGE_API_KEY=your_voyage_api_key_here
```

All other settings are optional and have sensible defaults (see [Configuration](#-configuration)).

### 5. Run the application

```bash
python run.py
```

Alternatively, start them separately:

```bash
# Terminal 1 — Backend
uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# Terminal 2 — Frontend
streamlit run app.py
```

### 6. Open in your browser

| Service | URL |
|---|---|
| Streamlit UI | http://localhost:8501 |
| FastAPI docs (Swagger) | http://localhost:8000/docs |
| FastAPI docs (Redoc) | http://localhost:8000/redoc |

---

## 💡 Usage

1. Open the Streamlit UI at `http://localhost:8501`.
2. In the **sidebar**, click **Browse files** and select a PDF.
3. Click **Process Document** and wait for the confirmation message.
4. Type your question in the chat box and press **Enter**.
5. To start fresh with a new document, click **🗑️ Clear Document & Start New**.

---

## 🌐 API Reference

The FastAPI backend exposes the following endpoints:

### System

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Health check + vector store stats |
| `GET` | `/stats` | ChromaDB collection statistics |

### Documents

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/ingest` | Upload a PDF and embed its contents |
| `DELETE` | `/document/{doc_id}` | Remove a document's embeddings |

### Chat

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/chat` | Send a question and receive a grounded answer |

### Session

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/session/new` | Create a new session ID |
| `GET` | `/session/list` | List all active sessions |
| `POST` | `/session/clear` | Clear session history |

#### Example `/chat` request

```json
POST /chat
{
  "session_id": "abc-123",
  "question": "What are the main conclusions of this document?",
  "doc_id": "some-doc-id"
}
```

---

## ⚙️ Configuration

All settings live in `config.py` and are loaded from your `.env` file.

| Variable | Default | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | *(required)* | Anthropic API key |
| `VOYAGE_API_KEY` | *(required)* | Voyage AI API key |
| `LLM_MODEL` | `claude-sonnet-4-6` | Anthropic model to use |
| `LLM_TEMPERATURE` | `0.2` | LLM sampling temperature |
| `MAX_TOKENS` | `1024` | Max tokens in LLM response |
| `EMBEDDING_MODEL` | `voyage-4-large` | Voyage AI embedding model |
| `CHROMA_PERSIST_DIR` | `./chroma_db` | ChromaDB storage directory |
| `CHROMA_COLLECTION_NAME` | `rag_documents` | ChromaDB collection name |
| `CHUNK_SIZE` | `1000` | Characters per text chunk |
| `CHUNK_OVERLAP` | `200` | Overlap between chunks |
| `TOP_K_RETRIEVAL` | `5` | Number of chunks retrieved per query |
| `HISTORY_WINDOW_SIZE` | `10` | Max turns kept in conversation history |
| `API_HOST` | `0.0.0.0` | FastAPI host |
| `API_PORT` | `8000` | FastAPI port |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| LLM | [Anthropic Claude](https://www.anthropic.com/) (`claude-sonnet-4-6`) |
| Embeddings | [Voyage AI](https://www.voyageai.com/) (`voyage-4-large`) |
| Vector Database | [ChromaDB](https://www.trychroma.com/) |
| LLM Framework | [LangChain](https://www.langchain.com/) |
| Backend | [FastAPI](https://fastapi.tiangolo.com/) + [Uvicorn](https://www.uvicorn.org/) |
| Frontend | [Streamlit](https://streamlit.io/) |
| PDF Parsing | [pypdf](https://pypdf.readthedocs.io/) |

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to open an issue or submit a pull request.

---

## 📄 License

This project is open source. See the repository for details.
