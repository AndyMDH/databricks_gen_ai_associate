# GenAI Application Deployment and Monitoring

---

## 1. Model Management

### 1.1 The Gen AI System Lifecycle

The lifecycle starts with a well-defined business problem and proceeds through evaluation, deployment, and continuous monitoring with feedback loops.

```
Business Problem
  → Define Success Criteria
  → Data Collection & Preparation
  → AI System Building (fine-tuning, pre-training, RAG chains)
  → Offline Evaluation
  → Deployment & Production
  → Monitoring
  → (feedback loop back to Data Collection / Building)
```

**Two parallel tracks after building:**

- **System Development** — uses static data for R&D and evaluation
- **Deployment & Production** — deals with continuously changing new data

**Key principle:** Establish existing processes and baseline models before building. Monitor continuously and feed real-world data back into development for improvement.

---

### 1.2 GenAI Deployment Packaging Forms

When packaging a GenAI model or pipeline for deployment, the logic can take several forms:

| Form | Description |
|---|---|
| Engineered prompt | Instruction stored as a template |
| LangChain / LlamaIndex chain | Chain-based orchestration framework |
| Lightweight LLM API call | Internal/self-hosted foundation model or external provider (e.g., OpenAI) |
| External proprietary model | Direct call to OpenAI, Anthropic, etc. |
| Fine-tuned model | Custom-trained on domain data |
| Pretrained model | Off-the-shelf base model |
| LLM + tokenizer pipeline | Locally running on GPU(s), e.g., Hugging Face pipeline |

**All forms are still "models and pipelines"** — any packaging approach defines how the GenAI model/chain is deployed and managed.

---

### 1.3 MLflow Components

**Definition:** MLflow is an open-source tool (not limited to Databricks) that manages ML and GenAI workflows from development through production.

**Key capabilities:**

- Manage end-to-end ML and GenAI workflows
- Unified platform for both traditional ML and GenAI applications
- GenAI-specific model flavors and evaluation metrics
- Combined with Unity Catalog: organize and track all data and AI assets in one place
- Lifecycle tracking at every step

**MLflow is the solution for packaging complex generative AI systems cleanly through model flavors.**

---

### 1.4 MLflow Model Flavors

**Definition:** A standard format for packaging ML models. Each MLflow Model is a directory containing arbitrary files and an `MLmodel` file that specifies one or more *flavors*.

**Structure example (LangChain flavor):**

```
my_model/
  MLmodel
  metadata.yaml
  requirements.txt
  artifacts/
    chain.pkl
    ...
```

**What a flavor provides:**

- Multiple views/interfaces a model can be used through
- Additional metadata: signature, input examples, runtime versions, dependencies
- Standard functions: `load_model`, `save_model`, `predict`

**Built-in model flavors:**

| Flavor | Use Case |
|---|---|
| `mlflow.langchain` | LangChain chains and agents |
| `mlflow.openai` | OpenAI API calls |
| `mlflow.transformers` | Hugging Face Transformers |
| `mlflow.pytorch` | PyTorch models |
| `mlflow.onnx` | ONNX models |
| `mlflow.pyfunc` | Any custom Python code (default/fallback) |

**`mlflow.pyfunc` — the default model interface:**

- Any MLflow Python model is expected to be loadable as a Python function
- Use when your logic doesn't fit a pre-supported flavor (LangChain, OpenAI, etc.)
- Wrap any custom Python code or API calls inside a `predict()` method
- Enables tracking and management of models regardless of framework

```python
import mlflow.pyfunc

class MyCustomModel(mlflow.pyfunc.PythonModel):
    def predict(self, context, model_input):
        # custom logic — chain, API call, etc.
        return result
```

**Why flavors matter for GenAI:** MLflow records all steps, dependencies, environment settings, and runtime versions. Makes it easy to package, reproduce, and manage even complex multi-step logic as a single unified MLflow model.

---

### 1.5 MLflow and the Development Lifecycle

```
Model/Chain Building
  → MLflow Tracking & Evaluation (experiment tracking, different flavors)
  → MLflow Registry (Unity Catalog) — versioning, aliases, staging
  → Model Deployment (Databricks Model Serving, cloud, local)
```

**Process steps:**

