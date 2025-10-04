# References & Citations

## Purpose

This document compiles all sources, standards, and external documentation referenced throughout the Personal Data Vault technical documentation. Each reference includes:
- Direct URL (no link shorteners)
- Date last checked
- Brief description of relevance
- Document type (official spec, research paper, library documentation, etc.)

---

## Official Standards & Specifications

### Cryptography Standards

**NIST SP 800-38D: Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM)**  
**URL:** https://csrc.nist.gov/publications/detail/sp/800-38d/final  
**Date Checked:** 04 Oct 2025  
**Relevance:** Specification for AES-GCM authenticated encryption used for message encryption  
**Publisher:** National Institute of Standards and Technology (NIST)  
**Document Type:** Official Standard

**NIST SP 800-132: Recommendation for Password-Based Key Derivation**  
**URL:** https://csrc.nist.gov/publications/detail/sp/800-132/final  
**Date Checked:** 04 Oct 2025  
**Relevance:** Guidelines for PBKDF2 key derivation from user passphrases  
**Publisher:** NIST  
**Document Type:** Official Standard

**NIST SP 800-175B Rev. 1: Guideline for Using Cryptographic Standards**  
**URL:** https://csrc.nist.gov/publications/detail/sp/800-175b/rev-1/final  
**Date Checked:** 04 Oct 2025  
**Relevance:** Overall cryptographic best practices and algorithm selection  
**Publisher:** NIST  
**Document Type:** Official Guideline

**NIST SP 800-57 Part 1 Rev. 5: Recommendation for Key Management**  
**URL:** https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final  
**Date Checked:** 04 Oct 2025  
**Relevance:** Key lifecycle management, rotation policies, key sizes  
**Publisher:** NIST  
**Document Type:** Official Standard

**FIPS 180-4: Secure Hash Standard (SHS)**  
**URL:** https://csrc.nist.gov/publications/detail/fips/180/4/final  
**Date Checked:** 04 Oct 2025  
**Relevance:** SHA-256 hash function used in HKDF and Merkle trees  
**Publisher:** NIST  
**Document Type:** Federal Standard

### Internet Protocols

**RFC 5869: HMAC-based Extract-and-Expand Key Derivation Function (HKDF)**  
**URL:** https://datatracker.ietf.org/doc/html/rfc5869  
**Date Checked:** 04 Oct 2025  
**Relevance:** Per-message key derivation algorithm  
**Publisher:** IETF (Internet Engineering Task Force)  
**Document Type:** RFC Standard

**RFC 3501: INTERNET MESSAGE ACCESS PROTOCOL - VERSION 4rev1**  
**URL:** https://datatracker.ietf.org/doc/html/rfc3501  
**Date Checked:** 04 Oct 2025  
**Relevance:** IMAP protocol specification for email retrieval  
**Publisher:** IETF  
**Document Type:** RFC Standard

**RFC 2595: Using TLS with IMAP, POP3 and ACAP**  
**URL:** https://datatracker.ietf.org/doc/html/rfc2595  
**Date Checked:** 04 Oct 2025  
**Relevance:** STARTTLS encryption for IMAP connections  
**Publisher:** IETF  
**Document Type:** RFC Standard

**RFC 5545: Internet Calendaring and Scheduling Core Object Specification (iCalendar)**  
**URL:** https://datatracker.ietf.org/doc/html/rfc5545  
**Date Checked:** 04 Oct 2025  
**Relevance:** iCalendar format for parsing calendar invites from emails  
**Publisher:** IETF  
**Document Type:** RFC Standard

**RFC 5322: Internet Message Format**  
**URL:** https://datatracker.ietf.org/doc/html/rfc5322  
**Date Checked:** 04 Oct 2025  
**Relevance:** Email message structure and headers  
**Publisher:** IETF  
**Document Type:** RFC Standard

**RFC 2045-2049: Multipurpose Internet Mail Extensions (MIME)**  
**URL:** https://datatracker.ietf.org/doc/html/rfc2045  
**Date Checked:** 04 Oct 2025  
**Relevance:** Email multipart parsing for attachments and calendar invites  
**Publisher:** IETF  
**Document Type:** RFC Standard Series

