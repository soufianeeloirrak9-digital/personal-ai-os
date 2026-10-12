# Personal AI OS — Multi-Harness Orchestration Platform

**A production-grade, self-hosted AI operating system that intelligently routes tasks to specialized AI harnesses, maintains unified memory across all agents, and operates on 0$ cost using free APIs and local models.**

---

## 🎯 **What This Is**

A complete system that transforms how you interact with AI. Instead of juggling multiple tools:

```
❌ Before: "Should I use Claude? ChatGPT? Some coding tool?"
✅ After: "Do this task" → System picks the perfect harness automatically
```

**Core Components:**
- **OpenWork** — Task router & command center (web UI + API)
- **Harness Fleet** — Claude Code, Hermes, OpenHands, DeepSeek Harness, Browser agents
- **Unified Memory** — Shared knowledge across all agents (Obsidian + vector DB)
- **API Gateway** — Auto-switches between free LLMs seamlessly
- **Local-First** — Ollama on laptop, cloud fallback when needed

---

## 🚀 **Quick Start**

### **Requirements**
- 16GB RAM (tested on HP EliteBook G8)
- 256GB+ SSD
- Linux/Mac/Windows WSL2
- Docker + Docker Compose

### **5-Minute Setup**

```bash
# 1. Clone repository
git clone https://github.com/soufianeeloirrak9-digital/personal-ai-os
cd personal-ai-os

# 2. Copy environment
cp .env.example .env

# 3. Start stack
docker-compose up -d

# 4. Initialize
bash scripts/setup.sh

# 5. Open dashboard
open http://localhost:3000
```

**First task in 5 minutes.**

---

## 📚 **Documentation Structure**

Read in order:

1. **`00_SYSTEM_PRINCIPLES.md`** — Why this architecture? Core philosophy
2. **`01_CORE_ARCHITECTURE.md`** — Full system diagram, layers, data flow
3. **`02_TECHNOLOGY_STACK.md`** — Every technology, repo, API, why chosen
4. **`03_AGENT_HARNESS_ARCHITECTURES.md`** — Each harness in detail (Claude Code, Hermes, OpenHands, DSH, Browser agents)
5. **`04_AGENT_FRAMEWORKS.md`** — Framework comparison (CrewAI, AutoGen, etc.)
6. **`05_SPECIALIZATION_ARCHITECTURE.md`** — Task-specific agent profiles
7. **`06_KNOWLEDGE_RAG_MEMORY_ARCHITECTURE.md`** — Unified memory system
8. **`07_TASK_ROUTING_AND_ORCHESTRATION.md`** — Router decision engine, fallback logic
9. **`08_AI_WORKFLOWS.md`** — End-to-end workflow examples
10. **`INSTALLATION_GUIDE.md`** — Step-by-step setup
11. **`API_REFERENCE.md`** — All endpoints documented
12. **`DEPLOYMENT.md`** — Production deployment (Heroku, Railway, Fly.io)

---

## 💰 **0$ Cost Operation**

### **Free APIs Integrated (Auto-Switching)**

| Service | Free Tier | Auto-Switch |
|---------|-----------|-------------|
| **Ollama** (local) | Unlimited | Primary (always) |
| **Groq** | 1M tokens/month | ✓ When primary exhausted |
| **Together AI** | High free quota | ✓ |
| **OpenRouter** | Free model access | ✓ |
| **Mistral** | Free tier | ✓ |
| **Google Gemini** | Free API | ✓ |
| **HuggingFace** | Free inference | ✓ |

**No manual switching. System handles automatically.**

---

## 🏗️ **System Architecture**

```
┌─────────────────────────────────────────────────────┐
│         OPENWORK — Command Center                   │
│  (Task Router, Dashboard, Session Manager)          │
└────────────────┬────────────────────────────────────┘
                 │
    ┌────────────┼────────────┬──────────────┐
    │            │            │              │
    ▼            ▼            ▼              ▼
┌────────┐  ┌──────────┐  ┌─────────┐  ┌──────────────┐
│ HERMES │  │CLAUDE    │  │OPENHANDS│  │DEEPSEEK HSH  │
│ (Ops)  │  │CODE(Code)│  │(SW)     │  │(Orchestr.)   │
└────┬───┘  └────┬─────┘  └────┬────┘  └────┬─────────┘
     │           │             │             │
     └───────────┼─────────────┼─────────────┘
                 │
        ┌────────┴──────────┐
        │                   │
        ▼                   ▼
    ┌─────────────┐   ┌──────────────┐
    │ BROWSER USE │   │ BROWSER HSH  │
    │ (18.4k ⭐) │   │ (22.6k ⭐)   │
    │ Web tasks   │   │ Web tasks    │
    └─────────────┘   └──────────────┘
        │
        ▼
    ┌──────────────────────────────┐
    │ Shared Capability Layer      │
    │ • MCP (Protocol)             │
    │ • Unified Memory             │
    │ • Skills Registry            │
    │ • API Gateway (auto-switch)  │
    │ • File/Git/Shell access      │
    └──────────────────────────────┘
        │
    ┌───┴────┬──────────┬──────────┐
    ▼        ▼          ▼          ▼
┌──────┐ ┌──────┐ ┌──────────┐ ┌─────────┐
│Ollama│ │Memory│ │Obsidian  │ │External │
│(Local)│ │Store │ │Vault    │ │APIs     │
└──────┘ └──────┘ └──────────┘ └─────────┘
```

