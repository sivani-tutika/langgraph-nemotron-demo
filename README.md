# LangGraph Multi-Agent Demo · Powered by NVIDIA Nemotron 3 Super

> A browser-native implementation of the LangGraph multi-agent orchestration pattern, powered by NVIDIA Nemotron 3 Super (120B MoE). Built the day of the LangChain × NVIDIA platform announcement. Zero dependencies. Zero build step. Works by opening a file.

![Demo preview](preview.png)

---

## Live Demo

👉 **[Try it on GitHub Pages](https://YOUR_USERNAME.github.io/langgraph-nemotron-demo)**

Needs a free OpenRouter key → [openrouter.ai/keys](https://openrouter.ai/keys) (no credit card, Nemotron 3 Super is $0/token on free tier)

---

## What it does

Type any query and watch a 4-node agent pipeline run in real time — each node powered by actual Nemotron 3 Super inference:

```
🗺 PLANNER  →  🔍 RESEARCHER  →  ⚖️ CRITIC  →  ✍️ WRITER
```

Each agent receives the **accumulated outputs of all prior agents** as context — this is the core state-passing mechanism that LangGraph implements in production Python environments, expressed here as a minimal JS loop.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Browser (HTML/JS)                         │
│                                                                  │
│   User Query                                                     │
│       │                                                          │
│       ▼                                                          │
│  ┌─────────┐    state     ┌────────────┐    state               │
│  │ PLANNER │ ──────────►  │ RESEARCHER │ ──────────►  ...       │
│  │  node 1 │              │   node 2   │                        │
│  └─────────┘              └────────────┘                        │
│       │                        │                                 │
│       │    outputs[] grows with each node                        │
│       │                        │                                 │
│       ▼                        ▼                                 │
│  ┌─────────┐              ┌────────────┐                        │
│  │  CRITIC │ ──────────►  │   WRITER   │ ──► Final answer       │
│  │  node 3 │    state     │   node 4   │                        │
│  └─────────┘              └────────────┘                        │
│                                                                  │
│                  Each node calls:                                │
│                                                                  │
│         POST openrouter.ai/api/v1/chat/completions              │
│                model: nvidia/nemotron-3-super-120b-a12b:free    │
└─────────────────────────────────────────────────────────────────┘
```

### State accumulation (the core LangGraph concept)

In LangGraph's Python SDK, graph state is a typed `TypedDict` that gets updated as nodes execute. In this demo, it's the `outputs[]` array that grows with each node:

```js
// Agent 0 (Planner) sees only the query
// Agent 1+ sees query + all prior outputs
const priorContext = outputs
  .map((o, j) => `[${AGENTS[j].label}]\n${o}`)
  .join('\n\n');

const userMsg = originalQuery + priorContext;
// ↑ This is the graph state being forwarded
```

This is why the Writer produces a synthesised answer rather than just responding to the raw query — it has the plan, research, and critique all available.

---

## File structure

```
langgraph-nemotron-demo/
├── langgraph-nemotron-demo.html   # The entire app — one file
└── README.md
```

That's it. No `node_modules`. No `package.json`. No build step.

---

## Code structure (inside the HTML)

| Section | What it does |
|---|---|
| `:root` CSS variables | All colours in one place — NVIDIA green `#76b900` as accent |
| `AGENTS[]` array | The only thing you edit to change agent behaviour — 4 objects, each with a `system` prompt |
| `resetNodes / activateNode / doneNode` | CSS class toggling — all animation is CSS, JS just switches classes |
| `callNemotron()` | Single `fetch` to OpenRouter's OpenAI-compatible endpoint |
| `runPipeline()` | Sequential `async` loop — `outputs[]` grows, state flows forward |

---

## Run it locally

```bash
# 1. Clone
git clone https://github.com/YOUR_USERNAME/langgraph-nemotron-demo.git
cd langgraph-nemotron-demo

# 2. Open (literally just open the file)
open langgraph-nemotron-demo.html      # macOS
start langgraph-nemotron-demo.html     # Windows
xdg-open langgraph-nemotron-demo.html  # Linux

# 3. Get a free OpenRouter key
# → https://openrouter.ai/keys
# Paste it into the key field in the UI
```

---

## Extending it

### Add a 5th agent

```js
// In the AGENTS array, add:
{
  id: 'n-summariser',
  edge: 'e-4',           // add matching <div class="edge" id="e-4"> in HTML
  icon: '📋',
  label: 'SUMMARISER',
  system: `You are a Summariser. Distill the Writer's output
into 2 bullet points for an executive audience. Max 40 words.`
}
```

### Add parallel execution (mirrors NVIDIA's speculative execution)

```js
// Instead of sequential await per agent,
// run independent nodes concurrently:
const [researchResult, factCheckResult] = await Promise.all([
  callNemotron(apiKey, researcherSystem, query),
  callNemotron(apiKey, factCheckerSystem, query),
]);
// Then pass both results to the next dependent node
```

### Swap to a different model

```js
// Change one line in callNemotron():
model: 'nvidia/nemotron-3-nano-8b-instruct:free'   // faster, smaller
model: 'meta-llama/llama-3.3-70b-instruct:free'    // alternative free model
model: 'nvidia/nemotron-3-ultra-253b-v1:free'       // more powerful (when available)
```

---

## How this connects to the LangChain × NVIDIA announcement

| What this demo has | What the full platform adds |
|---|---|
| Nemotron 3 Super inference | Nemotron Nano / Super / Ultra via NVIDIA NIM (2.6× throughput) |
| LangGraph-pattern sequential graph | LangGraph Python runtime with typed state, conditional edges, cycles |
| Sequential node execution | Parallel + speculative execution (compile-time NVIDIA optimisation) |
| In-browser trace panel | LangSmith — 15B+ traces, AI-powered debugging, cost monitoring |
| OpenRouter gateway | NVIDIA NIM microservices — cloud, on-prem, hybrid |

This demo is the concept proof. The announcement is the production system.

---

## Stack

- **Model** — NVIDIA Nemotron 3 Super · 120B params · 12B active (MoE)
- **API** — [OpenRouter](https://openrouter.ai) · OpenAI-compatible · free tier
- **Frontend** — Vanilla HTML / CSS / JS · no framework · no build step
- **Fonts** — JetBrains Mono + Syne via Google Fonts

---

## Roadmap

- [ ] Conditional routing (branching edges — LangGraph's real power)
- [ ] Parallel node execution with `Promise.all()`
- [ ] LangSmith-compatible trace export (JSON)
- [ ] Local NVIDIA NIM deployment instructions
- [ ] Streaming token output per agent

---

## Related

- [LangGraph](https://github.com/langchain-ai/langgraph) — the Python framework this is based on
- [LangSmith](https://smith.langchain.com) — production observability
- [NVIDIA Nemotron](https://developer.nvidia.com/nemotron) — the model family
- [NVIDIA NeMo Agent Toolkit](https://github.com/NVIDIA/NeMo-Agent-Toolkit)
- [OpenRouter Nemotron 3 Super](https://openrouter.ai/nvidia/nemotron-3-super-120b-a12b:free)

---

## License

MIT — fork it, remix it, add your own agents.
