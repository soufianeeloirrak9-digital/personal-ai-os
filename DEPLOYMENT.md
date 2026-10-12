# Deployment Guide

## Overview

This guide covers local deployment, minimal cloud deployment, and production-ready deployment for the Personal AI OS.

---

## 1. Local Deployment

### Recommended local setup
- Docker Compose for the full stack
- Ollama models locally
- Postgres + Redis for state
- OpenWork UI for administration

### Steps

```bash
cp .env.example .env
docker-compose up -d
```

Then check:
- http://localhost:3000 — UI
- http://localhost:3001 — API
- http://localhost:11434 — Ollama

---

## 2. Cloud Deployment Options

### Option A: Railway
Good for quick deployment and GitHub-based workflow.

```bash
railway login
railway init
railway up
```

### Option B: Fly.io
Good for private deployment with low monthly cost.

```bash
flyctl launch
flyctl deploy
```

### Option C: DigitalOcean
Good when you want more control and a simple VPS model.

---

## 3. Production Best Practices

- Keep API keys in `.env` or secret manager
- Encrypt secrets and avoid storing them in plaintext
- Use PostgreSQL with volume persistence
- Keep Redis for cache and session state
- Secure webhook endpoints
- Add rate limiting for public APIs
- Monitor execution traces and cost logs

---

## 4. Security Checklist

- Use environment secrets
- Keep local model files inside a secure directory
- Restrict shell execution in production
- Only allow file writes in explicitly approved directories
- Use sandboxing for autonomous tasks

---

## 5. Cost-Saving Production Plan

Use this order by default:
1. Local Ollama
2. Groq free tier
3. Together free tier
4. OpenRouter low-cost provider
5. Premium model only when necessary

This ensures the system remains effectively $0 for most workloads.

---

## 6. Scaling

### Small scale
- single server
- Postgres on the same host
- Redis on same host
- UI and API on same host

### Medium scale
- multiple API replicas
- Postgres primary + read replica
- Redis cluster optional

### Large scale
- Kubernetes / Docker swarm
- autoscaling API layer
- persistent DB pool
- separate memory infrastructure

---

## 7. Production Summary

The AI OS can stay cheap and still be professional if the architecture is designed around:
- local-first inference
- free provider fallback
- modular execution runtime
- structured observability
- secure and reusable deployment pattern
