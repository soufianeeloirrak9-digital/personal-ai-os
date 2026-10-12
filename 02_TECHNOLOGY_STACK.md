# Technology Stack: Tools, Repositories, and Free APIs

## Table of Contents
1. Local Model Runtime (Ollama)
2. Cloud LLM Providers (Free Tier APIs)
3. Harness Implementations
4. Browser Automation Tools
5. Storage & Persistence
6. Task Routing & Orchestration
7. Development Tools
8. Infrastructure
9. Free APIs Registry
10. Cost Model & Decision Matrix

---

## 1. Local Model Runtime: Ollama

### Why Ollama?

- ✅ **Zero cost** – Unlimited local inference
- ✅ **Instant latency** – No API calls
- ✅ **Privacy** – All data stays local
- ✅ **Offline capable** – Works without internet
- ✅ **Easy management** – CLI-based model downloads
- ✅ **OpenAI-compatible API** – Plug-and-play

### Recommended Models (16GB RAM Laptop)

#### Primary: Qwen 2.5 Coder 8B
- **Size**: 4.7GB quantized (4-bit)
- **VRAM**: 5-6GB when loaded
- **Best for**: General coding, everyday tasks
- **Speed**: 40-80 tokens/sec
- **Quality**: ⭐⭐⭐⭐ (Excellent for 8B)
- **Install**: `ollama pull qwen2.5-coder:8b`

#### Fallback 1: DeepSeek Coder 6.7B
- **Size**: 3.8GB quantized (4-bit)
- **VRAM**: 4-5GB when loaded
- **Best for**: Code generation, programming
- **Speed**: 50-100 tokens/sec
- **Quality**: ⭐⭐⭐⭐
- **Install**: `ollama pull deepseek-coder:6.7b`

#### Fallback 2: GLM-4 9B
- **Size**: 5.2GB quantized (4-bit)
- **VRAM**: 6-7GB when loaded
- **Best for**: Reasoning, multi-language
- **Speed**: 35-70 tokens/sec
- **Quality**: ⭐⭐⭐⭐
- **Install**: `ollama pull glm4:9b`

#### Heavy Tasks (Swap to): Qwen 32B (When needed)
- **Size**: 18GB full precision
- **VRAM**: 20-22GB with swap
- **Best for**: Complex reasoning, advanced tasks
- **Speed**: 10-20 tokens/sec (slow, use as last resort)
- **Install**: `ollama pull qwen:32b`

### Ollama Setup

```bash
# macOS
brew install ollama

# Linux
curl https://ollama.ai/install.sh | sh

# Windows (WSL2)
curl https://ollama.ai/install.sh | sh

# Start service
ollama serve

# In another terminal, download models
ollama pull qwen2.5-coder:8b
ollama pull deepseek-coder:6.7b
ollama pull glm4:9b

# Test
curl http://localhost:11434/api/generate -d '{
  "model": "qwen2.5-coder:8b",
  "prompt": "Write a Python function for fibonacci",
  "stream": false
}'
```

### Ollama Configuration

```bash
# ~/.ollama/ollama_env (or environment variables)

# Memory limits (for 16GB laptop)
OLLAMA_NUM_GPU=0              # Set to 1 if you have NVIDIA GPU
OLLAMA_NUM_THREADS=8          # CPU cores to use
OLLAMA_MAX_LOADED_MODELS=2    # Keep 2 models in memory

# Storage
OLLAMA_MODELS=/path/to/models # Where to store downloaded models

# API
OLLAMA_HOST=127.0.0.1:11434   # Default
```

---

## 2. Cloud LLM Providers (Free Tier APIs)

### Provider Hierarchy (Auto-Switching Order)

#### TIER 0: Local (Ollama) — Always Try First
- **Cost**: $0
- **Tokens**: Unlimited
- **Speed**: Variable (CPU-dependent)
- **Availability**: 100% (offline capable)
- **Decision**: Use for all everyday tasks

#### TIER 1: Groq (Free API)
- **Website**: https://groq.com
- **Cost**: $0 (free tier)
- **Tokens**: 1M tokens/month
- **Models**: Mixtral 8x7B, LLaMA 2 70B
- **Speed**: 500+ tokens/sec (blazing fast)
- **Latency**: <100ms
- **Setup**:
  ```bash
  # Get API key from groq.com
  export GROQ_API_KEY="gsk_..."
  
  # Python client
  pip install groq
  ```
- **Use When**: Ollama hits token limit, need faster inference

