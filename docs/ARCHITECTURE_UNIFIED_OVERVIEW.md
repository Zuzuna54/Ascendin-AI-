# Unified Architecture Overview: Mac-Primary Ingestion with iPhone Fallback

## Executive Summary

This document describes the **revised unified architecture** for the Personal Data Vault, resolving ambiguities in the original design. The key shift is a **Mac-primary ingestion model** where the MacBook acts as the central aggregation hub for all message platforms (WhatsApp, iMessage, email, calendars), with intelligent **iPhone fallback** for WhatsApp when the Mac is unavailable.

**Key Changes from Original Design:**
1. **Unified Ingestion Gateway:** MacBook runs all platform adapters (not split between iPhone/Mac)
2. **iPhone Fallback Mode:** WhatsApp ingestion falls back to iPhone when Mac offline >24 hours
3. **Explicit Batch vs Realtime Paths:** Separate processing pipelines for historical backlog vs incremental sync
4. **Cloud-Primary Embeddings:** Encrypted ephemeral keys enable zero-knowledge AI compute (detailed key lifecycle)
5. **Local Embeddings Fallback:** On-device Sentence-BERT on iPhone when cloud unavailable or privacy mode enabled

**Rationale for Mac-Primary Design:**
- **Centralized Processing:** MacBook has superior compute (M1/M2 chips), persistent uptime, and unrestricted background execution
- **Platform Coverage:** macOS is ONLY platform with iMessage database access; consolidating all ingestion on Mac simplifies architecture
- **Power Efficiency:** MacBook on AC power supports 24/7 background jobs; iPhone battery constraints limit background processing
- **Operational Simplicity:** Single ingestion point reduces sync complexity vs. distributed ingestion across devices

---

## Architecture Principles

### 1. Local-First with Cloud Assistance

**Principle:** User devices are the source of truth; cloud provides optional backup, sync coordination, and compute assistance.

**Implementation:**
- **Local SQLite:** Primary storage on each device (iPhone, MacBook, iPad)
- **CRDT Sync:** Automerge replicates changes across devices via Redis pub/sub
- **Cloud Storage:** User-chosen provider (S3, iCloud, Google Drive) for encrypted backups
- **Ephemeral Compute:** AWS Lambda with time-limited decryption keys for AI processing

**Benefit:** System works fully offline; cloud enhances but isn't required

### 2. Zero-Knowledge Cloud Processing

**Principle:** Cloud services never have persistent access to plaintext data or decryption keys.

**Implementation:**
- **Client-Side Encryption:** AES-256-GCM before upload
- **Ephemeral Keys:** Short-lived (5-minute TTL) decryption keys sent with compute requests
- **Memory-Only Processing:** Lambda decrypts in RAM, processes, re-encrypts, terminates
- **No Key Persistence:** Ephemeral keys never written to disk, logs, or databases

**Verification:** User can audit that cloud storage contains only encrypted blobs

### 3. Graceful Degradation

**Principle:** System adapts to component failures without breaking core functionality.

**Degradation Scenarios:**

| Failure | Impact | Fallback Behavior |
|---------|--------|-------------------|
| **Mac Offline >24h** | WhatsApp ingestion stops | iPhone takes over WhatsApp; syncs to vault when Mac returns |
| **Cloud Storage Down** | Backups fail | Local vault continues; queue backups for retry |
| **OpenAI API Unavailable** | Cloud embeddings fail | Switch to local Sentence-BERT on iPhone (58% MTEB vs 62.3%) |
| **Network Partition** | Multi-device sync stops | Devices work independently; CRDT merges when reconnected |
| **Redis Pub/Sub Down** | Real-time sync paused | Queue operations; sync via S3 snapshots (slower but works) |

---

## Mac-Primary Ingestion Architecture

### Why Mac as Primary Gateway

**Technical Constraints:**

1. **iMessage Access:** ONLY macOS provides access to `~/Library/Messages/chat.db`
   - iOS: Sandboxed; cannot access Messages database
   - macOS: Full Disk Access permission grants read access
   - **Conclusion:** Mac is mandatory for iMessage; consolidate other platforms here

