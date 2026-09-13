# AI Asset Register, Data Classification Matrix & Level-0 Data Flow Diagram

The founding asset inventory for Cortexa AI Inc.'s AI Security Programme — the single source of truth for what models, datasets, and APIs the company actually runs, built as Milestone 1's first deliverable before any threat modelling or risk-register work can begin.

## About this repository

This repository contains a single-file governance package covering:

- A **25-asset AI Asset Register** spanning CorteXa Copilot, CorteXa Search, CorteXa Agents, and shared infrastructure (model registry, CI/CD, vector databases, IAM, security telemetry)
- A **four-tier Data Classification Matrix** (Public / Internal / Confidential / Restricted) that every asset in the register cross-references
- A **Level-0 data flow diagram** tracing data from user query through each AI product to storage and back out, with every trust-boundary crossing marked
- Residency, retention, and privacy-obligation notes for every Confidential or Restricted asset

**Fictional scenario:** Cortexa AI Inc. is a fictional San Francisco-based enterprise AI SaaS company (~650 employees, $85M ARR) serving Fortune 500 customers in healthcare, financial services, and government. This is the first project under Cortexa's newly signed AI Security Programme Charter — commissioned by Data Science Lead **Priya Sharma**, who flagged that no one at Cortexa had ever produced a single source of truth for the company's models, datasets, and API integrations.

## Frameworks referenced

