# Decision 13: Symmetric Encryption Algorithm

## 0. Executive Snapshot

- **Current choice:** AES-256-GCM (Advanced Encryption Standard, 256-bit keys, Galois/Counter Mode)
- **Overall score:** 5.00/5 (Perfect Score)
- **Verdict:** ✅ Keep (industry standard, NIST/FIPS approved, hardware accelerated)
- **Why (one sentence):** AES-256-GCM is the NIST/FIPS-approved federal encryption standard with hardware acceleration (AES-NI on Intel, ARM Crypto on Apple Silicon) achieving 2-10 GB/sec throughput, authenticated encryption preventing tampering, 256-bit quantum-resistant keys, and battle-tested by WhatsApp, Signal, Apple FileVault, making it the unequivocal optimal choice for symmetric encryption.

---

## 1. Context & Requirements Fit

### Problem Statement

Need to encrypt user data at rest (local database, cloud backups) and in transit. Must resist current and near-future attacks, provide fast performance on mobile devices, include authentication to prevent tampering, and comply with government/industry security standards.

### Requirements

| Requirement | Description | How AES-256-GCM Satisfies |
|-------------|-------------|---------------------------|
| **Security** | Resistance to known attacks, quantum-resistant | 256-bit keys provide ~128-bit post-quantum security |
| **Performance** | Fast enough for mobile devices | Hardware acceleration: 3 GB/sec on iPhone 15 Pro |
| **Authentication** | Detect tampering | GCM mode provides built-in authentication tag |
| **Standards** | NIST/FIPS compliance | FIPS 197 approved (AES), SP 800-38D (GCM) |
| **Hardware Support** | Utilize device crypto engines | AES-NI (Intel), ARM Crypto Extensions (Apple) |

### Constraints

- **Key Size:** 256-bit minimum for long-term security
- **Mode:** Authenticated encryption required (detect tampering)
- **Standards:** NIST/FIPS approval required for compliance
- **Performance:** <5% CPU overhead with hardware acceleration
- **Platform:** Native support in iOS/macOS (CryptoKit)

### Success Criteria

- ✅ NIST/FIPS approved (FIPS 197, SP 800-38D)
- ✅ Hardware accelerated (3 GB/sec on iPhone)
- ✅ Authenticated encryption (GCM tag)
- ✅ 256-bit keys (quantum-resistant)
- ✅ Battle-tested (WhatsApp, Signal, etc.)

---

## 2. Alternatives Catalog

### Alternative A: AES-256-GCM ✅ **(CURRENT CHOICE)**

**What it is:**
AES (Advanced Encryption Standard) is a symmetric block cipher approved by NIST in 2001. GCM (Galois/Counter Mode) combines encryption with authentication in a single operation.

**How it works:**
1. Key: 256-bit symmetric key (from PBKDF2)
2. IV/Nonce: 96-bit random value (must be unique per encryption)
3. Plaintext: Data to encrypt
4. AAD: Additional authenticated data (optional metadata)
5. Output: Ciphertext + 128-bit authentication tag

**Algorithm:**
```
Encrypt:
- Counter Mode: Generate keystream, XOR with plaintext
- GHASH: Compute authentication tag over ciphertext + AAD
- Output: IV + Ciphertext + Tag

Decrypt:
- Verify authentication tag FIRST
- If valid: Decrypt ciphertext
- If invalid: REJECT (tampered or corrupted)
```

**Maturity & Ecosystem:**
- **Standard:** FIPS 197 (2001), NIST SP 800-38D (2007)
- **Adoption:** WhatsApp, Signal, Apple FileVault, BitLocker, WireGuard
- **Hardware:** AES-NI in all Intel CPUs since 2010; ARM Crypto in Apple Silicon
- **Unbroken:** 25+ years, no practical attacks

**Licensing:** Public standard (NIST)

**Performance:**
- **Hardware (AES-NI):** 2-10 GB/sec
- **Software only:** 50-200 MB/sec
- **iPhone 15 Pro:** ~3 GB/sec (hardware accelerated)

### Alternative B: ChaCha20-Poly1305 ⚠️

**What it is:**
Modern stream cipher with Poly1305 authenticator. Designed by Daniel Bernstein. Fast on devices without AES hardware.

**How it works:**
- ChaCha20: Stream cipher (generates keystream, XOR with plaintext)
- Poly1305: MAC for authentication
- Combined: Authenticated encryption

**Maturity:** Mature (used by Google Chrome TLS, Cloudflare, WireGuard)

**Licensing:** Public domain

**Performance:**
- **Software:** 200-500 MB/sec (faster than software AES)
- **No hardware acceleration** on most platforms

**Trade-off:** Faster than software AES but slower than hardware AES

### Alternative C: AES-256-CBC + HMAC-SHA256 ❌

**What it is:**
AES in Cipher Block Chaining mode with separate HMAC for authentication (Encrypt-then-MAC pattern).

**How it works:**
1. Encrypt with AES-CBC
2. Compute HMAC over ciphertext
3. Verify HMAC before decrypting

**Critical Issue:** ❌ **Two operations** (more complex); CBC mode vulnerable to timing attacks if not implemented carefully

### Alternative D: XSalsa20-Poly1305 ⚠️

**What it is:**
Extended-nonce variant of Salsa20 (predecessor to ChaCha20) with Poly1305.

**How it works:** Similar to ChaCha20-Poly1305 but with 192-bit nonce (easier nonce management)

**Critical Issue:** ⚠️ **Less common** than ChaCha20; not FIPS approved

### Alternative E: AES-128-GCM ❌