1. Create different versions of model/chain using suitable flavor
2. Use MLflow to automatically track metadata, metrics, parameters
3. Once a version performs well in offline evaluation, push to MLflow Registry
4. Registry manages versioning and deployment (tag with `@champion`, `@challenger`)
5. Deploy from registry to serving endpoint

**Three benefits of MLflow for deployment:**

1. **Dependency & Environment Management** — ensures deployment environments match training environments; consistent behavior regardless of location
2. **Packaging Models and Code** — bundles code, config, and artifacts into a single package; no missing components on deployment
3. **Multiple Deployment Options** — built-in Flask/MLServer, major cloud providers, Databricks Model Serving

---

### 1.6 Unity Catalog for Model Governance

**Definition:** Single governance solution for all data and AI assets on the Lakehouse.

**Key features:**

- **Unified visibility** — discover and understand data and AI assets in one place
- **Single permission model** — consistent access control for data and models
- **Open data sharing** — open protocol for sharing models with third parties and outside organizations

**Assets managed:** Tables, Files, Models, Notebooks, Dashboards

**UC provides end-to-end lineage** between data and models, making assets organized, secure, and shareable while maintaining traceability and control throughout the lifecycle.

---

### 1.7 MLflow Unity Catalog Model Registry

**Capabilities:**

| Feature | Description |
|---|---|
| Model lifecycle management | Version models with aliases: `@champion`, `@challenger` |
| Deploy and organize models | Promote models across environments |
| Collaboration and ACLs | Control who can access, manage, create versions, promote |
| Full model lineage | Track upstream data and downstream deployments |
| Tagging and annotations | Tag by business keywords, project, or stage |

**Workflow:** Register model → Assign alias → Control access → Promote to production

---

## 2. Deployment Methods

### 2.1 Deployment Paradigms Overview

**Definition:** Gen AI model deployment is the process of integrating an AI model into a production environment, making it accessible for end-users or other systems to generate predictions or completions.

**Four deployment strategies:**

| Strategy | Latency | Throughput | Description |
|---|---|---|---|
| **Batch** | High (hours–days) | High | Predictions on scheduled intervals |
| **Streaming** | Moderate (seconds–minutes) | Moderate | Micro-batches as data is processed |
| **Real-time** | Low (milliseconds) | Low–High | Asynchronous, instant responses |
| **Edge/Embedded** | Low (device-dependent) | Low | On-device inference; challenging for LLMs |

> **Note:** Edge deployment is challenging with large language models due to memory/size requirements.

---

### 2.2 Deployment Methods Comparison Table

| Deployment Method | Throughput | Latency | Example Application |
|---|---|---|---|
| Batch | High | High (hours to days) | Summarizing financial reports, generating insights |
| Streaming | Moderate | Moderate (seconds to minutes) | Personalizing marketing messages |
| Real-time | Low–High | Low (milliseconds) | Chatbots (customer service, doc assistant) |
| Edge/Embedded | Low | Low (device-dependent) | Voice commands in a car |

**Focus areas for the exam:** Batch and real-time deployment.

---

## 3. Batch Deployment

### 3.1 What is Batch Deployment?

**Definition:** Generates model predictions or completions on a **regular schedule**, writing results to persistent storage (table, BI dashboard, ad-hoc).

**Batch deployment is the simplest deployment strategy.**

**Ideal conditions:**

- Immediate predictions are not necessary
- Predictions can be made in a batch fashion
- High volume of new records/observations
- **The pace at which input/records change is > 30 minutes** (key decision rule)

---

### 3.2 Typical Use Case

**Automated legal research:**

- Legal databases are continuously updated with new case laws, statutes, and literature
- AI system collects, preprocesses, and summarizes new data on a schedule
- Output: newsletter or report highlighting important legal developments
- Classic LLM-based summarization process run as a daily/weekly batch job

---

### 3.3 Advantages and Limitations

| Advantages | Limitations |
|---|---|
| Cheapest deployment method | High latency |
| Easy to implement | Results can become stale |
| Efficient per data point | Not suitable for dynamic/rapidly changing data |
| Can handle high volumes | Not suitable for streaming or real-time applications |

**Operational note:** Important to have monitoring and quality checks to ensure fresh data is being used.

---

### 3.4 Typical Batch Workflow

