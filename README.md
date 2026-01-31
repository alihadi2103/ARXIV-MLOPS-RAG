
# Arxiv Mlops Rag Systems 

<div align="center">
  <h3 Production RAG Systems</h3>
  <strong>RAG (Retrieval-Augmented Generation)</strong></p>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.12+-blue.svg" alt="Python Version">
  <img src="https://img.shields.io/badge/FastAPI-0.115+-green.svg" alt="FastAPI">
  <img src="https://img.shields.io/badge/OpenSearch-2.19-orange.svg" alt="OpenSearch">
  <img src="https://img.shields.io/badge/Docker-Compose-blue.svg" alt="Docker">
 
</p>

</br>

<p align="center">
  <a href="#-about-this-course">
    <img src="static/mother_of_ai_project_rag_architecture.gif" alt="RAG Architecture" width="700">
  </a>
</p>




## 📖 Project Overview (This Repository)

This repository contains a **production-oriented, end-to-end Agentic RAG system** together with a **separate data ingestion and indexing pipeline**, all orchestrated using **Docker Compose**.

The system is designed to mirror **real-world AI engineering practice**:

* Data ingestion is **decoupled** from serving
* Search is built on **strong keyword foundations (BM25)** and extended with **vector semantics**
* Reasoning is handled by **agentic workflows**, not single-pass prompts
* Performance, observability, and scalability are first-class concerns

This is not a demo-only RAG. It is an **MLOps-aware architecture** intended to scale toward real production environments.

---

## 🏗️ High-Level Architecture (AWS EKS Deployment Target)

This architecture is **designed to be deployed on AWS EKS (Elastic Kubernetes Service)**. Local Docker Compose is used for development and validation, while Kubernetes manifests and Helm charts are intended for cloud deployment.

Key cloud-native principles applied:

* Stateless application pods
* Externalized state (PostgreSQL, OpenSearch, Redis)
* Horizontal scalability
* Observability-first design

The system is composed of two major subsystems:

1. **Data Ingestion & Indexing Pipeline** (Airflow-driven)
2. **Agentic RAG Serving System** (FastAPI + LangGraph)

Both subsystems are deployed and coordinated via **Docker Compose**, while remaining logically independent.

---

## 🔁 Data Ingestion & Indexing Pipeline (Airflow)

The ingestion pipeline is responsible for **continuously building and maintaining the knowledge base**.

### Responsibilities

* Fetch academic papers from **arXiv**
* Parse scientific PDFs into structured text
* Extract and normalize metadata (title, authors, categories, dates)
* Chunk documents intelligently for retrieval
* Index data into **OpenSearch** for hybrid retrieval
* Persist metadata in **PostgreSQL**

### Why a Dedicated Pipeline?

In production systems, ingestion **must not** be coupled to user traffic. Using Airflow enables:

* Backfills and reprocessing
* Retry logic and fault tolerance
* Scheduled or event-driven ingestion
* Clear observability of data freshness

### Airflow Structure

```
airflow/
├── dags/              # DAG definitions (fetch → parse → index)
├── Dockerfile         # Airflow container image
├── workers/           # Task execution logic
├── entrypoint.sh      # Container bootstrap
├── worker.sh          # Worker startup script
├── requirements-airflow.txt
└── README.md
```

Each DAG represents a **data lifecycle**, from external API to searchable index.

---

## 🔍 Hybrid RAG Architecture (Core Concept)

This system follows a **hybrid retrieval-first philosophy**, which is critical for production RAG.

### Why Hybrid Retrieval?

Pure vector search often fails due to:

* Poor handling of rare keywords
* Loss of exact constraints (authors, dates, domains)
* Weak transparency and debuggability

This project solves that by combining:

### 1️⃣ Metadata & Keyword Search (BM25)

* Implemented using **OpenSearch BM25**
* Operates on:

  * Titles
  * Abstracts
  * Section text
  * Structured metadata (authors, categories, publication date)

This layer provides:

* Precision
* Filterability
* Explainable relevance scoring

### 2️⃣ Vector Search (Semantic Retrieval)

* Chunk embeddings generated via **Jina AI**
* Captures semantic similarity beyond exact wording
* Enables concept-level retrieval

### 3️⃣ Hybrid Fusion (RRF)

Results from keyword and vector search are merged using **Reciprocal Rank Fusion (RRF)**.

This ensures:

* Keyword precision is preserved
* Semantic recall is enhanced
* No single retrieval mode dominates

> This mirrors how **real search-driven AI systems** are built in industry.

---

## 🧠 Agentic RAG Layer (LangGraph)

On top of hybrid retrieval sits an **agentic reasoning layer**.

Instead of a single retrieval → generation step, the system executes a **stateful workflow**.

### Agent Responsibilities

* **Guardrails**: detect out-of-domain or unsafe queries
* **Retrieval orchestration**: choose hybrid strategies
* **Document grading**: assess relevance of retrieved chunks
* **Query rewriting**: refine user queries when results are weak
* **Answer generation**: produce grounded, source-aware responses

### Why Agentic RAG?

Agentic workflows allow the system to:

* Recover from poor retrieval
* Adapt to ambiguous queries
* Avoid hallucination
* Expose reasoning steps for debugging

This is essential for **complex research queries**.

---

## ⚡ Caching & Observability

### Redis (Caching Layer)

Redis is used as an **ephemeral performance layer**:

* Cache final LLM responses
* Cache intermediate agent decisions
* Reduce repeated LLM and retrieval costs

This enables:

* Lower latency
* Higher throughput
* Cost-efficient scaling

### Langfuse (Observability)

Langfuse provides:

* End-to-end tracing of RAG calls
* Prompt and response inspection
* Latency and error analysis

