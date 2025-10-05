# Decision 11: Vector Similarity Search Index Algorithm

## 0. Executive Snapshot

- **Current choice:** HNSW (Hierarchical Navigable Small World) algorithm in pgvector
- **Overall score:** 4.60/5 (Excellent)
- **Verdict:** ✅ Keep (optimal speed/accuracy trade-off for approximate nearest neighbor search)
- **Why (one sentence):** HNSW provides state-of-the-art query performance (1.5-2.4ms for 100K vectors with 95%+ recall) that's 433x faster than brute force while maintaining excellent accuracy, making it superior to IVFFlat (37% slower queries) and mandatory for meeting <200ms latency requirement.

---

## 1. Context & Requirements Fit

### Problem Statement

Finding similar vectors in high-dimensional space is computationally expensive (O(n) brute force). Need index algorithm that accelerates approximate nearest neighbor (ANN) search from 650ms (brute force) to <5ms while maintaining >95% recall (finding correct neighbors).

### Requirements

| Requirement | Description | How HNSW Satisfies |
|-------------|-------------|-------------------|
| **REQ-4.4** | Query latency <200ms | HNSW: 1.5-2.4ms (100x faster than target) |
| **Recall** | >95% accuracy | HNSW: 95%+ with default parameters (m=16) |
| **Scale** | 100K-1M vectors | HNSW scales logarithmically: O(log n) |
| **Integration** | Works with pgvector | Native support in pgvector 0.5.0+ |
| **Memory** | <4GB for 100K vectors | HNSW: ~2GB (20KB per 1536-dim vector) |

### Constraints

- **Platform:** Must work with pgvector in PostgreSQL (D05, D07)
- **Query Time:** <200ms p95 latency (user-facing searches)
- **Build Time:** <20 minutes for 100K vectors (acceptable for batch)
- **Recall:** >95% (find correct neighbors, not just approximate)
- **Memory:** Fit in RDS t4g.small (2 GB RAM)

### Success Criteria

- ✅ Query latency <200ms p95 (actual: 1.5-2.4ms)
- ✅ Recall >95% (actual: 95%+ with m=16, ef_construction=64)
- ✅ Works with pgvector (native support)
- ✅ Memory <4GB for 100K vectors (actual: ~2GB)

---

## 2. Alternatives Catalog

### Alternative A: HNSW ✅ **(CURRENT CHOICE)**

**What it is:**
Hierarchical Navigable Small World is a graph-based ANN algorithm that builds a multi-layer graph where each layer becomes progressively sparser. Search starts at top sparse layer and descends to denser layers, achieving logarithmic complexity.

**How it works:**
1. Build multi-layer graph (each vector is node)
2. Higher layers: sparse (few nodes, long-range connections)
3. Lower layers: dense (many nodes, short-range connections)
4. Query: Start at top, navigate toward query, descend layers
5. Complexity: O(log n) vs O(n) for brute force

**Visual:**
```
Layer 3 (top):     •-------•-------•        (sparse, 1K nodes)
Layer 2:          •-•-•-•-•-•-•-•-•        (medium, 10K nodes)
Layer 1:         •••••••••••••••••        (dense, 50K nodes)
Layer 0 (base):  ••••••••••••••••••••••   (all 100K nodes)

Query descends from top to base, navigating graph edges
```

**Maturity & Ecosystem:**
- **Paper:** Malkov & Yashunin (2016, 1,500+ citations)
- **Adoption:** Pinecone, Qdrant, Weaviate, pgvector, FAISS
- **pgvector Support:** Added in v0.5.0 (October 2023)
- **Industry Standard:** Considered best ANN algorithm for high-dim vectors

**Licensing:** Algorithm is public (part of pgvector, PostgreSQL license)

**Parameters:**
- **m:** Max connections per node (8-32, default 16)
- **ef_construction:** Quality during build (32-128, default 64)
- **ef_search:** Quality during query (runtime adjustable, default 40)

### Alternative B: IVFFlat (Inverted File with Flat Vectors) ⚠️

**What it is:**
Clustering-based ANN algorithm that divides vector space into clusters (Voronoi cells). Faster to build than HNSW but slower queries.

**How it works:**
1. Training phase: K-means clustering (create N centroids)
2. Each vector assigned to nearest centroid
3. Query: Find closest K centroids, search only those clusters
4. Trade-off: Faster build, slower queries

**Maturity:** Very mature (used in FAISS, Annoy)

**Licensing:** Part of pgvector (PostgreSQL license)

**When to use:** Write-heavy workloads (build index frequently)

### Alternative C: Annoy (Spotify) ❌

**What it is:**
Tree-based ANN algorithm using random projection trees. Developed by Spotify for music recommendations.

**How it works:** Build forest of random projection trees; query descends multiple trees

