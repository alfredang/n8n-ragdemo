# n8n RAG Demo — PDF → Vector Store → Telegram

A self-contained teaching demo for building a **Retrieval-Augmented Generation (RAG)** assistant with [n8n](https://n8n.io/). Upload a PDF from a single-page web app into an in-memory vector store, then ask questions about it through a Telegram bot powered by Google Gemini.

![n8n RAG Demo — home screen](screenshot.png)

![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?logo=tailwindcss&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-EA4B71?logo=n8n&logoColor=white)
![Google Gemini](https://img.shields.io/badge/Google_Gemini-8E75B2?logo=googlegemini&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-26A5E4?logo=telegram&logoColor=white)

## Overview

This repo accompanies a Tertiary Infotech n8n training module. It demonstrates the full RAG loop using only no-code/low-code building blocks:

```
PDF  →  Vector Store  →  AI Agent  →  Telegram
(your doc)  (text + embeddings)  (retrieves + answers)  (you chat here)
```

1. **Ingest** — A browser-based uploader ([index.html](index.html)) extracts text from a PDF client-side with PDF.js and POSTs it to an n8n `rag-upload` webhook.
2. **Embed & store** — n8n chunks the text, generates embeddings with Google Gemini, and inserts them into an in-memory vector store.
3. **Retrieve & answer** — A Telegram-triggered AI Agent searches the vector store (`knowledge_base` tool) and replies using only the retrieved content.

## Components

| File | Purpose |
|------|---------|
| [index.html](index.html) | Single-page PDF uploader — paste your webhook URL, drop a PDF, send extracted text to the vector store. No build step. |
| [rag-flow.json](rag-flow.json) | The complete n8n workflow — import this into your n8n instance. |
| [it-faq.pdf](it-faq.pdf) | Sample HR/IT FAQ document to test the pipeline. |

## The n8n Workflow

The workflow ([rag-flow.json](rag-flow.json)) has two halves:

**Ingestion path**
- `Upload Webhook` (`POST /webhook/rag-upload`) receives the PDF text
- `Edit Fields` maps `body.documents[0]` to the document text
- `Default Data Loader` + `Embeddings Google Gemini` → `Simple Vector Store` (insert mode, `clearStore: true`)

**Chat path**
- `Telegram Trigger` receives messages
- `AI Agent` (Google Gemini chat model + `Simple Memory` window buffer) answers
- `Simple Vector Store1` is exposed as a `knowledge_base` retrieve-as-tool the agent calls before answering
- `Send a text message` replies back in Telegram

> The agent is instructed to answer **only** from retrieved documents and to reply `"I couldn't find that in the uploaded documents."` when nothing relevant is found.

## Getting Started

### Prerequisites

- A running [n8n](https://n8n.io/) instance (cloud or self-hosted)
- A [Google Gemini / AI Studio](https://aistudio.google.com/) API key (for embeddings + chat)
- A [Telegram bot token](https://core.telegram.org/bots#how-do-i-create-a-bot) from `@BotFather`

### 1. Import the workflow

In n8n: **Workflows → Import from File →** select [rag-flow.json](rag-flow.json).

### 2. Connect credentials

Open the imported nodes and attach your own credentials:
- **Google Gemini** — on the three Gemini nodes (chat model + both embedding nodes)
- **Telegram** — on the Telegram Trigger and Send-message nodes

> The exported JSON references credentials by name only — no secrets are stored in this repo. You must supply your own.

### 3. Activate and grab the webhook URL

Activate the workflow, then copy the production URL of the **Upload Webhook** node (ends in `/webhook/rag-upload`).

### 4. Run the uploader

Open [index.html](index.html) in a browser (or serve it locally):

```bash
python3 -m http.server 8099
# then visit http://localhost:8099/index.html
```

Paste your webhook URL, click **Test**, then drop a PDF and **Send to Vector Store**.

### 5. Chat in Telegram

Message your bot — e.g. *"What is the IT password reset policy?"* — and it will answer from the uploaded document.

## Tech Stack

- **Frontend** — HTML + Tailwind CSS (CDN), [PDF.js](https://mozilla.github.io/pdf.js/) for client-side text extraction, [Lucide](https://lucide.dev/) icons
- **Orchestration** — n8n (LangChain nodes)
- **AI** — Google Gemini (embeddings + chat), in-memory vector store
- **Messaging** — Telegram Bot API

## Notes

- The vector store is **in-memory** and uses `clearStore: true` on insert — each upload replaces the previous document. This keeps the demo simple; swap in a persistent store (Pinecone, Qdrant, Supabase, etc.) for production.
- PDF text is extracted **in the browser** — scanned/image-only PDFs without a text layer won't work.

## Acknowledgements

Built as a training module for [Tertiary Infotech Academy](https://www.tertiaryinfotech.com/).

---

Powered by [Tertiary Infotech Academy Pte Ltd](https://www.tertiaryinfotech.com/)
