---
title: "Why LLM Code Reviewers Fail at Architecture—And How We Fixed It with a 9ms FalkorDB Knowledge Graph"
published: false
description: "How we built ArchGuard for the WeMakeDevs x FalkorDB Hackathon: a 3-agent pre-merge guardian that uses deterministic multi-hop Cypher traversals and episodic memory instead of prompt bloat."
tags: falkordb, wemakedevs, ai, architecture
cover_image: "https://raw.githubusercontent.com/SKfaizan-786/ArchGuard/main/screenshots/ArchGuard01.png"
---

## The Friday Afternoon Outage Story

It was 4:15 PM on a Friday when an engineer on the identity team pushed a seemingly harmless PR to `auth-service`:

```diff
--- a/schemas/user.proto
+++ b/schemas/user.proto
@@ -12,3 +12,3 @@ message UserProfile {
-  string user_id = 1; // UUIDv4 RFC 4122
+  int64 user_id = 1; // Optimized sequential 64-bit ID
 }
```

The pull request was titled: *"Optimize user lookup indexing: switch UUIDv4 strings to sequential 64-bit integers"*.

Every CI check passed green:
- TypeScript typechecks passed inside `auth-service`.
- Protobuf generated clean stubs.
- Unit tests executed cleanly with zero syntax failures.
- An AI code-review bot left a cheerful comment: *"Looks clean! Sequential IDs reduce B-tree index fragmentation by up to 25%."*

Twenty minutes after merging to staging, the payments gateway collapsed.

What the CI runner and the AI reviewer couldn't see was that two years ago, the architecture board ratified **ADR-001**: *"Canonical User Identification Standard"*, strictly mandating UUIDv4 strings across all internal APIs to prevent distributed collision attacks and enable multi-tenant data exports.

Even worse, three hops away in the microservice topology:
1. `auth-service` exposes `/v1/users`.
2. `checkout-service` consumes `/v1/users` to authorize customer carts.
3. `billing-service` and `ledger-service` deserialize `user_id` into strict UUID data stores.

By changing one field in an upstream schema, the PR silently bricked three downstream services, disrupted checkout flows, and dragged two engineering teams into an emergency triage incident.

Linters only check syntax. AI reviewers only evaluate isolated files. Neither has a clue how your company actually connects.

That is why I built **ArchGuard** for the **WeMakeDevs x FalkorDB Graph Hacks Hackathon** under the **Company Brain & Episodic Memory Track**.

---

## Why Vector RAG & Giant Context Windows Fail at Architecture

The natural modern reaction to this problem is: *"Just throw all the ADRs into an LLM context window or build a Vector RAG pipeline."*

When I started designing ArchGuard, I tested that exact approach. It failed immediately for three fundamental reasons:

### 1. Vector Similarity Searches for Topics, Not Invariant Rules
Vector embeddings measure semantic closeness. If a pull request modifies `int64 user_id`, vector search queries like `"user_id data type policy"` might return a blog post about database migration or an RFC on user profiles. But architectural governance is **binary and relational**:
- Does `ADR-001` govern `user_id`? **Yes.**
- Does `int64` match `ADR-001`'s expected format `UUIDv4`? **No.**
- An LLM guesses based on probabilities. A graph proves it with mathematical certainty.

### 2. Multi-Hop Transitive Blast Radius Is Invisible to RAG
In a microservice mesh, dependencies are directed paths:

```
(PR) ➔ (:SchemaField {name: "user_id"})
          ▲
          │ [:DEFINED_IN]
     (:Service {name: "auth-service"})
          ▲
          │ [:CALLS]
     (:Service {name: "checkout-service"})
          ▲
          │ [:DEPENDS_ON]
     (:Service {name: "billing-service"})
          │
          ▼ [:OWNED_BY]
     (:Team {name: "Billing Team"}) ➔ (@elena_fin: Person)
```

No vector database can follow that 4-hop chain in a single step to tell you: *"Changing this protobuf field breaks Elena's billing pipeline 3 services away."*

