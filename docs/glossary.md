# Glossary

## Purpose

This glossary provides plain-English definitions for all technical terms, acronyms, and domain-specific concepts used throughout the Personal Data Vault documentation. Terms are organized alphabetically within categories for easy reference.

---

## Core Concepts

### Personal Data Vault
A secure, encrypted repository where an individual stores their personal data (messages, emails, calendar events) with cryptographic guarantees that only they can access it. Combines local storage with optional cloud backup, maintaining user sovereignty over data location and access.

### Zero-Knowledge Architecture
A system design where service providers (cloud storage, compute) cannot decrypt or read user data, even though they host it. Achieved through client-side encryption where decryption keys never leave the user's device.

### Local-First Software
Applications that treat the local device as the primary source of truth, working fully offline, with cloud services acting only as backup and synchronization facilitators. Ensures data access without network dependency.

### End-to-End Encryption (E2EE)
Encryption where data is encrypted on the sender's device and only decrypted on the recipient's device, with no intermediate parties (including servers) able to access plaintext. In single-user systems like this vault, "sender" and "recipient" are the same user's different devices.

### Data Sovereignty
The principle that users maintain complete control over their data, including where it's stored, who can access it, and the ability to export or delete it at any time.

---

## Cryptography & Security

### AES-256-GCM
**Advanced Encryption Standard with 256-bit keys in Galois/Counter Mode**  
A symmetric encryption algorithm that provides both confidentiality (encrypted ciphertext) and integrity (authentication tag). Widely used industry standard; hardware-accelerated on modern processors.

- **256-bit key:** Extremely secure; would take billions of years to brute-force
- **GCM mode:** Combines encryption and authentication in single operation
- **Use case:** Encrypting message content in the vault

### Authentication Tag
A cryptographic checksum (in GCM: 128 bits) that allows detection of any tampering with encrypted data. If even one bit of the ciphertext is altered, decryption will fail, alerting to potential data corruption or malicious modification.

### PBKDF2
**Password-Based Key Derivation Function 2**  
An algorithm that converts a user's passphrase into a cryptographic key by applying a hash function thousands of times. The repetition makes brute-force attacks expensive.

- **Iterations:** 600,000 (OWASP recommendation for 2023)
- **Purpose:** Derive master encryption key from user passphrase
- **Time:** ~500ms on modern devices (intentionally slow to resist attacks)

### HKDF
**HMAC-based Key Derivation Function**  
A method to derive multiple keys from a single master key. Uses cryptographic hashing to generate unique, deterministic keys for specific purposes.

- **Use case:** Derive per-message encryption keys from master key
- **Benefit:** No need to store thousands of individual keys

### Salt
Random data added to a password before hashing to ensure the same password produces different keys for different users. Prevents rainbow table attacks.

- **Size:** 256 bits (32 bytes) in this system
- **Storage:** Stored alongside encrypted data (not secret, but required for decryption)

### Nonce
**Number Used Once**  
A random value used in encryption to ensure the same plaintext produces different ciphertexts each time. Critical for security: reusing a nonce with the same key can break encryption.

- **Size:** 96 bits (12 bytes) for AES-GCM
- **Generation:** Cryptographically secure random number generator
- **Storage:** Stored with ciphertext (not secret)

### Master Key
The primary encryption key from which all other keys are derived. Generated from user passphrase; never leaves the device; stored in Keychain.

- **Size:** 256 bits
- **Lifecycle:** Created on setup; rotated annually
- **Protection:** Encrypted by iOS/macOS Keychain, optionally by Secure Enclave

### Secure Enclave
A hardware security module built into Apple devices (iPhone 5s+, T2/M-series Macs) that stores cryptographic keys in a tamper-resistant environment. Keys cannot be extracted, even with physical access to the device.

### Keychain
Apple's password management system for iOS/macOS. Stores sensitive data (passwords, encryption keys) encrypted and protected by the device passcode and/or biometric authentication.

- **Access control:** Can require Touch ID/Face ID for retrieval
- **Backup:** Can optionally sync to iCloud Keychain (disabled for vault master key)

### Biometric Authentication
Authentication using physical characteristics: Touch ID (fingerprint) or Face ID (facial recognition). Provides convenient yet secure access without typing passphrase repeatedly.

### Signal Protocol
An end-to-end encryption protocol used by WhatsApp, Signal, and other messaging apps. Provides forward secrecy (past messages safe even if current keys compromised) and deniability.

- **Version:** Signal Protocol v3
- **Use case:** WhatsApp message encryption (handled by WhatsApp, not our vault)

### TLS (Transport Layer Security)
Cryptographic protocol for secure communication over networks. Ensures data in transit cannot be intercepted or tampered with.

- **Version:** TLS 1.2 or 1.3 required
- **Use case:** Connections to WhatsApp, IMAP servers, cloud storage