#### TIER 2: Together AI (Free API)
- **Website**: https://together.ai
- **Cost**: $0 (generous free tier)
- **Tokens**: High monthly quota
- **Models**: Llama 2, Mistral, Code Llama
- **Speed**: 100-300 tokens/sec
- **Latency**: <500ms
- **Setup**:
  ```bash
  export TOGETHER_API_KEY="..."
  pip install together
  ```
- **Use When**: Groq quota exceeded, need coding-specific models

#### TIER 3: OpenRouter (Cheap Paid)
- **Website**: https://openrouter.ai
- **Cost**: $0.50-2.00 per 1M tokens (varies by model)
- **Tokens**: No monthly limit
- **Models**: 100+ model options
- **Speed**: Depends on backend
- **Setup**:
  ```bash
  export OPENROUTER_API_KEY="..."
  pip install requests
  ```
- **Use When**: Free tiers exhausted, need specific model

#### TIER 4: Mistral Free API
- **Website**: https://console.mistral.ai
- **Cost**: $0 (free tier)
- **Tokens**: Monthly limit
- **Models**: Mistral 7B, Mistral Large
- **Speed**: 50-150 tokens/sec
- **Setup**:
  ```bash
  export MISTRAL_API_KEY="..."
  ```
- **Use When**: Free Groq/Together exhausted

#### TIER 5: Google Gemini (Free API)
- **Website**: https://ai.google.dev
- **Cost**: $0 (free tier)
- **Tokens**: 1500 RPM limit
- **Models**: Gemini 1.5 Pro (free tier)
- **Speed**: 100-200 tokens/sec
- **Setup**:
  ```bash
  export GOOGLE_API_KEY="..."
  ```
- **Use When**: Need multimodal (images), premium reasoning

#### TIER 6: HuggingFace Inference API (Free)
- **Website**: https://huggingface.co
- **Cost**: $0 (with free account)
- **Tokens**: Rate limited
- **Models**: Open-weight models
- **Setup**:
  ```bash
  export HF_API_KEY="..."
  ```
- **Use When**: Open-weight model enthusiasts

#### TIER 7: Claude/GPT (Premium — Last Resort)
- **Cost**: $0.01-1.00 per request
- **Use Only When**: All free tiers exhausted + high priority task

### Free APIs Auto-Switching Logic

```python
# src/api-gateway/provider-chain.ts

const PROVIDER_CHAIN = [
  {
    name: 'ollama',
    cost: 0,
    priority: 1,
    fallback_condition: 'model_too_slow || not_good_enough'
  },
  {
    name: 'groq',
    cost: 0,
    priority: 2,
    quota_reset: 'monthly',
    fallback_condition: 'quota_exceeded || network_error'
  },
  {
    name: 'together',
    cost: 0,
    priority: 3,
    quota_reset: 'monthly',
    fallback_condition: 'quota_exceeded || timeout'
  },
  {
    name: 'mistral_free',
    cost: 0,
    priority: 4,
    fallback_condition: 'all_above_failed'
  },
  {
    name: 'gemini_free',
    cost: 0,
    priority: 5,
    fallback_condition: 'need_multimodal || all_above_failed'
  },
  {
    name: 'openrouter_cheap',
    cost: 0.01,  // Cheapest paid
    priority: 6,
    fallback_condition: 'all_free_exhausted || cost_acceptable'
  },
  {
    name: 'claude',
    cost: 0.30,
    priority: 7,
    fallback_condition: 'critical_priority || explicit_request'
  }
];
```

---

## 3. Harness Implementations

### Claude Code (Anthropic)
- **GitHub**: Not open-source (proprietary)
- **Type**: Managed SaaS API
- **Role**: Code generation, architecture
- **API Endpoint**: `https://api.anthropic.com/v1/messages`
- **Cost**: $0.003-0.03 per 1K tokens (Claude 3 Sonnet is cheapest)
- **Setup**:
  ```bash
  export ANTHROPIC_API_KEY="sk-ant-..."
  pip install anthropic
  ```
- **When to Use**: Critical coding tasks, pay for premium quality

### Hermes (Multi-Agent Framework)
- **GitHub**: https://github.com/ai-research-lab/hermes
- **Type**: Open-source framework
- **Role**: AI operations, research, browser automation, memory
- **Setup**:
  ```bash
  git clone https://github.com/ai-research-lab/hermes
  cd hermes
  pip install -e .
  ```
- **When to Use**: Research tasks, long-running agents, memory-dependent work

### OpenHands (Autonomous Agent)
- **GitHub**: https://github.com/OpenDevin/OpenDevin
- **Type**: Open-source framework
- **Role**: Autonomous software tasks, bug fixing, testing
- **Setup**:
  ```bash
  git clone https://github.com/OpenDevin/OpenDevin.git
  cd OpenDevin
  pip install -e .
  ```
