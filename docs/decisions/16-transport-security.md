# Decision 16: Network Transport Security Protocol

## 0. Executive Snapshot

- **Current choice:** TLS 1.3 (Transport Layer Security version 1.3)
- **Overall score:** 5.00/5 (Perfect Score)
- **Verdict:** ✅ Keep (industry standard, mandatory for all services)
- **Why (one sentence):** TLS 1.3 (RFC 8446) is the universal industry standard with 1-RTT handshake (50% faster than TLS 1.2), Perfect Forward Secrecy protecting past communications, mandatory AEAD cipher suites, automatic support in iOS URLSession, and required by all services (AWS, OpenAI, Google), making it the only viable option for network transport security.

---

## 1. Context & Requirements Fit

### Problem Statement

Need to protect data in transit between clients and servers (Lambda, PostgreSQL, S3, OpenAI API). Must prevent eavesdropping, tampering, and man-in-the-middle attacks while maintaining acceptable latency and universal compatibility.

### Requirements

| Requirement | Description | How TLS 1.3 Satisfies |
|-------------|-------------|----------------------|
| **Standard** | IETF RFC compliant | RFC 8446 (2018), universal adoption |
| **Performance** | Low latency overhead | 1-RTT handshake (50% faster than 1.2) |
| **Forward Secrecy** | Past traffic secure | Ephemeral Diffie-Hellman mandatory |
| **Authentication** | AEAD encryption only | GCM, ChaCha20-Poly1305 required |
| **Compatibility** | All services support | AWS, OpenAI, Google all require TLS 1.2+ |

### Success Criteria

- ✅ RFC 8446 compliant
- ✅ 1-RTT handshake (<100ms)
- ✅ Perfect Forward Secrecy
- ✅ AEAD cipher suites only
- ✅ Automatic in iOS URLSession

---

## 2. Alternatives Catalog

### Alternative A: TLS 1.3 ✅ **(CURRENT)**

**What it is:**
Transport Layer Security version 1.3 (RFC 8446, 2018). Latest standard for encrypted network communications.

**How it works:**
1. Client sends ClientHello + KeyShare (includes public key)
2. Server responds with ServerHello + KeyShare + Encrypted data (all in 1-RTT)
3. Session keys established; encrypted communication begins
4. Ephemeral keys (Diffie-Hellman) provide Perfect Forward Secrecy

**Maturity:** Mature (RFC 2018, universal adoption by 2025)

**Licensing:** Public standard (IETF)

### Alternative B: TLS 1.2 ⚠️

**What it is:**
Previous TLS version (RFC 5246, 2008). Still secure if configured properly but slower.

**How it works:** 2-RTT handshake (1 extra round trip vs TLS 1.3)

**Trade-off:** Slower connection; optional weak cipher suites

### Alternative C: mTLS (Mutual TLS) ❌

**What it is:**
TLS with client certificates (both-way authentication).

**Critical Issue:** ❌ **Over-engineered** (client certificates unnecessary for Personal Vault)

### Alternative D: WireGuard VPN ❌

**What it is:**
Modern VPN protocol.

**Critical Issue:** ❌ **Not HTTP-compatible** (would require VPN infrastructure)

---

## 3. Pros & Cons (Comparison Table)

| Option | Key Pros | Key Cons | Performance Envelope | Ops Complexity | Cost/TCO Notes |
|--------|----------|----------|---------------------|----------------|----------------|
| **TLS 1.3** ✅ | Fastest (1-RTT), mandatory, PFS, AEAD, standard | None | <100ms handshake | Very Low (automatic) | Free (standard) |
| TLS 1.2 | Still secure, universal | Slower (2-RTT), optional weak ciphers | <150ms handshake | Low | Free |
| mTLS | Strongest auth, zero-trust | Complex cert management, overkill | Similar to TLS 1.3 | High (certs) | Free |
| WireGuard | Very fast, modern | Not HTTP, requires VPN | <50ms handshake | Medium (VPN) | Free |

---

## 4. Performance & Benchmarks

### TLS 1.3 Performance

**Source:** RFC 8446  
**Date Checked:** 05 Oct 2025

| Metric | TLS 1.3 | TLS 1.2 | Improvement |
|--------|---------|---------|-------------|
| **Handshake** | 1-RTT | 2-RTT | 50% faster |
| **Latency** | 50-100ms | 100-150ms | 33-50% faster |
| **0-RTT Resume** | Yes | No | Instant reconnect |

---

## 5. Evidence Log (Citations)

1. **RFC 8446 (TLS 1.3)**  
   URL: https://www.rfc-editor.org/rfc/rfc8446.html  
   Date Checked: 05 Oct 2025

2. **Apple Transport Security**  
   URL: https://developer.apple.com/documentation/security/preventing_insecure_network_connections  
   Date Checked: 05 Oct 2025

3. **Cloudflare TLS 1.3 Analysis**  
   URL: https://blog.cloudflare.com/rfc-8446-aka-tls-1-3/  
   Date Checked: 05 Oct 2025

---

## 6. Winner Rationale

1. **Industry Standard:** RFC 8446, universal adoption
2. **Fastest:** 1-RTT (50% faster than TLS 1.2)
3. **Perfect Forward Secrecy:** Ephemeral keys protect past traffic
4. **Mandatory:** All services require TLS 1.2+ minimum
5. **Automatic:** iOS URLSession uses TLS 1.3 by default

---

## 7. Losers' Rationale

**TLS 1.2:** Slower (2-RTT), optional weak ciphers  
**mTLS:** Over-engineered (client certs unnecessary)  
**WireGuard:** Not HTTP-compatible

---

## 8. Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| **Certificate expiration** | Automatic renewal (Let's Encrypt) |
| **MITM attacks** | Certificate validation automatic |

---

## 9. Recommendation & Roadmap

✅ **KEEP** TLS 1.3 (automatic in iOS, zero configuration needed)

---

## 10. Examples

```swift
// TLS 1.3 automatic
let url = URL(string: "https://api.openai.com")!
URLSession.shared.dataTask(with: url) { data, response, error in
    // Connection secured with TLS 1.3 automatically
}
```

---

## 11. Validation Plan

**Spike:** Verify TLS 1.3 active (Wireshark packet capture)  
**Success:** All connections use TLS 1.3  
**Timeline:** Week 1

---

## 12. Alternatives Table

| Criterion | Weight | TLS 1.3 | TLS 1.2 | mTLS |
|-----------|-------:|--------:|--------:|-----:|
| Fit | 0.30 | 5.0 | 4.5 | 4.0 |
| Reliability | 0.20 | 5.0 | 5.0 | 4.5 |
| Complexity | 0.20 | 5.0 | 5.0 | 2.5 |
| Cost | 0.15 | 5.0 | 5.0 | 4.0 |
| Speed | 0.15 | 5.0 | 4.0 | 3.5 |
| **TOTAL** | 1.00 | **5.00** ⭐ | 4.73 | 3.68 |

---

**Document Version:** 1.0  
**Last Updated:** 05 October 2025  
**Status:** ✅ APPROVED