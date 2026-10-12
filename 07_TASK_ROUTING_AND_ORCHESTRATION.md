# Task Routing and Orchestration: The Intelligent Decision Engine

## Table of Contents
1. Routing Algorithm
2. Task Classification
3. Capability Scoring
4. Harness Selection
5. Fallback Logic
6. Learning Loop
7. Real-World Examples

---

## 1. Routing Algorithm (Complete)

### High-Level Flow

```
User Task
    ↓
┌─────────────────────────────────┐
│ 1. PARSE & CLASSIFY             │
│ Extract: intent, entities, tags │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ 2. CAPABILITY SCORING           │
│ Score each available harness    │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ 3. SELECT BEST MATCH            │
│ Pick highest scored harness     │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ 4. CREATE SESSION               │
│ Initialize execution session    │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ 5. EXECUTE WITH MONITORING      │
│ Run task, stream output         │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ 6. COLLECT RESULT               │
│ Store execution trace           │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│ 7. UPDATE MEMORY & LEARN        │
│ Extract insights for next time  │
└─────────────────────────────────┘
```

### Router Implementation (TypeScript)

```typescript
// src/openwork/services/taskRouter.ts

import { TaskRequest, RoutingDecision, HarnessScore } from '../types';

class TaskRouter {
  private classifierModel: NLPClassifier;
  private scorerModel: CapabilityScorer;
  private harnessRegistry: HarnessRegistry;
  private memoryService: MemoryService;
  
  async routeTask(request: TaskRequest): Promise<RoutingDecision> {
    // Step 1: Parse and classify
    const classification = await this.classifyTask(request.description);
    
    // Step 2: Score each harness
    const scores = await this.scoreHarnesses(
      classification,
      request.context
    );
    
    // Step 3: Select best match with fallbacks
    const decision = this.selectBestMatch(scores);
    
    // Step 4: Retrieve relevant memory
    const relevantMemory = await this.memoryService.queryByTaskType(
      classification.taskType
    );
    
    // Step 5: Log decision
    await this.logRoutingDecision(request.taskId, decision);
    
    return {
      taskId: request.taskId,
      selectedHarness: decision.primary,
      alternatives: decision.alternatives,
      scores: scores,
      reasoning: this.generateReasoning(scores, classification),
      relevantMemory: relevantMemory,
      estimatedCost: this.estimateCost(decision.primary, request),
      estimatedDuration: this.estimateDuration(decision.primary, request),
    };
  }
  
  private async classifyTask(description: string): Promise<TaskClassification> {
    // Use NLP model to determine:
    // - Task type (code-gen, research, testing, etc.)
    // - Required capabilities
    // - Complexity level
    // - Special requirements (browser? sandbox? memory?)
    
    const features = await this.extractFeatures(description);
    const prediction = await this.classifierModel.predict(features);
    
    return {
      taskType: prediction.taskType,
      subtype: prediction.subtype,
      requiredCapabilities: prediction.capabilities,
      complexity: prediction.complexity, // 0-100
      requiresBrowser: prediction.needsBrowser,
      requiresSandbox: prediction.needsSandbox,
      requiresMemory: prediction.needsMemory,
      estimatedTokens: prediction.estimatedTokens,
      confidence: prediction.confidence,
    };
  }
  
  private async scoreHarnesses(
    classification: TaskClassification,
    context: any
  ): Promise<HarnessScore[]> {
    const harnesses = await this.harnessRegistry.getAllHarnesses();
    const scores: HarnessScore[] = [];
    
    for (const harness of harnesses) {
      // Get harness capabilities
      const capabilities = await harness.getCapabilities();
      
      // Score against task requirements
      let score = 0;
      
      // Primary capability match (weight: 40%)
      score += this.scoreCapabilityMatch(
        classification.requiredCapabilities,
        capabilities
      ) * 0.4;
      
      // Secondary considerations (weight: 20%)
      score += this.scoreSecondaryFactors(
        classification,
        harness
      ) * 0.2;
      
      // Performance estimation (weight: 20%)
      score += this.scorePerformance(
        classification.complexity,
        harness
      ) * 0.2;
      
      // Historical success rate (weight: 20%)
      score += (await this.getSuccessRate(harness)) * 0.2;
      
      scores.push({
        harness: harness.name,
        score: Math.min(100, score),
        reasoning: this.generateScoreReasoning(
          classification,
          harness,
          score
        ),
      });
    }
    
    return scores.sort((a, b) => b.score - a.score);
  }
  
  private scoreCapabilityMatch(
    required: string[],
    available: string[]
  ): number {
    // Calculate coverage percentage
    const matched = required.filter(r => available.includes(r)).length;
    return (matched / required.length) * 100;
  }
  
  private scoreSecondaryFactors(
    classification: TaskClassification,
    harness: Harness
  ): number {
    let score = 50; // Baseline
    
    // Browser requirement
    if (classification.requiresBrowser && harness.hasBrowser) {
      score += 20;
    }
    
    // Sandbox requirement
    if (classification.requiresSandbox && harness.hasSandbox) {
      score += 15;
    }
    
    // Memory requirement
    if (classification.requiresMemory && harness.hasMemory) {
      score += 15;
    }
    
    return score;
  }
  
  private scorePerformance(
    complexity: number,
    harness: Harness
  ): number {
    // Match complexity to harness capability
    const harnessPower = harness.estimatedCapability; // 0-100
    const diff = Math.abs(complexity - harnessPower);
    
    // Penalize mismatch
    return Math.max(0, 100 - (diff * 0.5));
  }
  
  private selectBestMatch(scores: HarnessScore[]): SelectionDecision {
    // Primary: Highest score
    const primary = scores[0];
    
    // Alternatives: Next 2 harnesses
    const alternatives = scores.slice(1, 3).map(s => s.harness);
    
    return {
      primary: primary.harness,
      primaryScore: primary.score,
      alternatives: alternatives,
      confidence: primary.score / 100, // 0-1
    };
  }
  
  private estimateCost(harness: string, request: TaskRequest): number {
    // Estimate API cost based on:
    // - Harness type
    // - Estimated tokens
    // - External API calls
    
    const harnessBaseCost = {
      'ollama': 0,           // Free local
      'groq': 0,             // Free tier
      'together': 0,         // Free tier
      'openrouter': 0.0001,  // $0.0001 per 1K tokens
      'claude': 0.003,       // $0.003 per 1K tokens (Sonnet)
      'gpt4': 0.03,          // $0.03 per 1K tokens
    };
    
    const baseCost = harnessBaseCost[harness] || 0.001;
    return (request.estimatedTokens / 1000) * baseCost;
  }
  
  private estimateDuration(harness: string, request: TaskRequest): number {
    // Estimate execution time in seconds
    const harnessSpeeds = {
      'ollama': 20,          // tokens/sec (CPU-dependent)
      'groq': 200,           // tokens/sec (very fast)
      'together': 50,        // tokens/sec
      'openrouter': 30,      // tokens/sec
      'claude': 40,          // tokens/sec
    };
    
    const speed = harnessSpeeds[harness] || 50;
    return request.estimatedTokens / speed;
  }
}
```

