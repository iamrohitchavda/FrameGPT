# Setup Guide

> Local development setup for FrameGPT.

---

## Prerequisites

- **Docker** & **Docker Compose** (for PostgreSQL, Redis, Qdrant)
- **Node.js** 18+ (for frontend)
- **Python** 3.11+ (for backend)
- **pnpm** 8+ (`npm i -g pnpm`)
- **yt-dlp** (`brew install yt-dlp` or `pip install yt-dlp`)
- **FFmpeg** (`brew install ffmpeg`)
- **Clerk Account** (for auth)
- **Google AI API Key** (for Gemini)
- **Cloudflare R2 Account** (for storage)
- **Stripe Account** (for billing)

---

## Environment Variables

### Frontend (`apps/web/.env.local`)

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Backend (`apps/api/.env`)

```env
DATABASE_URL=postgresql+asyncpg://framegpt:framegpt@localhost:5432/framegpt
REDIS_URL=redis://localhost:6379/0
QDRANT_URL=http://localhost:6333
CLERK_SECRET_KEY=
GEMINI_API_KEY=
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=framegpt
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
SENTRY_DSN=
POSTHOG_API_KEY=
```

---

## Getting Started

### 1. Start Infrastructure

```bash
docker compose up -d
```

This starts:
- PostgreSQL on `:5432`
- Redis on `:6379`
- Qdrant on `:6333`

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Run Database Migrations

```bash
# Backend
cd apps/api
alembic upgrade head
```

### 4. Start Development Servers

```bash
# From root — starts both frontend and backend
pnpm dev
```

Or individually:

```bash
# Frontend only
pnpm --filter web dev

# Backend only
pnpm --filter api dev
```

### 5. Start Celery Worker

```bash
cd apps/api
celery -A app.tasks.celery_app worker --loglevel=info
```

---

## Docker Compose Services

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: framegpt
      POSTGRES_PASSWORD: framegpt
      POSTGRES_DB: framegpt
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage

volumes:
  pgdata:
  qdrant_data:
```

---

## Useful Commands

```bash
# Database
pnpm db:migrate     # Run migrations
pnpm db:rollback    # Rollback last migration
pnpm db:seed        # Seed test data

# Linting
pnpm lint           # All packages
pnpm format         # Format code

# Testing
pnpm test           # Run all tests
pnpm test:api       # Backend tests
pnpm test:web       # Frontend tests

# Type checking
pnpm typecheck      # All packages
```

---

## Production Checklist

- [ ] Set Clerk production keys
- [ ] Configure Cloudflare R2 bucket (public + private)
- [ ] Set Gemini API key with quota limits
- [ ] Configure Stripe products & prices
- [ ] Set up Sentry project & DSN
- [ ] Configure PostHog project
- [ ] Set Neon PostgreSQL connection string
- [ ] Deploy Qdrant (via Railway or self-hosted)
- [ ] Configure Redis (Upstash or self-hosted)
- [ ] Set Vercel environment variables
- [ ] Set Railway environment variables
- [ ] Configure domain & SSL
- [ ] Set up monitoring alerts