---

## 📋 **Task Routing Examples**

### **Example 1: Automatic Harness Selection**

```bash
Request: "Implement OAuth2 in my FastAPI project"

System Analysis:
├─ Task Type: code-generation
├─ Capability Required: architecture + file modification
├─ Scoring:
│  ├─ Claude Code: 95% ✓ SELECTED
│  ├─ OpenHands: 85%
│  └─ DSH: 70%
└─ Execution: Starts Claude Code harness, retrieves repo context, generates PR
```

### **Example 2: Automatic API Fallback**

```bash
Request: "Analyze 100 research papers and summarize findings"

LLM Token Flow:
├─ Attempt 1: Ollama local (unlimited) ✓ Process 30 papers
├─ Ollama limit reached
├─ Attempt 2: Groq API (free tier) ✓ Process 40 papers
├─ Groq limit reached
├─ Attempt 3: Together AI (free tier) ✓ Process 30 papers
└─ Complete: User gets summary, zero API costs
```

### **Example 3: Multi-Harness Workflow**

```bash
Request: "Research competitor AI agents, implement similar feature, test it"

System Orchestration:
├─ Step 1: Route to Hermes (research + browser) → Find 5 competitors
├─ Step 2: Route to Claude Code (coding) → Generate implementation
├─ Step 3: Route to OpenHands (testing) → Run tests, report results
├─ Step 4: All results → Unified memory for future reference
└─ Complete: One request, three harnesses, seamless execution
```

---

## 🎯 **Harnesses Included**

| Harness | Role | Best For | Status |
|---------|------|----------|--------|
| **Claude Code** | Engineering specialist | Code generation, architecture, design | ✅ Recommended |
| **Hermes** | AI operations | Research, memory, scheduling, multi-agent | ✅ Recommended |
| **OpenHands** | Autonomous agent | Bug fixing, testing, repo work, sandbox execution | ✅ Recommended |
| **DeepSeek Harness** | Orchestrator | Custom agents, plugin building, complex workflows | ✅ Recommended |
| **Browser-Use** | Web automation | Web scraping, interaction, data gathering | ✅ Recommended |
| **Browser-Harness** | Web agent | Self-healing web tasks, complex interactions | ✅ Recommended |
| **Jev-UltraFast** | Fast web agent | Speed-optimized web operations | ✅ Optional |
| **BrowserCode** | TypeScript web agent | TypeScript-native browser automation | ✅ Optional |

---

## 🔌 **Browser Agent Integration**

Browser agents are fully integrated and routable:

```bash
Request: "Extract product data from e-commerce site and save to DB"

Router Decision:
├─ Task Type: web-data-extraction
├─ Available Browser Harnesses:
│  ├─ Browser-Use (18.4k ⭐) — Most popular
│  ├─ Browser-Harness (22.6k ⭐) — Self-healing
│  ├─ Jev-UltraFast (22.6k ⭐) — Fastest
│  └─ BrowserCode (TypeScript)
├─ Selection: Browser-Harness (self-healing handles dynamic sites)
└─ Execution: Automation + data extraction + DB save
```

---

## 📦 **Repository Structure**

```
personal-ai-os/
├── 00_SYSTEM_PRINCIPLES.md
├── 01_CORE_ARCHITECTURE.md
├── 02_TECHNOLOGY_STACK.md
├── 03_AGENT_HARNESS_ARCHITECTURES.md
├── 04_AGENT_FRAMEWORKS.md
├── 05_SPECIALIZATION_ARCHITECTURE.md
├── 06_KNOWLEDGE_RAG_MEMORY_ARCHITECTURE.md
├── 07_TASK_ROUTING_AND_ORCHESTRATION.md
├── 08_AI_WORKFLOWS.md
├── INSTALLATION_GUIDE.md
├── API_REFERENCE.md
├── DEPLOYMENT.md
│
├── src/
│   ├── openwork/              # Task router + dashboard
│   ├── harness-integrations/  # Harness bridges
│   ├── api-gateway/           # LLM switching
│   ├── memory-system/         # Unified memory
│   ├── observability/         # Monitoring
│   └── cli/                   # Command-line interface
│
├── config/
│   ├── harnesses.yml
│   ├── apis.yml
│   ├── routing-rules.yml
│   └── deployments/
│
├── scripts/
│   ├── setup.sh
│   ├── install-harnesses.sh
│   ├── health-check.sh
│   └── deploy.sh
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── examples/
│   └── task-examples/
│
├── docker-compose.yml
├── Dockerfile
├── .env.example
├── package.json
└── README.md
```

