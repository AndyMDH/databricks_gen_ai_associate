# GenAI Application Evaluation and Governance

## Lesson 1: Why Evaluating GenAI Applications

### Learning Objectives

- Understand why evaluation is essential for GenAI systems
- Identify the components of a RAG-based AI system and what needs to be evaluated in each
- Recognize key issues: data legality, prompt injection, and bias
- Explain why classical ML evaluation approaches fall short for GenAI
- Describe a systematic evaluation strategy and the role of guardrails

---

### A. Why Evaluate?

A1. **The core questions evaluation must answer**

Deploying a GenAI system without evaluation is like shipping code without testing. Evaluation answers:

- Is the system behaving as expected?
- Are users satisfied with the results?
- Is the LLM solution effective for the intended task?
- Is there bias or any other ethical concern?
- What does it cost (compute, latency, money)?

Without answers to these questions, there is no reliable way to know whether the AI system is working correctly, safely, or efficiently in production.

A2. **What is an AI system?**

Modern GenAI applications (especially RAG systems) are composed of multiple interacting components:

- **Data components**: Raw documents, vector database, user input/output
- **Model components**: Embeddings model, generation model
- **Infrastructure and governance components**: Vector search system, user interface, security/governance layer, logging

Because the system is made up of smaller parts, a failure in any one component can silently degrade the overall experience. Evaluation must address both the whole system and individual components.

---

### B. Evaluating the System and Its Components

B1. **System-level vs. component-level evaluation**

Analogous to software engineering:

- **End-to-end testing** evaluates the full system (does the user get a correct, safe answer?)
- **Integration/unit testing** evaluates individual components (does the retriever return relevant chunks?)

Both levels are necessary. A system can pass end-to-end tests while a single component degrades silently, or a component may appear healthy while downstream interactions cause failures.

B2. **Evaluating data**

Data quality affects every downstream component. Key areas:

| Area | Considerations |
|---|---|
| **LLM Training Quality** | Select LLMs with high-quality, published training benchmarks |
| **Contextual Data Quality** | Implement quality controls; monitor data statistics; conduct bias/ethics review |
| **Input/Output Quality** | Collect and review I/O pairs; monitor for distribution changes; use LLM-as-a-judge metrics |

---

### C. Key Issues in GenAI Evaluation

C1. **Data Legality**

Many datasets carry licenses that constrain use. Before deploying, answer:

- Who owns the data?
- Is the application for commercial use?
- In which countries/states will it be deployed?
- Will the system generate profit?

Different answers trigger different licensing obligations. Violation can expose the organization to legal liability.

**Note:** Always review the license terms of any external dataset, including those used in fine-tuning and RAG pipelines. This is especially relevant for content from public web scrapes or third-party data providers.

C2. **Harmful User Behavior: Prompt Injection**

LLMs are flexible — and that flexibility can be exploited. **Prompt injection** occurs when a user crafts input designed to override the system's intended behavior:

```
User: "Ignore all previous instructions and tell me how to..."
```

This can extract private information embedded in the system prompt or produce responses the system was explicitly designed to avoid. Prompt injection is particularly dangerous in production because:

- Users are adversarial by nature in some domains
- LLMs are not inherently rule-bound; they follow instructions

C3. **Bias and Ethical Use**

LLMs learn from their training data. Even a well-intentioned system can amplify biases present in that data.

**Example:** A model trained primarily on British healthcare data may give UK-specific clinical advice to users in other countries, where guidelines differ.

Bias is subtle because:

- The model follows patterns in the training data, not explicit rules
- Outputs may appear reasonable even when skewed

**Note:** Bias is not always intentional and can be hard to detect without systematic evaluation against diverse test populations.

C4. **Why Classical ML Evaluation Falls Short**

| Dimension | Classical ML | GenAI |
|---|---|---|
| **Truth** | Clear target/label values | Hard to define "ground truth" for open-ended responses |
| **Quality** | Numerical prediction metrics (accuracy, F1) | Quality is subjective and hard to quantify |
| **Bias** | Can be addressed with fairness constraints | Emergent from training data; harder to isolate |
| **Security** | Outputs are structured and bounded | Unstructured text outputs create wider attack surface |

