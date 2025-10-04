# Architecture Diagram Gallery

## Purpose

This gallery consolidates all Mermaid diagrams used throughout the Personal Data Vault documentation for quick reference and comparison. Each diagram is reproduced here with context about where it's used and what it illustrates.

---

## System-Level Diagrams

### 1. System Context Diagram (C4 Level 1)

**Source:** `02_architecture_overview.md`  
**Purpose:** Shows the Personal Data Vault system in its broader context, including external actors, systems, and data flow boundaries.

```mermaid
graph TB
    subgraph "External Systems"
        WA[WhatsApp Servers<br/>Signal Protocol]
        iCloud[iCloud<br/>iMessage Sync]
        IMAP[IMAP/Email Servers<br/>Gmail, Outlook]
        OpenAI[OpenAI API<br/>Embeddings]
    end
    
    subgraph "User's Personal Devices"
        iPhone[iPhone<br/>iOS App]
        MacBook[MacBook<br/>macOS App]
        iPad[iPad<br/>iOS App]
    end
    
    subgraph "Personal Data Vault System"
        Vault[Personal Data Vault<br/>Encrypted Storage + Semantic Search]
    end
    
    subgraph "User-Chosen Cloud Storage"
        CloudStorage[Encrypted Backup<br/>iCloud / S3 / Google Drive]
    end
    
    subgraph "Ephemeral Compute (Zero-Knowledge)"
        Lambda[AWS Lambda<br/>Embedding Generator]
    end
    
    User((User)) -->|Uses| iPhone
    User -->|Uses| MacBook
    User -->|Uses| iPad
    
    iPhone -->|Scans QR code| WA
    MacBook -->|Reads SQLite DB| iCloud
    MacBook -->|IMAP/STARTTLS| IMAP
    
    iPhone <-->|CRDT Sync| Vault
    MacBook <-->|CRDT Sync| Vault
    iPad <-->|CRDT Sync| Vault
    
    Vault <-->|Encrypted Blobs| CloudStorage
    Vault -->|Ephemeral Key + Encrypted Data| Lambda
    Lambda -->|Embedding Request| OpenAI
    Lambda -->|Encrypted Embeddings| Vault
    
    style Vault fill:#90EE90
    style CloudStorage fill:#FFB6C1
    style Lambda fill:#FFD700
```

**Key Insights:**
- Three trust boundaries: User Devices (trusted), Network (encrypted), Cloud (zero-knowledge)
- User devices as primary; cloud as backup/compute assistant
- External services (WhatsApp, OpenAI) accessed through secure protocols

---

### 2. Container Diagram (C4 Level 2)

**Source:** `02_architecture_overview.md`  
**Purpose:** Details the major containers (applications, databases, services) within the system.

```mermaid
graph TB
    subgraph "Client Applications (Swift/SwiftUI)"
        UI[User Interface<br/>SwiftUI Views]
        LocalDB[(Local Vault DB<br/>SQLite + Encryption)]
        CRDTEngine[CRDT Sync Engine<br/>Automerge]
        IngestionMgr[Platform Managers<br/>WhatsApp, iMessage, IMAP]
        CryptoKit[CryptoKit<br/>AES-GCM, PBKDF2]
    end
    
    subgraph "Cloud Backend (Serverless)"
        APIGateway[API Gateway<br/>REST]
        EmbedLambda[Embedding Lambda<br/>Python]
        SyncCoord[Sync Coordinator<br/>Node.js]
        AuditLogger[Audit Logger<br/>Go]
        S3[(S3<br/>Encrypted Blobs)]
        Redis[(Redis<br/>Pub/Sub)]
        Postgres[(PostgreSQL<br/>+ pgvector)]
        SQS[SQS Queue<br/>Message Broker]
    end
    
    UI -->|Read/Write| LocalDB
    UI -->|User Actions| IngestionMgr
    LocalDB -->|Encrypt/Decrypt| CryptoKit
    LocalDB <-->|Operations| CRDTEngine
    
    CRDTEngine <-->|Sync Events| Redis
    IngestionMgr -->|Upload| S3
    S3 -.->|Trigger| SQS
    SQS -->|Invoke| EmbedLambda
    EmbedLambda -->|Store| Postgres
    EmbedLambda -->|Log| AuditLogger
    
    APIGateway -->|Route| EmbedLambda
    APIGateway -->|Route| SyncCoord
    APIGateway -->|Route| AuditLogger
    
    style LocalDB fill:#90EE90
    style S3 fill:#FFB6C1
    style Postgres fill:#FFB6C1
    style EmbedLambda fill:#FFD700
```

