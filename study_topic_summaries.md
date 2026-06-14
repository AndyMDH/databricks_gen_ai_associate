# Databricks GenAI Engineer Exam — Topic Summaries

> Organized by exam section weight. Study heaviest sections first.

---

## 1. App Development — 30%

### Core Concept

App Development covers building functional GenAI applications: RAG pipelines, context engineering, guardrails, agents with tools, and minimizing hallucinations.

### RAG Architecture

The three-stage pipeline:

```
Retrieval → Augmentation → Generation
```

| Stage | What happens |
|---|---|
| **Retrieval** | Vector search finds relevant chunks from a knowledge base |
| **Augmentation** | Chunks are injected into the context window |
| **Generation** | LLM synthesizes answer using only injected context |

**Context Rot** — failure modes when too much is retrieved:

- **Context Poisoning**: irrelevant/conflicting chunks confuse the model
- **Lost in the Middle**: models prioritize info at the start/end; middle chunks are ignored

### Context Engineering

Moving from single-prompt optimization to managing the entire input environment:

| Component | Role |
|---|---|
| System Prompt | Behavioral program: role definition, negative constraints, output format |
| Conversation History | Multi-turn state; must be managed (summarize, window, or selectively persist) |
| Retrieved Data | Strictly grounded chunks — use Unity Catalog metadata filtering |
| User Input | Final query in the context window |

**System Prompt best practices:**

- Assign a persona: `"You are a Databricks Security Architect"`
- Negative constraints: `"Do not mention competitor products"`
- Enforce output format: JSON, YAML, Markdown table

**Grounding instruction example:**
> "Answer using ONLY the provided context chunks. If the answer is not present, state 'I do not have that information.'"

### Mosaic AI Vector Search (now: Databricks AI Search)

Three ingestion modes:

| Mode | How embeddings are created | When to use |
|---|---|---|
| **Managed Embeddings (Delta Sync)** | Databricks auto-computes via a Model Serving endpoint | Simplest setup; managed pipeline |
| **Self-Managed Embeddings (Delta Sync)** | You compute embeddings and store in Delta table; index syncs | Custom embedding pipeline with auto-sync |
| **Direct Vector Access (CRUD API)** | Insert/update/delete vectors directly via REST API | Real-time apps; no underlying Delta table |

**Delta Sync**: Vector index auto-updates when source Delta table changes. No manual re-indexing.

**Search methods:**

- **Similarity Search**: semantic / vector cosine distance
- **Full-Text Search**: keyword matching (exact terms, part numbers)
- **Hybrid Search**: combines both — highest accuracy

**Distance/similarity metrics:**

- Cosine Similarity — most common for text (orientation, not magnitude)
- Euclidean (L2) — absolute distance
- Manhattan (L1) — sum of absolute differences

**Search strategies:**

- **KNN** (K-Nearest Neighbors): exact, expensive, doesn't scale
- **ANN** (Approximate Nearest Neighbors): fast approximation; uses HNSW or FAISS

### Reranking

Two-step pipeline to improve precision:

1. ANN retrieves broad candidate set (top 20–50)
2. Cross-Encoder reranker rescores candidates against the query
3. Top 3–5 most relevant chunks passed to the LLM

**Trade-off:** Better accuracy but adds latency and cost.

### Guardrails

Prompt-level constraints in the system prompt to prevent harmful/off-target behavior:

```
"Do not provide advice on illegal activities. If asked, politely decline."
```

Layer guardrails with **Llama Guard** for defense-in-depth (see Governance section).

### Minimizing Hallucinations

| Technique | How it helps |
|---|---|
| Grounding instruction | Restricts model to retrieved context only |
| Metadata filtering | Prevents stale/irrelevant data from entering context |
| Reranking | Ensures only the most relevant chunks reach the LLM |
| Smaller context | Reduces "Lost in the Middle" effect |
| LLM-as-Judge (Groundedness metric) | Detects when answer goes beyond context |

### Token Budget Management

- **Just-in-Time Retrieval**: retrieve only when user asks a relevant question, not upfront
- **Reranking**: inject top 3–5 chunks, not top 50
- **Summarization**: periodically compress conversation history
- **Moving Window**: discard oldest messages
- **Selective Persistence**: keep only critical state (user name, current task ID)

### Prompt Augmentation

