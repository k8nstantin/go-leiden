# go-leiden

[![Go Reference](https://pkg.go.dev/badge/github.com/k8nstantin/go-leiden.svg)](https://pkg.go.dev/github.com/k8nstantin/go-leiden)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Go Version](https://img.shields.io/badge/go-1.22+-blue.svg)](https://golang.org/)
[![Built Autonomously](https://img.shields.io/badge/built%20by-AI%20agents-blueviolet)](https://github.com/k8nstantin/OpenPraxis)

> **Status: Active development**

Built entirely by autonomous AI agents on [OpenPraxis](https://github.com/k8nstantin/OpenPraxis). No human code commits.

**The first and only native Go implementation of the Leiden community detection algorithm.**

Zero external dependencies. Pure Go. Single static binary. Production quality.

---

## What problem does this solve?

Graph community detection is a fundamental operation in knowledge graphs, network analysis, code dependency mapping, recommendation systems, and document clustering. The two dominant algorithms are **Louvain** (2008) and **Leiden** (2019).

**The Go ecosystem has Louvain** (via `gonum/graph/community`).

**The Go ecosystem has no Leiden.**

This library fills that gap.

---

## Why Leiden over Louvain?

The Louvain algorithm has a well-documented flaw: it can produce **internally disconnected communities** — nodes grouped together that aren't actually reachable from each other within the community. This produces misleading results in any downstream system that assumes communities are coherent clusters.

The Leiden algorithm (Traag, Waltman & van Eck, 2019) introduces a **refinement phase** that explicitly prevents this. Leiden guarantees that every community it produces is well-connected. Communities represent genuine local structure, not artifacts of the greedy merge order.

In practice, for code knowledge graphs and document corpora, the difference is the separation between "these files belong together" and "these files happen to share a common import."

---

## Why this implementation?

We port from **[graspologic-native](https://github.com/graspologic-org/graspologic-native)** (Microsoft Research + Johns Hopkins University, MIT license) — the Rust implementation used in production by **[Microsoft GraphRAG](https://github.com/microsoft/graphrag)**.

We chose graspologic-native over the original [leidenalg](https://github.com/vtraag/leidenalg) (C++ + igraph) because:

| | leidenalg | graspologic-native | **go-leiden** |
|---|---|---|---|
| Language | C++ | Rust | **Go** |
| igraph dependency | ✓ (500k+ lines C) | ✗ | ✗ |
| External deps | Yes | No | **Zero** |
| Static binary | No | Yes | **Yes** |
| Production use | Research | Microsoft GraphRAG | **Research** |
| Resolution formula | Reference impl | Corrected¹ | **Corrected¹** |

> ¹ graspologic-native identified and corrected a resolution scaling bug in the original CWTSLeiden Java reference implementation. go-leiden uses the corrected formula.

---

## Installation

```bash
go get github.com/k8nstantin/go-leiden
```

Requires Go 1.22+. Zero external dependencies.

---

## Quick start

```go
package main

import (
    "fmt"
    leiden "github.com/k8nstantin/go-leiden"
)

func main() {
    // Two tight clusters connected by a weak bridge
    edges := []leiden.Edge{
        {Source: "auth.Login",   Target: "auth.Verify",   Weight: 1.0},
        {Source: "auth.Login",   Target: "auth.Session",  Weight: 1.0},
        {Source: "auth.Verify",  Target: "auth.Session",  Weight: 1.0},
        {Source: "db.Query",     Target: "db.Connect",    Weight: 1.0},
        {Source: "db.Query",     Target: "db.Cache",      Weight: 1.0},
        {Source: "db.Connect",   Target: "db.Cache",      Weight: 1.0},
        {Source: "auth.Session", Target: "db.Connect",    Weight: 0.05}, // weak bridge
    }

    result, err := leiden.Leiden(edges, leiden.DefaultOptions())
    if err != nil {
        panic(err)
    }

    fmt.Printf("Quality: %.4f\n", result.Quality)
    for node, cluster := range result.Partition {
        fmt.Printf("  %-20s → cluster %d\n", node, cluster)
    }
    // auth.* → cluster 0
    // db.*   → cluster 1
}
```

---

## Options

```go
opts := leiden.Options{
    Resolution:  1.0,               // community granularity; higher = more, smaller communities
    Randomness:  0.001,             // refinement stochasticity; 0 = deterministic greedy
    Iterations:  2,                 // full passes over the network
    Trials:      1,                 // run N times, return best quality
    Seed:        42,                // 0 = random; >0 = fully deterministic
    Quality:     leiden.Modularity, // or leiden.CPM
}
```

---

## Hierarchical Leiden

For large graphs where communities need to stay under a size limit (e.g. knowledge graph chunks for LLM context windows — the use case in Microsoft GraphRAG):

```go
result, err := leiden.HierarchicalLeiden(edges, leiden.HierarchicalOptions{
    Options:        leiden.DefaultOptions(),
    MaxClusterSize: 50,
})

for _, hc := range result {
    if hc.IsFinal {
        fmt.Printf("node=%-20s cluster=%d level=%d\n", hc.Node, hc.Cluster, hc.Level)
    }
}
```

---

## How this library is built — autonomous AI agents

## The case study — first agentic OS reverse-engineer and language port

This is the **first documented case** of an autonomous agentic OS using AI agents to reverse-engineer a production library and port it from one language to another — without a single human code commit.

**[graspologic-native](https://github.com/graspologic-org/graspologic-native)** (Rust, Microsoft Research) → **[go-leiden](https://github.com/k8nstantin/go-leiden)** (Go)

Orchestrated entirely by **[OpenPraxis](https://github.com/k8nstantin/OpenPraxis)** — a peer-to-peer agentic OS that runs autonomous coding agents.

**By the numbers:**
- **2 hours 49 minutes** wall time start to finish
- **609 agent turns** across 9 tasks
- **4,417 lines of Go** produced by agents
- **9 tasks** (5 implementation + 4 fixes), fully sequential DAG
- **0 human code commits**

This is not a human-authored codebase. go-leiden is built entirely by **autonomous AI agents** orchestrated by [OpenPraxis](https://github.com/k8nstantin/OpenPraxis) — a peer-to-peer workflow engine for autonomous coding agents.

### The product DAG

![go-leiden Product DAG](assets/dag.png)

Every component is modelled as an entity in the OpenPraxis graph. The build is a directed acyclic graph of manifests (components) and tasks (implementation units):

```
Product: go-leiden
├─ Skill: Senior Developer Coding Practices
├─ Skill: Open-Source Go Library Engineering
│
├─ M1 — Core data structures
│    CompactNetwork (CSR adjacency list), Clustering, Edge types, errors
│    └─ T1: Implement CompactNetwork, Clustering, Edge, errors
│         ↓
├─ M2 — Algorithm phases
│    Local-move phase, refinement phase (Leiden innovation), aggregation loop
│    └─ T1: Implement three algorithm phases
│         ↓
├─ M3 — Quality functions
│    Modularity, CPM, resolution scaling (corrected formula)
│    └─ T1: Implement Modularity and CPM
│         ↓
├─ M4 — Public API
│    Leiden(), HierarchicalLeiden(), go.mod, README
│    └─ T1: Implement public API + module setup
│         ↓
└─ M5 — Test suite
     Karate club validation, 10k-node benchmarks, fuzz testing
     └─ T1: Write tests, benchmarks, fuzz
```

Each manifest maps to a GitHub issue. Each task carries a cascading prompt:

```
Product prompt  →  "what are we building, why, module path, reference"
  ↓
Manifest prompt →  "this component's role, dependencies, deliverables"
  ↓
Task prompt     →  "exact types, function signatures, formulas, reference files to read"
  ↓
Skill prompts   →  "Go library standards, testing conventions, zero-dep rules"
```

The agent that runs M2/T1 knows: the product it's contributing to, where M2 fits in the chain, the exact graspologic-native Rust files to read before writing a line, and the precise QVI formula to implement. It doesn't guess — it executes from a precise spec.

### The Trace-Grounded Feedback Loop

go-leiden is built on top of a feedback system we built into OpenPraxis itself: the **[Trace-Grounded Feedback Loop](https://github.com/k8nstantin/OpenPraxis)**.

Three capabilities activate automatically for every task run:

**1. Prior context injection**
Every agent run sees its own history. If T2 (algorithm phases) fails and retries, the agent sees the prior run's execution trace, the tool calls it made, and the error it hit. It cannot repeat the same mistake blind. This is especially critical for M2's weight cache correctness — the hardest part of the port.

**2. Pass-rate tracking**
`GET /api/execution/frontier` tracks pass rates per task and per manifest. If M3's quality function math is wrong, the pass rate drops and we see it before it propagates to M4's public API.

**3. Autonomous proposer loop**
When a task hits 2 consecutive non-transient failures (`max_turns` or `deliverable_missing`), the system automatically fires a **proposer task** that analyzes the failure pattern and generates an improved prompt. The improved prompt is tested against the pass rate and kept only if it improves by ≥5%. The prompts self-improve without human intervention.

```
T2 fails (weight cache wrong)
  ↓
T2 fails again (same root cause, different expression)
  ↓
Proposer fires automatically
  ↓
Proposer analyzes: "agent misses the clusterWeights cache update on node removal"
  ↓
Proposer generates sharper prompt: adds explicit warning about cache invariants
  ↓
New prompt tested: pass rate improves → kept
  ↓
T2 retries with improved prompt → succeeds
```

This is recursive: we built a feedback loop to improve autonomous agents, and that same loop is now building this open-source library. go-leiden is a live demonstration of the system.

### Why this matters

This project answers a specific question: **can autonomous agents build a production-quality, ecosystem-ready Go library — from algorithm specification to published package — without human code commits?**

go-leiden is the answer in progress. The algorithm is well-specified (graspologic-native Rust source), the test target is clear (karate club graph, graspologic-native output), and the quality bar is measurable (go test, go vet, benchmarks). If the agents can ship a correct, performant, zero-dependency Leiden implementation that the Go community will actually import, the answer is yes.

---

## Build status

Each step is a manifest — one branch, one PR, one review before the next fires. Click any step to read the full agent prompt.

| Step | What it builds | Task | Issue | Status |
|---|---|---|---|---|
| [M1 — Core data structures](docs/manifests/m1.md) | `CompactNetwork` (CSR adjacency list), `Clustering`, `Edge` types, error sentinels. The foundation everything else builds on. | [T1 — Implement CompactNetwork, Clustering, Edge](docs/manifests/m1.md#task-t1--implement-compactnetwork-clustering-edge-types-errors) | [#1](https://github.com/k8nstantin/go-leiden/issues/1) | 🔄 In progress |
| [M2 — Algorithm phases](docs/manifests/m2.md) | The three Leiden phases: local-move (greedy), refinement (the Leiden innovation over Louvain), aggregation (recursive contraction). | [T1 — Implement three phases](docs/manifests/m2.md#task-t1--implement-local-move-refinement-aggregation-phases) | [#2](https://github.com/k8nstantin/go-leiden/issues/2) | ⏳ After M1 |
| [M3 — Quality functions](docs/manifests/m3.md) | Modularity and CPM scoring + resolution scaling (graspologic-native corrected formula). | [T1 — Implement Modularity and CPM](docs/manifests/m3.md#task-t1--implement-modularity-and-cpm-quality-functions) | [#3](https://github.com/k8nstantin/go-leiden/issues/3) | ⏳ After M2 |
| [M4 — Public API](docs/manifests/m4.md) | `Leiden()`, `HierarchicalLeiden()` (used by Microsoft GraphRAG), `Modularity()`. Sets up `go.mod` and README. | [T1 — Implement public API](docs/manifests/m4.md#task-t1--implement-public-api--gomod) | [#4](https://github.com/k8nstantin/go-leiden/issues/4) | ⏳ After M3 |
| [M5 — Tests + benchmarks](docs/manifests/m5.md) | Karate club validation, 10k-node benchmarks (< 5s), fuzz testing. The publication gate. | [T1 — Write test suite](docs/manifests/m5.md#task-t1--write-test-suite-benchmarks-fuzz-tests) | [#5](https://github.com/k8nstantin/go-leiden/issues/5) | ⏳ After M4 |

→ [How autonomous agents build this — prompts, DAG, feedback loop](docs/index.md)

---


## Build stats

This library was built entirely by autonomous AI agents on [OpenPraxis](https://github.com/k8nstantin/OpenPraxis). No human code commits.

| | |
|---|---|
| **Total build time** | 2 hours 49 minutes |
| **Agent turns** | 609 |
| **Lines of Go** | 4,417 |
| **Tasks** | 9 sequential (5 implementation + 4 review fixes) |
| **Human code commits** | 0 |

| Task | What | Turns |
|---|---|---|
| M1 | Core data structures — CompactNetwork, Clustering, Edge | 47 |
| M2 | Algorithm phases — local-move, refinement, aggregation | 57 |
| M3 | Quality functions — Modularity, CPM | 76 |
| M4 | Public API — Leiden(), HierarchicalLeiden(), GroupBy() | 65 |
| M5 | Tests, benchmarks, fuzz validation | 106 |
| M6/T1 | Fix 3 blocking PR review issues | 43 |
| M6/T2 | context.Context + GroupBy/CommunityCount | 112 |
| M6/T3 | Copyright headers + example tests | 57 |
| M6/T4 | CI + CHANGELOG + CONTRIBUTING + release prep | 46 |

## Credits

This library would not exist without:

- **[Traag, Waltman & van Eck (2019)](https://arxiv.org/abs/1810.08473)** — *From Louvain to Leiden: guaranteeing well-connected communities.* Scientific Reports 9, 5233. The algorithm.

- **[leidenalg](https://github.com/vtraag/leidenalg)** by Vincent Traag — the original C++ + Python implementation. The authoritative reference for algorithm correctness. MIT license.

- **[graspologic-native](https://github.com/graspologic-org/graspologic-native)** by Microsoft Research and Johns Hopkins University — the Rust implementation we port from. Self-contained (no igraph), production-proven in Microsoft GraphRAG. MIT license.

- **[CWTSLeiden/networkanalysis](https://github.com/CWTSLeiden/networkanalysis)** — the Java reference implementation by the original authors.

- **[Graphify](https://graphify.net)** — the open-source Python code knowledge graph tool (47k GitHub stars) that inspired this library's application to code analysis.

- **[OpenPraxis](https://github.com/k8nstantin/OpenPraxis)** — the autonomous agent workflow engine that is building this library. The Trace-Grounded Feedback Loop, the product DAG, the cascading prompts — all OpenPraxis.

---

## License

MIT License. See [LICENSE](LICENSE).

Built with [OpenPraxis](https://github.com/k8nstantin/OpenPraxis) — autonomous agents, self-improving prompts, zero human code commits.
