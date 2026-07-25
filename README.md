# 📧 AI Email Intelligence System

> An end-to-end, production-grade AI platform for classifying, prioritizing, routing, summarizing, and drafting replies to enterprise email — replacing manual triage and brittle keyword rules with an ML/NLP pipeline + LLM-RAG layer.

[![Status](https://img.shields.io/badge/status-Phase%201%20%E2%80%94%20Foundation-orange)]()
[![License](https://img.shields.io/badge/license-Internal%2FPortfolio-lightgrey)]()
[![Python](https://img.shields.io/badge/python-3.11%2B-blue)]()

> ⚠️ **Project Status:** This repository is currently at **Phase 1 (PRD + Architecture)**. The sections below describe the **target system** as specified in [`docs/PRD.md`](docs/PRD.md). Setup/Run/Docker instructions are the **planned developer workflow** — implementation is rolled out phase-by-phase per the [roadmap](#roadmap). Check `CHANGELOG.md` (once it exists) or open issues for actual build status before assuming a command below works today.

---

## Table of Contents

- [Why This Exists](#why-this-exists)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Setup Instructions](#setup-instructions)
- [Running Locally](#running-locally)
- [Docker Setup](#docker-setup)
- [API Documentation](#api-documentation)
- [Project Structure](#project-structure)
- [Roadmap](#roadmap)
- [Evaluation Metrics](#evaluation-metrics)
- [Security & Compliance](#security--compliance)
- [Contributing](#contributing)
- [License](#license)

---

## Why This Exists

Knowledge workers lose ~28% of their workday to email triage. Rule-based spam filters and manual routing don't scale against adversarial spam, ambiguous priority, and multi-intent messages. This system applies fine-tuned transformer models + a RAG-based auto-reply pipeline to automate classification, routing, and first-draft response generation — with a human-in-the-loop feedback system for continuous correction.

Full problem framing, ROI model, and competitive analysis: [`docs/PRD.md`](docs/PRD.md), §1–§3.

---

## Features

### Core ML/NLP Capabilities
| Capability | Description | Status |
|---|---|---|
| **Spam Detection** | Binary classification, Recall-optimized (>99% target) | 📋 Planned (Phase 3) |
| **Priority Classification** | 4-class (Critical/High/Medium/Low) using text + metadata signals | 📋 Planned (Phase 4) |
| **Multi-label Categorization** | Support/Sales/Billing/Legal/HR/IT/Marketing/General | 📋 Planned (Phase 5) |
| **Summarization** | Extractive + abstractive (T5/BART), thread-aware | 📋 Planned (Phase 6) |
| **Sentiment & Emotion Detection** | 5-class sentiment + emotion (anger, frustration, urgency...) | 📋 Planned (Phase 7) |
| **Intent Detection** | Multi-label intent (complaint, refund, escalation, etc.) + NER | 📋 Planned (Phase 7) |
| **Department Routing** | Hybrid rule-engine + ML classifier, configurable via UI | 📋 Planned (Phase 8) |
| **Semantic Similarity Search (RAG foundation)** | Top-K similar past emails via vector embeddings | 📋 Planned (Phase 9) |
| **Auto-Reply Generation** | LLM + RAG draft generation, human-approval gated | 📋 Planned (Phase 10) |
| **Manager Dashboard** | Real-time volume, SLA, sentiment trend, agent workload | 📋 Planned (Phase 11) |
| **Human Feedback Loop** | One-click correction → logged → triggers retraining | 📋 Planned (Phase 12) |
| **Multi-Agent Orchestration** | Async agent coordination via Kafka, circuit-breaker fallback | 📋 Planned (Phase 13) |

### Platform Capabilities
- PII detection & anonymization (Microsoft Presidio) before any external LLM call
- Explainable predictions (SHAP/LIME) — no black-box classification
- Configurable routing rules via admin UI (no redeploy needed)
- Full audit trail: every classification, override, and routing decision logged
- Horizontal scaling path: Docker Compose (MVP) → Kubernetes (production)

Full functional requirements (72 FRs across 13 modules): [`docs/PRD.md`](docs/PRD.md), §5.

---

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| **Language** | Python 3.11+ | Async support, ML ecosystem |
| **API Framework** | FastAPI 0.110+ | Auto OpenAPI docs, Pydantic validation |
| **ML Framework** | PyTorch + HuggingFace Transformers | BERT/RoBERTa/T5/BART fine-tuning |
| **LLM (Cloud)** | OpenAI GPT-4o / Anthropic Claude | Auto-reply generation |
| **LLM (Local fallback)** | Ollama + Mistral 7B | Self-hosted, PII-sensitive orgs |
| **LLM Orchestration** | LangChain 0.2+ | Prompt templates, RAG chains |
| **Embeddings** | sentence-transformers (`all-MiniLM-L6-v2`) | 384-dim, fast |
| **Vector DB** | ChromaDB (dev) → Pinecone (prod) | Semantic similarity search |
| **Relational DB** | PostgreSQL 15 | Metadata, classifications, feedback |
| **Cache** | Redis 7 | API caching, sessions |
| **Message Queue** | Apache Kafka 3.x | Event-driven agent pipeline |
| **Task Queue** | Celery 5.x | Async execution |
| **Object Storage** | AWS S3 / MinIO (local) | Raw `.eml` storage |
| **Experiment Tracking** | MLflow 2.x | Model registry, versioning |
| **Monitoring** | Prometheus + Grafana | Metrics, drift dashboards |
| **Logging/Tracing** | ELK Stack + OpenTelemetry/Jaeger | Structured logs, distributed tracing |
| **Containers** | Docker 25+, Docker Compose v2 | Local dev |
| **Orchestration (Prod)** | Kubernetes 1.28+ | Auto-scaling |
| **CI/CD** | GitHub Actions | Lint → test → build → deploy |
| **IaC** | Terraform 1.6+ | Cloud provisioning |
| **Frontend** | React 18 + TypeScript + TailwindCSS | Dashboard / Agent UI |

PII/annotation/NLP tooling (Presidio, spaCy, Optuna, SHAP, Label Studio): [`docs/PRD.md`](docs/PRD.md), §11.2.

---

## Architecture

```mermaid
flowchart TD
    subgraph Clients["Clients"]
        C1[Email Servers]
        C2[Web Dashboard]
        C3[Agent UI]
    end

    GW["API Gateway<br/>JWT Auth · Rate Limiting · SSL"]

    subgraph Entry["Entry Services"]
        ING[Ingestion Service]
        REST[REST API]
        WS[WebSocket Service]
    end

    KAFKA[("Apache Kafka<br/>email.raw → email.preprocessed → email.classified → feedback")]

    subgraph Consumers["Event Consumers"]
        PRE["Preprocessing<br/>(PII Detection + Clean)"]
        FB[Feedback / Alert Services]
    end

    subgraph Orchestrator["ML Orchestrator (Multi-Agent, parallel dispatch)"]
        A1[Spam Agent<br/>BERT]
        A2[Priority Agent<br/>DistilBERT]
        A3[Categorization Agent<br/>BERT multi-label]
        A4[Sentiment Agent<br/>RoBERTa]
        A5[Intent Agent<br/>BERT]
        A6[Summarization Agent<br/>T5/BART]
        A7[Routing Agent<br/>Rules + ML hybrid]
        A8[Auto-Reply Agent<br/>LLM + RAG]
    end

    subgraph Storage["Storage Layer"]
        PG[("PostgreSQL")]
        VDB[("Vector DB<br/>Chroma/Pinecone")]
        RD[("Redis")]
    end

    subgraph Observability["Observability Stack"]
        MLF[MLflow]
        PROM[Prometheus/Grafana]
        ELK[ELK]
        JAE[Jaeger]
    end

    C1 --> GW
    C2 --> GW
    C3 --> GW
    GW --> ING
    GW --> REST
    GW --> WS
    ING --> KAFKA
    REST --> KAFKA
    KAFKA --> PRE
    KAFKA --> FB
    PRE --> A1 & A2 & A3 & A4 & A5 & A6
    A1 & A2 & A3 & A4 & A5 --> A7
    A7 --> A8
    A6 --> A8
    A7 --> PG
    A8 --> PG
    A1 & A2 & A3 & A4 & A5 & A6 --> VDB
    REST --> RD
    PG --> MLF
    PG --> PROM
    PG --> ELK
    PG --> JAE

    classDef client fill:#E3F2FD,stroke:#1565C0,stroke-width:1px,color:#0D47A1
    classDef gateway fill:#FFF3E0,stroke:#EF6C00,stroke-width:1px,color:#E65100
    classDef entry fill:#F3E5F5,stroke:#7B1FA2,stroke-width:1px,color:#4A148C
    classDef queue fill:#FFFDE7,stroke:#F9A825,stroke-width:1px,color:#5D4037
    classDef consumer fill:#E8F5E9,stroke:#2E7D32,stroke-width:1px,color:#1B5E20
    classDef agent fill:#E8EAF6,stroke:#3949AB,stroke-width:1px,color:#1A237E
    classDef storage fill:#E0F7FA,stroke:#00838F,stroke-width:1px,color:#004D40
    classDef obs fill:#ECEFF1,stroke:#546E7A,stroke-width:1px,color:#263238

    class C1,C2,C3 client
    class GW gateway
    class ING,REST,WS entry
    class KAFKA queue
    class PRE,FB consumer
    class A1,A2,A3,A4,A5,A6,A7,A8 agent
    class PG,VDB,RD storage
    class MLF,PROM,ELK,JAE obs
```

**Key architectural patterns:** Event-driven (Kafka), microservices per ML module, CQRS (read/write path separation for dashboard vs. pipeline), circuit breaker (TF-IDF fallback if BERT service degrades), saga pattern for partial-failure handling.

Full diagrams, data flow (17-step pipeline), and pattern rationale: [`docs/PRD.md`](docs/PRD.md), §10.

---

## Setup Instructions

### Prerequisites
- Python **3.11+**
- Docker **25+** and Docker Compose **v2**
- Node.js **18+** (for the dashboard)
- (Optional) NVIDIA GPU + CUDA, for local model fine-tuning
- API keys: OpenAI / Anthropic (for auto-reply), Gmail/Microsoft Graph OAuth credentials (for ingestion)

### 1. Clone the repository
```bash
git clone https://github.com/<your-org>/email-intelligence-system.git
cd email-intelligence-system
```

### 2. Configure environment variables
```bash
cp .env.example .env
```
Populate `.env` with at minimum:
```ini
# Database
POSTGRES_USER=emailai
POSTGRES_PASSWORD=changeme
POSTGRES_DB=emailai
DATABASE_URL=postgresql://emailai:changeme@localhost:5432/emailai

# Message Queue
KAFKA_BROKER_URL=localhost:9092

# Vector DB
VECTOR_DB_PROVIDER=chromadb        # chromadb | pinecone
PINECONE_API_KEY=

# LLM Providers
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
OLLAMA_BASE_URL=http://localhost:11434

# Email Ingestion
GMAIL_CLIENT_ID=
GMAIL_CLIENT_SECRET=
MS_GRAPH_CLIENT_ID=
MS_GRAPH_CLIENT_SECRET=

# Security
JWT_SECRET_KEY=
```
> Never commit `.env`. Secrets in production are managed via HashiCorp Vault / AWS Secrets Manager, not env files (see PRD §6.4).

### 3. Install Python dependencies
```bash
python3.11 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 4. Install dashboard dependencies
```bash
cd dashboard
npm install
cd ..
```

---

## Running Locally

### Option A — Full stack via Docker Compose (recommended)
```bash
make up
# equivalent to: docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build
```
This brings up: PostgreSQL, Redis, Kafka + Zookeeper, ChromaDB, the FastAPI backend, ML agent workers, and the React dashboard.

### Option B — Run services individually (debugging)
```bash
# 1. Start infrastructure only
docker compose up -d postgres redis kafka zookeeper chromadb

# 2. Run DB migrations
alembic upgrade head

# 3. Start the API server
uvicorn src.api.main:app --reload --port 8000

# 4. Start ML agent workers (separate terminal)
python -m src.agents.orchestrator

# 5. Start the dashboard
cd dashboard && npm run dev
```

### Useful Make targets
```bash
make test       # run unit + integration tests
make lint       # ruff / black / mypy
make seed       # load sample email dataset for local testing
make down       # stop all services
```

- API: `http://localhost:8000`
- Swagger/OpenAPI docs: `http://localhost:8000/docs`
- Dashboard: `http://localhost:3000`
- MLflow UI: `http://localhost:5000`
- Grafana: `http://localhost:3001`

---

## Docker Setup

`docker-compose.yml` defines the core service topology:

| Service | Image / Build | Port | Purpose |
|---|---|---|---|
| `api` | `./infrastructure/docker/Dockerfile.api` | 8000 | FastAPI REST + WebSocket |
| `orchestrator` | `./infrastructure/docker/Dockerfile.orchestrator` | — | Multi-agent ML dispatcher |
| `postgres` | `postgres:15` | 5432 | Metadata, classifications, feedback |
| `redis` | `redis:7` | 6379 | Cache, sessions |
| `kafka` + `zookeeper` | `confluentinc/cp-kafka` | 9092 | Event bus |
| `chromadb` | `chromadb/chroma` | 8001 | Vector DB (dev) |
| `mlflow` | `./infrastructure/docker/Dockerfile.mlflow` | 5000 | Experiment tracking / model registry |
| `dashboard` | `./dashboard` (Node build) | 3000 | React UI |
| `prometheus` | `prom/prometheus` | 9090 | Metrics scrape |
| `grafana` | `grafana/grafana` | 3001 | Dashboards |

Build and run a single service:
```bash
docker compose build api
docker compose up -d api
```

Production deployment uses **Kubernetes manifests** (`infrastructure/kubernetes/`) and **Helm charts** (`infrastructure/helm/`) — Docker Compose is dev-only. See PRD §11 / §16 (Phase 14) for the AWS EKS rollout plan.

---

## API Documentation

Interactive docs are auto-generated by FastAPI at `/docs` (Swagger) and `/redoc`. Below is a summary — see [`docs/PRD.md`](docs/PRD.md) §13 for full request/response schemas.

**Base URL:** `https://api.emailai.yourdomain.com/api/v1`
**Auth:** Bearer JWT (`Authorization: Bearer <token>`)
**Rate limit:** 1,000 req/min/org (configurable)
**Errors:** RFC 7807 Problem Details

```
── Email Processing ─────────────────────────────────
POST   /emails/ingest                         Queue email for analysis
GET    /emails/{id}                           Full ML results
GET    /emails/{id}/status                    Async processing status
GET    /emails/{id}/summary                   Summary only
GET    /emails/{id}/similar                   Top-K similar emails
POST   /emails/{id}/reply/generate            Trigger auto-reply
GET    /emails/{id}/reply                     Get draft
POST   /emails/{id}/reply/{reply_id}/approve  Approve draft
POST   /emails/{id}/reply/{reply_id}/reject   Reject with reason

── Inbox ─────────────────────────────────────────────
GET    /emails   ?priority&category&status&department&date_from&date_to&limit&cursor

── Feedback ──────────────────────────────────────────
POST   /emails/{id}/feedback                  Submit correction
GET    /feedback/stats
GET    /feedback/pending-training

── Routing ───────────────────────────────────────────
GET/POST/PUT/DELETE   /routing/rules
POST   /routing/simulate

── Analytics ─────────────────────────────────────────
GET    /analytics/overview | volume | categories | sentiment | sla | agents | models
POST   /analytics/export

── Model Management ──────────────────────────────────
GET    /models
POST   /models/{module}/promote
POST   /models/retrain/trigger
GET    /models/metrics/current

── System ────────────────────────────────────────────
GET    /health | /health/ready | /metrics
```

### Example: Ingest → Result
```json
// POST /api/v1/emails/ingest → 202 Accepted
{
  "email_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "queued",
  "poll_url": "/api/v1/emails/550e8400-e29b-41d4-a716-446655440000/status",
  "estimated_completion_ms": 3500
}
```
```json
// GET /api/v1/emails/{id} (after processing)
{
  "status": "done",
  "spam": { "is_spam": false, "score": 0.02 },
  "priority": { "label": "High", "confidence": 0.91 },
  "categories": [{ "label": "Customer Support", "confidence": 0.95 }],
  "sentiment": { "label": "Very Negative", "emotions": [{ "emotion": "Frustration", "confidence": 0.89 }] },
  "intents": [{ "intent": "Complaint", "confidence": 0.94 }],
  "routing": { "department": "Customer Support — Shipping", "confidence": 0.93 },
  "total_processing_ms": 3847
}
```

---

## Project Structure

```
email-intelligence-system/
├── .github/workflows/        # CI (test), CD (deploy), scheduled model-eval
├── docs/                     # PRD, architecture, ADRs, runbook
├── data/                     # raw/ processed/ annotations/ schemas/ (git-ignored raw)
├── notebooks/                # EDA + model training notebooks (01–10)
├── src/
│   ├── ingestion/            # IMAP, Gmail, MS Graph clients + parser
│   ├── preprocessing/        # HTML cleaning, PII detection, normalization
│   ├── ml/                   # spam/ priority/ categorization/ summarization/
│   │                         # sentiment/ intent/ embeddings/
│   ├── agents/                # orchestrator, base_agent, specialist_agents/
│   ├── routing/               # rule_engine, ml_router
│   ├── rag/                   # vector_store, retriever, context_builder
│   ├── reply/                 # llm_client, prompt_templates, reply_generator
│   └── api/                   # FastAPI app, routers, schemas, middleware
├── dashboard/                 # React + TypeScript frontend
├── infrastructure/            # docker/ kubernetes/ helm/ terraform/
├── tests/                     # unit/ integration/ e2e/
├── monitoring/                # prometheus/ grafana/ configs
├── docker-compose.yml
├── docker-compose.dev.yml
├── Makefile
├── pyproject.toml
├── .env.example
└── README.md
```

Full tree with per-module file breakdown: [`docs/PRD.md`](docs/PRD.md), Appendix A.

---

## Roadmap

| Phase | Deliverable | Duration |
|---|---|---|
| 1 | PRD + Architecture (this doc) | 1 wk |
| 2 | Dataset collection + EDA | 1.5 wk |
| 3 | Spam Detection API | 1.5 wk |
| 4 | Priority Classification API | 1.5 wk |
| 5 | Categorization API | 1.5 wk |
| 6 | Summarization API | 2 wk |
| 7 | Sentiment + Intent API | 2 wk |
| 8 | Routing Engine | 1.5 wk |
| 9 | Vector DB + Similarity API | 2 wk |
| 10 | Auto-Reply (LLM+RAG) API | 2 wk |
| 11 | Dashboard & Analytics | 2 wk |
| 12 | Dockerization | 1 wk |
| 13 | CI/CD | 1 wk |
| 14 | Cloud Deployment (K8s/EKS) | 1.5 wk |
| 15 | Monitoring & MLOps | 1.5 wk |

~24 weeks part-time / ~12 weeks full-time. Details: PRD §16.

---

## Evaluation Metrics

| Module | Primary Metric | Threshold |
|---|---|---|
| Spam Detection | Recall | > 99% |
| Priority | F1-macro | > 0.85 (High/Critical) |
| Categorization | F1-micro | > 0.80 |
| Summarization | ROUGE-L / BERTScore | BERTScore > 0.85 |
| Sentiment | F1-weighted | > 0.82 |
| Intent | F1-macro | > 0.80 |
| Routing | Accuracy | > 92% |
| Similarity Retrieval | Hit Rate@5 | > 0.85 |
| Auto-Reply | Human Acceptance Rate | > 60% |

Full metric rationale (why Recall, not Accuracy, for spam) and system-level SLAs: PRD §14.

---

## Security & Compliance

- **PII anonymization** (Microsoft Presidio) before any data leaves the system for an external LLM
- **Encryption:** AES-256 at rest, TLS 1.3 in transit
- **AuthN/AuthZ:** JWT (RS256) + RBAC (Agent/Manager/Admin)
- **GDPR:** right-to-deletion endpoint, configurable retention, audit logging
- **Secrets:** HashiCorp Vault / AWS Secrets Manager — never hardcoded

Full requirements: PRD §6.4.

---

## Contributing

This is currently a solo portfolio build (MSc AI/ML project). Contribution guidelines, branch strategy, and PR templates will be added once Phase 2 (dataset/EDA) lands. For now: open an issue before submitting a PR.

## License

Internal / Portfolio use. No license has been formally assigned yet — treat as **all rights reserved** until a `LICENSE` file is added.
