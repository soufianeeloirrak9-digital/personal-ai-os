# API Reference

## Overview

This API reference documents the key endpoints used by OpenWork and the orchestration layer.

---

## Base URL

```text
http://localhost:3001
```

---

## 1. Health Check

### `GET /health`

Returns basic health status.

**Response**
```json
{
  "status": "ok",
  "version": "1.0.0",
  "uptime": 1234,
  "services": {
    "database": "ok",
    "redis": "ok",
    "ollama": "ok"
  }
}
```

---

## 2. Create a Task

### `POST /api/tasks`

Creates a new AI task and routes it to the best harness.

**Request Body**
```json
{
  "description": "Implement OAuth2 in this FastAPI project",
  "priority": "high",
  "context": {
    "repo": "https://github.com/user/project",
    "workspace": "/workspace/project"
  },
  "output_format": "pr"
}
```

**Response**
```json
{
  "task_id": "uuid",
  "status": "queued",
  "selected_harness": "claude-code",
  "fallbacks": ["openhands", "dsh"],
  "routing_score": 0.94,
  "created_at": "2026-10-12T00:00:00Z"
}
```

---

## 3. Get Task Status

### `GET /api/tasks/:task_id`

Returns full status of an execution session.

**Response**
```json
{
  "task_id": "uuid",
  "status": "running",
  "harness": "claude-code",
  "progress": 45,
  "start_time": "2026-10-12T00:01:00Z",
  "last_event": "writing files"
}
```

---

## 4. Stream Execution Updates

### `GET /api/tasks/:task_id/stream`

Server-sent events stream for status and logs.

**Example event**
```text
data: {"event":"progress","progress":30,"message":"Reading repo files"}

data: {"event":"result","output":"Implementation ready"}
```

---

## 5. Get Results

### `GET /api/tasks/:task_id/result`

Returns final output and metadata.

**Example response**
```json
{
  "task_id": "uuid",
  "status": "completed",
  "harness": "claude-code",
  "result": {
    "summary": "OAuth2 implemented successfully",
    "files_changed": ["auth.py", "router.py"]
  },
  "cost_estimate": 0.0,
  "execution_time_seconds": 16.5
}
```

---

## 6. Ask Memory

### `POST /api/memory/query`

Search memory and retrieve relevant prior context.

**Request Body**
```json
{
  "query": "How did we implement OAuth2 before?",
  "limit": 5
}
```

**Response**
```json
{
  "results": [
    {
      "title": "OAuth2 Previous Implementation",
      "relevance": 0.92,
      "content": "We use JWT + FastAPI + token refresh flow"
    }
  ]
}
```

---

## 7. Provider Switch

### `POST /api/providers/switch`

For manual override or policy changes.

**Request Body**
```json
{
  "provider": "groq",
  "enabled": true
}
```

---

## 8. Session History

### `GET /api/sessions`

Returns historical sessions for a user or workspace.

---

## 9. Error Format

```json
{
  "error": {
    "code": "TASK_FAILED",
    "message": "Selected harness failed and no fallback was available",
    "retryable": true,
    "fallback_harness": "openhands"
  }
}
```

---

## 10. Notes

- The AI OS should always keep execution traces for debugging and learning.
- We should log provider selection and provider cost even when the result is free.
