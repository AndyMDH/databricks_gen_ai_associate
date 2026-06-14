# Generative AI Fundamentals

<https://customer-academy.databricks.com/learn/learning-plans/315/generative-ai-engineering-pathway>

## Learning Objectives

- Define Generative AI and its role within broder artificial intelligence landscape, including its relationship to AI Agents as well as machine learning, deep learning, and large language models (LLMs).
- Differentiate between real Generative AI capabilities and common sources of hype and speculation.
- Explain the strategic value of Generative AI, including its potential to drive productivity, innovation, automation, and competitive advantage.
- Identify high-value business use cases for Generative AI, and determine where GenAI is likely to deliver measurable impact.
- Explain the importance of data governance and quality in enabling effective and trustworthy Generative AI, and articulate how governed enterprise data improves GenAI outcomes.

## What is Generative AI?

### Why GenAI Still Matters

- Landscape Shift: not limited to isolated experiments, GenAI is now a standard enterprise capability, and it is the foundational technology underpinning AI Agents.
- Immediate Value: GenAI offers immense business value and delivers standalone solutions immediately, moving beyond simple novelty.
- Foundation: It serves as the critical underlying technology for AI Agents, enabling the next generation of autonomous work.
- The Architecture: To succeed, you must move beyong the 'black box' view. Success requires rigorous governance, quality monitoring, and effective architecture.

### How Large Language Models Work

The Cognitive Engine: Massive Data --> LLM --> Human-Like Output

- Large Language Models are deep learning models (neural networks) created from language data.
  - Neural Network Size: Billions/Trillions of parameters.
  - Data Volume: Learning from vast, diverse text repositories.
  - Emergent Abilities: How massive scale leads to sophisticated reasoning.

Parameters

- Learned parameters are internal values the model learns from the data during training.
- Hyper parameters are the variables that the researcher sets to determine the model's characteristics.
- Together they determine the model's shape, size, and strength.
- The Power vs. Price Tradeoff:
  - Higher Capacity: More parameters allows for greater nuance and more complex reasoning.
  - Higher Cost: Larger models requires more GPU power, leading to higher API expenses.
- The Spectrum of Model Size:
  - Frontier Models (trillion+): For multi-step reasoning and deep creative synthesis.
  - Small Language Models (millions to low billions): Optimized for speed, specific tasks, and local deployment.
  
Tokenization: Breaking Language into Bits

- LLMs are trained on large amounts of text, but that text is broken down into units called tokens.
- Mathematical Operations: Once text is "numbers", the model can perform complex calculations on meaning and structure using probability.
- Not all LLMs tokenize in the same way.
- Input tokens: The number of tokens in the prompt.
- Output tokens: The number of tokens the LLM generates in response.
- For reasoning models, the act of 'reasoning' is included in the overall number of output tokens.

Context: AI's Working Memory

- The context window is the maximum token limit a moddel can remember at once.
- Includes the prompt + retrieved data + model response
- Acts as the working memory for a single session.

The Context Loss Threshold

- When the limit is exceeded, the model drops the earliest information (first-in, first-out).
- Consequences: Loss of initial instructions, logic gaps, and increased risk of hallucinations.

Capacity vs. Strategy

- Large windows enable reasoning over large documents
- Small windows require prompt engineering and shorter, more precise interactions.

Key LLM Elements

To determine performance, cost, latency, and the amount of information a model can consider:

1. Number of Parameters
2. Tokenization
3. Context Window

### How to interact with LLMs

- Without guidance, it's just bad output. The quality of the output is directly tied to the quality of the input.

Capabilities:

- Has read the entire internet: Wikipedia, clinical studies, competitive analysis.
- Never sleeps and never forgets a fact.
- Acts as a tireless reasoning engine.

Catch:

- Has zero context about specific company or private files
- Takes instructions very literally.
- Hallucinates (guesses) when it lacks access to facts.

#### Prompt Engineering: The First Strategy

The process of intentionally designing, refining, and optimizing inputs (prompts) to guide a Generative AI model toward producing the most accurate, relevant, and high-quality output possible.

- Grounding: Providing the specific context and background data the model needs to anchor its response
- Organization: Structuring the instructions logically to reduce ambiguity.
- Token efficiency: Being concise yet comprehensive to maximize the model's context window.

##### Grounding

- Even advanced models rely solely on broad, public training data.
- The solution: Grounding
- The outcome:
  - Ensures output are factually accurate rather than just creative.
  - Maintains relevance to a unique business environment.
  - Transformation from a general-purpose AI to a specialized expert.

##### Organization

- How you ask is as critical as what you ask.
- Poor structure can lead to ambiguity.
- The solution:
  - Clear, descriptive headings
  - Numbered lists for sequential tasks
  - Step-by-step logic sequences
- Increases speed, precision, and alignment.

##### Token Efficiency

- Be concise yet comprehensive
- Irrelevant "fluff" displaces space for:
  - Critical data points
  - Complex, multi-step instructions