**Key Insights:**
- Client-side: Local DB primary, CRDTs for sync, platform-specific ingestion
- Server-side: Stateless Lambda functions, S3 for blobs, PostgreSQL for vectors
- Event-driven: S3 upload triggers SQS → Lambda processing

---

### 3. Layered Architecture

**Source:** `02_architecture_overview.md`  
**Purpose:** Shows the four-layer architecture following clean architecture principles.

```mermaid
graph TD
    subgraph "Layer 4: Presentation"
        UI[SwiftUI Views<br/>Search, Settings, Message List]
    end
    
    subgraph "Layer 3: Application Logic"
        SearchSvc[Search Service<br/>Semantic + Keyword]
        SyncSvc[Sync Service<br/>CRDT Operations]
        IngestionSvc[Ingestion Services<br/>WhatsApp, iMessage, IMAP]
    end
    
    subgraph "Layer 2: Domain Core"
        Vault[Vault Manager<br/>Message, Contact, Event entities]
        Crypto[Encryption Manager<br/>Key derivation, AES-GCM]
        CRDT[CRDT State Machine<br/>Automerge document]
    end
    
    subgraph "Layer 1: Infrastructure"
        Storage[Storage Adapters<br/>SQLite, S3, Keychain]
        Network[Network Clients<br/>HTTP, WebSocket, IMAP]
        Platform[Platform APIs<br/>File System, Keychain, Biometric]
    end
    
    UI --> SearchSvc
    UI --> SyncSvc
    UI --> IngestionSvc
    
    SearchSvc --> Vault
    SyncSvc --> CRDT
    IngestionSvc --> Vault
    
    Vault --> Crypto
    Vault --> CRDT
    Crypto --> Storage
    CRDT --> Storage
    CRDT --> Network
    
    Storage --> Platform
    Network --> Platform
    
    style Vault fill:#90EE90
    style Crypto fill:#FFD700
    style CRDT fill:#87CEEB
```

**Key Insights:**
- Dependency flows inward: Outer layers depend on inner, never reverse
- Layer 2 (Domain) defines interfaces; Layer 1 implements them
- Presentation isolated from infrastructure details

---

### 4. Security Boundaries

**Source:** `02_architecture_overview.md`  
**Purpose:** Illustrates trust zones and data protection at each boundary.

```mermaid
graph TB
    subgraph "Trust Boundary 1: User Device (Trusted)"
        User[User]
        App[Vault App]
        Keychain[Keychain<br/>Master Key]
        LocalDB[(SQLite<br/>Encrypted)]
    end
    
    subgraph "Trust Boundary 2: Network (Untrusted)"
        TLS[TLS 1.3<br/>In Transit]
    end
    
    subgraph "Trust Boundary 3: Cloud (Zero-Knowledge)"
        S3[(S3<br/>Encrypted Blobs)]
        Lambda[Lambda<br/>Ephemeral Keys]
        PG[(pgvector<br/>Encrypted Embeddings)]
    end
    
    User -->|Passphrase| Keychain
    User -->|Biometric| Keychain
    Keychain -->|Decrypts| LocalDB
    App -->|Encrypts| LocalDB
    
    App -->|Encrypted Data| TLS
    TLS -->|Cannot Decrypt| S3
    TLS -->|Ephemeral Key + Data| Lambda
    
    Lambda -->|Processes in Memory| Lambda
    Lambda -->|Re-encrypts| PG
    Lambda -.->|Terminates| Nowhere[No Persistence]
    
    style User fill:#90EE90
    style Keychain fill:#90EE90
    style LocalDB fill:#90EE90
    style TLS fill:#FFD700
    style S3 fill:#FFB6C1
    style Lambda fill:#FFD700
    style PG fill:#FFB6C1
    style Nowhere fill:#FF6B6B
```

**Key Insights:**
- Three trust zones: Device (full trust), Network (encrypted), Cloud (zero-knowledge)
- Encryption happens before crossing trust boundaries
- Lambda ephemeral: decrypts, processes, terminates (no persistence)

---

### 5. Deployment Architecture

**Source:** `02_architecture_overview.md`  
**Purpose:** Physical deployment topology across AWS regions and user devices.

