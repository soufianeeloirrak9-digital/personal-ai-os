# Core Architecture: System Design and Data Flow

## Table of Contents
1. System Layers
2. Component Diagram
3. Data Flow
4. Harness Communication
5. Memory Architecture
6. API Gateway Flow
7. Deployment Architecture

---

## 1. System Layers

### Layer Stack (Top to Bottom)

```
┌─────────────────────────────────────────────────────────┐
│ EXPERIENCE LAYER                                        │
│ ─────────────────────────────────────────────────────── │
│ OpenWork Dashboard (Web UI)                              │
│ • Task board (Kanban)                                    │
│ • Session monitor (real-time logs)                       │
│ • Routing debugger (why this harness?)                   │
│ • Capability matrix (what each harness can do)           │
│ • Cost tracker ($ per task/harness)                      │
│ • Memory browser (search knowledge base)                 │
└──────────────────┬──────────────────────────────────────┘
                   │ (REST API + WebSocket)
┌──────────────────▼──────────────────────────────────────┐
│ CONTROL LAYER                                           │
│ ─────────────────────────────────────────────────────── │
│ Task Router (Intelligence)                               │
│ • Task classification (what type of task?)               │
│ • Capability scoring (match task to harness)             │
│ • Harness selection (pick best match)                    │
│ • Fallback management (if primary fails)                 │
│ • Session orchestration (coordinate parallel tasks)      │
│ • Result aggregation (combine harness outputs)           │
│ • Learning loop (update routing based on outcomes)       │
└──────────────────┬──────────────────────────────────────┘
                   │ (gRPC + Event Stream)
┌──────────────────▼──────────────────────────────────────┐
│ HARNESS LAYER                                           │
│ ─────────────────────────────────────────────────────── │
│ [Claude Code] [Hermes] [OpenHands] [DSH] [Browser Use]  │
│                                                          │
│ Each harness:                                            │
│ • Has specialized capabilities                           │
│ • Implements standard HarnessClient interface            │
│ • Accesses shared tools via MCP                          │
│ • Updates unified memory with learnings                  │
│ • Reports telemetry to control layer                     │
└──────────────────┬──────────────────────────────────────┘
                   │ (MCP Protocol)
┌──────────────────▼──────────────────────────────────────┐
│ CAPABILITY LAYER                                        │
│ ─────────────────────────────────────────────────────── │
│ Shared Tools & Services                                  │
│ ┌────────────────┬──────────────┬──────────────────┐   │
│ │ File Tools     │ Git Tools    │ Shell Tools      │   │
│ │ • Read         │ • Clone      �� • Execute        │   │
│ │ • Write        │ • Commit     │ • Monitor        │   │
│ │ • Delete       │ • Push       │ • Stream output  │   │
│ │ • Watch        │ • PR create  │                  │   │
│ └────────────────┴──────────────┴──────────────────┘   │
│ ┌────────────────┬──────────────┬──────────────────┐   │
│ │ Browser Tools  │ DB Tools     │ API Tools        │   │
│ │ • Navigate     │ • Query      │ • HTTP call      │   │
│ │ • Interact     │ • Insert     │ • Webhook post   │   │
│ │ • Screenshot   │ • Update     │ • Auth handle    │   │
│ │ • Extract data │ • Delete     │                  │   │
│ └────────────────┴──────────────┴──────────────────┘   │
└──────────────────┬──────────────────────────────────────┘
                   │ (Tool APIs)
┌──────────────────▼──────────────────────────────────────┐
│ PERSISTENCE LAYER                                       │
│ ─────────────────────────────────────────────────────── │
│ ┌─────────────┐  ┌─────────────┐  ┌──────────────┐    │
│ │ PostgreSQL  │  │ Vector DB   │  │ File System  │    │
│ │             │  │             │  │              │    │
│ │ • Sessions  │  │ • Embeddings│  │ • Obsidian   │    │
│ │ • Tasks     │  │ • Semantic  │  │ • Code repos │    │
│ │ • Memory    │  │   search    │  │ • Artifacts  │    │
│ │ • State     │  │ • Context   │  │ • Downloads  │    │
│ │ • Users     │  │   retrieval │  │              │    │
│ └─────────────┘  └─────────────┘  └──────────────┘    │
│                 (Persistence APIs)                      │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Component Diagram

### Full System Components

```
                    ┌──────────────────────┐
                    │   OpenWork UI        │
                    │   (React + Next.js)  │
                    └──────────┬───────────┘
                               │
                ┌──────────────┴──────────────┐
                │                             │
                ▼                             ▼
        ┌──────────────┐           ┌──────────────────┐
        │  REST API    │           │  WebSocket       │
        │  Port 3000   │           │  Real-time logs  │
        └──────┬───────┘           └────────┬─────────┘
               │                            │
               └──────────────┬─────────────┘
                              │
                    ┌─────────▼────────┐
                    │  Task Router     │
                    │  (Decision AI)   │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
  │ Classifier   │  │ Scorer       │  │ Selector     │
  │              │  │              │  │              │
  │ NLP models   │  │ Capability   │  │ Best match   │
  │ Rule engine  │  │ scoring      │  │ with         │
  │              │  │ algorithm    │  │ fallbacks    │
  └──────────────┘  └──────────────┘  └──────────────┘
                             │
        ┌────────────────────┼──────────────────────┐
        │                    │                     │
        ▼                    ▼                     ▼
  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐
  │ Harness      │  │ Harness      │  │ Harness       │
  │ Claude Code  │  │ Hermes       │  │ OpenHands     │
  └──────┬───────┘  └──────┬───────┘  └────────┬──────┘
         │                 │                    │
         └────────────┬────┴────────┬───────────┘
                      │             │
            ┌─────────▼─────┐   ┌───▼──────────────┐
            │ MCP Protocol  │   │ API Gateway      │
            │               │   │                  │
            │ Shared Tools: │   │ Ollama (local)   │
            │ • File access │   │ Groq (free)      │
            │ • Git ops     │   │ Together (free)  │
            │ • Shell exec  │   │ OpenRouter       │
            │ • Browser     │   │ Anthropic        │
            │ • DB access   │   │ Google           │
            └────────┬──────┘   └──────┬───────────┘
                     │                 │
         ┌───────────┴─────────┬───────┴──────────┐
         │                     │                  │
         ▼                     ▼                  ▼
    ┌─────────┐          ┌──────────┐      ┌──────────┐
    │ Memory  │          │Local LLM │      │Cloud LLM │
    │ System  │          │ (Ollama) │      │(APIs)    │
    │         │          │          │      │          │
    │Obsidian │          │Qwen 8B  │      │Groq API  │
    │Vector DB│          │DeepSeek │      │Together  │
    │Postgres │          │GLM 9B   │      │Mistral   │
    └─────────┘          └──────────┘      └──────────┘
