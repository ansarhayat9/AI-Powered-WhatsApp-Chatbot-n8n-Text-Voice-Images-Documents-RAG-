# AI-Powered WhatsApp Chatbot (Text, Voice, Images, Documents + RAG)

An [n8n](https://n8n.io) workflow that turns a WhatsApp Business number into an AI assistant. It understands text, voice notes, images and documents, remembers the conversation, and answers questions from your own knowledge base using RAG (Retrieval-Augmented Generation).

## Features

- **Text messages**: answered by an AI agent with conversation memory.
- **Voice notes**: downloaded, transcribed (translated to English) with OpenAI, then answered.
- **Images**: downloaded and described with OpenAI vision, then used as context for the answer.
- **Documents**: PDF, XLS and XLSX files are read and used as context.
- **RAG knowledge base**: documents from Google Docs are chunked, embedded and stored in MongoDB Atlas Vector Search. The agent searches it when answering.
- **Unsupported files**: the user gets a polite "not supported" reply.

## How it works

```
WhatsApp message
      |
Route by Message Type
  |-- text  -------------------> Build Text Prompt ----------\
  |-- audio --> Download --> Translate Voice Note -----------+--> RAG Chat Agent --> Send WhatsApp Reply
  |-- image --> Download --> Describe Image --> Build Image Prompt
  |-- document --> Download --> Route by File Type --> Extract text --> Build Document Prompt
                                        \--> Reply Unsupported File

Indexing (run manually):
Start Indexing --> Fetch Google Doc --> Document Loader + Text Splitter --> Embeddings --> Index Into MongoDB
```

The **RAG Chat Agent** uses three sub-nodes: a GPT chat model, a memory buffer (keyed by the sender's WhatsApp ID) and a MongoDB vector search tool (**Knowledge Base Search**).

## Requirements

| Service | What you need |
|---|---|
| n8n | Recent version (cloud or self-hosted). The AI Agent node is set to version 3.1. |
| WhatsApp | Meta developer app + WhatsApp Business Cloud API (a free test number is available). |
| OpenAI | An **API key** (a ChatGPT subscription does not include one). |
| MongoDB Atlas | A cluster with a collection and a vector search index. |
| Google Docs | Google credentials in n8n and a Google Doc with your knowledge base content. |

## Setup

1. **Import** `AI-powered_WhatsApp_chatbot_for_text__voice__images__and_PDF_with_RAG_updated.json` in n8n (Workflows > Import from file).
2. **Create credentials** and assign them to the nodes showing a red warning triangle:
   - OpenAI API: chat model, voice, image and both embeddings nodes.
   - WhatsApp (trigger and API): trigger and send nodes.
   - **Header Auth** for the three download nodes: name `Authorization`, value `Bearer <your WhatsApp access token>`.
   - MongoDB and Google Docs.
3. **WhatsApp node settings**: put your own **Phone Number ID** in **Send WhatsApp Reply** and **Reply Unsupported File**.
4. **MongoDB**: select your collection in **Knowledge Base Search** and **Index Into MongoDB**, and create an Atlas Vector Search index named `data_index`. Example definition (1536 dimensions, matches OpenAI `text-embedding-3-small` / `ada-002`):

   ```json
   {
     "fields": [
       { "type": "vector", "path": "embedding", "numDimensions": 1536, "similarity": "cosine" }
     ]
   }
   ```

   Use the same embedding model for indexing and searching.
5. **Knowledge base**: set your own Google Doc URL in **Fetch Google Doc**, then run the indexing branch once with **Start Indexing (Manual)**.
6. **Activate** the workflow and register the n8n webhook URL in your Meta app's WhatsApp configuration.

## Known limitations

- Only **PDF, XLS and XLSX** files are text-extracted. CSV, TXT, HTML, RTF, XML, ICS and JSON files are routed through but their content is not extracted.
- The voice node uses OpenAI's `whisper-1` model with the **translate** operation, so voice notes are converted to English. OpenAI has announced `whisper-1` will be shut down on 26 Feb 2027, so the node may need updating before then.
- Chat memory uses n8n's in-process Simple Memory, which is cleared when n8n restarts.
- Test the workflow with your own credentials before using it in production.

<img width="704" height="315" alt="image" src="https://github.com/user-attachments/assets/21139acb-a591-447e-b1d2-46a1c9a8fec5" />
<img width="841" height="405" alt="image" src="https://github.com/user-attachments/assets/917f9cbb-9ea0-4ac5-b7ba-62b9f3415e8f" />

