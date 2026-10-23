# Software Requirements Specification (SRS): StatIQ Knowledge Discovery System

## 1. Introduction & Scope Definition

### 1.1. Purpose
This document specifies the complete functional and non-functional requirements for the **StatIQ platform** — an enterprise-grade Retrieval-Augmented Generation (RAG) system with hybrid semantic routing designed for heterogeneous corporate regulations and OpenAPI 3.0 service contracts.

### 1.2. System Scope & Boundaries
* **In-Scope (MVP Deliverables):**
  * Automated ingestion and AST parsing of OpenAPI 3.0 (JSON/YAML) and Markdown documentation.
  * Unified PostgreSQL 16 + pgvector storage co-locating relational metadata, lexical tsvectors, and vector embeddings.
  * Pre-retrieval semantic intent routing with keyword overrides.
  * Hybrid sparse-dense retrieval fused via Reciprocal Rank Fusion (RRF).
  * Dual-persona presentation (`User` vs. `Analyst`).
  * Automated Mermaid process diagram validation and rendering.
  * Containerized deployment configuration using Docker Compose.
* **Out-of-Scope (System Boundaries & Exclusions):**
  * Processing non-searchable scanned PDFs, OCR layout reconstruction, or audio/video media streams.
  * Multi-region distributed database clustering or cross-datacenter replication.
  * Direct write-back or mutating transaction execution into external production APIs (Tool Calling / Agentic Mutations); the platform functions strictly as a read-only knowledge discovery engine.

### 1.3. Prioritization Framework (MoSCoW)
Requirements are prioritized using the MoSCoW framework:
* **Must Have (M):** Mandatory baseline requirements verified at pre-defense and milestone reviews.
* **Should Have (S):** Critical architectural capabilities required for enterprise reliability.
* **Could Have (C):** Non-critical operational enhancements implemented if milestone schedule permits.
* **Won't Have (W):** Explicitly excluded out-of-scope capabilities (non-goals defined in Section 1.2: write-back API mutations, deep OCR layout reconstruction, and multi-region database clustering) deferred beyond the diploma project scope.

---

## 2. User Roles & Operating Personas

1. **General Operational User (`User` Mode):** Business methodology specialists, registry operators, and operational support staff requiring structured procedural guidance, step-by-step instructions, business definitions, and visual workflow graphs. Raw JSON schemas and transport headers are abstracted away.
2. **Technical Integration Analyst (`Analyst` Mode):** System analysts, backend engineers, and enterprise integration specialists requiring exact REST endpoint signatures, path parameters, data types, nested request/response JSON schemas, and error code mappings.

---

## 3. Functional Requirements (FR)

### FR-01: Two-Pass OpenAPI AST Parsing and Graph Cycle Detection
* **Priority:** Must Have (M)
* **Description:** The system shall ingest OpenAPI 3.0 / Swagger 2.0 specifications (JSON/YAML) and decompose them via an Abstract Syntax Tree (AST) parser into self-contained semantic units using a memory-bounded two-pass streaming pipeline.
* **Functional Scope:**
  1. **Pass 1 (Streaming Schema Indexing):** The parser shall stream through `components/schemas` without building full-tree DOM heap models, indexing raw schema definitions into an in-memory or database lookup registry.
  2. **Pass 2 (Operation Extraction):** The parser shall stream through `paths`, isolating each individual HTTP operation (Method + Path) as an atomic unit.
  3. **Cycle-Safe Schema Resolution:** The parser shall recursively resolve `$ref` pointers by tracking the active traversal URI stack. If a `$ref` pointer target already exists in the current traversal path (`visited_refs.contains(pointer)`), it shall be immediately identified as a graph cycle and terminated with `{"$ref_cycle": "path/to/target"}`. Non-cyclical nested structures shall be resolved up to a maximum depth of 10.
