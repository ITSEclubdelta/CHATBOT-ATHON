# AI Chatbot

A minimal, clean chatbot web app that wraps an LLM API. Built with Next.js, TypeScript, and Tailwind CSS.

## Features

- Chat UI with auto-scroll and typing indicator
- Backend API route that calls any OpenAI-compatible model
- Fixed system prompt (configurable via env var)
- Short-term conversation memory per browser session (in-memory, last 20 messages)
- Token-efficient prompt assembly — system prompt is sent once, history is trimmed
- Usage logging per request (prompt/completion/total tokens)
- Basic error handling with user-facing messages

## Project structure

```
app/
  api/chat/route.ts   — POST handler: validates input, calls LLM, logs usage
  layout.tsx          — root layout
  page.tsx            — renders ChatInterface
components/
  ChatInterface.tsx   — chat UI (messages, input, loading, errors)
lib/
  session-store.ts    — in-memory Map of message history keyed by session ID
  llm-client.ts       — OpenAI client + config
  prompt-builder.ts   — assembles [system, ...history, user] for each request
types/
  chat.ts             — shared TypeScript types
```

## Quick start

```bash
# 1. Copy and fill in env vars
cp .env.local.example .env.local
# edit .env.local — set OPENAI_API_KEY at minimum

# 2. Install dependencies
npm install

# 3. Run dev server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

## Environment variables

| Variable               | Required | Default              | Description                                  |
|------------------------|----------|----------------------|----------------------------------------------|
| `OPENAI_API_KEY`       | Yes      | —                    | API key for OpenAI or compatible provider    |
| `OPENAI_BASE_URL`      | No       | OpenAI default       | Override for compatible APIs (e.g. Ollama)   |
| `OPENAI_MODEL`         | No       | `gpt-4o-mini`        | Model identifier                             |
| `MAX_COMPLETION_TOKENS`| No       | `1024`               | Max tokens in each assistant response        |
| `SYSTEM_PROMPT`        | No       | See `prompt-builder` | Override the fixed system prompt             |

## Token efficiency design

- The system prompt is **fixed** — same string on every call, enabling prompt caching if the provider supports it.
- History is **trimmed** to the last 20 messages (10 turns) before assembling each request.
- Dynamic context (history + new message) is placed **after** the static system prompt, so cached prefixes stay valid.
- `max_tokens` caps completion length.

## Extending

- **Swap the model provider**: set `OPENAI_BASE_URL` to point at any OpenAI-compatible endpoint (Ollama, Together, Groq, etc.).
- **Change the persona**: edit `SYSTEM_PROMPT` in `.env.local` or directly in `lib/prompt-builder.ts`.
- **Persist history**: replace the `Map` in `lib/session-store.ts` with Redis or a database.
- **Stream responses**: change `llm.chat.completions.create` to use `stream: true` and return a `ReadableStream` from the route handler.
# CHATBOT-ATHON
