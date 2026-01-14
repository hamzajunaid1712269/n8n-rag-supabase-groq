# n8n-rag-supabase-groq
This repository contains a two-workflow Retrieval-Augmented Generation (RAG) prototype built with n8n, Supabase (pgvector), Hugging Face embeddings, and a Groq-hosted LLM. The system ingests PDFs into a vector database once, then serves conversational Q&amp;A through an agent that retrieves relevant chunks via vector search.
