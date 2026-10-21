# Verification and Evaluation Plan: Empirical Benchmarking and Quality Assurance

## 1. Introduction & Verification Objectives

The primary aim of the **StatIQ platform** is to design, implement, and validate a scalable enterprise Retrieval-Augmented Generation (RAG) system with localized semantic routing, mitigating domain hallucinations and bridging the gap between narrative corporate regulations and machine-readable OpenAPI 3.0 service contracts.

In accordance with the pre-defense evaluation rubric (*"A testing plan, outcome metrics, measurement conditions, and a baseline solution for comparison; how the achievement of the project aim will be demonstrated"*), this document establishes:
1. **Multi-Tier Testing Strategy:** Unit, integration, and scenario regression suites aligned strictly with the system's modular Spring Boot architecture.
2. **Quantitative Outcome Metrics:** Rigorous Information Retrieval (IR) metrics, generation fidelity measures, and latency service level agreements (SLAs).
3. **Standardized Measurement Conditions:** Reproducible execution environment, reference enterprise testbeds, and concurrency profiles.
4. **Baseline Architecture for Empirical Comparison:** Formulation of the canonical Naive Dense RAG baseline against which StatIQ's performance gains are objectively demonstrated.

---

## 2. Multi-Tier Testing Strategy

The quality assurance pipeline is organized into four complementary verification tiers reflecting the actual backend subsystem topology:

```
+-----------------------------------------------------------------------------------+
| Tier 4: Empirical Benchmark Suite (IR Metrics: MRR, Recall@K, Latency vs Baseline)|
+-----------------------------------------------------------------------------------+
| Tier 3: End-to-End Contract & Scenario Tests (ChatController, Dual Persona Mode)  |
+-----------------------------------------------------------------------------------+
| Tier 2: Subsystem Integration Tests (RetrievalService, PgVector, Ingestion Runner)|
+-----------------------------------------------------------------------------------+
| Tier 1: Algorithmic Unit Tests (SwaggerSplitter, LocalIntentRouter, PostProcessor)|
+-----------------------------------------------------------------------------------+
```

### 2.1. Tier 1: Algorithmic Unit Tests
Focuses on in-memory, deterministic verification of core transformation components without external database or network I/O:
* **`SwaggerSplitterTest`:** Validates structural decomposition of OpenAPI specifications:
  * Verifies atomic chunk creation per individual HTTP operation (`Method + Path`).
  * Asserts extraction of operation identifiers, summaries, path tags, and parameter arrays (`name`, `in`, `required`, `description`).
  * Validates preservation of JSON structure boundaries, preventing arbitrary cross-chunk slicing of API definitions.
* **`LocalIntentRouterTest` / `RouterTest`:** Validates JVM in-memory query classification:
  * Measures vector cosine similarity against domain anchor clusters (`API_REQUEST`, `ROLES_AND_ACCESS`, `PROCESS_GUIDELINE`, `METHODOLOGY_REGULATION`).
  * Validates deterministic Keyword Guard heuristics (e.g., matching HTTP methods, specific status codes, or role keywords).
  * Validates fallback resolution when incoming prompts contain ambiguous cross-domain terms.
* **`MermaidPostProcessorTest`:** Validates process diagram extraction and sanitization:
  * Tests regex-based syntactic healing: node bracket normalization `([\\]\\}])\\s*([A-Za-z])`, unescaped quote repair, and HTML tag sanitization.
  * Verifies structural integrity of `graph TD` and `sequenceDiagram` code blocks extracted from LLM markdown.

### 2.2. Tier 2: Subsystem Integration Tests
Focuses on data persistence, vector indexing, and pipeline orchestration:
* **`RetrievalServiceIT`:**
  * Validates dense vector similarity lookup against PostgreSQL 16 `pgvector` with HNSW indexing using the in-process E5-Small-v2 embedding model.
  * Asserts multi-stage heuristic re-ranking: verification of domain keyword density bonuses and Swagger Penalization (suppressing technical API payloads when queries target general methodology regulations).
  * Validates heuristic context injection for critical reference documents (`ROLES.md`, `USER_GUIDE.md`).
* **`DocumentIndexerIT`:**
  * Tests automated scanning and parsing of repository directories (`./documents`) upon service startup (`ApplicationRunner`).
  * Asserts recursive document chunking parameters (verifying chunk size boundaries and metadata attribute tagging).