- The benefits:
  - Maximizing the model's ability to remember the beginning of a long dialogue.
  - Enabling the model to handle larger volumes of information within a single session.

### GenAI in AI Agents

From GenAI to AI Agents: Reason, Act, Observe

- Adds reasoning and multi-step workflows
- Breaks complex goals into sub-tasks and adjusts strategy based on real-time feedback.
- Agents automate processes that previously required human intervention.

The Agentic Toolkit

- Integration with browsers, code executors, and APIs allows the system to interact with the world.
- Connecting to your datasets
- Moves the system from a general solution to a custom expert
- Ensures workflows are aligned with specific organization contraints and needs

Model Context Protocol

- Open standard for integration between AI models and data sources
- Standardized architecture replaces fragmented, proprietary integrations.
- Enable LLMs to access outside systems without custom code.
- Acts as a bridge for real-time context to produce accurate and grounded responses.

#### AI Agent Systems

Non-agentic (static) workflows

- Hardcoded prompt response or fixed pipelines of hardcoded prompt response systems.
- Deterministic actions

Agentic (dynamic, iterative) workflows

- Planning and execution by AI
- Tool calling by AI
- Non-deterministic actions
- Iterative workflows

#### Core Components of AI Agent Systems

- LLM: A *brain* LLM to control the core logic and sequencing of actions, other LLMs or AI models as needed for sub-tasks and actions.
- Tools: External resournces that the agent uses via tool use or tool calling
- Planning: Complex goals become manageable tasks.
- Memory:
  - Short-term session and current conversational state tracking to help with planning and execution of subsequent actions.
  - Long-term episodic, semantic, and procedural memory for historical state, knowledge, preferences.

#### Use Cases

- Intelligent Document Processing: Extract insights from documents at scale
- Knowledge Base and Search: Search and retrieval
- Machine Learning and AI: Combine classical ML and GenAI

#### Choosing An AI Strategy

Non-Agentic:

- Linear, human-lead, and highly predictable.
- If-this-then-that workflows, with stable, predefined logic.
- Low Cost & High Speed
- Consistency
- Ease of Governance: Simple to debug and audit
- Summarization, Translation, Generating Content from a structured template.

Agentic:

- Non-linear, Goal-oriented, and iterative.
- Open-ended objectives requiring Reason-Act-Observe loop.
- Reasoning Capability: adjust its strategy based on real-world feedback
- Tool integration: Can autonomously use APIs, search the web, and query databases
- Proactive Action: Moves towards a goal with minimal human intervention.
- Writing a Python script to analyze a dataset, identifying outliers, and generating a visual report with strategic recommendations.

## GenAI Opportunities

### AI Model Classes

A Hierarchy of Intelligence

- Beyond Parameters: A focus on reasoning depth, latency, and cost-efficiency
- The Reasoning Revolution: Allowing models to stop and think to solve complex logic before responding
- Native Multimodality: Text, image, audio, and video processing are now standard across all tiers

Classes

1. Small: 7B - 13B
   - Fast, Inexpensive
   - Good for Extraction, Formatting, Simple Q&A
2. Medium: 30B - 70B
   - Balance quality and cost
   - Good for Support, Content generation
3. Large: 70B+
   - Strong reasoning and comprehensive abilities
   - Good for complex, ambiguous tasks
4. Frontier: State-of-the-art
   - High-stakes scenarios where quality outweighs cost

Selection:

- Minimum Level: Complexity of Tasks, Quality Needs
- Second Level: Cost, Latency
- Deal Breaker: Privacy, Deployment Constraints

### Recognizing High-Value GenAI Opportunities

#### Non-Agentic Applications

Content Generation:

- Drafting Short Form Copy
- Language Localization
- Product Descriptions

#### Agentic Applications

Content Generation:

- Hyper-Personlized Outreach
- Autonomous Competitor Research
- Compliance-Aware Content Review

#### Decision Tree

- Can output quality be measured clearly?
  - No -> Refine and Start Over
  - Yes -> Next
- Does the task require deterministic accuracy?
  - Yes -> Rules, Classical ML, SQL
  - No -> Next
- Does the tasks involve language understanding or generation?
  - No -> Rules, Classical ML, SQL
  - Yes -> Next
- Does the task require synthesis or open-ended output?
  - No -> Classical ML, classification
  - Yes -> Next
- Does the task depend on proprietary enterprise knowledge?
  - Yes -> GenAI + RAG
  - No -> GenAI

### Model Evaluation

- Evaluation allows you to:
  - Move from generic benchmarks to metrics based on your specific workflows
  - Confidently downgrade from high-cost frontier models
  - Combat hallucinations by using ground truth data

#### Techniques

Human-In-The-Loop

- Use human agents to Review, Edit, Validate
- Pros: Trust and safety, Nuance and context, Gold Standard
- Cons: Scalability & Cost, Latency, Subjectivity and bias

LLM-as-Judge