```mermaid
graph TB
    subgraph "User Devices (Edge)"
        iOS[iPhone/iPad App<br/>Local SQLite + CRDT]
        macOS[MacBook App<br/>Local SQLite + CRDT]
    end
    
    subgraph "AWS Region: us-east-1"
        subgraph "VPC (Private Subnets)"
            Lambda1[Lambda: Embedding Generator<br/>Python 3.11]
            Lambda2[Lambda: Sync Coordinator<br/>Node.js 18]
            Lambda3[Lambda: Audit Logger<br/>Go 1.20]
            
            RDS[RDS PostgreSQL<br/>Multi-AZ, pgvector]
            ElastiCache[ElastiCache Redis<br/>Cluster Mode]
        end
        
        subgraph "Public Subnets"
            ALB[Application Load Balancer]
            APIGw[API Gateway<br/>REST + WebSocket]
        end
        
        subgraph "Managed Services"
            S3[S3: Encrypted Blobs<br/>Standard Storage]
            SQS[SQS: Message Queue<br/>FIFO]
            CloudWatch[CloudWatch<br/>Logs + Metrics]
        end
    end
    
    subgraph "External"
        OpenAI[OpenAI API<br/>text-embedding-3-small]
    end
    
    iOS <-->|TLS 1.3| APIGw
    macOS <-->|TLS 1.3| APIGw
    
    APIGw -->|Invoke| Lambda1
    APIGw -->|Invoke| Lambda2
    APIGw -->|Invoke| Lambda3
    
    Lambda1 <-->|SQL| RDS
    Lambda2 <-->|Pub/Sub| ElastiCache
    Lambda3 -->|Write-only| RDS
    
    Lambda1 <-->|HTTPS| OpenAI
    
    iOS <-->|S3 API| S3
    macOS <-->|S3 API| S3
    S3 -.->|Event| SQS
    SQS -.->|Poll| Lambda1
    
    Lambda1 -->|Log| CloudWatch
    Lambda2 -->|Log| CloudWatch
    Lambda3 -->|Log| CloudWatch
    
    style iOS fill:#90EE90
    style macOS fill:#90EE90
    style Lambda1 fill:#FFD700
    style RDS fill:#FFB6C1
    style S3 fill:#FFB6C1
```

**Key Insights:**
- Multi-AZ for RDS and ElastiCache (high availability)
- Private subnets for databases (not internet-accessible)
- Managed services (S3, SQS) reduce operational burden

---

## Component-Specific Diagrams

### 6. WhatsApp Ingestion Data Flow

**Source:** `components/ingestion-whatsapp.md`  
**Purpose:** End-to-end flow from QR code pairing to message storage.

```mermaid
sequenceDiagram
    actor User
    participant iPhone as iPhone App
    participant WhatsAppPhone as WhatsApp (Phone)
    participant whatsmeow as whatsmeow Library
    participant VaultMgr as Vault Manager
    participant LocalDB as Local SQLite
    
    Note over User,LocalDB: Phase 1: Initial Pairing (one-time)
    
    User->>iPhone: Tap "Connect WhatsApp"
    iPhone->>whatsmeow: Initialize client
    whatsmeow->>whatsmeow: Generate pairing request
    whatsmeow-->>iPhone: Display QR code
    iPhone-->>User: Show QR code on screen
    
    User->>WhatsAppPhone: Open WhatsApp → Linked Devices
    User->>WhatsAppPhone: Scan QR code with phone camera
    WhatsAppPhone->>whatsmeow: Send encrypted pairing credentials
    whatsmeow->>whatsmeow: Establish Signal Protocol session
    whatsmeow->>iPhone: Store session keys (Keychain)
    whatsmeow-->>iPhone: Pairing successful
    iPhone-->>User: "Connected to WhatsApp"
    
    Note over User,LocalDB: Phase 2: Historical Backlog Sync
    
    iPhone->>whatsmeow: Request chat history
    whatsmeow->>WhatsAppPhone: Fetch encrypted messages
    WhatsAppPhone-->>whatsmeow: Stream messages (batches of 100)
    
    loop For each message batch
        whatsmeow->>whatsmeow: Decrypt with Signal Protocol keys
        whatsmeow-->>iPhone: Decrypted messages (plaintext)
        iPhone->>VaultMgr: Store message (normalized schema)
        VaultMgr->>LocalDB: Encrypt + Write
        LocalDB-->>iPhone: Progress: "1,234 / 10,000 messages"
    end
    
    Note over User,LocalDB: Phase 3: Real-Time Sync (continuous)
    
    loop Continuous
        WhatsAppPhone->>whatsmeow: New message event
        whatsmeow->>whatsmeow: Decrypt message
        whatsmeow-->>iPhone: Push notification
        iPhone->>VaultMgr: Store new message
        VaultMgr->>LocalDB: Encrypt + Write
        iPhone-->>User: Update UI (new message badge)
    end
```