2. **Background Execution:**
   - macOS: Unrestricted background processing (App Nap can be disabled)
   - iOS: Strict background limits (30s execution windows, terminated if exceeds)
   - **Conclusion:** Mac better suited for continuous ingestion tasks

3. **Compute Resources:**
   - MacBook M1: 8-16 GB RAM, AC power, persistent uptime
   - iPhone: 4-6 GB RAM, battery constraints, intermittent availability
   - **Conclusion:** Mac can handle concurrent platform adapters efficiently

4. **Network Bandwidth:**
   - MacBook: Typically on WiFi/Ethernet (no data caps)
   - iPhone: Cellular data costs; user may disable background data
   - **Conclusion:** Mac preferable for large backlog downloads

### Adapter Pattern for Platform Ingestion

**Design:** Unified `MessageSource` protocol with platform-specific implementations

```swift
protocol MessageSource {
    var platformName: String { get }
    
    func authenticate() async throws
    func fetchHistoricalMessages(since: Date?, limit: Int) async throws -> [RawMessage]
    func startRealtimeMonitoring() async throws
    func stopMonitoring()
}

class WhatsAppAdapter: MessageSource {
    // Uses whatsmeow library; QR code pairing
}

class iMessageAdapter: MessageSource {
    // SQLite queries on chat.db; FSEvents monitoring
}

class IMAPAdapter: MessageSource {
    // IMAP protocol; OAuth 2.0 authentication
}

class CalDAVAdapter: MessageSource {
    // CalDAV for calendar events
}
```

**Ingestion Orchestrator (runs on Mac):**

```swift
class IngestionOrchestrator {
    let sources: [MessageSource] = [
        WhatsAppAdapter(),
        iMessageAdapter(),
        IMAPAdapter(provider: .gmail),
        IMAPAdapter(provider: .outlook),
        CalDAVAdapter()
    ]
    
    func runHistoricalBacklog() async {
        for source in sources {
            try? await source.fetchHistoricalMessages(since: lastSyncDate, limit: 10_000)
            // Store to vault, queue embeddings
        }
    }
    
    func startRealtimeIngestion() async {
        await withTaskGroup(of: Void.self) { group in
            for source in sources {
                group.addTask {
                    try? await source.startRealtimeMonitoring()
                }
            }
        }
    }
}
```

**Benefits:**
- **Single Codebase:** All adapters in one app (Mac agent)
- **Unified Error Handling:** Consistent retry logic across platforms
- **Resource Sharing:** Single database connection pool, shared credential manager
- **Simplified Testing:** Test all ingestion in one environment

### iPhone Fallback for WhatsApp

**Trigger Conditions:**
- Mac offline for >24 hours (detected via heartbeat)
- Mac explicitly paused/shutdown
- User preference: "Use iPhone for WhatsApp" (force fallback)

**Fallback Mechanism:**

```swift
class WhatsAppFallbackManager {
    var primaryDevice: Device = .mac  // Default
    var lastMacHeartbeat: Date?
    
    func checkFallback() {
        guard let lastSeen = lastMacHeartbeat else { return }
        let hoursSinceMac = Date().timeIntervalSince(lastSeen) / 3600
        
        if hoursSinceMac > 24 {
            // Mac offline >24h → activate iPhone fallback
            activateiPhoneFallback()
        }
    }
    
    func activateiPhoneFallback() {
        // 1. iPhone initializes whatsmeow client
        // 2. Scans QR code (user action required)
        // 3. Downloads missed messages since last Mac sync
        // 4. Continues real-time monitoring
        // 5. When Mac returns: Hand off session back to Mac
    }
}
```

**Handoff Protocol:**
```
1. Mac offline detected (24h timeout)
2. User notified: "Mac unavailable. Continue WhatsApp on iPhone?"
3. User approves → iPhone displays QR code
4. User scans with WhatsApp phone → iPhone now receives messages
5. iPhone syncs messages to vault via CRDT
6. Mac comes back online → Detects iPhone is active
7. Mac requests handoff: "Stop iPhone ingestion, I'll take over"
8. iPhone acknowledges, stops monitoring
9. Mac re-initializes whatsmeow, continues from iPhone's last message
```

