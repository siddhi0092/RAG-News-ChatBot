# Real-Time RAG News Chatbot

An AI-powered Retrieval-Augmented Generation (RAG) chatbot that retrieves real-time news articles through the News API and enables users to ask questions in natural language. Instead of searching through multiple articles, users get structured, context-aware answers, and cited sources.

The system combines real-time news retrieval, chunking, local vector embeddings, FAISS indexing, and LLMs to provide precise and hallucination-free responses.

---

## Features

- **Real-Time News Retrieval**: Fetches the latest articles via News API on any topic.
- **Local Ingestion & Chunking**: Splits articles into manageable contexts using LangChain text splitters.
- **Local Embeddings**: Generates 1024-dimensional semantic embeddings using Ollama's `mxbai-embed-large`.
- **Local Vector Database**: Stores vectors in a fast, in-memory FAISS database.
- **Local LLM Reasoning**: Processes user prompts and context via Ollama's `llama3.2:3b`.
- **Interactive UI Dashboard**:
  - Sleek dark theme with glassmorphism styling and glowing accents.
  - Active search configuration controls.
  - Live sidebar displaying all indexed articles (source, publication date, description, links).
  - High-fidelity chat bubble interfaces with streaming indicators and source citation lists.
  - Live system status indicators for backend services.

---

## Technical Architecture

```text
               User Search Query
                       │
                       ▼
             [ News API Everything ]  ──► Fetches latest articles
                       │
                       ▼
            [ LangChain Splitter ]   ──► Chunking (800 char size, 150 overlap)
                       │
                       ▼
         [ Ollama mxbai-embed-large ] ──► Vector generation
                       │
                       ▼
           [ FAISS Vector Store ]     ──► In-memory semantic index
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
     User Message          FAISS Retrieve (Top 4 chunks)
             │                   │
             └─────────┬─────────┘
                       ▼
            [ Ollama llama3.2:3b ]    ──► Generate Answer with citations
                       │
                       ▼
                 Chat Response
```

---

## Project Structure

```text
News Chatbot/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI server & route handlers
│   ├── rag.py               # RAG operations (API search, vector store, QA chain)
│   └── static/              # Frontend web dashboard
│       ├── index.html       # UI page structural markup
│       ├── style.css        # Custom CSS styling (dark glassmorphic theme)
│       └── app.js           # Client-side status checking & chat logic
│
├── .env                     # App configurations & News API key (not tracked in Git)
├── requirements.txt         # Required Python packages
├── README.md                # Project documentation
└── notes.txt                # Developer workflows notes
```

---

## Installation & Setup

### 1. Prerequisites
- **Python 3.10+** (Python 3.11.9 is recommended)
- **Ollama** installed and running on your local machine.

### 2. Prepare Ollama Models
Ensure Ollama is running and download the embedding and chat models:
```bash
ollama pull mxbai-embed-large
ollama pull llama3.2:3b
```

### 3. Clone / Open Project Directory
Navigate to the root directory of the project:
```bash
cd "News Chatbot"
```

### 4. Create Virtual Environment
Create and activate a Python virtual environment:
```bash
# Windows
python -m venv venv
.\venv\Scripts\activate

# macOS / Linux
python3 -m venv venv
source venv/bin/activate
```

### 5. Install Dependencies
Install all required packages from `requirements.txt`:
```bash
pip install -r requirements.txt
```

### 6. Configuration (.env)
Create a `.env` file in the root directory (one is already prepared with your credentials):
```env
NEWS_API_KEY=YOUR_NEWS_API_KEY
EMBEDDING_MODEL=mxbai-embed-large
CHAT_MODEL=llama3.2:3b
OLLAMA_HOST=http://localhost:11434
```

---

## Running the Application

To start the FastAPI web server, run:
```bash
python app/main.py
```
Or use `uvicorn`:
```bash
uvicorn app.main:app --reload --port 8000
```

Open your browser and navigate to:
```text
http://127.0.0.1:8000/
```

---

## How to Use
1. **Search Context**: Input a keyword (e.g., "artificial intelligence", "stock market", "nasas space missions") in the **Configure & Ingest** box on the left, select article count, and click **Fetch & Index News**.
2. **Review Articles**: Look through the loaded cards in the **Indexed Articles** panel to review retrieved sources.
3. **Ask Questions**: Ask questions in the chatbot on the right. The assistant will answer using only details from the loaded articles and cite relevant sources at the bottom of its message bubble.