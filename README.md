# Graph RAG: PDF Knowledge Ingestion with Neo4j + Qdrant (n8n Workflow)

An automated Graph RAG (Retrieval-Augmented Generation) pipeline built on **n8n** that ingests PDF documents, builds a **Neo4j knowledge graph** of entities and relationships, indexes the same content into a **Qdrant vector store**, and exposes both to a conversational AI agent capable of multi-hop reasoning. Inspired by NVIDIA's knowledge graph and Graph RAG architecture patterns.

---

## 🚀 Key Features

| Feature | Description |
|---|---|
| **Automated PDF Ingestion** | Reads PDF files, extracts and splits text into chunks ready for parallel processing. |
| **LLM-Powered Entity Extraction** | Uses an LLM (Groq) to extract entities and relationships from text and dynamically build Cypher queries. |
| **Knowledge Graph Population** | Writes subject → predicate → object triples directly into Neo4j via generated Cypher. |
| **Semantic Vector Indexing** | Embeds each chunk with Google Gemini and stores it in Qdrant for similarity search. |
| **Graph RAG Agent** | A chat agent with two tools — Qdrant Vector Store and Neo4j Graph Tool — that chooses the right retrieval strategy per question. |
| **Multi-Hop Reasoning** | Answers questions that require traversing several relationships, not just single-document lookup. |

---

## 🏗️ Architecture & Pipelines

The project is built as two connected n8n workflows:

| Workflow | Purpose | Key Nodes |
|---|---|---|
| **Ingest: PDF → Graph Neo4j + RAG Qdrant** | Converts raw PDFs into both a knowledge graph and a vector index. | Read PDF Files, Extract Text, Split Text, Entity Extraction LLM, Parse Entity JSON, Build Cypher, Neo4j, Qdrant Store, Embeddings (Gemini) |
| **Chat: Query** | Conversational entry point that answers user questions using both retrieval sources. | Chat Trigger, AI Agent, Groq Chat Model, Simple Memory, Neo4j Graph Tool, Qdrant Vector Store |

### Ingestion Flow

```
PDF File
   ↓
Extract Text → Split Text
   ↓                    ↓
Entity Extraction   Set for Qdrant
   (LLM)                ↓
   ↓                Embeddings (Gemini)
Parse Entity JSON        ↓
   ↓                 Qdrant Store
Build Cypher
   ↓
Neo4j (executeQuery)
```

### Query Flow

```
User Question
      ↓
   AI Agent (Groq)
      ↓ decides which tool(s) to use
 ┌──────────────┬──────────────────┐
 Qdrant Vector    Neo4j Graph Tool
 Store (semantic  (Cypher traversal,
 similarity)      relationships)
 └──────────────┴──────────────────┘
              ↓
       Combined context
              ↓
        Final Answer
```

The agent's system message explicitly instructs it to try the Qdrant Vector Store for semantically similar passages and the Neo4j Graph Tool for questions involving connections between entities, always attempting retrieval before answering, and stating clearly when information isn't available in the knowledge base.

---

## 🛠️ Tech Stack

- **Orchestration:** n8n (visual workflow automation)
- **LLM / Reasoning:** Groq Chat Model
- **Embeddings:** Google Gemini
- **Vector Database:** Qdrant
- **Graph Database:** Neo4j (Cypher query language)
- **Memory:** Simple Memory (chat session context)

---

## 📄 Project Structure

| Workflow | Purpose | Key Components |
|---|---|---|
| `Ingest: PDF → Graph Neo4j + RAG Qdrant` | Builds the knowledge base from source PDFs. | Text extraction/splitting, LLM entity extraction, Cypher generation, Neo4j write, Qdrant embedding + storage. |
| `Chat: Query` | Interactive chat interface over the knowledge base. | Chat trigger, AI Agent, Groq model, session memory, Neo4j & Qdrant tools. |

---

## 💻 Usage

1. Import both n8n workflow JSON files into your n8n instance.
2. Configure credentials for Groq, Google Gemini, Neo4j, and Qdrant.
3. Run **Ingest: PDF → Graph Neo4j + RAG Qdrant** on your source PDF(s) to populate the graph and vector store.
4. Open **Chat: Query**, start a chat session, and ask questions — the agent will automatically choose between semantic search and graph traversal (or both) to answer.

---

## 📚 Background

This project was built as a practical follow-up to studying knowledge graphs and Graph RAG concepts, applying the theory (triples, multi-hop reasoning, entity resolution, Graph RAG retrieval patterns) to a working end-to-end pipeline.