# 🎨 PromptPixel

> **Just describe it. We'll perfect it.**

PromptPixel is a full-stack AI-powered image generation application that intelligently enhances your text prompts before generating high-quality images. It judges your input, searches its failure memory to avoid past mistakes, and uses an LLM to craft a superior prompt — all before a single pixel is painted.

---

## ✨ Features

- **🔍 AI Prompt Judge** — Evaluates your prompt on a 1–10 quality score, classifies it (`scene`, `portrait`, `style`, `abstract`), and gives actionable feedback.
- **🧠 Failure Memory** — Every image you mark as "not good" is stored in a SQLite database with a local vector embedding. Future generations avoid similar mistakes automatically.
- **⚡ Smart Prompt Enhancement** — Uses Groq's `llama-3.1-8b-instant` LLM (with a zero-dependency rule-based fallback) to produce a richer, more detailed prompt.
- **🖼️ Free Image Generation** — Generates images via [Pollinations.ai](https://pollinations.ai) — completely free, no API key required, no rate limits.
- **📊 Failure Memory Dashboard** — Browse the full history of logged failures with scores, feedback, and timestamps.
- **💾 Save or Retry** — Download generated images or send negative feedback to teach PromptPixel to improve.

---

## 🏗️ Architecture

```
pixel/
├── backend/                  # Node.js / Express API server
│   ├── server.js             # Entry point, middleware, routes
│   ├── database.js           # SQLite setup via better-sqlite3
│   ├── faissIndex.js         # In-memory cosine-similarity vector search
│   ├── routes/
│   │   └── api.js            # All API endpoints
│   ├── promptpixel.db        # SQLite database (auto-created)
│   ├── failure_ids.json      # Persisted vector index
│   └── .env                  # Environment variables (not committed)
│
└── frontend/                 # React + Vite + Tailwind CSS app
    └── src/
        ├── App.jsx           # Root component & router
        ├── pages/
        │   ├── Home.jsx      # Prompt input & generation trigger
        │   ├── Enhancement.jsx # Review enhanced prompt, generate image
        │   ├── Result.jsx    # Display image, save or log failure
        │   └── Memory.jsx    # Failure memory dashboard
        └── components/
            ├── Navbar.jsx
            ├── LoadingOverlay.jsx
            └── ScoreBadge.jsx
```

---

## 🔄 How It Works

```
User enters prompt
        │
        ▼
  POST /api/judge          ← Groq LLM scores prompt 1–10 + gives feedback
        │
        ▼
  POST /api/search-failures ← Searches vector index for similar past failures
        │
        ▼
  POST /api/enhance        ← LLM rewrites prompt using judge + failure context
        │
        ▼
  POST /api/generate       ← Pollinations.ai generates the image (free, no key)
        │
        ▼
   User Reviews Image
     /          \
  Save it    "Not Good"
               │
               ▼
     POST /api/log-failure  ← Embeds + stores in SQLite + vector index
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite 8, Tailwind CSS 4 |
| Routing | React Router DOM v7 |
| Notifications | React Hot Toast |
| HTTP Client | Axios |
| Backend | Node.js, Express 5 |
| Database | SQLite (`better-sqlite3`) |
| Vector Search | Custom cosine-similarity index (JSON-persisted) |
| Embeddings | Local TF-IDF-style bag-of-words (128-dim, offline) |
| LLM | Groq API — `llama-3.1-8b-instant` |
| Image Generation | Pollinations.ai (free, no API key) |

---

## 🚀 Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) v18 or later
- A free [Groq API key](https://console.groq.com/) *(optional — rule-based fallback works without one)*

---

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd pixel
```

---

### 2. Set Up the Backend

```bash
cd backend
npm install
```

Copy the example environment file and fill in your keys:

```bash
cp .env.example .env
```

Edit `.env`:

```env
# Optional: Get a free key at https://console.groq.com/
GROQ_API_KEY=your_groq_api_key_here

# Server port (default: 5000)
PORT=5000
```

> **Note:** If `GROQ_API_KEY` is not set, PromptPixel falls back to a built-in rule-based judge and enhancer — no external API needed.

Start the backend:

```bash
npm start
```

The server will start at `http://localhost:5000`. The SQLite database is created automatically on first run.

---

### 3. Set Up the Frontend

```bash
cd ../frontend
npm install
npm run dev
```

The frontend will start at `http://localhost:5173` and proxy API calls to the backend automatically.

---

## 📡 API Reference

All endpoints are prefixed with `/api`.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Health check |
| `POST` | `/judge` | Score and classify a prompt |
| `POST` | `/enhance` | Enhance a prompt using LLM + failure context |
| `POST` | `/generate` | Generate an image via Pollinations.ai |
| `POST` | `/log-failure` | Store a failed prompt in memory |
| `GET` | `/failures` | Fetch all stored failures |
| `POST` | `/search-failures` | Find semantically similar past failures |

### `POST /api/judge`
```json
// Request
{ "prompt": "a cat in space" }

// Response
{ "prompt_type": "scene", "score": 4, "feedback": "Add an art style and lighting details." }
```

### `POST /api/enhance`
```json
// Request
{ "prompt": "a cat in space", "judgeResult": { "score": 4, "feedback": "..." }, "failures": [] }

// Response
{ "enhanced_prompt": "A majestic cat floating in deep space, photorealistic, dramatic volumetric lighting, 8k, ultra-detailed..." }
```

### `POST /api/generate`
```json
// Request
{ "prompt": "enhanced prompt text" }

// Response
{ "image": "<base64-encoded-jpeg>" }
```

### `POST /api/log-failure`
```json
// Request
{
  "original_prompt": "a cat in space",
  "enhanced_prompt": "A majestic cat floating...",
  "prompt_type": "scene",
  "failure_reason": "User rated as bad",
  "judge_score": 4,
  "judge_feedback": "Add an art style..."
}

// Response
{ "success": true, "id": 1 }
```

---

## 🗄️ Database Schema

```sql
CREATE TABLE failure_memory (
  id              INTEGER PRIMARY KEY AUTOINCREMENT,
  original_prompt TEXT,
  enhanced_prompt TEXT,
  prompt_type     TEXT,      -- scene | portrait | style | abstract
  failure_reason  TEXT,
  judge_score     REAL,
  judge_feedback  TEXT,
  embedding       TEXT,      -- JSON array of 128 floats
  created_at      DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🧠 Failure Memory & Vector Search

PromptPixel uses a lightweight, **fully offline** vector search system:

1. **Embedding** — Each prompt is converted to a 128-dimensional vector using a custom TF-IDF-style bag-of-words hash. No external API or model download required.
2. **Storage** — Vectors are stored in `failure_ids.json` and persist across restarts.
3. **Search** — Uses cosine similarity to find the top-3 most similar past failures, which are then passed to the LLM to avoid repeating the same mistakes.

---

## 🌐 Pages

| Route | Page | Description |
|-------|------|-------------|
| `/` | Home | Enter a prompt and kick off the pipeline |
| `/enhance` | Enhancement | Review the original vs. enhanced prompt, then generate |
| `/result` | Result | View the generated image; save or mark as failure |
| `/memory` | Failure Memory | Browse all logged failures with scores and feedback |

---

## ⚙️ Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GROQ_API_KEY` | No | — | Groq API key for LLM-based judging and enhancement |
| `PORT` | No | `5000` | Port the backend server listens on |

---

## 📦 Dependencies

### Backend
| Package | Version | Purpose |
|---------|---------|---------|
| `express` | ^5.2.1 | HTTP server framework |
| `better-sqlite3` | ^12.8.0 | Synchronous SQLite driver |
| `axios` | ^1.13.6 | HTTP client for external APIs |
| `cors` | ^2.8.6 | Cross-origin resource sharing |
| `dotenv` | ^17.3.1 | Environment variable loading |

### Frontend
| Package | Version | Purpose |
|---------|---------|---------|
| `react` | ^19.2.4 | UI library |
| `react-router-dom` | ^7.13.2 | Client-side routing |
| `react-hot-toast` | ^2.6.0 | Toast notifications |
| `axios` | ^1.13.6 | HTTP client |
| `tailwindcss` | ^4.2.2 | Utility-first CSS framework |
| `vite` | ^8.0.1 | Build tool and dev server |

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add your feature"`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the ISC License.

---

<div align="center">
  <p>Built with ❤️ using React, Express, Groq, and Pollinations.ai</p>
</div>
