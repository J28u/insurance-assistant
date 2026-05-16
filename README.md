# Insurance Assistant

**Simplon / Wild Code School — AI Project Manager Path | Capstone Project** (2024–2025)

> Note: this is a Proof of Concept (POC) intended for local deployment only.

A local-first fullstack chatbot that lets users ask questions about their personal insurance contracts, without exposing sensitive data to third-party LLM services.

![Insurance Assistant demo](./screenshots/macaron_app.gif)

## Context

Capstone project developed as part of the AI Project Management curriculum (Simplon / Wild Code School), with a focus on functional architecture, data privacy, and LLM integration (RAG).

## Tech Stack

**Frontend**
- React 19 (Vite)
- Firebase Authentication

**Backend**
- Node.js / Express
- MongoDB (Mongoose)
- Firebase Admin SDK

**AI / ML pipeline**
- Python: LangChain, FAISS, HuggingFace Embeddings
- Ollama (local LLM inference)
- Kedro (pipeline orchestration)

## Key Features

- 📄 **PDF ingestion** — upload insurance contract PDFs; they are chunked, embedded, and stored in a per-user FAISS vector store, then deleted from disk
- 💬 **RAG chatbot** — ask questions about your contracts; answers are grounded in your documents via retrieval-augmented generation
- 🔒 **Privacy-first** — all LLM inference runs locally via Ollama; no data leaves your machine
- 🗂 **Conversation history** — past conversations are stored in MongoDB
- 👤 **Multi-user** — each user has their own isolated vector store and conversation history

## Security

- Firebase ID token verification on all protected routes
- Per-user access control: users can only access and modify their own resources
- Rate limiting per user
- PDF upload hardening: file size and count limits, strict MIME type filtering, secure filename generation, automatic deletion after ingestion
- Custom error classes + global error handler middleware (no sensitive info exposed to frontend)
- Full LLM logic (retriever, inference, streaming) runs server-side only
- Basic input sanitisation (regex) and controlled prompt encapsulation (system / context / user separation)

## Project Structure

```
.
├── backend/                     # Node.js API — Express + Mongoose
│   └── src/
│       ├── errors/              # Custom error classes
│       ├── middlewares/         # Auth, validation, error handler
│       ├── models/              # Mongoose schemas (User, Conversation, Message)
│       ├── routes/              # REST API routes (chat, conversations, upload, users)
│       ├── services/            # Business logic (conversations, retriever)
│       ├── firebaseAdmin.js     # Firebase Admin SDK initialisation
│       └── index.js             # Entry point: DB connection + route setup
│
├── frontend/                    # React app (Vite)
│   └── src/
│       ├── components/          # React components (Chat, Conversation, Library, Auth)
│       ├── App.jsx
│       ├── firebase.js          # Firebase client SDK initialisation
│       └── main.jsx
│
├── kedro_pipelines/             # ML/data pipelines (Kedro)
│   ├── conf/                    # Config: catalogs, parameters, prompt templates
│   └── src/rag/
│       ├── custom_datasets/     # Custom FAISS dataset
│       └── pipelines/
│           ├── embedding/       # Document vectorisation pipeline
│           └── rag_classic/     # Classic RAG pipeline (retrieval + prompt)
│
├── requirements.txt
└── README.md
```

## Main API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/conversations/` | Create a new conversation |
| `GET` | `/api/conversations/user/:userId` | Get all conversations for a user |
| `GET` | `/api/conversations/onlyone/:conversationId` | Get messages for a conversation |
| `POST` | `/api/upload/` | Upload PDFs, chunk, embed, and store in vector store |

## Installation

### Prerequisites

- Node.js v18+
- MongoDB (local or MongoDB Atlas)
- Python 3.10+
- [Ollama](https://ollama.com/download)

### 1. Clone the repository

```bash
git clone https://github.com/J28u/insurance-assistant.git
cd insurance-assistant
```

### 2. Configure environment variables

```bash
cp backend/src/.env.example backend/src/.env
# Fill in your Firebase credentials, MongoDB URL, and model name
```

### 3. Set up MongoDB

Create a database named `chatbotdb` and set the connection string in `.env`:

```env
MONGODB_URL=mongodb+srv://<user>:<password>@cluster0.mongodb.net/chatbotdb?retryWrites=true&w=majority
```

### 4. Install Python dependencies

```bash
pip install -r requirements.txt
```

### 5. Set up Ollama

```bash
ollama pull gemma3:4b-it-q4_K_M
ollama serve
```

Set the model name in `.env`:

```env
MODEL_CHAT=gemma3:4b-it-q4_K_M
```

### 6. Start the backend

```bash
cd backend && npm install
cd src && node index.js
```

### 7. Start the frontend

```bash
cd frontend && npm install
cd src && npm run dev
```

### 8. Run Kedro pipelines

To change the embedding model, edit `kedro_pipelines/conf/base/parameters.yml`:

```yaml
embedding_model_name: "OrdalieTech/Solon-embeddings-large-0.1"
```

To run a pipeline:

```bash
kedro run --pipeline embedding   # vectorise documents
kedro run --pipeline classic_rag # run RAG pipeline
```

See [Kedro docs](https://docs.kedro.org) for more options.