- **When to Use**: Autonomous software development, sandbox execution

### DeepSeek Harness (DSH)
- **GitHub**: https://github.com/deepseek-ai/deepseek-harness
- **Type**: Open-source orchestrator
- **Role**: Plugin-based orchestration, complex workflows
- **Setup**:
  ```bash
  npm install -g @deepseek-ai/dsh
  dsh web  # Start web UI on port 3080
  ```
- **When to Use**: Custom agent building, multi-step workflows

---

## 4. Browser Automation Tools

### Browser-Use (Most Popular)
- **GitHub**: https://github.com/browser-use/browser-use (18.4k ⭐)
- **Type**: Open-source Python library
- **Role**: Web automation, data extraction, interaction
- **Install**: `pip install browser-use`
- **When to Use**: Web scraping, user-like interactions

### Browser-Harness (Self-Healing)
- **GitHub**: https://github.com/browser-harness/browser-harness (22.6k ⭐)
- **Type**: Open-source framework
- **Role**: Self-recovering browser automation
- **Install**: `pip install browser-harness`
- **When to Use**: Complex dynamic websites, need fault recovery

### Jev-UltraFast (Speed Optimized)
- **GitHub**: https://github.com/jev-ai/jev-ultrafast (22.6k ⭐)
- **Type**: Lightweight alternative
- **Role**: Fast web automation
- **Install**: `pip install jev-ultrafast`
- **When to Use**: Performance-critical tasks, large-scale scraping

### BrowserCode (TypeScript Native)
- **GitHub**: https://github.com/browsercode/browsercode
- **Type**: TypeScript framework
- **Role**: Type-safe browser automation
- **Install**: `npm install browsercode`
- **When to Use**: TypeScript/Node.js projects

---

## 5. Storage & Persistence

### PostgreSQL
- **Purpose**: Relational data (sessions, tasks, memory metadata)
- **Version**: 14+
- **Docker**: `docker run -d --name postgres -e POSTGRES_PASSWORD=secret postgres:15-alpine`
- **Connection**: `postgresql://user:pass@localhost:5432/aidb`
- **Schema**:
  - `sessions` – Task execution sessions
  - `tasks` – Task definitions & metadata
  - `traces` – Execution traces (events)
  - `memory` – Memory records
  - `users` – User accounts

### Vector Database (pgvector)
- **Purpose**: Semantic search, embeddings
- **Setup**:
  ```sql
  CREATE EXTENSION IF NOT EXISTS vector;
  CREATE TABLE embeddings (
    id SERIAL PRIMARY KEY,
    task_id UUID,
    embedding vector(1536),
    text TEXT,
    created_at TIMESTAMP
  );
  CREATE INDEX ON embeddings USING ivfflat (embedding vector_cosine_ops);
  ```
- **Usage**: Store embeddings, find similar past tasks

### Redis
- **Purpose**: Cache, session state, rate limiting
- **Docker**: `docker run -d --name redis redis:7-alpine`
- **Use Cases**:
  - Cache LLM responses
  - Store temporary session state
  - Rate limiting per user/API
  - Queue for async tasks

### Obsidian Vault (File-Based Knowledge)
- **Purpose**: Local knowledge base, markdown docs, project notes
- **Location**: `~/.ai-os/obsidian-vault/`
- **Sync**: Git-based versioning
- **Structure**:
  ```
  obsidian-vault/
  ├── 00-index.md           # Central hub
  ├── projects/
  │   └── [project-name]/
  ├── learnings/            # What system learned
  │   └── [task-id].md
  ├── sops/                 # Standard operating procedures
  ├── research/             # Research notes
  └── brand/                # Brand guidelines
  ```
- **Sync to DB**: Obsidian → Embeddings → PostgreSQL + pgvector

---

## 6. Task Routing & Orchestration

### OpenWork (Task Router)
- **Type**: Custom Next.js application
- **Repository**: This repo's `src/openwork/`
- **Responsibilities**:
  - Task classification (NLP)
  - Harness capability scoring
  - Route selection logic
  - Result aggregation
  - Memory updates
  - User notifications

### LiteLLM (Optional LLM Proxy)
- **GitHub**: https://github.com/BerriAI/litellm
- **Purpose**: Unified interface for multiple LLM providers
- **Use If**: Want centralized rate limiting, cost tracking
- **Setup**: `pip install litellm`

---

## 7. Development Tools