### Alternative Cryptographic Algorithms (Evaluated but Not Chosen)

**RFC 8439: ChaCha20 and Poly1305 for IETF Protocols**  
**URL:** https://datatracker.ietf.org/doc/html/rfc8439  
**Date Checked:** 04 Oct 2025  
**Relevance:** Alternative to AES-GCM; evaluated but not chosen (AES-GCM faster on Apple hardware)  
**Publisher:** IETF  
**Document Type:** RFC Standard

**RFC 7914: The scrypt Password-Based Key Derivation Function**  
**URL:** https://datatracker.ietf.org/doc/html/rfc7914  
**Date Checked:** 04 Oct 2025  
**Relevance:** Alternative to PBKDF2; evaluated but not chosen (not in CryptoKit)  
**Publisher:** IETF  
**Document Type:** RFC Standard

**Argon2 Password Hashing Specification (v1.3)**  
**URL:** https://github.com/P-H-C/phc-winner-argon2/blob/master/argon2-specs.pdf  
**Date Checked:** 04 Oct 2025  
**Relevance:** Winner of Password Hashing Competition; evaluated but not chosen (no native support)  
**Publisher:** Password Hashing Competition  
**Document Type:** Specification

**NIST SP 800-38A: Recommendation for Block Cipher Modes of Operation**  
**URL:** https://csrc.nist.gov/publications/detail/sp/800-38a/final  
**Date Checked:** 04 Oct 2025  
**Relevance:** AES-CBC mode; evaluated but not chosen (GCM preferred for authenticated encryption)  
**Publisher:** NIST  
**Document Type:** Official Standard

---

## Research Papers & Academic Sources

**"A Comprehensive Study of Convergent and Commutative Replicated Data Types" (2011)**  
**Authors:** Marc Shapiro, Nuno Preguiça, Carlos Baquero, Marek Zawirski  
**URL:** https://hal.inria.fr/inria-00555588  
**Date Checked:** 04 Oct 2025  
**Relevance:** Foundational CRDT paper; theoretical basis for Automerge  
**Publisher:** INRIA (French National Research Institute)  
**Document Type:** Research Paper  
**Citations:** 2,400+ (highly influential)

**"Local-First Software: You Own Your Data, in Spite of the Cloud" (2019)**  
**Authors:** Martin Kleppmann, Adam Wiggins, Peter van Hardenberg, Mark McGranaghan  
**URL:** https://www.inkandswitch.com/local-first/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Design principles for local-first architecture; philosophical foundation  
**Publisher:** Ink & Switch Research  
**Document Type:** Research Essay  
**Impact:** Defined local-first movement; 1,000+ references

**"Efficient Data Structures for Tamper-Evident Logging" (2009)**  
**Authors:** Scott A. Crosby, Dan S. Wallach  
**URL:** https://www.usenix.org/legacy/event/sec09/tech/full_papers/crosby.pdf  
**Date Checked:** 04 Oct 2025  
**Relevance:** Merkle tree implementation for audit logs  
**Publisher:** USENIX Security Symposium  
**Document Type:** Conference Paper

---

## Open-Source Libraries & Tools

### Cryptography & Sync

**Automerge: A JSON-like data structure for building collaborative applications**  
**URL:** https://automerge.org/  
**GitHub:** https://github.com/automerge/automerge  
**Date Checked:** 04 Oct 2025  
**Relevance:** CRDT library for multi-device synchronization  
**License:** MIT  
**Language:** JavaScript (original), Swift port available  
**Stars:** 9,000+  
**Last Update:** Active (monthly releases)

**pgvector: Open-source vector similarity search for Postgres**  
**URL:** https://github.com/pgvector/pgvector  
**Date Checked:** 04 Oct 2025  
**Relevance:** PostgreSQL extension for vector storage and similarity search  
**License:** PostgreSQL License  
**Stars:** 8,500+  
**Last Update:** Active (weekly commits)  
**Performance Docs:** https://github.com/pgvector/pgvector#performance

**SQLCipher: SQLite Extension for Database Encryption**  
**URL:** https://www.zetetic.net/sqlcipher/  
**Documentation:** https://www.zetetic.net/sqlcipher/design/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Evaluated for database-level encryption (not chosen; per-message encryption preferred)  
**License:** BSD-style  
**Version:** 4.5+