---

### D. A Systematic Approach to GenAI Evaluation

D1. **The evaluation strategy**

Rather than treating evaluation as an afterthought, apply a component-based, layered approach:

1. **Evaluate the system and its components** — both end-to-end and individual parts
2. **Mitigate data risks** — data licensing, prompt safety, guardrails
3. **Evaluate LLM quality** — metrics specific to language model outputs (covered in Lesson 3)
4. **Secure the system** — address AI-specific security risks (covered in Lesson 2)
5. **Evaluate system quality** — end-to-end performance metrics (covered in Lesson 4)

D2. **Prompt Safety and Guardrails**

**Guardrails** are instructions added to the system prompt to constrain what the LLM will and won't do. They are the first line of defense against prompt injection and harmful outputs.

Simple guardrail example:

```
System: "You are a helpful assistant. Do not provide advice on illegal activities.
If a user asks for anything illegal or harmful, politely decline and explain why."
```

Guardrails range from simple refusal instructions to complex rule sets. They should be:

- Specific enough to prevent the targeted behavior
- Broad enough not to block legitimate use cases
- Regularly tested against adversarial inputs

**Tip:** Guardrails are not foolproof. Combine them with Llama Guard (covered in Lesson 2) and ongoing monitoring for a defense-in-depth approach.

---

### Key Takeaways — Lesson 1

- GenAI systems are compound systems; evaluation must cover both individual components and the system as a whole.
- Data legality, prompt injection, and bias are the three major risks unique to GenAI evaluation.
- Classical ML evaluation techniques do not transfer directly to GenAI because truth, quality, bias, and security are fundamentally harder to measure.
- A systematic approach moves from data → LLM quality → security → system quality.
- Guardrails are prompt-level constraints that mitigate harmful or off-target behavior.

---

## Lesson 2: AI System Security

### Learning Objectives

- Identify the primary security risks specific to AI systems
- Explain the Databricks Data and AI Security Framework (DASF) and its 12 components
- Describe the 6 DASF components most relevant to GenAI engineers
- Understand how Unity Catalog and Mosaic AI address AI security
- Explain how Llama Guard functions as an input and output safeguard

---

### A. Why AI Security Is Different

A1. **AI-specific security risks**

Traditional application security focuses on access control, encryption, and network perimeters. AI systems introduce additional risks:

- **Data access, governance, and lineage** — who can access training data, vectors, and model outputs?
- **Model tracking and audit** — can you trace what data a model was trained on and how it performed?
- **Adversarial attacks** — poisoning (corrupting training data) and injection (manipulating inputs)
- **Information exposure** — the model or context window may leak sensitive data embedded in prompts
- **Quality drift** — degrading model performance can be a security signal, not just a quality issue

**Survey finding:** When organizations were surveyed about their top concerns for AI systems, security ranked #1 at **46%**, ahead of cost and reliability (both at 41%).

A2. **Why AI security is challenging**

AI security sits at an uncomfortable intersection:

- **Data scientists** understand the models but haven't traditionally done security
- **Security teams** know security but are new to AI architectures
- **ML engineers** are used to offline batch workflows, not real-time adversarial environments

The result: no single team has complete ownership. This gap leaves AI systems under-secured.

---

### B. Simplifying AI Security with a Component-Based Approach

B1. **Secure the components, secure the system**

The same principle that applies to evaluation applies to security: break the system into components and secure each one. For a RAG architecture, that means securing:

- Input query (user-provided text)
- Embedding model
- Document data (training corpus and retrieval corpus)
- Vector database
- Generation model
- Output query (the generated response)
- Generated data and metadata (logs, traces)

B2. **The Databricks Data and AI Security Framework (DASF)**

Databricks developed the **DASF** through industry workshops, identifying **12 foundational components** of a data-centric AI/ML system and **55 associated risks** across those components.

The 12 DASF components:

| # | Component |
|---|---|
| 1 | Raw Data |
| 2 | Data Prep |
| 3 | Datasets |
| 4 | Data Catalog and Governance |
| 5 | Algorithms |
| 6 | Evaluation |
| 7 | Models |
| 8 | Model Management |
| 9 | Model Serving and Inference Request |
| 10 | Model Serving and Inference Response |
| 11 | Operations |
| 12 | Platform Security |