```

---

## 3. Data Flow

### Complete Request Flow

```
1. USER SUBMITS TASK
   ├─ Channel: REST API / CLI / UI
   ├─ Payload: Task description + optional context
   └─ Stored: PostgreSQL with UUID

2. TASK CLASSIFICATION
   ├─ Parse: Extract intent, entities, constraints
   ├─ Classify: ML model determines task category
   │  └─ Examples: code-generation, research, testing, automation
   └─ Result: TaskType enum

3. CAPABILITY SCORING
   ├─ Available Harnesses: [Claude Code, Hermes, OpenHands, DSH, Browser]
   ├─ For each harness:
   │  ├─ Get capability profile
   │  ├─ Score against task requirements
   │  └─ Calculate confidence (0-100)
   └─ Result: HarnessScore[] sorted by score

4. HARNESS SELECTION
   ├─ Select: Highest scoring harness
   ├─ Create: Session with UUID
   ├─ Notify: Selected harness via gRPC
   └─ Store: Session record in PostgreSQL

5. HARNESS EXECUTION (Primary)
   ├─ Session starts in selected harness
   ├─ Access shared capabilities via MCP
   │  ├─ File system: Read repo, write code
   │  ├─ Git: Clone, create branches, commit
   │  ├─ Shell: Run tests, execute scripts
   │  └─ Browser: Navigate, interact, scrape
   ├─ Internal model reasoning
   └─ Stream output to router (WebSocket)

6. RESULT COLLECTION
   ├─ Primary harness completes
   ├─ Store result in PostgreSQL
   ├─ Store execution trace
   ├─ Extract learnings
   └─ Update memory system