### Message Platform Integration

**whatsmeow: WhatsApp Web Multi-Device API library in Go**  
**URL:** https://github.com/tulir/whatsmeow  
**Date Checked:** 04 Oct 2025  
**Relevance:** Library for WhatsApp integration via multi-device protocol  
**License:** Mozilla Public License 2.0 (MPL 2.0)  
**Stars:** 1,500+  
**Last Update:** Active (last commit <1 month)  
**Community:** Matrix room for support

**go-whatsapp (Archived - Predecessor to whatsmeow)**  
**URL:** https://github.com/Rhymen/go-whatsapp  
**Date Checked:** 04 Oct 2025  
**Relevance:** Original library; archived when WhatsApp released multi-device protocol  
**Status:** Deprecated (reference only)

**yowsup (Deprecated - WhatsApp Web Protocol)**  
**URL:** https://github.com/tgalal/yowsup  
**Date Checked:** 04 Oct 2025  
**Relevance:** Reverse-engineered WhatsApp protocol; deprecated after ToS violations  
**Status:** Archived (no longer works)  
**Lesson:** Why whatsmeow (official protocol) was chosen

**MailCore2: IMAP/SMTP library for macOS and iOS**  
**URL:** https://github.com/MailCore/mailcore2  
**Date Checked:** 04 Oct 2025  
**Relevance:** Library for IMAP email retrieval (supports OAuth)  
**License:** BSD  
**Stars:** 2,500+  
**Status:** Maintained

---

## Apple Platform Documentation

**Apple CryptoKit Framework**  
**URL:** https://developer.apple.com/documentation/cryptokit  
**Date Checked:** 04 Oct 2025  
**Relevance:** Official framework for AES-GCM, HKDF, PBKDF2, random generation  
**Publisher:** Apple Inc.  
**Availability:** iOS 13+, macOS 10.15+

**Apple Security Framework**  
**URL:** https://developer.apple.com/documentation/security  
**Date Checked:** 04 Oct 2025  
**Relevance:** Keychain access, certificate management, secure random  
**Publisher:** Apple Inc.

**Apple Local Authentication Framework**  
**URL:** https://developer.apple.com/documentation/localauthentication  
**Date Checked:** 04 Oct 2025  
**Relevance:** Touch ID / Face ID biometric authentication  
**Publisher:** Apple Inc.

**Apple Secure Enclave Overview**  
**URL:** https://support.apple.com/guide/security/secure-enclave-sec59b0b31ff/web  
**Date Checked:** 04 Oct 2025  
**Relevance:** Hardware security module for key storage  
**Publisher:** Apple Inc.

**Apple Platform Security Guide**  
**URL:** https://support.apple.com/guide/security/welcome/web  
**Date Checked:** 04 Oct 2025  
**Relevance:** Overall security architecture of iOS/macOS  
**Publisher:** Apple Inc.  
**Document Type:** Official Guide

**Apple Keychain Services**  
**URL:** https://developer.apple.com/documentation/security/keychain_services  
**Date Checked:** 04 Oct 2025  
**Relevance:** Secure storage for passwords and cryptographic keys  
**Publisher:** Apple Inc.

**Apple Certifications - iOS/iPadOS**  
**URL:** https://support.apple.com/guide/certifications/ios-ipados-apc3fa917cb59/web  
**Date Checked:** 04 Oct 2025  
**Relevance:** FIPS 140-2 certification for Secure Enclave  
**Publisher:** Apple Inc.  
**Document Type:** Certification

---

## Cloud Provider Documentation

### AWS

**AWS Lambda Developer Guide**  
**URL:** https://docs.aws.amazon.com/lambda/latest/dg/welcome.html  
**Date Checked:** 04 Oct 2025  
**Relevance:** Serverless compute for ephemeral embedding generation  
**Publisher:** Amazon Web Services

**AWS S3 Developer Guide**  
**URL:** https://docs.aws.amazon.com/s3/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Object storage for encrypted message backups  
**Publisher:** Amazon Web Services

