# Agent Frameworks: Comparative Architecture and Recommended Stack

## Objective

This document compares the major agent frameworks and clarifies which should be used, which should be avoided in the base system, and which should be integrated only in specific phases.

---

## 1. Framework Comparison

### CrewAI

**Strengths**
- Simple agent orchestration model
- Good for beginner workflows
- Easy to prototype

**Weaknesses**
- Limited compared with modern orchestration runtimes
- Less flexible than a full plugin-based system
- More opinionated and less scalable for custom runtime growth

**Recommendation**
Use only for lightweight prototypes or experiments, not as a core runtime in the production AI OS.

---

### AutoGen

**Strengths**
- Strong conversational multi-agent patterns
- Good for agent-to-agent planning and tools
- Good research and experimentation

**Weaknesses**
- Heavier than needed for a personal AI OS
- More complex to manage in a lightweight local environment
- Harder to integrate cleanly with a full orchestration and routing engine

**Recommendation**
Interesting for experiments, but not the primary production runtime unless you need advanced multi-agent conversation patterns.

---

### PydanticAI

**Strengths**
- Good typed agent design
- Strong validation and structured outputs
- Good for control and reliability

**Weaknesses**
- More of a typed agent framework than full OS orchestration runtime
- Doesn’t replace DSH or OpenHands as the operating system layer

**Recommendation**
Useful for specific typed workflows, but not the main architecture of the OS.

---

### Mastra

**Strengths**
- Good JavaScript ecosystem integration
- Useful for UI-driven agent flows
- Good when the project is built around the JS stack

**Weaknesses**
- Not necessary for a local-first AI OS architecture
- Additional complexity for a system that already has OpenWork, DSH, and other orchestration layers

**Recommendation**
Optional. Keep it outside the core design unless a TypeScript-native workflow specifically needs it.

---

## 2. Recommended Final Framework Stack

The final architecture should remain intentionally minimal:

- OpenWork — control plane / task router
- OpenHands — autonomous software execution
- Claude Code — coding specialist
- Hermes — memory, research, browser, ops
- DSH — orchestration and plugin runtime
- Browser agents — browser-specific execution

This keeps the system modular and avoids framework sprawl.

---

## 3. Why the Core Stack is Better Than a Framework Stack Explosion

### Benefits of a focused runtime
- Lower maintenance overhead
- Easier debugging
- Cleaner routing logic
- Lower provider complexity
- Better cost control
- Easier to document and reason about

### Problems with too many frameworks
- Hard to debug which layer failed
- Multi-framework duplication of capability
- More API complexity
- Duplicated memory systems
- Higher cognitive load

---

## 4. Recommended Implementation Strategy

### Phase 1 — Foundation
- OpenWork
- DSH
- Browser agents
- Ollama local models
- LiteLLM or direct provider routing

### Phase 2 — Execution specialization
- Claude Code integration
- OpenHands integration
- Hermes research workflows

### Phase 3 — Memory and optimization
- PostgreSQL + pgvector
- Obsidian sync
- RAG pipeline
- fallback and cost optimization

---

## 5. Decision Rule

Use a framework only when it adds clear architectural value.

If a framework does not materially improve:
- orchestration
- routing
- memory
- capability composition
- observability

then it should not be in the core runtime.

---

## 6. Final Recommendation

Recommended final stack:

- OpenWork — central orchestration UI
- DSH — workflow runtime
- Hermes — memory + research + ops
- Claude Code — engineering tasks
- OpenHands — autonomous repo execution
- Browser agents — web automation and web tasks
- Ollama — local first inference
- Groq/Together/OpenRouter — free/cloud fallback chain

This is the cleanest, most scalable, most developer-friendly architecture for a personal AI OS.