7. FALLBACK (If Primary Fails)
   ├─ Check: Did harness return error?
   ├─ Analyze: Is it recoverable?
   ├─ Fallback: Route to next best harness
   ├─ Context transfer: Pass accumulated state
   └─ Resume: Continue from where it failed

8. MEMORY UPDATE
   ├─ Store: Task + result + execution trace
   ├─ Embed: Generate embeddings for semantic search
   ├─ Index: Add to vector DB
   ├─ Extract: Learn patterns, skills, preferences
   └─ Share: Make available to all harnesses

9. USER NOTIFICATION
   ├─ Channel: WebSocket / Webhook / Email
   ├─ Content: Results + cost + execution time
   ├─ Interactive: Show traces, execution details
   └─ Actionable: Provide next steps

10. LEARNING LOOP
    ├─ Analyze: Was this the right harness?
    ├─ Success: Did task complete successfully?
    ├─ Efficiency: Could we have done it cheaper?
    ├─ Speed: Could we have done it faster?
    └─ Update: Adjust routing model weights
```

---

## 4. Harness Communication Protocol

### Harness Client Interface

```typescript
interface HarnessClient {
  // Lifecycle
  connect(): Promise<void>;
  disconnect(): Promise<void>;
  isHealthy(): Promise<boolean>;
  
  // Execution
  executeTask(request: TaskRequest): Promise<SessionHandle>;
  streamOutput(sessionId: string): AsyncIterable<ExecutionEvent>;
  getStatus(sessionId: string): Promise<SessionStatus>;
  
  // Capability Declaration
  getCapabilities(): Promise<HarnessCapability[]>;
  scoreTask(task: TaskType): Promise<number>; // 0-100
  
  // Memory & Learnings
  updateMemory(memory: MemoryRecord): Promise<void>;
  queryMemory(query: string): Promise<MemoryRecord[]>;
}

interface TaskRequest {
  taskId: string;        // UUID
  description: string;   // User's task description
  taskType: TaskType;    // Classified task type
  context: TaskContext;  // Repo files, project info, etc.
  outputFormat: 'direct' | 'pr' | 'session' | 'report';
  priority: 'low' | 'medium' | 'high' | 'critical';
  estimatedComplexity: number; // 0-100
  memory?: MemoryContext; // Relevant past tasks
}

type ExecutionEvent = 
  | { type: 'thinking'; message: string }
  | { type: 'tool_call'; tool: string; args: any }
  | { type: 'tool_result'; result: any }
  | { type: 'progress'; percentage: number }
  | { type: 'complete'; result: TaskResult }
  | { type: 'error'; error: string; recoverable: boolean };

interface TaskResult {
  taskId: string;
  status: 'success' | 'partial' | 'failed';
  output: any;           // Varies by task type
  executionTrace: Event[];
  executionTime: number; // Seconds
  tokensUsed: number;
  costEstimate: number;  // $ (decimal)
  learnings?: string[];  // What did system learn?
}
```

---

## 5. Memory Architecture

### Unified Memory System

```
┌─────────────────────────────────────────────────────┐
│ MEMORY SYSTEM                                       │
│ (Shared across all harnesses)                       │
└─────────────────────────────────────────────────────┘

┌─────────────────────────┐
│ EPISODIC MEMORY         │ (Task-specific)
│                         │
│ • What task was done?   │
│ • When?                 │
│ • What was the result?  │
│ • What was learned?     │
│                         │
│ Store: Full execution   │
│        traces           │
│ Retrieve: By task ID or │
│           similarity    │
└────────────┬────────────┘
             │ (Foreign key)
             ▼
    ┌──────────────────┐
    │ PostgreSQL       │
    │ sessions table   │
    │ tasks table      │
    │ traces table     │
    └──────────────────┘

┌─────────────────────────┐
│ SEMANTIC MEMORY         │ (Knowledge)
│                         │
│ • What do we know?      │
│ • How do things work?   │
│ • What patterns exist?  │
│ • Best practices?       │
│                         │
│ Store: Embeddings +     │
│        text             │
│ Retrieve: Semantic      │
│           similarity    │
└────────────┬────────────┘
             │
             ▼
    ┌──────────────────┐
    │ Vector DB        │
    │ (pgvector)       │
    │ embeddings table │
    └──────────────────┘