---

## 2. Task Classification System

### Supported Task Types

```typescript
type TaskType =
  // Coding tasks
  | 'code-generation'
  | 'code-refactoring'
  | 'bug-fixing'
  | 'testing'
  | 'code-review'
  | 'architecture-design'
  
  // Research & analysis
  | 'research'
  | 'data-analysis'
  | 'research-synthesis'
  
  // Automation
  | 'workflow-automation'
  | 'scheduling'
  | 'monitoring'
  
  // Multi-agent
  | 'multi-agent-orchestration'
  | 'team-coordination'
  
  // General
  | 'question-answering'
  | 'content-generation'
  | 'summarization';
```

### Classification Rules (Rule Engine)

```typescript
// src/openwork/services/classifier.ts

const CLASSIFICATION_RULES = [
  {
    pattern: /implement|write|generate|create.*code/i,
    taskType: 'code-generation',
    requiredCapabilities: ['file-access', 'syntax-validation'],
    complexity: 60,
  },
  {
    pattern: /fix|bug|error|debug/i,
    taskType: 'bug-fixing',
    requiredCapabilities: ['file-access', 'shell-exec', 'testing'],
    complexity: 70,
  },
  {
    pattern: /test|unit test|integration test/i,
    taskType: 'testing',
    requiredCapabilities: ['shell-exec', 'file-access'],
    complexity: 50,
  },
  {
    pattern: /research|investigate|find|explore|learn/i,
    taskType: 'research',
    requiredCapabilities: ['browser', 'memory'],
    complexity: 40,
  },
  {
    pattern: /refactor|improve|optimize|clean/i,
    taskType: 'code-refactoring',
    requiredCapabilities: ['file-access', 'git'],
    complexity: 65,
  },
  {
    pattern: /multiple.*tasks|coordinate|orchestrate/i,
    taskType: 'multi-agent-orchestration',
    requiredCapabilities: ['multi-agent', 'memory'],
    complexity: 80,
  },
];
```

---

## 3. Capability Scoring Matrix

### Harness Capabilities

