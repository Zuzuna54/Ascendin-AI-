# Personal Data Vault - Documentation Index

## Welcome

This documentation suite provides comprehensive technical documentation for the Personal Data Vault system—a privacy-first, multi-platform message aggregation solution with semantic intelligence capabilities.

## About This Documentation

All documentation is derived from:
- **project.md** — Official project requirements and specifications
- **arch.md** — Approved architectural design and implementation guide

**Last Updated:** 04 October 2025

---

## Recommended Reading Order

### 1. Foundation (Start Here)
- **[Requirements Traceability Matrix](01_requirements_traceability_matrix.md)** — Maps every project requirement to architecture components
- **[Architecture Overview](02_architecture_overview.md)** — System-level design, principles, and diagrams
- **[Glossary](glossary.md)** — Definitions of all technical terms and acronyms

### 2. Core Components (Read in Order)

**Data Ingestion Layer:**
1. [WhatsApp Ingestion](components/ingestion-whatsapp.md)
2. [iMessage Ingestion](components/ingestion-imessage.md)
3. [IMAP Email/Calendar Ingestion](components/ingestion-imap.md)

**Storage & Synchronization Layer:**
4. [Local Vault Storage](components/local-vault-storage.md)
5. [CRDT Synchronization](components/crdt-sync.md)

**Security Layer:**
6. [Encryption & Key Management](components/encryption-key-management.md)
7. [Security & Privacy Architecture](components/security-and-privacy-architecture.md)

**Intelligence Layer:**
8. [Semantic Embeddings & Vector Store](components/semantic-embeddings-and-vector-store.md)

**Infrastructure Layer:**
9. [Backend Services & Queues](components/backend-services-and-queues.md)
10. [Observability & Audit Logging](components/observability-and-audit.md)

### 3. Analysis & Planning
- **[Alternatives & Trade-offs](alternatives-and-tradeoffs.md)** — Design decisions explained with comparisons
- **[Testing & Validation Plan](testing-and-validation-plan.md)** — QA strategy and acceptance criteria
- **[Security Threat Model](security-threat-model.md)** — STRIDE analysis and mitigations

### 4. Reference Materials
- **[Architecture Diagram Gallery](diagrams/architecture-gallery.md)** — All system diagrams in one place
- **[References](references.md)** — Citations and sources with verification dates

---

## Documentation Standards

### Diagrams
All diagrams use **Mermaid syntax** and render inline. No external image dependencies.

### Citations
Every technical claim is backed by official sources with:
- Direct URLs (no link shorteners)
- Date checked: DD Mon YYYY format
- Preference for official documentation over blog posts

### Terminology
All specialized terms are:
- Explained in plain English on first use
- Defined in the [Glossary](glossary.md)
- Used consistently throughout

---

## Quick Navigation by Role

**Product Managers / Business Stakeholders:**
→ Start with [Requirements Traceability Matrix](01_requirements_traceability_matrix.md) and [Architecture Overview](02_architecture_overview.md)

**Software Architects:**
→ Focus on [Architecture Overview](02_architecture_overview.md), [Alternatives & Trade-offs](alternatives-and-tradeoffs.md), and component docs

**Security Engineers:**
→ Read [Security & Privacy Architecture](components/security-and-privacy-architecture.md) and [Security Threat Model](security-threat-model.md)

**QA Engineers:**
→ Start with [Testing & Validation Plan](testing-and-validation-plan.md)

**New Team Members:**
→ Begin with [Glossary](glossary.md), then [Architecture Overview](02_architecture_overview.md)

---

## Document Structure

Each component document follows this template:
1. **Purpose & Responsibilities** — What it does and why
2. **Interfaces & Contracts** — Inputs, outputs, APIs
3. **Data Flow** — Sequence diagrams
4. **Deployment/Runtime** — Infrastructure and scaling
5. **Security & Privacy** — Encryption, permissions, PII handling
6. **Reliability & Performance** — SLIs, SLOs, error handling
7. **Alternatives Considered** — Options evaluated
8. **Risks & Mitigations** — Known issues and solutions
9. **Validation & Test Plan** — How we verify it works
10. **Deltas & Rationale** — Any deviations from requirements

---

## Contributing to Documentation

To maintain quality:
- All changes must reference either project.md or arch.md
- Add citations for new technical claims
- Update the glossary for new terms
- Regenerate diagrams if architecture changes
- Update "Last Updated" dates

---

## Questions or Feedback?

For clarifications or suggestions, please refer to:
- The specific component documentation
- The [Glossary](glossary.md) for term definitions
- The [References](references.md) for source materials