┌─────────────────────────┐
│ PROCEDURAL MEMORY       │ (Skills)
│                         │
│ • What can we do?       │
│ • How do we do it?      │
│ • What are edge cases?  │
│ • Performance tips?     │
│                         │
│ Store: Skill metadata   │
│        with parameters  │
│ Retrieve: By skill name │
│           or capability │
└────────────┬────────────┘
             │
             ▼
    ┌──────────────────┐
    │ PostgreSQL       │
    │ skills table     │
    │ techniques table │
    └──────────────────┘

┌─────────────────────────┐
│ REFERENCE MEMORY        │ (Obsidian)
│                         │
│ • Markdown files        │
│ • Project docs          │
│ • Brand guidelines      │
│ • SOPs & processes      │
│ • Research notes        │
│                         │
│ Store: File system +    │
│        Git              │
│ Retrieve: Full-text +   │
│           embedding     │
└────────────┬────────────┘
             │
             ▼
    ┌──────────────────┐
    │ File System      │
    │ (~/.ai-os/      │
    │  /obsidian-vault)│
    └──────────────────┘
```

### Memory Query API

```typescript
interface MemoryService {
  // Store
  storeEpisodic(event: EpisodicMemory): Promise<void>;
  storeSemantic(knowledge: SemanticMemory): Promise<void>;
  storeSkill(skill: Skill): Promise<void>;
  
  // Retrieve
  queryByTaskId(taskId: string): Promise<TaskMemory>;
  querySemantic(query: string, limit: number = 10): Promise<SemanticResult[]>;
  querySkills(capability: string): Promise<Skill[]>;
  
  // Search
  semanticSearch(embedding: number[]): Promise<SemanticResult[]>;
  fullTextSearch(query: string): Promise<SearchResult[]>;
  
  // Update
  updateSuccess(taskId: string, success: boolean): Promise<void>;
  addLearning(taskId: string, learning: string): Promise<void>;
  
  // Cleanup
  archive(taskId: string): Promise<void>;
  delete(taskId: string): Promise<void>;
}
```

---

## 6. API Gateway Flow

### LLM Selection Flow (Auto-Switching)

```
Request for LLM inference
       │
       ▼
┌──────────────────────────────────┐
│ Token Budget Check               │
│ • How many tokens do we need?    │
│ • What's our quota?              │
│ • How much is remaining?         │
└────────┬─────────────────────────┘
         │
         ▼
   ┌──────────────────────────────────┐
   │ PRIMARY: Try Ollama (Local)      │
   │                                  │
   │ • Zero cost                      │
   │ • Unlimited tokens               │
   │ • Instant latency                │
   └──────────┬───────────────────────┘
              │
              ├─ Success? → Return result (Cost: $0)
              │
              └─ Fail (model too small, not good enough, error)?
                 │
                 ▼
   ┌──────────────────────────────────┐
   │ FALLBACK 1: Try Groq (Free API)  │
   │                                  │
   │ • 1M tokens/month free           │
   │ • Faster than Ollama             │
   │ • No quota tracking (auto-reset) │
   └──────────┬───────────────────────┘
              │
              ├─ Success? → Return result (Cost: $0)
              │
              └─ Fail (quota exceeded, error)?
                 │
                 ▼
   ┌──────────────────────────────────┐
   │ FALLBACK 2: Try Together (Free)  │
   │                                  │
   │ • High free tier limit           │
   │ • Multiple model support         │
   │ • Good for coding tasks          │
   └──────────┬───────────────────────┘
              │
              ├─ Success? → Return result (Cost: $0)
              │
              └─ Fail (quota, model unavailable)?
                 │
                 ▼
   ┌──────────────────────────────────┐
   │ FALLBACK 3: Try OpenRouter       │
   │                                  │
   │ • Cheapest paid option           │
   │ • Multiple model aggregation     │
   │ • ~$0.01-0.05 per 1k tokens      │
   └──────────┬───────────────────────┘
              │
              ├─ Success? → Return result (Cost: <$0.05)
              │
              └─ Fail (cost limit, network error)?
                 │
                 ▼
   ┌──────────────────────────────────┐
   │ FALLBACK 4: Premium APIs         │
   │                                  │
   │ • Claude (Anthropic) - $0.30+    │
   │ • GPT-4 (OpenAI) - $0.03-0.06    │
   │ • Gemini (Google) - $0.01        │
   │                                  │
   │ Only if:                         │
   │ • Explicitly required            │
   │ • All free options exhausted     │
   │ • High priority task             │
   └──────────┬───────────────────────┘
              │
              ├─ Success? → Return result (Cost: $0.01-1.00)
              │
              └─ All failed? → Error: No available providers