### CSPRNG
**Cryptographically Secure Pseudo-Random Number Generator**  
A random number generator suitable for cryptographic use (unpredictable, uniform distribution). Used for generating nonces, keys, salts.

- **Implementation:** `SecRandomCopyBytes` on Apple platforms

---

## Data Synchronization

### CRDT
**Conflict-Free Replicated Data Type**  
A data structure designed to automatically resolve conflicts when the same data is modified on multiple devices simultaneously, without requiring a central server or user intervention.

- **Property:** Eventual consistency (all devices converge to same state)
- **Example:** Both iPhone and MacBook tag a message while offline; when they sync, both tags are preserved
- **Library:** Automerge (Swift implementation)

### Eventual Consistency
A consistency model where, given no new updates, all replicas of data will eventually converge to the same state. Trades immediate consistency for availability and partition tolerance.

### Operational Transform (OT)
An alternative to CRDTs for conflict resolution; transforms operations to be compatible. More complex; requires central server. Not used in this system (CRDTs chosen instead).

### Last-Write-Wins (LWW)
A simple conflict resolution strategy where the most recent modification (by timestamp) is kept. Rejected for this system because it can cause data loss if concurrent edits occur.

### Automerge
An open-source CRDT library that automatically merges changes from multiple sources. Used for synchronizing vault data across iPhone, MacBook, iPad.

- **License:** MIT
- **Language:** JavaScript (original), Swift port available
- **Use case:** Multi-device sync engine

### Sync Latency
Time from when data changes on one device to when it appears on another. Target: <5 seconds for full device sync.

---

## Message Platforms

### WhatsApp
End-to-end encrypted messaging app owned by Meta. Uses Signal Protocol; requires phone as primary device.

- **Multi-device:** Supports up to 4 linked companion devices
- **Integration:** Via `whatsmeow` library (open-source)

### iMessage
Apple's messaging service integrated into iOS/macOS. Messages stored locally in SQLite database; sync via iCloud.

- **Database:** `~/Library/Messages/chat.db` on macOS
- **Access:** Requires "Full Disk Access" permission
- **Integration:** Direct SQLite queries

### IMAP
**Internet Message Access Protocol**  
Standard protocol for retrieving emails from a server. Supports folders, search, synchronization.

- **Version:** IMAP4rev1 (RFC 3501)
- **Port:** 993 (with TLS)
- **Use case:** Fetching emails and calendar invites

### iCalendar (iCal)
Standard format for calendar events, specified in RFC 5545. Used by email calendar invites.

- **Format:** Text-based (VEVENT, VTODO, etc.)
- **MIME type:** `text/calendar`
- **Extension:** `.ics` files

### MIME
**Multipurpose Internet Mail Extensions**  
Standard for formatting email messages with multiple parts (text, HTML, attachments).

- **Spec:** RFC 2045-2049
- **Use case:** Parsing email structure to extract calendar invites

---

## Semantic Search

### Embedding
A numerical representation of text (vector of floating-point numbers) that captures semantic meaning. Similar meanings → similar vectors.

- **Dimensions:** 1536 (OpenAI text-embedding-3-small)
- **Example:** "meeting" and "appointment" have similar embeddings

### Vector Database
A database optimized for storing and searching high-dimensional vectors. Supports similarity search (find nearest neighbors).

- **Implementation:** pgvector (PostgreSQL extension)

### pgvector
Open-source PostgreSQL extension for vector similarity search. Adds vector data type and indexing (HNSW) for efficient nearest neighbor queries.

- **License:** PostgreSQL License (permissive)
- **Operators:** `<->` (L2 distance), `<=>` (cosine distance)

### Cosine Similarity
A measure of similarity between two vectors, calculated as the cosine of the angle between them. Range: -1 (opposite) to 1 (identical).

- **Formula:** `similarity = (A · B) / (||A|| × ||B||)`
- **Use case:** Finding messages similar to a search query

### HNSW
**Hierarchical Navigable Small World**  
A graph-based algorithm for approximate nearest neighbor search. Very fast for high-dimensional vectors (<200ms for 100K vectors).

- **Trade-off:** Approximate (may miss some results) vs. exhaustive (slower but perfect)
- **Accuracy:** Typically >95% recall

### Semantic Search
Search based on meaning rather than exact keyword matching. Uses embeddings to find conceptually similar content.

- **Example:** Query "car" matches "automobile," "vehicle"
- **Contrast:** Keyword search requires exact word match

### Hybrid Ranking
Combining multiple signals (semantic similarity, temporal proximity, participant overlap) to rank search results.

- **Weights:** 60% semantic, 20% temporal, 20% participants (configurable)

---

## Architecture Patterns

### Serverless Computing
Cloud execution model where code runs in ephemeral containers managed by the provider. Auto-scales; pay per execution; no persistent state.

