# n8n Advanced AI

This document covers n8n's AI capabilities including LangChain integration, AI agents, and chat workflows.

## Overview

n8n provides advanced AI functionality through LangChain integration, available in version 1.19.4 and above. Build AI-powered workflows for chatbots, document processing, RAG systems, and more.

## AI Node Architecture

### Cluster Nodes

AI features use a cluster node pattern with a root node and sub-nodes attached via special connectors:

```
Chat Trigger → AI Agent (root node)
                ├── Chat Model (required, ai_languageModel)
                ├── Memory (optional, ai_memory)
                ├── Tools (optional, ai_tool)
                ├── Output Parser (optional, ai_outputParser)
                └── Retriever (optional, ai_retriever)
```

All AI Agent nodes work as **Tools Agent** (since v1.82.0), which uses LangChain's tool-calling interface.

### Root Nodes

| Node | Purpose |
|------|---------|
| **AI Agent** | Autonomous agent with tools |
| **Basic LLM Chain** | Simple prompt → response |
| **Summarization Chain** | Text summarization |
| **Q&A Chain** | Question answering |
| **Conversational Agent** | Multi-turn chat |

### Sub-Nodes

| Category | Nodes |
|----------|-------|
| **Chat Models** | OpenAI, Anthropic, Google AI, Ollama, Azure OpenAI |
| **Embeddings** | OpenAI Embeddings, Cohere, HuggingFace |
| **Vector Stores** | Pinecone, Qdrant, Supabase, Postgres pgvector |
| **Memory** | Buffer Memory, PostgreSQL Chat Memory, Redis |
| **Tools** | Calculator, Code, HTTP Request, Wikipedia, MCP Client |

---

## AI Agent Node

```json
{
  "parameters": {
    "options": {
      "systemMessage": "You are a helpful assistant that answers questions accurately.",
      "maxIterations": 10,
      "returnIntermediateSteps": false
    }
  },
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 1.7,
  "name": "AI Agent"
}
```

**Key parameters:**
- `systemMessage`: The system prompt that defines agent behavior
- `maxIterations`: Max tool-calling loops (default 10)
- `returnIntermediateSteps`: Include agent reasoning in output
- Prompt source: `"Take from previous node automatically"` expects `chatInput` field from trigger

**Output fields:**
- `output`: The agent's final text response
- `intermediateSteps`: Array of tool calls and results (if enabled)

---

## Chat Models

### OpenAI Chat Model
```json
{
  "parameters": {
    "model": "gpt-4o",
    "options": {
      "temperature": 0.7,
      "maxTokens": 1000,
      "topP": 1
    }
  },
  "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
  "typeVersion": 1.2,
  "name": "OpenAI Chat Model",
  "credentials": { "openAiApi": { "id": "cred-id", "name": "OpenAI Account" } }
}
```

Supported models: gpt-4, gpt-4-turbo, gpt-4o, gpt-3.5-turbo, custom fine-tuned models

### Anthropic Chat Model
```json
{
  "parameters": {
    "model": "claude-sonnet-4-20250514",
    "options": {
      "temperature": 0.7,
      "maxTokensToSample": 4096
    }
  },
  "type": "@n8n/n8n-nodes-langchain.lmChatAnthropic",
  "typeVersion": 1.3,
  "name": "Anthropic Chat Model",
  "credentials": { "anthropicApi": { "id": "cred-id", "name": "Anthropic Account" } }
}
```

Supported models: claude-3-opus, claude-3-sonnet, claude-3-haiku

### Google Gemini Chat Model
```json
{
  "parameters": {
    "modelName": "models/gemini-2.0-flash",
    "options": { "temperature": 0.7 }
  },
  "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
  "typeVersion": 1.1,
  "name": "Google Gemini Chat Model",
  "credentials": { "googlePalmApi": { "id": "cred-id", "name": "Google Gemini" } }
}
```