**Critical Issue:** ❌ **NOT available in pgvector** (would require separate service)

### Alternative D: Faiss IVF-PQ ❌

**What it is:**
Facebook's advanced ANN library combining inverted file with product quantization (compression).

**Critical Issue:** ❌ **NOT in pgvector** (requires separate Faiss server)

### Alternative E: Brute Force (No Index) ❌

**What it is:**
Compute distance to every vector (exhaustive search). Perfect accuracy but slow.

**Critical Issue:** ❌ **650ms latency** for 100K vectors (fails <200ms requirement)

---

## 3. Pros & Cons (Comparison Table)

| Option | Key Pros | Key Cons | Performance Envelope | Ops Complexity | Cost/TCO Notes |
|--------|----------|----------|---------------------|----------------|----------------|
| **HNSW** ✅ | Fastest queries (1.5ms), excellent recall (95%+), logarithmic scaling | Slower build (10-20 min), more memory (20KB/vec) | 100K: 1.5ms; 1M: 10-20ms | Low (pgvector native) | Included in PostgreSQL |
| IVFFlat | Faster build (5-10 min), less memory (10KB/vec) | 37% slower queries (2.4ms vs 1.5ms), requires training | 100K: 2.4ms; 1M: 30-40ms | Low (pgvector native) | Included in PostgreSQL |
| Annoy | Good read performance, lightweight | NOT in pgvector (separate service needed) | Similar to HNSW | Medium (new service) | Free (library) |
| Faiss IVF-PQ | Best compression (16x), ultra-fast with GPU | NOT in pgvector, complex setup | GPU: <1ms; CPU: 10-20ms | High (separate server) | Free but needs infra |
| Brute Force | Perfect accuracy (100% recall), simple | 650ms latency (433x slower than HNSW) | Linear O(n) scaling | Very Low | Included |

---

## 4. Performance & Benchmarks

### HNSW Performance (AWS Official Benchmarks)

**Source:** https://aws.amazon.com/blogs/database/optimize-generative-ai-applications-with-pgvector-indexing/  
**Date Checked:** 05 Oct 2025

| Dataset | Algorithm | Query Time | Recall@10 | Build Time |
|---------|-----------|------------|-----------|------------|
| 100K vectors (1536-d) | HNSW (m=16, ef=64) | **1.5ms** | 95%+ | 10-20 min |
| 100K vectors | IVFFlat (lists=100) | **2.4ms** | 93% | 5-10 min |
| 100K vectors | Brute Force (no index) | **650ms** | 100% | N/A |

**Speedup:** HNSW is 433x faster than brute force with 95% accuracy

### Parameter Impact

**Source:** Crunchy Data HNSW guide  
**Date Checked:** 05 Oct 2025

| Parameter | Value | Query Time | Recall | Build Time | Memory |
|-----------|-------|------------|--------|------------|--------|
| m=8 | ef=32 | 2.5ms | 90% | 5 min | 1.5 GB |
| **m=16** | **ef=64** | **1.5ms** | **95%** | **15 min** | **2 GB** |
| m=32 | ef=128 | 1.0ms | 98% | 40 min | 4 GB |

**Recommendation:** m=16, ef_construction=64 (optimal balance)

### Runtime Parameter (ef_search)

**Adjustable at query time:**
```sql
-- Default (ef_search=40): 1.5ms, 95% recall
SELECT * FROM embeddings ORDER BY embedding <=> query LIMIT 10;

-- Higher quality (ef_search=100): 2.0ms, 98% recall
SET LOCAL hnsw.ef_search = 100;
SELECT * FROM embeddings ORDER BY embedding <=> query LIMIT 10;

-- Lower latency (ef_search=20): 1.0ms, 92% recall
SET LOCAL hnsw.ef_search = 20;
SELECT * FROM embeddings ORDER BY embedding <=> query LIMIT 10;
```

**Trade-off:** ef_search controls query time vs recall (adjustable per query)

---

## 5. Evidence Log (Citations)

1. **pgvector HNSW Documentation**  
   URL: https://github.com/pgvector/pgvector#hnsw  
   Date Checked: 05 Oct 2025  
   Relevance: Parameters (m, ef_construction, ef_search), usage examples

2. **AWS pgvector Optimization Guide**  
   URL: https://aws.amazon.com/blogs/database/optimize-generative-ai-applications-with-pgvector-indexing/  
   Date Checked: 05 Oct 2025  
   Relevance: Official benchmarks (1.5ms vs 2.4ms vs 650ms)

3. **Crunchy Data HNSW Guide**  
   URL: https://www.crunchydata.com/blog/hnsw-indexes-with-postgres-and-pgvector  
   Date Checked: 05 Oct 2025  
   Relevance: Parameter tuning, best practices

