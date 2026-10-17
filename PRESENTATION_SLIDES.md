# 🛡️ ArchGuard — Hackathon Presentation Slide Deck

**Event:** WeMakeDevs x FalkorDB "Graph Hacks" Hackathon  
**Track:** Company Brain & Episodic Memory  
**Presenter:** SK Faizanuddin  
**Project:** ArchGuard — The Pre-Merge Architectural Guardian for Autonomous Engineering Teams

---

## 📊 Slide Deck Structure (10 Slides)

---

### **Slide 1: Title & Hook**
- **Title:** ArchGuard 🛡️
- **Subtitle:** The Pre-Merge Architectural Guardian for Autonomous Engineering Teams
- **Tagline:** *"Linters check your syntax. ArchGuard protects your architecture."*
- **Track:** Company Brain & Episodic Memory Track
- **Presenter:** SK Faizanuddin
- **Visuals:** Project Logo, Hackathon Badges, Hero Landing Page Screenshot ([`ArchGuard01.png`](screenshots/ArchGuard01.png))

---

### **Slide 2: The Core Problem — Silent Architectural Drift**
- **Headline:** A 1-Line Protobuf Change Can Brisk Downstream Services
- **The Friday 4:15 PM Outage Scenario:**
  - Developer changes `user_id` from `UUIDv4` string to `int64` for faster B-tree indexing in `auth-service`.
  - **Unit tests pass:** Protobuf compiles cleanly.
  - **Syntax passes:** Local types are valid.
  - **AI review bots comment:** *"Looks great! Sequential IDs reduce index fragmentation."*
  - **20 Minutes Later:** `billing-service` and `checkout-service` crash 3 hops downstream because `ADR-001` mandated UUIDs two years ago.
- **The Core Issue:** Linters only see syntax; AI bots only see isolated files. Neither understands company connections.

---

### **Slide 3: Why Vector RAG & Giant Prompts Fail**
- **Headline:** Architecture is an Invariant Graph, Not a Similarity Search
- **Failure 1: Vector Search is Non-Deterministic**
  - Embeddings measure semantic closeness, not binary policy compliance.
  - You cannot ask a vector database: *"Does this protobuf change violate ADR-001?"* with mathematical certainty.
- **Failure 2: Blindness to Transitive Blast Radius**
  - Microservice dependencies form multi-hop directed graphs (`SchemaField ➔ Service ➔ Downstream Dependency ➔ Endpoint ➔ Team Lead`).
  - Vector stores cannot follow graph reachability.
- **Failure 3: Context Window Degradation**
  - Shoving 50 ADRs, 40 OpenAPI specs, and team rosters into every LLM prompt is slow, expensive, and unreliable.

---

### **Slide 4: The Solution — Deterministic Detection, LLM-Explained**
- **Headline:** Decoupling Mathematical Traversal from Language Synthesis
- **The Latency Split:**
  - ⚡ **FalkorDB Graph Engine:** Executes multi-hop Cypher queries in **`<15ms`** with zero hallucinations.
  - ⏱️ **LLM Synthesis Engine:** Generates natural language summaries and compliant migration diffs in **`~1.2s`**.
- **Key Takeaway:** The graph does the heavy lifting; the LLM explains the deterministic proof.

---

### **Slide 5: The 3-Agent Graph-Mediated Architecture**
- **Headline:** Autonomous Coordination via Shared Graph State (Zero Prompt Chaining)
- **Agent 1 (Spec Extractor):**
  - Parses Git diff, OpenAPI, or SQL changes.
  - Writes `:Proposal {status: 'PARSED'}` and `[:TARGETS_FIELD]` into FalkorDB.
- **Agent 2 (Graph Auditor):**
  - Receives **only `proposalId`**.
  - Runs Cypher traversal in <15ms to find ADR collisions and downstream consumer blast radius.
  - Mutates graph: creates `[:VIOLATES_ADR]` and `[:BREAKS_CONSUMER]` edges; sets `status = 'CHECKED'`.
- **Agent 3 (Advisor Memory):**
  - Receives **only `proposalId`**.
  - Recalls historical incident workarounds (`:ArchitecturalEpisode`) from FalkorDB.
  - Synthesizes compliant migration plan; sets `status = 'RESOLVED'`.
