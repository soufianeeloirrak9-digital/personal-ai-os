# Agent Harness Architectures: Detailed Specifications

## Objective

This document explains the exact role of each harness in the Personal AI OS. The system routes tasks to the best-suited agent based on capability, cost, constraints, and performance.

---

## 1. OpenWork — Experience Layer and Control Plane

### Role
OpenWork is the user-facing operating layer. It acts as the task router, workspace shell, session monitor, and orchestrator.

### Core responsibilities
- Central dashboard for tasks and sessions
- Task classification and routing
- Capability registry for all harnesses
- Real-time monitoring of execution state
- Shared memory and skills access
- API gateway and model fallback logic

### Why OpenWork matters
OpenWork gives the AI OS a single interface. The user does not decide which tool to use manually; the system decides automatically.

### Best fit
- General use case entrypoint
- Operations orchestration
- Workspace coordination across multiple agents

---

## 2. Claude Code — Engineering Specialist

### Profile
- Best at software engineering and implementation
- Very strong coding, architecture, debugging, refactoring
- Less suited for autonomous long-running browser tasks

### Strengths
- Code generation quality
- Architecture reasoning
- Refactoring decisions
- Pull request style output
- Technical writing and good code design

### Weaknesses
- Limited persistent memory by default
- Not ideal for browser automation
- Not ideal for long autonomous repo tasks
- Not best for orchestration-heavy workflow patterns

### Best use cases
- Implementing features from scratch
- Refactoring a module
- Writing code based on repo context
- Designing backend APIs and services
- Producing clean code with high quality

### Routing rule
Use Claude Code when the task is primarily engineering and code correctness is the highest priority.

---

## 3. Hermes — AI Operations and Research Agent

### Profile
Hermes is the research and operating intelligence layer. It is memory-heavy and strong in browser tasks and long-running operational workflows.

### Strengths
- Strong memory and context retention
- Good at research and synthesis
- Browser automation and web interaction
- Background operations
- Multi-agent coordination
- Scheduling and long-running work

### Weaknesses
- Not always the best at complex code generation compared with Claude Code
- Some tasks need a code-specialist harness for clean implementation

### Best use cases
- Research tasks
- Data gathering from the internet
- Browser workflows
- Memory-based long-term assistants
- Workflow execution across tools and services

### Routing rule
Use Hermes when the task requires browsing, memory, research, or multi-step operation handling.

---

## 4. OpenHands — Autonomous Software Agent

### Profile
OpenHands is the autonomous software executor. It is excellent for debugging, repo exploration, test suite work, and sandbox-based execution.

### Strengths
- Autonomy in repo tasks
- Shell and repo operations
- Test execution and validation
- Bug fixing in a sandbox
- Long-running software execution
- Strong fit for end-to-end software handling

### Weaknesses
- More solver-like than elegantly designed
- May need guidance in high-level architecture tasks
- Browser tasks are possible but not its core identity

### Best use cases
- Bug fixing across multiple files
- Local repo testing
- Autonomous software tasks
- Branch-level work and validation
- Running terminal-driven tasks

### Routing rule
Use OpenHands when the task involves autonomous software work, repo execution, and test verification.

---

## 5. DeepSeek Harness (DSH) — Orchestration Engine

### Profile
DSH is the orchestration and plugin-driven runtime. It is the correct choice when you need workflow orchestration, custom agents, tool composition, and extensibility.

### Strengths
- Plugin architecture
- Multi-step orchestration
- Session traces and replayability
- Custom agent creation
- Infrastructure and runtime composition
- Strong system-integration posture

### Weaknesses
- Not the best default choice for a simple coding task
- Lower-quality UX if used for everything

### Best use cases
- Building custom agents
- Orchestrating multiple agents
- Complex workflow pipelines
- Tool runtime architecture
- Building reusable agent systems

### Routing rule
Use DSH for orchestration and custom agent construction, not for basic coding if there is a more specialized harness.

---

## 6. Browser Agents — Web Execution Harnesses

This category includes browser-use, browser-harness, browsercode, and high-speed alternatives.

### Common strengths
- Browser navigation
- Form filling and interaction
- Dynamic web extraction
- Data gathering from websites
- UI automation

### Best use cases
- Research via web pages
- Automating SaaS tasks
- Extracting product data
- Web-based task completion

### Best router rule
Route such tasks to a browser harness when the task is about interacting with websites rather than local code.

---

## 7. Harness Routing Matrix

| Task type | Primary harness | Backup harness | Reason |
|---|---|---|---|
| Code generation | Claude Code | OpenHands | Best coding quality |
| Architecture design | Claude Code | DSH | Strong reasoning and structure |
| Bug fixing | OpenHands | Claude Code | Good repo/test sandboxing |
| Repository work | OpenHands | Claude Code | Real execution + tests |
| Research and notes | Hermes | Claude Code | Browser and memory support |
| Browser automation | Browser harness | Hermes | Web interaction |
| Multi-step orchestration | DSH | Hermes | Design for workflow composition |
| Long-running personal ops | Hermes | DSH | Memory and operations |
| Multi-agent system building | DSH | Hermes | Plugin-first orchestration |

---

## 8. Shared Capability Layer

All harnesses should connect to a shared capability layer:

- MCP protocol
- File system access
- GitHub repo access
- Shell execution
- Browser automation
- Memory retrieval
- API gateway
- Logging and observability

This ensures each harness behaves like a specialized tool within one operating system rather than a standalone system.

---

## 9. Conclusion

The Personal AI OS is not built around a single model. It is built around a fleet of specialized agents. Each one has a clear role and all are coordinated through OpenWork.

This separation is the key to making the system:
- stable
- extensible
- low-cost
- robust
- production-grade

This blueprint is the foundation for the operating system architecture.
