# AI Systems Knowledge Base — Source Index

Hyperlinks to every real system distilled in [`ai_systems_knowledge_base.md`](./ai_systems_knowledge_base.md), grouped to match the file's Part structure. Updated as each batch is added.

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

### Compilers — *pending*
### Networking — *pending*
### ML Infrastructure — *pending*
### Observability — *pending*
### Performance — *pending*
### Build Systems — *pending*

---

*This index is generated alongside the knowledge base as each batch of research completes; sections marked "pending" haven't been written yet.*
