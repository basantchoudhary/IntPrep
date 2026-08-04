# North Star Data & AI Platform — Reference Architecture

> Source diagram: `north-star-platform.png` (drop the image in this folder).
> This file is the text transcription so the content is searchable, quotable in
> slides, and diffable as the design evolves.

**Build Once. Reuse Everywhere. Govern by Design. Operate Intelligently.**

A composable Data & AI platform with deterministic controls and agentic
intelligence powering trusted, scalable and cost-efficient outcomes.

---

## Consumers (top of stack)

| Consumer | Capabilities |
|---|---|
| **BI & Analytics** | Dashboards, Reports, Self-Service Analytics |
| **ML Engineering** | Feature Engineering, Training, Evaluation, Model Registry |
| **AI Engineering** | GenAI Apps, Agents, RAG, Inference, Copilots |

## Sources (left edge)

Enterprise Apps · Databases · APIs · Files / Documents · Streaming / Events ·
IoT / Devices · External / 3rd Party

---

## Data & AI Foundation (Lakehouse)

| Layer | Contents | Qualities |
|---|---|---|
| **Gold** — Curated / Serving | Business Models, Data Products, Semantic Models, Feature Store, Serving | High Quality, Business Certified, Optimized for Consumption |
| **Silver** — Conformed / Integrated | Conformed Dimensions, Aggregates, Cleansed Data, Standardized Entities | Integrated, Consistent, Reusable Across Domains |
| **Bronze** — Raw / Landing | Raw Ingestion, Immutable, Schema-on-Read, CDC, Events | Raw, Immutable, Traceable, Re-process Ready |

## Ingestion & Transformation Platform

| Service | Capabilities |
|---|---|
| **Ingestion Service** | Batch, Stream, CDC, APIs, Files, Events |
| **Transformation Service** | Batch, Stream, SQL, Python, DataOps, Auto Optimize |
| **Orchestration Service** | Workflows, Scheduling, Dependencies, Triggers |
| **Data Movement Service** | ELT, CDC, Replication, Cross-Cloud / Regions |
| **Integration Service** | Connectors, APIs, Data Sharing |

---

## Horizontal Platform Services (Reusable Assets)

These are the reuse surface — the layer that carries the T1/T2/T3 "reusable
pattern" story, and where the agent suite plugs in.

| Service | Capabilities |
|---|---|
| **Metadata & Data Catalog** | Catalog, Business Glossary, Lineage, Tags & Classification, Data Contracts |
| **Data Expectation Services** | Data Quality, Profiling, Reconciliation, Anomaly Detection, Rules & Expectations |
| **Testing Framework Service** | Unit / Integration Tests, Data Tests, Regression Tests, Test Data Management, Coverage & Reporting |
| **Observability & Telemetry** | Metrics, Logs, Traces, Dashboards, SLA / SLO Monitoring |
| **Cost & FinOps Service** | Cost Tracking, Budgeting, Showback / Chargeback, Forecasting, Cost Optimization |
| **Ops Gate Service** | Readiness Checks, Policy Enforcement, Quality Gate, Security Gate, Deployment Approval |
| **Self-Healing Ops Service** | Anomaly Detection, Root Cause Analysis, Auto Remediation, Validation, Learning & Feedback |
| **AI / Agentic & Governance** | IAM / RBAC / ABAC, Data Masking, Policy Enforcement, Audit & Compliance, Privacy & Retention |

## Observability & Operations (Cross-Cutting)

Centralized Monitoring (Metrics, Logs, Traces) · Alerting & Notification
(Real-time, Thresholds) · Incident Management (Correlation, RCA) · SLA / SLO
Management (Uptime, Latency, Quality) · Runbooks & Automation (Operational
Playbooks) · Capacity & Performance (Quotas, Scaling, Health)

## Foundational Enablers

Cloud & Compute (IaaS / PaaS / Serverless) · Storage & Networking (Data Lake,
VPC, DNS) · Platform Engineering (CI/CD, IaC, GitOps) · Data Governance
(Policies, Stewardship) · Compliance (ISO, GDPR, HIPAA, etc.) · People & Process
(Ownership, Ways of Working)

---

## Platform Principles

1. Trust & Quality by Design
2. Security & Governance First
3. Observability & Operability
4. Cost Efficiency & Sustainability
5. Composability & Reuse
6. Automation + Intelligence

## Composable Platform Lego Blocks

*Build Once. Reuse Everywhere.*

Ingestion · Transformation · Data Quality · Metadata · Testing · Observability ·
Cost · Ops Gate · Self Healing · Security · AI / Agentic

## Deterministic + Agentic

| Mode | Responsibilities |
|---|---|
| **Deterministic Controls** | Rules, Policies, Validations, Gates, Automation |
| **Agentic Intelligence** | Diagnose, Recommend, Decide, Remediate, Learn |

This split is the spine of the whole narrative: **deterministic where the answer
is knowable and must be repeatable; agentic only where judgement is genuinely
required.** Every agent in the suite is specified against this line.

## Outcomes

Trusted Data (High Quality, Reliable) · Faster Delivery (Reusable, Automated) ·
Operational Excellence (Observable, Self-Healing) · Cost Optimized (Efficient,
Transparent) · AI Ready (Scalable, Governed) · Business Impact (Insights to
Outcomes)
