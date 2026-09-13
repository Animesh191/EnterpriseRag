# Enterprise RAG Intelligence System

A local, role-aware Retrieval-Augmented Generation (RAG) system for searching enterprise documents across HR, Finance, IT, Compliance, and General information silos.

The system prevents unauthorized documents from entering the retrieval context, combines semantic and keyword search, and returns answers with source citations.

## Run From A Fresh Clone

Prerequisites:

- Windows PowerShell
- Python 3.10 or newer on `PATH`
- [Ollama](https://ollama.com/download) installed and on `PATH`
- Internet access on the first run to install Python packages and download the embedding model

Clone the repository, enter it, and run the bootstrap script:

```powershell
git clone https://github.com/<your-username>/EnterpriseRAG.git
cd EnterpriseRAG
powershell -ExecutionPolicy Bypass -File .\run.ps1
```

The script creates `RAG-MODEL/.venv`, installs `requirements.txt`, starts Ollama when needed, downloads `qwen2.5:3b`, and launches the interactive CLI. It is safe to run again; packages and indexed documents are reused.

If PowerShell execution policy is already configured for local scripts, this is equivalent:

```powershell
.\run.ps1
```

## Demo Commands

Run the predefined eight-query demonstration:

```powershell
.\run.ps1 -Demo
```

Run one non-interactive query:

```powershell
.\run.ps1 -User alice -Query "What was Q2 2025 revenue?"
```

The interactive CLI also supports:

- `whoami` - show the active role and permitted categories
- `switch` - change the active demo user
- `stats` - show vector store and retrieval settings
- `help` - show available commands
- `exit` or `quit` - leave the application

## Architecture

```text
Question
	-> User and role lookup
	-> RBAC allowed-category check
	-> Keyword-based department routing
	-> Routed categories intersected with allowed categories
	-> ChromaDB semantic search plus keyword scoring
	-> Top context chunks inserted into a grounded prompt
	-> Ollama qwen2.5:3b generation
	-> Answer with citations and relevance scores
```

### Access control

The demo users are defined in `RAG-MODEL/enterprise_rag.py`:

| User | Role | Access |
| --- | --- | --- |
| alice | admin | HR, Finance, IT, Compliance, General |
| bob | hr_manager | HR, General |
| carol | finance_analyst | Finance, General |
| dave | it_engineer | IT, General |
| eve | compliance_officer | Compliance, HR, Finance, IT, General |

### Retrieval

Documents are split into overlapping word chunks and stored in ChromaDB using the `all-MiniLM-L6-v2` embedding model. Retrieval combines:

```text
final score = 0.7 * semantic similarity + 0.3 * keyword score
```

Results below the confidence threshold are removed before generation. The prompt instructs Ollama to answer only from the accessible context and cite claims as `[Source N]`.

## Repository Layout

```text
EnterpriseRAG/
|-- run.ps1                         # One-command Windows bootstrap and launcher
|-- requirements.txt                # Python dependencies
|-- README.md
`-- RAG-MODEL/
		|-- enterprise_rag.py           # Application and CLI entry point
		|-- enterprise_data/             # Synthetic enterprise source documents
		`-- chroma_db/                   # Persistent local vector store
```

Supported source formats are TXT, CSV, JSON, and PDF. The included data is synthetic and intended for demonstration.

## Troubleshooting

### `ollama` is not recognized

Install Ollama and restart PowerShell so its installation directory is available on `PATH`.

### The first run is slow

The first run downloads Python packages, the `all-MiniLM-L6-v2` embedding model, and the `qwen2.5:3b` Ollama model. Later runs use the local cache.

### Running the Python file directly

Run it from `RAG-MODEL`, because the application uses relative paths:

```powershell
cd RAG-MODEL
.\.venv\Scripts\python.exe enterprise_rag.py
```

## Production Considerations

This is a local demonstration prototype. A production version should replace the hardcoded users with SSO or OAuth, store document-level ACL metadata, add audit logging and automated evaluation, and deploy the model and vector store behind authenticated services.