Injecting retrieved data into the prompt. The retrieved chunks + instructions together constitute the augmented prompt sent to the LLM.

---

## 2. Assembling and Deploying Applications — 22%

### MLflow Model Flavors

A **flavor** is a standard format for packaging ML/GenAI models. Each MLflow model is a directory with an `MLmodel` file specifying one or more flavors.

| Flavor | Use Case |
|---|---|
| `mlflow.langchain` | LangChain chains and agents |
| `mlflow.openai` | OpenAI API calls |
| `mlflow.transformers` | Hugging Face Transformers |
| `mlflow.pyfunc` | Any custom Python — default/fallback |
| `mlflow.pytorch` | PyTorch models |

**`pyfunc`** is the universal fallback — any MLflow model can be loaded as a Python function via `pyfunc`.

### MLflow + Unity Catalog Model Registry

```
mlflow.register_model("runs:/<run_id>/model", "catalog.schema.model_name")
```

**Three-level namespace:** `catalog.schema.model_name`

Key Registry features:

- **Aliases**: `@champion`, `@challenger` — human-readable pointers to versions
- **Versioning**: immutable snapshots per version
- **Lineage**: tracks data → model → endpoint chain
- **ACLs**: `SELECT`, `EXECUTE` permissions on registered models
- **Cross-workspace**: accessible across workspaces sharing the same metastore

### Databricks Model Serving

Provides a **unified API** for three model types:

| Type | Description |
|---|---|
| **Custom Models** | Any MLflow-registered model served as REST API |
| **Foundation Models API** | Databricks-curated models (DBRX, Llama, Mistral) |
| **External Models** | Proxy to OpenAI, Anthropic, etc. — same API surface |

**Production features:**

- Serverless, autoscaling
- Built-in payload logging to Delta tables
- A/B testing (canary deployments): e.g., 80% Champion / 20% Challenger
- Infrastructure metrics via API

### Inference Tables

Automatically log all request-response pairs to Unity Catalog Delta tables.

**Enable per monitoring workflow:**

```
Request → Endpoint → Inference Table (UC Delta) → Unpack JSON → Metrics → Lakehouse Monitoring
```

### Deploy Code vs Deploy Model

| Pattern | Description | Recommended? |
|---|---|---|
| **Deploy Model** | Move model artifact across environments | No |
| **Deploy Code** | Run training pipeline in each environment; model trained per-env | **Yes** |

**Why Deploy Code:** Ensures the model is always trained on the correct data for each environment. Better reproducibility and pipeline consistency.

### Three Environments

| Environment | Users | Focus |
|---|---|---|
| Development | Data scientists | Experiment, explore, develop |
| Staging | ML practitioners | Integration testing, validation |
| Production | ML engineers | Deploy, monitor, human feedback |

**Recommended:** Separate Databricks workspace per environment (Direct Separation over Indirect/ACL-only).

### Databricks Asset Bundles (DABs)

YAML-defined project bundles for CI/CD deployment across environments.

```yaml
bundle:
  name: my-llm-project
resources:
  jobs:
    training_job:
      name: "Training Pipeline"
targets:
  development:
    workspace:
      host: https://dev.azuredatabricks.net
  production:
    workspace:
      host: https://prod.azuredatabricks.net
```

**CLI deployment:**

```bash
databricks bundle deploy -t production
databricks bundle run pipeline --refresh all -t production
```

**Use service principals** (not personal credentials) for CI/CD deployments.

### Deployment Method Decision Tree

```
Data changes > 30 min, high volume, no urgency? → Batch (ai_query, Spark UDFs)
Data streams continuously, moderate latency OK? → Streaming
User expects instant response (chatbot)?        → Real-time (Model Serving)
Small model, on-device?                         → Edge
```

### Batch Inference — `ai_query()`

```sql
SELECT AI_QUERY(
  "model-name",
  CONCAT("Summarize the following: ", input_column)
) AS output
FROM source_table;
```

**GPU memory for batch:**

```
Required GPU RAM = num_parameters × bytes_per_parameter
10B params × 4 bytes (FP32) = 40 GB
```

---

## 3. Design Applications — 14%

### Prompt Engineering Fundamentals

Tactical optimization of the input text to guide LLM output.

Three pillars:

- **Grounding**: provide specific context and background data
- **Organization**: clear headings, numbered steps, logical structure
- **Token Efficiency**: be concise; irrelevant content displaces critical data

