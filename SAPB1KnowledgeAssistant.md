# KnowledgeAssistant

A **Retrieval-Augmented Generation (RAG)** application built with .NET that serves as an **SAP Business One AI assistant**. It ingests PDF documents, indexes them with OpenAI embeddings into a Qdrant vector database, and answers user questions by retrieving the most relevant document chunks and passing them as context to an LLM.

## Architecture

The solution follows **Clean Architecture** with four layers:

| Layer | Project | Purpose |
|-------|---------|---------|
| **Domain** | `Domain/` | Core entities (`Document`, `DocumentChunk`) and DTOs |
| **Application** | `Application/` | Interfaces and service orchestration (`DocumentProcessingService`) |
| **Infrastructure** | `Infrastructure/` | Implementations — OpenAI, Qdrant, PDF extraction, EF Core + SQL Server |
| **API** | `Api/` | REST API exposing `/api/chat` for question answering |
| **UI** | `BlazorUI/` | Blazor Server frontend (scaffolded, not yet wired) |

### How it works

1. **Upload** — A PDF document is stored via `DocumentRepository` (SQL Server)
2. **Process** — `DocumentProcessingService` extracts text with PdfPig, chunks it, generates OpenAI embeddings, and upserts them into Qdrant
3. **Query** — The `POST /api/chat` endpoint accepts a question, searches Qdrant for relevant chunks, and sends them as context to an OpenAI chat model
4. **Answer** — The LLM answers strictly from the provided context

## Tech Stack

- **.NET 8.0** (Api, Application, Domain, Infrastructure) / **.NET 10.0** (BlazorUI)
- **OpenAI** — text-embedding-3-small (embeddings) + GPT (chat)
- **Qdrant** — vector database for similarity search
- **SQL Server** — document metadata storage via Entity Framework Core
- **PdfPig** — PDF text extraction
- **Swagger** — API documentation and testing

## Prerequisites

- [.NET SDK 8.0+](https://dotnet.microsoft.com/download)
- [SQL Server](https://www.microsoft.com/sql-server) (local or remote)
- [Qdrant](https://qdrant.tech/) — easiest way is Docker:

```bash
docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant
```

- An [OpenAI API key](https://platform.openai.com/api-keys)

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/jtaptso/SBOneAIKnowlegde.git
cd SBOneAIKnowlegde/src/KnowledgeAssistant
```

### 2. Configure the API

Edit `Api/appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=SapAIAssistantDb;Trusted_Connection=True;TrustServerCertificate=True;Encrypt=True"
  },
  "OpenAI": {
    "EmbeddingModel": "text-embedding-3-small",
    "ApiKey": "sk-your-openai-api-key"
  },
  "Qdrant": {
    "Host": "localhost",
    "Port": 6334
  }
}
```

### 3. Apply database migrations

```bash
cd Api
dotnet ef database update
```

### 4. Run the API

```bash
cd Api
dotnet run
```

The API starts at `https://localhost:5001` (or `http://localhost:5000`). Swagger UI is available at `/swagger`.

### 5. Run the BlazorUI (optional)

```bash
cd BlazorUI
dotnet run
```

## API Usage

### Ask a question

```bash
curl -X POST https://localhost:5001/api/chat \
  -H "Content-Type: application/json" \
  -d '{"question": "How do I create a sales order in SAP B1?"}'
```

**Response:**

```json
{
  "question": "How do I create a sales order in SAP B1?",
  "answer": "To create a sales order, navigate to Sales → Sales Order..."
}
```

## Project Structure

```
KnowledgeAssistant/
├── Domain/                  # Entities & DTOs
│   ├── Document.cs
│   ├── DocumentChunk.cs
│   └── DTO/
│       └── VectorSearchResult.cs
├── Application/             # Interfaces & services
│   ├── Interfaces/
│   │   ├── IChatService.cs
│   │   ├── IDocumentProcessingService.cs
│   │   ├── IEmbeddingService.cs
│   │   ├── IPdfExtractionService.cs
│   │   ├── ITextChunkingService.cs
│   │   ├── IVectorSearchService.cs
│   │   └── IVectorStore.cs
│   └── Services/
│       └── DocumentProcessingService.cs
├── Infrastructure/          # Implementations
│   ├── AI/
│   │   ├── EmbeddingService.cs
│   │   └── OpenAIChatService.cs
│   ├── Pdf/
│   │   ├── PdfExtractionService.cs
│   │   └── TextChunkingService.cs
│   ├── Persistence/
│   │   └── ApplicationDbContext.cs
│   ├── Qdrant_VectorDB/
│   │   ├── QdrantVectorSearchService.cs
│   │   └── QdrantVectorStore.cs
│   └── Repositories/
│       └── DocumentRepository.cs
├── Api/                     # REST API
│   ├── Controllers/
│   │   └── ChatController.cs
│   └── Program.cs
└── BlazorUI/                # Blazor Server frontend
    └── Program.cs
```

## License

Proprietary — all rights reserved.