### 3. Context Window Bloat Destroys Attention
An enterprise with 60 microservices, 45 ADRs, and hundreds of OpenAPI specifications generates megabytes of technical documentation. Shoving all of that into a prompt on every pull request is expensive, slow (15–30 seconds), and suffers from catastrophic attention degradation where the model ignores critical bullet points buried in the middle.

---

## Enter ArchGuard: The Company Brain Architecture

ArchGuard flips the paradigm: **Deterministic Detection, LLM-Explained.**

Instead of asking an LLM to guess whether an architectural decision was violated, ArchGuard compiles company architecture into a high-performance **FalkorDB Knowledge Graph** (`archguard_brain`). 

FalkorDB handles deterministic graph traversals in **under 15 milliseconds** per Cypher query (with the full multi-agent graph pipeline executing in ~23ms–108ms, compared to 4.0s for LLM synthesis). The LLM's only job is to translate the raw input diff into AST parameters and synthesize a human-friendly remediation plan from historical precedents.

![ArchGuard Constellation Canvas](https://raw.githubusercontent.com/SKfaizan-786/ArchGuard/main/screenshots/ArchGuard02.png)
*Figure 1: ArchGuard Command Center showing real-time constellation topology, isolated collision path (red), and multi-hop blast radius.*

### The FinCore Domain Knowledge Graph

The system ships with a production-grade fintech topology (**FinCore Financial Systems**) modeled directly in FalkorDB:

```
                            FINCORE TOPOLOGY IN FALKORDB
                            
     🟡 [ ADR-001: UUIDv4 ]             🟡 [ ADR-002: Async Queue ]
              │                                      │
       [:ENFORCES]                              [:GOVERNS]
              ▼                                      ▼
    🟡 (user_id: SchemaField)              🔵 (/v1/webhooks/payment)
              │                                      │
        [:DEFINED_IN]                           [:EXPOSED_BY]
              ▼                                      ▼
      🔵 [ auth-service ]                    🔵 [ payment-gateway ]
              ▲                                      │
              │ [:CALLS]                             │ [:DEPENDS_ON]
      🔵 [ checkout-service ] ◄──────────────────────┘
              │
         [:OWNED_BY]
              ▼
      🟢 { Team: Checkout } ◄───[:LEADS]─── 🟢 (@dave_eng: Person)
              ▲
              │ [:APPLIES_TO]
      🧠 [ INC-2023-42: Legacy Workaround ]
```

The baseline graph contains **28 pristine nodes and 35 directed relationships**:
- 🟡 **Golden Rules & Contracts:** 4 `:ADR` nodes and 3 `:SchemaField` nodes.
- 🟢 **Organizational Context:** 4 `:Team` nodes and 5 `:Person` nodes (Tech Leads).
- 🔵 **Distributed Infrastructure:** 5 `:Service` nodes and 5 `:Endpoint` nodes.
- 🔴 **Transient PR State:** Dynamic `:Proposal` nodes created on-the-fly.
- 🧠 **Cross-Session Episodic Memory:** 2 `:ArchitecturalEpisode` nodes documenting historical incident precedents and approved trade-offs.

---

## How 3 Autonomous Agents Share State Through FalkorDB

Most multi-agent frameworks concatenate prompts: Agent A writes a huge JSON string, pastes it into Agent B's prompt, which appends another blob for Agent C. This causes compounding hallucinations and context explosion.

In ArchGuard, our 3 agents **never talk directly to each other**. They communicate strictly by reading and mutating shared subgraphs inside FalkorDB.

```
                              AGENT GRAPH PIPELINE
                              
   Git Diff / OpenAPI Spec
              │
              ▼
    ┌──────────────────┐
    │   AGENT 1        │  1. Ingests raw diff
    │  Spec Extractor  │  2. Identifies target field/endpoint
    └─────────┬────────┘  3. Executes Cypher MERGE
              │
              ▼
      ┌───────────────┐   MERGE (:Proposal {id: $id, status: 'PARSED'})
      │   FalkorDB    │   CREATE (p)-[:TARGETS_FIELD]->(target)
      │  Graph State  │
      └───────┬───────┘
              │
              ▼ (Passes ONLY proposalId)
    ┌──────────────────┐
    │   AGENT 2        │  1. Runs deterministic Cypher traversal (<15ms)
    │  Graph Auditor   │  2. Discovers ADR collisions & broken services
    └─────────┬────────┘  3. Writes findings directly to graph
              │
              ▼
      ┌───────────────┐   CREATE (p)-[:VIOLATES_ADR]->(adr)
      │   FalkorDB    │   CREATE (p)-[:BREAKS_CONSUMER]->(service)
      │  Graph State  │   SET p.status = 'CHECKED'
      └───────┬───────┘
              │
              ▼ (Passes ONLY proposalId)
    ┌──────────────────┐
    │   AGENT 3        │  1. Queries (:Proposal)-[:VIOLATES_ADR]->(:ADR)<-[:APPLIES_TO]-(:Episode)
    │  Advisor Memory  │  2. Recalls past incident compromise & approved workaround
    └─────────┬────────┘  3. Synthesizes compliant migration plan & replacement diff
              │
              ▼
      ┌───────────────┐   SET p.status = 'RESOLVED'
      │   FalkorDB    │   (Optional: Commits new :ArchitecturalEpisode)
      └───────────────┘
```

Notice how clean the orchestrator implementation is in `src/agents/orchestrator.ts`:

```typescript
// Agent 1: Parses diff and creates :Proposal node in FalkorDB
const extractorOutput = await runExtractorAgent(rawInput, title, options, tenant);
const proposalId = extractorOutput.proposalId;

// Agent 2: Receives ONLY the proposalId; reads and writes graph state
const auditorOutput = await runAuditorAgent(proposalId, tenant);

// Agent 3: Receives ONLY the proposalId; recalls episodic memory from graph
const advisorOutput = await runAdvisorAgent(proposalId, options, tenant);
```

Neither Agent 2 nor Agent 3 needs the raw git diff in memory. They query FalkorDB for the `:Proposal` node, inspect its relationships, and write their verdicts directly to the graph.

---

## The Real Cypher Queries Doing the Heavy Lifting

Here are the actual Cypher queries executed inside `src/engine/graphAuditor.ts` that make ArchGuard lightning fast:

### 1. Deterministic ADR Invariant Verification
When a proposed change targets a schema field or endpoint, Agent 2 checks if any active ADR governs that target with a conflicting invariant:

```cypher
MATCH (p:Proposal {id: $proposalId})-[t:TARGETS_FIELD]->(target)<-[:GOVERNS]-(adr:ADR)
WHERE adr.target_property = t.changed_property 
  AND adr.expected_value <> t.proposed_value
OPTIONAL MATCH (adr)<-[:AUTHORED]-(author:Person)-[:MEMBER_OF]->(authorTeam:Team)
RETURN 
  adr.code AS adrCode,
  adr.title AS adrTitle,
  adr.expected_value AS expectedValue,
  t.proposed_value AS proposedValue,
  author.name AS authorName,
  authorTeam.name AS authorTeam,
  authorTeam.slack_channel AS slackChannel;
```

If a match is found, Agent 2 immediately executes:

```cypher
MATCH (p:Proposal {id: $proposalId}), (a:ADR {code: $adrCode})
MERGE (p)-[r:VIOLATES_ADR {reason: $reason, detected_at: datetime()}]->(a)
SET p.status = 'CHECKED';
```

### 2. Multi-Hop Blast Radius Traversal
Next, Agent 2 determines which downstream microservices and tech leads will be impacted:

```cypher
MATCH (p:Proposal {id: $proposalId})-[:TARGETS_FIELD]->(field:SchemaField)
MATCH (field)<-[:USES_FIELD]-(ep:Endpoint)<-[:CONSUMES]-(consumer:Service)
MATCH (consumer)<-[:OWNS]-(consumerTeam:Team)
OPTIONAL MATCH (consumerTeam)<-[:MEMBER_OF]-(lead:Person)
WHERE lead.role CONTAINS 'Lead' OR lead.role CONTAINS 'Staff'
RETURN DISTINCT
  consumer.name AS serviceName,
  consumer.tier AS serviceTier,
  consumerTeam.name AS owningTeam,
  consumerTeam.slack_channel AS slackChannel,
  lead.handle AS techLeadHandle,
  ep.path AS viaEndpoint;
```

In a single query taking **4ms**, FalkorDB traverses from the modified schema field through exposed API endpoints to find every consumer service and their corresponding Tech Lead.

### 3. Recalling Cross-Session Episodic Memory
When an ADR is violated, teams don't always reject the PR. Sometimes, a production outage or migration forces a temporary workaround.

ArchGuard captures these workarounds as `:ArchitecturalEpisode` nodes. When a new violation occurs, Agent 3 recalls historical precedents:

```cypher
MATCH (p:Proposal {id: $proposalId})-[:VIOLATES_ADR]->(adr:ADR)<-[:APPLIES_TO]-(ep:ArchitecturalEpisode)
RETURN 
  ep.incident_id AS incidentId,
  ep.title AS title,
  ep.workaround_pattern AS workaroundPattern,
  ep.approved_by AS approvedBy,
  ep.resolution_date AS resolutionDate,
  ep.trade_offs AS tradeOffs;
```

For `ADR-001`, FalkorDB immediately recalls **INC-2023-42**:
> *"Legacy Billing System Integer Ingestion Workaround. Approved by @sarah_arch. Workaround: Maintain dual-representation in API gateway by exposing an X-Legacy-User-Id header while keeping the canonical body payload as UUIDv4."*

Instead of an AI reviewer hallucinating advice, Agent 3 presents the **exact battle-tested compromise approved by company architects**.

![Episodic Memory Recall UI](https://raw.githubusercontent.com/SKfaizan-786/ArchGuard/main/screenshots/ArchGuard04.png)
*Figure 2: Memory & Plan tab surfacing historical incident compromise INC-2023-42 with approved workaround steps and trade-offs.*

---

## Engineering Lessons & Real Bugs We Fixed

Building on a graph database gave us incredible superpowers, but we hit three real-world engineering hurdles that taught us valuable lessons:

### 1. Deduplicating Blast Radius Paths vs. Distinct Services
During early testing with PR #409, our UI reported: *"Breaks 5 downstream consumers"*. But when we opened the microservices list, only 3 services were listed (`billing-service`, `checkout-service`, and `ledger-service`).

**The Bug:** `billing-service` consumed two different endpoints (`/v1/users` and `/v1/ledger/entries`). A naive path count treated each traversal path as a separate broken consumer, inflating the count from 3 to 5.

**The Fix:** We refactored `src/engine/graphAuditor.ts` with an in-memory aggregation map:

```typescript
// Deduplicate by distinct serviceName (counting distinct services, not paths)
const serviceMap = new Map<string, BlastRadiusFinding>();

for (const r of rawRows) {
  const existing = serviceMap.get(r.serviceName);
  if (existing) {
    if (r.viaEndpoint && !existing.viaEndpoint.includes(r.viaEndpoint)) {
      existing.viaEndpoint = `${existing.viaEndpoint}, ${r.viaEndpoint}`;
    }
    continue;
  }
  serviceMap.set(r.serviceName, { ... });
}
return Array.from(serviceMap.values());
```

Now, the blast radius accurately reflects **3 distinct Tier-1 microservices**, while displaying all collapsed endpoint paths in the details card.

### 2. The Latency Split: Real-Time Graph Traversal vs. LLM Time
One of our proudest architectural decisions was displaying the latency split directly in the Telemetry and Cypher Trace panels:
- **⚡ FalkorDB Graph Traversal:** Individual Cypher queries execute in `0.8ms – 14ms` (with the full multi-agent 5-query traversal pipeline executing in `23ms – 108ms`)
- **⏱️ LLM Synthesis Time:** `~1,200ms – 4,000ms`

Seeing this in real-time demonstrates the immense power of in-memory graph databases for AI agents. Graph traversal is virtually instantaneous; LLM token streaming is the bottleneck.

Furthermore, we added a **zero-violation fast path**: If a developer fixes their code and changes `int64 user_id` back to `string user_id`, Agent 2 detects **0 violations and 0 blast radius**. ArchGuard skips the remediation LLM entirely, turns the proposal node emerald green (`:Proposal (Approved)`), and finishes the audit in milliseconds!

![Green Glow Approved Proposal](https://raw.githubusercontent.com/SKfaizan-786/ArchGuard/main/screenshots/ArchGuard03.png)
*Figure 3: Compliant proposals glow green on the pitch-black constellation canvas with zero blast radius.*

### 3. Bulletproof Offline Heuristic Fallback
During hackathon live demos or production network blips, external LLM APIs can hit rate limits or time out. 

To ensure ArchGuard never fails on stage, we built a zero-dependency deterministic heuristic fallback in `src/llm/client.ts`. If no API keys are provided or if external calls fail, our AST pattern matcher extracts schema types and generates compliant migration templates in `0.0s`. The system is 100% resilient.

---

## Live ADR Ingestion & Memory Recording

ArchGuard isn't a static demo—it is a live, evolving Company Brain:

### 1. Live Markdown ADR Ingestion (`+ Ingest ADR (.md)`)
Click the **`+ Ingest ADR`** button in the header, paste standard markdown with YAML frontmatter:

```markdown
---
code: ADR-005
title: Enforce Idempotency Keys on Payment Ingestion
status: Accepted
governs_target: /v1/webhooks/payment
target_property: idempotency_enforced
expected_value: "true"
author_handle: elena_fin
---
## Context
Double-charging incidents require mandatory Idempotency-Key headers on all payment webhooks.
```

Click **Compile into FalkorDB Brain**. FalkorDB dynamically parses the frontmatter, merges the `:ADR` node, creates the `[:GOVERNS]->(:Endpoint)` edge, and links `[:AUTHORED]->(:Person)` in real time. The node counter ticks up from 28 to 29 instantly.

### 2. Recording New Episodic Memory (`+ Record Memory`)
When an audit flags a violation, engineers can click **`+ Commit New Workaround to FalkorDB Memory`**. This records a new `:ArchitecturalEpisode` linked via `[:APPLIES_TO]` to the violated ADR. The next engineer who tests a similar change will immediately see that workaround in their audit results.

---

## Try It Yourself

ArchGuard is fully open source. You can clone the repo and run the full stack locally in under two minutes:

```bash
# 1. Clone repository
git clone https://github.com/SKfaizan-786/ArchGuard.git
cd ArchGuard

# 2. Launch FalkorDB via Docker
docker compose up -d

# 3. Install dependencies & launch full-stack
npm install
npm --prefix client install
npm run dev:all
```

- **ArchGuard UI:** [http://localhost:5173](http://localhost:5173)
- **FalkorDB Web Browser:** [http://localhost:3000](http://localhost:3000)
- **Backend API:** [http://localhost:4000/api/health](http://localhost:4000/api/health)

### Project Links
- 💻 **GitHub Repository:** [https://github.com/SKfaizan-786/ArchGuard](https://github.com/SKfaizan-786/ArchGuard)
- 🎥 **Live Demo Video:** REPLACE_WITH_YOUTUBE_VIDEO_URL

---

## Final Thoughts: The Future of Agentic Governance

AI coding agents are already generating pull requests faster than human tech leads can review them. If we rely on probabilistic LLMs to review probabilistic LLMs, architectural drift is guaranteed.

By anchoring AI agents to a **deterministic FalkorDB knowledge graph**, we get the best of both worlds:
- **The Graph** provides mathematical certainty, instant multi-hop reachability, and persistent episodic memory across sessions.
- **The Agents** provide human-friendly natural language synthesis, intelligent diff generation, and contextual remediation.

Linters check your syntax. ArchGuard protects your architecture.

---

*Built with ❤️ by SK Faizan for the **WeMakeDevs x FalkorDB "Graph Hacks" Hackathon** (Company Brain & Episodic Memory Track).*
