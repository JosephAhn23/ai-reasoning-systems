# AI Systems Knowledge Base — Source Index

Hyperlinks to every real system distilled in [`ai_systems_knowledge_base.md`](./ai_systems_knowledge_base.md), grouped to match the file's Part structure.

**Related context packs**

| File | Role |
|---|---|
| [`ai_systems_knowledge_base.md`](./ai_systems_knowledge_base.md) | Reasoning, agents, systems engineering, + agent craft (Part IV from system architecture) |
| [`system architecture.md`](./system%20architecture.md) | Standalone agent craft / system-architecture guide |
| [`frontend_design_knowledge_base.md`](./frontend_design_knowledge_base.md) | Frontend design judgment (UI systems, spacing, motion, landings, …) |
| [`ai_fullstack_knowledge_base.md`](./ai_fullstack_knowledge_base.md) | Combined pack: system architecture + frontend design |

## Estimated Impact (Qualitative — Not a Measured Benchmark)

**Important caveat first:** no evaluation suite was run to produce these numbers — there is no SWE-bench, no held-out coding tasks, no A/B test. This is a subjective estimate of relative coding/engineering capability, scored out of 100, based on general reasoning about model tiers and about what a large, curated context document can and can't do for a smaller model. Treat it as an informed guess with stated reasoning, not as data — do not cite these numbers as measured results.

| Model | Estimated score /100 | Reasoning |
|---|---|---|
| Frontier flagship (e.g. GPT-5.x-class) | ~92–96 | Largest pretraining + RL investment among compared models; strongest at novel architectural judgment calls, long-horizon multi-file reasoning, and recovering from ambiguous specs without hand-holding. Context documents help these models less in relative terms — they already have deep internalized priors across most of the domains this file covers. |
| Claude (latest flagship) | ~90–95 | Comparable tier to the above; particularly strong on agentic tool-use and long-context coherence, which is exactly the category this knowledge base's Part II is about — so a curated agent-systems doc closes less of a gap here than it would for a weaker base model. |
| **DeepSeek V4 Flash + this knowledge base as context** | ~78–85 | A context document can supply *declarative* knowledge (named patterns, tradeoffs, "what to carry away" checklists) that substitutes reasonably well for prior exposure to a system's design — e.g., recognizing "this is a leaderholder pattern" or "this needs a sparse index" once it's been named and explained. It does **not** substitute for the model's own architecture, parameter count, or RL-trained judgment: applying a named pattern correctly to a novel, messy, partially-specified real codebase is a different skill than recognizing the pattern in a clean explanation. Expect the gain to concentrate in recall/vocabulary/breadth-of-consideration, not in raw multi-step reasoning quality. |
| DeepSeek V4 Flash (no added context) | ~65–75 | Smaller/faster-tier model; likely has real gaps in some of the more specialized domains this file covers (e.g., LSM compaction tuning, MVCC edge cases, Raft safety proofs) simply from less exposure at pretraining/RL time relative to frontier-scale models. |

