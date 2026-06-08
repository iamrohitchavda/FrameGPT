# FrameGPT

> AI That Actually Sees YouTube — Chat with any YouTube video using visual understanding.

FrameGPT is a Visual Video RAG platform that understands **what is shown inside videos**, not just what is spoken. Users can paste a YouTube URL, let AI process the video visually, and then ask questions about diagrams, code, charts, whiteboards, slides, and UI screens.

---

## Features

- **Visual Video Understanding** — Extracts and understands visual content from YouTube videos
- **Smart Chat** — Ask questions and get answers backed by actual frame references
- **Auto Notes & Summaries** — Generate comprehensive notes, chapter summaries, and quizzes
- **Frame References** — Every answer includes timestamps and frames used
- **Team Workspaces** — Shared knowledge base for teams (v2)
- **API Access** — Public API for integrating FrameGPT into your workflow (v2)

---

## Architecture

```
YouTube URL
    ↓
yt-dlp Download
    ↓
OpenCV Extract Frames
    ↓
Gemini Vision Describe Frames
    ↓
Gemini Embeddings
    ↓
Qdrant Vector Storage

User Question
    ↓
Gemini Embedding
    ↓
Qdrant Search
    ↓
Load Matching Frames
    ↓
Gemini Vision
    ↓
Answer
```

---

## Tech Stack

| Layer        | Technology                                      |
|-------------|-------------------------------------------------|
| Frontend    | Next.js 15, TypeScript, Tailwind CSS, shadcn/ui, Redux Toolkit |
| Auth        | Clerk                                           |
| Backend     | FastAPI (Python)                                |
| Background  | Celery + Redis                                  |
| Database    | PostgreSQL (Neon)                               |
| Vector DB   | Qdrant                                          |
| Storage     | Cloudflare R2                                   |
| Vision AI   | Gemini 2.5 Flash                                |
| Embeddings  | gemini-embedding-001 (3072d)                   |
| Billing     | Stripe                                          |
| Monitoring  | Sentry, PostHog                                 |
| Deployment  | Vercel (frontend), Railway (backend)            |

---

## Quick Start

### Prerequisites

- Docker & Docker Compose
- Node.js 18+
- Python 3.11+
- pnpm (for Turborepo)

### Run Locally

```bash
# Clone the repo
git clone https://github.com/your-org/framegpt.git
cd framegpt

# Start infrastructure (PostgreSQL, Redis, Qdrant)
docker compose up -d

# Install dependencies
pnpm install

# Run database migrations
pnpm db:migrate

# Start development servers
pnpm dev
```

The monorepo uses **Turborepo** and is structured as:

```
framegpt/
├── apps/
│   ├── web/              # Next.js frontend
│   └── api/              # FastAPI backend
├── packages/
│   └── shared-types/     # Shared TypeScript/Pydantic types
├── docs/                 # Documentation & UML diagrams
└── docker-compose.yml    # Local infrastructure
```

---

## Project Status

| Phase | Status |
|-------|--------|
| 1 — Foundation (Monorepo, Docker, Infra) | ✅ Complete |
| 2 — Authentication (Clerk, Users)        | 🔜 In Progress |
| 3 — Video Management                     | 🔜 |
| 4 — Video Ingestion Engine               | 🔜 |
| 5 — Chat Engine                          | 🔜 |
| 6 — Dashboard UI                         | 🔜 |
| 7 — Payments (Stripe)                    | 🔜 |
| 8 — Production Deployment                | 🔜 |

---

## Documentation

- [Architecture & UML Diagrams](docs/ARCHITECTURE.md)
- [API Specification](docs/API.md)
- [Setup Guide](docs/SETUP.md)

---

## Roadmap

| Week | Focus |
|------|-------|
| 1    | Monorepo, Docker, Database, Redis, Qdrant |
| 2    | Clerk Auth, Dashboard, User Management |
| 3    | Video Upload, Video Tracking |
| 4    | yt-dlp, Frame Extraction, Cloudflare R2 |
| 5    | Gemini Vision, Embeddings, Qdrant Indexing |
| 6    | Chat Engine, Streaming Responses |
| 7    | Stripe, Limits, Billing |
| 8    | Deployment, Monitoring, Production Launch |

---

## License

MIT
