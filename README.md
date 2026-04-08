# AgenticAIWorkflows

A collection of agentic AI workflow notebooks in a Python **`uv`‑managed** monorepo. Each workflow illustrates a recognizable **design pattern** you can reuse in larger systems.

---

## Project structure

```
AgenticAIWorkflows/
├── src/
│   ├── evaluatorOptimizerWorkflow/
│   │   └── evaluator_optimizer_workflow.ipynb   # Cross-evaluation (Gemini + Ollama)
│   ├── threeAgentDebateLangGraph/
│   │   └── three_agent_debate.ipynb             # LangGraph routing + moderated debate
│   └── humanInTheLoop/
│       └── human_in_the_loop.ipynb                # (placeholder / work in progress)
├── pyproject.toml
├── uv.lock
├── .env                                          # Local secrets — not committed
└── README.md                                     # This file — single source of workflow docs
```

---

## Setup (all workflows)

1. **Install `uv`** (if needed):

   ```bash
   pip install uv
   ```

2. **Install project dependencies** from the repo root:

   ```bash
   uv sync
   ```

3. **Environment variables**  
   Create a `.env` file in the **project root** (copy from `.env.example` if present). Typical keys:

   | Variable | Used by | Purpose |
   |----------|---------|---------|
   | `GEMINI_API_KEY` | Debate notebook, optional for Gemini in evaluator | Google AI Studio API key |
   | `GEMINI_MODEL` | Debate notebook | Model id (e.g. `gemini-2.0-flash`); `models/` prefix is normalized in code where needed |
   | `GEMINI_BASE_URL` | Evaluator notebook (OpenAI-compatible Gemini) | Default points at Google’s OpenAI-compatible endpoint |
   | `OLLAMA_MODEL` | Evaluator notebook | Local model name (e.g. `llama3.2`) |

4. **Run notebooks** from the repo root:

   ```bash
   uv run jupyter lab
   ```

   or

   ```bash
   uv run jupyter notebook
   ```

### Extra steps for the evaluator workflow (Ollama)

Some cells call **Ollama** locally:

```bash
ollama serve
ollama pull llama3.2
```

Use the same model name as `OLLAMA_MODEL` in `.env`.

---

## Workflow design patterns

These names match common “agentic workflow” language (see e.g. Anthropic’s *Building effective agents* and similar overviews). Your implementations map to them like this:

### Evaluator–optimizer (and cross-evaluation)

**Idea:** One component **produces** a candidate (answer, plan, code), and another **scores or critiques** it against rubrics or goals. Tight loops pair generator + evaluator until quality is good enough.

**This repo — evaluator notebook:** It uses a **symmetric cross-evaluation** shape instead of a single optimizing loop:

1. Round A: Model A asks → Model B answers → Model A **evaluates** (numeric score + criteria).
2. Round B: Model B asks → Model A answers → Model B **evaluates** the same way.

So the pattern is still **evaluate-then-score**, but the goal is **comparison and benchmarking** between two stacks (here **Gemini** vs **Ollama**), not iteratively rewriting one artifact until an evaluator says “good.”

**When to use:** A/B evaluation, calibration of prompts, or when you want two systems to stress-test each other with shared grading rubrics.

---

### Routing (dynamic next-step selection)

**Idea:** The workflow **chooses which node or tool runs next** based on state, rules, or model output—instead of a single fixed sequence.

**Kinds of routing:**

| Style | What it looks like |
|-------|-------------------|
| **Rule / DSL routing** | `if state["step"] == "x": go to y` |
| **LLM routing** | Parse model JSON or tool call to pick the next agent or sub-graph |
| **Graph-native routing** | Framework returns an explicit **next node** from a node’s return value |