```
Data Lake / UC Volume (Training Data)
  → Build Model (download weights from Model Hub)
  → Log model to MLflow (transformer flavor)
  → MLflow UC Model Registry
  → Batch Inference (Python predict() / Spark UDF)
  → Data Lake (inference results)
  → Downstream consumers (BI, SQL)
```

**Two inference paths from the registry:**

1. **Python Batch Inference** — `predict()` method, scalable via Spark UDFs across multiple nodes
2. **SQL path** — `ai_query()` for direct SQL-based invocation on Foundation Models

---

### 3.5 Batch Inference from SQL: `ai_query()`

**Definition:** A Databricks SQL function that runs batch invocations on Foundation Models deployed within the platform. Sends multiple records at once, automatically parses completions into structured results.

**Syntax:**

```sql
SELECT AI_QUERY(
  "databricks-dbrx-instruct",
  CONCAT(
    "Based on the following customer review, answer to ensure satisfaction. Review: ",
    review
  )
) AS generated_answer
FROM reviews;
```

**Use case in the example:** Given a customer reviews table, determine satisfaction/intent and return a structured answer per row.

**Key properties:**

- Invokes Foundation Models API from Databricks SQL
- Handles batching and parsing automatically
- Returns clean, usable results without manual post-processing
- Integrates directly into SQL workflows

---

### 3.6 GPU Scaling Challenges

Scaling batch inference with large models is **not friction-free**.

**GPU memory requirement formula:**

```
Required GPU RAM = num_parameters × bytes_per_parameter
Example: 10B params × 4 bytes (FP32) = 40 GB GPU RAM
```

**Challenges:**

1. **Hardware access** — large models require GPUs with high memory (A100, V100); expensive to acquire/provision
2. **GPU utilization** — maximizing cost efficiency is non-trivial even with the right hardware
3. **Parallelization** — distributing inference across multi-node, multi-GPU clusters is complex

> Reference: **Databricks Blog: LLM Inference Best Practices**

---

### 3.7 OSS Integration Libraries

For custom batch inference on Databricks, open-source libraries are available:

| Library | Description |
|---|---|
| **TensorRT** | NVIDIA SDK for high-performance batch inference on GPUs; TensorFlow-friendly |
| **vLLM** | Memory-efficient inference on NVIDIA & AMD GPUs; supports DBRX, Mistral |
| **Ray on Spark** | Python distributed computing primitives for parallelizing/scaling Python apps across AWS/Azure/GCP |

Databricks provides example notebooks for TensorRT and vLLM deployments. These libraries handle memory management but setup is still non-trivial.

---

## 4. Real-Time Deployment

### 4.1 What is Real-Time Deployment?

**Definition:** Serving machine learning models in a production environment where **predictions are generated instantly in response to incoming data or requests**.

**Critical requirement:** Low-latency responses (milliseconds).

**Common use cases:**

- Chatbots
- Message intent detection
- Autonomous systems
- Any time-sensitive task

With the rise of GenAI applications, real-time deployment is increasingly the default — large language models are expected to deliver results instantly.

---

### 4.2 Typical Use Case

**Real-time intent detection for a social media platform:**

- Fine-tuned LLM deployed for classifying social media posts
- Model provides immediate classification over a REST API
- If a post is classified as toxic/violent/harmful → taken down immediately

**Requirements:**

- Low latency
- Immediate action on classification
- 24/7 uptime
- Continuous monitoring

---

### 4.3 Typical Real-Time Workflow

```
Data Lake / UC Volume (Training Data)
  → Build Model (create model/pipeline/vectorization)
  → Log model to MLflow (transformer flavor)
  → MLflow UC Model Registry
  → Databricks Model Serving (Serving Endpoint)
        ↑ Vector Search (for RAG)
  → Downstream App (web, chatbot, mobile)
        → Monitoring (all queries logged)
```

**Key point:** The model development process does not change — whether using a marketplace model, fine-tuning, or building from scratch. Once registered, deploying to a Model Serving endpoint is straightforward.

---

### 4.4 Challenges of Real-Time AI Systems

**Three core challenges:**