- ISO/IEC 42001:2023 Clause 8.3 — operational planning and control (resource identification for AI systems)
- GDPR Article 30 — record of processing activities (the register doubles as Cortexa's Article 30 record for its AI systems)
- EU AI Act — referenced for data-governance and record-keeping obligations relevant to Confidential/Restricted assets

> **Disclaimer:** Cortexa AI Inc., Priya Sharma, Rachel Donovan, and all named individuals are fictional, created for a governance-documentation exercise. The asset inventory, classifications, and data flows are illustrative work product, not a real company's actual technical environment.

---

# AI Asset Register & Data Classification Matrix

## Cortexa AI Inc. — Milestone 1, Project 2

| Field | Detail |
|---|---|
| Prepared by | AI Security Engineer |
| For sign-off by | Priya Sharma, Data Science Lead |
| Programme | AI Security Programme (per signed Charter v1.0) |
| Scope | CorteXa Copilot, CorteXa Search, CorteXa Agents, and shared infrastructure |
| Assets registered | 25 (target: minimum 20, minimum 5 per product) |
| Status | Ready for review and sign-off |

## Why This Comes First

Every subsequent Milestone 1 activity — the AI threat landscape mapping and the ISO/IEC 42001 risk register — depends on this asset register existing first. ISO/IEC 42001:2023 Clause 8.3 requires an organization to identify and document the resources an AI system depends on as part of operational planning and control; this register is exactly that documented inventory. It also functions as Cortexa's GDPR Article 30 record of processing activities for its AI systems, since it captures data types, purposes, and retention for every asset that touches personal or customer data.

An "AI asset" here is deliberately broader than the model file: it includes training/fine-tuning datasets, embeddings, system prompts, vector indexes, inference endpoints, model registry entries, and every third-party API integration that touches the pipeline.

## Task 1 & 2 — AI Asset Register (with Sensitivity Classification)

Every asset below carries a Sensitivity Classification that references a tier defined in the Data Classification Matrix (Section further down), so the two documents are internally consistent as required.

### CorteXa Copilot

| Asset ID | Asset Name | Asset Type | Description | Classification | Justification |
|---|---|---|---|---|---|
| AST-001 | Copilot — Primary LLM (OpenAI GPT-4o API) | Third-Party Model / API | Primary foundation model Copilot calls for chat completion and reasoning. | Confidential | Processes user prompts and conversation context that may include customer-confidential business text; disclosure would breach customer trust even without regulated PII. |
| AST-002 | Copilot — Secondary LLM (Azure OpenAI GPT-4o) | Third-Party Model / API | Azure-hosted deployment used for EU/regulated customers requiring data-residency guarantees. | Confidential | Same conversational data sensitivity as AST-001, routed specifically for EU residency commitments. |
| AST-003 | Copilot — Anthropic Claude (failover model) | Third-Party Model / API | Secondary LLM provider used for failover and customer-specific routing. | Confidential | Processes the same class of conversational data as the primary and secondary LLM providers. |
| AST-004 | Copilot — System Prompt Library | Configuration / Prompt Asset | Master and customer-specific system prompts defining Copilot's behaviour and guardrails. | Confidential | Embeds proprietary guardrail and behavioural design; disclosure would expose competitive IP and safety-control logic. |
| AST-005 | Copilot — Conversation Logs Store | Data Store (Log Storage) | Stores full conversation transcripts for support, QA, and abuse monitoring. | Restricted | May contain customer PII, financial, or health information depending on the customer vertical — the highest-sensitivity Copilot asset. |
| AST-006 | Copilot — Customer Integration API Layer | API / Inference Endpoint | REST/streaming API customers integrate against. | Confidential | Carries API keys and customer request payloads in transit. |

### CorteXa Search

| Asset ID | Asset Name | Asset Type | Description | Classification | Justification |
|---|---|---|---|---|---|
| AST-007 | Search — Production Vector Index (Pinecone) | Data Store (Vector DB) | Pinecone production index storing embeddings of customers' proprietary enterprise knowledge bases. | Confidential | Contains embeddings of customers' proprietary content; disclosure would breach customer contracts even though the content is not regulated personal data. |
| AST-008 | Search — Development Vector Index (Chroma) | Data Store (Vector DB) | Local/dev Chroma index for testing retrieval quality. | Internal | Restricted to synthetic/sanitized test data by policy; no live customer data permitted. |
| AST-009 | Search — Embedding Model | Model | Generates vector embeddings from indexed source documents. | Internal | Stateless processing; no data persisted by the model itself. |
| AST-010 | Search — Indexed Customer Knowledge Bases | Data Store (Source Documents) | Customers' proprietary source documents ingested and indexed by Search. | Restricted | Full text of customer documents may include PII, financial data, or health data depending on customer vertical. |
| AST-011 | Search — Retrieval API | API / Inference Endpoint | Accepts a query, retrieves relevant chunks, returns ranked/generated results. | Confidential | Carries query text and retrieved document snippets in transit. |
| AST-012 | Search — Query & Retrieval Logs | Data Store (Log Storage) | Logs queries and retrieved document references for quality monitoring. | Confidential | Contains query text and tenant identifiers, but not full document content. |

### CorteXa Agents

| Asset ID | Asset Name | Asset Type | Description | Classification | Justification |
|---|---|---|---|---|---|
| AST-013 | Agents — Orchestration Framework | Application / Framework | LangChain-based orchestration layer that plans and executes multi-step workflows. | Internal | Processes workflow state transiently; no persistent sensitive data in the framework itself. |
| AST-014 | Agents — Underlying LLM (GPT-4o / Claude) | Third-Party Model / API | Foundation model used for agent reasoning and planning. | Confidential | Processes task prompts and tool outputs that may reference customer data. |
| AST-015 | Agents — Tool / Plugin Integration Registry | Configuration / Integration Registry | Registry of approved tools an agent may invoke, with scoped credentials. | Restricted | Holds scoped API credentials/tokens for connected third-party systems — a high-impact target if exposed. |
| AST-016 | Agents — Action / Execution Logs | Data Store (Log Storage) | Immutable log of every action an agent proposed and executed. | Restricted | Action parameters may include customer PII, financial transaction details, or health data depending on the connected tool. |
| AST-017 | Agents — Human Approval / Oversight Interface | Application (Human Oversight Tool) | Interface for human review/approval of high-impact agent actions. | Confidential | Carries pending action details and reviewer decisions tied to Restricted action data. |
| AST-018 | Agents — Customer System Integration Credentials | Secrets / Credential Store | OAuth tokens/API keys allowing Agents to act within a customer's own systems. | Restricted | The single highest-impact asset if compromised — direct write access into a customer's live systems. |

### Shared Infrastructure

| Asset ID | Asset Name | Asset Type | Description | Classification | Justification |
|---|---|---|---|---|---|
| AST-019 | Shared — MLflow Model Registry | Model Registry | Central registry tracking all trained/fine-tuned model versions and lineage. | Confidential | Model metadata and lineage represent proprietary IP and a map of Cortexa's ML capability. |
| AST-020 | Shared — Model Artifact Storage (S3) | Data Store (Object Storage) | S3 buckets storing serialized model weights referenced by MLflow. | Confidential | Model weights may encode patterns learned from training data (model-inversion risk) and represent core IP. |
| AST-021 | Shared — Training / Fine-Tuning Dataset Store | Data Store (Training Data) | Curated datasets used to fine-tune models across all three products. | Restricted | May include customer-derived examples and historical conversation samples with potential PII if not fully anonymized. |
| AST-022 | Shared — CI/CD Pipeline (GitHub Actions / Docker / Kubernetes) | Infrastructure / Pipeline | Build, test, and deployment pipeline for all model and application code. | Internal | Carries source code and deployment config; no customer data expected, but a compromise vector for everything downstream. |
| AST-023 | Shared — Cloud Identity & Access Management (IAM) | Security Control / Identity System | AWS/Azure/GCP IAM governing access to all AI assets. | Restricted | Governs access to every other asset in this register; compromise here compromises everything else. |
| AST-024 | Shared — Security Telemetry / SIEM (Wazuh) | Security Tooling | Aggregates AI platform telemetry for security monitoring. | Confidential | Telemetry may reference user/tenant identifiers even though it is not itself customer content. |
| AST-025 | Shared — Published Developer API Documentation | Documentation | Public-facing API reference for Copilot and Search integration. | Public | Deliberately published for external developers; contains no customer or proprietary data. |

**Coverage check:** 6 Copilot assets, 6 Search assets, 6 Agents assets, 7 shared-infrastructure assets = **25 total**, exceeding the 20-asset minimum and the 5-per-product minimum, and explicitly covering the model registry, CI/CD pipeline, and both vector databases named in the brief.

## Task 2 (continued) — Data Classification Matrix

Cortexa's four-tier sensitivity scheme, referenced by every row in the register above.

| Classification Tier | Definition | Example Asset Types | Handling Requirements | Access Restrictions | Typical Regulatory Relevance |
|---|---|---|---|---|---|
| **Public** | Information approved for unrestricted external release; no confidentiality expectation. | Marketing content, published API documentation | No special handling required. | None — publicly accessible. | None typically applicable. |
| **Internal** | Information for use within Cortexa only; not for external release. | Internal architecture diagrams, non-customer-specific model configs, CI/CD pipeline metadata | Store on internal systems only; no external sharing without approval. | All Cortexa employees. | Low — general corporate confidentiality. |
| **Confidential** | Business- or contractually-sensitive information whose disclosure would harm Cortexa or a customer. | Customer knowledge-base documents, proprietary model weights, vector embeddings, LLM API integrations | Encrypt at rest and in transit; access on a need-to-know basis; audit logging required. | Named roles only (e.g. Data Science, AI Security); NDA required for contractors. | Customer contracts; GDPR Art. 28 processor terms where applicable. |
| **Restricted** | Regulated personal, financial, or health data, or information whose exposure creates legal/regulatory liability. | PII in training data, financial documents processed by Copilot, health-sector customer data, agent action logs, integration credentials | Strong encryption; strict least-privilege access; mandatory access reviews; breach-notification plan required. | Named individuals with documented business need + manager approval. | GDPR (incl. Art. 9 special categories), EU AI Act, sector regulation (e.g. financial/health). |

## Task 4 — Residency, Retention & Privacy Obligations (Confidential and Restricted Assets)

| Asset ID | Asset Name | Data Residency | Retention Period | Privacy / Regulatory Obligations |
|---|---|---|---|---|
| AST-001 | Copilot — Primary LLM (OpenAI) | United States | Per OpenAI API terms (zero-retention tier enabled) | GDPR Art. 28; customer DPA; EU AI Act GPAI provider obligations |
| AST-002 | Copilot — Secondary LLM (Azure OpenAI) | European Union (customer-selected region) | 30-day abuse-monitoring retention, then deleted | GDPR (EU residency); customer DPA; EU AI Act |
| AST-003 | Copilot — Anthropic Claude | United States | Per Anthropic commercial API terms; not used for training | GDPR Art. 28; customer DPA |
| AST-004 | Copilot — System Prompt Library | United States | Version-controlled indefinitely | Internal IP protection |
| AST-005 | Copilot — Conversation Logs Store | United States (EU option for EU customers) | 90 days default; extendable per contract | GDPR; EU AI Act Art. 12 (record-keeping); sector regulation |
| AST-006 | Copilot — Customer Integration API Layer | United States | Not retained beyond request lifecycle | Customer DPA; GDPR |
| AST-007 | Search — Production Vector Index | United States | Active contract + 30 days | Customer contractual confidentiality; GDPR Art. 28 |
| AST-010 | Search — Indexed Customer Knowledge Bases | United States (EU option for EU customers) | Active contract; deleted within 30 days of termination | GDPR; customer DPA; sector regulation |
| AST-011 | Search — Retrieval API | United States | Not retained beyond request lifecycle | Customer DPA; GDPR |
| AST-012 | Search — Query & Retrieval Logs | United States | 60 days | GDPR; customer DPA |
| AST-014 | Agents — Underlying LLM | United States | Per provider API terms | GDPR Art. 28; customer DPA; EU AI Act |
| AST-015 | Agents — Tool/Plugin Integration Registry | United States | Registry indefinite; credentials rotated per policy | Access-control critical; high-impact incident trigger if exposed |
| AST-016 | Agents — Action/Execution Logs | United States | 1 year (audit/evidence requirement) | GDPR; EU AI Act Art. 12; sector regulation |
| AST-017 | Agents — Human Approval Interface | United States | Tied to AST-016 (1 year) | GDPR; EU AI Act Art. 14 (human oversight evidence) |
| AST-018 | Agents — Customer System Credentials | United States | Rotated every 90 days; revoked on termination | Highest-impact if compromised; customer DPA; GDPR |
| AST-019 | Shared — MLflow Model Registry | United States | Indefinite (audit history) | GDPR where lineage references personal data; internal IP |
| AST-020 | Shared — Model Artifact Storage (S3) | United States | Indefinite while in use; archived on deprecation | Internal IP; GDPR (model-inversion risk) |
| AST-021 | Shared — Training/Fine-Tuning Dataset Store | United States | Per data-use agreement; anonymized data retained longer | GDPR Art. 5 & Art. 9; EU AI Act Art. 10 (data governance) |
| AST-023 | Shared — Cloud IAM | United States | Access logs retained 1 year | Internal HR-adjacent data; SOC 2 control evidence |
| AST-024 | Shared — Security Telemetry / SIEM | United States | 180 days (investigation window) | GDPR where telemetry includes personal identifiers |

## Task 3 — Level-0 Data Flow Diagram

Built from the provided `Data_Flow_Starter.drawio` shape library (External Entity, Process, Data Store, and dashed Trust Boundary conventions), exported below as the submission-ready diagram. A single central process represents the whole Cortexa AI platform — Copilot, Search, and Agents are deliberately **not** broken out separately at this Level 0 view.

![Cortexa AI Platform Level 0 Data Flow Diagram — a single central "Cortexa AI Platform" process inside a dashed red trust boundary representing Cortexa's cloud environment, surrounded by four internal data stores (Vector Databases, MLflow Model Registry + S3, Training/Fine-Tuning Dataset Store, and Conversation/Query/Action Logs) and six external entities (End Users/Customers, Customer Identity Provider, OpenAI API, Azure OpenAI API, Anthropic Claude API, and Customer Systems for Agents tool calls). Solid navy arrows show internal data flows; dashed red arrows mark every flow that crosses the trust boundary.](./assets/Level0_Data_Flow_Diagram.png)

**Trust-boundary crossings shown** (every one marked in dashed red, per the assignment requirement):

1. End Users/Customers ↔ Cortexa AI Platform — query/prompt and response
2. Customer Identity Provider (SSO/IdP) → Cortexa AI Platform — authentication
3. Cortexa AI Platform ↔ OpenAI API — inference request/response
4. Cortexa AI Platform ↔ Azure OpenAI API — inference request/response
5. Cortexa AI Platform ↔ Anthropic Claude API — inference request/response
6. Cortexa AI Platform ↔ Customer Systems (CRM/ticketing) — agent tool/action calls

**Internal flows** (solid navy, do not leave Cortexa's cloud environment): Vector Databases ↔ Platform (embed/retrieve); MLflow Registry + S3 ↔ Platform (load/register model); Training Dataset Store → Platform (read training data); Platform → Logs (write logs).

## Deliverable Readiness

| Deliverable | Status |
|---|---|
| `AI_Asset_Register.xlsx` | Complete — 25 assets, all four classification tiers represented |
| `Data_Classification_Matrix.xlsx` | Complete — matches Cortexa's four-tier scheme referenced by every register row |
| Level-0 data flow diagram (`.png`) | Complete — every trust-boundary crossing marked |
| Cross-consistency check | Passed — every "Sensitivity Classification" value in the register exactly matches a tier defined in the matrix |

**Ready for review and sign-off by Priya Sharma, Data Science Lead**, before AI threat landscape mapping and the ISO/IEC 42001 risk register begin.