### Ollama (Local) Chat Model
```json
{
  "parameters": {
    "model": "llama3",
    "options": { "temperature": 0.7 }
  },
  "type": "@n8n/n8n-nodes-langchain.lmChatOllama",
  "typeVersion": 1,
  "name": "Ollama Chat Model",
  "credentials": { "ollamaApi": { "id": "cred-id", "name": "Ollama" } }
}
```

For local LLM deployment at `http://localhost:11434`.

---

## Memory Nodes

### Simple Memory (Buffer Window)
Stores last N interactions in workflow memory. Does NOT persist between sessions.
```json
{
  "parameters": {
    "sessionKey": "={{ $json.sessionId ?? 'default' }}",
    "contextWindowLength": 10
  },
  "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
  "typeVersion": 1.3,
  "name": "Simple Memory"
}
```
**Warning**: Does not work in queue mode for production workflows.

### Postgres Chat Memory
Persists conversation history in PostgreSQL.
```json
{
  "parameters": {
    "sessionKey": "={{ $json.sessionId }}",
    "tableName": "n8n_chat_histories",
    "contextWindowLength": 20
  },
  "type": "@n8n/n8n-nodes-langchain.memoryPostgresChat",
  "typeVersion": 1.1,
  "name": "Postgres Chat Memory",
  "credentials": { "postgres": { "id": "cred-id", "name": "Postgres" } }
}
```

### Other Memory Types
| Memory | Type | Persistence |
|--------|------|-------------|
| Redis | `@n8n/n8n-nodes-langchain.memoryRedisChat` | Redis server |
| Motorhead | `@n8n/n8n-nodes-langchain.memoryMotorhead` | Motorhead server |
| Xata | `@n8n/n8n-nodes-langchain.memoryXata` | Xata database |
| Zep | `@n8n/n8n-nodes-langchain.memoryZep` | Zep server |

---

## Tool Nodes

Tools give the AI Agent ability to perform actions. Connect via `ai_tool` connector.

### n8n Workflow Tool (call any workflow as a tool)
```json
{
  "parameters": {
    "name": "lookup_customer",
    "description": "Look up customer information by email address",
    "workflowId": "workflow-uuid",
    "fields": {
      "values": [
        { "name": "email", "description": "Customer email to look up", "type": "string" }
      ]
    }
  },
  "type": "@n8n/n8n-nodes-langchain.toolWorkflow",
  "typeVersion": 2,
  "name": "Lookup Customer Tool"
}
```

### HTTP Request Tool
```json
{
  "parameters": {
    "name": "get_weather",
    "description": "Get current weather for a city",
    "method": "GET",
    "url": "=https://api.weather.com/v1/current?city={{ $fromAI('city', 'city name') }}",
    "options": {}
  },
  "type": "@n8n/n8n-nodes-langchain.toolHttpRequest",
  "typeVersion": 1.1,
  "name": "Weather Tool"
}
```

### Code Tool (custom JS function)
```json
{
  "parameters": {
    "name": "calculate",
    "description": "Perform mathematical calculations. Input should be a math expression.",
    "jsCode": "const math = require('mathjs');\nconst result = math.evaluate($input);\nreturn JSON.stringify({ result });"
  },
  "type": "@n8n/n8n-nodes-langchain.toolCode",
  "typeVersion": 1.1,
  "name": "Calculator Tool"
}
```

### Built-in Tool Nodes
| Tool | Type | Purpose |
|------|------|---------|
| Calculator | `@n8n/n8n-nodes-langchain.toolCalculator` | Math operations |
| Wikipedia | `@n8n/n8n-nodes-langchain.toolWikipedia` | Wikipedia search |
| WolframAlpha | `@n8n/n8n-nodes-langchain.toolWolframAlpha` | Computational queries |
| SerpAPI | `@n8n/n8n-nodes-langchain.toolSerpApi` | Web search |
| Vector Store Tool | `@n8n/n8n-nodes-langchain.toolVectorStore` | Query vector DB |
| MCP Client | `@n8n/n8n-nodes-langchain.mcpClient` | Model Context Protocol tools |

### $fromAI() Function
Use in tool parameters to let the AI dynamically provide values:
```
{{ $fromAI('paramName', 'description of what this param is', 'string') }}
```
The AI Agent will fill these parameters based on its reasoning.