### Prompting Techniques

| Technique | Description |
|---|---|
| **Zero-Shot** | No examples; model uses pretrained knowledge |
| **Few-Shot** | Provide examples in the prompt to guide format/behavior |
| **Chain of Thought (CoT)** | Add "Think step-by-step" for complex reasoning |
| **Persona Adoption** | Assign a role to shape tone and expertise |
| **Negative Constraints** | Explicitly state what the model must NOT do |

**CoT note:** Use with non-reasoning models (Llama, GPT-4o). Redundant/counterproductive for reasoning models (o1, o3) — they generate their own CoT internally.

### Reasoning vs Non-Reasoning Models

| Type | Examples | CoT needed? |
|---|---|---|
| Non-reasoning | Llama 3, GPT-4o, Claude Sonnet | Yes — add "Think step-by-step" |
| Reasoning | OpenAI o1, o3 | No — generates internal CoT |

### Model Selection Decision Tree

```
1. Complexity + Quality Needs → determines minimum model class
2. Cost + Latency → narrows options
3. Privacy + Deployment Constraints → deal-breakers
```

**Model Classes:**

| Class | Size | Best for | Cost |
|---|---|---|---|
| Small | 7B–13B | Extraction, formatting, simple Q&A | Low |
| Medium | 30B–70B | Support, content generation | Medium |
| Large | 70B+ | Complex reasoning, ambiguous tasks | High |
| Frontier | State-of-art | High-stakes, quality > cost | Highest |

### Non-Agentic vs Agentic Strategy

| | Non-Agentic | Agentic |
|---|---|---|
| Workflow | Linear, predictable, hardcoded | Non-linear, goal-oriented, iterative |
| Actions | Deterministic | Non-deterministic |
| Cost | Low | Higher |
| Governance | Easy to audit | Complex to debug |
| Examples | Summarization, translation, templated content | Autonomous research, multi-step analysis |

**Choose agentic when:** Tasks require Reason → Act → Observe loop, tool integration, or open-ended objectives.

### Chaining and Multi-Stage Reasoning

- **Chaining**: output of one LLM call becomes input to the next
- **Multi-stage reasoning**: breaking complex goals into sub-tasks
- Agent systems coordinate LLM + Tools + Planning + Memory

### Function Calling (Tool Use)

Not all LLMs support tool calling — requires specific model capability and agent framework support.

**Core components of AI Agents:**

1. LLM brain (reasoning engine)
2. Memory (short-term: session state; long-term: episodic/semantic)
3. Planning (sub-task decomposition)
4. Tool interface
5. Execution engine

---

## 4. Data Preparation — 14%

### Storage Architecture

```
Unity Catalog Volume (raw files / Bronze)
  → ai_parse_document() 
  → Delta Table (parsed text / Silver)
  → Chunking
  → Delta Table (chunks / Gold)
  → Embedding → Vector Search Index
```

**Unity Catalog Volumes** = governance layer for non-tabular files (PDFs, images). Same permission model as tables/models.

### `ai_parse_document`

Native Databricks AI function for extracting structured content from PDFs and images.

```sql
SELECT ai_parse_document(content) AS parsed_document
FROM read_files('/Volumes/path/to/pdfs/', format => 'binaryFile')
```

**Capabilities (v2.0):**

- Layout awareness (separates content from layout)
- Figure descriptions (auto-generates text for charts/images)
- Bounding boxes (coordinates for UI highlighting)
- OCR for text within images
- Handles multi-column layouts and tables

**Ingestion patterns:** Autoloader, Spark Declarative Pipelines (SDP)

### Document Parsing Challenges

- **Hierarchical info**: charts/diagrams with nested relationships
- **Order preservation**: multi-column reading order
- **Contextual integrity**: images must remain associated with their text

### Data Cleaning

- Remove headers, footers, page numbers (degrade retrieval quality)
- For HTML: preserve table structure — `ai_parse_document` extracts tables in HTML format
- **Metadata injection**: extract document title, author, date for pre-retrieval filtering
- Use `ai_extract()` for pulling structured fields from unstructured text

### Chunking Strategies