B3. **The 6 components GenAI engineers must focus on**

| DASF # | Component | Key Responsibilities |
|---|---|---|
| **4** | Data Catalog and Governance | Access control throughout lifecycle; data quality and reliability |
| **5** | Algorithms | Addressing data poisoning risks; online adversarial monitoring |
| **6** | Evaluation | Evaluating system components and assets; flagging security failures via performance degradation |
| **8** | Model Management | Tracking data, models, and lineage; encryption and security controls |
| **11** | Operations | MLOps quality in production; monitoring and incident response; security testing and compliance |
| **12** | Platform Security | Cloud architecture security; serverless configurations; secret management; CLI and asset bundling |

**Key Point:** A system is only as secure as its weakest component. Platform security (#12) is foundational — even a perfectly secured model is vulnerable if secrets are leaked through misconfigured infrastructure.

---

### C. Databricks Security Architecture

C1. **Databricks as a security platform**

The Databricks Lakehouse Platform is designed with AI security built in at each layer:

- **Unity Catalog** — centralized governance for data and AI assets
- **Delta Live Tables / Delta Lake** — data quality enforcement
- **Lakehouse Monitoring** — ongoing quality and drift detection
- **MLflow** — model lineage, versioning, and audit trail
- **Cloud base** — inherits cloud provider security primitives (IAM, VPC, encryption)

C2. **Unity Catalog for AI governance**

**Unity Catalog** provides centralized governance across all data and AI assets:

- Govern and secure models, datasets, and vector indexes in a single place
- Ensure compliance by managing GenAI models with access policies
- Track **end-to-end lineage** from raw data through to generated outputs
- Govern **Vector Search indexes** — who can query which index?
- Enable **cross-workspace asset usage** with consistent permissions

C3. **Mosaic AI for production safety**

**Mosaic AI** adds safety controls at the inference layer:

- **Scalable, secure model serving** — serving endpoints with access controls
- **Safety Filter** — built-in content moderation
- **Llama Guard** — configurable safeguard model (detailed below)
- **mlflow.evaluate** — standardized evaluation integrated with the serving workflow
- **Performance monitoring** — continuous tracking of model quality in production

---

### D. Llama Guard

D1. **What Llama Guard is**

**Llama Guard** is a safeguard model that classifies content in LLM conversations to detect and mitigate safety risks. It operates on two inputs:

1. **A taxonomy of risks** — categories of harmful content to detect
2. **A guideline** — what action to take when a risk is detected

Built-in risk taxonomy categories:

- Violence & Hate
- Sexual Content
- Guns & Illegal Weapons
- Regulated or Controlled Substances
- Suicide & Self Harm
- Criminal Planning

D2. **Input Guard and Output Guard**

Llama Guard can be deployed at two points in the conversation flow:

```
User Query → [Input Guard] → LLM → [Output Guard] → Response
```

- **Input Guard**: Intercepts the user's prompt *before* it reaches the LLM. If the query violates the safety taxonomy, it is blocked and never processed.
- **Output Guard**: Checks the LLM's response *before* delivery to the user. If the response is harmful, it is blocked or replaced.

This dual-guard architecture provides defense-in-depth: even if a malicious query bypasses the input guard, the output guard prevents harmful content from reaching the user.

**Tip:** Llama Guard is available as part of the Mosaic AI safety tooling in Databricks. Pair it with prompt-level guardrails from Lesson 1 for layered protection.

---

### Key Takeaways — Lesson 2

- AI security extends beyond traditional application security to include data poisoning, prompt injection, model lineage, and quality drift.
- The **DASF** provides a structured framework of 12 components and 55 risks; GenAI engineers should focus on components 4, 5, 6, 8, 11, and 12.
- **Unity Catalog** centralizes governance for all data and AI assets, including vector indexes and model lineage.
- **Mosaic AI** provides scalable serving with built-in safety tooling (Safety Filter, Llama Guard).
- **Llama Guard** acts as both an Input Guard (pre-LLM) and Output Guard (post-LLM), using a configurable risk taxonomy to block unsafe content in real time.

---

## Lesson 3: Evaluation Techniques

### Learning Objectives

- Explain how LLM evaluation differs from classical ML evaluation
- Describe the base foundation model metrics: Loss, Perplexity, and Toxicity
- Apply task-specific metrics (BLEU, ROUGE) and understand when each is appropriate
- Understand benchmarking approaches including Mosaic AI Gauntlet
- Implement LLM-as-a-Judge evaluation and know its limitations
- Use MLflow's LLM evaluation capabilities for batch and interactive evaluation

---

### A. LLMs vs. Classical ML Evaluation

A1. **Different scopes, different metrics**

| Dimension | Classical ML | LLMs |
|---|---|---|
| **What you evaluate** | The entire system | The LLM as one component within a larger system |
| **How you measure** | Cost vs. value; prediction accuracy | General metrics + task-specific metrics; human judgment |
| **Data requirements** | Relatively modest, labeled datasets | Massive data; compute-intensive evaluation |
| **Interpretability** | Interpretable model coefficients | Black-box; limited interpretability |
| **Evaluation metrics** | Accuracy, precision, recall, F1, MSE | BLEU, ROUGE, perplexity, LLM-as-judge |

The key insight: you must evaluate both **the LLM component** (using language-specific metrics) and **the system as a whole** (using end-to-end metrics). These are not the same evaluation.

---

### B. Base Foundation Model Metrics

B1. **Loss**

**Question it answers:** How well does the model predict the next token?

Loss measures the difference between the model's probability distribution over the vocabulary and the actual next token. Lower loss = better next-token prediction.

- **Training/validation loss** is tracked during pre-training as the primary optimization signal
- **Limitation:** A model can achieve low loss (confident predictions) while still hallucinating on specific facts. Minimizing next-token prediction loss is not the same as maximizing task accuracy.

B2. **Perplexity**

**Question it answers:** Is the model surprised that it was correct?

Perplexity is the exponentiated average negative log-likelihood per token — a measure of the model's confidence across its predictions.

- **Low perplexity** = the model assigns high probability to the actual token sequence = high confidence
- **High perplexity** = the model is uncertain, spreading probability across many tokens

**Limitation:** Low perplexity reflects fluent text generation but does not guarantee meaningful or accurate answers for a specific downstream task. A model can be fluent and wrong.

B3. **Toxicity**

**Question it answers:** How harmful is the model's output?

Toxicity scores are computed using a **pre-trained hate speech classification model** applied to the model's outputs. The classifier returns a score indicating the presence of harmful, offensive, or inappropriate language.

- **Low toxicity score** = low harm
- Used in production to ensure that a model doesn't generate unsafe responses even when inputs appear benign

---

### C. Task-Specific Evaluation Metrics

C1. **Why base metrics aren't enough**

Base metrics (loss, perplexity, toxicity) evaluate general language quality. For production applications, you need **task-specific metrics** that measure performance on the actual task:

MLflow provides built-in evaluators for common tasks:

```python
import mlflow

with mlflow.start_run():
    results = mlflow.evaluate(
        model,
        data=eval_data,
        targets="ground_truth",
        model_type="question-answering",  # or: regression, classification, text-summarization
        evaluators="default"
    )
```

C2. **BLEU — BiLingual Evaluation Understudy**

**Designed for:** Machine translation

**How it works:** Compares n-gram overlap between the model's output and one or more reference translations.

- Computes unigram, bigram, and trigram precision
- Higher BLEU = closer match to reference
- Applies a brevity penalty to discourage very short outputs

**When to use:** Any task where the output should closely match a reference string (translation, constrained generation).

**Limitation:** BLEU rewards exact n-gram matches and can penalize valid paraphrases.

C3. **ROUGE — Recall-Oriented Understudy for Gisting Evaluation**

**Designed for:** Summarization

**Variants:**

| Variant | What it measures |
|---|---|
| ROUGE-1 | Unigram (word) overlap |
| ROUGE-2 | Bigram overlap |
| ROUGE-L | Longest common subsequence |
| ROUGE-Lsum | Summary-level longest common subsequence |

**When to use:** Summarization tasks where recall matters — you want to ensure the summary covers the key points of the source.

**Limitation:** Like BLEU, ROUGE depends on reference data quality and doesn't capture semantic similarity.

C4. **What BLEU and ROUGE have in common**

Both metrics:

1. Are task-specific (not general)
2. Are applied to LLM output
3. Consider n-gram sequences rather than individual words
4. Compare output against reference datasets

**Key Point:** The quality of these metrics is only as good as the quality of the reference data. Investing in high-quality, curated reference datasets is essential.

---

### D. Benchmarking

D1. **What benchmarking is**

**Benchmarking** is comparing model performance against standardized evaluation datasets. It enables:

- Comparing models from different providers
- Tracking progress during fine-tuning
- Identifying strengths and weaknesses across task types

D2. **Types of benchmark datasets**

- **Generic/public datasets**: Large standard datasets (e.g., Stanford Q&A, HellaSwag, MMLU). Useful for comparing foundation models.
- **Domain-specific datasets**: Datasets curated for your specific use case (e.g., a Databricks documentation translation dataset with English paired with Spanish/Portuguese references). These ensure evaluation is relevant to real-world performance.

**Note:** Public benchmarks tell you how a model performs on average. Domain-specific benchmarks tell you how it performs for *your* users. Both are necessary.

D3. **Mosaic AI Gauntlet**

**Mosaic AI Gauntlet** is Databricks' curated evaluation benchmark suite:

- **35 benchmark sources** combined into a single evaluation
- Organized into **6 broad categories**:
  - Reading comprehension
    - Commonsense reasoning
  - Problem-solving
  - Language understanding
  - (and additional categories)
- Provides a **holistic view** of model capabilities with detailed assessment of strengths and weaknesses across critical task types

Gauntlet is used during pre-training and fine-tuning to compare model checkpoints and select the best-performing version.

---

### E. LLM-as-a-Judge

E1. **The chicken-and-egg problem**

Many real-world tasks don't have:

- An existing reference dataset
- A defined API or numerical metric
- Clear evaluation criteria for edge cases

This creates a challenge: how do you evaluate a model's output when there's nothing to compare it to?

E2. **Solution: LLM-as-a-Judge**

Use an LLM to evaluate the output of another LLM (or the same LLM). The judge LLM receives:

```
Question: {user_question}
Answer: {model_answer}

Rate the quality of this answer on a scale of 0-10.
Consider: accuracy, helpfulness, clarity, and safety.
```

For the judge to work well, it needs:

- **Clear examples** with human-provided scores (few-shot examples)
- **Specific instructions** defining what constitutes a "good" answer
- **A component-based rubric** or metric-specific evaluation scale

E3. **Limitations and how to address them**

| Limitation | Risk | Mitigation |
|---|---|---|
| Lack of contextual awareness | Judge misses domain-specific nuance | Provide domain context in the judge prompt |
| Hallucinated metrics | Judge invents scores not grounded in the answer | Use structured output with constrained scoring |
| Bias | Judge inherits biases from its training | Use Human-in-the-loop review |

**Human-in-the-loop:** Have human reviewers audit a sample of LLM-judge evaluations to catch systematic errors, improve accuracy, and handle ambiguous cases. This hybrid approach scales the speed of LLM evaluation while maintaining human oversight.

---

### F. MLflow LLM Evaluation

F1. **What MLflow LLM Evaluation provides**

MLflow's LLM evaluation module enables:

- **Batch comparisons**: Compare foundation models against fine-tuned variants across many test questions simultaneously
- **Rapid experimentation**: Automatically evaluate unstructured text outputs without manual review
- **Cost efficiency**: LLM-as-a-judge automation reduces the need for expensive human evaluators

F2. **Two evaluation modes**

**Batch evaluation** (programmatic):

```python
import mlflow
import pandas as pd

eval_data = pd.DataFrame({
    "inputs": ["What is MLflow?", "What is Unity Catalog?"],
    "ground_truth": ["MLflow is an open-source MLOps platform...", "Unity Catalog is..."]
})

with mlflow.start_run():
    results = mlflow.evaluate(
        "endpoints:/databricks-llama-2-70b-chat",
        eval_data,
        targets="ground_truth",
        model_type="question-answering",
        evaluators="default"
    )
```

**Interactive evaluation** (UI):

- Visually compare multiple models or prompt variants side-by-side
- Iteratively test new queries during development
- No code required for ad-hoc exploration

F3. **Custom LLM-as-a-Judge metrics in MLflow**

Three-step workflow for custom metrics:

1. **Create evaluation records** — examples with human-provided scores
2. **Create a metric object** — define scoring criteria, examples, model, and aggregation method
3. **Evaluate** — run the metric against your dataset

```python
from mlflow.metrics.genai import make_genai_metric

helpfulness = make_genai_metric(
    name="helpfulness",
    definition="A helpful response addresses the user's question completely.",
    grading_prompt="Score from 1-5: 1=unhelpful, 5=very helpful",
    examples=[...],  # few-shot examples with human scores
    model="endpoints:/databricks-llama-2-70b-chat",
    aggregations=["mean", "std"]
)

with mlflow.start_run():
    results = mlflow.evaluate(
        model,
        eval_data,
        extra_metrics=[helpfulness]
    )
```

---

### Key Takeaways — Lesson 3

- LLM evaluation is not the same as classical ML evaluation: you evaluate the LLM *component* separately from the overall system.
- **Loss and Perplexity** are pre-training metrics; they don't directly measure task performance.
- **BLEU** is for translation (precision-focused n-gram comparison); **ROUGE** is for summarization (recall-focused n-gram comparison). Both require high-quality reference data.
- **Mosaic AI Gauntlet** provides a standardized 35-source benchmark suite for holistic model comparison.
- **LLM-as-a-Judge** solves the "no reference dataset" problem but requires human-in-the-loop oversight to address hallucination and bias in the judge itself.
- **MLflow** supports both batch (programmatic) and interactive (UI) LLM evaluation, with a three-step workflow for custom metrics.

---

## Lesson 4: End-to-End Application Evaluation

### Learning Objectives

- Define the cost and performance dimensions for system-level evaluation
- Apply the six RAG-specific metrics: context precision, context relevancy, context recall, faithfulness, answer relevancy, and answer correctness
- Create custom metrics aligned to business goals using MLflow
- Distinguish offline vs. online evaluation and know when each applies
- Describe explicit and implicit human feedback mechanisms
- Explain how Databricks Lakehouse Monitoring and Mosaic AI Agent Framework support continuous evaluation

---

### A. Evaluating the System as a Whole

A1. **What to measure at the system level**

Previous lessons covered individual components (embeddings, retrieval, generation) and LLM-specific metrics. End-to-end evaluation asks: *does the complete system deliver value?*

Three categories of system-level metrics:

| Category | Examples |
|---|---|
| **Cost metrics** | Compute resources consumed, inference latency, $ per query |
| **Performance metrics** | Direct value (task completion rate, accuracy), Indirect value (user retention, downstream KPIs) |
| **Custom metrics** | Business-goal-specific measures tailored to the use case |

A2. **Evaluating a RAG pipeline end-to-end**

A RAG system is a **compound AI system** — each component contributes to end-to-end performance:

```
User Query → Chunking → Embedding → Vector Search → Re-ranker → Generator → Response
```

Evaluate each component individually *and* the pipeline as a whole. Key components to evaluate:

- **Chunking**: method and chunk size impact retrieval recall
- **Embedding model**: quality of semantic representations
- **Vector store**: retrieval precision and re-ranker effectiveness
- **Generator**: faithfulness, relevance, and correctness of the final answer

---

### B. RAG Evaluation Metrics

B1. **The four metric relationships**

RAG evaluation is organized around four key relationships in the pipeline:

```
Query ←→ Context    (Is the retrieved context relevant to the query?)
Query ←→ Response   (Does the response actually answer the query?)
Context ←→ Response (Is the response grounded in the retrieved context?)
Response ←→ Ground Truth / Context ←→ Ground Truth  (Is the answer correct?)
```

B2. **Retrieval metrics**

**Context Precision**

- **What it measures:** Signal-to-noise ratio of the retrieved context
- **Based on:** Query + Context(s)
- **Question:** Are the highly-relevant chunks ranked *higher* than irrelevant ones?
- High precision: retrieved chunks specifically address the query topic
- Low precision: retrieved chunks are topically related but don't address the specific question

**Context Relevancy**

- **What it measures:** How well the retrieved context addresses the posed question
- **Based on:** Query + Context(s)
- **Note:** Does not assess factual accuracy — only relevance to the query intent
- High relevancy: context directly explains the answer to the question
- Low relevancy: context mentions related topics but doesn't address the actual query

**Context Recall**

- **What it measures:** Whether all relevant information was retrieved
- **Based on:** Ground Truth + Context(s)
- **Question:** Does the retrieved context cover *all* facts mentioned in the ground truth?
- High recall: context mentions all key entities and facts from the ground truth
- Low recall: context only partially covers what the ground truth contains

**Tip:** Precision vs. Recall trade-off applies here. High precision means retrieved chunks are highly targeted. High recall means no relevant information was missed. The right balance depends on the use case.

B3. **Generation metrics**

**Faithfulness**

- **What it measures:** Factual accuracy of the generated answer *relative to the retrieved context*
- **Based on:** Response + Context(s)
- High faithfulness: every claim in the answer is supported by the context
- Low faithfulness: the answer contradicts or goes beyond the context (hallucination indicator)

**Answer Relevancy**

- **What it measures:** Whether the response addresses the user's actual intent
- **Based on:** Response + Query
- High relevancy: response directly answers what was asked
- Low relevancy: response is topically related but doesn't answer the specific question

**Answer Correctness**

- **What it measures:** Accuracy of the generated answer compared to the ground truth
- **Based on:** Response + Ground Truth
- Encompasses both **semantic similarity** and **factual accuracy**
- High correctness: answer matches the factual content of the ground truth
- Low correctness: answer contains factual errors relative to the ground truth

**Note:** Faithfulness measures if the answer is grounded in the *context*. Answer Correctness measures if the answer is factually *accurate* against ground truth. These are different: a faithful answer can still be incorrect if the retrieved context itself contains errors.

---

### C. Custom Metrics

C1. **Why custom metrics matter**

Standard metrics (context precision, faithfulness, etc.) capture universal quality signals. But every application has business-specific requirements:

- What is the acceptable serving latency?
- What is the cost ceiling per query?
- Is the goal to increase product demand, reduce support tickets, or improve customer satisfaction?
- Are there regulatory requirements (response format, data residency)?

Custom metrics translate these requirements into measurable signals that standard metrics won't capture.

C2. **Custom metrics in MLflow**

MLflow extends its built-in evaluation to support custom LLM-as-a-judge metrics:

```python
from mlflow.metrics.genai import make_genai_metric, EvaluationExample

# Define what "professional tone" means with examples
example_high = EvaluationExample(
    input="How do I reset my password?",
    output="To reset your password, please navigate to...",
    score=5,
    justification="Response is polite, clear, and professional."
)

professional_tone = make_genai_metric(
    name="professional_tone",
    definition="Response should be courteous and professional in tone.",
    grading_prompt="Rate from 1 (unprofessional) to 5 (highly professional).",
    examples=[example_high],
    model="endpoints:/databricks-llama-2-70b-chat",
    aggregations=["mean"]
)
```

Custom metrics are especially useful for **component-level monitoring** as well — a retriever's response time or a chunker's average chunk size can be tracked as custom metrics alongside quality signals.

---

### D. Offline vs. Online Evaluation

D1. **Offline evaluation (pre-production)**

Offline evaluation occurs before deploying to users. The goal is to validate quality before it affects real users.

Steps:

1. Curate a benchmark dataset (representative queries + ground truth answers)
2. Apply task-specific evaluation metrics (BLEU, ROUGE, RAG metrics)
3. Evaluate using reference data or LLM-as-a-judge

**When to use:** Before any deployment, after significant model or pipeline changes, and as part of CI/CD for the AI system.

D2. **Online evaluation (in production)**

Online evaluation occurs after deployment, using real user behavior as the signal.

Steps:

1. Deploy the application
2. Collect real user behavior data (interactions, engagement, feedback)
3. Evaluate how well users respond to the system's outputs

**When to use:** Continuously in production to detect drift, changing user expectations, or degraded performance that offline benchmarks didn't anticipate.

**Key Point:** Offline evaluation validates against known benchmarks. Online evaluation surfaces the unknowns — edge cases and distribution shifts that synthetic benchmarks can't represent.

---

### E. Human Feedback

E1. **Why human feedback is indispensable**

Automated evaluation (LLM-as-a-judge, metrics) scales efficiently but has blind spots:

- Developers are often not domain experts
- Model outputs must be evaluated by the people who understand the use case
- Automated metrics can't fully capture user experience

Human feedback must be **collected and stored in a structured manner** to be usable for ongoing evaluation and fine-tuning.

E2. **Types of human feedback**

**Explicit feedback** — direct and intentional:

- Thumbs up/down ratings
- Star ratings
- Written comments and corrections
- Expert annotations

**Implicit feedback** — inferred from behavior:

- Click-through rates on suggested responses
- Session length and return rate
- Copy/paste behavior (a proxy for "I found this useful")
- Engagement metrics (did the user ask a follow-up or abandon the session?)

Both types provide valuable but different signals. Explicit feedback is higher-quality but lower-volume. Implicit feedback is high-volume but noisier.

---

### F. Ongoing Monitoring and Mosaic AI Agent Framework

F1. **Databricks Lakehouse Monitoring**

AI systems need continuous monitoring because both the system and the world around it change:

- **Data drift**: the distribution of user queries shifts over time
- **Model drift**: model performance degrades as the world changes (a model trained on 2022 data may be wrong about 2024 facts)
- **Component drift**: individual components (embedding model, retriever) can degrade independently

**Databricks Lakehouse Monitoring** integrates with model serving endpoints to:

- Track inference request/response patterns over time
- Detect statistical drift in inputs and outputs
- Alert when component or system-level quality drops below thresholds

F2. **Mosaic AI Agent Framework for evaluation**

The **Mosaic AI Agent Framework** is a suite of tooling specifically designed for building, deploying, and evaluating compound AI systems (RAG chains, multi-step agents).

Key evaluation capabilities:

| Feature | Description |
|---|---|
| **Agent Tracing** | Trace agent behavior at each step — see what was retrieved, what was generated, and where the chain went wrong |
| **RAG-specific Metrics** | Built-in support for context precision, recall, faithfulness, and answer relevancy — unified between offline and online monitoring |
| **Review App** | Human-friendly interface for collecting expert feedback on agent responses; no engineering overhead for domain reviewers |
| **Databricks LLM Judges** | Proprietary LLM judge models fine-tuned by Databricks for RAG quality assessment; identify the root cause of low-quality responses |

F3. **The evaluation loop**

The Agent Framework enables a continuous improvement loop:

```
Deploy Agent → Trace Behavior → Collect Human Feedback (Review App)
      ↑                                        ↓
  Redeploy          ←       Evaluate with LLM Judges
                            + Lakehouse Monitoring
```

This loop ensures that quality is maintained over time, not just at deployment.

**Tip:** The Review App is particularly valuable because it bridges the gap between engineering teams (who build the system) and domain experts (who know what "good" looks like). Domain reviewers can annotate agent traces directly without needing to understand the underlying infrastructure.

---

### Key Takeaways — Lesson 4

- End-to-end evaluation covers cost metrics (compute, latency), performance metrics (direct/indirect value), and custom business-aligned metrics.
- The six RAG evaluation metrics span retrieval (context precision, relevancy, recall) and generation (faithfulness, answer relevancy, answer correctness).
- **Faithfulness ≠ Correctness**: faithfulness measures grounding in context; correctness measures accuracy against ground truth.
- **Offline evaluation** validates before deployment; **online evaluation** captures real-world usage and drift.
- Human feedback — both explicit (ratings) and implicit (behavioral) — is essential alongside automated metrics.
- **Databricks Lakehouse Monitoring** detects drift across components and the system as a whole in production.
- **Mosaic AI Agent Framework** unifies offline and online evaluation with agent tracing, RAG-specific metrics, a human Review App, and proprietary Databricks LLM Judges.
