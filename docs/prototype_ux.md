# User Experience (UX) Design, Interactive Scenarios, and Prototype Walkthrough

## 1. Introduction & Interaction Model Rationale

The **StatIQ platform** bridges the operational divide between two distinct enterprise user groups:
1. **General Operational Users & Methodology Specialists (`User` Mode):** Require procedural clarity, natural-language guidance, process lifecycle visualizers, and direct source citations while abstracting raw transport protocols and technical JSON payload structures.
2. **System & Integration Analysts (`Analyst` Mode):** Require deterministic schema fidelity, exact OpenAPI route signatures, parameters, request body models, response status codes, and code snippets.

To satisfy the pre-defense evaluation rubric (*"Whether it is clear how users or external systems will interact with the solution"*), this document specifies the dual-role graphical user interface (GUI) wireframes, step-by-step user interaction scenarios, interactive Mermaid diagram visualizers, and the programmatic REST API contracts for headless and external system integration.

---

## 2. Web Client Architecture & Layout Specification

The web application (React, Vite, TypeScript) provides a responsive, single-page interface integrated with the core Spring Boot backend service.

### 2.1. Unified Interface Layout Wireframe

```
+----------------------------------------------------------------------------------------------------+
|  StatIQ | Enterprise Knowledge Discovery                    [Role Toggle: (•) User  ( ) Analyst ]  |
+-----------------------------------+----------------------------------------------------------------+
|  KNOWLEDGE BASE CORPUS            |  ACTIVE CONVERSATION & DISCOVERY THREAD                        |
|                                   |                                                                |
|  [Ingested Sources]               |  [User]: How do I submit an annual balance reporting package,  |
|  * core-regulations.md            |          and what happens if validation status 104 is returned?|
|  * metadata-dictionary.md         |                                                                |
|  * meta-swagger.json              |  [StatIQ Assistant] [Mode: User] [Intent: PROCESS_GUIDELINE]   |
|                                   |                                                                |
|  [Active Service Status]          |  To submit the annual balance reporting package, follow this   |
|  ✓ Core Service: Ready            |  procedural sequence:                                          |
|  ✓ Indexed Segments: Active       |  1. Access the Registry Submission Portal.                     |
|                                   |  2. Upload your verified ledger XML document.                  |
|  [Session Controls]               |  3. Monitor state transitions: Stage 010 -> Validation.        |
|  + New Discovery Session          |                                                                |
|  🗑 Clear Context Cache           |  Process Lifecycle Flowchart:                                  |
|                                   |  +----------------------------------------------------------+  |
|                                   |  |  [Draft 010] ---> [Validation] ---> [Approved 020]        |  |
|                                   |  |                        |                                 |  |
|                                   |  |                        +-----> [Error 104: Resubmit]     |  |
|                                   |  +----------------------------------------------------------+  |
|                                   |                                                                |
|                                   |  If Status 104 is returned, the package failed schema checks.  |
|                                   |  Correct the balance indicator in accordance with Article 4.2. |
|                                   |                                                                |
|                                   |  Citations & Grounding:                                        |
|                                   |  [📎 Source: core-regulations.md # Section 4.2]                 |
+-----------------------------------+----------------------------------------------------------------+
|  [Ask about API schemas, endpoints, or regulatory guidelines...]              [Submit Query (Enter)]|
+----------------------------------------------------------------------------------------------------+
```

---

## 3. End-to-End Key User Scenarios

### 3.1. Scenario 1: Operational User Querying Procedural Workflow (`User` Mode)

* **Actor:** Operational Specialist / Business Methodologist.
* **Active Mode:** `User` Mode (Default).
* **User Goal:** Understand a process lifecycle and error remediation steps without seeing technical JSON structures.
* **Execution Walkthrough:**
  1. The user selects **User** mode via the top navigation toggle.
  2. The user enters a natural-language query: *"What are the required steps to complete an annual balance submission, and how do I resolve error status 104?"*
  3. The client dispatches an HTTP POST request to `/api/chat` with `{ mode: "user" }`.
  4. `LocalIntentRouter` routes the query to `PROCESS_GUIDELINE`.
  5. The assistant returns a step-by-step narrative explanation followed by an automatically validated Mermaid state machine diagram.
  6. Direct citation tags are appended at the bottom: `Source: core-regulations.md # Section 4.2`.
  7. **Role Verification:** All technical OpenAPI schemas, HTTP headers, and cURL snippets are strictly suppressed.