### Git & GitHub
- **Purpose**: Version control, repository hosting
- **Use**: All code, docs, Obsidian vault
- **Key Integration**: GitHub API for repo operations

### Docker & Docker Compose
- **Purpose**: Containerization, local development
- **Use**: Run entire stack locally
- **Compose File**: `docker-compose.yml` (included in repo)

### Node.js & npm
- **Version**: 18+
- **Purpose**: Runtime for OpenWork API server
- **Packages**: See `package.json`

### Python & pip
- **Version**: 3.10+
- **Purpose**: Runtime for harness bridges, tools
- **Dependencies**: See `requirements.txt`

### Playwright
- **Purpose**: Browser automation library
- **Install**: `pip install playwright`
- **Use**: Browser tests, automation fallback

---

## 8. Infrastructure

### Local Development Stack

```yaml
Services:
  openwork-api:        Node.js + Express + WebSocket
  openwork-ui:         Next.js + React
  postgresql:          Database
  redis:               Cache + Queues
  ollama:              Local LLM
```

### Production Stack (Cloud)

**Option A: Railway.app ($5-30/month)**
- Auto-deploy from GitHub
- PostgreSQL managed
- Auto-scaling
- Free tier available

**Option B: Fly.io ($0-30/month)**
- Global deployment
- PostgreSQL add-on
- Generous free tier
- Simple CLI deployment

**Option C: DigitalOcean ($5/month)**
- Droplet + managed databases
- Full control
- Good for scaling

---

## 9. Free APIs Registry (Complete)

### Status Monitoring

Create `config/apis.yml`:

```yaml
free_apis:
  - name: ollama
    type: local
    cost: 0
    quota: unlimited
    status_url: http://localhost:11434/api/tags
    
  - name: groq
    type: cloud
    cost: 0
    quota: 1M tokens/month
    reset_date: monthly
    status_url: https://groq.com/status
    
  - name: together
    type: cloud
    cost: 0
    quota: high
    status_url: https://together.ai/status
    
  - name: mistral_free
    type: cloud
    cost: 0
    quota: monthly_limit
    status_url: https://console.mistral.ai
    
  - name: gemini_free
    type: cloud
    cost: 0
    quota: 1500_rpm
    status_url: https://ai.google.dev
```

---

## 10. Cost Model & Decision Matrix

### Task Type → Provider Decision

```
Task Type              | Primary        | Fallback 1     | Fallback 2
---                    | ---            | ---            | ---
Code generation        | Qwen 8B        | DeepSeek 6.7B  | Groq
Bug fixing            | Qwen 8B        | Groq           | Claude
Research              | Qwen 8B        | Groq           | Gemini
Testing               | Qwen 8B        | DeepSeek       | Together
Browser automation    | Browser-Use    | Browser HSH    | Jev UltraFast
Multi-agent work      | DSH            | Hermes         | OpenHands
Critical/complex      | Qwen 32B (swap)| Groq           | Claude
```

### Monthly Cost Estimate (1000 tasks)

```
Scenario 1: Offline-Only (No internet)
• Ollama local: $0/month
• Browser automation: $0/month
• Total: $0

Scenario 2: Normal Usage (Some cloud)
• Ollama 70%: $0
• Groq free 20%: $0
• Together free 8%: $0
• OpenRouter paid 2%: $2
• Total: $2/month

Scenario 3: Quality-First (Premium models)
• Ollama 40%: $0
• Free APIs 30%: $0
• OpenRouter 20%: $5
• Claude 10%: $5
• Total: $10/month

Scenario 4: Production (Large scale)
• Assuming 10k tasks/month
• Free APIs cover 95%: $0
• Premium APIs 5%: $50
• Total: $50/month
```

---

## Summary Table

| Component | Tech | Cost | Purpose |
|-----------|------|------|----------|
| Local LLM | Ollama | $0 | 70-80% of tasks |
| Cloud LLM (Free) | Groq, Together, Mistral | $0 | 15-20% of tasks |
| Cloud LLM (Paid) | OpenRouter, Claude, GPT | <$1 | Edge cases |
| Browser | Browser-Use + alts | $0 | Web automation |
| Database | PostgreSQL | $0 (local) | Persistence |
| Cache | Redis | $0 (local) | Performance |
| Harnesses | Open-source | $0 | Execution engines |
| **TOTAL** | | **$0-5/month** | **Complete AI OS** |

---

## Next Steps

1. Set up Ollama with recommended models
2. Sign up for Groq API (free)
3. Sign up for Together AI (free)
4. Configure `.env` file with API keys
5. Run `docker-compose up` to start local stack
6. See `INSTALLATION_GUIDE.md` for detailed setup
