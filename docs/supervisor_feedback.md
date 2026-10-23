# Academic Supervisor Feedback and Continuous Revision Log

## 1. Official Supervisor Review & Endorsement (Pre-Defense 1)

In accordance with course regulations for Component 11 (*Supervisor Feedback*), the project deliverables for the Pre-Defense 1 milestone were reviewed by the academic supervisor:

* **Project Title:** StatIQ: Scalable RAG Knowledge Discovery System with Hybrid Semantic Routing for Enterprise Documentation and API Services
* **Student Researcher / Lead Software Engineer:** Aldiyar Khassenov
* **Academic Supervisor:** Islam Omirzak (Senior Lecturer / PhD Candidate, M.Sc.Ed in Computer Science)
* **Milestone:** Pre-Defense 1 Baseline (`pre-defense-1`)

### 1.1. Formal Evaluation Statement from the Supervisor
> *"The student has demonstrated satisfactory progress in the analysis and design stage of the diploma project. The key deliverables include a clearly formulated problem statement, project scope, stakeholder and literature analyses, software requirements specification, architectural models, technology trade-off analysis, UX scenarios, project roadmap, risk register, and verification plan. The proposed architecture appropriately combines hierarchical OpenAPI and Markdown ingestion, hybrid sparse–dense retrieval, semantic query routing, a role-adaptive web interface, and containerized deployment. These deliverables establish a coherent and traceable foundation for the implementation of the StatIQ system."*
> 
> — **Islam Omirzak**, Academic Supervisor

---

## 2. Review of Substance: Iterative Feedback & Decisions Matrix

To satisfy the evaluation criterion (*"Whether the supervisor reviewed the substance of the work and whether that review led to meaningful improvements"*), the table below records the substantive comments delivered across consultative reviews, the architectural decisions formulated, and the concrete revisions implemented in the repository deliverables:

| # | Review Area | Supervisor's Substantive Feedback | Decision on How to Address the Comment | Meaningful Improvements & Deliverables Impact |
|---|---|---|---|---|
| **1** | **Scope Definition & Bounding (STATIQ-1, STATIQ-6)** | Warned against open-ended agentic execution (such as live write-back API mutations or unbounded Tool Calling) within a bachelor's thesis timeline. Emphasized focusing on deterministic RAG retrieval, schema grounding, and parameter extraction. | Exclude write-back API transactions and optical character recognition (OCR) from MVP scope. Formally assign them to the Out-of-Scope exclusions category (Won't Have) in the project SRS. | Bounded project scope in `docs/proposal.md` and explicitly defined non-goals in Section 1.2 of `docs/requirements.md`. |
| **2** | **Evaluation Rigor & Baseline (STATIQ-9)** | Pointed out that retrieval and generation quality cannot be demonstrated purely qualitatively. Required establishing a formal, reproducible baseline from literature alongside standardized Information Retrieval (IR) metrics. | Adopt the canonical Naive Dense RAG architecture (Lewis et al., 2020) as an explicit comparative baseline. Define formal mathematical formulations for MRR and Recall@K, alongside quantitative SLA targets for parameter hallucination rates. | Created `docs/verification_plan.md` featuring 250 evaluation queries, hardware reference specifications, and hypothesis benchmarks. |
| **3** | **OpenAPI Ingestion Integrity (STATIQ-7, STATIQ-8)** | Highlighted that naive sliding-window text chunking destroys nested `$ref` schema definitions and parameter semantics in OpenAPI specifications, leading to parameter drift during generation. | Design a specialized AST-aware parser (`SwaggerSplitter`) that isolates HTTP operations (`Method + Path`) and preserves structured parameter dictionaries without arbitrary token slicing. | Formulated requirement FR-01 in `docs/requirements.md` and integrated the component into C4 Level 2 architecture in `docs/architecture.md`. |
| **4** | **Project Traceability & Git Hygiene (STATIQ-10)** | Mandated strict adherence to progressive version control: prohibited bulk code/document uploads ("dumps") prior to deadlines and required bidirectional linking between YouTrack tasks and Git Pull Requests. | Establish a feature-branch workflow (`STATIQ-{ID}-{name}`), enforce standardized commit prefixes (`STATIQ-{ID}: ...`), and merge all deliverables exclusively via reviewed Pull Requests into `main`. | Formulated the governance policy in `docs/project_management.md`, achieved 100% PR traceability across all 11 milestone tasks. |
| **5** | **Dual-Role Interaction Model (STATIQ-8)** | Recommended clarifying how different organizational roles consume system outputs to avoid presenting raw JSON payloads to business users or vague summaries to integration engineers. | Implement deterministic prompt adaptation (`USER_MODE` vs. `ANALYST_MODE`) within `PromptBuilder` and design client-side Mermaid state diagram rendering. | Authored end-to-end interaction scenarios and synchronous REST API mockups (with SSE streaming defined as a target architectural extension) in `docs/prototype_ux.md`. |

---

## 3. Summary of Traceable Improvements

The consultative supervision protocol directly contributed to the quality of the Pre-Defense 1 baseline:
1. **Academic Feasibility:** Eliminated unfeasible feature bloat, resulting in a realistic, defendable scope for an individual software engineering thesis.
2. **Methodological Validity:** Shifted evaluation from subjective impressions to reproducible quantitative metrics ($MRR \ge 0.85$, $Recall@3 \ge 90\%$).
3. **Engineering Integrity:** Ensured architectural alignment between requirements, structural diagrams, and verified Spring Boot / pgvector components.
4. **Governance Traceability:** All feedback items are linked to corresponding YouTrack issues, commits, and pull requests in the repository.