┌────────────────────────────────────────┐
│ RESULT: User never sees complexity     │
│ • Seamless fallback                    │
│ • Transparent cost tracking            │
│ • Optimized for free-first experience  │
└────────────────────────────────────────┘
```

---

## 7. Deployment Architecture

### Local Development

```
Your Laptop (16GB RAM)
│
├─ Docker Compose
│  │
│  ├─ OpenWork Container (Port 3000)
│  │  └─ Next.js + React dashboard
│  │
│  ├─ API Server Container (Port 3001)
│  │  └─ Task router, harness orchestration
│  │
│  ├─ PostgreSQL Container (Port 5432)
│  │  └─ Sessions, tasks, memory metadata
│  │
│  ├─ Redis Container (Port 6379)
│  │  └─ Cache, session state, rate limiting
│  │
│  └─ Ollama Container (Port 11434)
│     └─ Local LLM inference (Qwen 8B, DeepSeek)
│
├─ Local Storage
│  ├─ ~/.ai-os/models/       (Ollama models)
│  ├─ ~/.ai-os/obsidian/     (Knowledge vault)
│  ├─ ~/.ai-os/data/         (PostgreSQL files)
│  └─ ~/.ai-os/cache/        (Redis persistence)
│
└─ External APIs
   ├─ Groq API (token from groq.com)
   ├─ Together AI (token from together.ai)
   ├─ Anthropic (optional, for Claude)
   └─ GitHub API (for repo operations)
```

### Production Deployment (Railway/Fly.io)

```
Production Server ($5-30/month)
│
├─ Kubernetes Cluster (or managed service)
│  │
│  ├─ OpenWork Deployment (3 replicas)
│  │  └─ Horizontal scaling, load balancing
│  │
│  ├─ API Server Deployment (3 replicas)
│  │  └─ Stateless, auto-scaling
│  │
│  ├─ PostgreSQL StatefulSet (1 master + 1 replica)
│  │  └─ Persistent volumes
│  │
│  ├─ Redis StatefulSet (1 master + 1 replica)
│  │  └─ Session replication
│  │
│  └─ Ollama DaemonSet (1 per node)
│     └─ GPU acceleration available
│
├─ External Services
│  ├─ S3-compatible storage (for artifacts)
│  ├─ CloudFlare (CDN + DDoS protection)
│  ├─ Sentry (error tracking)
│  └─ Grafana (monitoring)
│
└─ Scaling Policy
   ├─ CPU > 70% → Add API replica
   ├─ Memory > 80% → Alert, consider upgrade
   └─ API latency > 2s → Scale horizontally
```

---

## Summary Table

| Component | Purpose | Technology | Scaling |
|-----------|---------|-----------|----------|
| OpenWork | Dashboard & UI | Next.js + React | Stateless, scale horizontally |
| API Server | Task routing & orchestration | Node.js/Python | Stateless, scale horizontally |
| PostgreSQL | Sessions, tasks, memory metadata | PostgreSQL 14+ | Vertical scaling, read replicas |
| Redis | Cache, session state, queues | Redis 7+ | Cluster mode for HA |
| Ollama | Local LLM inference | Ollama containers | Vertical (GPU) or distributed |
| Vector DB | Semantic search, embeddings | pgvector extension | Partitioned by user/project |
| File Storage | Obsidian vault, artifacts | S3-compatible or local | Local SSD or cloud storage |

---

## Next Steps

1. Read `02_TECHNOLOGY_STACK.md` for tool selection
2. Read `03_AGENT_HARNESS_ARCHITECTURES.md` for harness details
3. Run `INSTALLATION_GUIDE.md` for local setup
4. See `API_REFERENCE.md` for endpoint documentation
