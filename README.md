# go-leiden

The first native Go implementation of the [Leiden algorithm](https://arxiv.org/abs/1810.08473) for community detection in graphs.

> **Status: In active development** — built autonomously by AI agents.

## What is Leiden?

The Leiden algorithm (Traag, Waltman & van Eck, 2019) partitions graph nodes into communities that are guaranteed to be well-connected. It fixes the core flaw of the Louvain algorithm: Louvain can produce internally disconnected communities; Leiden cannot.

## Installation

```bash
go get github.com/k8nstantin/go-leiden
```

## Quick start

```go
import leiden "github.com/k8nstantin/go-leiden"

edges := []leiden.Edge{
    {Source: "A", Target: "B", Weight: 1.0},
    {Source: "B", Target: "C", Weight: 1.0},
    {Source: "A", Target: "C", Weight: 1.0},
    {Source: "D", Target: "E", Weight: 1.0},
    {Source: "E", Target: "F", Weight: 1.0},
    {Source: "D", Target: "F", Weight: 1.0},
    {Source: "C", Target: "D", Weight: 0.1}, // weak bridge
}

result, err := leiden.Leiden(edges, leiden.DefaultOptions())
// result.Partition: {"A":0, "B":0, "C":0, "D":1, "E":1, "F":1}
```

## Implementation

This library is a Go port of [graspologic-native](https://github.com/graspologic-org/graspologic-native) (Microsoft Research, MIT license) — the Rust Leiden implementation used in production by Microsoft GraphRAG. Zero external dependencies.

## License

MIT
