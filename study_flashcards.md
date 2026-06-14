# Databricks GenAI Engineer — Flashcards

> ~90 Q&A pairs weighted by exam section. Study App Development (30%) most.

---

## App Development (30%) — 27 cards

**Q1:** What are the three stages of RAG?
**A:** Retrieval (find relevant chunks via vector search) → Augmentation (inject chunks into context window) → Generation (LLM synthesizes answer using provided context only)

---

**Q2:** What is "Context Rot" and what are its two failure modes?
**A:** Context Rot occurs when too much data is retrieved and dumped into the prompt. Two failure modes: (1) **Context Poisoning** — irrelevant/conflicting chunks confuse the model; (2) **Lost in the Middle** — models ignore information buried in the middle of a long context, prioritizing the beginning and end.

---

**Q3:** What is the difference between RAG and a Retrieval Agent?
**A:** RAG is an architectural pattern (retrieve + augment + generate). A **Retrieval Agent** is a concrete implementation that operationalizes RAG — handling query routing, retrieval orchestration, and context assembly within real systems.

---

**Q4:** What are the four components of Context Engineering?
**A:** System Instructions, Conversation History, Retrieved Data, User Constraints. Context Engineering designs the entire input window — not just the user's query.

---

**Q5:** What are the three key parts of an effective system prompt?
**A:** (1) **Role Definition** — assign a persona ("You are a Databricks Security Architect"); (2) **Negative Constraints** — what the model cannot do; (3) **Output Formatting** — enforce JSON, YAML, Markdown tables for deterministic downstream parsing.

---

**Q6:** What is a grounding instruction? Give an example.
**A:** An explicit instruction that binds the model to only the retrieved context. Example: "Answer using ONLY the provided context chunks. If the answer is not present, state 'I do not have that information.'"

---

**Q7:** What are the three Vector Search ingestion modes?
**A:** (1) **Managed Embeddings (Delta Sync)** — Databricks auto-computes embeddings; (2) **Self-Managed Embeddings (Delta Sync)** — you compute and store embeddings in a Delta table, index syncs automatically; (3) **Direct Vector Access (CRUD API)** — insert/update/delete vectors directly, no underlying Delta table.

---

**Q8:** What is Delta Sync in the context of Vector Search?
**A:** A feature where the vector index automatically syncs when the source Delta table is updated. Add/update/delete data → index updates automatically. No manual re-indexing required.

---

**Q9:** What are the three search methods in Mosaic AI Vector Search?
**A:** (1) **Similarity Search** — semantic/vector cosine distance; (2) **Full-Text Search** — keyword matching; (3) **Hybrid Search** — combines both, typically delivers highest retrieval accuracy.

---

**Q10:** What is the difference between KNN and ANN search?
**A:** **KNN** (K-Nearest Neighbors) is exact — calculates distance to every vector; highly accurate but doesn't scale. **ANN** (Approximate Nearest Neighbors) uses indexing algorithms (HNSW, FAISS) to approximate results; much faster with small accuracy trade-off. ANN is used in production.

---

**Q11:** Why use Cosine Similarity for text embeddings instead of Euclidean distance?
**A:** Cosine measures the angle between vectors (semantic orientation) rather than absolute distance. This makes it robust to document length differences — a short and long document on the same topic will still have high cosine similarity.

---

**Q12:** What is reranking and why is it used?
**A:** After ANN retrieves a broad candidate set (top 20–50), a Cross-Encoder reranker rescores each candidate against the specific query. The top 3–5 most relevant chunks are kept. **Benefits:** improves precision, reduces hallucinations. **Trade-off:** adds latency and cost.

---

**Q13:** What is "Just-in-Time Retrieval"?
**A:** Instead of loading a full document set at the start of a conversation, give the agent a tool to retrieve specific sections only when the user asks a relevant question. Reduces token costs and avoids context bloat.

---

