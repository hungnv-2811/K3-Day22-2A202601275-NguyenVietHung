# Lab Requirements — Day 22: LangSmith + Prompt Versioning

## Python Version
Python 3.10 or higher

## Install All Dependencies

```bash
pip install -r requirements.txt
```

## requirements.txt

See [`requirements.txt`](requirements.txt) for the exact pinned versions that are verified to work together.

> ⚠️ **Why versions are pinned, not just `>=`:** `ragas==0.4.3` hard-imports
> `langchain_community.chat_models.vertexai` internally. That module was
> removed in `langchain-community` 0.4.x, so an unpinned `langchain-community>=0.3.0`
> install can silently resolve to 0.4.x and crash with `ModuleNotFoundError`
> partway through Step 3 (after 15+ minutes of RAGAS calls). Installing from
> the pinned `requirements.txt` avoids this.

## Package Purposes

| Package | Used For |
|---------|---------|
| `langchain` | Core LLM framework |
| `langchain-openai` | ChatOpenAI, OpenAIEmbeddings |
| `langchain-community` | FAISS vectorstore integration |
| `langchain-text-splitters` | RecursiveCharacterTextSplitter |
| `langsmith` | LangSmith tracing, Prompt Hub client |
| `openai` | Direct OpenAI API calls |
| `faiss-cpu` | Similarity search index |
| `ragas` | RAG evaluation metrics |
| `guardrails-ai` | Output validation framework |
| `python-dotenv` | Load `.env` file |
| `tiktoken` | Token counting for text splitters |
| `datasets` | Required by RAGAS internally |
| `numpy` | Averaging RAGAS score lists |

## Important Version Notes

### RAGAS 0.4.x
- Use `from ragas.metrics import faithfulness, answer_relevancy, ...` (NOT from `ragas.metrics.collections`)
- `result[metric_name]` returns a **list** of floats for multiple samples — use `numpy.mean()` to average
- Pass `llm=` and `embeddings=` to the `evaluate()` function, not to metric constructors

### Guardrails AI 0.11.x
- `on_fail` parameter belongs in the **validator constructor**: `MyValidator(on_fail=OnFailAction.FIX)`
- `Guard.use()` accepts validator **instances**, not classes
- `Guard.validate(text)` is the main entry point

### LangChain 0.3.x
- Use `ChatOpenAI(api_key=..., base_url=..., model=...)` for custom endpoints
- Use `OpenAIEmbeddings(api_key=..., base_url=..., model=...)` for custom embedding endpoints

## Environment Variables

Copy the template and fill in your own keys:

```bash
cp .env.example .env
```

See [`.env.example`](.env.example) for the full list of variables (LangSmith keys are
required for every step; only the section matching your chosen `PROVIDER` needs real values).

> ⚠️ **Never commit `.env` to git.** It's already listed in `.gitignore` — verify with
> `git status` before every commit.

## Verify Installation

Run the config check:
```bash
cd src && python config.py
```

Expected output:
```
✅ Config OK  |  Provider: OPENAI  |  Project: day22-lab
```

If you see `⚠️ Thiếu biến môi trường` instead, one or more required keys in `.env` are
still the placeholder value (`your_..._here`) — go back and fill them in.