* **Verification & Acceptance Criteria:**
  * **Test Case FR-01.1:** Ingest an enterprise OpenAPI 3.0 file containing recursive data models (e.g., self-referencing category trees) and linear models nested 7 levels deep.
  * **Pass Condition:** 100% of endpoints are parsed into atomic units; zero `StackOverflowError` exceptions occur; recursive cycles are terminated with `{"$ref_cycle": ...}`; linear models nested up to 10 levels deep are fully preserved without premature truncation.

### FR-02: Two-Tier Structural Markdown Ingestion
* **Priority:** Must Have (M)
* **Description:** The system shall ingest semi-structured corporate guidelines and manuals formatted in Markdown, bounding chunk sizes to the embedding model's vocabulary limit.
* **Functional Scope:**
  1. The primary parser shall segment documents along structural header boundaries (`#`, `##`, `###`) to preserve thematic cohesion.
  2. Token lengths shall be evaluated strictly using the underlying embedding model's tokenizer (BERT WordPiece / `E5-Small-v2` vocabulary).
  3. If a section bounded by headers exceeds 450 WordPiece tokens (guaranteeing adherence to the 512-token model ceiling alongside the `passage: ` prefix), the parser shall apply secondary recursive window chunking (target window: 400 tokens, overlap: 50 tokens).
  4. All child chunks shall inherit the complete breadcrumb header metadata hierarchy (e.g., `Document > Chapter 2 > Section 2.1`).
* **Verification & Acceptance Criteria:**
  * **Test Case FR-02.1:** Ingest a 50+ page regulatory document containing sections exceeding 2000 WordPiece tokens.
  * **Pass Condition:** Zero generated chunks exceed 450 WordPiece tokens; 100% of chunks retain full breadcrumb path metadata in their relational metadata payload.

### FR-03: Pre-Retrieval Intent Routing with Ambiguity Disambiguation
* **Priority:** Must Have (M)
* **Description:** The system shall classify incoming user queries into discrete domain intents prior to database query execution to isolate target document silos.
* **Functional Scope:**
  1. The router shall categorize queries into: `API_REQUEST`, `ROLES_AND_ACCESS`, `PROCESS_GUIDELINE`, or `METHODOLOGY_REGULATION`.
  2. Classification shall be performed via vector cosine similarity against pre-computed domain anchor embeddings in JVM memory, combined with deterministic keyword guards.
  3. **Collision Disambiguation Rule:** If a query contains conflicting keyword triggers across multiple domains (e.g., querying access roles for a specific API endpoint), the keyword guard shall not force a static override; instead, routing shall automatically defer to semantic vector cosine proximity over domain anchor clusters.
* **Verification & Acceptance Criteria:**
  * **Test Case FR-03.1:** Execute an automated benchmark against a curated test suite of $N = 200$ balanced queries (50 per domain intent, including 20 multi-keyword boundary queries).
  * **Pass Condition:** Intent classification accuracy achieves $\ge 90\%$ ($p < 0.05$ with statistical power $\beta = 0.8$) without external LLM calls.

### FR-04: Hybrid Sparse-Dense Retrieval Execution
* **Priority:** Must Have (M)
* **Description:** The system shall execute parallel lexical inverted search and dense vector proximity search, merging candidates using Reciprocal Rank Fusion (RRF).
* **Functional Scope:**
  1. Lexical retrieval shall execute PostgreSQL full-text search (`tsvector` with inverted GIN indexing).
  2. Dense retrieval shall execute HNSW approximate nearest neighbor search on 384-dimensional embeddings (`pgvector`).
  3. Candidates shall be fused via Reciprocal Rank Fusion ($RRF(d) = \sum \frac{1}{60 + r(d)}$), applying domain penalties to suppress out-of-intent silos.
* **Verification & Acceptance Criteria:**
  * **Test Case FR-04.1:** Query the system using a curated synthetic dataset containing 50 unique alphanumeric tokens (exact error codes, parameter keys, endpoint verbs) assigned uniquely to single parent documents.
  * **Pass Condition:** The target document containing the unique token appears in Top-3 retrieved candidates in 100% of test runs. Across broader queries with repeated tokens, Mean Reciprocal Rank satisfies $MRR \ge 0.85$.