#### Client Presentation Mockup (`User` Mode Output):

```
+----------------------------------------------------------------------------------------------------+
| StatIQ Response [Mode: User]                                                                       |
+----------------------------------------------------------------------------------------------------+
| To successfully submit an annual balance reporting package, execute the following steps:          |
|                                                                                                    |
| 1. Prepare Primary Record: Ensure company identification and balance aggregates match the ledger.  |
| 2. Initiate Package Upload: Transmit the file through the reporting gateway.                      |
| 3. State Progression: The package automatically transitions from Created (010) to Processing.      |
|                                                                                                    |
| Workflow State Lifecycle:                                                                          |
| ```mermaid                                                                                         |
| stateDiagram-v2                                                                                    |
|     [*] --> Status_010_Draft                                                                       |
|     Status_010_Draft --> Status_020_Processing : Submit Package                                    |
|     Status_020_Processing --> Status_030_Approved : Validation Successful                         |
|     Status_020_Processing --> Status_104_SchemaError : Attribute Validation Failure                |
|     Status_104_SchemaError --> Status_010_Draft : Rectify Ledger & Re-upload                       |
| ```                                                                                                |
|                                                                                                    |
| Remediation for Status 104:                                                                        |
| Error 104 indicates field boundary validation errors. Verify that all monetary indicators are       |
| non-negative and formatted to two decimal places before re-submitting.                             |
|                                                                                                    |
| 📎 Source: core-regulations.md (Section: "Annual Reporting & Verification Cycles")                 |
+----------------------------------------------------------------------------------------------------+
```

---

## 3.2. Scenario 2: Technical Integration Analyst Querying API Contracts (`Analyst` Mode)

* **Actor:** System Analyst / Integration Backend Engineer.
* **Active Mode:** `Analyst` Mode.
* **User Goal:** Extract the exact OpenAPI 3.0 REST endpoint, HTTP method, required query/path parameters, and JSON request/response schema without searching raw repository code.
* **Execution Walkthrough:**
  1. The analyst switches the navigation toggle to **Analyst** mode.
  2. The analyst enters the query: *"What endpoint submits the package, and what is the exact JSON body schema for package creation?"*
  3. The request is dispatched with `{ mode: "analyst" }`.
  4. `LocalIntentRouter` classifies the intent as `API_REQUEST`.
  5. `RetrievalService` retrieves the atomic `SwaggerSplitter` chunk for `POST /api/v1/packages/submit`.
  6. The LLM synthesizes a technical contract summary, detailing HTTP verbs, parameter schemas, and payload examples.

#### Client Presentation Mockup (`Analyst` Mode Output):

```
+----------------------------------------------------------------------------------------------------+
| StatIQ Response [Mode: Analyst]                                                                    |
+----------------------------------------------------------------------------------------------------+
| Endpoint Signature:                                                                                |
| POST /api/v1/packages/submit                                                                       |
| Content-Type: application/json                                                                     |
| Authorization: Bearer <JWT_TOKEN>                                                                  |
|                                                                                                    |
| Header Parameters:                                                                                 |
| - X-Idempotency-Key (string, required): Unique UUID v4 transaction token.                          |
|                                                                                                    |
| Request Body Payload Schema (ApplicationPackageDTO):                                               |
| ```json                                                                                            |
| {                                                                                                  |
|   "organizationId": "string (UUID, required)",                                                     |
|   "reportingPeriod": "integer (YYYYMM, e.g., 202610, required)",                                   |
|   "packageType": "string (enum: ['ANNUAL_BALANCE', 'QUARTERLY_TAX'], required)",                   |
|   "indicators": [                                                                                  |
|     {                                                                                              |
|       "indicatorCode": "string (pattern: '^[A-Z]{3}_[0-9]{3}$', required)",                        |
|       "metricValue": "number (double, precision: 2, required)",                                    |
|       "currency": "string (ISO-4217, default: 'KZT')"                                              |
|     }                                                                                              |
|   ]                                                                                                |
| }                                                                                                  |
| ```                                                                                                |
|                                                                                                    |
| Response Status Codes:                                                                             |
| - 201 Created: Package accepted for asynchronous validation. Returns `{"packageId": "UUID"}`.      |
| - 400 Bad Request: Schema validation failure (returns code `104` with detail list).                |
| - 401 Unauthorized: Expired or invalid Bearer token.                                               |
|                                                                                                    |
| 📎 Source: meta-swagger.json (Path: /api/v1/packages/submit [POST])                                |
+----------------------------------------------------------------------------------------------------+
```

