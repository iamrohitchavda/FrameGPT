# API Specification

> FrameGPT REST API — Base URL: `https://api.framegpt.ai/v1`

---

## Authentication

All API requests require a `Authorization: Bearer <token>` header. Tokens are obtained via Clerk.

```
Authorization: Bearer sk-framegpt-xxxxxxxxxxxx
```

---

## Endpoints

### Videos

#### Submit Video

```http
POST /videos
Content-Type: application/json

{
  "youtube_url": "https://youtube.com/watch?v=dQw4w9WgXcQ"
}
```

**Response** `201 Created`

```json
{
  "id": "uuid",
  "youtube_url": "https://youtube.com/watch?v=dQw4w9WgXcQ",
  "status": "queued",
  "created_at": "2026-06-08T00:00:00Z"
}
```

#### List Videos

```http
GET /videos?page=1&per_page=20
```

**Response** `200 OK`

```json
{
  "items": [
    {
      "id": "uuid",
      "title": "Video Title",
      "duration": "12:34",
      "status": "ready",
      "thumbnail_url": "https://r2.framegpt.ai/thumbnails/uuid.jpg",
      "frame_count": 25,
      "created_at": "2026-06-08T00:00:00Z"
    }
  ],
  "total": 42,
  "page": 1,
  "per_page": 20
}
```

#### Get Video

```http
GET /videos/:id
```

**Response** `200 OK`

```json
{
  "id": "uuid",
  "youtube_url": "https://youtube.com/watch?v=dQw4w9WgXcQ",
  "title": "Video Title",
  "duration": "12:34",
  "status": "ready",
  "thumbnail_url": "https://r2.framegpt.ai/thumbnails/uuid.jpg",
  "frame_count": 25,
  "frame_interval": 30,
  "r2_video_key": "videos/uuid.mp4",
  "created_at": "2026-06-08T00:00:00Z",
  "updated_at": "2026-06-08T00:05:00Z"
}
```

#### Delete Video

```http
DELETE /videos/:id
```

**Response** `204 No Content`

---

### Chat

#### Create Chat Session

```http
POST /videos/:video_id/chats
```

**Response** `201 Created`

```json
{
  "id": "uuid",
  "video_id": "uuid",
  "created_at": "2026-06-08T00:00:00Z"
}
```

#### Send Message (Streaming)

```http
POST /chats/:id/messages
Content-Type: application/json

{
  "content": "What does the diagram on slide 5 show?"
}
```

**Response** `200 OK` — Server-Sent Events

```
event: token
data: "The diagram shows..."

event: token
data: " a comparison of..."

event: complete
data: {
  "message_id": "uuid",
  "content": "The diagram shows a comparison of...",
  "frame_references": [
    {
      "frame_number": 5,
      "timestamp": "02:30",
      "description": "Slide showing architecture diagram",
      "image_url": "https://r2.framegpt.ai/frames/uuid/0005.jpg"
    },
    {
      "frame_number": 6,
      "timestamp": "03:00",
      "description": "Detailed component breakdown",
      "image_url": "https://r2.framegpt.ai/frames/uuid/0006.jpg"
    }
  ]
}
```

#### Get Messages

```http
GET /chats/:id/messages
```

**Response** `200 OK`

```json
{
  "items": [
    {
      "id": "uuid",
      "role": "user",
      "content": "What does the diagram on slide 5 show?",
      "created_at": "2026-06-08T00:10:00Z"
    },
    {
      "id": "uuid",
      "role": "assistant",
      "content": "The diagram shows a comparison of...",
      "frame_references": [...],
      "created_at": "2026-06-08T00:10:05Z"
    }
  ]
}
```

#### List Chats

```http
GET /videos/:video_id/chats
```

**Response** `200 OK`

```json
{
  "items": [
    {
      "id": "uuid",
      "message_count": 12,
      "created_at": "2026-06-08T00:00:00Z"
    }
  ]
}
```

---

### User & Subscription

#### Get Current User

```http
GET /users/me
```

**Response** `200 OK`

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "name": "John Doe",
  "plan": "pro",
  "usage": {
    "videos_used": 12,
    "videos_limit": 50,
    "queries_used": 245,
    "queries_limit": 1000
  }
}
```

#### Get Subscription

```http
GET /subscriptions
```

**Response** `200 OK`

```json
{
  "plan": "pro",
  "status": "active",
  "current_period_end": "2026-07-08T00:00:00Z",
  "cancel_at_period_end": false
}
```

---

### Webhooks

#### Clerk Webhook

```http
POST /webhooks/clerk
```

Syncs user data on `user.created`, `user.updated`, `user.deleted`.

#### Stripe Webhook

```http
POST /webhooks/stripe
```

Handles `checkout.session.completed`, `invoice.paid`, `customer.subscription.updated`, `customer.subscription.deleted`.

---

## Rate Limiting

| Plan       | Rate Limit              |
|-----------|-------------------------|
| Free      | 10 req/min              |
| Pro       | 60 req/min              |
| Enterprise| Custom                  |

---

## Error Codes

| Status | Code                | Description                    |
|--------|---------------------|--------------------------------|
| 400    | invalid_url         | Invalid YouTube URL            |
| 400    | invalid_input       | Invalid request body           |
| 401    | unauthorized        | Missing or invalid token       |
| 403    | quota_exceeded      | Plan limit reached             |
| 404    | not_found           | Resource not found             |
| 409    | already_processing  | Video already being processed  |
| 429    | rate_limited        | Too many requests              |
| 500    | internal_error      | Server error                   |
| 502    | ai_service_down     | AI service unavailable         |