```typescript
// Each harness declares what it can do

interface HarnessCapability {
  name: string;
  version: string;
  capabilities: {
    coding: 95,           // Code generation quality (0-100)
    reasoning: 85,
    memory: 100,          // Memory retention
    browser: 80,
    shell: 90,
    git: 85,
    testing: 70,
    autonomy: 75,         // Can work without supervision
    multiagent: 60,       // Can coordinate with other agents
  };
  constraints: {
    maxContextLength: 8000,  // tokens
    maxExecutionTime: 300,   // seconds
    maxConcurrentTasks: 1,
  };
  successRate: 0.92;        // Historical success %
  avgResponseTime: 2.5;     // seconds
  costPerTask: 0.01;        // dollars
}

// Example: Claude Code capabilities
const CLAUDE_CODE_CAPABILITIES = {
  coding: 98,           // Excellent at coding
  reasoning: 95,        // Strong logical thinking
  memory: 20,           // No persistent memory
  browser: 0,           // Can't browse
  shell: 0,             // Can't execute shell
  git: 70,              // Can understand Git concepts
  testing: 75,          // Can write tests
  autonomy: 40,         // Needs guidance
  multiagent: 60,       // Can collaborate
};

// Example: Hermes capabilities
const HERMES_CAPABILITIES = {
  coding: 60,           // Decent at coding
  reasoning: 70,
  memory: 100,          // Excellent memory
  browser: 95,          // Strong browser automation
  shell: 85,            // Can execute commands
  git: 80,
  testing: 60,
  autonomy: 90,         // Highly autonomous
  multiagent: 95,       // Great at multi-agent
};
```

---

## 4. Fallback Logic

### What Triggers Fallback?

```typescript
type FallbackTrigger =
  | 'timeout'                    // Harness took too long
  | 'token_limit'               // Hit API token limit
  | 'rate_limit'                // API rate limit exceeded
  | 'quality_threshold_not_met'  // Output quality < threshold
  | 'capability_gap'            // Harness can't complete
  | 'network_error'             // Connection failed
  | 'out_of_memory'             // Insufficient resources
  | 'sandbox_error'             // Sandbox execution failed
  | 'user_requested'            // User manually requested fallback
  | 'cost_threshold'            // Exceeded cost budget;

interface FallbackDecision {
  triggerReason: FallbackTrigger;
  originalHarness: string;
  fallbackHarness: string;
  contextTransfer: {
    executionTrace: any[];
    currentState: any;
    errors: string[];
  };
  resumePoint: string; // Where to resume
}
```

### Automatic Fallback Chain

```typescript
// src/openwork/services/fallbackManager.ts

class FallbackManager {
  async handleFallback(
    trigger: FallbackTrigger,
    session: ExecutionSession,
    scores: HarnessScore[]
  ): Promise<FallbackDecision> {
    // Get list of alternative harnesses (pre-ranked)
    const alternatives = scores.slice(1, 5);
    
    // Filter based on trigger
    const viableAlternatives = this.filterAlternatives(
      trigger,
      alternatives,
      session
    );
    
    if (viableAlternatives.length === 0) {
      throw new Error(`No viable fallback for trigger: ${trigger}`);
    }
    
    // Select best alternative
    const fallbackHarness = viableAlternatives[0].harness;
    
    // Transfer context
    const contextTransfer = await this.transferContext(
      session,
      fallbackHarness
    );
    
    // Create new session in fallback harness
    const fallbackSession = await fallbackHarness.resumeTask(
      session.taskId,
      contextTransfer
    );
    
    return {
      triggerReason: trigger,
      originalHarness: session.harness,
      fallbackHarness: fallbackHarness.name,
      contextTransfer: contextTransfer,
      resumePoint: fallbackSession.resumePoint,
    };
  }
  
  private filterAlternatives(
    trigger: FallbackTrigger,
    alternatives: HarnessScore[],
    session: ExecutionSession
  ): HarnessScore[] {
    switch (trigger) {
      case 'timeout':
        // Choose harnesses with faster response time
        return alternatives.filter(alt => 
          alt.harness !== 'ollama' // Ollama might be slow
        );
      
      case 'token_limit':
        // Choose harnesses with better token efficiency
        return alternatives.filter(alt => {
          // Prefer Groq (very fast, no token limit)
          return alt.harness === 'groq' || alt.harness === 'together';
        });
      
      case 'quality_threshold_not_met':
        // Choose harnesses with higher capability scores
        return alternatives.filter(alt => alt.score > 70);
      
      case 'capability_gap':
        // Choose harnesses with required capability
        return alternatives.filter(alt => {
          return session.requiredCapabilities.every(cap => {
            return harness_capabilities[alt.harness][cap] > 50;
          });
        });
      
      case 'cost_threshold':
        // Choose cheaper alternatives
        return alternatives.filter(alt => {
          return this.estimateCost(alt.harness) < session.costBudget;
        });
      
      default:
        return alternatives; // Return all alternatives
    }
  }
  
  private async transferContext(
    session: ExecutionSession,
    targetHarness: Harness
  ): Promise<ContextTransfer> {
    // Compress execution trace
    const compressedTrace = this.compressTrace(session.executionTrace);
    
    // Extract current state
    const currentState = {
      progress: session.progress,
      output: session.output,
      errors: session.errors,
      lastSuccessfulStep: session.lastSuccessfulStep,
    };
    
    // Build context string for new harness
    const contextString = `
      Previous attempt with ${session.harness} failed due to: ${session.lastError}
      
      Progress so far:
      ${compressedTrace}
      
      Current state:
      ${JSON.stringify(currentState, null, 2)}
      
      Please continue from step: ${session.lastSuccessfulStep + 1}
      Focus on: ${session.focusArea}
    `;
    
    return {
      contextString: contextString,
      currentState: currentState,
      errors: session.errors,
      previousHarness: session.harness,
    };
  }
}
```

