# Databricks GenAI Engineer — Exam Cheat Sheet

---

## Exam Weights

| Section | % | Focus |
|---|---|---|
| App Development | **30%** | RAG, context engineering, tools, guardrails |
| Assembling & Deploying | **22%** | MLflow, Model Serving, DABs, environments |
| Design Applications | **14%** | Prompts, model selection, agentic/non-agentic |
| Data Preparation | **14%** | Chunking, embeddings, ai_parse_document |
| Evaluation & Monitoring | **12%** | RAG metrics, MLflow eval, Lakehouse Monitoring |
| Governance | **8%** | Llama Guard, DASF, prompt injection, licensing |

---

## Decision Trees

### Deployment Method

```
High volume, > 30 min data intervals, no urgency  →  Batch (ai_query, Spark UDFs)
Continuous stream, moderate latency               →  Streaming
Instant response needed (chatbot, API)            →  Real-time (Model Serving)
On-device, small model                            →  Edge
```

### Model Class Selection

```
1. Minimum: How complex is the task? What quality is required?
   → Small (7B–13B): extraction, formatting, simple Q&A
   → Medium (30B–70B): support, content generation
   → Large (70B+): complex reasoning, ambiguous tasks
   → Frontier: high-stakes, quality > cost
2. Narrow: cost and latency constraints
3. Deal-breaker: privacy, deployment constraints
```

### Agentic vs Non-Agentic

```
Predictable, linear, deterministic logic?               →  Non-Agentic
Requires reasoning loop (Reason→Act→Observe)?          →  Agentic
Open-ended goal, tool integration, multi-step?          →  Agentic
Summarization, translation, templated generation?       →  Non-Agentic
```

### GenAI Decision Tree

```
Can output quality be measured clearly?          No → refine
Does it require deterministic accuracy?          Yes → Rules/Classical ML/SQL
Involves language understanding or generation?   No → Classical ML/SQL
Requires synthesis or open-ended output?         No → Classical ML classification
Depends on proprietary enterprise knowledge?     Yes → GenAI + RAG  /  No → GenAI
```

---

## Vector Search — 3 Ingestion Modes

| Mode | Embeddings | Auto-sync? | Use when |
|---|---|---|---|
| **Managed** (Delta Sync) | Databricks computes via Model Serving endpoint | Yes | Simplest; let Databricks manage |
| **Self-Managed** (Delta Sync) | You compute, store in Delta table | Yes | Custom embedding pipeline |
| **Direct Vector Access** (CRUD API) | You insert/update/delete directly | No (no Delta table) | Real-time apps, no batch table |

**Search types:** Similarity (semantic/cosine) · Full-Text (keywords) · **Hybrid** (both, highest accuracy)

**Similarity metrics:** Cosine (text, orientation) · Euclidean L2 (clustering) · Manhattan L1 (grid/sparse)

