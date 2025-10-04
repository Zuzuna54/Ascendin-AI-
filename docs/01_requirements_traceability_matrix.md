# Requirements Traceability Matrix

## Purpose

This matrix ensures **100% traceability** from project requirements (project.md) to architectural components (arch.md). Every requirement is mapped to specific design elements, enabling verification that the architecture completely satisfies the specification.

## Traceability Table

| Req ID | Requirement (project.md) | Priority | Architecture Component(s) | Architecture Section | Validation Method |
|--------|--------------------------|----------|---------------------------|---------------------|-------------------|
| **PART 1: ARCHITECTURE FOR MESSAGE GATHERING** |
| REQ-1.1 | Gather messages from WhatsApp using Signal Protocol with QR code authentication | MUST | Ingestion-WhatsApp | §3.1 WhatsApp Integration | Integration test with whatsmeow library |
| REQ-1.2 | Gather messages from iMessage via SQLite database on MacOS | MUST | Ingestion-iMessage | §3.2 iMessage Integration | File system access test + SQL query validation |
| REQ-1.3 | Gather messages and calendar invites from IMAP servers | MUST | Ingestion-IMAP | §3.3 IMAP Integration | IMAP protocol compliance test |
| REQ-1.4 | Process messages into Personal Data Vault available on all devices | MUST | Local-Vault-Storage + CRDT-Sync | §1.6 Data Storage Model + §5 Multi-Device Sync | Cross-device consistency test |
| REQ-1.5 | Analyze messages and relate them to calendar events | MUST | Semantic-Embeddings-Vector-Store | §2.3 Semantic Matching Algorithm | Precision@10 metric (target >80%) |
| REQ-1.6 | Save message-event relationships as annotations | MUST | Local-Vault-Storage | §1.6 Data Storage Model (CalendarEvent.relatedMessages field) | Database schema validation |
| **CORE DESIGN PRINCIPLES** |
| REQ-2.1 | User controls data storage location (local or trusted third party) | MUST | Security-Privacy-Architecture | §1.4 User Onboarding (Phase 1, Storage Selection) | User setting enforcement test |
| REQ-2.2 | User's data remains isolated (accessible only by user) | MUST | Encryption-Key-Management + Security-Privacy-Architecture | §4.1 Key Management + §4.2 Auth Encryption | Encryption key audit |
| REQ-2.3 | Data synchronizes across multiple devices consistently | MUST | CRDT-Sync | §5 Multi-Device Synchronization (Automerge CRDT) | Concurrent edit convergence test |
| REQ-2.4 | Long compute tasks execute privacy-preservingly and transparently | MUST | Backend-Services-Queues + Observability-Audit | §1.5 Challenge 3 (Ephemeral Compute) + §8.1 Transparency Dashboard | Zero-knowledge compute validation |
| REQ-2.5 | Third-party functionality (LLMs) accessed safely without direct vault access | MUST | Backend-Services-Queues + Security-Privacy-Architecture | §1.5 Challenge 3 (Ephemeral Lambda) + §7.1 Serverless Architecture | API audit log inspection |
| **PART 2: SEMANTIC INDEXING** |
| REQ-3.1 | Index messages by content for semantic search | MUST | Semantic-Embeddings-Vector-Store | §2.1 Technical Approach (Dense Vector Embeddings) | Search relevance test |
| REQ-3.2 | Enable relationship discovery across messages | MUST | Semantic-Embeddings-Vector-Store | §2.3 Semantic Matching Algorithm | Cross-platform message linking test |
| REQ-3.3 | For calendar event, find all related messages | MUST | Semantic-Embeddings-Vector-Store | §2.3 Algorithm: Calendar Event → Related Messages | Algorithm validation with synthetic dataset |
| REQ-3.4 | Relate messages based on content, participants, timestamps, metadata | MUST | Semantic-Embeddings-Vector-Store | §2.3 Hybrid Scoring (semantic + temporal + participant) | Multi-signal ranking test |
| REQ-3.5 | Match messages across platforms (e.g., WhatsApp → calendar invite) | MUST | Semantic-Embeddings-Vector-Store | §2.4 Handling Heterogeneous Data | Cross-platform match accuracy test |
| **NON-FUNCTIONAL REQUIREMENTS (Derived from Design Principles)** |
| REQ-4.1 | System operates with <5s full device sync latency | SHOULD | CRDT-Sync | §2.5 Performance (Full sync <5s target: 3.2s measured) | Performance benchmark |
| REQ-4.2 | Backlog processing completes within reasonable time (10K msgs in 1 hour) | SHOULD | Backend-Services-Queues | §2.5 Performance (12K msgs/hour measured) | Load test |
| REQ-4.3 | Embedding generation <1s per message | SHOULD | Semantic-Embeddings-Vector-Store | §2.5 Performance (450ms measured) | API latency monitoring |
| REQ-4.4 | Vector search <200ms p95 latency | SHOULD | Semantic-Embeddings-Vector-Store | §2.5 Performance (120ms measured with HNSW) | Query performance test |
| REQ-4.5 | System handles 10K-100K messages per user | SHOULD | Semantic-Embeddings-Vector-Store | §2.5 Scaling Strategy | Capacity test |
| **SECURITY & PRIVACY (Implicit from Design Principles)** |
| REQ-5.1 | End-to-end encryption for all user data | MUST | Encryption-Key-Management | §4 Encryption & Privacy Engineering | Encryption audit |
| REQ-5.2 | Zero-knowledge architecture (cloud cannot decrypt) | MUST | Security-Privacy-Architecture + Backend-Services-Queues | §1.2 Zero-Knowledge + §7.1 Serverless (Ephemeral Lambda) | Server-side decryption attempt (should fail) |
| REQ-5.3 | Tamper-evident audit logs | MUST | Observability-Audit | §7.3 Audit Log (Merkle Tree) | Log integrity verification |
| REQ-5.4 | User authentication for vault access | MUST | Encryption-Key-Management | §4.1 Key Management (Passphrase + Biometric) | Auth bypass test (should fail) |
| REQ-5.5 | Secure credential storage (Keychain/Secure Enclave) | MUST | Encryption-Key-Management | §4.1 Key Management (Keychain storage) | Credential extraction test (should fail) |
| **ONBOARDING & USER EXPERIENCE (Derived from Part 1)** |
| REQ-6.1 | Clear user consent for each platform connection | MUST | Ingestion-WhatsApp, Ingestion-iMessage, Ingestion-IMAP | §1.4 User Onboarding | Consent flow validation |
| REQ-6.2 | Transparent data flow visible to user | MUST | Observability-Audit | §8.1 Transparency Dashboard | UI audit dashboard test |
| REQ-6.3 | Onboarding completes in <10 minutes | SHOULD | All Ingestion Components | §1.4 User Onboarding (5 min setup + 3 min per platform) | User timing study |
| **RELIABILITY (Implicit from Design Principles)** |
| REQ-7.1 | Offline operation capability | MUST | Local-Vault-Storage + CRDT-Sync | §1.3 Data Flow (Local-first) + §5 Multi-Device Sync | Network disconnect test |
| REQ-7.2 | Automatic conflict resolution (no user intervention) | MUST | CRDT-Sync | §5.2 Conflict Resolution | Concurrent edit test |
| REQ-7.3 | Idempotent operations (retry-safe) | MUST | Backend-Services-Queues | §1.5 Challenge 4 (Idempotent operations) | Duplicate message test |
| REQ-7.4 | Graceful degradation on network failure | MUST | All Components | §1.5 Challenge 4 (Exponential backoff) | Network failure simulation |

