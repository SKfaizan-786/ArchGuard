# 🛡️ ArchGuard — 2.5-Minute Demo Video Script

**Hackathon:** WeMakeDevs x FalkorDB "Graph Hacks"  
**Track:** Company Brain & Episodic Memory  
**Target Duration:** 2 minutes 30 seconds – 2 minutes 45 seconds  
**Screen Setup:** 1080p full-screen browser at `http://localhost:5173` (with split-view active).

---

## ⏱️ Video Timeline Overview

| Time | Scene | On-Screen Action | Key Message |
|:---:|---|---|---|
| **0:00 – 0:25** | The Problem Hook | Landing page hero widget / Slide | Linters check syntax; ArchGuard protects architecture. |
| **0:25 – 0:50** | The Company Brain Baseline | Command Center (`28 Nodes • 35 Edges`) | FalkorDB models ADRs, services, teams, and endpoints. |
| **0:50 – 1:25** | Auditing PR-409 (Violation) | Click `409 • Schema` ➔ `Run 3-Agent Audit` | Sub-15ms Cypher collision + 3 downstream services broken. |
| **1:25 – 1:55** | Episodic Memory Recall | Click `Memory & Plan` tab | Recalls INC-2023-42 dual-write precedent approved by leads. |
| **1:55 – 2:20** | Compliant Fix & Fast Path | Change `int64` to `string` ➔ Re-audit | Green glow (`:Proposal Approved`), status flips, 0 blast radius. |
| **2:20 – 2:40** | Live ADR Ingestion | Click `+ Ingest ADR` ➔ Ingest ADR-005 | Live graph compilation (counter increments 28 ➔ 29). |
| **2:40 – 2:55** | Wrap-Up & Pitch | GitHub Repo / Summary slide | Deterministic Detection, LLM-Explained. |

---

## 🎬 Word-for-Word Narration & Action Cue Script

### **SCENE 1: The Hook & The Problem (0:00 – 0:25)**
- **Video Visual:** Start on the ArchGuard Landing Page (`http://localhost:5173`) showing the hero headline: *"The Pre-Merge Guardian for Autonomous Teams"* and the animated graph widget.
- **Mouse Action:** Slowly hover over the interactive graph node on the right showing `CYPHER Traversal Latency: 0.8ms`.
- **Spoken Voiceover:**
  > *"Every backend engineer knows the Friday afternoon nightmare: a developer pushes a PR changing a single protobuf or API field. Unit tests pass, linters pass, and AI review bots say 'looks good!'*  
  > *Twenty minutes later, billing and checkout silently collapse 3 hops downstream.*  
  > *Linters only check your syntax. ArchGuard protects your architecture."*

---

### **SCENE 2: The FalkorDB Company Brain Baseline (0:25 – 0:50)**
- **Video Visual:** Click **`Launch Command Center`** to open the 3-pane cockpit.
- **Mouse Action:** 
  1. Point cursor to the top header badge: `archguard_brain | 28 Nodes • 35 Edges`.
  2. Briefly hover over a node on the left (`ADR-001`), then hover over a node on the right (`transaction_id` or `INC-2023-88`) to show the smart tooltip flipping inward.
- **Spoken Voiceover:**
  > *"This is the ArchGuard Command Center, built on FalkorDB. Rather than dumping 50 markdown files into an LLM prompt, we compiled our entire enterprise architecture into a live Company Brain.*  
  > *Here, 28 canonical nodes map our Architecture Decision Records, microservices, API endpoints, schema fields, and team ownership rosters with mathematical certainty."*

---

### **SCENE 3: Auditing PR-409 & Deterministic Collision (0:50 – 1:25)**
- **Video Visual:** Focus on the left Proposal Terminal.
- **Mouse Action:**
  1. Click the preset button: **`409 • Schema`** (Migrate `user_id` from UUIDv4 to Int64).
  2. Click the glowing **`⚡ EXECUTE 3-AGENT AUDIT`** button.
  3. Let the canvas animate and isolate the red glowing collision path.
