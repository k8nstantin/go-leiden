---
layout: default
title: go-leiden — How We Build It
---

# go-leiden

**The native Go implementation of the Leiden community detection algorithm.**  
Built entirely by autonomous AI agents. Zero human code commits.

→ [GitHub Repository](https://github.com/k8nstantin/go-leiden) · [OpenPraxis](https://github.com/k8nstantin/OpenPraxis)

---

## What we're building

A standalone Go library — `github.com/k8nstantin/go-leiden` — that any Gopher can import:

```go
result, err := leiden.Leiden(edges, leiden.DefaultOptions())
```

The Leiden algorithm guarantees **well-connected communities** in graph partitioning. The Go ecosystem has Louvain. It has no Leiden. This library fills that gap.

We port from [graspologic-native](https://github.com/graspologic-org/graspologic-native) (Microsoft Research, MIT) — the same Rust implementation used in production by Microsoft GraphRAG.

---

## The Product DAG

Every component is a node in the [OpenPraxis](https://github.com/k8nstantin/OpenPraxis) entity graph. The build executes as a directed acyclic graph — each manifest feeds the next:

```
go-leiden
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

Each manifest maps to a GitHub issue and a page below. Each task runs with a **cascading prompt**:

```
Product prompt  →  "what are we building, why, module path, reference"
      ↓
Manifest prompt →  "this component's role, dependencies, deliverables"
      ↓
Task prompt     →  "exact types, function signatures, formulas, reference files"
      ↓
Skill prompts   →  "Go library standards, testing conventions, zero-dep rules"
```

The agent running M2/T1 knows: the product it's contributing to, where M2 fits in the chain, the exact graspologic-native Rust files to read first, and the precise QVI formula to implement.

---

## Manifests

| | Component | GitHub Issue | Status |
|---|---|---|---|
| [M1](manifests/m1) | Core data structures | [#1](https://github.com/k8nstantin/go-leiden/issues/1) | 🔄 In progress |
| [M2](manifests/m2) | Algorithm phases | [#2](https://github.com/k8nstantin/go-leiden/issues/2) | ⏳ Pending |
| [M3](manifests/m3) | Quality functions | [#3](https://github.com/k8nstantin/go-leiden/issues/3) | ⏳ Pending |
| [M4](manifests/m4) | Public API | [#4](https://github.com/k8nstantin/go-leiden/issues/4) | ⏳ Pending |
| [M5](manifests/m5) | Test suite | [#5](https://github.com/k8nstantin/go-leiden/issues/5) | ⏳ Pending |

---

## How the autonomous build works

### The Trace-Grounded Feedback Loop

go-leiden runs inside [OpenPraxis](https://github.com/k8nstantin/OpenPraxis) — an autonomous agent workflow engine. Every task benefits from three feedback mechanisms:

**1. Prior context injection**  
Every agent run sees its own execution history. If a task fails and retries, the agent reads exactly what it tried before and why it failed. It cannot repeat the same mistake blind.

**2. Pass-rate tracking**  
`GET /api/execution/frontier` tracks per-task pass rates. If M3's quality function math is wrong, the pass rate drops before it reaches M4.

**3. Autonomous proposer loop**  
After 2 consecutive non-transient failures, the system automatically fires a proposer that analyzes the failure, generates an improved prompt, and tests it. Prompts self-improve without human intervention.

```
Task fails twice
      ↓
Proposer fires automatically
      ↓
Analyzes failure pattern
      ↓
Generates sharper prompt
      ↓
Tests against pass rate → keep if improves ≥5%
      ↓
Task retries with better prompt
```

### Why this is the case study

go-leiden answers: **can autonomous agents build a production Go library — from algorithm specification to published package — without human code commits?**

The algorithm is well-specified (graspologic-native Rust source). The test target is clear (karate club graph). The quality bar is measurable (go test, go vet, benchmarks). Every step is logged, every failure is traced, every improvement is recorded.

This is eating our own dog food: we built the feedback loop to improve agents, and that same loop is now building an open-source library.

---

## Credits

- [Traag, Waltman & van Eck (2019)](https://arxiv.org/abs/1810.08473) — the algorithm
- [leidenalg](https://github.com/vtraag/leidenalg) — original C++ implementation by Vincent Traag
- [graspologic-native](https://github.com/graspologic-org/graspologic-native) — Rust reference we port from (Microsoft Research + Johns Hopkins, MIT)
- [CWTSLeiden/networkanalysis](https://github.com/CWTSLeiden/networkanalysis) — Java reference implementation
- [Graphify](https://graphify.net) — inspiration for applying Leiden to code knowledge graphs
- [OpenPraxis](https://github.com/k8nstantin/OpenPraxis) — the engine building this