---

## 4. Programmatic REST API Contract for External Systems

External development environments, automated test runners, and client applications interact with StatIQ through standardized REST API endpoints.

### 4.1. Conversational Query & Discovery Endpoint (`POST /api/chat`)

Processes natural language queries and returns role-adapted responses with grounded citations.

* **URL:** `/api/chat`
* **Method:** `POST`
* **Request Headers:**
  * `Content-Type: application/json`
  * `Accept: application/json`

#### Request Payload:
```json
{
  "question": "What is the endpoint for uploading reporting packages and its schema?",
  "mode": "analyst",
  "messages": [
    {
      "role": "user",
      "content": "Hello, I am integrating the submission service."
    },
    {
      "role": "assistant",
      "content": "Hello. How can I assist with your integration?"
    }
  ]
}
```

*Note: The backend controller accepts both `"messages"` and `"history"` interchangeably for cross-platform compatibility.*

#### Field Specifications:
* `question` (string, mandatory): The natural language query.
* `mode` (string, mandatory): Active persona view. Values: `"user"` | `"analyst"`.
* `messages` / `history` (array of objects, optional): Prior conversational turns for contextual disambiguation.

#### Baseline Response Format (`200 OK` - Synchronous JSON):
```json
{
  "answer": "### Endpoint Signature\n`POST /api/v1/packages/submit`\n\n```json\n{\n  \"organizationId\": \"string (UUID, required)\",\n  \"packageType\": \"string (enum)\"\n}\n```\n\n📎 **Source:** [meta-swagger.json]"
}
```

*Note on Streaming Architecture (Target Extension): The baseline MVP delivers complete answers synchronously to ensure atomic validation of Mermaid diagram syntax via `MermaidPostProcessor`. An incremental Server-Sent Events (SSE) streaming extension is planned as an architectural deliverable for subsequent pre-defense stages.*

---

### 4.2. Corpus Management & Ingestion Mechanism

To guarantee predictable memory utilization and avoid runtime heap exhaustion during multi-megabyte specification parsing, document ingestion in the baseline system operates via batch initialization:

* **Ingestion Runner:** On application startup, `DocumentIndexer` (`ApplicationRunner`) automatically scans the designated documents repository directory (`./documents`).
* **Ingestion Pipelines:**
  * **OpenAPI Contracts:** Handled by `SwaggerSplitter` — parses paths, methods, request/response models, and resolves `$ref` schemas.
  * **Methodology Documents:** Handled by `DocumentIndexer` — segments Markdown sections and preserves breadcrumb header context.
* **Dynamic Pipeline Contract (Target Extension):** A multipart HTTP endpoint (`POST /api/ingest`) is specified for future CI/CD runner integration to trigger on-demand index updates without service restarts.

---

### 4.3. Service Health & Ingestion Monitoring Endpoints

Clients and container orchestration tools (Docker Compose) monitor system readiness via built-in endpoints:

* **Health Endpoint:** `GET /api/health`
  * **Response (`200 OK`):** `{"status": "UP"}`
* **Ingestion Status Endpoint:** `GET /api/status`
  * **Response (`200 OK`):**
    ```json
    {
      "status": "ready",
      "documentsLoaded": 12,
      "segmentsIndexed": 148
    }
    ```

---

## 5. UI Prototype Verification & Scenario Validation

The interactive prototype demonstrates the realization of the system requirements:

1. **Deterministic Role Adaptation (FR-06):** Switching between `User` and `Analyst` modes triggers immediate prompt adaptation in `PromptBuilder`, eliminating JSON noise for operational users while ensuring complete schema visibility for technical staff.
2. **Dynamic Workflow Rendering (FR-07):** Generative responses describing multi-state lifecycles compile into SVG state flowcharts via client-side Mermaid.js, backed by syntax verification in `MermaidPostProcessor`.
3. **Traceable Attribution (FR-05):** Every conversational turn surfaces source citations linking directly to the underlying documentation source.
4. **Reliable Response Delivery:** Atomic response delivery ensures complete Mermaid graph syntax integrity prior to client-side visualization.