**Constraints:**
- **WhatsApp 4-Device Limit:** Vault uses 1 slot (either Mac OR iPhone, not both simultaneously)
- **iOS Background Limits:** iPhone fallback works only when app in foreground or using Background Modes
- **User Friction:** Requires re-scanning QR code when switching devices

---

## Batch vs Realtime Processing Paths

### Two Distinct Pipelines

**Historical Batch Path (One-Time or Catch-Up):**
- **Trigger:** Initial setup, Mac returning after >24h offline, user manual "Re-sync All"
- **Volume:** 10,000-100,000 messages per platform
- **Processing:** Batched (100 messages per job), checkpointed (resume on failure)
- **Embeddings:** Queued to SQS FIFO; processed by 100 parallel Lambdas
- **Priority:** Low (background job; doesn't block user)
- **Duration:** 30-60 minutes for 10,000 messages

**Realtime Incremental Path (Continuous):**
- **Trigger:** New message arrives (FSEvents for iMessage, WebSocket for WhatsApp, IMAP IDLE for email)
- **Volume:** 1-100 messages per hour
- **Processing:** Immediate (single-message transactions)
- **Embeddings:** Streamed to Lambda (low latency)
- **Priority:** High (user expects to see new message in vault)
- **Duration:** <5 seconds end-to-end

**Schema Distinction:**

```sql
-- Track processing state
CREATE TABLE ingestion_jobs (
    id TEXT PRIMARY KEY,
    source TEXT NOT NULL,  -- 'whatsapp', 'imessage', 'email'
    type TEXT NOT NULL,  -- 'historical_batch' or 'realtime_stream'
    status TEXT NOT NULL,  -- 'pending', 'running', 'completed', 'failed'
    checkpoint_data JSON,  -- Resume state (last message ID, cursor)
    messages_processed INT DEFAULT 0,
    messages_total INT,
    started_at TIMESTAMP,
    completed_at TIMESTAMP
);

-- Separate queues for batch vs realtime
CREATE TABLE embedding_queue (
    id TEXT PRIMARY KEY,
    message_id TEXT NOT NULL,
    priority INT NOT NULL,  -- 1=realtime, 2=batch
    queued_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_embedding_priority ON embedding_queue(priority, queued_at);
```

### Checkpoint & Resume Strategy

**Batch Processing with Checkpoints:**

```swift
class BatchIngestionJob {
    var checkpoint: BatchCheckpoint?
    
    func processHistoricalMessages(source: MessageSource) async throws {
        let batchSize = 100
        var cursor = checkpoint?.lastCursor ?? source.initialCursor()
        
        while let batch = try await source.fetchBatch(from: cursor, limit: batchSize) {
            // Process batch
            for message in batch {
                try await vault.store(message)
                try await embeddingQueue.enqueue(message.id, priority: .batch)
            }
            
            // Update checkpoint
            cursor = batch.lastCursor
            checkpoint = BatchCheckpoint(lastCursor: cursor, processedCount: checkpoint.processedCount + batch.count)
            try await saveCheckpoint(checkpoint)
            
            // Yield for power/thermal management
            try await Task.sleep(nanoseconds: 100_000_000)  // 100ms between batches
        }
        
        markJobComplete()
    }
}

struct BatchCheckpoint: Codable {
    let lastCursor: String  // Platform-specific cursor (UID, ROWID, timestamp)
    let processedCount: Int
    let timestamp: Date
}
```

**Resume Logic:**

```swift
func resumeBatchJob(jobId: String) async throws {
    guard let checkpoint = loadCheckpoint(jobId) else {
        // No checkpoint → start from beginning
        return startNewBatchJob(jobId)
    }
    
    // Resume from last successful position
    logger.info("Resuming job \(jobId) from checkpoint: \(checkpoint.processedCount) messages processed")
    processBatchFrom(cursor: checkpoint.lastCursor)
}
```

---

## Embeddings Compute Model: Cloud-Primary with Encrypted Ephemeral Keys

### Cloud Embeddings Architecture

**Zero-Knowledge Guarantee:** OpenAI sees message content (necessary for embeddings), but AWS storage/logs NEVER see plaintext.

**Ephemeral Key Lifecycle (Detailed):**

```
Phase 1: CLIENT-SIDE KEY MINTING
──────────────────────────────────
Client generates ephemeral key (AES-256):
  ephemeral_key = SecRandomCopyBytes(32)  // 256-bit random key
  key_id = UUID()
  ttl = 300 seconds (5 minutes)
  scope = "embedding-generation-only"

Key never stored persistently:
  ✅ Generated in memory
  ✅ Used for single batch encryption
  ✅ Sent to Lambda via SQS metadata
  ❌ NEVER written to disk
  ❌ NEVER stored in Keychain
  ❌ NEVER logged

Phase 2: ENVELOPE ENCRYPTION (Messages)
────────────────────────────────────────
For each message in batch:
  1. Encrypt with ephemeral key:
     encrypted_msg = AES-256-GCM.encrypt(plaintext, key=ephemeral_key, nonce=random())
  
  2. Upload to S3:
     s3.putObject(
       bucket: "user-vault-staging",
       key: "ephemeral/\(key_id)/msg-\(msg_id).enc",
       data: encrypted_msg
     )

Phase 3: LAMBDA INVOCATION (via SQS)
─────────────────────────────────────
Client queues job:
  sqs.sendMessage(
    queue: "embedding-generation.fifo",
    body: {
      "key_id": key_id,
      "ephemeral_key": base64(ephemeral_key),  // IN SQS MESSAGE BODY
      "s3_keys": ["ephemeral/.../msg-1.enc", "ephemeral/.../msg-2.enc", ...],
      "ttl_expires_at": "2025-10-05T14:05:00Z",
      "user_id": "user-123"
    }
  )

SQS → Lambda trigger (automatic polling)

Phase 4: LAMBDA EXECUTION (Ephemeral Decryption)
─────────────────────────────────────────────────
Lambda receives SQS message:
  1. Extract ephemeral_key from message body (memory-only)
  2. Check TTL: if expired → reject, log error, move to DLQ
  3. Download encrypted messages from S3
  4. Decrypt IN MEMORY (ephemeral_key):
       plaintexts = [AES-GCM.decrypt(enc_msg, key=ephemeral_key) for enc_msg in s3_data]
  
  5. Generate embeddings (OpenAI API):
       embeddings = openai.create_embeddings(input=plaintexts)
  
  6. Re-encrypt embeddings (USER'S master key, derived):
       user_embedding_key = HKDF(user_master_key, context="embeddings")
       encrypted_embeddings = [AES-GCM.encrypt(emb, key=user_embedding_key) for emb in embeddings]
  
  7. Store in pgvector:
       db.execute("INSERT INTO message_embeddings ...")
  
  8. Delete S3 staging files:
       s3.deleteObjects(keys=s3_keys)  // Ephemeral data cleaned up
  
  9. Overwrite ephemeral_key in memory:
       ephemeral_key = bytes(32)  // Zeroed
  
  10. Lambda terminates → memory cleared, no logs persist

Phase 5: KEY DISPOSAL & AUDIT
──────────────────────────────
✅ Ephemeral key used exactly once
✅ S3 staging files deleted after processing
✅ Lambda logs no sensitive data (only key_id, message_id)
✅ Audit log records: "Embeddings generated for 100 messages (ephemeral key: key-abc123, OpenAI API called, re-encrypted, stored)"
❌ Ephemeral key NEVER persisted anywhere server-side
```

**Security Properties:**
- **Temporal Isolation:** Key valid only 5 minutes (prevents delayed attacks)
- **Scope Isolation:** Key only decrypts specific batch (not entire vault)
- **Audit Trail:** Key usage logged (without logging key value)
- **Verification:** User can confirm S3 staging area is empty after processing

**Sources:**
- Ephemeral Key Pattern: W3C Encrypted Data Vaults Spec (https://identity.foundation/edv-spec/)
- TTL-Based Keys: Temporal credentials pattern (AWS STS)
- Date Checked: 05 Oct 2025

### iPhone Local Embeddings Fallback

**Trigger Conditions:**
1. User enables "Privacy Mode" in settings (explicit opt-in)
2. OpenAI API unavailable (HTTP 503, timeouts)
3. User on metered connection (cellular data; minimize API costs)
4. Message flagged as sensitive (future: auto-detect medical/financial terms)

**On-Device Model: Sentence-BERT (all-MiniLM-L6-v2)**

**Specifications:**
- **Model Family:** Sentence-Transformers (HuggingFace)
- **Variant:** all-MiniLM-L6-v2 (optimized for mobile)
- **Dimensions:** 384 (vs 1536 for OpenAI)
- **Model Size:** 80 MB (quantized: 22 MB with Core ML)
- **Quality:** 58% MTEB score (vs 62.3% OpenAI = 4% lower recall)
- **Latency:** 50ms per message on iPhone 15 (M-series Neural Engine)
- **Battery Impact:** ~2% per 1,000 messages (acceptable for occasional use)

**Performance Comparison:**

| Model | Latency (per msg) | Quality (MTEB) | Dimensions | Privacy | Cost |
|-------|-------------------|----------------|------------|---------|------|
| **OpenAI 3-small (Cloud)** | 2ms (batched) | 62.3% | 1536 | Sent to OpenAI | $0.0001 |
| **Sentence-BERT (iPhone)** | 50ms | 58.0% | 384 | 100% local | $0 |

**Trade-off:** 25x slower, 4% lower quality, but complete privacy and zero cost

**Implementation:**

```swift
import CoreML

class LocalEmbeddingService {
    let model: SentenceBERTModel  // Core ML compiled model
    
    func generateEmbedding(_ text: String) -> [Float] {
        let input = preprocessText(text)  // Tokenize
        let output = try! model.prediction(input: input)
        return output.embedding  // 384-dimensional vector
    }
    
    func batchGenerate(_ messages: [String]) -> [[Float]] {
        // Use Neural Engine for acceleration
        return messages.map { generateEmbedding($0) }
    }
}
```

**Core ML Optimization:**
- **Quantization:** 32-bit floats → 8-bit integers (4x smaller model: 80MB → 22MB)
- **Neural Engine:** Apple's dedicated ML accelerator (3x faster than CPU)
- **Compilation:** Convert PyTorch model → Core ML format (`.mlmodelc`)

**Reconciliation with Cloud Index:**
- **Problem:** Local embeddings (384-dim) incompatible with cloud embeddings (1536-dim) in same pgvector table
- **Solution:** Separate columns in database:

```sql
CREATE TABLE message_embeddings (
    message_id TEXT PRIMARY KEY,
    embedding_cloud vector(1536),      -- OpenAI embeddings
    embedding_local vector(384),       -- Sentence-BERT embeddings
    source TEXT NOT NULL CHECK(source IN ('cloud', 'local')),
    generated_at TIMESTAMP DEFAULT NOW()
);

-- Query adapts based on available embedding
SELECT m.id, m.content,
       COALESCE(
         1 - (e.embedding_cloud <=> query_cloud),
         1 - (e.embedding_local <=> query_local)
       ) as similarity
FROM messages m
JOIN message_embeddings e ON m.id = e.message_id
ORDER BY similarity DESC
LIMIT 10;
```

**Migration Path (Local → Cloud):**
- User switches from Privacy Mode to Cloud Mode
- Background job: Re-generate embeddings via OpenAI API
- Replace `embedding_local` with `embedding_cloud`
- Delete local embeddings (free up space)

**Sources:**
- Sentence-BERT Paper: https://arxiv.org/abs/1908.10084
- Core ML Documentation: https://developer.apple.com/documentation/coreml
- Date Checked: 05 Oct 2025

---

## Offline & Backlog Behavior

### Scenario 1: Mac Offline for 3 Days

**Day 0 (Friday Evening):**
- Mac shutdown (user traveling)
- Last sync: 100 messages processed (up to 2025-10-04 18:00)

**Days 1-3 (Weekend):**
- iPhone receives ~50 new WhatsApp messages
- Messages stored in iPhone local vault (encrypted)
- CRDT operations queued (cannot reach Redis; Mac down)
- **No embeddings generated** (Mac offline; iPhone in low-power mode)

**Day 3 (Monday Morning):**
- Mac boots up
- Detects queued CRDT operations from iPhone (via S3 snapshot fallback)
- Merges iPhone changes: 50 new messages appear in Mac vault
- Triggers **backlog job**: "50 new messages need embeddings"
- Queues to SQS → Lambda processes in 30 seconds
- Embeddings indexed → Semantic search now works for weekend messages

**SLO:** Backlog processing completes within 1 hour of Mac reconnection

### Scenario 2: Mac Always-On, Email Backlog

**User adds Gmail account with 50,000 historical emails:**

```swift
class BacklogOrchestrator {
    func processEmailBacklog(account: EmailAccount) async throws {
        let totalMessages = try await account.getMessageCount()  // 50,000
        let batchSize = 1000
        var processedCount = 0
        
        while processedCount < totalMessages {
            // Check system health before each batch
            guard isSystemHealthy() else {
                logger.warning("System unhealthy; pausing backlog")
                try await Task.sleep(for: .seconds(60))
                continue
            }
            
            // Fetch batch
            let batch = try await account.fetchMessages(
                offset: processedCount,
                limit: batchSize
            )
            
            // Store to vault
            for message in batch {
                try await vault.store(message)
            }
            
            // Queue embeddings (batch priority)
            let embeddingBatch = batch.map { EmbeddingTask(messageId: $0.id, priority: .batch) }
            try await embeddingQueue.enqueueBatch(embeddingBatch)
            
            // Update checkpoint
            processedCount += batch.count
            updateProgress(current: processedCount, total: totalMessages)
            
            // Yield CPU (prevent thermal throttling)
            try await Task.sleep(for: .milliseconds(100))
        }
        
        logger.info("Backlog complete: \(totalMessages) messages processed")
    }
    
    func isSystemHealthy() -> Bool {
        // Check CPU temperature, battery level, disk space
        let cpuTemp = ProcessInfo.processInfo.thermalState
        let diskSpace = FileManager.default.availableCapacity
        
        guard cpuTemp != .critical else { return false }
        guard diskSpace > 1_000_000_000 else { return false }  // 1GB minimum
        
        return true
    }
}
```

**Backlog Scheduling:**
- **Trigger:** User idle (no keyboard/mouse input for 5 minutes)
- **Throttling:** Process 1,000 messages → wait 10 seconds → next batch
- **Pause Conditions:** High CPU temp, low battery (<20%), user becomes active
- **Resume:** Automatic when conditions improve

**macOS App Nap Handling:**

```swift
import Foundation

class BackgroundJobManager {
    var activityToken: NSObjectProtocol?
    
    func startBacklog() {
        // Prevent App Nap during backlog processing
        let options: ProcessInfo.ActivityOptions = [.userInitiated, .idleSystemSleepDisabled]
        activityToken = ProcessInfo.processInfo.beginActivity(
            options: options,
            reason: "Processing message backlog"
        )
        
        Task {
            try await runBacklogJob()
            endActivity()
        }
    }
    
    func endActivity() {
        if let token = activityToken {
            ProcessInfo.processInfo.endActivity(token)
        }
    }
}
```

**Sources:**
- macOS App Nap: https://developer.apple.com/library/archive/documentation/Performance/Conceptual/power_efficiency_guidelines_osx/AppNap.html
- Background Modes: https://developer.apple.com/documentation/backgroundtasks
- Date Checked: 05 Oct 2025

---

## Mapping to Requirements (Part 2 Focus)

### Requirements from requirements.md Part 2

**REQ-Part2-1:** High-level method to index messages by content for semantic search and relationship discovery

**Implementation:**
- ✅ **Semantic Indexing:** OpenAI text-embedding-3-small converts messages to 1536-d vectors
- ✅ **Vector Storage:** pgvector extension in PostgreSQL with HNSW indexing
- ✅ **Search:** Cosine similarity queries (<200ms for 100K messages)
- ✅ **Relationship Discovery:** Hybrid ranking (60% semantic + 20% temporal + 20% participants)
- **Location in Docs:** `docs/components/semantic-embeddings-and-vector-store.md`

**REQ-Part2-2:** Algorithm proposal to find all related messages for a calendar event

**Implementation:**
- ✅ **Algorithm:** Described in `FINAL_REPORT.md` §2.3 (Calendar Event → Related Messages)
- ✅ **Inputs:** Event title, description, start_time, participants
- ✅ **Processing:** Generate event embedding → pgvector similarity search with filters → hybrid re-ranking
- ✅ **Output:** Top 10 related messages with relevance scores
- **Location in Docs:** `docs/components/semantic-embeddings-and-vector-store.md`, Algorithm section

**REQ-Part2-3:** Technical approach description (data extraction, semantic analysis)

**Implementation:**
- ✅ **Data Extraction:** Platform-specific adapters normalize to unified schema (`docs/INGESTION_UNIFIED_MAC_PRIMARY.md`)
- ✅ **Semantic Analysis:** Embedding generation (cloud-primary with ephemeral keys; iPhone fallback)
- ✅ **Contact Deduplication:** Fuzzy matching (fuzzywuzzy library, 80% threshold)
- **Location in Docs:** `docs/EMBEDDINGS_COMPUTE_MODEL.md`

**REQ-Part2-4:** AI/ML technologies specification (NLP, embeddings, vector databases)

**Implementation:**
- ✅ **Embeddings:** OpenAI text-embedding-3-small (62.3% MTEB) + Sentence-BERT fallback (58% MTEB)
- ✅ **Vector DB:** pgvector 0.8.1 with HNSW indexing (1.5ms queries)
- ✅ **NLP:** spaCy 3.7+ (tokenization, NER, language detection)
- **Location in Docs:** `docs/decisions/10-embedding-model.md`, `docs/decisions/05-vector-database.md`, `docs/decisions/12-nlp-library.md`

**REQ-Part2-5:** Explanation of heterogeneous data handling challenges

**Implementation:**
- ✅ **Challenge 1:** Platform-specific formats (WhatsApp JSON, iMessage SQLite, IMAP MIME) → Unified schema normalization
- ✅ **Challenge 2:** Contact identity resolution (same person, multiple identifiers) → Probabilistic record linkage
- ✅ **Challenge 3:** Multilingual content → OpenAI multilingual embeddings (100+ languages)
- ✅ **Challenge 4:** Temporal distribution skew → Adaptive temporal windows per platform
- **Location in Docs:** `FINAL_REPORT.md` §2.4, §2.6

**REQ-Part2-6:** Adherence to all 5 design principles

**Implementation:**
- ✅ **User Control:** Storage location choice (local, iCloud, S3, custom)
- ✅ **Data Isolation:** Zero-knowledge architecture; client-side encryption
- ✅ **Multi-Device Sync:** CRDT (Automerge) with automatic conflict resolution
- ✅ **Privacy-Preserving Compute:** Ephemeral keys; Lambda terminates without persistence
- ✅ **Safe Third-Party Integration:** OpenAI sees content (necessary), AWS never sees plaintext
- **Location in Docs:** `docs/components/security-and-privacy-architecture.md`

---

## Risks & Mitigations

### Risk 1: Mac Unavailability (User Traveling)

**Likelihood:** Medium (users travel 1-2 weeks per year)  
**Impact:** Medium (WhatsApp ingestion paused if >24h)

**Mitigation:**
1. **iPhone Fallback:** Automatically activates after 24h Mac offline
2. **User Control:** Manual toggle: "Use iPhone for WhatsApp while traveling"
3. **Seamless Handoff:** Mac resumes when returns; no data loss
4. **Monitoring:** Alert user: "Mac offline 24h. Enable iPhone fallback?"

### Risk 2: iOS Background Task Limits

**Likelihood:** High (iOS strictly limits background execution)  
**Impact:** Low (iPhone fallback only for urgent scenarios)

**iOS Background Constraints:**
- **BGAppRefreshTask:** 30-second execution window
- **BGProcessingTask:** Longer window but requires device charging + WiFi
- **Push Notifications:** Can wake app, but 30-second limit still applies

**Mitigation:**
1. **Mac Primary:** Avoid relying on iPhone background processing
2. **User Awareness:** Document: "iPhone fallback requires app open or device charging"
3. **Notification Strategy:** Push notification wakes app → process messages in 30s window
4. **Foreground Mode:** Encourage users to open app periodically while Mac offline

**Sources:**
- Background Execution Limits: https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background
- Date Checked: 05 Oct 2025

### Risk 3: Ephemeral Key Leakage

**Likelihood:** Low (requires SQS message interception)  
**Impact:** Medium (attacker can decrypt single batch)

**Mitigation:**
1. **SQS Encryption:** Server-side encryption (SSE-SQS) encrypts message bodies at rest
2. **TLS in Transit:** SQS API calls over HTTPS
3. **TTL Expiration:** Key invalid after 5 minutes (limits damage window)
4. **Scope Limitation:** Key only decrypts specific batch (not entire vault)
5. **Audit Monitoring:** Alert if ephemeral keys accessed outside Lambda execution

**Blast Radius:** Worst case = 100 messages compromised (single batch); entire vault remains secure

### Risk 4: Backlog Processing Never Completes

**Likelihood:** Medium (user shuts Mac before completion)  
**Impact:** Medium (missing embeddings; semantic search incomplete)

**Mitigation:**
1. **Checkpointing:** Save progress every 100 messages
2. **Resume Automatic:** Next Mac startup resumes from checkpoint
3. **User Feedback:** Progress bar: "Indexing: 8,234 / 50,000 emails (83%)"
4. **Pause/Resume:** User can manually pause; resume when convenient
5. **SLO:** 90% of backlogs complete within 24 hours of initiation

---

## Comparison: Old vs New Architecture

| Aspect | Original Design | Unified Design |
|--------|----------------|----------------|
| **Ingestion** | Split: iPhone (WhatsApp), Mac (iMessage, email) | **Unified: Mac-primary (all platforms), iPhone fallback (WhatsApp only)** |
| **Complexity** | Two ingestion codebases; coordination overhead | Single codebase; adapter pattern; simpler |
| **Offline Behavior** | Ambiguous (unclear what happens if Mac down) | **Explicit: iPhone fallback after 24h; checkpointed resume** |
| **Embeddings** | Vague "ephemeral Lambda" | **Detailed: Encrypted ephemeral keys, 5-min TTL, envelope encryption, audit trail** |
| **Batch vs Realtime** | Not distinguished | **Separate pipelines: Historical batch (checkpointed) vs Realtime stream (low-latency)** |
| **Diagrams** | High-level only | **7 detailed diagrams: context, ingestion paths, key lifecycle, onboarding, sync, errors, security** |

---

## Next Steps

**Immediate Implementation (Weeks 1-4):**
1. Refactor ingestion to Mac-primary adapter pattern
2. Implement iPhone fallback logic for WhatsApp
3. Add checkpoint/resume for batch processing
4. Implement ephemeral key lifecycle for embeddings
5. Deploy local Sentence-BERT model to iPhone

**Validation (Weeks 5-6):**
1. Test Mac offline scenario (verify iPhone fallback)
2. Test 50K email backlog with checkpointing
3. Validate ephemeral key security (no persistence)
4. Measure iPhone local embedding performance

**Documentation (Week 7):**
1. Update all component docs to reflect unified architecture
2. Create migration guide from split to unified design
3. Update FINAL_REPORT.md with precise diagrams

---

**Document Version:** 1.0  
**Last Updated:** 05 October 2025  
**Status:** ✅ APPROVED - Replaces Split Ingestion Design