### 2.3. Tier 3: End-to-End REST Contract & Scenario Tests
Validates the complete HTTP request and response cycle via Spring Boot `MockMvc`:
* **`ChatControllerScenarioTest`:**
  * Validates role-adaptive output generation:
    * In `User` mode: verifies step-by-step guidance and suppression of raw JSON schema code blocks.
    * In `Analyst` mode: verifies inclusion of endpoint signatures, parameter lists, and structured schema specifications.
  * Verifies provenance and citation attribution: confirms factual responses append verified source markers (`📎 **Источник:** [файл]`).
  * Validates out-of-domain query handling: verifies the system transparently identifies ungrounded questions and appends the standard domain disclaimer.

### 2.4. Tier 4: Empirical Benchmark Execution
Automated benchmark harness evaluating retrieval quality, ranking accuracy, and execution latency across standardized query suites.

---

## 3. Quantitative Outcome Metrics

To objectively validate the achievement of the project aim, the platform is benchmarked across three quantitative dimensions:

### 3.1. Information Retrieval (IR) Metrics
Quantifies the search engine's ability to locate and prioritize authoritative document chunks:

1. **Recall@K ($K \in \{3, 5\}$):** The proportion of evaluation queries where the ground-truth document chunk is retrieved within the top $K$ positions:
   $$\text{Recall@K} = \frac{1}{|Q|} \sum_{q=1}^{|Q|} \mathbb{I}(\text{rank}_q \le K)$$
   *Target SLA:* $\text{Recall@3} \ge 90\%$, $\text{Recall@5} \ge 95\%$ on enterprise testbeds.

2. **Mean Reciprocal Rank (MRR):** Evaluates the ranking precision of the first relevant document chunk across all test queries:
   $$\text{MRR} = \frac{1}{|Q|} \sum_{q=1}^{|Q|} \frac{1}{\text{rank}_q}$$
   Where $\text{rank}_q$ is the position of the first ground-truth document segment.
   *Target SLA:* $\text{MRR} \ge 0.85$.

### 3.2. Generation Quality & Grounding Metrics
Quantifies LLM output fidelity and grounding adherence:

1. **Parameter Accuracy Rate ($A_p$):** Percentage of generated API calls where HTTP methods, route paths, and parameter names perfectly match the underlying OpenAPI contract:
   *Target SLA:* $A_p \ge 95\%$ (parameter drift and hallucination rate $\le 5\%$).

2. **Source Citation Completeness ($C_{\text{src}}$):** Proportion of factual responses containing verified file references:
   *Target SLA:* $C_{\text{src}} = 100\%$ across grounded corpus inquiries.

3. **Domain Grounding Transparency ($T_{\text{ood}}$):** Successful identification rate of out-of-domain questions with transparent contextual boundaries:
   *Target SLA:* $T_{\text{ood}} \ge 95\%$.

### 3.3. Performance & Latency SLAs
Measured on the reference compute environment under sustained load:

| Pipeline Stage | Metric Description | Target Threshold (p95) |
|---|---|---|
| **Semantic Intent Classification** | On-device ONNX embedding + anchor cosine calculation | $\le 25 \text{ ms}$ |
| **Database Retrieval & Re-ranking** | `pgvector` HNSW lookup + heuristic domain re-scoring | $\le 35 \text{ ms}$ |
| **Total Pre-Generation Latency** | Full retrieval and prompt assembly pipeline ($C=10$) | $\le 95 \text{ ms}$ |
| **End-to-End Interactive Roundtrip** | Full response generation including external LLM API | $\le 3000 \text{ ms}$ |

---

## 4. Measurement Conditions & Standardized Testbed

To guarantee experimental reproducibility, all empirical measurements are conducted under strictly controlled hardware, software, and data conditions:

### 4.1. Hardware & Runtime Reference Environment
* **Compute Infrastructure:** Dedicated virtualized compute node:
  * **Processor:** 4 dedicated vCPUs (x86_64 architecture, 2.8 GHz baseline clock, AVX2 enabled).
  * **Memory:** 8 GB RAM (JVM heap configured with `-Xms2g -Xmx4g`).
  * **Storage:** High-throughput NVMe SSD storage.
* **Operating System & Runtime:** Linux Ubuntu 22.04 LTS, OpenJDK 21 (Temurin), Docker Engine 26+.
* **Database Engine:** PostgreSQL 16.3 with `pgvector` 0.7.0 extension running in an isolated Docker container with calibrated shared memory buffers (`shared_buffers = 1GB`, `work_mem = 64MB`).