**Key Insights:**
- Three phases: Pairing, backlog, real-time
- QR code establishes secure Signal Protocol session
- Batching for backlog efficiency; streaming for real-time

---

### 7. Encryption Key Derivation Flow

**Source:** `components/encryption-key-management.md`  
**Purpose:** How user passphrase becomes master key and per-message keys are derived.

```mermaid
sequenceDiagram
    actor User
    participant App
    participant CryptoKit
    participant Keychain
    participant SecureEnclave
    
    Note over User,SecureEnclave: First-Time Setup
    
    User->>App: Enter passphrase (e.g., "MySecurePass123!")
    App->>CryptoKit: Generate 32-byte salt (random)
    App->>CryptoKit: PBKDF2(passphrase, salt, iterations=600,000)
    Note over CryptoKit: ~500ms computation (intentionally slow)
    CryptoKit-->>App: Master Key (256-bit)
    
    App->>Keychain: Store Master Key
    Note over Keychain: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
    Keychain->>SecureEnclave: Encrypt with hardware key
    SecureEnclave-->>Keychain: Encrypted key blob
    Keychain-->>App: Success
    
    App->>Keychain: Store salt (not secret, but needed for re-derivation)
    App-->>User: "Vault created successfully"
    
    Note over User,SecureEnclave: Subsequent Access (with Biometric)
    
    User->>App: Tap "Unlock Vault"
    App->>LocalAuthentication: Request biometric auth
    LocalAuthentication->>User: Show Touch ID / Face ID prompt
    User->>SecureEnclave: Provide fingerprint / face scan
    SecureEnclave-->>LocalAuthentication: Authenticated ✓
    LocalAuthentication-->>App: Success
    
    App->>Keychain: Retrieve Master Key (with biometric)
    Keychain->>SecureEnclave: Decrypt key blob
    SecureEnclave-->>Keychain: Decrypted Master Key
    Keychain-->>App: Master Key (in memory)
    
    Note over App: Master Key stays in memory for session<br/>Cleared on app background or timeout
```

**Key Insights:**
- PBKDF2 slow by design (600K iterations) to resist brute-force
- Secure Enclave encrypts master key at rest
- Biometric auth for convenient access

---

### 8. Message Encryption Flow

**Source:** `components/encryption-key-management.md`  
**Purpose:** Step-by-step encryption of a single message.

```mermaid
sequenceDiagram
    participant VaultMgr as Vault Manager
    participant EncryptMgr as Encryption Manager
    participant CryptoKit
    participant Keychain
    participant LocalDB as SQLite
    
    VaultMgr->>EncryptMgr: encrypt(messageId, plaintext, metadata)
    EncryptMgr->>Keychain: Retrieve Master Key
    Keychain-->>EncryptMgr: Master Key (cached in memory)
    
    EncryptMgr->>EncryptMgr: Derive per-message key
    Note over EncryptMgr: HKDF(MasterKey, salt=messageId, info="message-encryption")
    EncryptMgr-->>EncryptMgr: Message Key (256-bit, unique per message)
    
    EncryptMgr->>CryptoKit: Generate random nonce (12 bytes)
    CryptoKit-->>EncryptMgr: Nonce
    
    EncryptMgr->>EncryptMgr: Prepare AAD
    Note over EncryptMgr: AAD = {"messageId": "...", "timestamp": "...", "version": 1}
    
    EncryptMgr->>CryptoKit: AES.GCM.seal(plaintext, key=MessageKey, nonce, aad=AAD)
    CryptoKit-->>EncryptMgr: SealedBox {nonce, ciphertext, tag}
    
    EncryptMgr->>EncryptMgr: Zero out Message Key in memory
    Note over EncryptMgr: Overwrite with zeros immediately
    
    EncryptMgr-->>VaultMgr: EncryptedBlob {nonce, ciphertext, tag, aad}
    VaultMgr->>LocalDB: Store encrypted blob
```

**Key Insights:**
- Per-message keys via HKDF (no storage overhead)
- AAD (Additional Authenticated Data) ensures metadata integrity
- Key zeroing prevents memory extraction attacks

---

