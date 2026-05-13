# 🐝 HiveStaff AI Chatbot

An AI-powered sales assistant for **HiveStaff ERP** built on Google Colab. It talks to potential customers, collects and validates their contact details, and saves qualified leads to a Neon PostgreSQL database.

---

## 🛠️ Tech Stack

| Component | Tool |
|---|---|
| LLM | Ollama — `cogito` model |
| Embeddings | `all-MiniLM-L6-v2` (sentence-transformers) |
| Vector Search | FAISS (IndexFlatIP) |
| PDF Parsing | Docling + PyMuPDF (fallback) |
| Database | Neon PostgreSQL (serverless) |
| Phone Validation | `phonenumbers` library |
| Email Validation | Regex + `dns.resolver` |
| Runtime | Google Colab (T4 GPU) |

---

## 📋 Prerequisites

- Google Colab with **T4 GPU** runtime
- `hivestaff_docs.pdf` uploaded to `/content/sample_data/`
- Neon PostgreSQL connection string

---

## 🚀 How to Run

Run the 5 cells **in order**. Do not skip any cell.

---

### Cell 1 — Install Dependencies

```bash
!apt-get install -y zstd
!curl -fsSL https://ollama.com/install.sh | sh
!pip install -q ollama sentence-transformers faiss-cpu numpy docling pymupdf psycopg2-binary
```

Installs the Ollama binary and all required Python packages.

> ⚠️ Also install `phonenumbers` and `dnspython` which are used in Cell 3:
> ```bash
> !pip install -q phonenumbers dnspython
> ```

**Expected output:**
```
>>> Install complete. Run "ollama" from the command line.
```

---

### Cell 2 — Start Ollama & Pull Model

Starts the Ollama server in the background, waits 5 seconds, then downloads the `cogito` model.

**Expected output:**
```
✅ Ollama server started
✅ cogito ready
```

> ⏱ Model download takes 5–10 minutes on first run (~4 GB).

---

### Cell 3 — Build RAG Index

Processes the knowledge base and builds the vector search index.

**Steps it performs:**

1. Converts `hivestaff_docs.pdf` to Markdown using **Docling**
2. Falls back to **PyMuPDF** if Docling output is empty
3. Saves Markdown to `/content/docs_markdown/`
4. Splits text into chunks (`size=400`, `overlap=80`)
5. Embeds all chunks using `all-MiniLM-L6-v2`
6. Builds a **FAISS flat inner-product** index
7. Runs a test query to verify the index works

**Expected output:**
```
📄 Converting 1 files with Docling...
   ✅ Docling: 59,004 chars

💾 Markdown saved to /content/docs_markdown/
📐 Chunking 1 documents...
   hivestaff_docs.pdf: 185 chunks

🎉 RAG system ready!
   Files   : 1
   Chunks  : 185
   Vectors : 185
```

> ❌ If you see `File not found`, make sure `hivestaff_docs.pdf` is at `/content/sample_data/hivestaff_docs.pdf`.

---

### Cell 4 — Connect to Neon PostgreSQL

Connects to the database and sets up the `leads` table.

- Creates the table if it does not exist
- Safely adds `UNIQUE` constraint on `phone` if missing
- Safely adds `demo_datetime` column if missing
- Prints current lead count

**Expected output:**
```
✅ Connected to Neon PostgreSQL!
✅ leads table ready — 3 existing leads in DB
```

**Table schema:**
```sql
CREATE TABLE leads (
    id            SERIAL PRIMARY KEY,
    timestamp     TIMESTAMPTZ DEFAULT NOW(),
    score         INT,
    reason        TEXT,
    name          TEXT,
    company       TEXT,
    email         TEXT UNIQUE,
    phone         TEXT UNIQUE,
    demo_datetime TEXT
);
```

> ❌ If connection fails, verify `NEON_URL` at the top of Cell 4 is correct.

---

### Cell 5 — Run the Chatbot

Starts the interactive chat session.

```
=======================================================
  🐝 HiveStaff AI Assistant  |  type 'exit' to quit
=======================================================
```

Type your messages at the `You:` prompt. Type `exit` to end the session.

---

## 🔄 How a Chat Turn Works

```
You type a message
        │
        ├─ Extract email / phone / demo datetime from your text
        │
        ├─ Validate email
        │     format check (regex)
        │       → DNS domain check (MX / A record)
        │         → duplicate check in DB
        │
        ├─ Validate phone
        │     digit length (7–15)
        │       → fake pattern check (e.g. 9999999999)
        │         → phonenumbers library check
        │           → duplicate check in DB
        │
        ├─ If invalid or duplicate → AI explains error, asks to re-enter
        │
        └─ If valid → RAG retrieves top-2 chunks from FAISS
                        → AI replies grounded in retrieved context
                          → doc link appended if keyword matched
```

**When you type `exit`:**
```
Full conversation sent to cogito for sentiment analysis
        │
        └─ Returns: { interested, score 0–10, reason, name, company, email, phone, demo_datetime }
                        │
                        ├─ score >= 6 → lead saved to Neon PostgreSQL ✅
                        └─ score < 6  → lead skipped ❌
```

---

## 📊 Lead Scoring

| Score | Meaning |
|---|---|
| 0–2 | Just browsing, one-word replies |
| 3–4 | Generic questions, no real interest |
| 5 | Mildly curious, no demo intent |
| 6–7 | Clear interest, specific questions |
| 8 | Provided contact details or confirmed demo |
| 9 | All contact details AND confirmed demo time |
| 10 | All details + deep engagement + demo scheduled |

`interested = true` only when `score >= 6`. Only interested leads are saved to the database.

---

## 🗺️ Documentation Link Mapping

When a user asks about a module, the bot appends the relevant documentation URL automatically:

| User mentions | Link |
|---|---|
| login, dashboard, password, role, permission | `/getting-started` |
| hr, employee, attendance, leave, payroll, salary, loan, performance | `/modules/human-resource` |
| recruitment, hiring, candidate, interview, job | `/modules/recruitment` |
| letter, template | `/modules/letter-generate` |
| asset, inventory | `/modules/assets` |
| notice, announcement | `/modules/notice-board` |
| reception, visitor | `/modules/reception` |
| lead, crm, client | `/modules/leads` |
| contractor, vendor | `/modules/contractor` |
| construction, project | `/modules/work-construction` |
| purchase, procurement, quotation | `/modules/purchase` |
| material, stock | `/modules/materials` |
| warehouse | `/modules/warehouse` |
| company, branch | `/modules/company` |

Base URL: `https://document.hivestaff.in`

---

## ⚠️ Common Issues

| Problem | Fix |
|---|---|
| `File not found: hivestaff_docs.pdf` | Upload the PDF to `/content/sample_data/` in the Colab file panel |
| `❌ Connection failed` in Cell 4 | Check `NEON_URL` in Cell 4; verify your Neon project is not paused |
| `ModuleNotFoundError: phonenumbers` | Run `!pip install phonenumbers dnspython` before Cell 3 |
| Empty greeting from Hive | Re-run Cell 2; increase `time.sleep(5)` to `time.sleep(10)` |
| `⚠️ Could not parse sentiment` | The `cogito` model returned malformed JSON; re-run the chat session |
| DNS check skipped warning | Normal in Colab — email format check still runs, DNS is best-effort |

---

## 🔐 Before Sharing

Replace your live database credentials in Cell 4:

```python
NEON_URL = "postgresql://YOUR_USER:YOUR_PASSWORD@YOUR_HOST/YOUR_DB?sslmode=require"
```
