# Architecture & UML Diagrams

> Visual documentation for the FrameGPT system architecture, data models, and key workflows.

---

## 1. System Architecture (Deployment Diagram)

```mermaid
graph TB
    subgraph Client["Client Layer"]
        WC["Web Client<br/>(Next.js 15)"]
    end

    subgraph CDN["CDN & Auth"]
        V["Vercel<br/>Frontend Hosting"]
        CL["Clerk<br/>Authentication"]
    end

    subgraph API["API Layer"]
        FA["FastAPI<br/>Backend"]
        CE["Celery Worker<br/>Background Tasks"]
    end

    subgraph Infra["Infrastructure"]
        PG[("PostgreSQL<br/>(Neon)")]
        RD[("Redis")]
        QD[("Qdrant<br/>Vector DB)")]
        R2[("Cloudflare R2<br/>Storage")]
    end

    subgraph AI["AI Layer"]
        GV["Gemini 2.5 Flash<br/>Vision Model"]
        GE["gemini-embedding-001<br/>Embeddings"]
    end

    subgraph Ext["External"]
        YT["YouTube"]
        ST["Stripe<br/>Billing"]
        SN["Sentry<br/>Monitoring"]
        PH["PostHog<br/>Analytics"]
    end

    WC --> V
    V --> FA
    WC --> CL
    FA --> CL
    FA --> PG
    FA --> RD
    FA --> QD
    FA --> R2
    FA --> GV
    FA --> GE
    FA --> ST
    FA --> SN
    FA --> PH
    CE --> RD
    CE --> R2
    CE --> YT
    CE --> GV
    CE --> GE
    CE --> QD
```

---

## 2. Entity Relationship Diagram (Database Models)

```mermaid
erDiagram
    users {
        uuid id PK
        string clerk_id UK
        string email
        string name
        string avatar_url
        enum plan "free | pro | enterprise"
        timestamp created_at
        timestamp updated_at
    }

    videos {
        uuid id PK
        uuid user_id FK
        string youtube_url
        string title
        string duration
        enum status "queued | downloading | extracting | indexing | ready | failed"
        string thumbnail_url
        string r2_video_key
        int frame_count
        timestamp created_at
        timestamp updated_at
    }

    frames {
        uuid id PK
        uuid video_id FK
        int frame_number
        float timestamp_seconds
        string r2_image_key
        text description
        vector embedding "3072d"
        timestamp created_at
    }

    chats {
        uuid id PK
        uuid video_id FK
        uuid user_id FK
        timestamp created_at
        timestamp updated_at
    }

    messages {
        uuid id PK
        uuid chat_id FK
        enum role "user | assistant"
        text content
        jsonb frame_references "[]"
        timestamp created_at
    }

    subscriptions {
        uuid id PK
        uuid user_id FK
        string stripe_customer_id
        string stripe_subscription_id
        enum plan "free | pro | enterprise"
        enum status "active | canceled | past_due"
        timestamp current_period_start
        timestamp current_period_end
        timestamp created_at
    }

    usage_records {
        uuid id PK
        uuid user_id FK
        enum metric "video_count | query_count"
        int count
        date date
    }

    users ||--o{ videos : "owns"
    users ||--o{ chats : "has"
    users ||--o{ subscriptions : "has"
    users ||--o{ usage_records : "tracks"
    videos ||--o{ frames : "contains"
    videos ||--o{ chats : "has"
    chats ||--o{ messages : "contains"
```

---

## 3. Use Case Diagram

```mermaid
graph TB
    subgraph Actors
        U["User"]
        A["Admin"]
    end

    subgraph System["FrameGPT System"]
        UC1["Register / Login"]
        UC2["Submit YouTube URL"]
        UC3["Track Video Processing"]
        UC4["Chat with Video"]
        UC5["View Frame References"]
        UC6["Generate Notes"]
        UC7["Generate Quiz"]
        UC8["Manage Subscription"]
        UC9["View Usage Limits"]
        UC10["Manage Billing"]
    end

    U --> UC1
    U --> UC2
    U --> UC3
    U --> UC4
    U --> UC5
    U --> UC6
    U --> UC7
    U --> UC8
    U --> UC9
    U --> UC10
    A --> UC3
    A --> UC8
```

---

## 4. Video Ingestion Sequence

