# 🩺 CareBot AI — Medical Assistant Chatbot

An AI-powered **Retrieval-Augmented Generation (RAG)** chatbot that helps users understand medical documents. Upload your medical PDFs, and CareBot AI will answer your health-related questions using only the information from your documents.

> **⚠️ Disclaimer:** CareBot AI is designed for **informational purposes only**. It does **not** provide medical advice, diagnoses, or treatment recommendations.

---

## ✨ Features

- 📄 **PDF Upload** — Upload multiple medical documents (PDFs) via a sidebar widget
- 🧠 **RAG Pipeline** — Documents are chunked, embedded, and stored in a Pinecone vector database for semantic retrieval
- 💬 **Conversational Chat** — Ask questions in natural language and get context-aware answers powered by Llama 3.3 70B
- 📥 **Chat History Download** — Export your entire conversation as a `.txt` file
- 🔍 **Source Attribution** — Responses are grounded in retrieved document chunks (no hallucination)
- ⚡ **Fast Inference** — Uses Groq's ultra-fast LLM inference for near-instant responses

---

## 🏗️ Architecture

```
┌─────────────────────┐        HTTP         ┌─────────────────────────┐
│                     │ ◄──────────────────► │                         │
│   Streamlit Client  │    REST API Calls    │   FastAPI Server        │
│   (Frontend)        │                      │   (Backend)             │
│                     │                      │                         │
│  • Chat UI          │                      │  • /upload_pdfs/        │
│  • PDF Uploader     │                      │  • /ask/                │
│  • History Download │                      │                         │
└─────────────────────┘                      └────────┬────────────────┘
                                                      │
                                        ┌─────────────┼─────────────┐
                                        ▼             ▼             ▼
                                  ┌──────────┐ ┌───────────┐ ┌───────────┐
                                  │ Pinecone │ │  Google    │ │   Groq    │
                                  │ Vector   │ │  Gemini    │ │  LLM API  │
                                  │ Store    │ │ Embeddings │ │ (Llama 3) │
                                  └──────────┘ └───────────┘ └───────────┘
```

---

## 📁 Project Structure

```
Medical/
├── client/                         # Streamlit Frontend
│   ├── app.py                      # Main Streamlit app entry point
│   ├── config.py                   # API URL configuration
│   ├── requirements.txt            # Client dependencies
│   ├── components/
│   │   ├── chatUI.py               # Chat interface component
│   │   ├── upload.py               # PDF upload sidebar component
│   │   └── history_download.py     # Chat history export component
│   └── utils/
│       └── api.py                  # HTTP client for backend API calls
│
├── server/                         # FastAPI Backend
│   ├── main.py                     # FastAPI app initialization & middleware
│   ├── logger.py                   # Custom logging setup
│   ├── requirements.txt            # Server dependencies
│   ├── .env                        # Environment variables (API keys)
│   ├── middlewares/
│   │   └── exception_handlers.py   # Global exception handling middleware
│   ├── modules/
│   │   ├── load_vectorstore.py     # PDF processing, embedding & Pinecone upsert
│   │   ├── llm.py                  # LLM chain setup (Groq + RetrievalQA)
│   │   ├── query_handlers.py       # Query execution & response formatting
│   │   └── pdf_handlers.py         # File saving utilities
│   └── routes/
│       ├── upload_pdfs.py          # POST /upload_pdfs/ endpoint
│       └── ask_question.py         # POST /ask/ endpoint
│
├── .gitignore
└── README.md
```

---

## 🛠️ Tech Stack

| Layer         | Technology                                                        |
|---------------|-------------------------------------------------------------------|
| **Frontend**  | [Streamlit](https://streamlit.io/)                                |
| **Backend**   | [FastAPI](https://fastapi.tiangolo.com/) + Uvicorn                |
| **LLM**       | [Llama 3.3 70B](https://groq.com/) via Groq API                  |
| **Embeddings**| [Google Gemini Embedding](https://ai.google.dev/) (768 dimensions)|
| **Vector DB** | [Pinecone](https://www.pinecone.io/) (Serverless)                |
| **Framework** | [LangChain](https://www.langchain.com/) (RetrievalQA chain)      |
| **PDF Parsing**| PyPDF via LangChain Community                                    |

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- API keys for **Google AI**, **Groq**, and **Pinecone**
- A Pinecone index (768 dimensions, `dotproduct` metric)

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/CareBot-AI.git
cd CareBot-AI
```

### 2. Create a Virtual Environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
# Server dependencies
pip install -r server/requirements.txt

# Client dependencies
pip install -r client/requirements.txt
```

### 4. Configure Environment Variables

Create a `server/.env` file:

```env
GOOGLE_API_KEY=your_google_api_key
GROQ_API_KEY=your_groq_api_key
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX_NAME=medicalindex
PINECONE_INDEX_HOST=https://your-index-host.pinecone.io
```

### 5. Start the Backend Server

```bash
cd server
uvicorn main:app --reload
```

The API will be available at `http://127.0.0.1:8000`. You can view the interactive docs at `http://127.0.0.1:8000/docs`.

### 6. Start the Frontend Client

In a **new terminal**:

```bash
cd client
python -m streamlit run app.py
```

The Streamlit app will open at `http://localhost:8501`.

---

## 📡 API Endpoints

| Method | Endpoint         | Description                                  | Request Body               |
|--------|------------------|----------------------------------------------|----------------------------|
| `POST` | `/upload_pdfs/`  | Upload PDF files to be processed & indexed   | `multipart/form-data` — `files` |
| `POST` | `/ask/`          | Ask a question against the uploaded documents | `form-data` — `question`   |

### Example — Ask a Question

```bash
curl -X POST http://127.0.0.1:8000/ask/ \
  -F "question=What is diabetes?"
```

**Response:**
```json
{
  "response": "Based on the provided documents, diabetes is a chronic condition...",
  "sources": ["uploaded_doc.pdf"]
}
```

---

## 🔄 How It Works

1. **Upload** — User uploads medical PDFs through the Streamlit sidebar
2. **Process** — Backend loads the PDFs, splits them into 500-character chunks with 50-character overlap
3. **Embed** — Each chunk is embedded using Google's Gemini Embedding model (768 dimensions)
4. **Store** — Embeddings are upserted into a Pinecone serverless vector index
5. **Query** — When a user asks a question, it's embedded and the top 3 most relevant chunks are retrieved from Pinecone
6. **Answer** — The retrieved chunks are passed as context to Llama 3.3 70B (via Groq) which generates a grounded response

---

## 🌐 Deployment

The backend is deployed on **[Render](https://render.com)** at:

```
https://carebot-ai.onrender.com
```

To point the client to the deployed backend, update `client/config.py`:

```python
API_URL = "https://carebot-ai.onrender.com"
```

For local development, switch back to:

```python
API_URL = "http://127.0.0.1:8000"
```

---

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

---

## 🙏 Acknowledgments

- [LangChain](https://www.langchain.com/) for the RAG framework
- [Groq](https://groq.com/) for blazing-fast LLM inference
- [Pinecone](https://www.pinecone.io/) for managed vector search
- [Google AI](https://ai.google.dev/) for embedding models
- [Streamlit](https://streamlit.io/) for rapid UI development
