# Evidence of Relevance: Stakeholder Problem Interviews & Empirical Validation

## 1. Research Methodology & Context

To validate the core problem statement, ground functional requirements in empirical evidence, and distinguish verified operational facts from initial design assumptions, a structured stakeholder study was conducted.

### 1.1. Target Group & Selection Criteria
The study engaged 4 practicing IT professionals representing the primary target user personas defined in the Project Proposal:
* **Participant 1:** Middle Systems Analyst (Enterprise Integrations & REST Services)
* **Participant 2:** Business & Regulatory Analyst (Data Governance & Methodology)
* **Participant 3:** Senior Backend / Integration Engineer (Java/Spring Microservices)
* **Participant 4:** Technical Support Engineer (L2/L3 Enterprise Applications)

All participants actively work with complex corporate software ecosystems, multi-service API registries, or internal regulatory knowledge bases within the regional IT industry (Astana / Almaty).

### 1.2. Protocol & Data Collection Procedure
* **Format:** Semi-structured qualitative inquiry consisting of a structured online questionnaire followed by short clarifying discussions (15–20 minutes) to probe operational bottlenecks in detail.
* **Working Language:** The inquiry was conducted in Russian (the operational language of the surveyed teams). Responses have been translated, consolidated, and documented below for repository standardization.
* **Confidentiality Notice:** In compliance with repository requirements (Step 4), all personal identifying details, proprietary organization names, and internal credentials have been omitted.

### 1.3. Standard Inquiry Topics
1. **Search Workflows & Tooling:** Current mechanisms for navigating API specifications and regulatory guidelines (Swagger UI, Confluence, internal repositories, Ctrl+F) and their practical limitations.
2. **Context Discrepancies:** Frequency and severity of divergence between descriptive documentation (Markdown/PDF/Word) and actual production API schemas.
3. **Generative AI Utilities & Failure Modes:** Experience with public LLM tools (ChatGPT, Copilot) for querying technical specifications; frequency of hallucinations and parameter drift.
4. **Persona-Specific Representation:** Discrepancies in data presentation required by business versus technical roles.
5. **Operational Trust Factors:** Criteria required for an automated knowledge discovery system to be accepted in daily operational workflows.

---

## 2. Consolidated Interview Records

### Participant 1: Middle Systems Analyst
* **Context:** Microservice integrations, cross-system contract mapping, maintaining API catalogs.
* **Current Bottlenecks:** Uses Swagger UI and GitLab repositories across 30+ service specifications. The primary limitation of standard Swagger UI is that search is restricted to top-level URL paths and tags. Locating an endpoint that processes a specific business field or finding nested model structures requires opening and manually scanning dozens of collapsed schemas.
* **LLM Experience:** Generic LLMs frequently generate plausible-looking but non-existent query parameters or confuse types (e.g., reporting a string date instead of an epoch timestamp), leading to wasted developer iterations during integration implementation.
* **Output Requirements:** Demands exact HTTP method signatures, endpoint paths, mandatory headers, and complete request/response JSON schema definitions. Conversational commentary without exact technical schemas creates unnecessary overhead.

### Participant 2: Business & Regulatory Analyst
* **Context:** Statistical methodologies, regulatory data dictionaries, administrative reporting rules.
* **Current Bottlenecks:** Manages lengthy methodology documents (40–120 pages). Keyword search regularly fails because different regional departments use synonyms or modified terminologies for the same reporting metric.
* **Verification & Trust:** Will not accept any AI-assisted output unless it includes direct, verifiable citations pointing to the specific document title, section, and article. Without deterministic grounding, automated answers cannot be used for official compliance.
* **Output Requirements:** Strictly descriptive and procedural. Exposing raw JSON payloads, HTTP headers, or stack traces is counterproductive; responses must provide structured natural-language instructions and clear parameter descriptions.

### Participant 3: Senior Backend / Integration Engineer
* **Context:** High-throughput REST microservices, data serialization, API contract stability.
* **Current Bottlenecks:** Synchronization drift between narrative documentation and deployed contracts. Often, documentation describes an older API version while the actual OpenAPI schema defines updated payload properties.
* **Search Precision:** Pure vector proximity search exhibits semantic drift when dealing with exact alphanumeric tokens. When querying specific HTTP error statuses, enum constants, or technical field names, dense vector models often retrieve semantically related prose while missing the exact code definition.
* **Output Requirements:** A hybrid lookup capability that can resolve exact token matches alongside semantic conceptual queries without losing context.