**Q14:** What are the three strategies for managing multi-turn conversation context?
**A:** (1) **Summarization** — compress conversation history into key decisions/facts; (2) **Moving Window** — discard oldest messages to free token space; (3) **Selective Persistence** — keep only critical state (user name, project ID) permanently.

---

**Q15:** What is Metadata Filtering in RAG and why does it matter?
**A:** Using Unity Catalog metadata to filter retrieval before vector similarity search. Example: if user asks about "2024 Revenue," filter chunks where `year=2024` to prevent the model from seeing outdated 2023 data. Prevents hallucinations from stale information.

---

**Q16:** What is an MLflow Trace vs. a Span?
**A:** A **Trace** represents the entire request lifecycle (user query → final answer). A **Span** represents a single unit of work within the trace (e.g., "query_embedding", "retrieval_tool", "context_generation"). Spans follow the OpenTelemetry standard.

---

**Q17:** If an agent's retrieval span returns the correct document but the answer ignores it, what does that indicate?
**A:** A **reasoning failure** — the LLM is not adhering to the retrieved context. Fix by refining the system prompt to enforce strict grounding: "Answer ONLY using the provided context."

---

**Q18:** If the vector_search span takes 4 seconds while the LLM span takes 500ms, where should you optimize?
**A:** Optimize the **database query / vector search** — the trace waterfall shows the bottleneck is retrieval, not the LLM.

---

**Q19:** What are the three Unity Catalog tool types for agents?
**A:** (1) **Unity Catalog Function tools** (SQL or Python UDFs) — primary, governed, discoverable; (2) **Agent-code tools** — defined in agent code (REST APIs, arbitrary code) — less governance; (3) **MCP tools** — Model Context Protocol standard for cross-agent interoperability.

---

**Q20:** What are the key differences between SQL and Python agent tools in Unity Catalog?
**A:** **SQL**: optimized for data querying/analytics, `CREATE OR REPLACE FUNCTION`, serverless-only, no custom dependencies. **Python**: custom logic, external API integration, flexible execution (serverless + local), requires type hints and Google-style docstrings.

---

**Q21:** What is the ResponsesAgent interface?
**A:** MLflow's recommended production interface for agents. Inherits from `mlflow.pyfunc.ResponsesAgent`, implements `predict(request: ResponsesAgentRequest) → ResponsesAgentResponse`. Compatible with OpenAI Responses schema. Replaces the older `ChatAgent` interface.

---

**Q22:** What is Chain of Thought prompting and when should you NOT use it?
**A:** CoT adds "Think step-by-step" to prompt a non-reasoning model to break down complex logic before answering. **Do NOT use with reasoning models** (o1, o3) — they generate their own internal CoT; adding explicit CoT is redundant or counterproductive.

---

**Q23:** What does it mean for embeddings to be "aligned"?
**A:** Query and document embeddings must be created using the **same embedding model**. If different models are used, vectors exist in different mathematical spaces and cannot be meaningfully compared — leading to poor retrieval.

---

**Q24:** What is the "Lost in the Middle" phenomenon?
**A:** LLMs tend to ignore information buried in the middle of a long context window, paying more attention to content at the beginning and end. Mitigation: use reranking to put the most relevant chunks first, and keep context concise.

---

**Q25:** What are Databricks AI Bridge packages?
**A:** Integration packages connecting agents to Databricks AI features: `databricks-openai` (OpenAI), `databricks-langchain` (LangChain/LangGraph), `databricks-dspy` (DSPy), `databricks-ai-bridge` (pure Python agents). Built on top of Mosaic AI Agent Framework.

---

**Q26:** Name four supported agent authoring frameworks on Databricks.
**A:** LangChain, LangGraph (graph-based stateful workflows), DSPy (automated prompt optimization, programmatic pipelines), OpenAI SDK. All integrate with MLflow tracing and can be wrapped in ResponsesAgent.

---