- **Implementation:** AWS Lambda
- **Use case:** Embedding generation (runs only when needed)

### Ephemeral Compute
Computing environment that exists only during execution and is destroyed afterward. No data persists; logs disabled for privacy.

- **Use case:** Zero-knowledge AI processing (Lambda decrypts, processes, re-encrypts, terminates)

### Lambda Function
AWS's serverless compute service. Runs code in response to events (HTTP request, file upload, queue message).

- **Languages:** Python, Node.js, Go, Java, Ruby, .NET
- **Timeout:** Max 15 minutes per execution

### SQS
**Simple Queue Service**  
AWS message queue for asynchronous communication. Ensures at-least-once delivery; supports FIFO (first-in-first-out).

- **Use case:** Queue messages for embedding generation

### Redis
In-memory data store often used for caching and pub/sub messaging. Very fast (sub-millisecond latency).

- **Use case:** Real-time sync coordination (publish changes, devices subscribe)

### S3
**Simple Storage Service**  
AWS object storage with 99.999999999% durability. Stores files (blobs) with versioning, encryption, lifecycle policies.

- **Use case:** Encrypted message backups

### PostgreSQL
Open-source relational database. ACID-compliant; supports advanced features (JSON, full-text search, extensions).

- **Version:** 15+
- **Extension:** pgvector for vector storage

### REST API
**Representational State Transfer**  
Architectural style for web services using standard HTTP methods (GET, POST, PUT, DELETE).

- **Characteristics:** Stateless, cacheable, uniform interface
- **Use case:** Client-server communication

---

## Data Formats

### JSON
**JavaScript Object Notation**  
Lightweight text format for structured data. Human-readable; language-independent.

- **Use case:** Message serialization, API payloads

### SQLite
Lightweight, file-based relational database. No separate server process; entire database in a single file.

- **Use case:** Local message storage on devices
- **Encryption:** SQLCipher extension for transparent database encryption

### UUID
**Universally Unique Identifier**  
128-bit identifier generated with extremely low collision probability (~0 for practical purposes).

- **Format:** `550e8400-e29b-41d4-a716-446655440000`
- **Use case:** Message IDs, contact IDs

---

## Performance & Reliability

### SLI
**Service Level Indicator**  
A quantitative measure of service behavior (latency, error rate, throughput).

- **Example:** p95 search latency (95th percentile response time)

### SLO
**Service Level Objective**  
Target value for an SLI. Defines acceptable performance.

- **Example:** SLI = search latency; SLO = <200ms (p95)

### SLA
**Service Level Agreement**  
Contract guaranteeing specific SLOs, typically with penalties if violated.

- **Example:** AWS S3 SLA guarantees 99.9% availability

### p95 Latency
95th percentile latency: 95% of requests complete faster than this value. Filters out worst 5% (outliers).

- **Why not average?** Average hides slow requests; p95 shows user experience

### Idempotency
Property where an operation can be repeated multiple times with the same result. Enables safe retries.

- **Example:** Storing message with ID "abc" twice results in only one stored message

### Backpressure
System feedback mechanism to slow down producers when consumers are overwhelmed. Prevents queue overflow.

- **Strategy:** Buffer messages; pause ingestion when buffer >80% full

### Exponential Backoff
Retry strategy where wait time doubles after each failure: 1s, 2s, 4s, 8s, 16s...

- **Benefit:** Prevents thundering herd (many clients retrying simultaneously)

### Circuit Breaker
Design pattern that stops calling a failing service after threshold reached, giving it time to recover.

