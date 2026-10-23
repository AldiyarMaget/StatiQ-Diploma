# Project Proposal: StatIQ Knowledge Hub

## 1. Project Title
Design and Implementation of a Scalable RAG Knowledge Discovery System with Hybrid Semantic Routing for Enterprise Documentation and API Services

## 2. Team Members
* **Aldiyar Khassenov** Lead Software Engineer (Architecture, Backend Pipeline, Data Storage, Web Client)
*(Individual Project)*

## 3. Supervisor
* **Islam Omirzak** (Senior Lecturer / PhD Candidate, M.Sc.Ed in Computer Science)

---

## 4. Problem Statement
In modern distributed enterprise architectures, critical operational knowledge is divided between two disparate environments:
1. **Enterprise Metadata Catalogs:** Semi-structured narrative regulations, methodology guidelines, and business indicator dictionaries.
2. **Core Data Platforms & Microservice Registries:** Highly structured, nested OpenAPI/Swagger REST contracts and API schemas.

Standard corporate search mechanisms and generic Large Language Model (LLM) interfaces fail in this environment due to three major limitations:
* **Information Fragmentation:** Analysts and developers must manually cross-reference business documentation with technical API contracts located in isolated storage silos.
* **Domain Hallucinations & Parameter Drift:** Generic LLMs lack deterministic domain ground-truth, frequently generating non-existent API parameters, incorrect HTTP endpoints, and hallucinated payload structures.
* **Persona Misalignment:** Existing search tools lack role adaptability; business users are overwhelmed by low-level technical JSON payloads, while engineers struggle to extract exact schema contracts from generic descriptive text.

---

## 5. Target Users
* **System & Integration Analysts:** Require deterministic, rapid access to OpenAPI endpoint definitions, JSON request/response schemas, and integration parameters without manual inspection of production codebases.
* **Methodology Specialists:** Require verified citations of regulatory documentation, business definitions, and operational workflow guidelines.
* **Core Operational Engineers & Support Staff:** Require clear procedural instructions and structured schema visualizers for rapid issue diagnosis.

---

## 6. Aim
To design, implement, and validate **StatIQ** a scalable, enterprise-grade Retrieval-Augmented Generation (RAG) knowledge discovery system featuring hybrid sparse-dense search, intent-based semantic routing, and a role-adaptive web interface for unified exploration of enterprise documentation and API services.

---

## 7. Objectives
1. **Stakeholder Validation:** Conduct structured interviews with 3–5 industry practitioners to formalize domain search pain points and distinguish verified operational requirements from assumptions.
2. **Hierarchical Ingestion Pipeline:** Implement an ingestion engine capable of parsing, chunking, and indexing both Markdown methodology guidelines and structured OpenAPI specifications while preserving schema hierarchies.
3. **Hybrid Search Architecture:** Develop a hybrid retrieval service combining lexical full-text indexing (PostgreSQL `tsvector`) and dense vector proximity search (`pgvector` with HNSW indexing) fused via Reciprocal Rank Fusion (RRF).
4. **Semantic Query Routing:** Design a lightweight routing component to classify user intent and direct queries to relevant data silos, minimizing contextual noise and retrieval latency.
5. **Role-Adaptive Web Client:** Construct a responsive Web GUI featuring role-differentiated views (User vs. Analyst), streaming response visualization, and rendering of architectural diagrams and interface workflows.
6. **Empirical Benchmarking & Evaluation:** Evaluate the system against a standard Dense RAG baseline across retrieval quality metrics (MRR, Recall@K) and execution latency.

---

## 8. Expected Outcomes
* **Server-Side Core:** A high-throughput backend service (Java 21 / Spring Boot 3.2.5) orchestrating document parsing, vector indexing, and streaming LLM integration.
* **Unified Knowledge Store:** A relational database (PostgreSQL 16 with `pgvector`) ensuring transactional consistency across document chunks, relational metadata, and high-dimensional embeddings.
* **Web Client Application:** A web client application (React / Vite + TypeScript) providing dual-role interaction modes, real-time diagram rendering, and streaming output.
* **Evaluation Report:** Documented benchmark results verifying retrieval accuracy improvements over baseline retrieval methods.

---

## 9. Project Scope

### In-Scope (MVP Deliverables)
* Automated parsing and indexing of Markdown documentation and OpenAPI 3.0 (JSON/YAML) files.
* PostgreSQL with `pgvector` for unified relational and vector storage.
* Hybrid search algorithm (PostgreSQL full-text search + dense vector retrieval with Reciprocal Rank Fusion).
* Heuristic-based semantic routing for query classification.
* Web client interface supporting role switching (User vs. Analyst) and rendering of technical diagrams.
* Production deployment configuration using containerized environments (Docker Compose).

### Out-of-Scope (Excluded from Current Stage)
* Processing audio, video, or scanned non-searchable image archives.
* Multi-region distributed database clustering.
* Direct write-back/mutation operations into external enterprise platforms.