### FR-05: Grounded Response Generation and Deterministic Refusal
* **Priority:** Must Have (M)
* **Description:** The generative engine shall synthesize responses strictly from retrieved context chunks and deterministically refuse out-of-domain queries.
* **Functional Scope:**
  1. When retrieved context does not contain sufficient factual evidence, the model shall return a standardized machine-readable refusal payload: `[INSUFFICIENT_CONTEXT]`.
  2. Every generative response shall include source citations referencing the exact source filename, section title, and endpoint path.
* **Verification & Acceptance Criteria:**
  * **Test Case FR-05.1:** Submit 100 out-of-domain evaluation prompts under standardized generation parameters (`temperature = 0.0`, fixed random seed).
  * **Pass Condition:** The system outputs the `[INSUFFICIENT_CONTEXT]` refusal token in $\ge 95\%$ of test runs, demonstrating absence of external hallucinations.

### FR-06: Dual-Role Adaptive Output Formatting
* **Priority:** Must Have (M)
* **Description:** The system shall adapt prompt construction and output formatting based on the active user role (`User` vs. `Analyst`).
* **Functional Scope:**
  1. In `User` mode, responses shall present natural-language summaries, operational steps, and visual diagrams. Raw JSON schema payload trees and transport headers shall be strictly omitted.
  2. In `Analyst` mode, responses shall include HTTP methods, path URLs, parameter types, JSON schema definitions, and response codes.
* **Verification & Acceptance Criteria:**
  * **Test Case FR-06.1:** Submit 30 identical technical inquiries across both `User` and `Analyst` modes.
  * **Pass Condition:** Zero `User` mode responses contain valid multi-line JSON payload objects (verified via AST JSON parser validation); 100% of `Analyst` mode responses contain valid schema blocks.

### FR-07: Interactive Workflow & State Diagram Rendering
* **Priority:** Should Have (S)
* **Description:** The system shall detect procedural state lifecycles and render interactive visual workflow graphs.
* **Functional Scope:**
  1. The LLM pipeline shall output Mermaid.js syntax (`graph TD` or `sequenceDiagram`) for multi-step lifecycles.
  2. A server-side post-processing layer shall validate Mermaid syntax validity before delivery to the client.
* **Verification & Acceptance Criteria:**
  * **Test Case FR-07.1:** Query a multi-state processing lifecycle.
  * **Pass Condition:** The frontend client parses and renders an interactive SVG graph without syntax compilation errors.

### FR-08: Multi-Provider LLM Fallback Orchestration
* **Priority:** Should Have (S)
* **Description:** The system shall maintain high availability across external generative LLM APIs through an automated fallback and retry pipeline.
* **Functional Scope:**
  1. Primary inference requests shall target Google Gemini Flash.
  2. If the primary provider returns HTTP 429, 503, or times out after 6000 ms, the system shall execute an exponential backoff retry loop and failover to Anthropic Claude.
  3. During fallback transitions, the backend shall transmit Server-Sent Events (SSE) keep-alive heartbeat comments to prevent reverse-proxy timeout disconnections.
* **Verification & Acceptance Criteria:**
  * **Test Case FR-08.1:** Simulate a forced HTTP 429 response on the primary LLM client during an active streaming request.
  * **Pass Condition:** The fallback provider is invoked, and the complete response streams to the client without reverse-proxy dropouts.

### FR-09: Semantic Query Result Caching
* **Priority:** Could Have (C)
* **Description:** The system may cache high-confidence query responses in Redis or local cache based on query vector similarity ($\ge 0.96$) to reduce LLM token expenditure.
* **Verification Criteria:** Verified reduction in repeated query execution latency to $< 10 \text{ ms}$.

### FR-10: Automated Synthetic Evaluation Query Generation
* **Priority:** Could Have (C)
* **Description:** The system may incorporate automated synthetic question generation over ingested chunks using offline LLM prompts to expand evaluation testbeds.
* **Verification Criteria:** Generation of at least 100 synthetically grounded QA pairs matching document citations.

---