- **Architectural Edge:** No prompt concatenation. FalkorDB is the single source of truth.

---

### **Slide 6: The FinCore Domain Knowledge Graph**
- **Headline:** Enterprise Fintech Topology Grounded in FalkorDB
- **Canonical Baseline:** **28 Pristine Nodes • 35 Directed Relationships**
  - 🟡 **Golden Rules & Contracts:** 4 `:ADR` nodes (`ADR-001` to `004`), 3 `:SchemaField` nodes.
  - 🟢 **Organizational Hierarchy:** 4 `:Team` nodes, 5 `:Person` nodes (Tech Leads).
  - 🔵 **Distributed Infrastructure:** 5 `:Service` nodes, 5 `:Endpoint` nodes.
  - 🔴 **Transient Proposal State:** Dynamic `:Proposal` nodes created on-the-fly.
  - 🧠 **Cross-Session Episodic Memory:** 2 `:ArchitecturalEpisode` nodes (`INC-2023-42`, `INC-2023-88`).
- **Visual:** Constellation Canvas Split View Screenshot ([`ArchGuard02.png`](screenshots/ArchGuard02.png))

---

### **Slide 7: Cross-Session Episodic Memory Learning**
- **Headline:** Preventing Teams from Having the Same Argument Twice
- **How Episodic Memory Works:**
  - When an outage forces a deliberate architectural compromise, engineers commit an `:ArchitecturalEpisode` linked via `[:APPLIES_TO]` to the violated ADR.
  - Captures: Incident ID, approved workaround pattern, approving lead (`@sarah_arch`), and technical trade-offs.
- **Cross-Session Recall in Action:**
  - Auditing PR-409 immediately surfaces **INC-2023-42**: *"Dual-Write Shadow Column Pattern for User UUID Migration"*.
  - Teams receive proven, approved solutions instead of hallucinated advice.
- **Visual:** Memory & Plan Tab Screenshot ([`ArchGuard04.png`](screenshots/ArchGuard04.png))

---

### **Slide 8: Real-World Engineering Insights**
- **Headline:** Production Polish & Lessons Learned
- **1. Blast Radius Deduplication:**
  - Naive path counting reported 5 broken services when `billing-service` was hit via two endpoints.
  - Refactored Cypher grouping to report the exact 3 distinct Tier-1 microservices.
- **2. Zero-Violation Fast Path:**
  - When code conforms to policies, Agent 2 detects 0 violations in 23ms, bypassing LLM synthesis entirely and turning the proposal emerald green.
- **3. Bulletproof Offline Heuristic Fallback:**
  - Deterministic AST pattern matcher ensures the system runs with 0.0s LLM time if external APIs hit rate limits. Guaranteed zero demo crashes.

---

### **Slide 9: Hackathon Track Alignment — Company Brain**
- **Headline:** Purpose-Built for the Company Brain & Episodic Memory Track
- **Track Criteria Checklist:**
  - [x] **Connected Enterprise Context:** Models decisions (ADRs), services, endpoints, schemas, and team ownership.
  - [x] **Episodic Memory Across Sessions:** Historical workarounds persist and inform future PR audits.
  - [x] **Multi-Agent State Sharing:** 3 agents read and write directly to FalkorDB subgraphs.
  - [x] **Explainable Decisions:** Every violation includes the exact Cypher path proving why it failed.
  - [x] **Sub-15ms Traversal:** Proven in-memory speed over vector RAG.

---

### **Slide 10: Conclusion & Call to Action**
- **Summary:**
  - *"Linters check your syntax. ArchGuard protects your architecture."*
  - Fast, deterministic, and grounded in real organizational memory.
- **Try It Locally:**
  - `docker compose up -d`
  - `npm run dev:all`
- **Links:**
  - 💻 **GitHub:** [`https://github.com/SKfaizan-786/ArchGuard`](https://github.com/SKfaizan-786/ArchGuard)
  - ✍️ **Dev.to Flagship Article:** Published for the Best Blog Post prize!
  - 🎥 **Demo Video:** 2.5-minute walkthrough
- **Thank You!** (Questions & Feedback)
