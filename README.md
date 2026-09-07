# AyurEvidence

Windows users: follow [START_WINDOWS.md](START_WINDOWS.md) for the exact PowerShell and Docker Desktop workflow.

Working full-stack prototype for **AI-powered Ayurvedic research discovery and evidence mapping**. It connects a modern research query to terminology, demo entities, lawful user-uploaded PDFs, live PubMed metadata, evidence categories, and a traceable research map.

> Research-use only. This is not a prescribing, diagnosis, or treatment system. DEMO relationships are unverified and never imply clinical effectiveness.





How to run it
1. Start Docker Desktop.
2. Open PowerShell.
3. Enter the Desktop project:
Set-Location -LiteralPath "C:\Users\Swarraj\OneDrive\Desktop\SIH AyurEvidence"
4. Start the project:
docker compose up -d
5. Check the services:
docker compose ps
All three should show Up; Neo4j should show healthy.
6. Open:
   - http://localhost:3000
   - http://localhost:8000/docs
   - http://localhost:7474
Neo4j credentials:
Username: neo4j
Password: ayurevidence-demo
Rebuild after code changes
docker compose down
docker compose up --build -d
docker compose ps
Stop the project
docker compose down
Do not add -v, because that would delete the Neo4j volume.
View errors
docker compose logs --tail=100 frontend
docker compose logs --tail=100 backend
docker compose logs --tail=100 neo4j





## Quick start (Docker)

Requirements: Docker Desktop with Compose and about 2 GB free memory.

```bash
cp .env.example .env
docker compose up --build
```

Open the dashboard at http://localhost:3000, Swagger at http://localhost:8000/docs, API health at http://localhost:8000/api/health, and Neo4j Browser at http://localhost:7474 (`neo4j` / password from `.env`).

After Neo4j is healthy, load the replaceable DEMO dictionaries:

```bash
docker compose exec backend python scripts/ingest_data.py
```

The backend remains useful without Neo4j: CSV hybrid search, PDF extraction/RAG, SQLite document storage, and live PubMed search still operate.

## Local development without Docker

Frontend requires Node 22+ and pnpm. Backend requires Python 3.11+.

```bash
pnpm install
pnpm dev
```

In a second terminal:

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r backend/requirements.txt
uvicorn backend.main:app --reload --port 8000
```

Neo4j is optional for this mode; start it with `docker compose up neo4j` and run `python scripts/ingest_data.py`.

## Implemented workflow

1. Rule-based parsing recognizes `AND`, `OR`, and `NOT` and expands a curated synonym dictionary.
2. Local hybrid retrieval combines lexical term coverage, deterministic hashing-vector cosine similarity, metadata, and source weighting. If `sentence-transformers` is installed, `all-MiniLM-L6-v2` is used automatically.
3. Filters are translated to official NCBI E-utilities parameters; PubMed pages are linked by exact PMID.
4. The research map exposes the query, retrieved entities, relationship labels, scores, and expert validation status.
5. PDFs are stored locally, extracted page-by-page with PyMuPDF, chunked with overlap, embedded, and indexed in SQLite.
6. Document chat retrieves relevant chunks by semantic + lexical score and produces an extractive answer with document, page, passage, and relevance. Below threshold it returns `Not found in the uploaded sources.`
7. Evidence categories remain separate: Classical, Traditional, In-vitro, Animal/Preclinical, Clinical, Systematic Review, Meta-analysis, Review, or Unclassified.

The configurable production formula in `.env.example` is BM25 0.45 + semantic 0.35 + metadata 0.10 + source quality 0.10. The dependency-light fallback uses equivalent lexical/semantic/source components and does not pretend to be a trained BM25/vector service.

## PDF and OCR behavior

Text PDFs work out of the box. Image-only pages are explicitly marked as having no extractable text in the lightweight Docker image. To enable OCR, install Tesseract and `pytesseract`; the extraction service is designed so OCR can replace that marker. No uploaded document content leaves the machine.

## Sample data and provenance

`data/` contains 25 concept terms, 55 plant/single-drug entity names, 30 formulation entity names, and five source pointers. Rows are labelled `DEMO_UNVERIFIED`, `DEMO_ENTITY_ONLY`, or `DEMO_SOURCE_POINTER`. They test retrieval/UI flow; they are not a licensed classical corpus and do not assert efficacy.

No paper citations are fabricated. `research_papers.csv` initially contains only a live-fetch marker. Create an exact 100+ metadata seed through the official API with:

```bash
python scripts/fetch_pubmed_seed.py
python scripts/ingest_data.py
```

Only use public-domain/licensed classical texts or PDFs you are legally permitted to process.

## API

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/api/search` | Hybrid query, concept mapping, live papers, graph, summary |
| `POST` | `/api/upload` | Multi-PDF extraction and indexing |
| `POST` | `/api/chat` | Uploaded-document-only grounded Q&A |
| `GET` | `/api/documents` | Document list and processing status |
| `GET` | `/api/document/{id}` | Metadata and extracted passages |
| `GET` | `/api/graph/{query}` | Traceable retrieval graph |
| `GET` | `/api/research` | PubMed-style modern research search |
| `GET` | `/api/evidence/{id}` | Evidence record boundary |
| `POST` | `/api/feedback` | Verify, reject, or correct a relationship |
| `GET` | `/api/health` | Service status and mode |

Interactive schemas are available at `/docs`.

## Structure

```text
app/                    React + TypeScript research dashboard
backend/                FastAPI, Pydantic, retrieval, PDF RAG
data/                   Replaceable CSV DEMO dictionaries
scripts/                Neo4j schema/ingestion and PubMed seed fetch
storage/                Runtime SQLite and uploaded PDFs (ignored)
docker-compose.yml      Frontend, backend, Neo4j
```

## Demo

1. Search `Anti-inflammatory activity`.
2. Inspect terminology, DEMO concepts/plants, and live PubMed records.
3. Open Knowledge Graph and verify/reject a retrieval relationship.
4. Upload PDFs in My Documents and ask `What formulations are mentioned?`.
5. Expand citations to inspect document, page, passage, and relevance.
6. Ask about absent material to confirm the explicit not-found response.

## Safety controls

- No treatment or dosage generation path exists.
- Document answers are extractive in keyless mode with page-level citations.
- No effectiveness score combines unlike evidence categories.
- Failed live metadata is displayed honestly, never replaced by invented papers.
- Lack of indexed clinical evidence is never described as global lack of evidence.
- Suggested graph relationships start `unverified` and preserve expert decisions.

## Known limitations

- OCR is opt-in, not included in the small image.
- Offline hashing vectors are less multilingual than a downloaded transformer.
- Common PubMed filters are shown; custom date ranges are supported by the API.
- Neo4j stores graph entities/validation; PDF chunks use local SQLite vectors.
- Uploaded files have no authentication layer; use on a trusted local machine.
- Classical pointers lack chapter/verse/page metadata until licensed data is supplied.

## Next production steps

Add authenticated workspaces, malware scanning, background jobs, OCR language packs, Neo4j vector indexes, audited multilingual NER, licensed connectors, DOI/PMID deduplication, expert audit trails, and citation-faithfulness/retrieval evaluations.
