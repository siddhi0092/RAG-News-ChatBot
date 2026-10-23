# News Chatbot Project Overview

## Purpose
This project is a Real-Time Retrieval-Augmented Generation (RAG) news chatbot. It fetches real-time news from the News API, indexes the data locally using FAISS embeddings, and answers user questions by combining the retrieved news context with a local LLM through Ollama.

## File Structure

- `README.md`
  - High-level documentation for the project, including features, architecture, setup steps, and usage instructions.

- `requirements.txt`
  - Python dependency list required to run the app, including FastAPI, Uvicorn, LangChain packages, FAISS, and Ollama integration.

- `.env`
  - Environment configuration file storing secrets and runtime settings such as `NEWS_API_KEY`, `EMBEDDING_MODEL`, `CHAT_MODEL`, and `OLLAMA_HOST`.
  - This file should remain local and is not normally committed to version control.

- `app/main.py`
  - The FastAPI web server entrypoint.
  - Defines API endpoints:
    - `GET /api/status` to check service health.
    - `POST /api/fetch-news` to fetch and index news articles.
    - `POST /api/chat` to query the chatbot against indexed news context.
  - Serves static frontend files from `app/static` and routes the root URL to `index.html`.

- `app/rag.py`
  - Core RAG operations and application state.
  - Connects to News API to fetch articles.
  - Processes articles into `langchain_core.document.Document` objects.
  - Uses `RecursiveCharacterTextSplitter` to chunk article text.
  - Creates embeddings with `OllamaEmbeddings` and builds a FAISS vector store.
  - Builds a retrieval pipeline with `OllamaLLM` and LangChain chains to answer user queries.
  - Maintains a lightweight `SessionState` object to cache the current query, indexed articles, and active vector store.

- `app/static/index.html`
  - Frontend user interface for the chatbot.
  - Includes search and ingest controls, status badges, article list display, and chat UI.

- `app/static/app.js`
  - Client-side JavaScript for the frontend.
  - Handles service checks, article fetch requests, chat submissions, and dynamic rendering of messages and article cards.
  - Provides basic markdown formatting and HTML escaping for safety.

- `app/static/style.css`
  - Styling for the UI, including dark theme, layout, card design, status indicators, and responsive input controls.

- `notes.txt`
  - Developer notes or temporary project comments.

## Technical Explanation

### Backend
1. `app/main.py` launches a FastAPI application.
2. It uses `dotenv` to load environment variables from `.env`.
3. The `/api/status` endpoint verifies that:
   - Ollama is running and reachable at `OLLAMA_HOST`.
   - News API key is valid and the NewsAPI endpoint responds.
4. The `/api/fetch-news` endpoint:
   - Validates the query.
   - Calls `fetch_realtime_news()` in `app/rag.py`.
   - Processes and indexes returned articles into a local FAISS store.
   - Returns a list of indexed metadata for frontend display.
5. The `/api/chat` endpoint:
   - Validates the user message.
   - Ensures a vector store already exists.
   - Uses a retrieval chain to answer from the indexed news context.

### Retrieval Pipeline (`app/rag.py`)
1. `fetch_realtime_news(query, page_size)`
   - Sends the query to NewsAPI `everything` endpoint.
   - Filters out removed articles and items missing content/description.
2. `process_and_index_news(articles, query)`
   - Builds full-text documents containing title, source, date, description, and content.
   - Splits each article into overlapping chunks using `RecursiveCharacterTextSplitter`.
   - Generates embeddings with Ollama's embedding model.
   - Stores embeddings in an in-memory FAISS vector database.
   - Updates global `state` so subsequent chat requests can retrieve context.
3. `chat_with_context(user_query)`
   - Creates a `ChatPromptTemplate` with a strict system prompt forcing contextual answers.
   - Retrieves the top relevant chunks from FAISS.
   - Uses `create_stuff_documents_chain` and `create_retrieval_chain` to combine retrieved documents with the LLM.
   - Returns a response and the source documents used for citations.

### Frontend
- The browser UI calls backend endpoints to:
  - check service health,
  - fetch news content,
  - start a chat conversation.
- It renders fetched article cards and chat messages.
- It provides status indicators for Ollama and NewsAPI connectivity.
- It enables the chat input only after news has been indexed.

## Running the App
1. Activate the virtual environment.
2. Install dependencies from `requirements.txt`.
3. Ensure Ollama is running and the required models are available.
4. Run the API server:
   ```bash
   python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
   ```
5. Open `http://127.0.0.1:8000/` in a browser.

## Notes
- The project relies on local Ollama services for both embeddings and chat generation.
- FAISS is used as an in-memory vector store, so the index resets when the server restarts.
- The project is designed for quick prototyping with real-time news and local LLM inference.