**AWS S3 Service Level Agreement**  
**URL:** https://aws.amazon.com/s3/sla/  
**Date Checked:** 04 Oct 2025  
**Relevance:** 99.999999999% durability guarantee (11 nines)  
**Publisher:** Amazon Web Services  
**Document Type:** SLA

**AWS SQS Developer Guide**  
**URL:** https://docs.aws.amazon.com/sqs/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Message queue for asynchronous embedding tasks  
**Publisher:** Amazon Web Services

**AWS API Gateway Documentation**  
**URL:** https://docs.aws.amazon.com/apigateway/  
**Date Checked:** 04 Oct 2025  
**Relevance:** REST API routing for backend services  
**Publisher:** Amazon Web Services

### PostgreSQL

**PostgreSQL 15 Documentation**  
**URL:** https://www.postgresql.org/docs/15/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Database for vector storage with pgvector extension  
**Publisher:** PostgreSQL Global Development Group

### Redis

**Redis Documentation**  
**URL:** https://redis.io/docs/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Pub/sub for real-time sync coordination  
**Publisher:** Redis Ltd.

---

## AI & Machine Learning

**OpenAI Embeddings API Documentation**  
**URL:** https://platform.openai.com/docs/guides/embeddings  
**Date Checked:** 04 Oct 2025  
**Relevance:** text-embedding-3-small model for semantic search  
**Publisher:** OpenAI  
**Model:** 1536 dimensions, multilingual

**OpenAI Models Overview**  
**URL:** https://platform.openai.com/docs/models  
**Date Checked:** 04 Oct 2025  
**Relevance:** Comparison of embedding models  
**Publisher:** OpenAI

**Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks**  
**Paper URL:** https://arxiv.org/abs/1908.10084  
**Code URL:** https://github.com/UKPLab/sentence-transformers  
**Date Checked:** 04 Oct 2025  
**Relevance:** Alternative local embedding model (all-MiniLM-L6, 384 dimensions)  
**Publisher:** ArXiv / UKP Lab  
**License:** Apache 2.0

---

## Security Best Practices

**OWASP Password Storage Cheat Sheet**  
**URL:** https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html  
**Date Checked:** 04 Oct 2025  
**Relevance:** PBKDF2 iteration count recommendation (600,000 for 2023)  
**Publisher:** Open Web Application Security Project  
**Document Type:** Best Practice Guide

**OWASP Top Ten Web Application Security Risks**  
**URL:** https://owasp.org/www-project-top-ten/  
**Date Checked:** 04 Oct 2025  
**Relevance:** General security awareness for backend services  
**Publisher:** OWASP

**CWE-330: Use of Insufficiently Random Values**  
**URL:** https://cwe.mitre.org/data/definitions/330.html  
**Date Checked:** 04 Oct 2025  
**Relevance:** Why CSPRNG is required for nonce generation  
**Publisher:** MITRE Corporation

---

## Privacy & Legal

**W3C Encrypted Data Vaults Specification**  
**URL:** https://identity.foundation/edv-spec/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Standard for zero-knowledge data vaults  
**Publisher:** W3C Decentralized Identifier Foundation  
**Status:** Working Draft

**GDPR Official Text**  
**URL:** https://gdpr-info.eu/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Data protection requirements (right to access, deletion, portability)  
**Publisher:** European Union

**CCPA (California Consumer Privacy Act)**  
**URL:** https://oag.ca.gov/privacy/ccpa  
**Date Checked:** 04 Oct 2025  
**Relevance:** California data privacy law  
**Publisher:** California Attorney General

---

## Message Platform Official Resources

**WhatsApp Business API Documentation**  
**URL:** https://developers.facebook.com/docs/whatsapp  
**Date Checked:** 04 Oct 2025  
**Relevance:** Official API (evaluated but not used; requires business account)  
**Publisher:** Meta Platforms, Inc.

**WhatsApp Business API Pricing**  
**URL:** https://developers.facebook.com/docs/whatsapp/pricing  
**Date Checked:** 04 Oct 2025  
**Relevance:** Cost comparison ($0.005-$0.09 per message)  
**Publisher:** Meta Platforms, Inc.

