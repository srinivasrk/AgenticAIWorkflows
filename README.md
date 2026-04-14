# 🤖 AgenticAIWorkflows

> **Notebooks · design patterns · LangGraph · learn by running**

Welcome. This repo is a **small playground** for agentic workflows: runnable Jupyter notebooks, one **design pattern** at a time. Think of it as a set of **recipes** you can copy into bigger systems—not a framework, not a course website, just code plus enough explanation that future-you remembers *why* it’s shaped that way.

Everything is tied together with **`uv`**, so you spend less time fighting Python environments and more time watching graphs and models do interesting things.

---

## 🎨 Pattern cheat sheet (at a glance)

| | Pattern | What it *feels* like | Where it lives in this repo |
|---|--------|------------------------|------------------------------|
| ⚖️ | ![Evaluator](https://img.shields.io/badge/Evaluator–optimizer-cross--check%20%26%20score-E74C3C?style=flat-square) | Two roles: **produce** vs **judge** (here: symmetric **A grades B, B grades A**) | **Lab 1** — evaluator notebook |
| 🔀 | ![Routing](https://img.shields.io/badge/Routing-dynamic%20next%20step-6C5CE7?style=flat-square) | The path **isn’t** fixed; something picks **what runs next** | **Lab 2** — debate + `Command` |
| 🎭 | ![Orchestrator](https://img.shields.io/badge/Orchestrator–worker-parallel%20then%20merge-00B894?style=flat-square) | One **coordinator**, many **workers** at once, then **one** merge | **Lab 3** — orchestrator notebook |
| ✋ | ![Human in the loop](https://img.shields.io/badge/Human--in--the--loop-pause%20%26%20resume-FD7E14?style=flat-square) | **Checkpoint** + **`interrupt()`**; human approves or edits, then **`Command(resume=...)`** | **Lab 4** — human-in-the-loop notebook |
| 🌳 | ![Hierarchy](https://img.shields.io/badge/Hierarchical%20decomposition-concept-0984E3?style=flat-square) | **Parent** hands a **slice** of work to a **child**, waits, **continues** | *Concept* (see primer + Mermaid below) |

<details>
<summary><strong>🧭 Jump to a lab (colored badges)</strong></summary>

[![Evaluator–optimizer](https://img.shields.io/badge/Lab-Evaluator–optimizer-E74C3C?style=for-the-badge)](src/evaluatorOptimizerWorkflow/evaluator_optimizer_workflow.ipynb)  
[![Routing](https://img.shields.io/badge/Lab-Debate%20%2B%20routing-6C5CE7?style=for-the-badge)](src/threeAgentDebateLangGraph/three_agent_debate.ipynb)  
[![Orchestrator–worker](https://img.shields.io/badge/Lab-Orchestrator–worker-00B894?style=for-the-badge)](src/orchastratorWorkerPattern/orchastrator_worker_pattern.ipynb)  
[![Human-in-the-loop](https://img.shields.io/badge/Lab-Human--in--the--loop-FD7E14?style=for-the-badge)](src/humanInTheLoop/human_in_the_loop.ipynb)  
[![Hierarchical primer](https://img.shields.io/badge/Read-Hierarchical%20primer-0984E3?style=for-the-badge)](#hierarchical-decomposition-parent-delegates-child-delivers)

</details>

---

## 🎓 What you’ll learn here

By the time you’ve run the notebooks (in any order you like), you should be able to:

- **Name** a few standard agentic patterns—evaluator loops, routing, orchestrator–worker, human-in-the-loop pauses, hierarchical handoffs—and **point to working code** for several of them.
- **Read a LangGraph** sketch and tell whether the next step is **fixed** (edges) or **chosen at runtime** (`Command` / `goto`).
- **Use interrupts** where it matters: **`interrupt()`** in a tool, **`__interrupt__`** in the result, then **`Command(resume=...)`** on the **same `thread_id`** (see [LangGraph interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)).
- **Wire API keys** once in `.env` and reuse the same stack across notebooks.

If a section feels dense, skip to **Hands-on notebooks** below, run something, then circle back. That’s a perfectly valid tutorial path.

---

## 🗺️ Map of the repo

```
AgenticAIWorkflows/
├── src/
│   ├── evaluatorOptimizerWorkflow/
│   │   └── evaluator_optimizer_workflow.ipynb   # Two models grade each other (Gemini + Ollama)
│   ├── threeAgentDebateLangGraph/
│   │   └── three_agent_debate.ipynb             # LangGraph + moderator + debaters
│   ├── orchastratorWorkerPattern/
│   │   └── orchastrator_worker_pattern.ipynb    # Orchestrator → parallel writers → merge
│   └── humanInTheLoop/
│       └── human_in_the_loop.ipynb              # Lab 4: StateGraph + ToolNode; interrupt() before send_email
├── pyproject.toml
├── uv.lock
├── .env                                          # Your secrets — never commit this
└── README.md                                     # You are here — the main “syllabus”
```

---

## ⚙️ Before you run anything: setup

Treat this as **Lesson 0**. Once it works, every notebook is just “open and Run All (mindfully).”

### 1️⃣ Install `uv` (one-time)

```bash
pip install uv
```

### 2️⃣ Install dependencies from the repo root

```bash
uv sync
```

### 3️⃣ Teach the project your API keys

Create a **`.env`** file in the **project root** (copy `.env.example` if the repo has one). Here’s what usually matters:

| Variable | Shows up in | What it’s for |
|----------|-------------|----------------|
| `GEMINI_API_KEY` | Debate, orchestrator–worker, human-in-the-loop; optional Gemini path in evaluator | [Google AI Studio](https://aistudio.google.com/) key |
| `GEMINI_MODEL` | Debate, orchestrator–worker, human-in-the-loop | Model id (e.g. `gemini-2.0-flash`); notebooks strip a leading `models/` if you paste the full id |
| `SERPER_API_KEY` | Orchestrator–worker | [Serper.dev](https://serper.dev) — powers `GoogleSerperAPIWrapper` search |
| `GEMINI_BASE_URL` | Evaluator notebook | OpenAI-compatible Gemini endpoint (notebook has a sensible default) |
| `OLLAMA_MODEL` | Evaluator notebook | Local model name, e.g. `llama3.2` |

### 4️⃣ Launch Jupyter from the root

```bash
uv run jupyter lab
```

or

```bash
uv run jupyter notebook
```

### 🦙 Side quest: evaluator + Ollama

The evaluator notebook expects **Ollama** on your machine:

```bash
ollama serve
ollama pull llama3.2
```

Match the pulled name to **`OLLAMA_MODEL`** in `.env`.

---

## 🧩 Pattern primer (the vocabulary)

These names line up with how people talk about agents in the wild (for example Anthropic’s [*Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents) and similar writeups). Use this section as a **glossary** while you read the code.

---

### ⚖️ Evaluator–optimizer (and cross-evaluation)

![pattern](https://img.shields.io/badge/Focus-scoring%20%26%20comparison-E74C3C?style=for-the-badge)

> **🔑 In one line:** *Someone produces; someone else scores—repeat or compare stacks under the same rubric.*

**The picture in your head:** Model A drafts; Model B judges. Rinse and repeat until quality is good—or until you stop iterating.

**Twist in this repo:** The evaluator notebook is a **symmetric cross-evaluation**, not a single rewrite loop:

1. **Round A:** Gemini asks → Ollama answers → Gemini **scores** (0–100 + rubric).
2. **Round B:** Ollama asks → Gemini answers → Ollama **scores** the same way.

So you still get **evaluate-then-score**, but the *point* is **comparison**: two stacks (cloud Gemini vs local Ollama) under the same grading lens.

**✅ Reach for this when** you’re calibrating prompts, running A/B checks, or you want two systems to **stress-test** each other with a shared rubric.

---

### 🔀 Routing (who goes next?)

![pattern](https://img.shields.io/badge/Focus-dynamic%20graph%20edges-6C5CE7?style=for-the-badge)

> **🔑 In one line:** *The next node isn’t always the same—the workflow **chooses** where to go.*

**The picture:** The workflow doesn’t always march A → B → C. Something—rules, another model, a human—**picks the next step**.

**Flavors:**

| Style | Mental model |
|-------|----------------|
| **Rule / DSL** | `if state["step"] == "x": go to y` |
| **LLM routing** | Parse JSON or a tool call: “next = researcher” |
| **Graph-native** | The node’s return value literally says **next node** |

**In this repo — debate notebook:** LangGraph [**`Command`**](https://langchain-ai.github.io/langgraph/) does the heavy lifting. Debaters and the moderator return `Command(goto=<next>, update=<partial_state>)`. Only **`START → coin_toss`** is a boring static edge; after that, the path is **dynamic**.

**✅ Reach for this when** turns matter: debates, support bots that escalate, or any time “what happens next” shouldn’t be hard-coded.

---

### 🎭 Orchestrator–worker (split the work, merge the glory)

![pattern](https://img.shields.io/badge/Focus-fan--out%20%2F%20fan--in-00B894?style=for-the-badge)

> **🔑 In one line:** *One boss coordinates; specialists work **in parallel**; one step **stitches** the result.*

**The picture:** One node **coordinates** (maybe with tools like search). Then **several specialists** work **at the same time**. Finally something **stitches** the pieces into one answer.

**In this repo — orchestrator notebook:** A `StateGraph` with an orchestrator, **three parallel writers** (summary / body / conclusion), and an **aggregator** that builds `final_report`. Serper gives the orchestrator a search tool.

**🆚 Not the same as routing:** The **shape** of the graph is mostly fixed. You’re not constantly re-deciding *which* node exists—you’re **fanning out** and **fanning in**.

**✅ Reach for this when** subtasks are **independent**, you want **shorter wall-clock** time, or you want to swap **one** specialist’s prompt/model without rewiring everything else.

---

### ✋ Human-in-the-loop (pause, persist, resume)

![pattern](https://img.shields.io/badge/Focus-approvals%20%26%20safety-FD7E14?style=for-the-badge)

> **🔑 In one line:** *The graph **stops** at a dangerous step, **saves** state, and **waits** until a person (or service) supplies input—then **continues** with that input.*

**The picture:** An agent calls tools freely for “safe” work (research, drafting). Before something irreversible—**sending email**, placing an order, deleting rows—the tool calls **`interrupt(...)`** with a JSON-serializable payload. Your UI or notebook reads **`__interrupt__`**, the human approves or edits, and you call **`invoke(Command(resume=...), config)`** with the **same `thread_id`** so the return value flows back into the tool.

**In this repo — human-in-the-loop notebook:** An explicit **ReAct** graph (`StateGraph(MessagesState)` + **`ToolNode`** + **`tools_condition`**), same hand-rolled style as the debate notebook’s `StateGraph`, **not** the deprecated prebuilt ReAct factory. Gemini runs the **`agent`** node; **`send_email`** wraps approval in **`interrupt()`** per the official [Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) guide (including *Interrupts in tools*).

**🆚 Vs static breakpoints:** `interrupt_before` / `interrupt_after` are great for **stepping through** a graph in development. For **production approvals**, prefer **`interrupt()`** so the pause is **conditional** and lives next to the business logic.

**✅ Reach for this when** you need **auditability**, **legal sign-off**, or a **human veto** on high-impact tool calls.

---

### 🌳 Hierarchical decomposition (parent delegates, child delivers)

![pattern](https://img.shields.io/badge/Focus-depth%20%26%20delegation-0984E3?style=for-the-badge)

> **🔑 In one line:** *Parent breaks the job into **sub-jobs**, waits for **children**, then **continues** with their output.*

**The picture:** The job is too big for one context, or you want strict separation (e.g. only the researcher hits the web). A **parent** breaks work into chunks, **waits** for children, then **continues** with their outputs in mind.

**🆚 Vs routing:** Routing is **which branch next**. Hierarchy is **depth**: hand off a *slice* of work, sync, compose—not pick among peers in a flat graph.

**Example (conceptual):** `ReportWriter` asks `ResearchAssistant` for notes; the assistant uses `WebSearch` + `Summarizer`; the writer folds results into prose.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#74b9ff', 'primaryTextColor': '#2d3436', 'primaryBorderColor': '#0984E3', 'lineColor': '#636e72', 'secondaryColor': '#dfe6e9', 'tertiaryColor': '#ffeaa7'}}}%%
flowchart TD
    RW[ReportWriter<br/>parent: outline + prose]
    RA[ResearchAssistant<br/>sub-agent: gather + condense]
    WS[WebSearch]
    SUM[Summarizer]

    RW -->|"sub-task: research topic X;<br/>wait for structured result"| RA
    RA --> WS
    RA --> SUM
    RA -->|"summaries + sources"| RW
    RW --> RW2[Continue: draft using<br/>returned research]
```

On **GitHub**, Mermaid renders in the Markdown preview.

**✅ Reach for this when** reports are long, budgets differ per role, or compliance says “only this agent gets network access.”

---

## 🧪 Hands-on notebooks

Suggested **first run** if you’re undecided: start with whichever pattern you’re most curious about—they don’t depend on each other.

---

### 🥇 Lab 1 — Evaluator optimizer (`evaluator_optimizer_workflow.ipynb`)

[![Lab 1](https://img.shields.io/badge/Lab%201-Evaluator–optimizer-E74C3C?style=for-the-badge)](src/evaluatorOptimizerWorkflow/evaluator_optimizer_workflow.ipynb)

| | |
|---|---|
| **📂 Open** | [`src/evaluatorOptimizerWorkflow/evaluator_optimizer_workflow.ipynb`](src/evaluatorOptimizerWorkflow/evaluator_optimizer_workflow.ipynb) |
| **🎯 Pattern** | ![Evaluator](https://img.shields.io/badge/Cross--evaluation-two%20rounds-E74C3C?style=flat-square) Symmetric **scoring** |

**Play-by-play**

- **Round 1:** Gemini → question → Ollama → answer → Gemini → score (0–100).
- **Round 2:** Ollama → question → Gemini → answer → Ollama → score (0–100).

**📋 Rubric:** The notebook’s `GRADING_CRITERIA` spells out dimensions like accuracy, completeness, clarity, depth, relevance—tweak them and watch scores move.

**🛠️ Stack:** Gemini via an **OpenAI-compatible** client (`GEMINI_BASE_URL`, `GEMINI_API_KEY`). **Ollama** must be up for the local half.

---

### 🥈 Lab 2 — Three-agent debate (`three_agent_debate.ipynb`)

[![Lab 2](https://img.shields.io/badge/Lab%202-Routing%20%2B%20debate-6C5CE7?style=for-the-badge)](src/threeAgentDebateLangGraph/three_agent_debate.ipynb)

| | |
|---|---|
| **📂 Open** | [`src/threeAgentDebateLangGraph/three_agent_debate.ipynb`](src/threeAgentDebateLangGraph/three_agent_debate.ipynb) |
| **🎯 Pattern** | ![Routing](https://img.shields.io/badge/Command%20routing-moderator%20%2B%202-6C5CE7?style=flat-square) Dynamic **next step** |

**What to notice**

- **`coin_toss`**, **`for_agent`**, **`against_agent`**, **`moderator`** all return **`Command(goto=..., update=...)`** so LangGraph gets **next node + state patch** in one shot.
- **`moderator`** uses structured output (`ModeratorResponse`). **`MAX_MODERATOR_CALLS`** can force an end; then **`FinalVerdictResponse`** picks **`for_agent` / `against_agent` / `tie`** so you don’t limp off as **`undecided`**.
- **TypedDict** + **`Annotated[list[str], add]`** keeps transcript history tidy.
- **Coin toss** + **both openings** before the first moderation (fights first-speaker bias).
- **Rebuttals** use the opponent’s latest line; at least **one moderator follow-up** before a normal **end**.

**🔐 Env:** `GEMINI_API_KEY`, optional `GEMINI_MODEL` (notebook Step 1 uses native `google_genai`).

#### 🗺️ Debate graph (for drawing on a whiteboard)

Only `START → coin_toss` is a fixed `add_edge`. Everything else mirrors **`goto`** from **`Command`**.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#a29bfe', 'lineColor': '#6C5CE7'}}}%%
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

---

### 🥉 Lab 3 — Orchestrator–worker (`orchastrator_worker_pattern.ipynb`)

[![Lab 3](https://img.shields.io/badge/Lab%203-Orchestrator–worker-00B894?style=for-the-badge)](src/orchastratorWorkerPattern/orchastrator_worker_pattern.ipynb)

| | |
|---|---|
| **📂 Open** | [`src/orchastratorWorkerPattern/orchastrator_worker_pattern.ipynb`](src/orchastratorWorkerPattern/orchastrator_worker_pattern.ipynb) |
| **🎯 Pattern** | ![Orchestrator](https://img.shields.io/badge/Parallel%20workers-fan--out%20fan--in-00B894?style=flat-square) **Merge** at the end |

**Play-by-play**

1. **`research_assistant_agent`** — LLM + **Serper** tool for research-flavored turns.
2. **Three writers** — summary, body, conclusion — run **in parallel** (different system prompts).
3. **`research_report_aggregator`** — glues everything into **`final_report`**.

**🛠️ Stack:** `init_chat_model` + `google_genai` (same vibe as the debate notebook). **`SERPER_API_KEY`** unlocks search.

---

### 🏅 Lab 4 — Human in the loop (`human_in_the_loop.ipynb`)

[![Lab 4](https://img.shields.io/badge/Lab%204-Human--in--the--loop-FD7E14?style=for-the-badge)](src/humanInTheLoop/human_in_the_loop.ipynb)

| | |
|---|---|
| **📂 Open** | [`src/humanInTheLoop/human_in_the_loop.ipynb`](src/humanInTheLoop/human_in_the_loop.ipynb) |
| **🎯 Pattern** | ![Human in the loop](https://img.shields.io/badge/interrupt%28%29-Command%28resume%29-FD7E14?style=flat-square) **Approvals** before side effects |

**Play-by-play**

1. **`agent`** — Gemini with **`bind_tools`**: may call `research_topic`, `draft_email`, or `send_email`.
2. **`tools`** — **`ToolNode`** runs tool calls; `send_email` hits **`interrupt({...})`** and execution pauses with **`__interrupt__`** in the invocation result.
3. **Resume** — Same **`thread_id`**: **`invoke(Command(resume={"action": "approve", ...}), config)`** (or reject) so the tool finishes and the loop returns to **`agent`** for a final answer.

**What to notice**

- **`MemorySaver`** checkpointer is **required** for pause/resume; use a durable saver in production.
- **Side effects** (real SMTP, etc.) should run **after** `interrupt()` returns approval, not before—see *Rules of interrupts* in the [same doc page](https://docs.langchain.com/oss/python/langgraph/interrupts).

**🔐 Env:** `GEMINI_API_KEY`, optional `GEMINI_MODEL` (native `google_genai`, same Step 1 pattern as the debate and orchestrator notebooks).

---

## 📦 Dependencies (the boring-but-important bit)

Shared packages live in **`pyproject.toml`** and **`uv.lock`**. When you add a dependency for a new notebook, add it **there** so everyone (including you, next month) stays on one toolchain.

---

## ➕ Adding your own notebook

1. Drop a folder under **`src/`** and add the `.ipynb`.
2. Update **Map of the repo**, **Hands-on notebooks**, and (if it’s a new pattern) the **cheat sheet** / **pattern primer** in this file (one README beats scattered docs unless something is huge).
3. Say which **pattern** you’re teaching—bonus points for a matching badge in the cheat sheet.

---

🚀 **Happy experimenting.**