| Challenge | Detail |
|---|---|
| **Infrastructure is hard** | Fast, scalable serving infrastructure is costly to build and maintain; GPU hardware is expensive and utilization is difficult |
| **Deploying requires disparate tools** | Data teams use diverse frameworks; separate platforms for data, LLMs, and serving add complexity and cost |
| **Operating production AI requires expert resources** | Steep learning curve; deployment is time-consuming; limited engineering resources limit the ability to scale AI |

---

### 4.5 Databricks Model Serving Overview

**Purpose:** "Integrate your model into your websites and applications as an API"

**Three key features:**

1. **Production-Grade Serving** — highly available, low latency, scalable serving for small and large workloads
2. **Lakehouse-Unified Serving** — automatic feature lookups, monitoring, and unified governance that automates deployment and reduces errors
3. **Simplified Deployment** — deploy through UI or API with just a few clicks

---

## 5. Databricks Model Serving

### 5.1 MLflow Integration and Lifecycle

```
RAG/Chain Building
  → MLflow Tracking and Evaluation
  → MLflow Registry
  → Databricks Model Serving
```

Model Registry manages versioning and staging; Model Serving deploys any registered model version as a REST endpoint. The unified approach reduces operational costs and lets Data Science teams focus on integration rather than infrastructure.

---

### 5.2 Unified API: Custom, Foundation, and External Models

Databricks Model Serving provides a **unified UI, API, and SDK** for three model types:

| Model Type | Description |
|---|---|
| **Custom Models** | Deploy any model as a REST API with serverless compute |
| **Foundation Models APIs** | Databricks-curated foundation models (DBRX, Llama, etc.) with experimentation capabilities |
| **External Models** | Access proprietary models and external APIs (OpenAI, Anthropic, etc.) |

**Key benefit:** Switch between model types without rewriting backend code. The same API surface works for all three.

---

### 5.3 Production-Ready Features

- **Serverless, production-grade** solution for real-time ML model deployment
- Deploy models as APIs for integration with applications and websites
- **Built-in payload logging** — automatic request recording to Delta tables for analysis
- **Infrastructure metrics** available through API
- **Autoscaling** — serverless compute management handles load automatically
- Built-in **A/B testing** capabilities
- Works with custom, foundational, and external models

---

### 5.4 Online Evaluation and A/B Testing

Databricks Model Serving supports **canary deployments** and A/B testing by serving multiple models to a single endpoint with traffic splitting:

**Example:**

```
Endpoint: /serving/my-app
  → 80% traffic → Champion Model v1
  → 20% traffic → Challenger Model v2
```

- Real-time production metrics feed back into evaluation before full rollout
- Enables safe, data-driven model promotions

---

### 5.5 Inference Tables

**Definition:** A feature for effective real-time monitoring of deployed systems. Automatically logs all request-response pairs with metadata to Unity Catalog.

**Components stored in Unity Catalog:**

| Component | Description |
|---|---|
| UC Volume | Raw data storage |
| Raw/Processed text tables | Processed feature data |
| Embeddings/Index | Vector representations |
| Model/Chain metadata | Model configuration |
| **Inference table** | Request-response records with full metadata |

**What inference tables enable:**

- Monitor and debug models in production
- Perform diagnostics on suspicious inferences
- Create mislabeled/relabeled data for retraining
- Use real-world data for model improvement and edge case handling

---

## 6. AI Application Monitoring

### 6.1 Why Monitor AI Systems?

**Goal:** Diagnose issues before they become severe or costly.

**What to monitor:**

| Category | Examples |
|---|---|
| **Data** | Input data, raw data, features, completions, index changes, data shifts, embeddings, human feedback |
| **AI Assets** | Mid-training checkpoints, calibration checks, system evaluation metrics, performance details |

---

### 6.2 Databricks Lakehouse Monitoring

**Definition:** Fully managed, automated insights on data and ML pipelines built on the Lakehouse.

**Key properties:**

- **Frictionless setup** with out-of-box metrics and auto-generated dashboards
- **Unified solution** for both data and models (holistic understanding)
- **No infrastructure management** required
- **Data-centric** approach built on the Lakehouse

---

### 6.3 Unity Catalog Integration

Lakehouse Monitoring is a **background service** that incrementally processes data in Unity Catalog tables.

**For each monitored table, it generates:**

