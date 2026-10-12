# Specialization Architecture: Agent Design by Task Type

## Objective

A modern AI OS should route each task to the agent that is best suited for it. Specialization is more important than monolithic generalism.

---

## 1. Specialization Principle

Important rule:

Not every task should go to the same harness.

Examples:
- Writing code → Claude Code or OpenHands
- Research and memory → Hermes
- Browser tasks → Browser harness
- Workflow orchestration → DSH
- Autonomous repo execution → OpenHands

---

## 2. Task-to-Agent Matrix

| Task category | Best harness | Why |
|---|---|---|
| Feature implementation | Claude Code | Best coding quality |
| Refactoring | Claude Code | Design and structure |
| Bug fixing in repo | OpenHands | Terminal + tests + sandbox |
| Research synthesis | Hermes | Browser + memory |
| Long-running personal ops | Hermes | Memory + automation |
| Browser extraction | Browser agent | Browser-first |
| Multi-agent orchestration | DSH | Plugin-first orchestration |
| Full repo work | OpenHands | Autonomous task execution |
| Documentation generation | Claude Code | Clear writing and structure |
| Workflow design | DSH | Runtime composition |
| Memory-backed analysis | Hermes | Persistent context |

---

## 3. Specialized Agent Profiles

### Engineering agent
- Inputs: code tasks, feature work, refactor requests
- Primary harness: Claude Code
- Secondary: OpenHands
- Key outputs: code patches, PR descriptions, test plans

### Research agent
- Inputs: web investigation, learning tasks, synthesis tasks
- Primary harness: Hermes
- Secondary: DSH or browser agents
- Key outputs: findings, reports, summaries, references

### Autonomous software agent
- Inputs: repo debugging, testing, feature completion, repo-level tasks
- Primary harness: OpenHands
- Secondary: Claude Code
- Key outputs: working code, tests, final status

### Workflow orchestration agent
- Inputs: multi-step tasks, workflows, automation, plugin tasks
- Primary harness: DSH
- Secondary: Hermes
- Key outputs: orchestrated pipeline results, workflows, sessions

### Browser agent
- Inputs: website interaction, extraction, UI automation
- Primary harness: Browser harness
- Secondary: Hermes
- Key outputs: structured data, automation results

---

## 4. Route Selection Logic

```typescript
function selectHarness(task: TaskDescriptor): string {
  if (task.category === 'research' || task.requiresBrowser) {
    return 'hermes';
  }

  if (task.category === 'coding' && task.complexity < 70) {
    return 'claude-code';
  }

  if (task.category === 'bugs' || task.requiresSandbox) {
    return 'openhands';
  }

  if (task.category === 'workflow' || task.requiresOrchestration) {
    return 'dsh';
  }

  if (task.category === 'web-extraction') {
    return 'browser-harness';
  }

  return 'hermes';
}
```

---

## 5. Cross-Harness Coordination

A real AI OS should allow harnesses to share information.

Example:
- Hermes researches a competitor and stores result in memory
- Claude Code reads the memory and uses it to design a better feature
- OpenHands validates the implementation in sandbox
- DSH keeps the session trace and orchestrates the flow

This is how the system becomes greater than the sum of its parts.

---

## 6. Practical Design Rule

The system should never be a single-model abstraction. It should be a team of specialists under one roof.

The operating logic is:
- detect task
- understand requirements
- route to correct specialist
- provide shared memory and tool access
- collect result and update memory

---

## 7. Summary

The Personal AI OS should be designed as a managed fleet of specialists:

- research specialist = Hermes
- coding specialist = Claude Code
- repo execution specialist = OpenHands
- orchestration specialist = DSH
- web interaction specialist = browser agents

This produces a system that behaves like a real operating environment, not like a single chatbot wrapper.