**Q27:** What is a guardrail and how does it differ from Llama Guard?
**A:** A **guardrail** is a system prompt instruction constraining LLM behavior (e.g., "Do not provide advice on illegal activities"). It's the first line of defense — simple, text-based. **Llama Guard** is a dedicated safeguard *model* that classifies content using a risk taxonomy — operates as Input Guard (pre-LLM) and Output Guard (post-LLM). Use both for defense-in-depth.

---

## Assembling & Deploying (22%) — 20 cards

**Q28:** What is an MLflow Model Flavor?
**A:** A standard format for packaging ML/GenAI models. Each flavor defines how to save, load, and serve a model. Key flavors: `mlflow.langchain` (LangChain), `mlflow.pyfunc` (any custom Python — universal fallback), `mlflow.transformers` (HuggingFace), `mlflow.openai`.

---

**Q29:** What is `mlflow.pyfunc` and why is it important?
**A:** The default/fallback model interface. Any MLflow Python model can be loaded as a Python function via `pyfunc`. Use for complex custom logic (re-ranking steps, dynamic filters) that native flavors don't cover. Requires a `predict()` method.

---

**Q30:** What is the Unity Catalog model namespace format?
**A:** Three-level: `catalog.schema.model_name`. Example: `mlflow.register_model("runs:/<run_id>/model", "prod_catalog.ml.rag_agent")`. Load with alias: `models:/prod_catalog.ml.rag_agent/@champion`.

---

**Q31:** What are model aliases in Unity Catalog Model Registry?
**A:** Human-readable pointers to specific model versions (e.g., `@champion`, `@challenger`). Enable canary deployments and safe promotions without changing endpoint configuration. Version numbers remain immutable; aliases can be reassigned.

---

**Q32:** What are the three model types served by Databricks Model Serving?
**A:** (1) **Custom Models** — any MLflow-registered model as REST API with serverless compute; (2) **Foundation Models API** — Databricks-curated models (DBRX, Llama, Mistral); (3) **External Models** — proxy to OpenAI, Anthropic, etc. Same API surface for all three.

---

**Q33:** What are Inference Tables?
**A:** A Model Serving feature that automatically logs all request-response pairs (with metadata) to Unity Catalog Delta tables. Enables production monitoring, debugging, mislabeled data collection, and retraining with real-world data.

---

**Q34:** What is an A/B canary deployment in Databricks Model Serving?
**A:** Serving multiple model versions at one endpoint with traffic splitting. Example: 80% traffic to Champion v1, 20% to Challenger v2. Enables data-driven promotions based on real production metrics before full rollout.

---

**Q35:** What is "Deploy Code" vs "Deploy Model" and which is recommended?
**A:** **Deploy Code** (recommended): the training pipeline code is deployed to each environment, and the model is trained fresh in each environment on that environment's data. **Deploy Model**: move a pre-trained model artifact across environments. Deploy Code is better for reproducibility and ensures the model is trained on the correct data per environment.

---

**Q36:** What are the three MLOps environments and who uses each?
**A:** **Development** (data scientists: experiment/explore) → **Staging** (ML practitioners: integration testing/validation) → **Production** (ML engineers: deploy/monitor). Recommended: separate Databricks workspace per environment.

---