| Output | Description |
|---|---|
| **Profile metrics table** | Distribution and statistical summaries |
| **Drift metrics table** | Tracks changes in data/model behavior over time |
| **Custom metrics** | Defined as SQL expressions |
| **DBSQL dashboard** | Auto-generated visualization of metrics over time |

**Additional capabilities:**

- Automatic PII detection and redaction
- Input expectations and data quality rules

---

### 6.4 Monitoring Model Responses (Inference Table Workflow)

**Full workflow for monitoring LLM responses:**

```
Inference Table (raw request-response logs)
  → Unpack JSON payloads into columns
  → Calculate evaluation metrics:
       - Token counts
       - Toxicity scores
       - Perplexity scores
       - LLM-judged quality metrics
  → Materialize into processed table (Delta)
  → Schedule as triggered streaming job
  → Enable Lakehouse Monitoring on processed table
  → Analyze dashboard + create SQL alerts
```

**Result:** Continuous monitoring enabling early anomaly detection. Delta table allows streaming processing of newly arriving inference data.

---

### 6.5 Lakehouse Monitoring Workflow Tips

**Recommended setup steps:**

1. Set up monitoring tables for all components (data, models, endpoints)
2. Track and record model performance baseline
3. Refresh tables and dashboard regularly
4. Set up key monitoring alerts on critical metrics
5. Connect rerun triggers to automate performance tracking

**Environment progression:**

```
Development → Testing (integration tests for component validation)
           → Production (baseline + monitoring tables in prod catalog)
```

---

## 7. MLOps Primer

### 7.1 What is MLOps?

**Definition:** Set of processes and automation for managing data, code, and models to improve performance, stability, and long-term efficiency.

**Formula:**

```
MLOps = DataOps + DevOps + ModelOps
```

**MLOps rigor ensures quality through:**

- Data quality
- Streamlined production processes
- Operational cost and performance monitoring

---

### 7.2 Why MLOps Matters

**Real-world example — CareSource (Databricks customer):**

- Used a self-service MLOps solution
- Reduced model development project time from **8 weeks to 3–4 weeks**

**MLOps success depends on:**

- Quality data
- Streamlined processes
- Operational practices

**Benefits:** Accelerated time to business value, reduced manual oversight, catch problems early.

---

### 7.3 Multi-Environment Semantics

| Environment | Role | Users |
|---|---|---|
| **Development** | Explore, experiment, develop | Data scientists |
| **Staging** | Test solutions | ML practitioners |
| **Production** | Deploy and monitor | ML engineers |

Progression through environments ensures data quality, streamlined processes, and integrated operational approaches.

---

### 7.4 Environment Separation Approaches

**Two approaches:**

| Approach | Setup | Pros | Cons |
|---|---|---|---|
| **Direct Separation** | Separate Databricks workspaces per environment | Simpler individual environments; scales to multiple projects; less permission overhead | More infrastructure to manage |
| **Indirect Separation** | Single workspace with Unity Catalog ACL enforcement | Simpler infrastructure | Complex per-environment management; doesn't scale well to multiple projects |

---

### 7.5 Deployment Patterns: Deploy Model vs Deploy Code

**Deploy Model:**

```
Train in Development → Move model artifact to Staging/Production
(Other code, monitoring, operational elements handled separately)
```

**Deploy Code (Recommended):**

```
Develop code in Development
  → Test in Staging (training pipeline runs here)
  → Deploy to Production (training pipeline runs again, model deployed)
```

**Why "Deploy Code" is recommended:** The training pipeline runs in all environments, ensuring the model is always trained on the correct data for that environment. Better for comprehensive pipeline management and reproducibility.

---

### 7.6 Recommended Deploy-Code Architecture

```
Development Workspace
  → Data Scientists experiment and develop
  → Unit tests via CI/CD tools

Staging Workspace
  → ML Practitioners test
  → Integration tests via CI/CD tools (Git Actions, Azure DevOps)
  → Validation workflows

Production Workspace
  → ML Engineers deploy and monitor
  → Monitoring metrics written to Lakehouse
  → All environments share single data source of truth
```

---

### 7.7 Single Platform for Modern MLOps

**Databricks unifies three operational domains:**

| Domain | Components |
|---|---|
| **DataOps** | Model Registry, Model Serving, Lakehouse Monitoring |
| **DevOps** | Repos, code management, version control, workflow scheduling, infrastructure |
| **ModelOps** | Workflows, Databricks Asset Bundles (DABs) |

