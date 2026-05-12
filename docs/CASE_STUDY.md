# go-leiden v0.1.0 — Case Study

**Build report for peer review. Last updated 2026-05-12.**

> *"The first complete open-source Go library ported end-to-end by autonomous AI agents from a product DAG, with zero human source-code commits."*

This document substantiates that claim. It defines terms precisely, describes the method, lists every verifiable artifact, and discloses the human role honestly. Every number is verifiable from the public git history, the `docs/manifests/` prompt pages, the [OpenPraxis](https://github.com/k8nstantin/OpenPraxis) engine source, and the [graspologic-native](https://github.com/graspologic-org/graspologic-native) Rust reference.

---

## 1. Definitions

**Synthetic library.** A software library whose source code is produced by autonomous AI agent processes operating against a structured specification (a *product DAG*), rather than by humans typing into editors or by humans iteratively prompting a chatbot. Distinguishing properties:

1. The agents run as task subprocesses, not as inline assistants.
2. Execution is orchestrated by a DAG of dependent manifests, each with a deliverable contract.
3. Each task runs in isolated state (a fresh git worktree off `origin/main`).
4. Correctness is verified by an independent server-side audit the agent cannot see or influence (the OpenPraxis [Trace-Grounded Feedback Loop](https://github.com/k8nstantin/OpenPraxis)).

Not synthetic under this definition: code completed by GitHub Copilot, code generated turn-by-turn in an IDE chat panel, code written by humans referring to AI suggestions. Those involve a human author at every step. Synthetic means the human authored the *spec and the orchestration*, the agent authored the *code*.

**Zero human source-code commits.** No `.go` source file was created or modified by a human in any commit. Verifiable from the git history (§5). The only file with a human-authored line of Go is `go.mod` — three lines of module declaration in the initial scaffold commit. Documentation, the product DAG image, manifest prompt pages, and the repository scaffold were human-authored as *build inputs*.

**Agentic OS.** [OpenPraxis](https://github.com/k8nstantin/OpenPraxis) — a peer-to-peer workflow engine for autonomous coding agents. Single Go binary: MCP stdio server + HTTP dashboard + mDNS peer discovery + Automerge CRDT sync. Products break into manifests, manifests into tasks, tasks spawn LLM agent subprocesses in isolated git worktrees.

**First, to our knowledge.** We have not located a prior published Go module produced under all four properties above. We acknowledge prior art in adjacent categories in §9.

---

## 2. Method

### 2.1 Product DAG

The build was modelled as five sequential manifests, each a node in the OpenPraxis entity graph:

```
Product: go-leiden (uid 019e1c3d-07e4-7b69-a43f-1bc6cb68b179)
│
├─ Skill: Senior Developer Coding Practices
├─ Skill: Open-Source Go Library Engineering
│
├─ M1 — Core data structures        →  CompactNetwork (CSR), Clustering, Edge, errors
├─ M2 — Algorithm phases            →  local-move, refinement, aggregation
├─ M3 — Quality functions           →  Modularity, CPM, resolution scaling
├─ M4 — Public API                  →  Leiden(), HierarchicalLeiden(), go.mod, README
└─ M5 — Tests + benchmarks          →  karate-club validation, fuzz, benchmarks
```

Each manifest had `depends_on` edges enforcing sequential execution. M2 could not fire until M1's deliverables passed audit, and so on. The full per-manifest prompts are public in [`docs/manifests/m1.md`](manifests/m1.md) through [`docs/manifests/m5.md`](manifests/m5.md).

### 2.2 Cascading prompts

Each task ran with four layers of context concatenated into the agent's system prompt:

```
Product prompt   →  "what are we building, why, module path, reference Rust source files"
      ↓
Manifest prompt  →  "this component's role, dependencies, deliverables, file layout"
      ↓
Task prompt      →  "exact types, function signatures, formulas, files to read first"
      ↓
Skill prompts    →  "Go library standards, testing conventions, zero-dep rules"
```

The agent that ran M2/T1 entered its subprocess knowing: the product it was contributing to, where M2 sat in the chain, the exact `graspologic-native` Rust files to read before writing a line, and the precise quality-value-increment (QVI) formula to implement. The agent could not invent — every formula and signature was specified upstream.

### 2.3 Execution model

Each task launched as a subprocess against a freshly-checked-out worktree from `origin/main`. The agent had a turn budget (default 200), a wall-clock budget, and a `deliverable_missing` exit code if the contract wasn't met. On task completion, OpenPraxis:

1. Ran `go test ./...`, `go vet`, and the audit checks.
2. If audits passed, squash-merged the task branch back to `main`.
3. Fired the next manifest's task.

If audits failed twice in a row with the same root cause, the **autonomous proposer** activated: it analyzed the failure traces, generated an improved task prompt, tested it against pass rates, and kept the new prompt only if it improved by ≥5%. Prompts self-improve without human intervention.

---

## 3. Inputs (human-authored)

The human contribution to v0.1.0, exhaustively:

| Input | What | Files / lines |
|---|---|---|
| Repository scaffold | `go.mod`, initial `README.md` | `go.mod` (3 lines), `README.md` |
| Product definition | go-leiden product entity in OpenPraxis, pointing at graspologic-native as source | OpenPraxis entity `019e1c3d-07e4-7b69-a43f-1bc6cb68b179` |
| Manifest prompts | Five manifest specs with deliverable contracts | [`docs/manifests/m1.md`](manifests/m1.md) – [`m5.md`](manifests/m5.md) |
| Skill prompts | Two reusable skill prompts | "Senior Developer Coding Practices", "Open-Source Go Library Engineering" (stored in OpenPraxis) |
| Source library | `graspologic-native` repo path passed to agents to read | external reference, MIT |
| Review of PR #6 | Senior-architect code review finding ~20 blocking/risk items | comments thread on PR #6 |

**No `.go` source code was authored by a human at any stage.**

---

## 4. The build run

### 4.1 Timeline (UTC −04:00)

| Time | Event | Commit |
|---|---|---|
| 09:07:45 | Human: `init: go module + README scaffold` | `e12c4ec` |
| 09:10 – 11:28 | Human: prepares manifest prompt pages, DAG image, repo docs | `e4862c3` → `4523a95` |
| 11:28:40 | **All manifest prompts published — agent build starts** | `4523a95` |
| 14:15:04 | **M1–M5 complete; squash-merge lands** | `5dd09c2` |
| 14:15 – 15:59 | Senior-architect code review of PR #6 (human reviews; ~20 findings) | PR #6 thread |
| 15:59:08 | **M6 review-fix iteration complete; merge lands** | `aacc8ea` |
| 16:22:07 | v0.1.0 tagged + released | `b8c7097` |
| 16:25:12 | Post-release docs cleanup | `64d37e2` |

**Agent execution window** (manifests published → M1–M5 squash): **2h 46m 24s**. Adding M6 review iteration (14:15:04 → 15:59:08) brings total active build time to **4h 30m 24s**. The "2h 49m" figure quoted elsewhere is the M1–M5 wall-clock at minute resolution from OpenPraxis trace; the git-derived 2h 46m is the conservative lower bound.

### 4.2 Turn count

OpenPraxis recorded **609 agent turns** across M1/T1, M2/T1, M3/T1, M4/T1, M5/T1, plus the M6 review-fix tasks. A per-manifest breakdown is preserved in the OpenPraxis task store but is not included here because the squash-and-rewrite cleanup in §6 prevents a clean automated extraction from the post-release database state. The headline total is the trace-recorded sum at build completion.

### 4.3 Output

At tag `v0.1.0`:

- **27 `.go` files**, **4,340 total lines** of Go.
- **108 test/benchmark/fuzz functions** (97 `Test*`, 8 `Benchmark*`, 4 `Fuzz*`).
- **Public API surface:**
  - Functions: `Leiden`, `HierarchicalLeiden`, `Modularity`, `NewClustering`, `NewClusteringFromAssignment`, `NewSingletonClustering`, `NewCompactNetwork`, `NewCompactNetworkWithNodeWeights`, `DefaultOptions`, `GroupBy`, `CommunityCount`.
  - Types: `Clustering`, `CompactNetwork`, `Edge`, `Options`, `Result`, `HierarchicalResult`, `LevelResult`.
- **Zero external dependencies** (`go.mod` has no `require` block).
- **MIT licensed**, module path `github.com/k8nstantin/go-leiden`.

---

## 5. Verification

### 5.1 Algorithm correctness

- **Refinement-from-singletons** is implemented per Traag, Waltman & van Eck (2019) [equations (14) and (15)](https://www.nature.com/articles/s41598-019-41695-z), with the well-connectedness γ-test. Independent senior review of PR #6 confirmed the formula derivation in `refinement.go`.
- **Karate club validation:** `TestValidation_KarateClub_ModularityWithinPublishedRange` asserts modularity is in [0.35, 0.45], matching the range produced by `graspologic-native` and `leidenalg`. `TestIntegration_KarateClub_TwoLeadersSeparated` asserts that Mr. Hi and Officer end up in distinct communities.
- **CPM move-delta** invariant: `TestCPMQuality_MoveDeltaMatchesRecompute` asserts the incremental ΔQ formula matches a full re-evaluation, preventing drift bugs.
- **Modularity** matches the textbook formula: `TestModularityQuality_MatchesTextbookFormula`.
- **Resolution scaling** uses the graspologic-native–corrected formula, not the original CWTSLeiden Java formula (this is documented in `quality.go`).
- **Fuzz tests** for `NewCompactNetwork`, `Modularity`, `Leiden`, and `HierarchicalLeiden` run against arbitrary inputs; included in `fuzz_test.go`.

### 5.2 Git verifiability

Every claim in §3 and §4.1 is reproducible from a fresh clone:

```bash
git clone https://github.com/k8nstantin/go-leiden
cd go-leiden
git log --reverse --format='%ai %h %s' main          # full commit history
git ls-tree -r --name-only v0.1.0 | grep '\.go$'    # 27 files
git show 5dd09c2 --stat                              # M1–M5 squash: all .go files added in one commit
git show e12c4ec --stat                              # initial human scaffold: README + go.mod only
go test ./...                                        # all tests pass
go vet ./...
```

The only commits authored before `5dd09c2` (M1–M5 squash) touch `README.md`, `go.mod`, `docs/`, and `assets/` — never any `.go` file.

---

## 6. Caveats and honest disclosure

### 6.1 Commit authorship rewrite

OpenPraxis's task subprocesses produce git commits authored by the agent's configured identity (Claude, in this build). After v0.1.0 was tagged, the commit history was rewritten with `git filter-branch --env-filter` and `--msg-filter` to:

1. Replace agent author identity with the project owner's identity (`Constantin Alexander <constantin@dedomena.io>`).
2. Strip `Co-Authored-By: Claude` trailers from commit messages.

This is a presentation choice for the release, not a substantive claim about who wrote the code. The substantive fact remains: **no human wrote any of the `.go` source.** The rewrite is acknowledged here so a peer reviewer comparing `git log` against this document doesn't infer authorship from `%an`. The original agent attribution is recoverable from OpenPraxis trace records.

### 6.2 The M6 review iteration

PR #6 (the M1–M5 output) received a senior-architect code review that identified ~20 blocking/risk/library-hygiene items. **The review was human-authored.** The *fixes* in response to the review (M6/T1 through M6/T4) were agent-authored, executed via the same task-subprocess model. This is consistent with the synthetic-library definition (human spec/review, agent code), but a reviewer should know that human oversight materially shaped v0.1.0.

### 6.3 The repository scaffold

`go.mod` was created by the human. It contains three lines:

```
module github.com/k8nstantin/go-leiden

go 1.22
```

This is module-system boilerplate, not library code. We list it explicitly rather than gloss over it.

### 6.4 "First, to our knowledge"

We use the qualified form throughout. We surveyed pkg.go.dev, GitHub topic searches, and the public output of similar agent platforms (Devin, Claude Code, GPT-Engineer, Aider, SWE-agent). We have not found a prior published Go library that meets all four synthetic-library criteria (§1). We welcome correction with citation.

---

## 7. Reproducibility

A reviewer can rebuild the methodology end-to-end:

| Artifact | Location |
|---|---|
| Library source | https://github.com/k8nstantin/go-leiden at tag `v0.1.0` |
| Module proxy | https://proxy.golang.org/github.com/k8nstantin/go-leiden/@v/v0.1.0.info |
| Manifest prompts | [`docs/manifests/m1.md`](manifests/m1.md) – [`m5.md`](manifests/m5.md) |
| Product DAG image | [`assets/dag.png`](../assets/dag.png) |
| Engine (OpenPraxis) | https://github.com/k8nstantin/OpenPraxis |
| Source library ported | https://github.com/graspologic-org/graspologic-native (Rust, MIT) |
| Source algorithm reference | https://www.nature.com/articles/s41598-019-41695-z (Traag et al. 2019) |
| Original C++ reference | https://github.com/vtraag/leidenalg (vtraag, MIT) |

The manifest prompts in [`docs/manifests/`](manifests/) contain the *actual prompt text* fed to the agents. A reviewer can read M2's prompt and verify it specifies (a) the three Leiden phases, (b) which `graspologic-native` Rust files to consult, and (c) the QVI formula — and then read `refinement.go` and verify the agent followed the spec.

---

## 8. What this case study does not claim

To avoid overclaiming:

- **Not "the first AI-generated code on GitHub."** AI-generated code is widespread. The synthetic-library definition (§1) is narrower.
- **Not "AGI" or "self-directed."** The agents executed against a human-authored DAG with human-authored prompts. The novelty is the *end-to-end production of a published, importable Go module* with no human in the implementation loop.
- **Not "no human reviewed the code."** PR #6 had a thorough human review; M6 was agent-authored fixes against human-authored findings.
- **Not "first Leiden in Go."** [`vsuryav/leiden-go`](https://github.com/vsuryav/leiden-go) (Nov 2025) preceded go-leiden as a Go Leiden module. We discuss this in [§ Prior art in Go](../README.md#prior-art-in-go) of the README.
- **Not a claim of state-of-the-art performance.** Performance is competitive with `graspologic-native` on small/medium graphs; we have not benchmarked at the >1M edge scale used in production GraphRAG deployments.

---

## 9. Prior art

### 9.1 In Go-language Leiden implementations

- **[`github.com/vsuryav/leiden-go`](https://github.com/vsuryav/leiden-go)** (Nov 2025, MIT). Preceded go-leiden as a Go Leiden module. Substantively different scope: see [Prior art in Go](../README.md#prior-art-in-go) and [Issue #10](https://github.com/k8nstantin/go-leiden/issues/10).
- **[`gonum/graph/community`](https://pkg.go.dev/gonum.org/v1/gonum/graph/community)**. Louvain only — does not implement Leiden.

### 9.2 In agent-produced software more broadly

- **GitHub Copilot, Claude Code, Cursor, Aider** — inline AI assistants. Human in the loop at every keystroke.
- **GPT-Engineer, MetaGPT, AutoGPT** — research/demo agents producing prototypes, mostly Python, mostly not subsequently published as importable libraries with verified algorithm correctness.
- **Devin (Cognition Labs), Magic, SWE-agent** — autonomous coding agents demonstrating multi-file PR generation. Not, to our knowledge, the source of a published Go module on pkg.go.dev meeting §1's four criteria.

### 9.3 In autonomous build orchestration

- **OpenHands (formerly OpenDevin)**, **AutoGen**, **CrewAI**, **LangGraph** — multi-agent orchestration frameworks. None of which, to our knowledge, have shipped a published Go library to pkg.go.dev as the product of a single end-to-end DAG run.

---

## 10. Open invitations

If you are a reviewer and you find:

- A prior published Go module on pkg.go.dev meeting §1's four criteria with date earlier than 2026-05-12 — please file an issue; we will update §1.4 and §9.
- An algorithmic discrepancy between `refinement.go`/`aggregation.go` and Traag (2019) — please file an issue with the equation reference.
- A reproducibility gap where an artifact in §7 is missing — please file an issue.

---

## References

- Traag, V.A., Waltman, L. & van Eck, N.J. (2019). *From Louvain to Leiden: guaranteeing well-connected communities.* Scientific Reports 9, 5233. https://www.nature.com/articles/s41598-019-41695-z
- [graspologic-native](https://github.com/graspologic-org/graspologic-native) — Microsoft Research and Johns Hopkins University. MIT. The Rust source go-leiden ports from.
- [leidenalg](https://github.com/vtraag/leidenalg) — Vincent Traag's original C++/Python implementation. MIT.
- [CWTSLeiden/networkanalysis](https://github.com/CWTSLeiden/networkanalysis) — Java reference by the original authors.
- [OpenPraxis](https://github.com/k8nstantin/OpenPraxis) — the agentic OS that built go-leiden.
