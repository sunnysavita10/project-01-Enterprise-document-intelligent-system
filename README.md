# Enterprise Document Intelligence System

## Implementation Status

### Architecture Layers

| Layer | Status | What EDIS Includes |
| --- | --- | --- |
| UI Layer | ✅ | HTML, CSS, and JavaScript. Login, signup, and workspace UI. |
| API Layer | ✅ | FastAPI endpoints for upload, chat, analysis, comparison, review, and health checks. |
| AI Layer | ✅ | LangChain and LangGraph workflows. Groq is the default provider. OpenAI and Gemini can be selected through `.env`. |
| Data Layer | ✅ | Chroma vector index, SQLite for accounts, and local JSON storage for chat history and review decisions. |
| Cloud / Deployment Layer | ⏳ | A Dockerfile is available. The container build is unverified, and the AWS ECS Fargate deployment workflow is pending. |
| Monitoring / Tracing Layer | ✅ | `structlog` JSONL logs, request IDs, and LangSmith traces. Infrastructure dashboards and alerts are pending. |
| Evaluation Layer | ✅ | An offline Document Chat benchmark that checks factual accuracy, citations, missing evidence, and routing. |
| Security Layer | ✅ | Login, password hashing, JWT support, session ownership, upload validation, and trace masking. |
| Testing Layer | ✅ | Flake8, pytest, and a GitHub Actions workflow for pushes and pull requests. 31 tests passed locally. |

### Feature Status

| Feature | Status | What EDIS Includes |
| --- | --- | --- |
| Validation (Pydantic) | ✅ | API schemas, intent routing, and structured result validation. |
| Guardrails | ✅ | One retry for invalid intent, citation ID validation, and a safe response when no evidence is found. |
| Evaluation | ✅ | Seven benchmark cases for Document Chat. |
| MCP Client / Server | ⏳ | Integration is pending. |
| Human-in-the-Loop (HITL) | ✅ | **Approve** and **Request changes** options for Analysis and Compare, with saved review notes. |
| Memory | ✅ | Separate histories for General Chat and Document Chat. Users can resume after a refresh in the same tab or start a **New Chat**. |
| Caching | ✅ | An in-memory `cachetools` cache for Analysis and Compare results. |
| LLM Gateway | ⏳ | Gateway options are documented below. The app currently calls providers directly. |
| Async | ✅ | Upload, dataset loading, analysis, and comparison use async API routes. Blocking operations run in a thread pool. |

## Configure the language model

Run the commands below from the project folder with the project's `env` Python environment installed. If you need a local configuration file, copy [`.env.example`](.env.example) to `.env`.

Add the API key for each provider you select. Groq is the default. Set `LLM_PROVIDER` to change the provider for document chat, analysis, and comparison. Each model variable is optional and accepts a model ID supported by the selected provider.

| Provider | `LLM_PROVIDER` | Model setting and default | API key |
| --- | --- | --- | --- |
| Groq | `groq` | `GROQ_CHAT_MODEL=qwen/qwen3.8-27b` | `GROQ_API_KEY` |
| OpenAI | `openai` | `OPENAI_CHAT_MODEL=gpt-4.1-mini` | `OPENAI_API_KEY` |
| Google Gemini | `google` | `GOOGLE_CHAT_MODEL=gemini-3.5-flash-lite` | `GOOGLE_API_KEY` |

Set `CHAT_LLM_PROVIDER` to override the provider for chat only. For example, `LLM_PROVIDER=groq` and `CHAT_LLM_PROVIDER=openai` use Groq for analysis and comparison and OpenAI for chat. Leave `CHAT_LLM_PROVIDER` blank in `.env` to have chat inherit `LLM_PROVIDER`.

Embeddings have their own settings. With the default `EMBEDDING_PROVIDER=huggingface`, set `HF_TOKEN` in `.env`.

### Save the selection in `.env`

For example, to use OpenAI for all three workflows, update these lines in `.env` and fill in `OPENAI_API_KEY`.

```dotenv
LLM_PROVIDER=openai
CHAT_LLM_PROVIDER=
OPENAI_CHAT_MODEL=gpt-4.1-mini
```

### Windows Command Prompt

Set the provider and model, then start the app in the same Command Prompt window.

```cmd
set "LLM_PROVIDER=openai"
set "OPENAI_CHAT_MODEL=gpt-4.1-mini"
env\Scripts\python.exe -m uvicorn app.main:app --port 8080
```

For Gemini, use `set "LLM_PROVIDER=google"` and `set "GOOGLE_CHAT_MODEL=gemini-3.5-flash-lite"` before the start command. To switch back to Groq, use `set "LLM_PROVIDER=groq"`. For a chat-only override, use `set "CHAT_LLM_PROVIDER=openai"`.

### Windows PowerShell

