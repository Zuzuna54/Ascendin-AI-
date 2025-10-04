# Alternatives & Trade-offs Analysis

## Purpose

This document consolidates all design decisions from the Personal Data Vault architecture, exploring alternatives evaluated and explaining the rationale for chosen approaches. It serves as a decision log for understanding why the architecture is designed as it is.

---

## High-Level Architectural Decisions

### Decision 1: Local-First vs Cloud-First Architecture

| Approach | Pros | Cons | When to Use |
|----------|------|------|-------------|
| **Cloud-First** | Best performance; unlimited storage; easy cross-device sync | Privacy concerns; vendor lock-in; requires network | Enterprise SaaS; acceptable to trust provider |
| **Fully Local** | Maximum privacy; no cloud costs; works offline | No cross-device sync; limited compute for AI; device capacity | Privacy-critical; single-device; no AI needs |
| **Hybrid Local-First (Chosen)** | Privacy + performance; user control; offline-capable | Moderate complexity; need sync protocol | Multi-device personal data with privacy requirements |

**Decision:** Hybrid local-first chosen  
**Rationale:** 
- Satisfies "user control" principle (REQ-2.1)
- Enables cross-device sync (REQ-2.3)
- Allows cloud AI assist while maintaining privacy (REQ-2.4, REQ-2.5)
- Works offline (REQ-7.1)

**Sources:**
- "Local-First Software" (Ink & Switch, 2019): https://www.inkandswitch.com/local-first/  
  Date Checked: 04 Oct 2025