**WhatsApp Export Chat Feature**  
**URL:** https://faq.whatsapp.com/1180414079177245/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Manual export alternative (evaluated but not chosen; poor UX)  
**Publisher:** WhatsApp LLC

**Signal Protocol Documentation**  
**URL:** https://signal.org/docs/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Technical specification for WhatsApp's encryption  
**Publisher:** Signal Foundation

**iMessage SQLite Database Analysis**  
**Title:** "Searching Your iMessage Database with SQL Queries"  
**URL:** https://spin.atomicobject.com/2020/05/22/search-imessage-sql/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Schema documentation for chat.db (reverse-engineered)  
**Publisher:** Atomic Object (Community Blog)  
**Document Type:** Technical Blog Post

---

## Password Security

**"Have I Been Pwned" - Common Passwords List**  
**URL:** https://haveibeenpwned.com/Passwords  
**Date Checked:** 04 Oct 2025  
**Relevance:** Dictionary of compromised passwords for passphrase validation  
**Publisher:** Troy Hunt  
**Dataset:** 850M+ compromised passwords

---

## Design & UX Resources

**Certificate Transparency (CT) Log Standard**  
**URL:** https://certificate.transparency.dev/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Inspiration for public audit log transparency  
**Publisher:** Google / IETF  
**Spec:** RFC 6962

**BIP39: Mnemonic Code for Generating Deterministic Keys**  
**URL:** https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki  
**Date Checked:** 04 Oct 2025  
**Relevance:** 12-word recovery phrase generation  
**Publisher:** Bitcoin Improvement Proposals

---

## Performance Benchmarking

**TPC-H Benchmark (Database Performance)**  
**URL:** https://www.tpc.org/tpch/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Industry-standard database benchmarking methodology  
**Publisher:** Transaction Processing Performance Council

---

## Community Resources & Tutorials

**"How End-to-End Encryption Works" (Visual Explainer)**  
**URL:** https://ssd.eff.org/module/deep-dive-end-end-encryption-how-do-public-key-encryption-systems-work  
**Date Checked:** 04 Oct 2025  
**Relevance:** Beginner-friendly explanation of E2EE concepts  
**Publisher:** Electronic Frontier Foundation (EFF)  
**Document Type:** Educational Article

**"A Stick Figure Guide to AES"**  
**URL:** https://www.moserware.com/2009/09/stick-figure-guide-to-advanced.html  
**Date Checked:** 04 Oct 2025  
**Relevance:** Accessible explanation of AES encryption  
**Publisher:** Jeff Moser (Community)  
**Document Type:** Educational Blog Post

---

## Tools & Development Resources

**Mermaid Diagram Syntax**  
**URL:** https://mermaid.js.org/intro/  
**Date Checked:** 04 Oct 2025  
**Relevance:** Diagram generation for all architecture visuals  
**License:** MIT  
**Document Type:** Official Documentation

---

## Deprecated / Historical References

**WhatsApp Web Protocol (Reverse-Engineered)**  
**Status:** Deprecated (WhatsApp blocked access)  
**Relevance:** Historical context for why whatsmeow (official protocol) is used  
**Note:** Included for completeness; not a recommended approach

---

## Citation Format

All references follow this format:
```
**[Title]**  
**URL:** [Direct link]  
**Date Checked:** DD Mon YYYY  
**Relevance:** [Why this source is cited]  
**Publisher:** [Organization]  
**Document Type:** [Standard/Paper/Guide/etc.]  
[Optional: License, Stars, Last Update for open-source]
```

---

## Update Policy

- **Quarterly Review:** All URLs verified every 3 months
- **Broken Links:** Replaced with archived version (Wayback Machine) or updated URL
- **New References:** Added with pull request, must include all required fields
- **Version Changes:** Document major version updates (e.g., NIST publishes new revision)

---

## How to Contribute

1. Add new reference to appropriate section
2. Follow citation format exactly
3. Verify URL is direct (no redirects/shorteners)
4. Update "Date Checked" to current date
5. Include brief relevance description

---

**Document Maintainer:** Documentation Team  
**Last Full Audit:** 04 October 2025  
**Next Scheduled Audit:** 04 January 2026  
**Total References:** 80+