- **Spoken Voiceover:**
  > *"Let's test PR-409, which proposes changing user_id to a 64-bit integer for indexing speed.*  
  > *We click Execute Audit. Three autonomous agents coordinate strictly through FalkorDB state mutations.*  
  > *In under 15 milliseconds, Agent 2 traverses the graph and catches a direct collision with ADR-001, which mandates UUIDv4 strings.*  
  > *Even better, it traverses 3 hops downstream to reveal that billing-service, checkout-service, and payment-gateway will break—and immediately pings their Tech Leads, Dave and Elena!"*

---

### **SCENE 4: Recalling Cross-Session Episodic Memory (1:25 – 1:55)**
- **Video Visual:** In the right pane, click the **`Memory & Plan`** tab.
- **Mouse Action:**
  1. Point to the green-bordered card: **`INC-2023-42`**.
  2. Highlight the *"Approved Pattern Solution"* and *"Trade-offs"* sections.
- **Spoken Voiceover:**
  > *"Normally, teams reject the PR and waste days arguing in Slack. But ArchGuard has cross-session episodic memory!*  
  > *Agent 3 queries FalkorDB for past incident precedents and instantly recalls incident INC-2023-42, approved by Alex.*  
  > *It surfaces the battle-tested Dual-Write Shadow Column workaround, complete with its 30% storage trade-off, and provides a production-ready compliant diff."*

---

### **SCENE 5: Compliant Fix & Fast-Path Approval (1:55 – 2:20)**
- **Video Visual:** Focus on the left code editor.
- **Mouse Action:**
  1. In the diff editor, replace:
     ```protobuf
     - string user_id = 1; // UUIDv4 RFC 4122
     + int64 user_id = 1; // Optimized sequential 64-bit ID
     ```
     With the compliant code:
     ```protobuf
     - string user_id = 1; // UUIDv4 RFC 4122
     + string user_id = 1; // UUIDv4 RFC 4122
     ```
  2. Click **`⚡ EXECUTE 3-AGENT AUDIT`** again.
  3. Watch the proposal node turn **emerald green** (`:Proposal Approved`) and the right pane display `✓ ARCHITECTURAL POLICY COMPLIANT`.
- **Spoken Voiceover:**
  > *"When the developer applies the fix to conform to ADR-001 and re-runs the audit, ArchGuard triggers its zero-violation fast path.*  
  > *Agent 2 verifies zero ADR collisions and zero broken consumers in 23 milliseconds. The proposal node turns vibrant emerald green, blast radius drops to zero, and the PR is approved for merge."*

---

### **SCENE 6: Live Markdown ADR Ingestion (2:20 – 2:40)**
- **Video Visual:** Top right header bar.
- **Mouse Action:**
  1. Click **`+ Ingest ADR (.md)`**.
  2. The modal opens with a sample `ADR-005: Enforce Idempotency Keys`.
  3. Click **`Compile into FalkorDB Brain`**.
  4. Watch the top badge live-increment from **`28 Nodes` ➔ `29 Nodes`**!
- **Spoken Voiceover:**
  > *"Architecture isn't static. Watch this: an architect can write a new markdown ADR with YAML frontmatter, click Compile, and FalkorDB dynamically parses and attaches it to the live graph in real time without restarting any services. The node counter ticks up live."*

---

### **SCENE 7: Outro & Pitch (2:40 – 2:55)**
- **Video Visual:** Switch to **Canvas Focus** mode (showing the full 29-node constellation expanding smoothly) or show your GitHub repository.
- **Mouse Action:** Move mouse across the illuminated constellation.
- **Spoken Voiceover:**
  > *"By combining FalkorDB's sub-15ms multi-hop traversals with agentic episodic memory, ArchGuard guarantees deterministic governance for autonomous engineering teams.*  
  > *Deterministic Detection, LLM-Explained. Built for the WeMakeDevs Graph Hacks Hackathon. Thank you!"*

---

## 💡 Quick Tips for Recording
1. **Resolution:** 1080p (1920x1080) at 100% or 125% browser zoom.
2. **Audio:** Speak with energy, clear pacing, and confidence.
3. **Pacing:** Let the red collision path glow for 2 seconds before speaking about blast radius—judges love seeing the visual payoff!
4. **Tool:** Use OBS Studio, Loom, or Windows Xbox Game Bar (`Win + Alt + R`).
