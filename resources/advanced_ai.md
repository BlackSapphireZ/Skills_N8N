# n8n Advanced AI

This document covers n8n's AI capabilities including LangChain integration, AI agents, and chat workflows.

## Overview

n8n provides advanced AI functionality through LangChain integration, available in version 1.19.4 and above. Build AI-powered workflows for chatbots, document processing, RAG systems, and more.

## AI Node Architecture

### Cluster Nodes

AI features use a cluster node pattern:

```
Root Node (AI Agent)
    ├── Chat Model (OpenAI/Anthropic)
    ├── Memory (Buffer/PostgreSQL)
    ├── Tools (Calculator/HTTP/Code)
    └── Vector Store (for RAG)
         └── Embeddings
```

### Root Nodes

Main AI processing nodes:

| Node | Purpose |
|------|---------|
| **AI Agent** | Autonomous agent with tools |
| **Basic LLM Chain** | Simple prompt → response |
| **Summarization Chain** | Text summarization |
| **Q&A Chain** | Question answering |
| **Conversational Agent** | Multi-turn chat |

### Sub-Nodes

Connected to root nodes:

| Category | Nodes |
|----------|-------|
| **Chat Models** | OpenAI, Anthropic, Google AI, Ollama, Azure OpenAI |
| **Embeddings** | OpenAI Embeddings, Cohere, HuggingFace |
| **Vector Stores** | Pinecone, Qdrant, Supabase, Postgres pgvector |
| **Memory** | Buffer Memory, PostgreSQL Chat Memory, Redis |
| **Tools** | Calculator, Code, HTTP Request, Wikipedia |

---

## Chat Models

### OpenAI

```javascript
// Configuration
Credentials: OpenAI API
Model: gpt-4-turbo-preview
Temperature: 0.7
Max Tokens: 1000
```

Supported models:
- gpt-4, gpt-4-turbo, gpt-4o
- gpt-3.5-turbo
- Custom fine-tuned models

### Anthropic

```javascript
// Configuration
Credentials: Anthropic API
Model: claude-3-sonnet
Max Tokens to Sample: 1000
Temperature: 0.7
```

Supported models:
- claude-3-opus
- claude-3-sonnet
- claude-3-haiku

### Google AI (Gemini)

```javascript
// Configuration
Credentials: Google AI API
Model: gemini-pro
Temperature: 0.7
Max Output Tokens: 1000
```

### Ollama (Self-hosted)

For local LLM deployment:

```javascript
// Configuration
Base URL: http://localhost:11434
Model: llama2
Temperature: 0.7
```

---

## AI Agent

### Basic AI Agent Setup

```
Chat Trigger → AI Agent → Response
                 ↑
            Chat Model
```

### Agent with Tools

```
Chat Trigger → AI Agent → Response
                 ↑
            Chat Model
                 ↑
         ┌──────┼──────┐
     Calculator  HTTP   Code
```

### Tool Types

| Tool | Purpose |
|------|---------|
| **Calculator** | Math operations |
| **Code** | Execute custom code |
| **HTTP Request Tool** | API calls |
| **Wikipedia** | Knowledge lookup |
| **SerpAPI** | Web search |
| **Custom Tool** | Any n8n workflow |

### Agent Configuration

```javascript
// AI Agent settings
System Message: "You are a helpful assistant..."
Max Iterations: 10
Return Intermediate Steps: false
```

---

## Vector Stores (RAG)

### Retrieval Augmented Generation

```
Document → Chunk → Embed → Store → Query → Context → LLM
```

### Supported Vector Stores

#### Pinecone

```javascript
// Configuration
Credentials: Pinecone API
Index Name: my-index
Namespace: default
```

#### Qdrant

```javascript
// Configuration
Credentials: Qdrant API
Collection Name: my-collection
URL: https://xyz.qdrant.io
```

#### Supabase Vector

```javascript
// Configuration
Credentials: Supabase API
Table Name: documents
Query Name: match_documents
```

#### PostgreSQL pgvector

```javascript
// Configuration
Credentials: PostgreSQL
Table Name: embeddings
Embedding Column: embedding
Content Column: content
```

### Document Loading

Load documents for indexing:

| Loader | Source |
|--------|--------|
| **Text File** | .txt files |
| **PDF** | PDF documents |
| **JSON** | JSON data |
| **CSV** | Spreadsheets |
| **URL/Web** | Web pages |
| **Notion** | Notion pages |

### Text Splitters

Chunk documents for processing:

| Splitter | Strategy |
|----------|----------|
| **Character Splitter** | Fixed character count |
| **Token Splitter** | Token-based chunking |
| **Markdown Splitter** | Respect markdown structure |
| **HTML Splitter** | Parse HTML structure |

---

## Memory

### Buffer Memory

In-session conversation memory:

```javascript
// Configuration
Session Key: {{ $json.sessionId }}
Memory Key: chat_history
```

### PostgreSQL Chat Memory

Persistent conversation storage:

```javascript
// Configuration
Credentials: PostgreSQL
Session ID: {{ $json.sessionId }}
Table Name: chat_history
```

### Redis Chat Memory

Fast distributed memory:

```javascript
// Configuration
Credentials: Redis
Session ID: {{ $json.sessionId }}
TTL: 3600
```

### Memory in Workflows

```
Chat Trigger → AI Agent → Response
                 ↑
            Chat Model
                 ↑
              Memory
```

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

## AI Workflow Examples

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
