# 🤖 HiveStaff AI Voice Agent

An intelligent, emotion-aware phone support agent for HiveStaff — a workforce management platform. The system answers inbound and outbound calls, transcribes user speech, retrieves answers from your documentation using RAG, responds in a natural human voice, and logs every conversation with full call recordings in a live browser dashboard.

---

## 📋 Table of Contents

- [Features](#features)
- [Architecture & Workflow](#architecture--workflow)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [API Keys Required](#api-keys-required)
- [Setup & Installation](#setup--installation)
- [Cell-by-Cell Execution Guide](#cell-by-cell-execution-guide)
- [Twilio Configuration](#twilio-configuration)
- [Live Dashboard](#live-dashboard)
- [Project Structure](#project-structure)
- [Configuration Reference](#configuration-reference)
- [Troubleshooting](#troubleshooting)

---

## ✨ Features

- 📞 **Inbound & Outbound Calls** via Twilio
- 🎙️ **Speech-to-Text** using Groq Whisper (`whisper-large-v3-turbo`) — fast & accurate
- 🧠 **RAG (Retrieval-Augmented Generation)** — answers only from your HiveStaff documentation PDF
- 🤖 **LLM Responses** via Groq LLaMA 3.1 8B — short, conversational, human-like
- 🎭 **Emotion Detection** — detects user sentiment and adjusts AI tone accordingly
- 🔊 **Natural TTS** via Groq PlayAI voices — not robotic, natural-sounding
- 📼 **Full Call Recordings** — both AI and user sides captured, stored locally, playable in browser
- 📊 **Live Call Log Dashboard** — real-time browser UI showing transcripts, emotions, recordings
- 🔁 **Conversation Memory** — maintains context across multiple turns per call

---

## 🏗️ Architecture & Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                        FULL CALL FLOW                           │
└─────────────────────────────────────────────────────────────────┘

  📱 Phone Call
       │
       ▼
  ┌─────────┐     webhook      ┌──────────────────────────────┐
  │ Twilio  │ ──────────────► │  Flask Server (ngrok tunnel)  │
  │  Cloud  │ ◄────────────── │  running on Google Colab      │
  └─────────┘    TwiML XML     └──────────────────────────────┘
                                         │
              ┌──────────────────────────┼──────────────────────────┐
              │                          │                           │
              ▼                          ▼                           ▼
       ┌────────────┐          ┌──────────────────┐       ┌──────────────────┐
       │  /voice    │          │ /process_speech  │       │ /recording_status│
       │            │          │                  │       │                  │
       │ 1. Start   │          │ 1. Download      │       │ 1. Download full │
       │    full    │          │    recording     │       │    call .wav     │
       │    call    │          │    from Twilio   │       │ 2. Save locally  │
       │    record  │          │ 2. Transcribe    │       │ 3. Update URL in │
       │ 2. Play    │          │    (Groq Whisper)│       │    dashboard     │
       │    greeting│          │ 3. Detect emotion│       └──────────────────┘
       │ 3. Start   │          │ 4. RAG retrieval │
       │    STT     │          │    (FAISS)       │
       │    record  │          │ 5. LLaMA response│
       └────────────┘          │    (Groq)        │
                               │ 6. TTS audio     │
                               │    (PlayAI)      │
                               │ 7. Play audio    │
                               │ 8. Listen again  │
                               └──────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                         RAG PIPELINE                            │
└─────────────────────────────────────────────────────────────────┘

  hivestaff_docs.pdf
       │
       ▼
  Docling PDF Parser  ──►  Markdown Text
       │
       ▼
  Chunker (400 chars, 80 overlap)
       │
       ▼
  SentenceTransformer Embeddings (all-MiniLM-L6-v2)
       │
       ▼
  FAISS Vector Index
       │
       ▼  (at query time)
  User Query ──► Embed ──► FAISS Search (top 3) ──► Context ──► LLaMA

┌─────────────────────────────────────────────────────────────────┐
│                      EMOTION PIPELINE                           │
└─────────────────────────────────────────────────────────────────┘

  User Speech Text
       │
       ▼
  Keyword Detection
  (angry / sad / happy / excited / fearful / surprised / neutral)
       │
       ▼
  AI Response Emotion Mapping
  (angry → sad tone | happy → happy tone | fearful → calm tone)
       │
       ▼
  System Prompt Adjustment + Voice Selection
  (Aaliyah-PlayAI for soft/calm | Cole-PlayAI for upbeat/neutral)
```

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Platform | Google Colab (T4 GPU recommended) |
| Phone calls | Twilio Programmable Voice |
| Public tunnel | ngrok |
| Web server | Flask |
| Speech-to-Text | Groq `whisper-large-v3-turbo` |
| LLM | Groq `llama-3.1-8b-instant` |
| Text-to-Speech | Groq PlayAI TTS (`playai-tts`) |
| PDF Parsing | Docling + PyMuPDF (fallback) |
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` |
| Vector Search | FAISS (CPU) |
| TTS Fallback | gTTS |

---

## ✅ Prerequisites

- Google account (for Google Colab)
- HiveStaff documentation exported as a **PDF file**
- A phone number that can receive calls

---

## 🔑 API Keys Required

You need to collect the following before running:

### 1. Groq API Key
- Go to [console.groq.com](https://console.groq.com)
- Create account → API Keys → Create new key
- Used for: STT (Whisper), LLM (LLaMA), TTS (PlayAI)

### 2. Twilio Credentials
- Go to [twilio.com/console](https://www.twilio.com/console)
- Get `Account SID` and `Auth Token` from dashboard
- Buy a phone number (or use trial number)
- Used for: Making/receiving calls, recording calls

### 3. ngrok Auth Token
- Go to [dashboard.ngrok.com](https://dashboard.ngrok.com)
- Sign up → Your Authtoken
- Used for: Exposing local Flask server to Twilio's webhooks

---

## 🚀 Setup & Installation

### Step 1 — Open the notebook in Google Colab

Upload `twilio_trial_rag_docling.ipynb` to [colab.research.google.com](https://colab.research.google.com) or open directly from Google Drive.

> **Recommended:** Set runtime to **T4 GPU** for faster Whisper and embedding operations.
> Runtime → Change runtime type → T4 GPU

### Step 2 — Upload your PDF

Upload your HiveStaff documentation PDF to Colab at this exact path:

```
/content/sample_data/hivestaff_docs.pdf
```

To do this in Colab: click the **Files** icon (📁) on the left sidebar → navigate to `sample_data/` → upload your PDF.

### Step 3 — Fill in your API keys

In **Cell 2**, replace the placeholder values:

```python
GROQ_API_KEY        = "your_groq_api_key_here"
TWILIO_ACCOUNT_SID  = "your_twilio_account_sid"
TWILIO_AUTH_TOKEN   = "your_twilio_auth_token"
TWILIO_PHONE_NUMBER = "+1XXXXXXXXXX"   # your Twilio number
NGROK_AUTH_TOKEN    = "your_ngrok_token"
```

Also update your personal phone number in **Cell 9**:

```python
YOUR_PHONE_NUMBER = "+91XXXXXXXXXX"   # number to receive the outbound call
```

---

## 📓 Cell-by-Cell Execution Guide

Run each cell **in order**. Do not skip any cell.

---

### Cell 1 — Install Dependencies
**What it does:** Installs all required Python packages and system tools.

```
flask, pyngrok, twilio, openai-whisper, groq, gtts, pydub,
beautifulsoup4, sentence-transformers, faiss-cpu, ffmpeg, docling, pymupdf
```

⏱️ Takes 2–4 minutes. Run once per Colab session.

---

### Cell 2 — Configuration
**What it does:** Sets all API keys, model names, and tuning parameters. Also initialises the `twilio_client` used across the project.

Key settings:
```python
LLAMA_MODEL       = "llama-3.1-8b-instant"   # LLM
CHUNK_SIZE        = 400                        # RAG chunk size
TOP_K_CHUNKS      = 3                         # chunks retrieved per query
RECORD_MAX_LENGTH = 45                        # max user speech seconds
RECORD_TIMEOUT    = 2                         # silence = end of speech
```

---

### Cell 2.5 — Build RAG Index
**What it does:** Converts your PDF using Docling, splits it into chunks, generates embeddings, and builds the FAISS vector index.

What happens internally:
1. Docling parses `hivestaff_docs.pdf` → clean Markdown
2. Text is split into 400-character chunks with 80-character overlap
3. `all-MiniLM-L6-v2` embeds each chunk into a 384-dim vector
4. FAISS `IndexFlatIP` (inner product) index is built and tested

✅ Success output looks like:
```
🎉 RAG system ready!
   Files   : 1
   Chunks  : 312
   Vectors : 312
```

---

### Cell 3 — Load Whisper Model
**What it does:** Loads the local OpenAI Whisper model (`base`) into memory. This is used only as a local fallback — the primary STT is Groq's cloud Whisper.

---

### Cell 4 — LLM + RAG + Emotion Engine
**What it does:** Defines the full AI response pipeline:

- `detect_emotion(text)` — keyword-based emotion classifier
- `retrieve_context(query)` — FAISS semantic search returning top-3 relevant chunks
- `get_ai_response(session_id, text)` — full pipeline: emotion → RAG → LLaMA → reply

Quick test runs automatically at the end to verify the pipeline works.

---

### Cell 5 — Text-to-Speech
**What it does:** Defines the TTS function using Groq PlayAI voices. Voices are selected based on detected emotion.

| Emotion | Voice |
|---|---|
| neutral, happy, excited | `Cole-PlayAI` |
| sad, calm | `Aaliyah-PlayAI` |

Also **pre-generates the greeting audio** so the call is answered instantly with no delay.

---

### Cell 6.5 — Call Logger
**What it does:** Sets up the in-memory logging system tracking all call events, transcripts, emotions, and recording URLs per session.

Useful functions you can call anytime:
```python
print_all_sessions()              # summary of all calls
print_full_transcript("session_id")  # full log for one call
```

---

### Cell 6 — Flask Server
**What it does:** Defines all HTTP routes for the voice agent and the browser dashboard.

| Route | Purpose |
|---|---|
| `GET/POST /voice` | Entry point for every call — starts recording, plays greeting |
| `POST /process_speech` | Receives recorded audio, transcribes, generates and plays AI reply |
| `POST /recording_status` | Twilio callback when full call recording is ready — downloads & saves locally |
| `GET /audio/<file>` | Serves audio files (AI responses + call recordings) |
| `GET /logs` | Browser dashboard — live transcripts + audio players |
| `GET /status` | JSON health check |

---

### Cell 7 — Start ngrok + Flask
**What it does:** Clears port 5000, starts a fresh ngrok tunnel, launches Flask in a background thread.

Output:
```
╔══════════════════════════════════════════════════╗
║  🚀 Voice Agent is LIVE                          ║
╠══════════════════════════════════════════════════╣
║  Public URL : https://xxxx-xxxx.ngrok-free.app   ║
║  Webhook    : https://xxxx-xxxx.ngrok-free.app/voice ║
╠══════════════════════════════════════════════════╣
║  👉 Paste Webhook URL into Twilio console        ║
╚══════════════════════════════════════════════════╝
```

---

### Cell 9 — Make Outbound Call
**What it does:** Places an outbound call to `YOUR_PHONE_NUMBER`. When you pick up, the AI speaks immediately.

```python
YOUR_PHONE_NUMBER = "+91XXXXXXXXXX"
```

---

### Cell 10 — View Logs
**What it does:** Prints a summary of all call sessions to the Colab console, and prints the browser dashboard URL.

```python
print_all_sessions()
# Optional — full transcript for a specific session:
print_full_transcript("session_id_here")
```

---

## 📡 Twilio Configuration

### For Inbound Calls (people calling your Twilio number)

1. Go to [twilio.com/console](https://www.twilio.com/console) → Phone Numbers → Manage → Active Numbers
2. Click your Twilio number
3. Under **Voice Configuration → A call comes in**, set:
   - **Webhook:** `https://your-ngrok-url/voice`
   - **Method:** `HTTP POST`
4. Click Save

### For Outbound Calls
No Twilio console setup needed — Cell 9 places the call programmatically.

> ⚠️ **Twilio Trial Accounts:** You can only call verified phone numbers. Go to Twilio Console → Verified Caller IDs to add your number.

---

## 📊 Live Dashboard

Once the server is running, open the dashboard in your browser:

```
https://your-ngrok-url/logs
```

The dashboard shows:
- **Session header** — caller number, total turns, recording count, start time
- **Full Call Recording** — single audio player with both AI + user sides (appears after call ends)
- **Conversation table** — timestamped transcript with role and emotion badge per message
- **Refresh button** — manually refresh to see latest data

---

## 📁 Project Structure

```
twilio_trial_rag_docling.ipynb     ← Main notebook (run this)
/content/sample_data/
  └── hivestaff_docs.pdf           ← Your documentation PDF (upload this)
/content/docs_markdown/
  └── doc_01.md                    ← Auto-generated markdown (for inspection)
/tmp/audio_responses/
  ├── <uuid>.wav                   ← AI TTS response audio files
  └── rec_<RecordingSid>.wav       ← Full call recordings (both sides)
```

---

## ⚙️ Configuration Reference

All tunable settings are in **Cell 2**:

| Parameter | Default | Effect |
|---|---|---|
| `LLAMA_MODEL` | `llama-3.1-8b-instant` | LLM used for responses |
| `CHUNK_SIZE` | `400` | Characters per RAG chunk |
| `CHUNK_OVERLAP` | `80` | Overlap between chunks |
| `TOP_K_CHUNKS` | `3` | Chunks retrieved per query (lower = faster) |
| `EMBED_MODEL` | `all-MiniLM-L6-v2` | Embedding model for RAG |
| `RECORD_MAX_LENGTH` | `45` | Max seconds to record user speech |
| `RECORD_TIMEOUT` | `2` | Seconds of silence before recording stops |

In **Cell 4** (LLM settings):

| Parameter | Default | Effect |
|---|---|---|
| `max_tokens` | `80` | Max tokens in AI reply (shorter = faster) |
| `temperature` | `0.4` | Response creativity |

In **Cell 5** (TTS settings):

| Parameter | Default | Effect |
|---|---|---|
| `speed` | `1.1` | Speech speed (1.0 = normal, 1.1 = slightly faster) |

---

## 🔧 Troubleshooting

**Recording shows "processing..." and never loads**
- Make sure Cell 7 runs *before* Cell 9 so `PUBLIC_URL` is set
- The `recording_status_callback` URL must be reachable by Twilio — confirm ngrok is still alive
- Recordings complete 30–60 seconds after the call ends — hit Refresh on the dashboard

**AI response is very slow**
- Switch Colab runtime to T4 GPU
- Reduce `TOP_K_CHUNKS` to `2` in Cell 2
- Reduce `max_tokens` to `60` in Cell 4

**"I didn't catch that" repeating**
- Check Colab console for Groq Whisper errors
- Ensure `RECORD_TIMEOUT` is at least `2` seconds
- Verify the audio file size is above 5000 bytes (printed in console)

**Twilio not reaching webhook**
- Confirm ngrok tunnel is running (Cell 7 output shows URL)
- Check the webhook URL in Twilio console matches current ngrok URL exactly
- ngrok free tier URLs change every restart — always re-paste after Cell 7

**PDF not loading**
- Confirm file is at exactly `/content/sample_data/hivestaff_docs.pdf`
- If Docling gives empty output, PyMuPDF fallback runs automatically
- Check Cell 2.5 console output for chunk count — must be > 0

**Outbound call not connecting**
- Twilio trial accounts require verified numbers — add yours at Twilio → Verified Caller IDs
- Confirm `YOUR_PHONE_NUMBER` in Cell 9 includes country code (e.g. `+91...`)

---

## 📌 Notes

- The ngrok free tier URL changes every time Cell 7 is re-run. Always re-paste the new webhook URL into Twilio console after restarting.
- Call logs and recordings are stored **in memory and in `/tmp/`** — they reset when the Colab runtime restarts.
- The system answers questions **only from your PDF documentation**. If the answer isn't in the PDF, it responds with a polite fallback message.
- Conversation history is kept for the last 16 messages per session to maintain context without hitting token limits.