### 4.2. Ingested Evaluation Corpus
The evaluation corpus mirrors enterprise data heterogeneity:
1. **Machine-Readable API Contracts:** OpenAPI 3.0 / Swagger specifications comprising 85 distinct HTTP operations and associated parameter structures.
2. **Semi-Structured Regulatory Documents:** Corporate regulatory manuals, procedural guidelines, and role matrices totaling over 140 pages of Markdown documentation.

### 4.3. Standardized Evaluation Query Suite ($N = 250$)
The benchmark suite comprises 250 curated queries with human-annotated ground-truth targets:
* **Exact Alphanumeric & Code Queries ($N = 50$):** Inquiries containing specific HTTP status codes (e.g., `404`, `503`, internal lifecycle status `104`), parameter IDs, or technical routing keys.
* **API Schema & Contract Queries ($N = 75$):** Inquiries requesting payload structures, mandatory fields, or endpoint routes.
* **Business & Regulatory Guidelines ($N = 75$):** Conceptual natural-language inquiries regarding multi-step operational rules and definitions.
* **Out-of-Domain / Negative Prompts ($N = 50$):** Unrelated or fabricated questions designed to evaluate hallucination resistance and transparent refusal boundaries.

### 4.4. Concurrency & Load Profiling Protocol
Performance measurements are orchestrated via automated load test runners:
* **Warm-up Phase:** 50 unmetered requests to ensure JVM JIT optimization and database buffer cache population.
* **Sustained Load Phase:** 500 consecutive requests executed under a sustained concurrency level of 10 virtual clients ($C = 10$).

---

## 5. Baseline Solution for Empirical Comparison

To substantiate the architectural advantages of StatIQ, the system is benchmarked against the canonical baseline architecture established in literature (Lewis et al., 2020; Gao et al., 2023):

### 5.1. Baseline Architecture Specification: Naive Dense RAG
* **Ingestion Strategy:** Standard naive sliding-window text chunking (fixed window of 512 tokens with 50-token overlap). No structural OpenAPI parsing; JSON/YAML contracts are split linearly as flat text.
* **Storage & Indexing:** Dedicated dense vector index (`pgvector` HNSW index on 384D E5 embeddings) without lexical keyword density modifiers.
* **Routing Strategy:** Pure global lookup. No pre-retrieval intent classification; all user queries are executed against a single unified vector index.
* **Retrieval Logic:** Standard approximate nearest neighbor search ($k$-NN) using pure cosine distance, retrieving the top $K=5$ closest vectors.

### 5.2. Systematic Comparative Hypotheses

The empirical benchmark is structured to validate four formal performance hypotheses:

| Evaluation Dimension | Naive Dense RAG Baseline (Lewis et al., 2020) | StatIQ Target Platform (Proposed) | Target Demonstration Hypothesis |
|---|---|---|---|
| **Alphanumeric Code Retrieval** | Fails on exact status codes and parameter IDs due to semantic vector dilution. | Multi-stage retrieval with keyword density scoring guarantees exact token retrieval. | StatIQ achieves $\ge 25\%$ improvement in Recall@3 on exact code queries. |
| **OpenAPI Contract Integrity** | Slices through JSON brackets, detaching parameters from endpoints. | `SwaggerSplitter` indexes complete operations with parameter context intact. | Parameter error rate drops from $>15\%$ (Baseline) to $<5\%$ (StatIQ). |
| **Context Pollution in Prompts** | Unfiltered global search injects irrelevant API code into regulatory prompts. | `LocalIntentRouter` isolates domain silos prior to database search. | $\ge 20\%$ increase in Mean Reciprocal Rank (MRR) due to noise elimination. |
| **Role-Adaptive Formatting** | Employs a uniform verbose response format for all user categories. | `PromptBuilder` dynamically adapts output style (`User` vs. `Analyst`). | $100\%$ role-compliant response presentation across evaluation scenarios. |

---

## 6. How the Achievement of the Project Aim Will Be Demonstrated

The final validation milestone demonstrates the achievement of the diploma project aim through the following documented verification artifacts:

1. **Automated Test Suite Execution:** Verified pass rate across JUnit unit and integration tests (`mvn clean test`), confirming functional correctness of `SwaggerSplitter`, `LocalIntentRouter`, and `RetrievalService`.
2. **Empirical Evaluation Benchmark Report:** A consolidated analytical benchmark table reporting measured MRR, Recall@K, and parameter error rates across the 250-query suite for both StatIQ and the Naive Dense RAG baseline.
3. **Reproducibility Manifest:** Provision of test suites, evaluation query datasets, and configuration manifests allowing independent replication of all benchmark figures on the Reference Benchmark Environment.