```mermaid
sequenceDiagram
    actor User
    participant Web as Web App
    participant API as FastAPI
    participant Celery as Celery Worker
    participant YT as YouTube
    participant R2 as Cloudflare R2
    participant GV as Gemini Vision
    participant GE as Gemini Embeddings
    participant QD as Qdrant

    User->>Web: Paste YouTube URL
    Web->>API: POST /videos
    API->>API: Validate URL
    API->>API: Create video record (status: queued)
    API-->>Web: Return video ID

    API->>Celery: Enqueue process_video task
    Celery->>Celery: Update status → downloading
    Celery->>YT: yt-dlp download
    YT-->>Celery: Video file
    Celery->>R2: Upload video
    Celery->>Celery: Update status → extracting

    Celery->>Celery: OpenCV extract frames (1/30s)
    Celery->>R2: Upload frames
    Celery->>Celery: Update status → indexing

    loop Each Frame
        Celery->>GV: Describe frame contents
        GV-->>Celery: Text description
        Celery->>GE: Generate embedding
        GE-->>Celery: Vector (3072d)
        Celery->>QD: Store vector + metadata
    end

    Celery->>Celery: Update status → ready
    API-->>Web: WebSocket notification
    Web-->>User: Video ready for chat
```

---

## 5. Chat Query Sequence

```mermaid
sequenceDiagram
    actor User
    participant Web as Web App
    participant API as FastAPI
    participant GE as Gemini Embeddings
    participant QD as Qdrant
    participant GV as Gemini Vision

    User->>Web: Ask question about video
    Web->>API: POST /chats/{id}/messages
    API->>GE: Embed user question
    GE-->>API: Query vector (3072d)
    API->>QD: Search top 3 frames
    QD-->>API: Frame IDs + descriptions + timestamps

    API->>API: Load frame images from R2
    Note over API: Construct prompt:<br/>Frame images + descriptions<br/>+ User question

    API->>GV: Generate answer with context
    GV-->>API: Streaming response

    API-->>Web: SSE stream answer
    Web-->>User: Display answer + frame references

    API->>API: Save message to chat history
    API->>API: Increment usage counter
```

---

## 6. Component Diagram

```mermaid
graph TB
    subgraph Frontend["Frontend - Next.js 15"]
        TC["Tailwind CSS<br/>shadcn/ui"]
        RTK["Redux Toolkit<br/>State Management"]
        CH["Chat Interface"]
        VD["Video Dashboard"]
        ST["Settings"]
        PR["Pricing"]
    end

    subgraph Backend["Backend - FastAPI"]
        R["Routers"]
        S["Services Layer"]
        M["Models"]
        T["Tasks<br/>(Celery)"]
    end

    subgraph Auth["Auth"]
        CK["Clerk SDK"]
        MW["Auth Middleware"]
    end

    subgraph Queue["Queue"]
        RD["Redis Broker"]
        CW["Celery Workers"]
    end

    subgraph AI["AI Services"]
        VS["Vision Service<br/>Gemini 2.5 Flash"]
        ES["Embedding Service<br/>gemini-embedding-001"]
    end

    subgraph Storage["Data Storage"]
        PSQL[("PostgreSQL")]
        QDR[("Qdrant")]
        CR2[("Cloudflare R2")]
    end

    Frontend --> CK
    Frontend --> R
    R --> MW
    MW --> CK
    R --> S
    S --> PSQL
    S --> QDR
    S --> CR2
    S --> VS
    S --> ES
    S --> T
    T --> RD
    RD --> CW
    CW --> VS
    CW --> ES
    CW --> QDR
    CW --> CR2
```

---

## 7. State Machine (Video Processing)

```mermaid
stateDiagram-v2
    [*] --> queued: URL submitted
    queued --> downloading: Worker picks up task
    downloading --> extracting: Video saved to R2
    extracting --> indexing: Frames saved to R2
    indexing --> ready: All vectors stored
    ready --> [*]: Available for chat

    queued --> failed: Invalid URL
    downloading --> failed: Download error
    extracting --> failed: FFmpeg error
    indexing --> failed: AI service error

    failed --> queued: Retry
```

---

## 8. Data Flow Diagram