**Why the gap between "with context" and "without context" is real but bounded:** a good context document raises the floor (fewer flatly wrong claims, better vocabulary for naming what's happening, a checklist to self-check against) more than it raises the ceiling (it can't make a smaller model reason as deeply through a genuinely novel problem a frontier model hasn't seen a close analogue of). If you want an actual number instead of an estimate, the right next step is running this file as a system-prompt/context addition through a real coding benchmark (SWE-bench-style tasks) with and without it, on the same model, and diffing the pass rate — happy to help set that up if useful.

## Part I — Reasoning & Math

| File | System | Source |
|---|---|---|
| LOGIC.md | Lean 4 | https://github.com/leanprover/lean4 |
| FORMALIZATION.md | Mathlib4 | https://github.com/leanprover-community/mathlib4 |
| PROOF_SEARCH.md | AlphaGeometry | https://github.com/google-deepmind/alphageometry |
| PIPELINES.md | DSPy | https://github.com/stanfordnlp/dspy |
| DISCOVERY.md | PySR | https://github.com/MilesCranmer/PySR |

## Part II — Coding & Agent Systems

| File | System | Source |
|---|---|---|
| ORCHESTRATION.md | OpenHands / Agent Canvas | https://github.com/All-Hands-AI/OpenHands |
| EDIT_FORMATS.md | Aider | https://github.com/Aider-AI/aider |
| ACI.md | SWE-agent | https://github.com/SWE-agent/SWE-agent |
| SDK_LAYERING.md | Cline | https://github.com/cline/cline |
| TOOL_SURFACE.md | Claude Code | https://docs.claude.com/en/docs/claude-code |
| LANGGRAPH.md | LangGraph | https://github.com/langchain-ai/langgraph |
| AGENTS_SDK.md | OpenAI Agents SDK | https://github.com/openai/openai-agents-python |
| MCP_PROTOCOL.md | Model Context Protocol | https://github.com/modelcontextprotocol/specification |
| LIVE_DOCS_CONTEXT.md | Context7 | https://github.com/upstash/context7 |
| 12_FACTOR_AGENTS.md | 12-Factor Agents | https://github.com/humanlayer/12-factor-agents |
| PRODUCTION_SYSTEM_PROMPTS.md | Extracted production system prompts | https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools |

## Part III — Systems Engineering

### Distributed Systems

| File | System | Source |
|---|---|---|
| K8S_CONTROL_PLANE.md | Kubernetes | https://github.com/kubernetes/kubernetes |
| CONSENSUS.md | etcd (Raft) | https://github.com/etcd-io/etcd |
| SHARDING.md | TiKV | https://github.com/tikv/tikv |
| REPLICATION.md | CockroachDB | https://github.com/cockroachdb/cockroach |
| DURABLE_EXECUTION.md | Temporal | https://github.com/temporalio/temporal |
| DISTRIBUTED_COMPUTE.md | Ray | https://github.com/ray-project/ray |
| LOG_AS_DATABASE.md | Apache Kafka | https://github.com/apache/kafka |
| LIGHTWEIGHT_MESSAGING.md | NATS | https://github.com/nats-io/nats-server |

### Databases

| File | System | Source |
|---|---|---|
| QUERY_PLANNING.md | PostgreSQL | https://github.com/postgres/postgres |
| BTREE_STORAGE.md | SQLite | https://www.sqlite.org/src/ (docs: https://www.sqlite.org/arch.html) |
| IN_MEMORY_STRUCTURES.md | Redis | https://github.com/redis/redis |
| LSM_TREES.md | RocksDB | https://github.com/facebook/rocksdb |
| VECTORIZED_OLAP.md | DuckDB | https://github.com/duckdb/duckdb |
| EXTREME_COLUMNAR_SCANS.md | ClickHouse | https://github.com/ClickHouse/ClickHouse |

### Operating Systems

| File | System | Source |
|---|---|---|
| KERNEL_SCHEDULING.md | Linux kernel (CFS/EEVDF, VFS) | https://github.com/torvalds/linux |
| MINIMAL_OS_DESIGN.md | xv6 (MIT) | https://github.com/mit-pdos/xv6-riscv |
| FROM_SCRATCH_ENGINEERING.md | SerenityOS / Ladybird | https://github.com/SerenityOS/serenity · https://github.com/LadybirdBrowser/ladybird |
| VERIFIED_MICROKERNEL.md | seL4 | https://github.com/seL4/seL4 |

### Compilers

| File | System | Source |
|---|---|---|
| LLVM_IR.md | LLVM | https://github.com/llvm/llvm-project |
| MLIR.md | MLIR | https://github.com/llvm/llvm-project/tree/main/mlir |
| TREE_SITTER.md | Tree-sitter | https://github.com/tree-sitter/tree-sitter |
| CLANG.md | Clang | https://github.com/llvm/llvm-project/tree/main/clang |

### Networking

| File | System | Source |
|---|---|---|
| ENVOY.md | Envoy | https://github.com/envoyproxy/envoy |
| CADDY.md | Caddy | https://github.com/caddyserver/caddy |
| NGINX.md | nginx | https://github.com/nginx/nginx |
| CURL.md | curl | https://github.com/curl/curl |

### ML Infrastructure

| File | System | Source |
|---|---|---|
| VLLM.md | vLLM | https://github.com/vllm-project/vllm |
| LLAMA_CPP.md | llama.cpp | https://github.com/ggml-org/llama.cpp |
| TRITON_INFERENCE_SERVER.md | Triton Inference Server | https://github.com/triton-inference-server/server |
| TENSORRT_LLM.md | TensorRT-LLM | https://github.com/NVIDIA/TensorRT-LLM |
| SGLANG.md | SGLang | https://github.com/sgl-project/sglang |
| DEEPSPEED.md | DeepSpeed | https://github.com/deepspeedai/DeepSpeed |

### Observability

| File | System | Source |
|---|---|---|
| PROMETHEUS.md | Prometheus | https://github.com/prometheus/prometheus |
| GRAFANA.md | Grafana | https://github.com/grafana/grafana |
| OPENTELEMETRY.md | OpenTelemetry | https://github.com/open-telemetry/opentelemetry-specification |
| JAEGER.md | Jaeger | https://github.com/jaegertracing/jaeger |

### Performance

| File | System | Source |
|---|---|---|
| MIMALLOC.md | mimalloc | https://github.com/microsoft/mimalloc |
| JEMALLOC.md | jemalloc | https://github.com/jemalloc/jemalloc |
| FOLLY.md | Folly | https://github.com/facebook/folly |
| ABSEIL_CPP.md | abseil-cpp | https://github.com/abseil/abseil-cpp |

### Build Systems

| File | System | Source |
|---|---|---|
| BAZEL.md | Bazel | https://github.com/bazelbuild/bazel |
| CMAKE.md | CMake | https://gitlab.kitware.com/cmake/cmake |
| BUCK2.md | Buck2 | https://github.com/facebook/buck2 |

## Part IV — Agent System Architecture

Merged into [`ai_systems_knowledge_base.md`](./ai_systems_knowledge_base.md) from [`system architecture.md`](./system%20architecture.md) (SYSTEM_PROMPT, PLANNING, AGENTS, ARCHITECTURE, TOOL_USAGE, TESTING, CODE_REVIEW, STYLE_GUIDE, …). The standalone file remains for lighter agent-craft-only context.

## Top 10 repos (maximize engineering IQ)

If picking only ten repositories to study:

1. OpenHands
2. LLVM
3. PostgreSQL
4. Linux Kernel
5. Kubernetes
6. Redis
7. Ray
8. Envoy
9. vLLM
10. SQLite

---

## Frontend & Fullstack

Frontend design lives in [`frontend_design_knowledge_base.md`](./frontend_design_knowledge_base.md). For agent craft + UI in one file, use [`ai_fullstack_knowledge_base.md`](./ai_fullstack_knowledge_base.md).

Frontend is one of the biggest weaknesses of cheaper models. They often:

* make everything look similar,
* have weak visual hierarchy,
* poor spacing,
* mediocre UX,
* generic dashboards,
* inconsistent component composition.

The sections below are the study map distilled into that frontend KB.

### UI Systems

These teach how professionals build interfaces.

1. shadcn/ui
2. Magic UI
3. Aceternity UI
4. Origin UI
5. HeroUI (formerly NextUI)

### Animation

Models are terrible at tasteful animation.

1. Motion (Framer Motion)
2. React Bits
3. Animate UI
4. Motion Primitives
5. GSAP examples

### Dashboard Design

1. Tremor
2. Taxonomy
3. Refine
4. Cal.com
5. Supabase Dashboard

### Modern Landing Pages

These teach composition.

1. Vercel
2. Linear
3. Raycast
4. Stripe
5. Resend

### Design Systems

1. Radix UI
2. Material UI
3. Chakra UI
4. Mantine
5. Adobe React Aria

### UX

1. Laws of UX
2. Refactoring UI
3. Nielsen Norman Group
4. Apple HIG
5. Material Design 3

### Visualization

1. Tremor
2. Recharts
3. Nivo
4. ECharts
5. Observable

### Forms

1. React Hook Form
2. TanStack Form
3. Zod
4. Conform
5. Vest

### Icons

1. Lucide
2. Heroicons
3. Tabler Icons
4. Phosphor
5. Remix Icons

### Color

1. Tailwind Colors
2. Radix Colors
3. Open Color
4. Material Color
5. OKLCH examples

### Creativity

These matter way more than people realize. Scrape examples from:

* Awwwards
* Godly
* Land-book
* Mobbin
* Lapa Ninja

Not for copying — for learning spacing, hierarchy, typography, composition, layouts, and interaction patterns.

### Frontend Context Engineering shape (in the KB)

```
Frontend/
  Design Systems/ Typography/ Spacing/ Animations/
  Dashboards/ Landing Pages/ Mobile UI/ Accessibility/
  Forms/ Charts/ Navigation/ Dark Mode/
  Glassmorphism/ Gradients/ Component Patterns/
  Color Theory/ Visual Hierarchy/ Microinteractions/
```

### Repositories prioritized in the frontend KB

| Rank | Repo |
|---|---|
| ★★★★★ | shadcn/ui |
| ★★★★★ | Magic UI |
| ★★★★★ | Aceternity UI |
| ★★★★★ | Motion |
| ★★★★★ | Refactoring UI (concepts) |
| ★★★★☆ | Radix UI |
| ★★★★☆ | Tremor |
| ★★★★☆ | React Bits |
| ★★★★☆ | Taxonomy |
| ★★★★☆ | React Aria |

### Screenshots over code

For frontend, screenshots may be even more valuable than code. A design system isn't just components — it's visual judgment. The frontend KB calls this out explicitly: curated UI examples paired with distilled principles teach patterns raw React often doesn't capture.
