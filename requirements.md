# Project Requirements and Deliverables

Based on the project.md file, here are the **precise requirements and deliverables**:

## Final Deliverables

1. **Written Report (PDF format)**
   - Length: Approximately 3 pages
   - Must cover both Part 1 and Part 2
   - Required sections for each part:
     - Your approach and design decisions
     - Tools, techniques, libraries, or models that can be used
     - Trade-offs considered
     - Diagrams or visualizations to support your explanation (where appropriate)
   - Assumptions must be supported by relevant material (research papers, blog posts, notebooks, code examples)

2. **Presentation**
   - Showcase your solution
   - Can be delivered online or in-person
   - To be scheduled at a mutually convenient time

## Part 1 Requirements: Architecture for Message Gathering and Processing

**Must address:**
1. **Architecture diagram** outlining key components and data flow
2. **User onboarding description** for connecting to each platform:
   - WhatsApp (QR code scanning, Signal Protocol)
   - iMessage (SQLite database access on macOS)
   - IMAP (credential-based email and calendar access)
3. **Discussion of challenges:**
   - Authentication
   - Transparency
   - Privacy
   - Reliability
4. **Adherence to design principles:**
   - User control over data storage location
   - Data isolation (user-only access)
   - Multi-device synchronization
   - Privacy-preserving compute for long-running tasks
   - Safe third-party integration (LLMs)

## Part 2 Requirements: Semantic Indexing and Matching Methodology

**Must provide:**
1. **High-level method** to index messages by content for:
   - Semantic search
   - Relationship discovery
2. **Algorithm proposal** that can:
   - Find all related messages for a given calendar event
   - Relate messages and events based on:
     - Content
     - Participants
     - Timestamps
     - Other metadata
   - Match messages across platforms (e.g., WhatsApp chats to calendar invites)
3. **Technical approach description:**
   - Data extraction methodology
   - Semantic analysis approach
4. **AI/ML technologies specification:**
   - NLP techniques
   - Embeddings
   - Vector databases
5. **Brief explanation** of challenges in handling heterogeneous data
6. **Adherence to design principles** (same as Part 1)

## Explicit Constraints

- **No working prototype required** - Focus is on design, technical decisions, and trade-offs, NOT code
- Must follow all 5 core design principles throughout
- All assumptions must be backed by references/supporting material

## Summary Checklist

- [ ] 3-page PDF report
- [ ] Architecture diagram with components and data flow
- [ ] User onboarding flows for WhatsApp, iMessage, and IMAP
- [ ] Challenge analysis (authentication, transparency, privacy, reliability)
- [ ] Semantic indexing methodology
- [ ] Algorithm for message-event relationship discovery
- [ ] AI/ML technology stack specification
- [ ] Heterogeneous data handling explanation
- [ ] Trade-offs analysis
- [ ] Supporting diagrams/visualizations
- [ ] References to research papers/blog posts/examples
- [ ] Presentation slides/materials
- [ ] All design principles addressed
