## Assignment: Personal Data Vault 

### Case Study: 

The user that we wish to assist is a busy professional who communicates across a multitude of platforms (WhatsApp, iMessage, email, calendar). This communication is fragmented, making it difficult to maintain an overview and see connections. The user wants a system that centralizes all communication in a secure, private manner and proactively offers insights by relating messages and appointments with each other. How should data be collected, processed and stored over a software platform that is installed on multiple personal Apple devices (iPhone, MacBook, iPad), assisted by a cloud backend? Propose a solution that aggregates and annotates all messages and calendar events into a secure Personal Data Vault, which is accessible across the user’s Apple devices—iPhone, MacBook, and iPad. 

In this assignment we wish you to follow these core design principles: 

* The user should be in control of where its data is stored: on a local device or with a trusted third party (iCloud, Google Drive, etc). 

* The user's data should remain isolated, i.e. only accessible by the user itself and not by us or any other user. 

* Data must remain synchronized across multiple devices, such that the user has a consistent experience. 

* Some compute tasks may take a long time to process, especially for a backlog of data. How can such a task be executed in a privacy-preserving way, while remaining transparent to the user? 

* Third parties don't need direct access to the Personal Data Vault. If the user needs functionality by third parties (e.g. LLM by OpenAI or Anthropic) to process messages, then this should be done a safe and transparent manner. 

### The Assignment: 

#### Part 1: Architecture for Message Gathering and Processing 

Describe how to gather and process messages from WhatsApp, iMessage, and IMAP (including email and calendar invites) into a Personal Data Vault available on all devices. The system analyzes user messages and relates them to calendar events, saving them as annotations in the Personal Data Vault. Each message platform has its own data retrieval protocol: 

* WhatsApp uses the Signal Protocol and has all messages stored on your smartphone, not on a central server. You can retrieve these messages after the user has given permission by scanning a QR code from the WhatsApp application on the phone. (Open Source implementation for this: https://github.com/tulir/whatsmeow) 

* Messages received through iMessage are stored on the user’s iPhone but are not directly accessible from the device. These messages are also synchronized with MacOS devices when logged in with iCloud. The messages are kept locally on MacOS devices in an SQLite database file. 

* Email messages, including calendar invitations, are stored on an IMAP server. Users can access and retrieve these messages from the server using their credentials. 

 
_What we expect for part 1:_

* Provide an architecture diagram outlining key components and data flow. 

* Describe user onboarding for connecting to each platform. 

* Discuss potential challenges, such as authentication, transparency, privacy, and reliability. 

* Adhere to our design principles. 

 
#### Part 2: Semantic Indexing and Matching Methodology  

Outline a high-level method to index messages by content for semantic search and relationship discovery. Propose an algorithm that, for a calendar event, can find all related messages. Relate messages and events based on their content, participants, timestamps, or other metadata. The method should be able relate messages across platforms, such as matching WhatsApp chats to calendar invites. 

_What we expect for part 2:_

* A description of the technical approach (e.g., data extraction, semantic analysis). 

* What specific AI/ML technologies you consider using (e.g., NLP techniques, embeddings, vector databases). 

* A brief explanation of the challenges in handling heterogeneous data. 

* Follow our design principles. 
 

#### Deliverables: 

* A single written report in PDF format (approximately 3 pages) describing your approach for the challenges in Part 1 and Part 2. Include the following for each part:  

  * Your approach and design decisions   

  * Tools, techniques, libraries, or models that can be used

  * Trade-offs considered   

  * Diagrams or visualizations to support your explanation (where appropriate) 

* A presentation in which you showcase your solution. 

Assumptions should be supported by relevant material, such as references to research papers, blog posts, notebooks, or code examples. Don't build a working prototype. Focus on design, technical decisions, and trade-offs — not code. 

We look forward to the presentation of your approach to above challenges, which may be delivered either online or in person. An appointment for the presentation will be scheduled at a mutually convenient time. 