**Search algorithms:** KNN (exact, doesn't scale) · **ANN** (approximate, fast — uses HNSW/FAISS)

---

## RAG Pipeline Components

```
User Query
→ [Embedding Model: same as index embedding model!]
→ [Vector Search: ANN/Hybrid, retrieve top 20-50]
→ [Reranker: Cross-Encoder, keep top 3-5]
→ [Context Window: System Prompt + Retrieved Chunks + History]
→ [LLM: grounded generation]
→ Response
```

---

## 6 RAG Evaluation Metrics

| Metric | Inputs | Question |
|---|---|---|
| **Context Precision** | Query + Context | Are relevant chunks ranked first? (signal/noise ratio) |
| **Context Relevancy** | Query + Context | Does context address query intent? |
| **Context Recall** | Ground Truth + Context | Does context cover ALL facts in ground truth? |
| **Faithfulness / Groundedness** | Response + Context | Is every claim supported by context? (hallucination check) |
| **Answer Relevancy** | Response + Query | Does the answer address what was asked? |
| **Answer Correctness** | Response + Ground Truth | Is the answer factually accurate vs ground truth? |

**Key distinction:** Faithfulness ≠ Correctness. Faithful = grounded in retrieved context. Correct = matches ground truth. A faithful answer can be wrong if context itself was wrong.

**Databricks official metric names:** `groundedness`, `chunk_relevance/precision`, `context_sufficiency`, `correctness`, `relevance_to_query`

---

## Chunking Strategies

| Strategy | Method | Use |
|---|---|---|
| Fixed-Size | Split by character/token count | Legacy/baseline only |
| **Semantic/Recursive** | Split on sentences, paragraphs, sections | Recommended standard |
| **Chunk Overlap** | 10–20% overlap between chunks | Prevents context loss at boundaries |
| Embedding-Based | Break when semantic similarity drops | Advanced; best coherence |
| Windowed Summarization | Each chunk includes summary of prior chunks | Rich context without full history |

**Rule:** max chunk size < embedding model context window limit (or text is silently truncated)

---

## MLflow Key APIs

```python
# Set experiment
mlflow.set_experiment("/my-experiments/rag-agent")

# Log model
mlflow.langchain.log_model(chain, "model", input_example={"query": "..."})
mlflow.pyfunc.log_model(...)

# Register to Unity Catalog
mlflow.register_model("runs:/<run_id>/model", "catalog.schema.model_name")

# Load from UC
model = mlflow.pyfunc.load_model("models:/catalog.schema.model_name/@champion")

# Evaluate
mlflow.evaluate(model, eval_data, targets="ground_truth", model_type="question-answering")

# Custom metric
from mlflow.metrics.genai import make_genai_metric
metric = make_genai_metric(name="helpfulness", definition="...", grading_prompt="Score 1-5", examples=[...])

# Tracing decorator
@mlflow.trace(span_type="RETRIEVER")
def retrieve(query): ...
```

### MLflow Trace Structure

- **Trace** = entire request lifecycle (user query → final answer)
- **Span** = one unit of work (embedding, retrieval, LLM call)
- Spans follow **OpenTelemetry** standard
- Auto-logging: `mlflow.langchain.autolog()`, `mlflow.openai.autolog()`

---

## MLflow Model Flavors

| Flavor | Use Case |
|---|---|
| `mlflow.langchain` | LangChain chains |
| `mlflow.openai` | OpenAI API |
| `mlflow.transformers` | HuggingFace |
| `mlflow.pyfunc` | **Default fallback** — any custom Python |
| `mlflow.pytorch` | PyTorch |

**`pyfunc` ResponsesAgent** — recommended production interface:

```python
class MyAgent(mlflow.pyfunc.ResponsesAgent):
    def predict(self, request: ResponsesAgentRequest) -> ResponsesAgentResponse: ...
```

---

## ai_query() — Batch SQL

```sql
SELECT AI_QUERY(
  "databricks-meta-llama-3-1-70b-instruct",
  CONCAT("Summarize: ", text_column)
) AS summary
FROM bronze_table;
```

---

## ai_parse_document() — Document Parsing

```sql
SELECT ai_parse_document(content) AS parsed_document
FROM read_files('/Volumes/catalog/schema/volume/', format => 'binaryFile')
```

Capabilities: OCR, layout awareness, figure descriptions, bounding boxes (v2.0)

---

## Unity Catalog Namespace

```
catalog . schema . asset_name
 └── 3-level namespace for tables, models, functions, volumes, vector indexes
```

Register model: `"catalog.schema.model_name"`
Load with alias: `"models:/catalog.schema.model_name/@champion"`

---

## LLMOps vs MLOps Differences

| Dimension | MLOps | LLMOps |
|---|---|---|
| Dev start | Train from data | Start with off-shelf API |
| Packaging | Single model artifact | **Entire application** (chain + vector DB + UI + APIs) |
| Serving | Model inference endpoint | + GPU infra, vector DBs, conversation state |
| Governance | Model version control | **API rate limits**, credential management |
| Cost | Compute/throughput | **Token-based** (input + output tokens) |
| Feedback | Optional | **Essential** — primary quality signal |
| New artifacts | — | Prompt templates, system prompts as code |

---

## Llama Guard

```
User → [Input Guard] → LLM → [Output Guard] → Response
```

**6 risk categories:** Violence & Hate · Sexual Content · Guns & Illegal Weapons · Regulated Substances · Suicide & Self Harm · Criminal Planning

---

## DASF — 6 Components for GenAI Engineers

4: Data Catalog & Governance · 5: Algorithms · 6: Evaluation · 8: Model Management · 11: Operations · **12: Platform Security** (foundational)

---

## GPU Memory Formula

```
Required GPU RAM = num_parameters × bytes_per_parameter
Example: 10B params × 4 bytes (FP32) = 40 GB
```

---

## Deployment Environments

```
Development → Staging → Production
(separate Databricks workspace per environment — "Direct Separation")
```

**Deploy Code** (recommended) > Deploy Model — run training pipeline in each environment.

---

## DABs (Databricks Asset Bundles)

```bash
databricks bundle deploy -t production
databricks bundle run my_job -t production
```

Use **service principals** for CI/CD (not personal credentials). Bundle YAML: `bundle`, `resources`, `targets` sections.

---

## Core Component Quick Reference

| Component | Purpose |
|---|---|
| **Unity Catalog Volumes** | Governed storage for raw files (PDFs, images) — Bronze |
| **Delta Lake** | ACID tables for structured/processed data — Silver/Gold |
| **ai_parse_document** | OCR + layout extraction from PDFs/images |
| **Mosaic AI Vector Search** | Vector DB integrated with Delta Sync and Unity Catalog |
| **MLflow Tracking** | Log params, metrics, artifacts per run |
| **MLflow Model Registry (UC)** | Version, alias, ACL models in catalog.schema.model |
| **MLflow Tracing** | Traces + Spans for agent observability |
| **Databricks Model Serving** | REST endpoints for custom/foundation/external models |
| **Inference Tables** | Auto-logged request-response pairs in UC Delta |
| **Lakehouse Monitoring** | Profile + drift metrics tables + DBSQL dashboards |
| **Databricks Asset Bundles** | YAML CI/CD bundles for multi-environment deployment |
| **Agent Bricks** | Declarative framework: Knowledge Assistant, Info Extraction, MAS, Custom LLM |
| **Llama Guard** | Input + output safety classifier |
| **AI Gateway** | Rate limits, API governance, unified endpoint |

---

## Agent Bricks — 4 Types

| Type | Category | Function |
|---|---|---|
| **Knowledge Assistant (KA)** | Interactive | RAG over knowledge base; citations |
| **Information Extraction (IE)** | Automated | Unstructured → structured (JSON schema) |
| **Multi-Agent Supervisor (MAS)** | Interactive | Orchestrates multiple agents/tools |
| **Custom LLM (CLLM)** | Automated | Fine-tuned/optimized for specific tasks |

**ALHF** = Agent Learning from Human Feedback (Agent Bricks optimization loop)

---

## Evaluation Checklist

| Type | When | Method |
|---|---|---|
| Offline | Pre-deployment | Benchmark datasets + MLflow evaluate |
| Online | Post-deployment | Inference Tables + Lakehouse Monitoring |
| Human | Continuous | Review App (explicit) + behavioral signals (implicit) |
| LLM-as-Judge | When no reference data | `make_genai_metric` + human-in-the-loop oversight |