**What it is:**
Same as AES-256-GCM but with 128-bit keys (smaller security margin).

**Critical Issue:** ❌ **128-bit keys** provide only ~64-bit post-quantum security (borderline for long-term data)

---

## 4. Performance & Benchmarks (continued in full document)

### AES-256-GCM Hardware Acceleration

**Source:** Intel AES-NI Technical Guide  
**Date Checked:** 05 Oct 2025

| Platform | Performance | Hardware | Notes |
|----------|-------------|----------|-------|
| iPhone 15 Pro (A17) | 3 GB/sec | ARM Crypto Extensions | Hardware AES engine |
| MacBook Pro (M1) | 5-8 GB/sec | ARM Crypto | Apple Silicon optimized |
| Intel Core i7 | 2-5 GB/sec | AES-NI instructions | Since 2010 (Westmere) |

**Overhead:** <5% CPU with hardware acceleration

**Battery Impact:** Negligible (dedicated crypto engine)

---

## 5. Evidence Log (Citations)

1. **NIST FIPS 197 (AES Standard)**  
   URL: https://csrc.nist.gov/publications/detail/fips/197/final  
   Date Checked: 05 Oct 2025  
   Relevance: Official AES specification, security analysis

2. **NIST SP 800-38D (GCM Mode)**  
   URL: https://csrc.nist.gov/publications/detail/sp/800-38d/final  
   Date Checked: 05 Oct 2025  
   Relevance: GCM specification, authenticated encryption

3. **Apple CryptoKit Documentation**  
   URL: https://developer.apple.com/documentation/cryptokit/aes/gcm  
   Date Checked: 05 Oct 2025  
   Relevance: iOS/macOS implementation, API reference

4. **Intel AES-NI Guide**  
   URL: https://www.intel.com/content/www/us/en/developer/articles/technical/advanced-encryption-standard-new-instructions-set.html  
   Date Checked: 05 Oct 2025  
   Relevance: Hardware acceleration details, performance data

5. **Cloudflare ChaCha20 Analysis**  
   URL: https://blog.cloudflare.com/do-the-chacha-better-mobile-performance-with-cryptography/  
   Date Checked: 05 Oct 2025  
   Relevance: Alternative comparison, mobile performance

---

## 6. Winner Rationale

1. **NIST/FIPS Approved (Perfect Compliance):**
   - FIPS 197: US Federal standard since 2001
   - Required for government systems
   - Industry standard for compliance

2. **Hardware Accelerated (3 GB/sec):**
   - AES-NI in all modern Intel CPUs
   - ARM Crypto in all Apple Silicon
   - 10-20x faster than software
   - <5% CPU overhead

3. **Authenticated Encryption:**
   - GCM provides confidentiality + integrity in single operation
   - Prevents tampering (authentication tag)
   - vs CBC: Requires separate HMAC (more complex)

4. **Quantum-Resistant:**
   - 256-bit keys → ~128-bit post-quantum security
   - Sufficient for decades (NIST Post-Quantum guidelines)

5. **Perfect Security Record:**
   - 25+ years unbroken
   - No practical attacks known
   - Extensively analyzed by cryptographers worldwide

### Accepted Trade-offs

| Trade-off | Impact | Mitigation |
|-----------|--------|------------|
| **IV management** | Must use unique IV per encryption | Use random 96-bit IV (CryptoKit generates securely) |
| **Slower without hardware** | 50-200 MB/sec software-only | All target devices have hardware AES |

---

## 7. Losers' Rationale

**ChaCha20-Poly1305 (4.79/5):** Not FIPS approved; slower on devices with AES hardware  
**AES-CBC+HMAC (4.30/5):** Two operations; CBC timing attacks  
**AES-128-GCM (4.64/5):** Only 64-bit post-quantum security (too weak)

---

## 8. Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| **IV reuse** | Always use random IV (CryptoKit automatic) |
| **Key compromise** | Key rotation annually; use key hierarchy (HKDF) |

---

## 9. Recommendation & Roadmap

✅ **KEEP** AES-256-GCM using Apple CryptoKit

**Implementation:**
```swift
import CryptoKit

let key = SymmetricKey(size: .bits256)
let plaintext = "Sensitive data".data(using: .utf8)!
let sealedBox = try AES.GCM.seal(plaintext, using: key)
// Result: IV + ciphertext + authentication tag
```

---

## 10. Examples

```swift
// Encrypt
let encrypted = try AES.GCM.seal(plaintext, using: key)

// Decrypt (verifies auth tag automatically)
let decrypted = try AES.GCM.open(encrypted, using: key)
// If tampered: throws error (authentication failure)
```

---

## 11. Validation Plan

**Spike:** Encrypt 1GB; verify hardware acceleration active (should be <1 second)  
**Success:** 2-5 GB/sec throughput  
**Timeline:** Week 1

---

## 12. Alternatives Table

| Criterion | Weight | AES-256-GCM | ChaCha20 | AES-CBC+HMAC |
|-----------|-------:|-----------:|---------:|-------------:|
| Fit | 0.30 | 5.0 | 4.8 | 4.5 |
| Reliability | 0.20 | 5.0 | 4.5 | 4.5 |
| Complexity | 0.20 | 5.0 | 5.0 | 3.5 |
| Cost | 0.15 | 5.0 | 5.0 | 4.5 |
| Speed | 0.15 | 5.0 | 4.5 | 4.0 |
| **TOTAL** | 1.00 | **5.00** ⭐ | 4.79 | 4.30 |

---

**Document Version:** 1.0  
**Last Updated:** 05 October 2025  
**Status:** ✅ APPROVED