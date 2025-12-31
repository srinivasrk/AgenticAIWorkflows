# AgenticAIWorkflows

A collection of agentic AI workflow projects organized in a Python `uv`-managed repository.

## Project Structure

```
AgenticAIWorkflows/
├── src/
│   └── evaluatorOptimizerWorkflow/  # Individual workflow projects
│       ├── evaluator_optimizer_workflow.ipynb
│       └── README.md
├── pyproject.toml                 # Common dependencies for all projects
├── uv.lock                        # Locked dependency versions
├── .env                           # Environment variables (not in git)
└── README.md                      # This file
```

## Setup

1. **Install uv** (if not already installed):
   ```bash
   pip install uv
   ```

2. **Install dependencies**:
   ```bash
   uv sync
   ```

3. **Configure environment variables**:
   - Copy `.env.example` to `.env` (if available)
   - Add your API keys and configuration

4. **Start Jupyter**:
   ```bash
   uv run jupyter notebook
   # or
   uv run jupyter lab
   ```

## Workflows

### Evaluator Optimizer Workflow
Cross-evaluation system between Gemini and Ollama. See `src/evaluatorOptimizerWorkflow/README.md` for details.

## Dependencies

All dependencies are managed in the root `pyproject.toml` file and shared across all projects. This ensures consistency and easier maintenance.

## Adding New Workflows

1. Create a new folder in `src/` for your workflow
2. Add Jupyter notebooks and any necessary files
3. Dependencies are already available from the root `pyproject.toml`
4. Add a README.md in your workflow folder to document it