```powershell
$env:LLM_PROVIDER = "openai"
$env:OPENAI_CHAT_MODEL = "gpt-4.1-mini"
.\env\Scripts\python.exe -m uvicorn app.main:app --port 8080
```

For Gemini, set `$env:LLM_PROVIDER = "google"` and `$env:GOOGLE_CHAT_MODEL = "gemini-3.5-flash-lite"` before starting the app. To switch back to Groq, set `$env:LLM_PROVIDER = "groq"`. For a chat-only override, set `$env:CHAT_LLM_PROVIDER = "openai"`.

Console settings apply to the current terminal and take precedence over the corresponding values in `.env`. The app can still read API keys from `.env`. If the app is already running, stop it with **Ctrl+C**, make the change, and start it again. Open [the local app](http://localhost:8080) after startup.

## LLM Gateway Options

| Gateway | Description |
| --- | --- |
| [LiteLLM](https://docs.litellm.ai/) | Open-source LLM proxy with a unified API, model routing, fallbacks, budgets, and rate limiting. |
| [Portkey](https://portkey.ai/) | AI gateway with model routing, caching, guardrails, and observability. Now branded as Prisma AIRS AI Gateway. |
| [Kong AI Gateway](https://developer.konghq.com/ai-gateway/) | Enterprise API and AI gateway with authentication, traffic management, and AI-specific policies. |
| [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/) | Managed AI gateway with caching, rate limiting, analytics, retries, and model fallbacks. |
| [Helicone](https://www.helicone.ai/) | AI gateway focused on LLM observability, usage monitoring, and request management. |
| [OpenRouter](https://openrouter.ai/) | Hosted, unified API for accessing models from multiple providers. |
| [Azure API Management AI Gateway](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities) | Enterprise AI gateway for model access, security, governance, and traffic management. |
| [Databricks Unity Gateway](https://learn.microsoft.com/en-us/azure/databricks/ai-gateway/) | Centralized AI model access, governance, and usage management within Databricks. |

## Automated tests

The **Python CI** workflow runs on GitHub pushes and pull requests using Python 3.12. Flake8 checks for syntax errors and Pyflakes findings such as undefined names and unused imports. Pytest runs the application regression suite with mocked model responses. The workflow needs no API keys.

Run the same checks locally from the project root after installing `requirements.txt`.

```powershell
$env:PYTHON_DOTENV_DISABLED = "1"
$env:LANGSMITH_TRACING = "false"
$env:LANGSMITH_TRACING_V2 = "false"
.\env\Scripts\python.exe -m flake8 --select E9,F app data_downloading_script tests
.\env\Scripts\python.exe -m pytest -q tests
```

## Custom instructions

Open **Custom instructions** in Analysis & Summary or Compare to set the language, emphasis, or level of detail for that run. Choose **Save as my default** to prefill that workflow for your account. Clear the field and save it to reset the default. Changing the instructions creates a separate cache entry. Document evidence rules and response validation still apply.

## Human review

Analysis and Compare results start as pending review. Choose **Approve** to accept a result or **Request changes** with a note explaining what needs revision. EDIS saves the review status and note for the user who generated the result. API clients can retrieve the decision using `GET /api/reviews/{review_id}` and the `X-Review-ID` header from the original result.

## Evaluation

### Document Chat evaluation

Run the offline benchmark from the project root using the chat and embedding providers configured in `.env`.

```powershell
.\env\Scripts\python.exe -m app.ai_module.evals
```

The runner creates a synthetic two-page handbook and a temporary Chroma index. It checks answer facts and citation pages, missing evidence, and whether greetings, follow-ups, and an unrelated weather question bypass retrieval. It uses real model and embedding calls with the configured providers. The run keeps its JSON report local and disables LangSmith tracing. Application logs go to stderr.

Exit code `0` means all cases passed, `1` means at least one case failed, and `2` means the evaluation could not start or finish setup.

The following tables describe the broader evaluation plan for EDIS.

### Metrics by business module

| Business module | Evaluation metrics |
| --- | --- |
| Document Parsing | Text extraction accuracy, table extraction accuracy, page mapping |
| Document Chat (RAG) | Faithfulness, answer correctness, answer relevance |
| Retrieval | Context precision, context recall, Recall@K, MRR |
| General Chat | Intent classification, routing accuracy, response quality |
| Document Analysis | Factual correctness, metadata extraction accuracy, completeness |
| Document Summarization | Faithfulness, coverage, coherence |
| Document Comparison | Change detection accuracy, precision, recall, false positives |
| Source Citations | Citation accuracy, page accuracy, evidence support |
| Security | Prompt injection resistance, sensitive data leakage, unauthorized retrieval |
| Performance | Latency, token consumption, cost per request |

### Tools and responsibilities

| Library / Platform | Responsibility |
| --- | --- |
| `pytest` | Deterministic tests and regression testing |
| `ragas` | RAG-specific evaluation metrics |
| `deepeval` | LLM, RAG, and custom evaluation metrics |
| `langsmith` | Datasets, evaluators, experiments, and dashboards |
| `openevals` | Prebuilt and customizable evaluators |
| `pydantic` | Evaluation input and output schema validation |
| Custom Python | Business-specific evaluation logic |
| LLM-as-a-judge | Semantic correctness and qualitative evaluation |
| `locust` | Load and performance testing |

### Evaluation methods and EDIS examples

| Evaluation method | EDIS example |
| --- | --- |
| Deterministic evaluation | Check whether a citation points to the correct page. |
| Reference-based evaluation | Compare the actual answer with the expected answer. |
| Reference-free evaluation | Check whether the retrieved context supports the answer. |
| LLM-as-a-judge | Assess whether a summary is meaningful and complete. |
| Human evaluation | Have a domain expert verify the answer. |
| Component-level evaluation | Evaluate the retriever or parser in isolation. |
| End-to-end evaluation | Evaluate a complete application workflow. |

## Caching

Analysis and Compare use a per-process `cachetools` cache. Successful results are scoped to the authenticated user and kept for up to one hour, with at most 100 entries shared across both workflows. A hit skips document parsing and model calls. Look for `cache_hit` or `cache_miss` in the application logs and, when enabled, in the LangSmith trace.

The table outlines the local setup and options for distributed caching and monitoring.

| Phase | Technology |
| --- | --- |
| Local development | `cachetools` |
| Local distributed testing | Redis / Valkey with Docker |
| AWS production | AWS ElastiCache for Valkey |
| Cache monitoring | Prometheus / CloudWatch |
| Cache visualization | Grafana / CloudWatch Dashboard |
| LLM cost and latency monitoring | LangSmith |

### Caching tools by requirement

| Requirement | Recommended tool |
| --- | --- |
| Local in-memory caching | `cachetools` |
| Local persistent caching | `diskcache` |
| Asynchronous caching | `aiocache` |
| FastAPI response caching | `fastapi-cache2` |
| Distributed caching | Redis / Valkey |
| LLM response caching | LangChain Cache |
| Semantic caching | `RedisSemanticCache` / GPTCache |
| AWS production caching | AWS ElastiCache |
| Provider-level prompt caching | OpenAI / Anthropic / Gemini |
| Frontend and static content caching | AWS CloudFront |

## Observability

EDIS currently provides structured JSON logs with request IDs and the `GET /api/health` endpoint. The table below describes the proposed deployment setup.

| Layer | Tools | What they monitor |
| --- | --- | --- |
| **AI / LLM Observability** | LangSmith / Langfuse | prompts, retrieval, tool calls, tokens, model traces, evals |
| **Application / Infrastructure Observability** | OpenTelemetry + Prometheus + Grafana / CloudWatch | CPU, memory, HTTP latency, error rate, service traces, infra health |

### Where traces appear

| Situation | LangSmith | Langfuse Cloud | Langfuse Self-hosted |
| --- | --- | --- | --- |
| Local app traces | LangSmith web UI | Langfuse web UI | Your Langfuse UI |
| ECS traces | LangSmith web UI | Langfuse web UI | Your Langfuse UI |

### Enable LangSmith tracing

1. In LangSmith, create an EDIS agent and an API key in the same workspace. Put the key in your local `.env` as `LANGSMITH_API_KEY`. Set `LANGSMITH_ENDPOINT` if your workspace uses a regional API endpoint.
2. From the project root, get the exact name of the agent's Local tracing environment. Replace `your-agent-id` with the ID from LangSmith.

   ```text
   .\env\Scripts\python.exe -c "from dotenv import load_dotenv; from langsmith import Client; load_dotenv(); agent = Client().agents.retrieve('your-agent-id'); print(next(e.tracer_session_name for e in agent.environments if e.environment == 'LOCAL'))"
   ```

3. Add these values to your local `.env`, using the name printed by the command.

   ```dotenv
   LANGSMITH_TRACING=true
   LANGSMITH_API_KEY=<your-workspace-api-key>
   LANGSMITH_PROJECT=<local-tracer-session-name>
   ```

4. Restart the app and run a chat or analysis request. Open the EDIS agent and its Local environment in LangSmith. Match the trace's `request_id` with the `X-Request-ID` response header or the JSON logs.

Tracing is opt-in and requires an API key and a tracing project name. Operation and model traces retain timings and selected metadata. The trace client hides inputs and outputs and replaces raw error text with a reference to the application logs.

On ECS, inject `LANGSMITH_API_KEY` from AWS Secrets Manager. Set `LANGSMITH_TRACING=true` and use the Production environment's `tracer_session_name` as `LANGSMITH_PROJECT`. Set the region endpoint if required. [LangSmith compatibility settings](https://docs.langchain.com/langsmith/create-an-agent#configure-integrations-that-require-a-project-name) · [ECS secrets](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