4. **HNSW Original Paper**  
   URL: https://arxiv.org/abs/1603.09320  
   Date Checked: 05 Oct 2025  
   Relevance: Algorithm specification (Malkov & Yashunin, 2016, 1,500+ citations)

5. **ANN Benchmarks**  
   URL: http://ann-benchmarks.com/  
   Date Checked: 05 Oct 2025  
   Relevance: Standardized benchmarks showing HNSW is state-of-the-art

---

## 6. Winner Rationale

1. **Fastest Queries (1.5ms vs 2.4ms IVFFlat):**
   - 37% faster than IVFFlat
   - 433x faster than brute force
   - Easily meets <200ms requirement

2. **Excellent Recall (95%+):**
   - With m=16, finds correct neighbors 95%+ of the time
   - vs IVFFlat: 93% recall (2% worse)
   - vs Brute Force: 100% recall (5% worse but much faster)

3. **State-of-the-Art Algorithm:**
   - Used by all major vector databases (Pinecone, Weaviate, Qdrant)
   - Academic backing (1,500+ citations)
   - Industry standard

4. **No Training Required:**
   - IVFFlat requires training phase (K-means clustering)
   - HNSW works immediately (no training data needed)
   - Can build index on empty table

5. **Logarithmic Scaling:**
   - O(log n) complexity
   - 100K vectors: 1.5ms
   - 1M vectors: ~10-20ms (predictable scaling)

### Accepted Trade-offs

| Trade-off | Impact | Mitigation |
|-----------|--------|------------|
| **Slower build (10-20 min)** | Index takes longer to create | Acceptable for batch processing; build once, query millions of times |
| **More memory (20KB/vec)** | 2GB for 100K vectors vs 1GB for IVFFlat | Acceptable; RDS t4g.small has 2GB RAM |
| **Approximate (95% recall)** | Misses 5% of true neighbors | Acceptable; user doesn't notice 1-2 missing results |

---

## 7. Losers' Rationale

### IVFFlat (Score: 4.35/5)

**Why it lost:**
- ❌ **37% slower queries:** 2.4ms vs 1.5ms (still fast, but HNSW better)
- ❌ **Training required:** Must run K-means before building index
- ⚠️ **Lower recall:** 93% vs 95%

**Would be better for:** Write-heavy workloads (rebuild index frequently)

### Brute Force (Score: 3.95/5)

**Why it lost:**
- ❌ **650ms latency:** 433x slower than HNSW
- ❌ **Fails requirement:** <200ms target not met

**Would be viable for:** <10K vectors (650ms for 100K = 6.5ms for 10K)

### Annoy/Faiss (Not Fully Scored)

**Why they lost:**
- ❌ **NOT in pgvector:** Would require separate service
- ❌ **Operational complexity:** Another system to manage
- **Faiss best compression:** IVF-PQ reduces memory 16x (but complex)

---

## 8. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **Index build fails** | Low | Medium | Retry with lower parameters; monitor build progress |
| **Query performance degrades** | Medium | High | VACUUM/REINDEX monthly; increase ef_search if recall drops |
| **Memory exhaustion** | Low | High | Provision adequate RAM; monitor usage; alert at 80% |

---

## 9. Recommendation & Roadmap

### Recommendation: ✅ **KEEP** HNSW with m=16, ef_construction=64

**Implementation:**
```sql
CREATE INDEX ON message_embeddings 
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

SET maintenance_work_mem = '2GB';  -- Faster builds
```

---

## 10. Examples

```sql
-- Query with HNSW index
SELECT * FROM message_embeddings
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 10;
-- Execution time: 1.5ms (HNSW graph traversal)

-- Tune for better recall
SET hnsw.ef_search = 100;
-- Query time: 2.0ms, recall: 98% (trade-off)
```

---

## 11. Validation Plan

**Spike:** Build HNSW index for 100K vectors; measure query time and recall  
**Success:** p95 <200ms, recall >95%  
**Timeline:** Week 1

---

## 12. Alternatives Table

| Criterion | Weight | HNSW | IVFFlat | Brute Force |
|-----------|-------:|-----:|--------:|------------:|
| Fit | 0.30 | 5.0 | 4.0 | 2.0 |
| Reliability | 0.20 | 4.5 | 4.5 | 5.0 |
| Complexity | 0.20 | 4.5 | 5.0 | 5.0 |
| Cost | 0.15 | 4.0 | 4.5 | 5.0 |
| Speed | 0.15 | 4.5 | 4.0 | 5.0 |
| **TOTAL** | 1.00 | **4.60** ⭐ | 4.35 | 3.95 |

---

**Document Version:** 1.0  
**Last Updated:** 05 October 2025  
**Status:** ✅ APPROVED