---

## Document Loaders & Vector Stores

### Document Loaders (load data for RAG)
| Loader | Type | Source |
|--------|------|--------|
| File (Binary) | `@n8n/n8n-nodes-langchain.documentDefaultDataLoader` | Workflow binary data |
| GitHub | `@n8n/n8n-nodes-langchain.documentLoaderGithub` | GitHub repos |
| Notion | `@n8n/n8n-nodes-langchain.documentLoaderNotion` | Notion pages |

### Text Splitters
| Splitter | Type |
|----------|------|
| Recursive Character | `@n8n/n8n-nodes-langchain.textSplitterRecursiveCharacterTextSplitter` |
| Token Splitter | `@n8n/n8n-nodes-langchain.textSplitterTokenSplitter` |

### Vector Stores
| Store | Type |
|-------|------|
| Pinecone | `@n8n/n8n-nodes-langchain.vectorStorePinecone` |
| Supabase | `@n8n/n8n-nodes-langchain.vectorStoreSupabase` |
| Qdrant | `@n8n/n8n-nodes-langchain.vectorStoreQdrant` |
| In-Memory | `@n8n/n8n-nodes-langchain.vectorStoreInMemory` |
| PGVector | `@n8n/n8n-nodes-langchain.vectorStorePGVector` |

### Embeddings
| Model | Type |
|-------|------|
| OpenAI | `@n8n/n8n-nodes-langchain.embeddingsOpenAi` |
| Google | `@n8n/n8n-nodes-langchain.embeddingsGoogleGemini` |
| Ollama | `@n8n/n8n-nodes-langchain.embeddingsOllama` |
| HuggingFace | `@n8n/n8n-nodes-langchain.embeddingsHuggingFaceInference` |

### RAG Pipeline

```
Document → Chunk → Embed → Store → Query → Context → LLM
```

**Configuration examples:**

#### Pinecone
```javascript
Credentials: Pinecone API
Index Name: my-index
Namespace: default
```

#### Qdrant
```javascript
Credentials: Qdrant API
Collection Name: my-collection
URL: https://xyz.qdrant.io
```

#### Supabase Vector
```javascript
Credentials: Supabase API
Table Name: documents
Query Name: match_documents
```

#### PostgreSQL pgvector
```javascript
Credentials: PostgreSQL
Table Name: embeddings
Embedding Column: embedding
Content Column: content
```

---

## Output Parsers

### Structured Output Parser
```json
{
  "parameters": {
    "schemaType": "fromJson",
    "jsonSchemaExample": "{\n  \"sentiment\": \"positive\",\n  \"confidence\": 0.95,\n  \"summary\": \"brief summary\"\n}"
  },
  "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
  "typeVersion": 1.2,
  "name": "Structured Output Parser"
}
```

### Auto-Fixing Output Parser
Wraps another parser and retries on failure:
```json
{
  "parameters": {},
  "type": "@n8n/n8n-nodes-langchain.outputParserAutofixing",
  "typeVersion": 1,
  "name": "Auto-Fixing Output Parser"
}
```

---

## AI Connection Types

When building connections for AI nodes, use these special types:

| Connection Type | Purpose | Connector |
|----------------|---------|-----------|
| `main` | Standard data flow | Regular node connections |
| `ai_languageModel` | Chat/LLM model | Agent ← Model |
| `ai_memory` | Conversation memory | Agent ← Memory |
| `ai_tool` | Agent tools | Agent ← Tool |
| `ai_outputParser` | Response formatting | Chain/Agent ← Parser |
| `ai_retriever` | RAG retriever | Chain ← Retriever |
| `ai_document` | Document input | Vector Store ← Document Loader |
| `ai_embedding` | Embedding model | Vector Store ← Embeddings |
| `ai_textSplitter` | Text chunking | Document Loader ← Splitter |

---

## Chat Trigger

### Configuration

```javascript
// Chat Trigger settings
Allowed Origins: *
Authentication: None/Basic/JWT
Initial Message: "How can I help you?"
```