- arch.md §1.1 (Core Design Decision #1)

---

### Decision 2: CRDT vs OT vs LWW for Multi-Device Sync

| Approach | Pros | Cons | When to Use |
|----------|------|------|-------------|
| **Last-Write-Wins (LWW)** | Simple; low overhead | Data loss on concurrent edits | Single master; rare conflicts |
| **Operational Transform (OT)** | Fine-grained text merging | Requires central server; complex | Real-time collaboration (Google Docs) |
| **CRDT (Chosen)** | Automatic resolution; offline-first; no central server | ~1KB metadata per entity; complex internals | Multi-device; offline; personal data |

**Decision:** CRDT (Automerge) chosen  
**Rationale:**
- No central coordination needed (REQ-7.1 offline support)
- Automatic conflict resolution (REQ-7.2)
- Proven in production (Automerge used by Ink & Switch apps)
- Sync latency <5s (REQ-4.1: measured 3.2s)

**Trade-off Accepted:** ~1KB overhead per message (acceptable for <1M messages)

**Sources:**
- CRDT Research Paper (Shapiro et al., 2011): https://hal.inria.fr/inria-00555588  
  Date Checked: 04 Oct 2025
- Automerge Documentation: https://automerge.org/  
  Date Checked: 04 Oct 2025
- arch.md §5 (Multi-Device Synchronization)

---

### Decision 3: Serverless vs Containers vs VMs

| Approach | Pros | Cons | When to Use |
|----------|------|------|-------------|
| **EC2 (VMs)** | Full control; persistent; predictable cost | Manual scaling; ops overhead; always-on cost | Long-running services; predictable load |
| **ECS/Fargate (Containers)** | Docker standard; more control than Lambda | 10× Lambda cost; min resources wasted | Containerized microservices; >15min tasks |
| **Kubernetes** | Industry standard; portable | High complexity; $72/month control plane | Large teams; multi-cloud; complex orchestration |
| **Lambda (Chosen)** | Auto-scale 0→1000; pay-per-use; ephemeral (privacy) | 15-min timeout; cold starts | Event-driven; ephemeral compute; privacy-critical |

**Decision:** AWS Lambda chosen  
**Rationale:**
- Ephemeral execution aligns with zero-knowledge (REQ-5.2)
- Auto-scales for backlog processing (REQ-4.2)
- Cost: $0.35/month for 10K messages vs $50/month EC2
- No ops overhead

**Trade-off Accepted:** Cold start latency (200ms), but mitigated with reserved concurrency

**Sources:**
- AWS Lambda Pricing: https://aws.amazon.com/lambda/pricing/  
  Date Checked: 04 Oct 2025
- arch.md §7.1 (Serverless Architecture)

---

## Component-Level Technology Decisions

### Storage: SQLite vs Core Data vs Realm

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| **Core Data** | Native Apple; migrations; iCloud sync | Verbose; complex; no FTS | ❌ Not chosen |
| **Realm** | ORM; reactive; fast | 10MB binary; deprecated Swift API | ❌ Not chosen |
| **SQLite (Chosen)** | Lightweight (1MB); FTS5; ACID; ubiquitous | Lower-level API | ✅ Chosen |

**Trade-off:** Verbose SQL vs ORM convenience → SQL chosen for control + FTS5

**Source:** SQLite About Page: https://www.sqlite.org/about.html  
Date Checked: 04 Oct 2025

---

### Vector Database: pgvector vs Pinecone vs FAISS

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| **FAISS** | Fastest (optimized by Facebook) | Separate server; no SQL | ❌ Not chosen |
| **Pinecone** | Managed; easy | $70/month; vendor lock-in | ❌ Not chosen |
| **pgvector (Chosen)** | PostgreSQL extension; SQL-native; open-source | Slower than FAISS for >10M vectors | ✅ Chosen |

**Trade-off:** Speed vs integration → pgvector chosen for SQL + metadata filtering in one query

**Source:** pgvector GitHub: https://github.com/pgvector/pgvector  
Date Checked: 04 Oct 2025

---

### Embedding Model: OpenAI vs Sentence-BERT

| Option | Dimensions | Quality | Cost | Latency | Decision |
|--------|-----------|---------|------|---------|----------|
| **OpenAI text-embedding-3-small** | 1536 | High | $0.0001/msg | 450ms | ✅ Primary |
| **Sentence-BERT (MiniLM-L6)** | 384 | Medium | Free | 120ms local | ✅ Fallback |

**Trade-off:** Cost + quality vs privacy + speed → OpenAI primary, local fallback

**Source:** OpenAI Embeddings Guide: https://platform.openai.com/docs/guides/embeddings  
Date Checked: 04 Oct 2025

---

### Encryption: AES-GCM vs ChaCha20-Poly1305

| Algorithm | Speed (Apple) | Hardware Accel | Standard | Decision |
|-----------|---------------|----------------|----------|----------|
| **AES-256-GCM** | Fast (AES-NI) | ✅ Yes | NIST SP 800-38D | ✅ Chosen |
| **ChaCha20-Poly1305** | Fast (ARM) | ❌ No on Intel | RFC 8439 | ❌ Not chosen |

**Trade-off:** Hardware acceleration (AES-GCM) vs universality (ChaCha20)

**Source:** NIST SP 800-38D: https://csrc.nist.gov/publications/detail/sp/800-38d/final  
Date Checked: 04 Oct 2025

---

### Key Derivation: PBKDF2 vs Argon2 vs scrypt

| Algorithm | Memory-Hard | Native Support | Iterations | Decision |
|-----------|-------------|----------------|------------|----------|
| **PBKDF2-HMAC-SHA256** | ❌ No | ✅ CryptoKit | 600K | ✅ Chosen |
| **Argon2** | ✅ Yes | ❌ Requires library | N/A | ❌ Not chosen |
| **scrypt** | ✅ Yes | ❌ Requires library | N/A | ❌ Not chosen |

**Trade-off:** Memory-hardness (Argon2) vs native support (PBKDF2) → PBKDF2 chosen

**Source:** OWASP Password Storage: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html  
Date Checked: 04 Oct 2025

---

## Summary Decision Matrix

### Architecture Pattern Decisions

| Decision Point | Options Evaluated | Chosen | Key Trade-off | Requirement Driving Choice |
|----------------|-------------------|--------|---------------|---------------------------|
| **Data Location** | Local-only / Cloud-only / Hybrid | Hybrid | Complexity vs functionality | REQ-2.1 (user control) |
| **Sync Strategy** | LWW / OT / CRDT | CRDT | Overhead vs automatic resolution | REQ-7.2 (automatic conflicts) |
| **Compute Model** | VMs / Containers / Serverless | Serverless | Ops overhead vs cold starts | REQ-5.2 (ephemeral for privacy) |
| **Storage Model** | Relational / Document / Key-Value | Relational (SQLite) | Complexity vs query power | REQ-3.1 (semantic indexing) |
| **Embedding Gen** | Local / Cloud | Cloud (+ local fallback) | Privacy vs quality | REQ-3.1 (semantic search quality) |

### Security Decisions

| Decision Point | Options Evaluated | Chosen | Key Trade-off | Requirement Driving Choice |
|----------------|-------------------|--------|---------------|---------------------------|
| **Encryption** | AES-GCM / ChaCha20 / RSA | AES-256-GCM | Speed vs universality | REQ-5.1 (E2E encryption) |
| **Key Derivation** | PBKDF2 / Argon2 / scrypt | PBKDF2 (600K) | Native support vs memory-hardness | REQ-5.4 (user auth) |
| **Key Storage** | Keychain / File / Memory | Keychain + Secure Enclave | Convenience vs extreme security | REQ-5.5 (secure storage) |
| **Auth Method** | Password / Biometric / 2FA | Passphrase + Biometric | UX vs security | REQ-5.4 (user auth) |

### Performance Decisions

| Decision Point | Options Evaluated | Chosen | Key Trade-off | Requirement Driving Choice |
|----------------|-------------------|--------|---------------|---------------------------|
| **Vector Index** | HNSW / IVF / Flat | HNSW | Accuracy vs speed | REQ-4.4 (search <200ms) |
| **Sync Mode** | Real-time / Polling / Batch | Real-time (debounced 1s) | Latency vs bandwidth | REQ-4.1 (sync <5s) |
| **Embedding Batch** | Single / Batch-10 / Batch-100 | Batch-10 | Cost vs latency | REQ-4.2 (10K in 1 hour) |

---

## Cost-Benefit Analysis

### Monthly Cost Estimate (Per User, 10K Messages)

| Component | Technology | Cost | Justification |
|-----------|-----------|------|---------------|
| **Compute** | Lambda (10K invocations) | $0.35 | Pay-per-use; no idle cost |
| **Storage** | S3 (5 GB) | $0.12 | 11 nines durability |
| **Database** | RDS t4g.micro (pgvector) | $12.00 | Shared across users; $0.01/user amortized |
| **Queue** | SQS (10K messages) | $0.00 | Free tier (1M requests/month) |
| **Embeddings** | OpenAI API (10K × $0.0001) | $1.00 | High quality; multilingual |
| **Network** | Data transfer (5 GB egress) | $0.45 | CloudFront caching |
| **Total** | — | **$1.92/user/month** | — |

**Comparison:**
- Managed service alternative (e.g., Notion): $10-20/month
- Self-hosted (EC2 t3.small 24/7): $15/month + ops time
- **Our hybrid:** $1.92/month + user privacy control

---

## When to Deviate from This Architecture

### Use Case 1: Enterprise Deployment (1,000+ Users)
**Changes:**
- Replace Lambda with ECS Fargate (predictable load → containers cheaper)
- Add dedicated pgvector cluster (RDS too expensive at scale)
- Consider self-hosted embeddings (avoid OpenAI cost at scale)

### Use Case 2: Maximum Privacy (No Cloud)
**Changes:**
- Remove Lambda, S3, Redis (fully local)
- Use Sentence-BERT for local embeddings (384-d)
- Peer-to-peer sync (no central Redis; use mDNS/Bonjour)

### Use Case 3: Real-Time Collaboration (Multiple Users per Vault)
**Changes:**
- Add authorization layer (users, roles, permissions)
- Use Yjs instead of Automerge (better real-time performance)
- WebSocket-based sync (instead of polling Redis)

---

## Key Trade-offs Summary

| Choice | Benefit | Cost | Mitigation |
|--------|---------|------|------------|
| **Local-First** | Privacy; offline-capable | Sync complexity | CRDTs eliminate conflicts |
| **Serverless** | Auto-scale; low cost | Cold starts (200ms) | Reserved concurrency |
| **OpenAI Embeddings** | High quality (1536-d) | $1/10K messages; API dependency | Local fallback (Sentence-BERT) |
| **CRDT** | Automatic conflicts | ~1KB overhead per message | Acceptable for <1M messages |
| **pgvector** | SQL integration | Slower than FAISS for >10M vectors | Sufficient for personal vault scale |
| **Zero-Knowledge** | User privacy | Cannot debug user data | Extensive local logging + user tools |

---

## Decision Criteria Framework

When evaluating alternatives, we prioritized:

1. **Privacy > Performance** (when conflicting)
2. **User Control > Convenience** (per design principle)
3. **Open-Source > Proprietary** (avoid lock-in)
4. **Simplicity > Features** (maintainability)
5. **Standards-Based > Custom** (interoperability)

**Example:** Local embeddings (Sentence-BERT) are slower but offered as option for privacy-conscious users.

---

## References

All sources cited in component-specific "Alternatives Considered" sections. See `references.md` for complete list.

---

**Last Updated:** 04 October 2025  
**Maintainer:** Architecture Team
