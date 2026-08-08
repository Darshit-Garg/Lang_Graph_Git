# 🎓 Smart Campus Multi-Agent System (LangGraph)

An intelligent, multi-agent AI campus assistant built with **LangGraph**, **LangChain**, and **Groq (`llama-3.3-70b-versatile`)**. The system features stateful agentic workflows, centralized orchestration, domain-specific tool execution, and optional Retrieval-Augmented Generation (RAG) using **ChromaDB**.

---

## 📁 Repository Structure

```text
.
├── folder/
│   ├── chroma_db/             # Persistent ChromaDB vector store (RAG embeddings)
│   ├── campus_policies.txt    # Raw knowledge base data (examination rules, campus policies)
│   ├── main.ipynb             # Full LangGraph workflow WITH RAG integration
│   ├── wRAG.ipynb             # LangGraph workflow WITHOUT RAG integration
│   └── students.json          # Mock database for student records & academic stats
├── src/
│   ├── .env.example.txt       # Example environment configuration
│   ├── .gitignore             # Standard git ignore file
│   ├── .python-version        # Specified Python version pin
│   ├── README.md              # Project documentation
│   ├── pyproject.toml         # Dependencies & package configuration (`uv`)
│   └── uv.lock                # Locked dependency tree

```

---

## 🤖 System Architecture
The core of this project uses an Orchestrator Pattern to dynamically route user queries across domain-specialized agents:

```text
                            ┌────────────────┐
                            │  User Input    │
                            └───────┬────────┘
                                    │
                                    ▼
                          ┌──────────────────┐
                          │   Orchestrator   │
                          └─────────┬────────┘
                                    │
    ┌──────────────┬──────────────┬─┴──────────┬──────────────┬──────────────────┐
    ▼              ▼              ▼            ▼              ▼                  ▼
┌────────┐    ┌────────┐    ┌──────────┐  ┌─────────┐  ┌────────────┐     ┌──────────────┐
│Academic│    │Knowledge│   │Placement │  │ Events  │  │Communication│    │Student Services│
│ Agent  │    │ Agent  │    │  Agent   │  │ Agent   │  │   Agent    │    │    Agent     │
└───┬────┘    └───┬────┘    └────┬─────┘  └───┬─────┘  └─────┬──────┘     └──────┬───────┘
    │             │              │            │              │                   │
  [Tools]      [RAG / DB]     [ATS/Tools]  [Calendar]     [Email]            [Services]
```

---

## 🧩 Specialized Agents & Capabilities

- **Academic Agent** — Retrieves student academic records, CGPA, and attendance stats (`get_student_academics`).
- **Knowledge Agent (RAG)** — Uses RAG via ChromaDB to query official campus handbooks, examination regulations, and institutional FAQs (`query_institutional_knowledge`).
- **Placement Agent** — Evaluates company eligibility, conducts deep ATS resume parsing, delivers interview prep roadmaps, and checks drive schedules.
- **Events Agent** — Handles campus event registration and updates calendar entries.
- **Communication Agent** — Generates formal academic emails to professors, Deans, or coordinators.
- **Student Services Agent** — Answers queries regarding hostel facilities, library status, transport routes, and grievances.

---

## 🚀 RAG vs. Non-RAG Implementations

This project offers two notebook implementations to demonstrate the architectural differences between RAG-augmented agent networks and purely tool-driven agent networks:

| Feature | `wRAG.ipynb` (WITH RAG) | `main.ipynb` (WITHOUT RAG) |
|---|---|---|
| **Knowledge Engine** | ChromaDB Vector Store (HuggingFaceEmbeddings) | Direct Tool / Deterministic Mocks |
| **Policy Search** | Semantic Similarity Search over `campus_policies.txt` | Hardcoded policy functions / Direct LLM reasoning |
| **Persistence** | Persistent database stored in `folder/chroma_db/` | Memory-only session state |
| **Best For** | Querying unstructured handbooks & dynamic PDF policies | Low-latency structured queries & mock API calls |

---

## ⚡ Getting Started

### 1. Prerequisites

Ensure you have `uv` installed (recommended package manager) or standard Python 3.10+.

### 2. Installation

Clone the repository and install dependencies:

```bash
# Sync dependencies using uv
uv sync
```

### 3. Environment Setup

Rename `.env.example.txt` inside `src/` (or create a `.env` in the root) and add your API keys:

```
GROQ_API_KEY=your_groq_api_key_here
HF_TOKEN=your_huggingface_token_here  # Optional: suppresses HF hub rate warnings
```

---

## 🧪 Usage & Testing

You can run either notebook in your preferred Jupyter environment (VSCode, Jupyter Lab, etc.).

### Running with RAG (`folder/main.ipynb`)

1. Open `folder/main.ipynb`.
2. Run all cells to initialize the vector store from `campus_policies.txt`.
3. Test a multi-agent multi-step prompt:

   > "Check attendance stats for student S105. Based on campus regulations, do I qualify for a makeup exam? If yes, draft a formal email to the Dean requesting permission."

### Running WITHOUT RAG (`folder/wRAG.ipynb`)

1. Open `folder/wRAG.ipynb`.
2. Run all cells to launch the lightweight agentic workflow without ChromaDB initialization.

---

## 🔒 Safety & Loop Prevention

Both workflows include strict guardrails against common multi-agent execution issues:

- **Recursion Limits** — Graph executions are capped using `recursion_limit: 15` inside the runtime config.
- **Single-Invocation Rules** — Agent prompts explicitly prevent tool looping (e.g., instructing the RAG/Placement agents to call tools once per query cycle).
