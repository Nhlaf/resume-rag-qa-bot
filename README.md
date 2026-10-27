# Resume RAG QA Bot

A TypeScript API for ingesting resumes and answering questions about them. It combines MongoDB Atlas Vector Search with keyword, vector, or hybrid retrieval and can use an LLM to rerank results. Conversational chat keeps short-lived in-memory history per conversation.

## Features

- Ingest PDF and DOCX resumes from a local folder into MongoDB with embeddings.
- Search resumes by keyword, vector similarity, or hybrid search.
- Chat with retrieved resumes and ask follow-up questions.
- Ask questions about an individual document or supplied text.
- Choose Groq, OpenAI, Anthropic, or Testleaf as the chat model provider.
- Choose Mistral or OpenAI for embeddings.

## Requirements

- Node.js 18.17 or later
- MongoDB Atlas configured for Vector Search
- API credentials for a supported chat model and embedding provider

## Setup

1. Install dependencies:

   ```sh
   npm install
   ```

2. Copy `.env.example` to `.env` and set your MongoDB URI and provider credentials. Keep `.env` private; it is ignored by Git.

3. Configure the MongoDB Atlas vector search index named by `MONGODB_VECTOR_INDEX` (default: `resume_vector_index`) for the embeddings collection. The index must use the same vector dimension as `EMBEDDING_DIMENSION`.

4. Put the PDF or DOCX resumes you want to process in `documents/`. Local documents are ignored by Git and are not included in this repository.

## Run

Start the development server with reload:

```sh
npm run dev
```

Build and run the compiled server:

```sh
npm run build
npm start
```

Ingest the resumes in `documents/`:

```sh
node ./node_modules/tsx/dist/cli.mjs src/pipelines/ingestion/pipeline.ts
```

Pass `--clear` to delete existing resume records before ingesting:

```sh
node ./node_modules/tsx/dist/cli.mjs src/pipelines/ingestion/pipeline.ts --clear
```

The API listens on port `8787` by default. Override it with `PORT`.

## API

All request and response bodies are JSON.

### Health

`GET /health`

Reports server health, configured chat model, retrieval initialization status, and active conversations.

### Search resumes

`POST /search/resumes`

```json
{
  "query": "TypeScript developer with MongoDB experience",
  "searchType": "hybrid",
  "topK": 3
}
```

`searchType` must be `keyword`, `vector`, or `hybrid`. `topK` is optional and defaults to `3`.

### Conversational resume search

`POST /chat`

```json
{
  "message": "Find candidates with healthcare experience",
  "topK": 10
}
```

The response includes a `conversationId`. Send it with later messages to continue the conversation. Set `includeHistory` to `false` to omit history from the RAG prompt.

### Get conversation history

`POST /chat/history`

```json
{
  "conversationId": "your-conversation-id"
}
```

### Delete a conversation

`DELETE /chat/:conversationId`

### Ask about one document

`POST /search/document`

Provide either `documentText` or a server-local `documentPath` together with a question. `promptType` can be `default`, `detailed`, `concise`, or `technical`.

```json
{
  "question": "What are the candidate's main skills?",
  "documentText": "Paste document text here...",
  "promptType": "concise"
}
```

## Configuration

See `.env.example` for the primary settings. `MODEL_PROVIDER` supports `groq`, `openai`, `anthropic`, and `testleaf`; configure the corresponding API key and model variable. `EMBEDDING_PROVIDER` supports `mistral` and `openai`.

## Development

```sh
npm run typecheck
npm run build
```

Resume contents and contact details are personal data. Use appropriate access controls for the API, database, and any documents you process.
