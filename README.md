# StatIQ: Scalable RAG Knowledge Discovery System

> **Design and Implementation of a Scalable RAG Knowledge Discovery System with Hybrid Semantic Routing for Enterprise Documentation and API Services**

[![YouTrack Project](https://img.shields.io/badge/YouTrack-Project%20STATIQ-blue?logo=youtrack)](https://aldiyar.youtrack.cloud/projects/STATIQ)
[![Java](https://img.shields.io/badge/Java-21-orange?logo=openjdk)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2.5-brightgreen?logo=springboot)](https://spring.io/projects/spring-boot)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16%20%2B%20pgvector-blue?logo=postgresql)](https://github.com/pgvector/pgvector)
[![React](https://img.shields.io/badge/Frontend-React%20%2F%20Vite%20%2B%20TS-61DAFB?logo=react)](https://react.dev/)

---

## 📌 Project Overview

**StatIQ** is an enterprise-grade Retrieval-Augmented Generation (RAG) knowledge discovery system designed to unify access across heterogeneous corporate data environments: semi-structured regulatory and methodological guidelines, and machine-readable OpenAPI/Swagger REST contracts.

The solution incorporates hybrid sparse-dense retrieval (Reciprocal Rank Fusion) and lightweight semantic query routing to eliminate contextual noise, parameter drift, and hallucinations in generative language models.

---

## 👥 Project Team & Supervision

* **Lead Software Engineer:** Aldiyar Khassenov (Architecture, Backend Pipeline, Vector Storage, Web Client) — *Individual Project*
* **Academic Supervisor:** Islam Omirzak (Senior Lecturer / PhD Candidate, M.Sc.Ed in Computer Science)
* **Institution:** Astana IT University (AITU), School of Software Engineering
* **Current Stage / Status:** Pre-defense 1 preparation (Analysis, architectural design, requirements specification, and traceability baseline)

---

## 🎯 Problem Statement

In modern distributed enterprise architectures, operational knowledge is divided into two disconnected environments:
1. **Enterprise Metadata Catalogs:** Semi-structured narrative regulations, methodology guidelines, and business indicator dictionaries.
2. **Core Data Platforms & Microservice Registries:** Highly structured, nested OpenAPI/Swagger REST contracts and API schemas.

Standard enterprise search tools and generic LLM interfaces fail due to:
* **Information Fragmentation:** Analysts and developers must manually cross-reference business documentation with technical API contracts located in isolated data silos.
* **Domain Hallucinations & Parameter Drift:** Generic LLMs lack deterministic domain grounding, frequently inventing non-existent API parameters, incorrect HTTP endpoints, and hallucinated payload structures.
* **Persona Misalignment:** Conventional search tools lack role adaptability; business users are overwhelmed by low-level technical JSON payloads, while engineers struggle to extract exact schema contracts from generic descriptive prose.

---

## 🎯 Target Users

* **System & Integration Analysts:** Require deterministic, rapid access to OpenAPI endpoint definitions, JSON request/response schemas, and integration parameters without manual inspection of raw specifications.
* **Methodology Specialists:** Require verified citations of regulatory documentation, business definitions, and operational workflow guidelines.
* **Core Operational Engineers & Support Staff:** Require clear procedural instructions, process state transition workflows, and structured schema visualizers for rapid issue diagnosis.

---

## 🚀 Aim & Objectives

**Aim:** To design, implement, and validate a scalable RAG knowledge discovery system featuring hybrid sparse-dense search, intent-based semantic routing, and a role-adaptive web interface for unified exploration of enterprise documentation and API services.

**Key Objectives:**
1. **Stakeholder Validation:** Conduct structured interviews with 3–5 industry practitioners to formalize domain search pain points and distinguish verified operational requirements from assumptions.
2. **Hierarchical Ingestion Pipeline:** Implement an ingestion engine capable of parsing, chunking, and indexing both Markdown methodology guidelines and structured OpenAPI specifications while preserving schema hierarchies.
3. **Hybrid Search Architecture:** Develop a hybrid retrieval service combining lexical full-text indexing (PostgreSQL tsvector) and dense vector proximity search (pgvector with HNSW indexing) fused via Reciprocal Rank Fusion (RRF).
4. **Semantic Query Routing:** Design a lightweight routing component to classify user intent and direct queries to relevant data silos, minimizing contextual noise and retrieval latency.
5. **Role-Adaptive Web Client:** Construct a responsive Web GUI featuring role-differentiated views (User vs. Analyst), streaming response visualization, and rendering of architectural diagrams and interface workflows.
6. **Empirical Benchmarking & Evaluation:** Evaluate the system against a standard Dense RAG baseline across retrieval quality metrics (MRR, Recall@K) and execution latency.

---

## 📦 Expected Outcomes & Scope

### In-Scope (MVP Deliverables):
* **Server-Side Core:** High-throughput backend service (Java 21 / Spring Boot 3.2.5) orchestrating document parsing, vector indexing, and streaming LLM integration.
* **Unified Knowledge Store:** Relational database (PostgreSQL 16 with pgvector) ensuring transactional consistency across document chunks, relational metadata, and high-dimensional embeddings.
* **Hierarchical Contract Parser:** Specialized chunker for OpenAPI 3.0 (JSON/YAML) and Markdown documentation.
* **Web Client Application:** Responsive frontend (React / Vite + TypeScript) providing dual-role interaction modes (User vs. Analyst) and real-time diagram rendering.
* **Deployment:** Containerized deployment configuration using Docker Compose.

### Out-of-Scope (Excluded from Current Stage):
* Processing non-searchable scanned images, audio, or video streams.
* Multi-region distributed database clustering.
* Direct write-back or mutating transaction execution into external enterprise systems.

---

## 🛠 Technology Stack

* **Backend:** Java 21, Spring Boot 3.2.5, LangChain4j
* **Database & Vector Search:** PostgreSQL 16, pgvector (HNSW Indexing)
* **In-Process Embeddings:** ONNX Runtime (`langchain4j-embeddings-e5-small-v2-q`, 384 dimensions)
* **LLM Integration:** Google GenAI SDK (Gemini Flash), Anthropic Messages API fallback
* **Frontend:** React, Vite, TypeScript
* **DevOps & Tooling:** Docker Compose, Apache Maven, Git

---

## 🔗 Project Management & Documentation

### Task Tracking & Workflow
* **Issue Tracker (YouTrack):** [StatIQ Project Board](https://aldiyar.youtrack.cloud/projects/STATIQ) *(Tasks: [Issues List](https://aldiyar.youtrack.cloud/issues/STATIQ))*

### Project Documentation (`/docs`)
* **Project Initiation & Scope:**
  * [Project Proposal (Markdown)](docs/proposal.md)
  * [Project Proposal (DOCX)](docs/project_proposal.docx)
* **Domain & Empirical Research:**
  * [Stakeholder Problem Interviews & Empirical Validation](docs/interviews.md)
  * [Literature Review: RAG, Semantic Routing & API Ingestion](docs/literature_review.md)
* **Architectural & Comparative Analysis:**
  * [Architectural Description and Structural Models](docs/architecture.md)
  * [Existing System Comparison & Gap Analysis](docs/system_comparison.md)
  * [Architectural Trade-Off Analysis & Technology Comparison](docs/tech_comparison.md)
* **Specifications & Verification:**
  * [Software Requirements Specification (SRS) & Verification Protocol](docs/requirements.md)

### Architectural Diagrams (`/diagrams`)
* **System Context (C4 Level 1):** [PNG](diagrams/system_context.png) | [Source (Mermaid)](diagrams/sources/system_context.mmd)
* **Component Architecture (C4 Level 2):** [PNG](diagrams/component_diagram.png) | [Source (Mermaid)](diagrams/sources/component_diagram.mmd)
* **Execution Flow:** [PNG](diagrams/execution_flow.png) | [Source (Mermaid)](diagrams/sources/execution_flow.mmd)
* **Use Case Model:** [PNG](diagrams/use_case.png) | [Source (Mermaid)](diagrams/sources/use_case.mmd)