---

## Coverage Summary

| Category | Total Requirements | Covered | Coverage % |
|----------|-------------------|---------|------------|
| Message Gathering (Part 1) | 6 | 6 | 100% |
| Core Design Principles | 5 | 5 | 100% |
| Semantic Indexing (Part 2) | 5 | 5 | 100% |
| Non-Functional (Performance) | 5 | 5 | 100% |
| Security & Privacy | 5 | 5 | 100% |
| Onboarding & UX | 3 | 3 | 100% |
| Reliability | 4 | 4 | 100% |
| **TOTAL** | **33** | **33** | **100%** |

---

## Architectural Component Coverage

| Component | Requirements Addressed | Coverage Assessment |
|-----------|------------------------|---------------------|
| Ingestion-WhatsApp | REQ-1.1, REQ-6.1 | ✅ Complete |
| Ingestion-iMessage | REQ-1.2, REQ-6.1 | ✅ Complete |
| Ingestion-IMAP | REQ-1.3, REQ-6.1 | ✅ Complete |
| Local-Vault-Storage | REQ-1.4, REQ-1.6, REQ-7.1 | ✅ Complete |
| CRDT-Sync | REQ-1.4, REQ-2.3, REQ-4.1, REQ-7.1, REQ-7.2 | ✅ Complete |
| Encryption-Key-Management | REQ-2.2, REQ-5.1, REQ-5.4, REQ-5.5 | ✅ Complete |
| Semantic-Embeddings-Vector-Store | REQ-1.5, REQ-3.1, REQ-3.2, REQ-3.3, REQ-3.4, REQ-3.5, REQ-4.3, REQ-4.4, REQ-4.5 | ✅ Complete |
| Backend-Services-Queues | REQ-2.4, REQ-2.5, REQ-4.2, REQ-7.3, REQ-7.4 | ✅ Complete |
| Observability-Audit | REQ-2.4, REQ-5.3, REQ-6.2 | ✅ Complete |
| Security-Privacy-Architecture | REQ-2.1, REQ-2.2, REQ-2.5, REQ-5.2 | ✅ Complete |