Together, Redis and Langfuse make the system **operationally transparent**.

---

## 🌐 Serving Layer & Interfaces

The serving system exposes functionality through multiple interfaces:

* **FastAPI**: REST endpoints for programmatic access
* **Gradio**: interactive UI for experimentation
* **Telegram Bot**: mobile-first conversational access

All interfaces share the same **agentic backend**, ensuring consistency.


<p align="center">
  <img src="static/week7_telegram_and_agentic_ai.png" alt="Week 7 Agentic RAG & Telegram Architecture" width="900">
</p>

---

## 🐳 Docker Compose Deployment

Docker Compose orchestrates all services:

* FastAPI application
* OpenSearch + dashboards
* PostgreSQL
* Airflow (scheduler + workers)
* Redis
* Langfuse
* Ollama (local LLM)

This provides:

* Reproducible environments
* Clear service boundaries
* Easy local-to-cloud transition

---

## 📁 Repository Structure

```
MLOPS-ARXIV-RAG-PIPELINE/
├── airflow/                 # Data ingestion & indexing pipeline
├── src/                     # Agentic RAG application code
├── static/                  # Architecture diagrams & assets
├── tests/                   # Automated tests
├── compose.yml              # Multi-service orchestration
├── Dockerfile               # Application container
├── gradio_launcher.py       # UI launcher
├── Makefile                 # Dev & ops commands
├── pyproject.toml           # Python dependencies
├── .env.example             # Environment template
└── README.md                # This file
```

---

## 🎯 Who This Project Is For

* **AI / ML Engineers** building production RAG systems
* **Software Engineers** integrating LLMs into real services
* **Data Scientists** moving from notebooks to systems
* **MLOps practitioners** interested in AI pipelines




## 🚀 Quick Start

### **📋 Prerequisites**
- **Docker Desktop** (with Docker Compose)  
- **Python 3.12+**
- **UV Package Manager** ([Install Guide](https://docs.astral.sh/uv/getting-started/installation/))
- **8GB+ RAM** and **20GB+ free disk space**

### **⚡ Get Started**

```bash
# 1. Clone and setup
git clone <repository-url>
cd arxiv-paper-curator

# 2. Configure environment (IMPORTANT!)
cp .env.example .env
# The .env file contains all necessary configuration for OpenSearch, 
# arXiv API, and service connections. Defaults work out of the box.
# You need to add Jina embeddings free api key and langfuse keys (check the blogs)

# 3. Install dependencies
uv sync

# 4. Start all services
docker compose up --build -d

# 5. Verify everything works
curl http://localhost:8000/health
```


### **📊 Access Your Services**

| Service | URL | Purpose |
|---------|-----|---------|
| **API Documentation** | http://localhost:8000/docs | Interactive API testing |
| **Gradio RAG Interface** | http://localhost:7861 | User-friendly chat interface |
| **Langfuse Dashboard** | http://localhost:3000 | RAG pipeline monitoring & tracing |
| **Airflow Dashboard** | http://localhost:8080 | Workflow management |
| **OpenSearch Dashboards** | http://localhost:5601 | Hybrid search engine UI |


## 📚 Week 1: Infrastructure Foundation ✅

**Start here!** Master the infrastructure that powers modern RAG systems.

### **🎯 Learning Objectives**
- Complete infrastructure setup with Docker Compose
- FastAPI development with automatic documentation and health checks
- PostgreSQL database configuration and management
- OpenSearch hybrid search engine setup
- Ollama local LLM service configuration
- Service orchestration and health monitoring
- Professional development environment with code quality tools




## 🔧 Reference & Development Guide

### **🛠️ Technology Stack**

| Service | Purpose | Status |
|---------|---------|--------|
| **FastAPI** | REST API with automatic docs | ✅ Ready |
| **PostgreSQL 16** | Paper metadata and content storage | ✅ Ready |
| **OpenSearch 2.19** | Hybrid search engine (BM25 + Vector) | ✅ Ready |
| **Apache Airflow 3.0** | Workflow automation | ✅ Ready |
| **Jina AI** | Embedding generation (Week 4) | ✅ Ready |
| **Ollama** | Local LLM serving (Week 5) | ✅ Ready |
| **Redis** | High-performance caching (Week 6) | ✅ Ready |
| **Langfuse** | RAG pipeline observability (Week 6) | 
### **📡 API Endpoints Reference**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Service health check |  
| `/api/v1/papers` | GET | List stored papers | 
| `/api/v1/papers/{id}` | GET | Get specific paper | 
| `/api/v1/search` | POST | BM25 keyword search | 
| `/api/v1/hybrid-search/` | POST | Hybrid search (BM25 + Vector) | **Week 4** |

**API Documentation:** Visit http://localhost:8000/docs for interactive API explorer

### **🔧 Essential Commands**

#### **Using the Makefile** (Recommended)
```bash
# View all available commands
make help

# Quick workflow
make start         # Start all services
make health        # Check all services health
make test          # Run tests
make stop          # Stop services
```

#### **All Available Commands**
| Command | Description |
|---------|-------------|
| `make start` | Start all services |
| `make stop` | Stop all services |
| `make restart` | Restart all services |
| `make status` | Show service status |
| `make logs` | Show service logs |
| `make health` | Check all services health |
| `make setup` | Install Python dependencies |
| `make format` | Format code |
| `make lint` | Lint and type check |
| `make test` | Run tests |
| `make test-cov` | Run tests with coverage |
| `make clean` | Clean up everything |

#### **Direct Commands** (Alternative)
```bash
# If you prefer using commands directly
docker compose up --build -d    # Start services
docker compose ps               # Check status
docker compose logs            # View logs
uv run pytest                 # Run tests
```