## 4. Non-Functional Requirements (NFR)

### NFR-01: Retrieval Latency SLA under Sustained Concurrency
* **Priority:** Must Have (M)
* **Description:** The internal retrieval and context assembly pipeline shall maintain strict latency boundaries on a defined reference hardware profile under concurrent load.
* **Reference Benchmark Environment:**
  * CPU: 4 dedicated vCPUs (x86_64 architecture, AVX2 instruction set enabled).
  * Memory: 8 GB RAM.
  * Storage: NVMe SSD.
* **Quantifiable Latency Thresholds (p95 under 10 concurrent clients, post-50 warm-up requests):**
  * Pre-retrieval intent classification: $t \le 10 \text{ ms}$.
  * In-process query vectorization (ONNX E5-Small-v2, 384D): $t \le 20 \text{ ms}$.
  * Hybrid database search execution (PostgreSQL RRF): $t \le 25 \text{ ms}$.
  * Total pre-generation retrieval latency: $t_{\text{total}} \le 55 \text{ ms}$.
* **Verification Method:** Automated Gatling load benchmark measuring internal telemetry across 500 consecutive test queries with 10 concurrent virtual users following a 50-request warm-up phase.

### NFR-02: Data Co-Location & ACID Transactional Consistency
* **Priority:** Must Have (M)
* **Description:** Document chunks, relational access matrices, inverted text indexes, and HNSW vector embeddings shall reside within a single ACID-compliant database boundary.
* **Quantifiable Metric:**
  * Zero dual-write synchronization drift between document metadata and vector embeddings ($0\%$ inconsistency across crash restarts).
* **Verification Method:** Automated integration test injecting document updates during simulated kill-9 database process termination; verified via SQL integrity checks.

### NFR-03: Hardware Compute Footprint & Two-Pass Memory Bounding
* **Priority:** Must Have (M)
* **Description:** The backend service and in-process vectorization shall execute on CPU-only infrastructure within bounded memory limits.
* **Quantifiable Metrics:**
  * Continuous document ingestion peak heap allocation: $\le 1500 \text{ MB}$ RAM.
  * Idle JVM memory allocation: $\le 512 \text{ MB}$ RAM.
  * Ingestion pipeline shall utilize Jackson two-pass streaming parsing to prevent full-DOM heap exhaustion during multi-megabyte specification ingestion.
* **Verification Method:** Memory profiling using VisualVM / Micrometer telemetry under continuous indexing of a 50,000-line OpenAPI specification.

### NFR-04: Concurrency & Virtual Thread Pinning Prevention
* **Priority:** Must Have (M)
* **Description:** The system shall leverage Java 21 Virtual Threads (Project Loom) for high-concurrency request routing while preventing carrier thread pinning during JNI native inference calls.
* **Quantifiable Metrics:**
  * JNI ONNX runtime calls shall be strictly isolated inside a dedicated `FixedThreadPool` sized to physical CPU cores.
  * Zero pinned carrier thread events recorded during continuous vectorization load.
* **Verification Method:** Continuous execution profiling using JDK Flight Recorder (JFR) monitoring the `jdk.VirtualThreadPinned` event, and execution with JVM argument `-Djdk.tracePinnedThreads=full`, confirming zero pinned thread events during load testing.

### NFR-05: Enterprise Security & Secret Sanitization
* **Priority:** Must Have (M)
* **Description:** Proprietary enterprise documentation, API tokens, and internal credentials shall be protected against unauthorized disclosure.
* **Quantifiable Metrics:**
  * Zero external transmission of proprietary corporate credentials or private API keys.
  * 100% of sensitive configuration parameters injected strictly via encrypted environment variables; zero hardcoded secrets in version control history.
  * Embeddings computed entirely in-process; all external LLM communications enforced via TLS 1.3.
* **Verification Method:** Automated static application security testing (SAST) and git-secrets commit history scanning.

