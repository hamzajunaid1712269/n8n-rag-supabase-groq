# n8n RAG Agent with Supabase (pgvector) + Hugging Face Embeddings + Groq

## Demo
See `demo.md` for the recording link

This repository contains a two-workflow Retrieval-Augmented Generation (RAG) prototype built in **n8n**:
1) **Document ingestion (run once/on demand)**: PDF → text extraction → embeddings → store in Supabase (pgvector)
2) **Conversational retrieval (runs per user question)**: Chat trigger → AI Agent (Groq) → tool-based vector retrieval from Supabase

The demo uses the paper **“Attention Is All You Need”** as the ingested source. The agent is instructed to answer only using retrieved context and to respond with “I don’t know” if the answer is not present in the source.

---

## Architecture

### Workflow 1: Document Ingestion (Manual / one-time)
Export: `attention-embedding.json`

- Manual trigger (run when needed)
- Read PDF from disk (Docker volume mount)
- Extract text from PDF
- Insert into **Supabase Vector Store** (`documents` table)
- Create embeddings via **Hugging Face Inference**
  - Model: `sentence-transformers/distilbert-base-nli-mean-tokens`
  - Dimension: 768

### Workflow 2: Conversational Retrieval (Chat)
Export: `chatbot.json`

- Chat Trigger receives messages
- AI Agent uses **Groq Chat Model** (low temperature)
- Supabase Vector Store connected as a **tool**:
  - retrieves top matches using vector similarity via `match_documents(...)`
- Simple chat memory enabled for short conversational context

---

## Prerequisites
- Docker (for running n8n)
- Supabase project with pgvector enabled
- API keys:
  - Supabase service role key (for server-side insert/search)
  - Groq API key
  - Hugging Face API key (embeddings)

---

## Setup

### 1) Supabase: create table + function
Run the SQL in `schema.sql` in the Supabase SQL editor.

This creates:
- `documents` table with `embedding vector(768)`
- `match_documents(query_embedding vector(768), match_count int)` function used by n8n’s Supabase Vector Store node

### 2) Run n8n with a mounted `/files` folder
Mount a local folder that contains your PDF(s) into the container at `/files`.

Example:
- Put your PDF into `./files/attention-is-all-you-need.pdf`
- Ensure the workflow node path matches: `/files/attention-is-all-you-need.pdf`

### 3) Import workflows into n8n
In n8n UI:
- Import:
  - `workflows/attention-embedding.json`
  - `workflows/chatbot.json`

### 4) Configure credentials in n8n
Set up credentials in n8n for:
- Supabase
- Hugging Face Inference (embeddings)
- Groq (LLM)

### 5) Run ingestion once
Execute the ingestion workflow to populate the `documents` table.

### 6) Test chat retrieval
Open the n8n chat for the Chat Trigger and ask questions about the ingested document.

---

## Notes / Known Issues
- If the ingestion workflow cannot find the PDF, verify Docker volume mounting and that the workflow path starts with `/files/...`.
- If retrieval fails with a missing function error, ensure `match_documents` exists and the embedding dimension is **vector(768)**.

---

## Security
Do not commit secrets:
- API keys
- `.env`
- customer documents or proprietary data

Use `.env.example` and configure credentials inside n8n.

---