- Uses a judge model that grades other models, uses rubrics, provides detailed justification
- Pros: Explainable verdicts, Extreme scalability & lower cost, Elimination of human fatigue
- Cons: Inherent model biases (verbosity, positional, ecosystem), Lack of human intuition, Version drift and reproducibility.

Benchmark

- Establish an objective baseline, identify critical weaknesses
- Pros: Rapid regression testing, Cost and latency optimization, Operational consistency
- Cons: Gaming the system, The "lab vs. wild" gap, Data contamination

Databricks Synthetic Data Generation API

- Rapid dataset creation
- Proprietary grounding
- SME optimization
- Seamless integration

## Governance

### Limitations and Risks

- Hallucinations: Model generates factually incorrect but highly confident statements
- Stale Knowledge: training data has a fixed cutoff date
- Privacy Leakage: Unauthorized or Unintended exposure of sensitive data during the lifecycle of an AI model
- PII: data that can be used to distinguish or trace an individual's identity, either alone or when combined with other data.
- Auditability: degree to which an AI system's decision-making process, data inputs, and operational behaviours can be monitored, recorded and verified by an independent party
- Lineage: process of tracking and documenting the entire lifecycle of a data asset as it moves from its point of origin to its final consumption

### Why Governance is Essential

- Tracks Lineage
  - Implement a continuous paper trail
  - Regulatory & audit readiness
  - Verifiable data provenance
  - Accountability & risk mitigation
- Ensures Data & Model Quality
  - Accurate, representative, and high-fidelity
  - Data profiling and validation checks
  - Fostering organizational trust
- Provides Model Versioning
  - Move beyond chaotic experimentation
  - Seamless rollback
  - Operational stability, testing, meeting legal requirements

### RAG

- Benefits
  - Updates instantly when data changes
  - Improves accuracy
  - Reduces hallucinations
  - Enhances trust
- Uses
  - Customer support & technical troubleshooting
  - Legal, risk, and compliance
  - Internal search & knowledge management
  - HR and operational policy inquiries

## Implementing Generative AI on Databricks

### Data Intelligence Platform

- Unified platform for data + genAI
- Model flexibility & side-by-side comparison
- Integrated governance with Unity Catalog
- End-to-end development & fast iteration

Workflows:

1. Preparation: Unity Catalog Delta Lake
2. Vector Indicies: Databricks Vector Search
3. Retrieve Context via RAG: Databricks Vector Search
4. Generate Grounded Outputs: mlflow.predict()
5. Register the model: MLFlow Unity Catalog
6. Deploy: Databricks Model Serving, AI Gateway, Lakebase Postgres, Databricks Apps
7. Evaluate & Debug: MLFlow

### Evaluating and Improving GenAI Outputs

LLM-as-Judge on Databricks

- High-reasoning models
- Key quality metrics
  - Groundedness
  - Relevance
  - Correctness
- Custom Judges
  - Examples: Brand voice, regulatory compliance

MLFlow-powered capabilities

- Tunable Judges
  - Natural language instructions
  - Human-in-the-loop
  - Alignment
- Agent-as-a-Judge
  - Automated trace intelligence
  - Evaluates tool call correctness, argument validity, and redundancy
- Judge Builder
  - Visual Workflow
  - Lifecycle management

Trace Based Debugging

- MLFlow tracing
- **Span**: represents a single unit of work or a discrete "step" within a larger Generative AI workflow
- Root cause analysis
- OpenTelemetry
- Agent Bricks Custom Agents

Human Review Apps

- Capture feedback from subject matter experts
- Align the whole application or the LLM judges being used to evaluate the application
- Can help to signal when a model:
  - Is ready for production
  - Needs refinement
  - Should be retired

Continuous Monitoring

- Moving from *deploy & forget* to perpetual improvement
- Inference tables: Captures the inputs to, and outputs from, an application
- Data Quality Monitoring
  - Anomaly detection
  - Data profiling
- Semantic drift
  - User queries or model output shifts over time
- Integrates with Databricks SQL Alerts and Dashboards

### Agent Bricks

- Specify problem: Give data and tell us the task
- Optimize on enterprise data: Build the best agent system on quality vs. cost
- Continuously improve: Get to quality with Agent Learning on Human Feedback (ALHF)

#### Information Extraction Brick

- Automated data structuring
- Optimized for complexity and scale
- Enterprise-grade governance & quality
  - Unity Catalog integration
  - MLFlow
- Rapid assessment

#### Knowledge Assistant Brick

- Advanced Grounding with **Instructed Retrieval**
  - More accurate
  - Context-aware
  - Reliable citations
- Integrated with Unity Catalog and MLFlow tracing
  - Governed
  - Transparent
  - Auditable

#### Supervisor Agent

- Advanced orchestration
- Coordinates multiple specialized AI Agents
- "Routes" by analyzing user intent and directs requests to other agents
- Scalable and maintainable
  - Develop separate agents
  - Optimize independently