### NFR-06: OpenAPI Specification Compatibility
* **Priority:** Should Have (S)
* **Description:** The API ingestion parser shall achieve syntax compliance with the official OpenAPI Specification Version 3.0.3.
* **Quantifiable Metric:**
  * $100\%$ parsing success across valid Swagger 2.0 and OpenAPI 3.0.x contracts in JSON and YAML formats.
* **Verification Method:** Execution of the OpenAPI Initiative test suite through `SwaggerSplitter`.

---

## 5. Requirements Traceability Matrix (RTM)

| Proposal Objective | Stakeholder Finding (`interviews.md`) | Derived Requirement | Primary Verification Method |
|---|---|---|---|
| **Objective 1 (Validation)** | Analysts confirm manual contract cross-referencing takes 30–60 min; exact schemas required. | **FR-01, FR-06** | Inspection of AST parser output and dual-role UI modes. |
| **Objective 2 (Ingestion)** | Naive character chunking breaks JSON schema nesting and parameter context. | **FR-01, FR-02, NFR-06** | Bounded two-pass AST ingestion with graph cycle detection and WordPiece chunking. |
| **Objective 3 (Hybrid Search)** | Pure vector search fails on exact numeric status codes and parameter IDs. | **FR-04, NFR-01** | Hybrid RRF retrieval benchmarks on unique token dataset ($MRR \ge 0.85$). |
| **Objective 4 (Semantic Routing)** | Cross-domain documentation causes context contamination and parameter drift. | **FR-03, FR-05** | Intent classification benchmark ($N=200$, accuracy $\ge 90\%$) and refusal test. |
| **Objective 5 (Web Client & UX)** | Operators require visual state graphs; methodologists reject raw JSON schemas. | **FR-06, FR-07** | Dual-mode schema validation via JSON AST parser and Mermaid rendering verification. |
| **Objective 6 (Evaluation)** | Lack of empirical baseline comparison undermines architectural credibility. | **NFR-01, NFR-03** | Gatling performance test under 10 concurrent clients (latency p95 $\le 55$ ms). |

---

## 6. Verification Protocol for Mandatory (Must Have) Requirements

| Requirement ID | Verification Procedure | Pass / Fail Acceptance Criteria |
|---|---|---|
| **FR-01** | Automated JUnit Test | $100\%$ endpoints parsed; cyclic references terminated with `{"$ref_cycle": ...}`; linear nesting up to 10 preserved; zero `StackOverflowError`. |
| **FR-02** | Automated Ingestion Test | Zero chunks $>450$ WordPiece tokens; 100% child chunks retain complete breadcrumb header metadata. |
| **FR-03** | Automated Intent Suite | $\ge 90\%$ classification accuracy across $N = 200$ balanced test prompts; ambiguous collisions resolved via vector cosine similarity. |
| **FR-04** | Automated Retrieval Test | $100\%$ Top-3 recall on unique token testbed; $MRR \ge 0.85$ across ambiguous multi-match queries. |
| **FR-05** | Automated Refusal Test | $\ge 95\%$ generation of `[INSUFFICIENT_CONTEXT]` on 100 out-of-domain prompts (`temp = 0.0`). |
| **FR-06** | Automated Schema Test | Zero multi-line JSON payload objects in `User` mode via JSON AST parser; 100% valid schema blocks in `Analyst` mode across 30 prompts. |
| **NFR-01** | Gatling Load Test | Pre-generation retrieval latency $t_{\text{total}} \le 55\text{ ms}$ (p95) under 10 concurrent clients following 50 warm-up requests. |
| **NFR-02** | Crash Injection Test | Zero data drift or orphaned vector records after simulated process kill during writes. |
| **NFR-03** | Memory Profiling | Heap allocation $\le 1500\text{ MB}$ during two-pass streaming parsing of 50,000-line OpenAPI contract. |
| **NFR-04** | JFR & JVM Trace Audit | Zero `jdk.VirtualThreadPinned` events in JFR and zero pinned traces under `-Djdk.tracePinnedThreads=full`. |
| **NFR-05** | Security & Secret Scan | 100% TLS 1.3 transport; zero secret keys detected by static analysis tools. |