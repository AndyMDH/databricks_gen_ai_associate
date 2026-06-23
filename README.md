# Databricks Generative AI Engineer Associate — Study Notes

Study notes and reference material for the [Databricks Generative AI Engineer Associate](https://partner-academy.databricks.com/learn/learning-plans/315/generative-ai-engineering-pathway) certification.

## What's in this repo

Content was generated using [Exa](https://exa.ai) and [Context7](https://context7.com) against the Databricks partner academy learning plan above.

| File | Description |
|------|-------------|
| [1_fundamentals.md](1_fundamentals.md) | Generative AI fundamentals |
| [2_retrieval_agents.md](2_retrieval_agents.md) | Retrieval and agents |
| [3_single_agent_applications.md](3_single_agent_applications.md) | Single-agent application patterns |
| [4_eval_govern.md](4_eval_govern.md) | Evaluation and governance |
| [5_deploy_monitor.md](5_deploy_monitor.md) | Deployment and monitoring |
| [exam_sections.md](exam_sections.md) | Exam domain breakdown with weightings |
| [study_cheat_sheet.md](study_cheat_sheet.md) | Quick-reference cheat sheet |
| [study_flashcards.md](study_flashcards.md) | Flashcards for key concepts |
| [study_practice_questions.md](study_practice_questions.md) | Practice exam questions |
| [study_topic_summaries.md](study_topic_summaries.md) | Topic summaries |

## Exam domains

| Domain | Weight |
|--------|--------|
| App Development | 30% |
| Assembling and Deploying Applications | 22% |
| Design Applications | 14% |
| Data Preparation | 14% |
| _(remaining domains)_ | 20% |

## Updating this content

If the learning plan changes, regenerate notes by re-running Exa + Context7 against the partner academy URL:

```
https://partner-academy.databricks.com/learn/learning-plans/315/generative-ai-engineering-pathway
```

### Step 1 — Pull fresh content with Exa

Install the SDK and run the script below to fetch up-to-date content from the learning plan page.

```bash
pip install exa-py
export EXA_API_KEY="your_key_here"
```

```python
from exa_py import Exa

exa = Exa(api_key=os.environ["EXA_API_KEY"])

# Fetch the current module content directly from the learning plan URL
results = exa.get_contents(
    ["https://partner-academy.databricks.com/learn/learning-plans/315/generative-ai-engineering-pathway"],
    text={"max_characters": 20000},
    # maxAgeHours=0 forces a live crawl so you always get the latest version
    max_age_hours=0,
)

print(results.results[0].text)
```

If you want to also search for supplementary Databricks GenAI content across the web:

```python
results = exa.search(
    "Databricks Generative AI Engineer Associate exam topics 2025",
    type="deep",
    num_results=10,
    contents={"highlights": True},
)

for r in results.results:
    print(r.title, r.url)
    print(r.highlights)
```

### Step 2 — Supplement with Context7

In Claude Code, use the Context7 MCP tool to pull documentation for specific Databricks/MLflow APIs referenced in the notes (e.g. `mlflow`, `databricks-sdk`). This fills in implementation details that Exa's web crawl may not surface.

### Step 3 — Refresh the markdown files

Paste the Exa/Context7 output into Claude Code and ask it to update the relevant section files. The files map to the learning plan modules:

| File | Module |
|------|--------|
| [1_fundamentals.md](1_fundamentals.md) | Generative AI Fundamentals |
| [2_retrieval_agents.md](2_retrieval_agents.md) | Retrieval & Agents |
| [3_single_agent_applications.md](3_single_agent_applications.md) | Single-Agent Applications |
| [4_eval_govern.md](4_eval_govern.md) | Evaluation & Governance |
| [5_deploy_monitor.md](5_deploy_monitor.md) | Deployment & Monitoring |

Update the section files first, then regenerate the derived study files last:
- [study_cheat_sheet.md](study_cheat_sheet.md)
- [study_flashcards.md](study_flashcards.md)
- [study_practice_questions.md](study_practice_questions.md)
- [study_topic_summaries.md](study_topic_summaries.md)
