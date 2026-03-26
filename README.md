# Memstate AI Memory Benchmark

An open-source benchmark for evaluating AI agent memory systems in real-world, multi-session coding scenarios.

**Blog post:** [The Challenges of Building a Fair AI Memory Benchmark](https://memstate.ai/blog/building-a-fair-ai-memory-benchmark)

**Live leaderboard:** [memstate.ai/docs/leaderboard](https://memstate.ai/docs/leaderboard)

---

## Why This Benchmark Exists

Most AI memory benchmarks test trivial single-turn recall: "The user's favorite color is blue. What is the user's favorite color?"

Real software engineering is nothing like that. Decisions evolve. Teams change their minds. Architecture gets refactored. A useful memory system for AI coding agents must handle non-linear decision histories, detect conflicts when facts change, and always return the *current* state, not a blend of past states.

This benchmark was built to test exactly that.

---

## Design Principles

**1. Level Playing Field**

Every memory system under test gets the same advantages:
- A custom system prompt prefix explaining how to use its specific MCP tools
- The same state-of-the-art LLM (Claude Opus 4.6 / Gemini 3)
- The same 5 coding scenarios
- The same evaluation criteria

**2. Real Coding Scenarios**

We use 5 realistic multi-session coding tasks, each spanning 3-4 sessions:

| Scenario | Description | Sessions |
|---|---|---|
| Web App Architecture Evolution | A project migrates from REST to GraphQL, then React to Next.js | 4 |
| Auth System Migration | JWT auth evolves through multiple security upgrades | 3 |
| Database Schema Evolution | Schema normalizes, then denormalizes for performance | 3 |
| API Versioning Conflicts | API versioning strategy changes across the project lifecycle | 3 |
| Team Decision Reversal | Architecture goes Monolith → Microservices → Monolith | 4 |

**3. Multi-Dimensional Scoring**

Each scenario is evaluated across 4 dimensions:

| Metric | What It Tests |
|---|---|
| Fact Recall | Can the agent retrieve a specific fact from memory? |
| Conflict Detection | Does the agent know when a fact has been superseded? |
| Decision Tracking | Can the agent explain the full history of a decision? |
| Context Continuity | Does the agent pick up exactly where the last session left off? |

---

## Key Finding: Ingestion Speed

During early benchmark runs, we let the coding agents move as fast as they wanted. This exposed a critical vulnerability in some memory systems: **ingestion latency**.

When an agent finishes a task and immediately starts the next one, it queries memory for context. If the previous session's memories have not finished being indexed yet, the agent gets nothing back and proceeds without context.

Memstate AI's custom-trained fact extraction models are optimized for speed. Memories are ingested and available for recall in milliseconds. This turned out to be a major differentiator in the benchmark results.

---

## Results

| System | Overall Score | Accuracy | Conflict Detection | Continuity |
|---|---|---|---|---|
| **Memstate AI** | **84.4%** | 92.2% | 95.0% | 88.7% |
| Naive RAG (Vector) | 38.9% | 45.1% | 12.3% | 41.2% |
| Mem0 | 20.4% | 24.5% | 0.0% | 18.2% |
| No Memory (baseline) | 12.1% | 14.3% | 0.0% | 11.8% |

Full results and methodology: [memstate.ai/docs/benchmarks](https://memstate.ai/docs/benchmarks)

---

## Running the Benchmark

### Prerequisites

- Node.js 18+
- An Anthropic API key (Claude Opus 4.6 recommended)
- API keys for the memory systems you want to test

### Install

```bash
git clone https://github.com/memstate-ai/memstate-benchmark
cd memstate-benchmark
npm install
```

### Run against Memstate

```bash
export ANTHROPIC_API_KEY=your-key
export MEMSTATE_API_KEY=your-memstate-key
npx memory-benchmark run -a memstate
```

### Run against Mem0

```bash
export ANTHROPIC_API_KEY=your-key
export MEM0_API_KEY=your-mem0-key
npx memory-benchmark run -a mem0
```

### Run a head-to-head comparison

```bash
npx memory-benchmark run -a memstate,mem0
```

### Run a specific scenario only

```bash
npx memory-benchmark run -a memstate --scenario team-decision-reversal
```

---

## Output

Results are saved to the `output/` directory:

| File | Contents |
|---|---|
| `<adapter>-<timestamp>.json` | Full JSON with all agent turns, tool calls, token counts, scores |
| `<adapter>-<timestamp>.md` | Human-readable markdown report |
| `comparison-<timestamp>.md` | Head-to-head comparison (when multiple adapters) |
| `latest-results.json` | Index pointing to the most recent run |

---

## Custom Adapters

To benchmark a memory system not in the presets, create a JSON config file:

```json
{
  "name": "MyMemory",
  "command": "npx",
  "args": ["-y", "my-memory-mcp-server"],
  "env": {
    "MY_MEMORY_API_KEY": "your-key"
  },
  "toolMapping": {
    "store": "my_store_tool",
    "get": "my_get_tool",
    "search": "my_search_tool"
  }
}
```

Save as `<name>-adapter.json` and run:

```bash
npx memory-benchmark run -a mymemory
```

---

## Interpreting Results

- **90-100:** Excellent — near-perfect recall, conflict detection, and continuity
- **70-89:** Good — solid performance with some gaps in edge cases
- **50-69:** Fair — basic recall works but struggles with contradictions
- **30-49:** Poor — frequently reports stale facts or misses conflicts
- **0-29:** Failing — fundamental memory storage/retrieval issues

**What to look for:**

If accuracy is high but conflict detection is 0%, the system can store facts but cannot track when they change. This is the most common failure mode for vector-based memory systems.

---

## Contributing

To add a new scenario:

1. Create a file in `src/scenarios/`
2. Define sessions with clear prompts and verification queries
3. Add it to `src/scenarios/index.ts`
4. Ensure verification queries cover all 4 types: `fact_recall`, `conflict_detection`, `decision_tracking`, `context_continuity`

---

## Related

- [Blog: The Challenges of Building a Fair AI Memory Benchmark](https://memstate.ai/blog/building-a-fair-ai-memory-benchmark)
- [Blog: Why Vector RAG Fails for AI Agent Memory](https://memstate.ai/blog/why-vector-rag-fails-for-agent-memory)
- [Memstate AI](https://memstate.ai)
- [Memstate MCP on npm](https://www.npmjs.com/package/@memstate/mcp)