- **States:** Closed (working), Open (failing, don't call), Half-Open (test if recovered)

---

## Development & Testing

### Mermaid
Markdown-based diagramming tool. Code blocks render as diagrams (flowcharts, sequence diagrams, etc.).

- **Benefit:** Version-controlled diagrams; no external tools
- **Use case:** All architecture diagrams in this documentation

### Integration Test
Test that verifies multiple components work together correctly (vs. unit test of single component).

- **Example:** Test WhatsApp ingestion → encryption → storage end-to-end

### Mock
Fake implementation of a dependency used in testing. Simulates real behavior without side effects.

- **Example:** Mock WhatsApp API to test ingestion logic without real account

### CI/CD
**Continuous Integration / Continuous Deployment**  
Automated pipeline that builds, tests, and deploys code on every commit.

---

## Protocols & Standards

### RFC
**Request for Comments**  
Document series describing Internet standards and protocols (managed by IETF).

- **Example:** RFC 3501 (IMAP), RFC 5545 (iCalendar)

### NIST
**National Institute of Standards and Technology**  
US government agency that publishes cryptographic standards.

- **Example:** NIST SP 800-38D (AES-GCM specification)

### FIPS 140-2
**Federal Information Processing Standard**  
US government security standard for cryptographic modules. Levels 1-4 (4 = highest).

- **Apple:** Secure Enclave is FIPS 140-2 Level 2 certified

### OWASP
**Open Web Application Security Project**  
Non-profit focused on software security best practices.

- **Example:** Password storage guidelines (PBKDF2 iterations)

### GDPR
**General Data Protection Regulation**  
European Union data privacy law. Key rights: access, deletion, portability, transparency.

- **Impact:** Users can export/delete vault data; privacy-first design

### CCPA
**California Consumer Privacy Act**  
California state law similar to GDPR. Grants data access and deletion rights.

---

## Apple Platform Specifics

### SwiftUI
Apple's modern UI framework for building interfaces declaratively (describe what, not how).

- **Benefit:** Cross-platform (iOS, macOS, iPadOS with shared code)

### Swift
Apple's programming language. Type-safe, performant, modern syntax.

- **Version:** 5.7+ (used in this project)

### CryptoKit
Apple's framework for cryptographic operations. Hardware-accelerated; easy-to-use API.

- **Includes:** AES, SHA, HKDF, PBKDF2, random generation

### Full Disk Access
macOS permission required to read files outside app's sandbox (e.g., iMessage database).

- **Grant:** System Preferences → Security & Privacy → Privacy → Full Disk Access

---

## Miscellaneous

### Backlog Sync
One-time download of historical messages when first connecting a platform (e.g., 12 months of WhatsApp messages).

### Tombstone
Marker indicating a deleted item in CRDT systems. Preserves deletion decision across devices.

- **Example:** Message deleted on iPhone; MacBook sees tombstone, removes message

### Merkle Tree
Tree structure where each node is the hash of its children. Root hash represents entire tree state.

- **Use case:** Tamper-evident audit logs (any change alters root hash)

### AAD
**Additional Authenticated Data**  
Data included in GCM authentication but not encrypted. Verified but visible.

- **Example:** Message metadata (ID, timestamp) as AAD ensures it can't be swapped between messages

### Multi-Device Protocol
WhatsApp's architecture allowing companion devices to access messages without phone being online.

- **Limit:** 4 companion devices
- **Use case:** Our app links as one of the 4 devices

### QR Code Pairing
Method to securely link devices by scanning a QR code displayed on one device with another.

- **Security:** QR contains encrypted challenge; prevents remote attacks

---

## Acronyms Quick Reference

| Acronym | Full Term | Category |
|---------|-----------|----------|
| AAD | Additional Authenticated Data | Cryptography |
| AES | Advanced Encryption Standard | Cryptography |
| API | Application Programming Interface | General |
| AWS | Amazon Web Services | Cloud |
| CCPA | California Consumer Privacy Act | Legal |
| CI/CD | Continuous Integration/Deployment | DevOps |
| CRDT | Conflict-Free Replicated Data Type | Synchronization |
| CSPRNG | Cryptographically Secure PRNG | Cryptography |
| E2EE | End-to-End Encryption | Security |
| FIPS | Federal Information Processing Standard | Standards |
| GCM | Galois/Counter Mode | Cryptography |
| GDPR | General Data Protection Regulation | Legal |
| HKDF | HMAC-based Key Derivation Function | Cryptography |
| HNSW | Hierarchical Navigable Small World | Search |
| IMAP | Internet Message Access Protocol | Protocols |
| JSON | JavaScript Object Notation | Data Format |
| LWW | Last-Write-Wins | Synchronization |
| MIME | Multipurpose Internet Mail Extensions | Email |
| NIST | National Institute of Standards | Standards |
| OWASP | Open Web Application Security Project | Security |
| OT | Operational Transform | Synchronization |
| PBKDF2 | Password-Based Key Derivation Function 2 | Cryptography |
| PII | Personally Identifiable Information | Privacy |
| REST | Representational State Transfer | API |
| RFC | Request for Comments | Standards |
| S3 | Simple Storage Service | Cloud |
| SLA | Service Level Agreement | Performance |
| SLI | Service Level Indicator | Performance |
| SLO | Service Level Objective | Performance |
| SQS | Simple Queue Service | Cloud |
| TLS | Transport Layer Security | Security |
| UUID | Universally Unique Identifier | Data Format |

---

## How to Use This Glossary

**For New Team Members:**
- Read "Core Concepts" section first
- Reference specific sections as you read component docs

**For Developers:**
- Bookmark for quick lookups during code review
- Ensure new technical terms added to future docs are defined here

**For Non-Technical Stakeholders:**
- Focus on "Core Concepts" and "Miscellaneous" sections
- Skip detailed cryptography unless interested

---

**Last Updated:** 04 October 2025  
**Maintainer:** Documentation Team  
**Contribution:** Add new terms via pull request with clear, plain-English definitions