### Chat Widget

n8n provides a chat widget for frontends:

```html
<!-- Add to your website -->
<script src="https://cdn.n8n.io/chat.js"></script>
<script>
  n8n.createChat({
    webhookUrl: 'https://your-n8n.com/webhook/chat',
    target: '#chat-container'
  });
</script>
```

### npm Package

```bash
npm install @n8n/chat
```

```javascript
import { createChat } from '@n8n/chat';

createChat({
  webhookUrl: 'https://your-n8n.com/webhook/chat',
  initialMessages: ['Hello! How can I help?']
});
```

---

## Minimal AI Agent Example

Complete Chat Trigger → AI Agent → OpenAI workflow (importable JSON):

```json
{
  "name": "AI Chat Agent",
  "nodes": [
    {
      "parameters": { "options": {} },
      "id": "chat-trigger-id",
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "options": {
          "systemMessage": "You are a helpful assistant. Be concise and accurate."
        }
      },
      "id": "agent-id",
      "name": "AI Agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 1.7,
      "position": [500, 300]
    },
    {
      "parameters": {
        "model": "gpt-4o",
        "options": { "temperature": 0.7 }
      },
      "id": "model-id",
      "name": "OpenAI Chat Model",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [500, 500],
      "credentials": { "openAiApi": { "id": "1", "name": "OpenAI Account" } }
    },
    {
      "parameters": { "sessionKey": "default", "contextWindowLength": 10 },
      "id": "memory-id",
      "name": "Simple Memory",
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [700, 500]
    }
  ],
  "connections": {
    "Chat Trigger": {
      "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]]
    },
    "Simple Memory": {
      "ai_memory": [[{ "node": "AI Agent", "type": "ai_memory", "index": 0 }]]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

## AI Workflow Patterns

### Simple Chatbot
```
Chat Trigger → AI Agent → Response
                 ↑
            OpenAI Chat Model
```

### RAG Chatbot
```
Chat Trigger → Retrieve Documents → AI Agent → Response
                      ↑                ↑
              Vector Store         Chat Model
                      ↑
                 Embeddings
```

### Document Processor
```
Webhook (PDF) → Extract Text → Summarize → Store Summary
                                   ↑
                              Chat Model
```

### Multi-Agent System
```
Input → Classifier Agent → Specialist Agent 1 → Merge → Output
              ↓
       Specialist Agent 2 ─────────────────────┘
```

---

## AI Starter Kit

### Quick Start with Docker

```yaml
# docker-compose.yml
version: '3.8'
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:ai-beta
    ports:
      - "5678:5678"
    environment:
      - OPENAI_API_KEY=your-key
    volumes:
      - n8n_data:/home/node/.n8n

  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage

  ollama:
    image: ollama/ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama

volumes:
  n8n_data:
  qdrant_data:
  ollama_data:
```

### Included Components

- n8n with AI capabilities
- Qdrant vector database
- Ollama for local LLMs
- Pre-built workflow templates

---

## Best Practices

### Prompt Engineering

1. **Be specific** - Clear instructions
2. **Use examples** - Few-shot prompting
3. **Set constraints** - Output format, length
4. **Handle errors** - Fallback behaviors

### RAG Optimization

1. **Chunk effectively** - Optimal chunk size (512-1024 tokens)
2. **Overlap chunks** - 10-20% overlap
3. **Quality embeddings** - Choose appropriate model
4. **Metadata filtering** - Include relevant metadata

### AI Workflow Tips

1. **Cache responses** - Reduce API costs
2. **Set timeouts** - Prevent hanging
3. **Log interactions** - Debug and improve
4. **Monitor costs** - Track API usage
5. **Test thoroughly** - Validate outputs

---

## Resources

- [AI Documentation](https://docs.n8n.io/advanced-ai/)
- [AI Starter Kit](https://docs.n8n.io/hosting/starter-kits/ai-starter-kit/)
- [AI Workflow Templates](https://n8n.io/workflows/?categories=25)
- [LangChain Documentation](https://docs.langchain.com/)
