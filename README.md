# 🧬 Biomedical Research Agent

An AI-powered agent for scientific literature search and biological concept analysis, built with **LangChain v1** and **OpenAI GPT-3.5-turbo**.

---

## Overview

This agent combines multiple biomedical databases and LLM reasoning to assist researchers in:
- Searching scientific literature on arXiv and PubMed
- Retrieving protein data from UniProt
- Explaining biological concepts
- Formatting scientific bibliographies

The agent autonomously decides which tools to call based on the user's query, iterating until a complete answer is reached.

---

## Architecture

```
User Query
    │
    ▼
┌─────────────────────────────┐
│   create_agent (LangChain)  │
│   model: gpt-3.5-turbo      │
└────────────┬────────────────┘
             │ ReAct loop
    ┌────────▼────────┐
    │   Tool Router   │
    └────┬────┬───────┘
         │    │
   ┌─────┘    └──────────────────────────┐
   │                                     │
   ▼                                     ▼
┌──────────┐  ┌──────────┐  ┌──────────────────┐  ┌───────────────────┐
│  arXiv   │  │  PubMed  │  │    UniProt API    │  │  LLM (internal)   │
│   API    │  │   API    │  │  (uniprotkb/v2)   │  │  concept explain  │
└──────────┘  └──────────┘  └──────────────────┘  └───────────────────┘
```

---

## Tools

| Tool | Source | Description |
|------|--------|-------------|
| `search_arxiv` | arXiv API | Searches preprint papers by topic (sorted by relevance) |
| `search_pubmed` | NCBI E-utilities | Searches PubMed biomedical literature |
| `fetch_protein_info` | UniProt REST API v2 | Retrieves human protein data by exact gene name |
| `analyze_biological_concept` | GPT-3.5-turbo | Explains biomedical concepts with clinical context |
| `format_bibliography_tool` | GPT-3.5-turbo | Formats paper references in APA style |

---

## Installation

```bash
git clone https://github.com/your-username/biomedical-research-agent.git
cd biomedical-research-agent

pip install langchain langchain-openai arxiv requests
```

---

## Configuration

Set your OpenAI API key:

```bash
export OPENAI_API_KEY="your-api-key-here"
```

Or in a `.env` file:

```
OPENAI_API_KEY=your-api-key-here
```

---

## Usage

```python
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from tools import search_arxiv, search_pubmed, fetch_protein_info, \
                  analyze_biological_concept, format_bibliography_tool

llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0.7)

agent = create_agent(
    model=llm,
    tools=[
        search_arxiv,
        search_pubmed,
        fetch_protein_info,
        analyze_biological_concept,
        format_bibliography_tool,
    ],
    system_prompt=(
        "You are a helpful scientific assistant that supports scientific research. "
        "You can use tools to search PubMed, arXiv and UniProt databases. "
        "Use analyze_biological_concept to explain concepts in a concise way."
    )
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "What are recent advances in CRISPR off-target effect mitigation?"}]
})
print(result["messages"][-1].content)
```

### Example queries

```python
# Literature search
"What are recent advances in CRISPR off-target effect mitigation? Search arXiv and PubMed."

# Protein information
"Tell me about the TP53 protein and its role in cancer."

# Concept explanation
"What is single-cell RNA-seq and why is it important?"
```

---

## Notes on the UniProt tool

The `fetch_protein_info` tool uses `gene_exact` queries and filters by `organism_id:9606` (Homo sapiens) with `reviewed:true` (Swiss-Prot only) to avoid returning related proteins such as TP53BP1 when searching for TP53.

---

## Requirements

| Package | Version |
|---------|---------|
| `langchain` | ≥ 1.0.0 |
| `langchain-openai` | ≥ 1.0.0 |
| `arxiv` | ≥ 2.0.0 |
| `requests` | ≥ 2.28.0 |
| Python | ≥ 3.10 |

---

## License

MIT