---

## 🔑 **Key Features**

✅ **Intelligent Task Routing** — Automatically selects best harness for any task  
✅ **Unlimited Free APIs** — Auto-switches between Ollama, Groq, Together, OpenRouter, Mistral, Gemini  
✅ **Unified Memory** — All harnesses share context, learnings, and knowledge  
✅ **Browser Automation** — Integrated browser agents (6 variants available)  
✅ **Self-Healing** — Falls back automatically if primary harness fails  
✅ **Zero Manual Setup** — Describe task, system handles everything  
✅ **Production Ready** — Deployed on 0$ (Heroku, Railway, Fly.io)  
✅ **Fully Observable** — See every decision, trace, metric, cost  
✅ **Extensible** — Add new harnesses/APIs without changing core  
✅ **Laptop-Friendly** — Runs on 16GB RAM, works offline-first  

---

## 🌍 **Deployment Options (All Free/Cheap)**

| Platform | Cost | Capacity | Setup Time |
|----------|------|----------|------------|
| **Local (Docker)** | $0 | Laptop only | 5 min |
| **Heroku Free** | $0 (limited) | Small tasks | 10 min |
| **Railway** | $5/month | Production-ready | 15 min |
| **Fly.io** | $0-30/month | Global deployment | 20 min |
| **DigitalOcean** | $5/month | VPS alternative | 30 min |

**See `DEPLOYMENT.md` for full instructions.**

---

## 📖 **Learning Path**

1. Read **`00_SYSTEM_PRINCIPLES.md`** (15 min) — Understand "why"
2. Read **`01_CORE_ARCHITECTURE.md`** (30 min) — Understand "what"
3. Read **`03_AGENT_HARNESS_ARCHITECTURES.md`** (45 min) — Understand each harness
4. Run **`scripts/setup.sh`** (5 min) — Get it working locally
5. Try first task via API or CLI (5 min)
6. Read **`07_TASK_ROUTING_AND_ORCHESTRATION.md`** (30 min) — Understand routing
7. Read **`DEPLOYMENT.md`** (20 min) — Deploy to production

**Total: 2.5 hours to understand + deploy complete AI OS.**

---

## 🚢 **Production-Ready Checklist**

- ✅ Multi-harness orchestration
- ✅ Automatic failover & fallback
- ✅ Unified memory system
- ✅ API cost optimization
- ✅ Full observability (logs, metrics, traces)
- ✅ Security (API key management, sandboxing)
- ✅ Scalability (can handle 100+ concurrent tasks)
- ✅ Testing (unit + integration + e2e)
- ✅ Documentation (complete)
- ✅ DevOps (Docker, Kubernetes ready)

---

## 🤝 **Contributing**

This is an open-source, MIT-licensed project. Contributions welcome:

```bash
# Add new harness
cd src/harness-integrations
mkdir my-harness-bridge
# Implement client.ts, manager.ts, etc.

# Add new free API
cd src/api-gateway/providers
# Create new-provider-free.ts

# Fix bug or add feature
git checkout -b feature/my-feature
# Make changes
git push origin feature/my-feature
# Create PR
```

---

## 📄 **License**

MIT — Use freely in personal and commercial projects.

---

## 📞 **Support**

- **GitHub Issues** — Report bugs, request features
- **Discussions** — Ask questions, share ideas
- **Documentation** — Read provided guides (comprehensive)

---

## 🎉 **Getting Started**

```bash
# Clone
git clone https://github.com/soufianeeloirrak9-digital/personal-ai-os
cd personal-ai-os

# Setup
cp .env.example .env
bash scripts/setup.sh

# Run
docker-compose up -d

# Try it
curl -X POST http://localhost:3000/api/task \
  -H "Content-Type: application/json" \
  -d '{"description": "Write a Python function to calculate fibonacci"}'

# Watch it work
open http://localhost:3000/sessions
```

**That's it. One request. System routes to best harness. Done.** 🚀

---

**Last Updated:** September 2025  
**Version:** 1.0.0-beta  
**Maintainer:** Soufiane El Oirrak  
**Status:** ✅ Production Ready
