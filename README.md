<<<<<<< HEAD
# EdgeAI LearnForge 🚀

EdgeAI LearnForge is an AI-powered educational platform that automatically converts messy, unstructured video transcripts (YouTube or local MP4 uploads) into textbook-quality study materials. It uses advanced NLP, local AI agents, and semantic analysis to extract structured knowledge, generate premium flashcards, create intelligent quizzes, build knowledge graphs, and provide an interactive Q&A tutor — all without requiring a cloud API key.

---

## 🌟 Features

### 📚 Study Materials
- **AI-Powered Study Notes:** Cleans conversational filler, extracts typed knowledge schemas (Definitions, How It Works, Applications, Common Pitfalls, Examples, Formulas), and renders cohesive third-person educational notes per topic.
- **Concept Density Assessment:** Tags topics with difficulty badges (🟢 Light · 🟡 Medium · 🔴 Heavy) so learners know where to focus.
- **Premium Flashcards:** Concept-based, reasoning-heavy flashcards with a 3D CSS flip-card UI. Progress bar and card-type badges (conceptual / application / misconception).
- **Intelligent Quizzes:** Application-focused multiple-choice questions with automatic answer-option randomization (correct answer never predictably at A) and detailed post-answer explanations with a full assessment results view.

### 🔀 Knowledge Graph
- **Structured Educational Hierarchy:** Generates a validated educational hierarchy — not random keyword nodes — by reusing the existing notes knowledge pipeline (definitions, steps, applications, best practices, warnings, formulas).
- **Topic-Type Classification:** Automatically classifies each topic as `algorithm | workflow | procedure | system | concept | general`, assigning distinct accent colors.
- **Content Validation:** Skips non-structural topics (motivational intros, conversational discussions) and shows a clean fallback message instead of a forced graph.
- **Two Visualizations:** Concept Tree (root node → subtopic cards with numbered badges, icons, descriptions, and bullet items) and Flowchart (step-by-step sequential blocks with downward arrows).
- **Right Sidebar:** Explanation panel, Key Takeaways, Examples, and Related Topics tag cloud — all sourced from real extracted content.

### 🎙️ Transcript Processing
- **Hybrid Transcript Extraction:** Fetches captions via the YouTube Transcript API. Falls back to local `Faster-Whisper` transcription for MP4 uploads.
- **Universal Translation:** Supports Hindi/Hinglish transcripts and generates all materials in professional English.
- **Chapter-Aware Segmentation:** Prioritizes YouTube chapter timestamps; falls back to semantic segmentation using SentenceTransformers + FAISS when chapters are absent.

### ⚡ Performance & Reliability
- **Progressive Topic Loading:** First topic loads immediately in the foreground; remaining topics are prefetched asynchronously in the background via `ThreadPoolExecutor`.
- **Atomic Caching:** All generated assets (notes, flashcards, quizzes, graphs) are atomically written to disk to prevent race conditions during concurrent generation. Cached results load instantly on revisit.
- **Zero-LLM Fallback:** Notes, quizzes, flashcards, and Knowledge Graphs all have pure heuristic fallbacks that work without Ollama or Gemini — using TF-IDF, regex classifiers, and sentence co-occurrence.

---

## 🛠️ Technology Stack

### Frontend
| Technology | Purpose |
|---|---|
| React + Vite | SPA framework and build tooling |
| Vanilla CSS + Inline Styles | Dark theme, 3D transforms, glassmorphism |
| Lucide React | Icon library |

### Backend
| Technology | Purpose |
|---|---|
| FastAPI (Python) | REST API server |
| Google Gemini 1.5 Flash | Cloud LLM (optional) |
| Ollama (`llama3.2:1b`) | Local LLM fallback (optional) |
| Heuristic NLP Pipeline | Zero-LLM extraction baseline (always active) |
| Faster-Whisper | Local MP4 audio transcription |
| youtube-transcript-api | YouTube caption fetching |
| FAISS-CPU + Sentence-Transformers | Semantic search and RAG embeddings |
| Python ThreadPoolExecutor | Concurrent asset prefetching |

---

## 🚀 Getting Started

### Prerequisites
- **Node.js** v16 or higher
- **Python** 3.9 or higher
- **Ollama** *(Optional)* — for local LLM-powered notes/quizzes
- **Gemini API Key** *(Optional)* — cloud LLM fallback

> **Note:** The app works fully offline without any API key. The heuristic pipeline handles all content generation.

---

### 1. Install Dependencies

**Frontend:**
```bash
# From the project root
npm install
```

**Backend:**
```bash
# Create and activate a virtual environment
python -m venv venv

# Windows
venv\Scripts\activate

# Mac/Linux
source venv/bin/activate

# Install backend dependencies
pip install fastapi "uvicorn[standard]" pydantic sentence-transformers faiss-cpu numpy \
  requests python-multipart youtube-transcript-api python-dotenv faster-whisper imageio-ffmpeg
```

---

### 2. Environment Variables

Create a `.env` file in the project root:

```env
# Optional — cloud LLM. Leave as placeholder if using Ollama or heuristic mode.
GEMINI_API_KEY=placeholder_key

# Optional — local LLM via Ollama
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=llama3.2:1b
```

---

### 3. Run the Application

You need two terminals running simultaneously.

**Terminal 1 — Backend:**
```bash
cd src/backend
python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```
Backend API: `http://localhost:8000` · Swagger docs: `http://localhost:8000/docs`

**Terminal 2 — Frontend:**
```bash
npm run dev
```
App: `http://localhost:5173`

---

## 🧠 Architecture Overview

```
edgeai_learnforge/
├── src/
│   ├── backend/
│   │   ├── main.py               # FastAPI entry point, routing, prefetch pipeline
│   │   ├── extractor.py          # Core NLP: TF-IDF, knowledge unit extraction, sentence classification
│   │   ├── segmenter.py          # Topic windowing (YouTube chapters → semantic fallback)
│   │   ├── notes_generator.py    # Per-topic notes with atomic caching + heuristic pipeline
│   │   ├── flashcard_generator.py# Concept-based flashcard generation
│   │   ├── quiz_generator.py     # MCQ generation with answer randomization
│   │   ├── graph_extractor.py    # Educational hierarchy graph from knowledge cache
│   │   ├── vector_db.py          # FAISS index + Sentence-Transformers embeddings
│   │   ├── qa_engine.py          # RAG-based Q&A tutor
│   │   ├── transcript_refiner.py # Filler removal + declarative language conversion
│   │   └── translator.py         # Hindi/Hinglish → English translation
│   └── components/
│       ├── TranscriptViewer.jsx  # Main study panel (Notes, Flashcards, Quiz, Knowledge Graph)
│       ├── UploadBox.jsx         # YouTube URL / MP4 upload input
│       └── ...
├── storage/                      # Generated assets per video_id (auto-created)
├── .env                          # Environment variables
├── package.json
└── README.md
```

### Key Pipeline Flow

```
Video URL / MP4
      │
      ▼
Transcript Fetch (YouTube API → Whisper fallback)
      │
      ▼
Topic Segmentation (YouTube Chapters → Semantic FAISS clustering)
      │
      ▼
Foreground: Topic 0 → Knowledge Extraction → Notes
      │
      ▼
Background (ThreadPoolExecutor):
  ├── Notes (topics 1…N)
  ├── Flashcards
  ├── Quiz (with option randomization)
  └── Knowledge Graph (from notes knowledge cache)
```

---

## 📝 License

This project is for educational purposes and personal portfolio development.
=======
# edgeai-learnforge
>>>>>>> 5f43d6d79a5ec5c5678f2f8bcfd9c947aefe6968