**This repo — three-agent debate:** Uses **LangGraph** [**`Command`**](https://langchain-ai.github.io/langgraph/) routing from inside nodes:

- Each debater and the moderator returns `Command(goto=<next_node>, update=<partial_state>)`.
- Only **`START → coin_toss`** is a static `add_edge`; the rest of the traversal is **dynamic** (`goto` targets: `for_agent`, `against_agent`, `moderator`, or graph **`END`**).

That is the **routing pattern** in graph form: the moderator (and opening-round debaters) **decide the edge** that executes next by returning `goto`, merged with partial state updates.

**When to use:** Multi-agent turns, debate loops, “human or model decides next step,” or any workflow where the path is not known at compile time.

---

## Workflows

### 1. Evaluator optimizer (`evaluator_optimizer_workflow.ipynb`)

| | |
|---|---|
| **Notebook** | [`src/evaluatorOptimizerWorkflow/evaluator_optimizer_workflow.ipynb`](src/evaluatorOptimizerWorkflow/evaluator_optimizer_workflow.ipynb) |
| **Pattern** | Cross-evaluation / evaluator–optimizer-style **scoring** (symmetric two-round design) |

**Round 1:** Gemini → question → Ollama → answer → Gemini → score (0–100).  
**Round 2:** Ollama → question → Gemini → answer → Ollama → score (0–100).

**Shared rubric (example dimensions):** accuracy, completeness, clarity, depth, relevance — see the notebook’s `GRADING_CRITERIA`.

**Stack notes:**

- Gemini is used through an **OpenAI-compatible client** (`GEMINI_BASE_URL`, `GEMINI_API_KEY` in `.env` as configured in the notebook).
- Ensure **Ollama** is running for the local half of the exercise.

---

### 2. Three-agent debate — LangGraph routing (`three_agent_debate.ipynb`)

| | |
|---|---|
| **Notebook** | [`src/threeAgentDebateLangGraph/three_agent_debate.ipynb`](src/threeAgentDebateLangGraph/three_agent_debate.ipynb) |
| **Pattern** | **Routing** via `Command`; moderated multi-agent loop |

**Routing in this graph**

- **`coin_toss`**, **`for_agent`**, **`against_agent`**, and **`moderator`** each return **`Command(goto=..., update=...)`** so LangGraph knows the **next node** and **state delta** in one step.
- **`moderator`** uses structured output (`ModeratorResponse`) to choose **`for_agent`**, **`against_agent`**, or **end** (`END`). A cap (`MAX_MODERATOR_CALLS`) can force termination; **`FinalVerdictResponse`** then supplies **`for_agent` / `against_agent` / `tie`** so the run does not end **`undecided`** when the cap bites.

**Other design choices**

- **`TypedDict`** state + **`Annotated[list[str], add]`** to append debate turns cleanly.
- **Coin toss** + mandatory **both openings** before first moderation (reduces first-speaker anchoring).
- **Rebuttals** after openings (each side sees the opponent’s latest reply in context).
- At least **one moderator follow-up** before the first normal **end**.

**Env:** `GEMINI_API_KEY`, and optionally `GEMINI_MODEL` (see notebook Step 1 — native `google_genai` init).

#### Debate graph (conceptual)

Static edge: `START → coin_toss`. Other arrows correspond to **`goto`** targets from **`Command`**.

```mermaid
flowchart TD
    S([START]) --> coin_toss[coin_toss]

    coin_toss -->|"random goto"| for_agent[for_agent]
    coin_toss -->|"random goto"| against_agent[against_agent]

    for_agent -->|"opening: goto"| against_agent
    for_agent -->|"else: goto"| moderator[moderator]

    against_agent -->|"opening: goto"| for_agent
    against_agent -->|"else: goto"| moderator

    moderator -->|"goto for_agent"| for_agent
    moderator -->|"goto against_agent"| against_agent
    moderator -->|"goto END"| E([END])
```

On **GitHub**, this Mermaid block renders in the Markdown view. In the editor, use a preview that supports Mermaid.

---

## Dependencies

Shared libraries live in the root **`pyproject.toml`** and **`uv.lock`**. Add new packages there so every workflow stays on one toolchain.

---

## Adding a new workflow

1. Create a folder under `src/` and add your notebook(s).
2. Extend the **Project structure** and **Workflows** sections in **this** `README.md` (avoid per-folder READMEs unless a workflow needs a very long standalone doc).
3. Say which **design pattern** it demonstrates (routing, evaluator–optimizer, orchestration, human-in-the-loop, etc.) so the catalog stays teachable.