```mermaid
flowchart LR
    subgraph Input["Input Flow"]
        URL["YouTube URL"]
        Q["User Question"]
    end

    subgraph Proc["Processing Pipeline"]
        DL["Download<br/>yt-dlp"]
        FE["Frame Extract<br/>OpenCV"]
        VD["Visual Desc.<br/>Gemini Vision"]
        EMB["Embeddings<br/>gemini-embedding-001"]
    end

    subgraph Store["Storage Layer"]
        R2["Cloudflare R2"]
        PSQL[("PostgreSQL<br/>Metadata")]
        QDR[("Qdrant<br/>Vectors")]
    end

    subgraph Query["Query Flow"]
        QEMB["Embed Query"]
        SRCH["Qdrant Search"]
        LOAD["Load Frames"]
        GEN["Generate Answer"]
    end

    URL --> DL
    DL --> FE
    FE --> VD
    VD --> EMB
    EMB --> QDR
    FE --> R2
    VD --> PSQL

    Q --> QEMB
    QEMB --> SRCH
    SRCH --> LOAD
    LOAD --> GEN
    SRCH --> QDR
    LOAD --> R2
```

---

## 9. Directory Structure

```
framegpt/
├── apps/
│   ├── web/                         # Next.js 15 Frontend
│   │   ├── app/
│   │   │   ├── (auth)/              # Authentication pages
│   │   │   │   ├── login/
│   │   │   │   ├── register/
│   │   │   │   └── layout.tsx
│   │   │   ├── (dashboard)/         # Protected routes
│   │   │   │   ├── dashboard/
│   │   │   │   ├── videos/
│   │   │   │   │   └── [id]/
│   │   │   │   │       ├── page.tsx         # Video details
│   │   │   │   │       └── chat/
│   │   │   │   │           └── page.tsx     # Chat interface
│   │   │   │   ├── settings/
│   │   │   │   └── layout.tsx
│   │   │   ├── pricing/
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx             # Landing page
│   │   ├── components/
│   │   │   ├── ui/                  # shadcn/ui components
│   │   │   ├── landing/
│   │   │   ├── dashboard/
│   │   │   ├── video/
│   │   │   └── chat/
│   │   ├── lib/
│   │   │   ├── store/               # Redux Toolkit store
│   │   │   ├── api/                 # API client
│   │   │   └── utils/
│   │   ├── middleware.ts            # Clerk auth middleware
│   │   ├── next.config.js
│   │   ├── tailwind.config.ts
│   │   └── package.json
│   │
│   └── api/                         # FastAPI Backend
│       ├── app/
│       │   ├── main.py
│       │   ├── config.py
│       │   ├── database.py
│       │   ├── routers/
│       │   │   ├── auth.py
│       │   │   ├── videos.py
│       │   │   ├── chats.py
│       │   │   ├── subscriptions.py
│       │   │   └── webhooks.py
│       │   ├── models/
│       │   │   ├── user.py
│       │   │   ├── video.py
│       │   │   ├── frame.py
│       │   │   ├── chat.py
│       │   │   └── subscription.py
│       │   ├── services/
│       │   │   ├── video_processor.py
│       │   │   ├── frame_extractor.py
│       │   │   ├── vision_service.py
│       │   │   ├── embedding_service.py
│       │   │   ├── vector_service.py
│       │   │   ├── storage_service.py
│       │   │   └── billing_service.py
│       │   ├── tasks/
│       │   │   ├── celery_app.py
│       │   │   └── process_video.py
│       │   └── schemas/
│       │       ├── video.py
│       │       ├── chat.py
│       │       └── user.py
│       ├── alembic/
│       │   └── versions/
│       ├── requirements.txt
│       └── Dockerfile
│
├── packages/
│   └── shared-types/                # Shared type definitions
│       ├── src/
│       │   ├── index.ts
│       │   ├── video.ts
│       │   ├── chat.ts
│       │   └── user.ts
│       ├── package.json
│       └── tsconfig.json
│
├── docs/
│   ├── ARCHITECTURE.md              # This file
│   ├── API.md                       # API specification
│   └── SETUP.md                     # Setup guide
│
├── .github/
│   └── workflows/
│       ├── ci.yml                   # CI pipeline
│       └── deploy.yml               # Deploy pipeline
│
├── docker-compose.yml
├── turbo.json
├── package.json                     # Root (Turborepo)
├── pnpm-workspace.yaml
└── README.md
```