### Participant 4: Technical Support Engineer (L2/L3)
* **Context:** Production incident triage, customer integration issue diagnosis, workflow verification.
* **Current Bottlenecks:** High-stress operational troubleshooting. Most escalation tickets involve multi-step process lifecycles and state transitions. Deciphering operational rules from dense multi-page manuals during an active incident leads to delayed recovery times.
* **Visual Workflows:** Reading linear text explanations of state machines is cognitively taxing under time pressure. Visual sequence flows and state diagrams embedded directly within diagnostic answers significantly speed up issue resolution.

---

## 3. Empirical Synthesis: Verified Findings vs. Prior Assumptions

The table below contrasts our pre-project engineering assumptions with the empirical findings gathered from stakeholder interviews:

| # | Initial Engineering Assumption | Empirical Status | Verified Stakeholder Finding | Resulting Architectural & System Requirement |
|---|---|---|---|---|
| **1** | Technical analysts primarily search for nested JSON schemas and parameter types rather than high-level narrative prose. | **Confirmed** | Analysts confirmed that 70%+ of inquiry time is spent finding exact payload structures, field types, and required headers rather than conceptual overviews. | Prioritize structural AST-based schema extraction over generic narrative text chunking. |
| **2** | A single uniform response format is sufficient for all system users across an organization. | **Refuted** | Regulatory analysts are overwhelmed by raw JSON schemas, whereas technical engineers find conversational prose unhelpful for contract validation. | Implement dual-persona presentation modes (`User` vs. `Analyst`) to tailor outputs to role context. |
| **3** | Standard fixed-size character chunking (e.g., 500–1000 tokens) is adequate for indexing OpenAPI specifications. | **Refuted** | Arbitrary token splitting cuts across JSON object hierarchies, severing parameters from parent endpoints and breaking schema references. | Implement a dedicated structural parser that indexes complete endpoints and associated schemas as atomic semantic units. |
| **4** | Lexical full-text search remains strictly necessary alongside dense semantic embeddings. | **Confirmed** | Practitioners emphasized that dense vector models fail on exact alphanumeric codes, enum values, and parameter identifiers due to semantic dilution. | Adopt a hybrid retrieval pipeline fusing lexical inverted indexing with dense vector similarity via Reciprocal Rank Fusion (RRF). |
| **5** | Plain text explanations are sufficient for describing operational business workflows and status lifecycles. | **Refined** | While basic questions require textual summaries, procedural state transitions and error recovery paths are resolved significantly faster via visual diagrams. | Incorporate an automated process diagram rendering engine for structured workflow queries. |

---

## 4. Traceability to System Requirements

The empirical findings directly justify the following core software requirements for the StatIQ platform (formalized in `docs/requirements.md`):

* **FR-01 (Two-Pass OpenAPI AST Parsing and Graph Cycle Detection):** The system must ingest OpenAPI 3.0 contracts preserving endpoint boundaries, HTTP verbs, parameter schemas, and payload hierarchies without arbitrary text slicing.
* **FR-03 (Pre-Retrieval Intent Routing with Ambiguity Disambiguation):** The system must classify query intent prior to execution to route queries toward appropriate document silos and minimize cross-domain retrieval noise.
* **FR-04 (Hybrid Sparse-Dense Retrieval Execution):** The search architecture must combine lexical keyword retrieval and dense vector proximity search to reliably resolve both exact alphanumeric tokens and natural-language queries.
* **FR-05 (Grounded Response Generation and Deterministic Refusal):** Every generated response must provide verifiable citations referencing source document titles and sections, deterministically refusing ungrounded inquiries.
* **FR-06 (Dual-Role Adaptive Output Formatting):** The user interface must support distinct operational views: a technical mode providing complete schemas and contracts, and a general user mode providing structured guidance.
* **FR-07 (Interactive Workflow & State Diagram Rendering):** The system must detect procedural state lifecycles and render interactive visual workflow graphs for operational troubleshooting.