**Foundational layer:**

- **Data Intelligence Engine** — semantic understanding, quality metrics
- **Unity Catalog** — central governance
- **Delta Lake** — central storage
- **Lakehouse Architecture** — unified data and AI platform

---

## 8. LLMOps

### 8.1 LLMOps vs MLOps: Key Differences

| Dimension | MLOps | LLMOps |
|---|---|---|
| **Dev Patterns** | Train custom models from data | Start with off-shelf external models; move toward customization |
| **Packaging** | Single model artifact | Entire application (chain, APIs, vector DBs, UI) |
| **Serving** | Model inference endpoint | ML + operations for output storage; API performance critical |
| **Governance** | Model version control | Control and governance of API usage; prevent misuse |
| **Cost & Performance** | Compute/throughput metrics | Token-based metrics; input/output token costs |
| **Human Feedback** | Optional/limited | Essential for evaluation and iteration guidance |

---

### 8.2 Development Patterns

**LLMOps follows incremental development:**

```
Off-shelf external model (API)
  → Prompt engineering
  → RAG / retrieval augmentation
  → Fine-tuning
  → Custom pre-training (if needed)
```

**Key difference from classical ML:** Begin with APIs, customize over time. Prompt engineering and text templates are critical new aspects — they become core components of the LLM pipeline.

---

### 8.3 Packaging Artifacts

**LLMOps packages entire applications, not single model artifacts:**

- Complex pipelines that may call other LLMs or GenAI services
- Each service in the chain requires separate environment configurations
- Tokens, credentials, and access credentials must be managed across all connected services

**Packaging must account for:**

- All model dependencies and external API calls
- Environment-specific configurations (dev/staging/prod)
- Secrets and credential management

---

### 8.4 Serving Applications

**Additional components in LLMOps serving:**

- **Vector databases** for retrieval augmentation
- **GPU infrastructure** for model serving and deployment
- **User Interface** (Gradio, Streamlit) may be part of the serving deployment
- **Multi-turn conversation state** management
- **Databricks Lakehouse Apps** — enables containerized deployment at scale

---

### 8.5 API Governance

**Managing access:**

- Govern endpoints for various project components
- Log all endpoint calls for evaluation and audit
- Set guardrails to prevent abuse

**Governing usage:**

- **Token rate limits** per group or business unit
- Prevent token leaks and credential overuse
- Unified API provides central governance for all model types

---

### 8.6 Cost and Performance

**Cost drivers:**

| Factor | Impact |
|---|---|
| Model size | Larger models cost more (pre-training is highest cost) |
| API pricing | Token-based: input tokens + output tokens |
| Fine-tuning | Can lower per-query cost by using a smaller fine-tuned model |

**Cost reduction techniques:**

- Reduce model size (distillation, quantization)
- Cache queries when possible (avoid redundant API calls)
- Reduce input/output token counts (prompt optimization)

---

### 8.7 Human Feedback

**Collection methods:**

| Type | Examples |
|---|---|
| Explicit feedback | Thumbs up / thumbs down ratings |
| Implicit feedback | User queries, follow-up corrections, suggestions |

**Uses of collected feedback:**

- Response examples for fine-tuning datasets
- Test cases for regression testing
- Inform architectural and strategic decisions
- Reinforcement learning from human feedback (RLHF)
- Rich data for continuous iteration

**Human feedback is central to LLMOps** — it's the primary mechanism for measuring quality and guiding improvement.

---

### 8.8 Adapting MLOps for LLMs

**Recommended LLMOps architecture per environment:**

| Environment | Focus |
|---|---|
| **Development** | Exploratory data analysis, system testing, infrastructure setup |
| **Staging** | Workflow execution testing, performance regression tests |
| **Production** | Gathering and prioritizing human feedback, managing retraining data |

**Key practices:**

- Single project code repository used across all environments
- Separate Databricks workspace per environment
- Single data/system component management solution with environment-specific catalogs
- Iterative development with model quality checks before every deployment
- Human feedback loop continuously feeds improvement

---

### 8.9 Databricks Asset Bundles (DABs)