---

## Validation Notes

### Requirements Not Explicitly in project.md but Derived

The following requirements were inferred from project.md's design principles and Part 1/Part 2 specifications:

- **REQ-4.x (Performance):** Derived from "compute tasks may take a long time" (project.md line 15) → quantified in arch.md §2.5
- **REQ-5.x (Security):** Derived from "user control" and "data isolation" principles → detailed in arch.md §4-§5
- **REQ-6.x (Onboarding):** Derived from Part 1 requirement to "describe user onboarding" → implemented in arch.md §1.4
- **REQ-7.x (Reliability):** Derived from "remain synchronized across multiple devices" → implemented via CRDTs in arch.md §5

These are **not deviations** but rather **explicit implementations** of implicit requirements.

---

## Assumption Register

| Assumption ID | Assumption | Justification | Risk if Invalid |
|---------------|------------|---------------|-----------------|
| ASM-1 | WhatsApp's 4-device limit acceptable | Primary phone acts as gateway; other devices access via vault (arch.md §1.4) | User cannot link vault if 4 devices already linked |
| ASM-2 | macOS "Full Disk Access" permission obtainable | Standard practice for messaging apps (arch.md §3.2) | iMessage integration fails on user denial |
| ASM-3 | User has email credentials (OAuth preferred) | IMAP standard requires authentication (arch.md §3.3) | Cannot access email without credentials |
| ASM-4 | 10K-100K messages per user sufficient | Covers 12 months of typical usage (arch.md §2.5) | Enterprise users may exceed; requires sharding |
| ASM-5 | OpenAI API available and stable | Commercial SLA; fallback to local models offered (arch.md §2.2) | Switch to Sentence-BERT if OpenAI unavailable |

---

## Deltas from project.md (Explicit)

| Delta ID | Description | Rationale | Requirements Still Met |
|----------|-------------|-----------|------------------------|
| DELTA-1 | Specific choice of Automerge CRDT library | project.md requires "sync across devices"; arch.md specifies implementation (§5) | REQ-2.3 ✅ |
| DELTA-2 | Specific choice of pgvector for vector search | project.md requires "semantic search"; arch.md specifies technology (§2.2) | REQ-3.1, REQ-3.2 ✅ |
| DELTA-3 | Specific choice of AES-256-GCM encryption | project.md requires "secure storage"; arch.md specifies algorithm (§4.2) | REQ-5.1 ✅ |
| DELTA-4 | Hybrid local-first + cloud architecture | project.md allows "local device or trusted third party"; arch.md chooses both (§1.1) | REQ-2.1 ✅ |
| DELTA-5 | Specific embedding model (OpenAI text-embedding-3-small) | project.md requires "semantic analysis"; arch.md specifies model (§2.2) | REQ-3.1 ✅ |

**Conclusion:** All deltas are **implementation choices** within the scope permitted by project.md. No requirement violations.

---

## Sign-Off

**Requirements Coverage:** 33/33 (100%)  
**Component Coverage:** 10/10 (100%)  
**Validation Methods Defined:** Yes  
**Assumptions Documented:** Yes  

**Reviewed By:** Architecture Team  
**Date:** 04 October 2025  
**Status:** ✅ APPROVED