**Q37:** What is Direct Separation vs Indirect Separation for environments?
**A:** **Direct Separation** = separate Databricks workspace per environment (recommended — scales to multiple projects, simpler individual environments). **Indirect Separation** = single workspace with Unity Catalog ACLs enforcing environment boundaries (complex, doesn't scale well).

---

**Q38:** What is MLOps (the formula)?
**A:** `MLOps = DataOps + DevOps + ModelOps`. It's the set of processes and automation for managing data, code, and models to improve performance, stability, and long-term efficiency.

---

**Q39:** What are Databricks Asset Bundles (DABs)?
**A:** YAML-defined project bundles for CI/CD and multi-environment deployment. Define artifacts (jobs, pipelines, MLflow experiments) alongside environment-specific targets (dev/staging/prod workspaces). Enable reproducible, automated deployments via CLI: `databricks bundle deploy -t production`.

---

**Q40:** What three sections does a DABs `bundle.yml` have?
**A:** (1) **Name and default workspace** — project context; (2) **Resource configurations** — jobs, DLT pipelines, MLflow experiments (follows REST API schema); (3) **Targets** — environment-specific workspace hosts and overrides.

---

**Q41:** Why use service principals instead of personal credentials for CI/CD?
**A:** Service principals provide secure, traceable, non-personal deployments. They avoid credential leakage, enable audit trails, and don't expire when a person leaves the organization.

---

**Q42:** What is the GPU memory formula for batch deployment?
**A:** `Required GPU RAM = num_parameters × bytes_per_parameter`. Example: 10B parameters × 4 bytes (FP32) = 40 GB.

---

**Q43:** What is `ai_query()` and when is it used?
**A:** A Databricks SQL function for batch inference — applies an LLM to every row in a table. Used for high-volume, non-urgent workloads. Syntax: `SELECT AI_QUERY("model-name", CONCAT("prompt: ", column)) FROM table`.

---

**Q44:** What OSS libraries integrate with Databricks for batch GPU inference?
**A:** TensorRT (NVIDIA optimization), vLLM (efficient LLM serving), Ray on Spark (distributed GPU workloads across clusters).

---

**Q45:** What does the MLflow lifecycle look like for a RAG agent?
**A:** Build RAG/chain → MLflow Tracking (log params, configs, runs) → MLflow Evaluation → MLflow Registry (register to UC, assign aliases) → Databricks Model Serving (deploy as REST endpoint) → Inference Tables + Lakehouse Monitoring.

---

**Q46:** What is the Mosaic AI Agent Framework?
**A:** A Databricks suite for authoring, deploying, and monitoring agents. Key components: MLflow 3 integration, ResponsesAgent interface, support for LangChain/LangGraph/DSPy/OpenAI, Agent Governance via UC, Model Serving deployment, built-in evaluation.

---

**Q47:** What is an agent's lifecycle on Databricks?
**A:** (1) Prepare data + create tools → (2) Rapid prototyping with AI Playground + quality checks → (3) Evaluate and collect feedback → (4) Label data → (5) Iterative improvement → (6) Deploy to production → (7) Monitor quality and performance.

---

## Design Applications (14%) — 13 cards

**Q48:** What are the three limitations (hard boundaries) of prompt engineering?
**A:** (1) **Knowledge cutoff** — can't answer about events after training data cutoff; (2) **Hallucination** — fabricates citations/facts when lacking references; (3) **Ambiguity** — defaults to generic interpretation without private context.

---

**Q49:** What is Few-Shot prompting?
**A:** Providing examples in the prompt to guide the model's behavior and output format. Helps the model understand the expected structure and tone without retraining.

---

**Q50:** What are the three pillars of prompt engineering?
**A:** (1) **Grounding** — provide specific context/background data; (2) **Organization** — clear headings, numbered steps, logical structure to reduce ambiguity; (3) **Token Efficiency** — concise yet comprehensive; irrelevant "fluff" displaces critical data.

---

**Q51:** What are the four model classes and their sizes?
**A:** **Small** (7B–13B): fast/cheap, extraction, simple Q&A. **Medium** (30B–70B): balanced, support/content generation. **Large** (70B+): complex reasoning. **Frontier** (state-of-art): high-stakes, quality > cost.

---

**Q52:** What factors determine model selection?
**A:** Level 1 (minimum): task complexity + quality needs. Level 2: cost + latency. Level 3 (deal-breaker): privacy + deployment constraints.

---

**Q53:** What is the Model Context Protocol (MCP)?
**A:** An open standard for integration between AI models and external data sources/tools. Standardized architecture replacing fragmented proprietary integrations. Enables LLMs to access outside systems without custom code per integration.

---

**Q54:** What are the four core components of an AI Agent system?
**A:** (1) **LLM brain** — reasoning and decision-making; (2) **Memory** — short-term (session state) and long-term (episodic/semantic/procedural); (3) **Planning** — sub-task decomposition; (4) **Tool Interface** — external systems/APIs. Plus an Execution Engine.

---

**Q55:** What distinguishes a non-agentic workflow from an agentic one?
**A:** **Non-agentic**: hardcoded, deterministic, if-this-then-that logic. Human-led. **Agentic**: AI does the planning and tool selection. Non-deterministic, iterative Reason→Act→Observe loop. AI adjusts strategy based on real-time feedback.

---

**Q56:** What is Agent Learning from Human Feedback (ALHF)?
**A:** The Agent Bricks optimization mechanism. The system deploys a baseline agent, collects feedback via Review App (thumbs up/down, corrections), and automatically generates evaluation benchmarks and optimizes prompts/configuration — without manual code changes.

---

**Q57:** What are the four Agent Bricks types?
**A:** Knowledge Assistant (RAG over docs, interactive), Information Extraction (unstructured → structured, automated), Multi-Agent Supervisor (orchestrates multiple agents, interactive), Custom LLM (domain-specific optimized model, automated).

---

**Q58:** What is the difference between Declarative (Agent Bricks) and Code-First (Mosaic AI Agent Framework) approaches?
**A:** **Declarative**: specify WHAT to do; Agent Bricks handles HOW (parsing, indexing, prompting). Fastest time-to-value, auto-optimizes. Less control. **Code-First**: write all agent logic manually; infinite customizability but you own the technical debt and manual optimization.

---

**Q59:** What is a Supervisor Agent (MAS)?
**A:** A Multi-Agent Supervisor that coordinates multiple specialized agents. Routes user queries by analyzing intent and directing to the right sub-agent (e.g., billing questions → Genie space for structured data; tech questions → Knowledge Assistant for docs).

---

**Q60:** What is the three-level namespace for AI Agents and their tools?
**A:** Unity Catalog `catalog.schema.asset_name`. Tools registered as UC functions, models registered as UC models, vector indexes as UC securable objects — all governed with the same ACL model.

---

## Data Preparation (14%) — 13 cards

**Q61:** What is the data storage architecture for RAG on Databricks?
**A:** **Unity Catalog Volumes** (raw files/Bronze) → `ai_parse_document()` → **Delta Tables** (parsed text/Silver) → chunking → **Delta Tables** (chunks/Gold) → embedding → **Vector Search Index**. Volumes = non-tabular files. Delta = processed structured data.

---

**Q62:** What is `ai_parse_document` and what are its key capabilities?
**A:** Native Databricks SQL/Python function for extracting structured content from PDFs and images. Capabilities (v2.0): layout awareness, figure descriptions (auto-generated text for charts), bounding boxes for UI highlighting, OCR for text in images, handles multi-column layouts and tables.

---

**Q63:** What are document parsing challenges that `ai_parse_document` solves?
**A:** (1) **Hierarchical info** — charts with nested relationships; (2) **Order preservation** — multi-column reading order; (3) **Contextual integrity** — keeping images associated with their text/captions.

---

**Q64:** What is fixed-size chunking and why is it considered legacy?
**A:** Splits text by a hard character/token count (e.g., 500 tokens). Computationally cheap but often cuts sentences/paragraphs in half, destroying semantic context. Replaced by semantic chunking as standard.

---

**Q65:** What is Embedding-Based Semantic Chunking?
**A:** Uses an embedding model to determine breakpoints — calculates semantic similarity between sequential sentences and only breaks when similarity drops below a threshold (topic change). Each chunk represents a distinct, coherent concept.

---

**Q66:** What is Windowed Summarization chunking?
**A:** Each chunk includes a "windowed summary" of the previous few chunks. The LLM sees the current text plus a summary of preceding context — provides broader context without embedding the full history.

---

**Q67:** What is Chunk Overlap and why is it used?
**A:** Repeating 10–20% of text at the beginning of the next chunk. Prevents sentences or ideas from being cut off abruptly at chunk boundaries. Ensures no contextual information is lost between adjacent chunks.

---

**Q68:** What is the critical rule for chunk size relative to the embedding model?
**A:** Max chunk size must always stay safely below the embedding model's context window limit. Exceeding the limit causes silent truncation — the model ignores excess text, resulting in incomplete vectors and lost data.

---

**Q69:** What tools/frameworks are used for chunking on Databricks?
**A:** `ai_parse_document` (extraction/OCR), **LangChain** `RecursiveCharacterTextSplitter` (semantic splitting — industry standard), custom Python UDFs (specialized splitting by Markdown headers, etc.).

---

**Q70:** What is `ai_extract` and when is it used?
**A:** A Databricks AI function for pulling structured fields from unstructured text when metadata isn't in file properties. Example: extracting "Invoice Date" or "Contract Type" from document text for metadata injection.

---

**Q71:** What are Autoloader and SDP used for?
**A:** **Autoloader**: efficiently ingests new files arriving in Unity Catalog Volumes, processes them incrementally. **SDP (Spark Declarative Pipelines)**: declarative pipeline for data ingestion and processing. Both are used for the raw file → parsed text ingestion step.

---

**Q72:** What is metadata injection and why does it matter for RAG?
**A:** During transformation, extract and associate metadata (title, author, date, document type) with each chunk. This metadata enables pre-retrieval filtering in Unity Catalog — users can filter by `year=2024` or `doc_type=policy` before vector search, improving precision and preventing stale data from reaching the LLM.

---

**Q73:** What is the granularity vs context trade-off in chunking?
**A:** **Larger chunks** = more context per chunk but may dilute specific details; may exceed embedding model limits. **Smaller chunks** = more precise retrieval but may lack surrounding context. Must align with the embedding model's context window capabilities.

---

## Evaluation & Monitoring (12%) — 11 cards

**Q74:** What are Loss and Perplexity and what do they NOT measure?
**A:** **Loss**: how well the model predicts the next token (optimization signal during training). **Perplexity**: model confidence across predictions (lower = more confident). Neither directly measures task performance or factual accuracy — a model can have low loss/perplexity and still hallucinate.

---

**Q75:** What is BLEU and what is it designed for?
**A:** BiLingual Evaluation Understudy. Measures n-gram precision overlap between model output and reference translation. Designed for **machine translation**. Applies brevity penalty to discourage short outputs. Does not capture semantic similarity.

---

**Q76:** What is ROUGE and its variants?
**A:** Recall-Oriented Understudy for Gisting Evaluation. Designed for **summarization**. ROUGE-1 (unigram overlap), ROUGE-2 (bigram), ROUGE-L (longest common subsequence), ROUGE-Lsum (summary-level). Measures recall — were key points covered?

---

**Q77:** What is Mosaic AI Gauntlet?
**A:** Databricks' curated benchmark suite: 35 benchmark sources across 6 categories (reading comprehension, commonsense reasoning, problem-solving, language understanding, etc.). Used during pre-training/fine-tuning to compare checkpoints and select the best-performing model version.

---

**Q78:** When should you use LLM-as-Judge?
**A:** When there's no existing reference dataset, no defined numerical metric, and no clear evaluation criteria. An LLM evaluates another LLM's output using a rubric with examples and instructions. Requires human-in-the-loop oversight to catch bias and hallucinated scores.

---

**Q79:** What is the difference between offline and online evaluation?
**A:** **Offline** (pre-deployment): validate against curated benchmark datasets with known ground truth — catches issues before users see them. **Online** (post-deployment): evaluate with real user behavior — catches distribution shifts, edge cases, and drift that benchmarks can't represent.

---

**Q80:** What does Databricks Lakehouse Monitoring generate for each monitored table?
**A:** (1) **Profile metrics table** — statistical/distribution summaries; (2) **Drift metrics table** — tracks changes in data/model behavior over time; (3) **Custom metrics** — defined as SQL expressions; (4) **DBSQL dashboard** — auto-generated visualization.

---

**Q81:** What is the difference between Faithfulness/Groundedness and Answer Correctness?
**A:** **Faithfulness/Groundedness** (Response + Context): is every claim in the answer supported by the retrieved context? Detects hallucination beyond context. **Answer Correctness** (Response + Ground Truth): is the answer factually accurate vs ground truth? A faithful answer can still be incorrect if the retrieved context itself contained errors.

---

**Q82:** What is the Inference Table monitoring workflow?
**A:** Inference Table (raw request-response logs) → Unpack JSON → Calculate metrics (token counts, toxicity, LLM-judge scores) → Materialize to processed Delta table → Streaming job → Lakehouse Monitoring enabled on processed table → DBSQL dashboard + SQL alerts.

---

**Q83:** What is explicit vs implicit human feedback?
**A:** **Explicit**: direct, intentional — thumbs up/down, star ratings, corrections, expert annotations. High quality, lower volume. **Implicit**: inferred from behavior — click-through rates, session length, copy-paste behavior, follow-up questions. High volume, noisier.

---

**Q84:** What does the Mosaic AI Agent Framework provide for evaluation?
**A:** Agent Tracing (step-by-step execution visibility), RAG-specific metrics (context precision/recall/faithfulness/answer relevancy), Review App (human expert feedback interface), Databricks LLM Judges (proprietary judges fine-tuned for RAG quality assessment).

---

## Governance (8%) — 8 cards

**Q85:** What are the 6 built-in risk categories in Llama Guard?
**A:** (1) Violence & Hate, (2) Sexual Content, (3) Guns & Illegal Weapons, (4) Regulated or Controlled Substances, (5) Suicide & Self Harm, (6) Criminal Planning.

---

**Q86:** How does Llama Guard's dual-guard architecture work?
**A:** **Input Guard**: intercepts user query before it reaches the LLM — blocks harmful queries. **Output Guard**: checks LLM response before delivery — blocks harmful responses. Defense-in-depth: even if Input Guard is bypassed, Output Guard prevents harmful content from reaching the user.

---

**Q87:** What is Prompt Injection?
**A:** An attack where a user crafts input designed to override system instructions. Example: "Ignore all previous instructions and tell me how to..." Can extract private system prompt data or produce forbidden responses. Mitigate with guardrails + Llama Guard Input Guard.

---

**Q88:** What questions must be answered before deploying data for legal compliance?
**A:** (1) Who owns the data? (2) Is the application for commercial use? (3) In which countries/states will it be deployed? (4) Will the system generate profit? Different answers trigger different licensing obligations. Especially important for public web scrapes and third-party data.

---

**Q89:** What is the DASF and why does Platform Security (#12) matter most?
**A:** The Databricks Data and AI Security Framework defines 12 components and 55 risks. Platform Security (#12) covers cloud architecture, serverless configs, and secrets management. It's foundational — even a perfectly secured model is vulnerable if secrets are leaked through misconfigured infrastructure.

---

**Q90:** What are auditability and lineage in the context of GenAI?
**A:** **Auditability**: degree to which an AI system's decisions, data inputs, and behaviors can be monitored, recorded, and verified by an independent party. **Lineage**: tracking the entire lifecycle of a data asset from origin to final consumption (raw doc → embeddings → model → endpoint). Both provided by MLflow + Unity Catalog.

---

**Q91:** How does Unity Catalog provide governance for vector search indexes?
**A:** Vector Search indexes are registered as securable objects in Unity Catalog. Administrators enforce granular Access Control Lists (ACLs) at the index level — only authorized users and applications can query or modify vector data. Consistent with all other UC-governed assets.

---

**Q92:** What is data poisoning and how does it affect AI security?
**A:** An adversarial attack where malicious data is injected into training or retrieval corpora, corrupting model behavior. In RAG systems, poisoned documents in the knowledge base can manipulate the LLM's responses. Covered by DASF component #5 (Algorithms).