---

## 5. Learning Loop

### Update Routing Scores Over Time

```typescript
// After task completion, learn from outcome

interface LearningRecord {
  taskId: string;
  classification: TaskClassification;
  selectedHarness: string;
  successRate: boolean;          // Did it succeed?
  executionTime: number;         // Seconds
  costActual: number;            // Actual cost $
  qualityRating: number;         // 0-100 (user feedback)
  suggestions: string[];         // Alternative harnesses?
}

class LearningLoop {
  async updateScoresFromOutcome(
    learningRecord: LearningRecord
  ): Promise<void> {
    // Extract insights
    const insights = {
      harness: learningRecord.selectedHarness,
      taskType: learningRecord.classification.taskType,
      success: learningRecord.successRate,
      efficiency: learningRecord.executionTime,
      cost: learningRecord.costActual,
      quality: learningRecord.qualityRating,
    };
    
    // Update success rate for this harness + task type combo
    await this.updateSuccessRate(
      learningRecord.selectedHarness,
      learningRecord.classification.taskType,
      learningRecord.successRate
    );
    
    // Update average execution time
    await this.updateAvgExecutionTime(
      learningRecord.selectedHarness,
      learningRecord.executionTime
    );
    
    // Update cost estimates
    await this.updateCostEstimate(
      learningRecord.selectedHarness,
      learningRecord.costActual
    );
    
    // Extract skills learned
    if (learningRecord.successRate) {
      const newSkills = await this.extractNewSkills(
        learningRecord.taskId
      );
      await this.memoryService.storeSkills(newSkills);
    }
  }
}
```

---

## 6. Real-World Examples

### Example 1: Simple Code Generation

```
Request: "Write a Python function to calculate factorial"

Classification:
- taskType: code-generation
- complexity: 30 (easy)
- requiredCapabilities: [syntax-validation]
- estimatedTokens: 500

Scoring:
- Qwen 8B: 92 (local, fast)
- DeepSeek 6.7B: 88 (local)
- Groq: 85 (cloud, slower for simple task)
- Claude: 95 (overkill, too expensive)

Decision:
→ Selected: Qwen 8B (local, free, fast)
→ Fallback 1: DeepSeek 6.7B
→ Fallback 2: Groq
→ Cost: $0
→ Duration: ~5 seconds

Result: Function generated, stored in memory
```

### Example 2: Complex Research Task

```
Request: "Research the latest AI safety measures and summarize key findings"

Classification:
- taskType: research
- complexity: 75 (complex)
- requiredCapabilities: [browser, memory, synthesis]
- estimatedTokens: 8000

Scoring:
- Hermes: 98 (excellent browser + memory)
- Groq: 85 (fast but no memory)
- Claude: 92 (good reasoning, no memory)
- Qwen 8B: 70 (lacks browser)

Decision:
→ Selected: Hermes
→ Fallback 1: Claude
→ Fallback 2: Groq
→ Cost: $0 (using Hermes with local memory)
→ Duration: ~30 seconds

Result: Research + findings + links stored in memory vault
```

### Example 3: Multi-Step Workflow

```
Request: "Analyze my codebase, fix bugs, run tests, create PR"

Classification:
- taskType: multi-agent-orchestration
- complexity: 90 (very complex)
- requiredCapabilities: [file-access, git, testing, shell]
- estimatedTokens: 15000

Scoring:
- DSH: 96 (designed for orchestration)
- OpenHands: 94 (autonomous)
- Claude Code: 80 (no multi-agent)
- Hermes: 85 (some orchestration)

Decision:
→ Selected: DSH
→ Fallback 1: OpenHands
→ Fallback 2: Hermes + Claude Code (sequential)
→ Cost: $0-2 (might need Groq overflow)
→ Duration: ~2 minutes

Result: Full workflow completed, results in session
```

---

## Summary

The routing system is the **intelligence** of the AI OS. It:

1. **Understands** what the user wants (classification)
2. **Scores** which tool is best (capability matching)
3. **Selects** the optimal choice (with alternatives)
4. **Executes** transparently (real-time monitoring)
5. **Learns** from outcomes (updates decision weights)

Result: User gets best tool for every task, automatically, without manual intervention.