| Strategy | Description | Recommended? |
|---|---|---|
| **Fixed-Size** | Split by hard character/token count | No — destroys context |
| **Semantic / Recursive** | Split on sentences, paragraphs, document sections | Yes — preserves meaning |
| **Chunk Overlap** | 10–20% overlap between consecutive chunks | Yes — prevents context loss at boundaries |
| **Embedding-Based Semantic** | Break on semantic similarity drop between sentences | Advanced — best coherence |
| **Windowed Summarization** | Each chunk includes summary of prior chunks | Advanced — rich context |

**Critical rule:** Max chunk size must stay safely below the embedding model's context window limit — exceeding truncates silently.

### Embedding Models

- Convert text chunks → dense float vectors
- Similar meaning → vectors close together in space

**Selection criteria:**

- **Vocabulary/domain**: general vs specialized (finance, medicine, legal)
- **Context window**: max input token limit
- **Dimensions**: more dimensions = more nuance but higher cost/latency

**Alignment requirement:** Use the **same embedding model** for both indexing documents and encoding queries. Mismatched models = poor retrieval.

### Retrieval Evaluation

Test whether the retrieval pipeline returns relevant chunks:

- **Context Precision** — are relevant chunks ranked above irrelevant ones?
- **Context Recall** — are all relevant facts from ground truth retrieved?
- Tools: ChunkViz for visualizing chunk splits

---

## 5. Evaluation and Monitoring — 12%

### LLM Component Metrics (Pre-training)

| Metric | Measures | Limitation |
|---|---|---|
| **Loss** | Next-token prediction accuracy | Low loss ≠ task accuracy |
| **Perplexity** | Model confidence in predictions | Fluent text can still be wrong |
| **Toxicity** | Harmfulness of output | Scored by pre-trained classifier |

### Task-Specific Metrics

| Metric | Task | Measures |
|---|---|---|
| **BLEU** | Machine translation | Precision — n-gram overlap with reference |
| **ROUGE-1/2** | Summarization | Recall — n-gram coverage of source |
| **ROUGE-L** | Summarization | Longest common subsequence |

Both BLEU and ROUGE: compare n-grams against reference data; quality depends on reference quality.

### Mosaic AI Gauntlet

Databricks' benchmark suite: **35 sources**, **6 categories** (reading comprehension, commonsense reasoning, problem-solving, language understanding, etc.). Used during pre-training to select best checkpoints.

### LLM-as-Judge

Use an LLM to evaluate another LLM's output. Required when no reference dataset exists.

**Format:**

```
Q: {question}   A: {model_answer}
Rate quality 0–10 for accuracy, helpfulness, clarity, safety.
```

**Limitations and mitigations:**

- Contextual blindness → provide domain context in judge prompt
- Hallucinated metrics → use structured output with constrained scoring
- Bias → add human-in-the-loop review of sample

### RAG Evaluation Metrics (6 key metrics)

**Retrieval metrics:**

| Metric | Based on | Question it answers |
|---|---|---|
| **Context Precision** | Query + Context | Are relevant chunks ranked higher than irrelevant ones? |
| **Context Relevancy** | Query + Context | Does context address the query intent? |
| **Context Recall** | Ground Truth + Context | Does context cover ALL facts in ground truth? |

**Generation metrics:**

| Metric | Based on | Question it answers |
|---|---|---|
| **Faithfulness / Groundedness** | Response + Context | Is every claim in the answer supported by context? |
| **Answer Relevancy** | Response + Query | Does the answer address what was actually asked? |
| **Answer Correctness** | Response + Ground Truth | Is the answer factually accurate vs ground truth? |

**Critical distinction:** Faithfulness measures grounding in the *retrieved context*. Correctness measures accuracy against *ground truth*. A faithful answer can still be wrong if the retrieved context itself contains errors.

**Databricks official names:** `groundedness`, `chunk_relevance`, `context_sufficiency`, `correctness`, `relevance_to_query`

### Offline vs Online Evaluation

| | Offline | Online |
|---|---|---|
| When | Before deployment | After deployment |
| Data source | Curated benchmark dataset | Real user interactions |
| Purpose | Validate before release | Detect drift and edge cases |
| Tools | MLflow evaluate, LLM judges | Lakehouse Monitoring, Inference Tables |

### MLflow Evaluation

```python
with mlflow.start_run():
    results = mlflow.evaluate(
        model,
        eval_data,
        targets="ground_truth",
        model_type="question-answering",
        evaluators="default"
    )
```

