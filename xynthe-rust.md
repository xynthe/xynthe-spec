# Xynthe Rust Specification (`xynthe-rs`)

> **The cognitive-execution substrate for autonomous intelligence.**  
> *Version 2.0 | April 2026 | Rust 2024 Edition*

[![Crates.io](https://img.shields.io/crates/v/xynthe-rs.svg)](https://crates.io/crates/xynthe-rs)
[![Documentation](https://img.shields.io/docsrs/xynthe-rs/latest)](https://docs.rs/xynthe-rs)
[![CI](https://img.shields.io/github/actions/workflow/status/xynthe/xynthe-rs/ci.yml)](https://github.com/xynthe/xynthe-rs/actions)
[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

`xynthe-rs` is the official Rust implementation of the **Xynthe Orchestration Framework**, providing the foundational substrate for architecting persistent, deterministic, and GIAM-aligned autonomous agents. It translates the Xynthe cognitive primitives into a type-safe, async-native, and WASM-compilable runtime, enforcing behavioral determinism through immutable execution traces and linear resource semantics.

---

## 📐 Core Architecture

Xynthe treats intelligence as *orchestrated continuity*. The Rust specification materializes this through three composable primitives that form the minimal cognitive substrate:

| Primitive | Rust Mapping | Purpose |
|-----------|--------------|---------|
| **Thought Streams** | `async` channels with typed `ThoughtEvent` enums | Structured reasoning pathways (`observation`, `hypothesis`, `intention`, `reflection`) with provenance tracking |
| **Capability Bindings** | Trait-based `Binding` + linear `Token` wrappers | Type-safe, contract-enforced interfaces to tools, models, and external systems |
| **Context Fabrics** | `ContextFabric<T>` backed by CRDTs & temporal indexing | Multi-layered memory (`immediate`, `working`, `episodic`, `semantic`) with async query/update semantics |

These primitives compose into a **GIAM-Aligned Temporal Execution Loop (G-TEL)**, managed by the `ExecutionKernel` and orchestrated via the `GiamRouter`.

---

## 🚀 Quick Start

Add to your `Cargo.toml`:
```toml
[dependencies]
xynthe-rs = { version = "2.0", features = ["tokio-runtime", "wasm-sandbox", "trace-storage"] }
```

Minimal agent initialization:
```rust
use xynthe_rs::prelude::*;
use xynthe_rs::context::ContextFabric;
use xynthe_rs::streams::ThoughtStream;
use xynthe_rs::bindings::CapabilityBinding;
use xynthe_rs::kernel::ExecutionKernel;

#[tokio::main]
async fn main() -> Result<(), XyntheError> {
    // 1. Initialize temporal context fabric
    let fabric = ContextFabric::new(CrdtConfig::default())
        .await?;

    // 2. Create a thought stream for task execution
    let mut stream = ThoughtStream::builder()
        .with_layer(StreamLayer::Working)
        .with_confidence_threshold(0.85)
        .build();

    // 3. Define a capability binding (e.g., web search)
    let search_binding = WebSearchBinding::new(ApiKey::from_env()?)
        .with_contract(SearchContract {
            preconditions: vec![Precondition::HasQuery],
            effects: vec![Effect::AddsEvidence],
        });

    // 4. Spin up the execution kernel
    let kernel = ExecutionKernel::new(fabric, stream, search_binding)
        .with_trace_recorder(TraceStorage::ContentAddressed)
        .with_human_boundary(AutonomyBoundary::PreExecutionReview)
        .start()
        .await?;

    // 5. Ingest intent & run the G-TEL loop
    kernel.submit_intent("Research quantum error correction thresholds for superconducting qubits").await?;
    kernel.run_until_completion().await?;

    Ok(())
}
```

---

## 🧩 API & Type Specifications

### Thought Streams
Typed channels carrying structured cognitive events with temporal provenance:
```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum ThoughtEvent {
    Observation { content: Payload, source: SourceId, confidence: f64 },
    Hypothesis { claim: String, evidence_refs: Vec<TraceId>, confidence: f64 },
    Intention { goal: GoalSpec, constraints: Vec<ConstraintToken> },
    Reflection { meta: MetaCognitiveUpdate, topology_adjustment: TopologyDelta },
}

pub struct ThoughtStream<T> {
    inner: async_channel::Receiver<T>,
    topology: DynamicHyperGraph,
    operators: StreamOperators, // merge, filter, reflect, project
}
```

### Capability Bindings
Enforce tool safety via Rust's ownership model + explicit linear tokens:
```rust
pub trait CapabilityBinding: Send + Sync {
    fn describe(&self) -> BindingSpec;
    async fn invoke(&self, state: CognitiveState, token: LinearUseToken) -> Result<ExecutionTrace>;
    async fn simulate(&self, state: CognitiveState) -> Result<PredictedEffect>;
    async fn reflect(&self, trace: &ExecutionTrace) -> Result<CapabilityTrustDelta>;
}
```
*Linear execution is enforced by consuming `LinearUseToken` on `invoke()`, preventing duplicate or out-of-order tool calls at compile-time.*

### Context Fabrics
Temporally indexed, CRDT-synchronized memory substrate:
```rust
pub struct ContextFabric {
    layers: LayeredStorage, // Immediate, Working, Episodic, Semantic
    temporal_index: TemporalIndex,
    sync_engine: CrdtSyncEngine,
}

impl ContextFabric {
    pub async fn recall(&self, query: &str, t_range: TimeWindow) -> Vec<CognitiveEvent>;
    pub async fn project(&self, trend: &TrendSpec, t_future: TemporalInstant) -> ProjectionResult;
    pub async fn diff(&self, t1: TraceId, t2: TraceId) -> SemanticDelta;
    pub async fn consolidate(&self, events: &[CognitiveEvent]) -> SemanticKnowledge;
}
```

---

## 🛡️ Safety, Alignment & Determinism

| Guarantee | Rust Implementation |
|-----------|---------------------|
| **Behavioral Determinism** | Immutable `ExecutionTrace` stored in content-addressed storage; cryptographic hashing enables exact replay debugging |
| **Constraint Propagation** | `ConstraintToken` structs flow through execution graphs; violations trigger compile-time type errors or runtime halts via `Result<XyntheError>` |
| **Resource Safety** | Linear token consumption + `Drop` semantics prevent resource leaks; `unsafe` boundaries strictly isolated to WASM sandbox |
| **Human-in-the-Loop** | `AutonomyBoundary` enum gates execution; `pause()`, `inject_correction()`, `rollback()` exposed via `ExecutionKernel` API |
| **Value Alignment** | `ValueReflectionLoop` periodically samples traces, computes alignment deltas, and updates topology via `meta_cognitive_adjust()` |

---

## 🌐 GIAM Integration & Routing

`xynthe-rs` provides first-class routing to the **General Intelligence Ascension Model** hierarchy:

```rust
use xynthe_rs::giam::{GiamLevel, GiamRouter, IntelligencePayload};

let router = GiamRouter::new()
    .register_level(GiamLevel::SI, SiExecutionKernel::new())
    .register_level(GiamLevel::UI, PersistentMemoryLayer::new())
    .register_level(GiamLevel::HI, AdaptiveSkillSystem::new())
    .register_level(GiamLevel::SPI, SubagentNetwork::new())
    .register_level(GiamLevel::ULI, GlobalOptimizationEngine::new())
    .register_level(GiamLevel::TI, UniversalExecutionFabric::new());

// Intent is automatically decomposed and routed
router.dispatch_intent("Optimize global supply chain routing under carbon constraints").await?;
```

| GIAM Level | Xynthe Subsystem | Responsibility |
|------------|------------------|----------------|
| `SI` | `ExecutionKernel` | Task lifecycle, deterministic scheduling |
| `UI` | `PersistentMemoryLayer` | Cross-session context, goal continuity |
| `HI` | `AdaptiveSkillSystem` | Self-improving workflows, meta-learning |
| `SPI` | `SubagentNetwork` | Multi-agent coordination, role specialization |
| `ULI` | `GlobalOptimizationEngine` | Predictive execution, resource allocation |
| `TI` | `UniversalExecutionFabric` | Cross-substrate runtime, distributed coordination |

---

## 🛠️ Development & Build

### Feature Flags
| Flag | Description |
|------|-------------|
| `tokio-runtime` | Async executor (default) |
| `wasm-sandbox` | Compile to `wasm32-unknown-unknown` for capability isolation |
| `trace-storage` | Content-addressed trace persistence |
| `crdt-sync` | Conflict-free replication for multi-device context fabrics |
| `giam-routing` | Full GIAM level dispatch & optimization |

### Building & Testing
```bash
# Core build
cargo build --release

# WASM sandbox compilation
cargo build --target wasm32-unknown-unknown --features wasm-sandbox

# Run deterministic trace replay tests
cargo test --test trace_replay

# Benchmark cognitive loop throughput
cargo bench --bench giam_dispatch
```

### Tracing & Debugging
Xynthe integrates with `tracing` and `opentelemetry`:
```rust
use tracing_subscriber::EnvFilter;
tracing_subscriber::FmtSubscriber::builder()
    .with_env_filter(EnvFilter::new("xynthe=debug,execution_trace=trace"))
    .init();
```
Immutable traces can be exported to `xynthe-studio` for visual topology debugging and behavioral cloning.

---

## 📖 References

1. *Xynthe: A Unified Orchestration Framework for Autonomous Intelligence* (White Paper v1.0–v2.0)
2. Milner, R. (1999). *Communicating and Mobile Systems: The π-Calculus*
3. Shapiro, L. (2025). *Process Algebra for Cognitive Architectures*. JAGI 16(2)
4. GIAM Consortium (2025–2026). *General Intelligence Ascension Model: Tier Specifications*
5. Xynthe Architecture Working Group (2026). *Deterministic Trace Replay & Linear Type Safety in Autonomous Systems*

---

## 📜 License

Distributed under **Dust Open Source License**.   
See `LICENSE`

---

> *AGI made intelligence general. GIAM made intelligence hierarchical. XYNTHE makes intelligence usable.*  
> **Where humans meet the future of execution.**