**Definition:** A tool for production-ready LLM operations. DABs define artifacts, resources, and configurations using YAML files.

**Why use DABs:**

- Automation-friendly: CLI commands instead of thousands of REST API calls
- Co-version configs with code and resources in the same repository
- Collaborative and confident deployment through isolation and automated testing
- Democratizes CI/CD and project management best practices

**`bundle.yml` structure:**

```yaml
bundle:
  name: my-llm-project

workspace:
  host: https://<databricks-workspace>.azuredatabricks.net

resources:
  jobs:
    my_training_job:
      name: "LLM Training Pipeline"
      tasks:
        - task_key: train
          notebook_task:
            notebook_path: ./notebooks/train.py

  pipelines:
    my_dlt_pipeline:
      name: "Data Ingestion Pipeline"

targets:
  development:
    mode: development
    workspace:
      host: https://dev.azuredatabricks.net

  staging:
    workspace:
      host: https://staging.azuredatabricks.net

  production:
    workspace:
      host: https://prod.azuredatabricks.net
```

**Bundle YAML three key sections:**

1. **Name and default workspace** — project context
2. **Resource configurations** — Jobs, DLT pipelines, MLflow experiments, etc. (follows REST API schema)
3. **Environment-based specs (targets)** — controls project behavior differently per environment

**When to use DABs:**

- During development and CI/CD processes
- Deploying to multiple workspaces for testing differences
- Running projects from IDEs, terminals, or Databricks

---

### 8.10 CI/CD Integration with DABs

**DABs are the core of the automation pipeline:**

```
Code merged to main branch (GitHub, Azure DevOps)
  → CI/CD server triggers (GitHub Actions)
  → Deploy bundle:
      databricks bundle deploy -t "development"
  → Run pipeline:
      databricks bundle run pipeline --refresh all -t "development"
  → Automated tests run
  → Promote to staging, then production
```

**Key CI/CD properties:**

- Bundles executed on CI/CD server (e.g., GitHub Actions)
- Triggered automatically on code merge
- Use **service principals** for secure, traceable deployments (not personal credentials)
- Environment-specific targets control which workspace receives the deployment

**Benefits of DABs + CI/CD:**

- Consistent, repeatable deployments across environments
- Reduced manual intervention and human error
- Full audit trail via service principal execution
- Code, configs, and resources versioned together

---

## Key Exam Concepts Summary

### Decision Framework: Choosing a Deployment Method

```
Data changes every > 30 minutes, high volume, no urgency?  → Batch
Data streams continuously, moderate latency OK?            → Streaming
User expects instant response (chatbot, classification)?   → Real-time
Small model, on-device processing required?                → Edge
```

### Core Databricks Components

| Component | Purpose |
|---|---|
| **MLflow Model Flavors** | Standardized packaging for ML/GenAI models (LangChain, pyfunc, Transformers, etc.) |
| **Unity Catalog Model Registry** | Versioning, aliases (`@champion`/`@challenger`), ACLs, lineage |
| **Databricks Model Serving** | Production REST endpoints for custom, foundation, and external models |
| **Foundation Models API** | Databricks-curated models (DBRX, Llama) served via unified API |
| **Inference Tables** | Auto-logged request-response records in Unity Catalog for monitoring |
| **Lakehouse Monitoring** | Automated profile/drift metrics tables + DBSQL dashboards |
| **Databricks Asset Bundles** | YAML-defined project bundles for CI/CD and multi-environment deployment |

### MLOps Formula

```
MLOps = DataOps + DevOps + ModelOps
```

### LLMOps vs MLOps Key Additions

- Text templates / prompt engineering as code artifacts
- Package entire applications (not single models)
- Token-based cost tracking
- API governance with rate limits
- Human feedback as primary quality signal
- Databricks Asset Bundles for production deployment

### `ai_query()` — Exam Syntax

```sql
SELECT AI_QUERY(
  "model-name",
  CONCAT("system prompt: ", input_column)
) AS output
FROM table_name;
```

### GPU Memory Rule

```
Required GPU RAM = num_parameters × bytes_per_parameter
10B params × 4 bytes (FP32) = 40 GB
```

### Environment Pattern (Both MLOps and LLMOps)

```
Development → Staging → Production
(separate Databricks workspace per environment is recommended)
```
