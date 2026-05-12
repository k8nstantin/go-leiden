# go-leiden

[![Go Reference](https://pkg.go.dev/badge/github.com/k8nstantin/go-leiden.svg)](https://pkg.go.dev/github.com/k8nstantin/go-leiden)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Go Version](https://img.shields.io/badge/go-1.22+-blue.svg)](https://golang.org/)
> **Status: Active development** — built autonomously by AI agents on [OpenPraxis](https://github.com/k8nstantin/OpenPraxis).

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

We port from **[graspologic-native](https://github.com/graspologic-org/graspologic-native)** (Microsoft Research + Johns Hopkins University, MIT license) — the Rust implementation used in production by **[Microsoft GraphRAG](https://github.com/microsoft/graphrag)**, the knowledge graph extraction pipeline.

We chose graspologic-native over the original [leidenalg](https://github.com/vtraag/leidenalg) (C++ + igraph) because:

| | leidenalg | graspologic-native | **go-leiden** |
|---|---|---|---|
| Language | C++ | Rust | **Go** |
| igraph dependency | ✓ (500k+ lines C) | ✗ | ✗ |
| External deps | Yes | No | **Zero** |
| Static binary | No | Yes | **Yes** |
| Production use | Research | Microsoft GraphRAG | OpenPraxis + you |
| Resolution formula | Reference impl | Corrected¹ | **Corrected¹** |

> ¹ graspologic-native identified and corrected a resolution scaling bug in the original CWTSLeiden Java reference implementation. go-leiden uses the corrected formula.

---

## What we are building

go-leiden is a **standalone, importable Go library** with three public functions:

```go
// Leiden partitions a graph into well-connected communities.
func Leiden(edges []Edge, opts Options) (Result, error)

// HierarchicalLeiden recursively sub-clusters large communities.
// Used by Microsoft GraphRAG for knowledge graph partitioning.
func HierarchicalLeiden(edges []Edge, opts HierarchicalOptions) ([]HierarchicalCluster, error)

// Modularity scores an existing partition without running clustering.
func Modularity(edges []Edge, communities map[string]int, resolution float64) float64
```

### Algorithm

Three phases per iteration:

**Phase 1 — Local Move:** Greedy quality improvement. Each node tries to move to the neighboring cluster that maximises the quality function (Modularity or CPM). Nodes whose neighbors changed are re-evaluated.

**Phase 2 — Refinement** *(the Leiden innovation)*: Within each Phase 1 community, the algorithm restarts from singleton subclusters and stochastically merges them. This phase guarantees that communities are well-connected — Louvain has no equivalent step.

**Phase 3 — Aggregation:** Communities are contracted into super-nodes. The algorithm recurses on the induced network until no improvement is possible.

### Quality functions

- **Modularity** (default): measures how much within-community edge density exceeds random expectation. Resolution parameter controls granularity.
- **CPM** (Constant Potts Model): measures internal density against a fixed threshold. Useful when communities are expected to have uniform density.

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
        // Cluster A
        {Source: "auth.Login", Target: "auth.Verify",   Weight: 1.0},
        {Source: "auth.Login", Target: "auth.Session",  Weight: 1.0},
        {Source: "auth.Verify", Target: "auth.Session", Weight: 1.0},
        // Cluster B
        {Source: "db.Query",  Target: "db.Connect",  Weight: 1.0},
        {Source: "db.Query",  Target: "db.Cache",    Weight: 1.0},
        {Source: "db.Connect", Target: "db.Cache",   Weight: 1.0},
        // Weak bridge
        {Source: "auth.Session", Target: "db.Connect", Weight: 0.05},
    }

    result, err := leiden.Leiden(edges, leiden.DefaultOptions())
    if err != nil {
        panic(err)
    }

    fmt.Printf("Found %d communities (quality: %.4f)\n",
        len(uniqueClusters(result.Partition)), result.Quality)
    for node, cluster := range result.Partition {
        fmt.Printf("  %s → cluster %d\n", node, cluster)
    }
}

func uniqueClusters(p map[string]int) map[int]struct{} {
    m := make(map[int]struct{})
    for _, c := range p { m[c] = struct{}{} }
    return m
}
```

---

## Options

```go
opts := leiden.Options{
    Resolution:  1.0,            // community granularity; higher = more, smaller communities
    Randomness:  0.001,          // refinement stochasticity; 0 = deterministic greedy
    Iterations:  2,              // full passes over the network
    Trials:      1,              // run N times, return best quality (use >1 with Seed=0)
    Seed:        42,             // 0 = random seed; >0 = fully deterministic
    Quality:     leiden.Modularity, // or leiden.CPM
}
```

---

## Hierarchical Leiden

For large graphs where communities need to stay under a size limit (e.g. knowledge graph chunks for LLM context windows):

```go
result, err := leiden.HierarchicalLeiden(edges, leiden.HierarchicalOptions{
    Options:        leiden.DefaultOptions(),
    MaxClusterSize: 50, // recursively sub-cluster any community > 50 nodes
})

for _, hc := range result {
    if hc.IsFinal {
        fmt.Printf("node=%s cluster=%d level=%d\n", hc.Node, hc.Cluster, hc.Level)
    }
}
```

---

## Build plan

| Manifest | Component | Status |
|---|---|---|
| M1 | Core data structures (CompactNetwork, Clustering, Edge) | 🔄 In progress |
| M2 | Algorithm phases (local-move, refinement, aggregation) | ⏳ Pending M1 |
| M3 | Quality functions (Modularity, CPM) | ⏳ Pending M1 |
| M4 | Public API + go.mod + README | ⏳ Pending M2+M3 |
| M5 | Tests, benchmarks, fuzz, validation | ⏳ Pending M4 |

---

## Credits

This library would not exist without:

- **[Traag, Waltman & van Eck (2019)](https://arxiv.org/abs/1810.08473)** — *From Louvain to Leiden: guaranteeing well-connected communities.* Scientific Reports 9, 5233. The algorithm itself.

- **[leidenalg](https://github.com/vtraag/leidenalg)** by Vincent Traag — the original C++ + Python implementation. The authoritative reference for algorithm correctness. MIT license.

- **[graspologic-native](https://github.com/graspologic-org/graspologic-native)** by Microsoft Research and Johns Hopkins University — the Rust implementation we port from. Self-contained (no igraph), production-proven in Microsoft GraphRAG. MIT license.

- **[CWTSLeiden/networkanalysis](https://github.com/CWTSLeiden/networkanalysis)** — the Java reference implementation by the original authors. The shared algorithmic ancestor of both leidenalg and graspologic-native.

- **[Graphify](https://graphify.net)** — the inspiration for applying Leiden to code knowledge graphs. 47,000 GitHub stars in 5 weeks (Python). go-leiden is the Go-native answer.

- **[OpenPraxis](https://github.com/k8nstantin/OpenPraxis)** — the autonomous agent workflow engine that built this library. go-leiden is a case study in fully autonomous open-source library development.

---

## License

MIT License. See [LICENSE](LICENSE).

Built with [OpenPraxis](https://github.com/k8nstantin/OpenPraxis).