### 9. Message Decryption Flow

**Source:** `components/encryption-key-management.md`  
**Purpose:** Decryption with authentication tag verification.

```mermaid
sequenceDiagram
    actor User
    participant UI
    participant EncryptMgr as Encryption Manager
    participant CryptoKit
    participant Keychain
    
    User->>UI: Open message
    UI->>EncryptMgr: decrypt(encryptedBlob, messageId)
    
    EncryptMgr->>Keychain: Retrieve Master Key
    Keychain-->>EncryptMgr: Master Key
    
    EncryptMgr->>EncryptMgr: Derive per-message key (same as encryption)
    Note over EncryptMgr: HKDF(MasterKey, salt=messageId, info="message-encryption")
    
    EncryptMgr->>CryptoKit: AES.GCM.open(SealedBox, key=MessageKey, aad=AAD)
    
    alt Authentication Success
        CryptoKit-->>EncryptMgr: Plaintext (tag verified ✓)
        EncryptMgr->>EncryptMgr: Zero out Message Key
        EncryptMgr-->>UI: Decrypted message
        UI-->>User: Display message
    else Authentication Failure (Tampered Data)
        CryptoKit-->>EncryptMgr: Error: GCM tag invalid
        EncryptMgr->>EncryptMgr: Log security incident
        EncryptMgr-->>UI: Error: Data corrupted
        UI-->>User: "Message corrupted or tampered"
    end
```

**Key Insights:**
- Authentication tag verification prevents tampering
- Same HKDF derivation yields same key (deterministic)
- Security incident logged if authentication fails

---

### 10. Component Dependencies

**Source:** `components/encryption-key-management.md`  
**Purpose:** How the Encryption Manager relates to other components.

```mermaid
graph TB
    EncryptMgr[Encryption Manager]
    VaultMgr[Vault Manager]
    LocalDB[Local SQLite]
    Keychain[iOS Keychain]
    SecureEnclave[Secure Enclave]
    CryptoKit[Apple CryptoKit]
    LocalAuth[Local Authentication]
    
    VaultMgr -->|Encrypt messages| EncryptMgr
    VaultMgr -->|Decrypt for display| EncryptMgr
    EncryptMgr -->|Store/retrieve keys| Keychain
    Keychain -->|Hardware-backed encryption| SecureEnclave
    EncryptMgr -->|Crypto primitives| CryptoKit
    EncryptMgr -->|Biometric auth| LocalAuth
    LocalAuth -->|Verify biometric| SecureEnclave
    
    style EncryptMgr fill:#FFD700
    style SecureEnclave fill:#90EE90
    style CryptoKit fill:#87CEEB
```

**Key Insights:**
- Encryption Manager central to security architecture
- Depends on Apple frameworks (CryptoKit, Keychain)
- Secure Enclave provides hardware-backed protection

---

## Usage Notes

### Diagram Rendering

All diagrams use **Mermaid syntax** and render inline in Markdown viewers that support Mermaid (GitHub, GitLab, VS Code with extensions, etc.).

**To render locally:**
```bash
# VS Code: Install "Markdown Preview Mermaid Support" extension
# CLI: Use mermaid-cli
npm install -g @mermaid-js/mermaid-cli
mmdc -i diagram.md -o diagram.png
```

### Color Coding Convention

Throughout all diagrams:
- **Green (#90EE90):** Trusted/User-controlled components (local storage, user devices)
- **Pink (#FFB6C1):** Cloud/Untrusted storage (S3, PostgreSQL)
- **Gold (#FFD700):** Compute/Processing (Lambdas, encryption engines)
- **Blue (#87CEEB):** Synchronization/Communication (CRDT, Redis)
- **Red (#FF6B6B):** Security warnings or terminated states

### Diagram Types Used

| Type | Count | Purpose |
|------|-------|---------|
| **Graph (TB/LR)** | 5 | System architecture, component relationships |
| **Sequence Diagram** | 5 | Data flows, interactions over time |
| **Total** | **10** | Complete system visualization |

---

## Diagram Source Files

Each diagram is maintained in its source document:
- Modify diagram → Update source file
- This gallery auto-updates by copying from sources

**Update Procedure:**
1. Edit diagram in source file (e.g., `02_architecture_overview.md`)
2. Test rendering locally
3. Copy updated diagram to this gallery
4. Update "Last Updated" date below

---

**Last Updated:** 04 October 2025  
**Diagram Count:** 10  
**Maintainer:** Architecture Team
