# Evaluator Optimizer Workflow

This workflow implements a cross-evaluation system between Gemini (via OpenAI-compatible API) and Ollama.

## Workflow Overview

### Round 1: Gemini → Ollama → Gemini
1. **Gemini generates** a challenging question
2. **Ollama answers** the question
3. **Gemini evaluates** Ollama's answer (0-100 score)

### Round 2: Ollama → Gemini → Ollama
1. **Ollama generates** a challenging question
2. **Gemini answers** the question
3. **Ollama evaluates** Gemini's answer (0-100 score)

## Standardized Grading

Both models use the same grading criteria:
- **Accuracy (30 points)**: Factual correctness
- **Completeness (25 points)**: Full coverage of the question
- **Clarity (20 points)**: Clear and well-structured response
- **Depth (15 points)**: Deep understanding and insight
- **Relevance (10 points)**: Direct relevance to the question

## Setup

1. **Install dependencies** (already in root `pyproject.toml`):
   ```bash
   uv sync
   ```

2. **Configure environment variables**:
   - Copy `.env.example` to `.env` in the project root
   - Add your Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey)
   - Set your preferred Ollama model (default: `llama3.2`)

3. **Ensure Ollama is running**:
   ```bash
   ollama serve
   ```

4. **Pull the Ollama model** (if not already installed):
   ```bash
   ollama pull llama3.2
   ```

## Usage

Open the `evaluator_optimizer_workflow.ipynb` notebook and run all cells.

## Notes

- The Gemini API is accessed via OpenAI-compatible endpoint
- Make sure your Gemini API key has access to the model you're using
- Adjust the model name in the notebook if needed (currently set to `models/gemini-2.0-flash-exp`)