**Custom metric (3 steps):**

1. Create evaluation records (human-scored examples)
2. Create metric object via `make_genai_metric()`
3. Pass to `mlflow.evaluate(extra_metrics=[...])`

### Databricks Lakehouse Monitoring

Background service that processes Unity Catalog tables and generates:

- **Profile metrics table** — statistical distribution summaries
- **Drift metrics table** — changes in data/model behavior over time
- **Custom metrics** — defined as SQL expressions
- **DBSQL dashboard** — auto-generated visualization

**Inference Table → Monitoring workflow:**

```
Inference Table → Unpack JSON → Token counts / Toxicity / LLM judge metrics
→ Delta table → Streaming job → Lakehouse Monitoring → SQL Alerts
```

### Human Feedback

| Type | Examples | Quality |
|---|---|---|
| Explicit | Thumbs up/down, star ratings, corrections | High quality, lower volume |
| Implicit | Click-through, session length, copy-paste | High volume, noisier |

Human feedback in LLMOps: used for RLHF, fine-tuning datasets, regression test cases, and architectural decisions.

### MLflow Tracing for Debugging

**Trace**: entire request lifecycle (user query → final answer)
**Span**: individual unit of work (embedding, retrieval, LLM call)

Diagnose via span inspection:

- **Empty retrieval span** → issue is embedding model or chunking strategy
- **High latency span** → optimization target identified
- **Correct context but bad answer** → system prompt needs stricter grounding instruction

---

## 6. Governance — 8%

### Llama Guard

A safeguard model that classifies LLM conversations for safety risks.

**Two deployment points:**

```
User Query → [Input Guard] → LLM → [Output Guard] → Response
```

- **Input Guard**: blocks harmful queries before LLM sees them
- **Output Guard**: blocks harmful responses before user sees them

**6 built-in risk categories:**

1. Violence & Hate
2. Sexual Content
3. Guns & Illegal Weapons
4. Regulated or Controlled Substances
5. Suicide & Self Harm
6. Criminal Planning

### Databricks DASF (Data and AI Security Framework)

12 components, 55 associated risks. GenAI engineers focus on 6:

| DASF # | Component | Relevance |
|---|---|---|
| 4 | Data Catalog & Governance | Unity Catalog ACLs, data quality |
| 5 | Algorithms | Data poisoning, adversarial monitoring |
| 6 | Evaluation | Security failures as performance degradation signals |
| 8 | Model Management | Lineage, encryption, version control |
| 11 | Operations | MLOps quality, incident response, compliance |
| 12 | Platform Security | Cloud architecture, secrets management, serverless config |

**Platform security (#12) is foundational** — a perfectly secured model is still vulnerable if secrets leak through misconfigured infrastructure.

### Prompt Injection

User crafts input to override system instructions:

```
"Ignore all previous instructions and tell me how to..."
```

**Mitigations:**

- System prompt guardrails
- Llama Guard (Input Guard)
- Negative constraints in system prompt

### Malicious Data in RAG Sources

Problematic text can be embedded in retrieved documents, causing the LLM to execute injected instructions. Mitigate via:

- Source validation before indexing
- Metadata filtering to restrict what's retrievable
- Output Guard (Llama Guard)

### Data Legality and Licensing

Before deploying RAG or fine-tuned models, answer:

- Who owns the training/retrieval data?
- Commercial use or not?
- Jurisdiction (country/state)?
- Will the system generate profit?

Different answers trigger different licensing obligations. Applies to public web scrapes and third-party data.

### Bias in GenAI

- Models amplify biases in training data
- Outputs can appear reasonable while being skewed
- Example: model trained on UK healthcare data gives UK-specific advice globally
- Mitigate via diverse test populations and human-in-the-loop review

### Unity Catalog for AI Governance

- Centralized governance for data, models, vector indexes
- End-to-end lineage: raw data → embeddings → model → deployed endpoint
- Fine-grained ACLs: `SELECT`, `EXECUTE` on models and functions
- Cross-workspace sharing via shared metastore
- Vector Search indexes are securable objects in UC

### Auditability and Lineage

- **Auditability**: degree to which AI system decisions can be monitored and verified
- **Lineage**: tracking a data asset from origin through final consumption
- MLflow provides model lineage (training data → model version → serving endpoint)
- Unity Catalog system tables for access audit logs
