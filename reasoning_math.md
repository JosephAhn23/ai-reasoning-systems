# Reasoning & Math Repos — A Study Guide

Five repositories, each teaching a *different* mathematical idea about how machines reason. Notes below are based on reading the actual repos, not just their marketing.

| # | Repo | Stars | Lang | The one idea to take away |
|---|------|------:|------|---------------------------|
| 1 | [leanprover/lean4](https://github.com/leanprover/lean4) | 8.7k | Lean | A proof is a program; type-checking *is* verification |
| 2 | [leanprover-community/mathlib4](https://github.com/leanprover-community/mathlib4) | 3.7k | Lean | How professional mathematics is actually encoded |
| 3 | [google-deepmind/alphageometry](https://github.com/google-deepmind/alphageometry) | 4.9k | Python | Neural *proposal* + symbolic *verification* beats either alone |
| 4 | [stanfordnlp/dspy](https://github.com/stanfordnlp/dspy) | 36.5k | Python | Compile reasoning pipelines instead of hand-tuning prompts |
| 5 | [MilesCranmer/PySR](https://github.com/MilesCranmer/PySR) | 3.7k | Python/Julia | Search equation-space directly for interpretable laws |

---

## 1. Lean 4 — *how to prove*

> "Lean 4 programming language and theorem prover" — Apache-2.0, ~41k commits

Lean is both a functional programming language and an interactive theorem prover. The two are the same thing: under the Curry–Howard correspondence, a proposition is a type and a proof is a term inhabiting that type. Checking a proof is checking a type.

```
Problem → Formal statement → Tactic script → Term → Kernel check → Verified
```

**What to study**
- Dependent type theory and the Curry–Howard correspondence
- Propositional and predicate logic as types
- Induction and recursion principles
- The tactic language (`simp`, `rw`, `induction`, `exact`) — the *search interface* to proof construction
- The trusted kernel: a small checker everything else must satisfy

**Where to look:** `src/kernel/` is the trust boundary — small, auditable, and the reason Lean proofs are believable. `src/Init/` holds the core logic definitions.

**Skip initially:** the compiler backend (`src/runtime/`, `LeanIR.lean`), Lake package manager internals.

**Why it matters for AI reasoning:** this is the only place in the list where "correct" is decidable. Any reasoning system that wants a ground-truth reward signal for mathematics ends up talking to something like Lean.

---

## 2. Mathlib4 — *how mathematics is represented*

> "The math library of Lean 4" — ~30 domain maintainers, the largest formal math library in existence

Lean gives you the language; Mathlib gives you the mathematics. The `Mathlib/` directory is effectively a map of the discipline:

```
Algebra/          AlgebraicGeometry/   AlgebraicTopology/   Analysis/
CategoryTheory/   Combinatorics/       Computability/       Dynamics/
FieldTheory/      Geometry/            GroupTheory/         InformationTheory/
LinearAlgebra/    Logic/               MeasureTheory/       ModelTheory/
NumberTheory/     Order/               Probability/         RepresentationTheory/
RingTheory/       SetTheory/           Topology/
```

**What to study**
- How abstraction hierarchies are built (`Monoid → Group → Ring → Field`) via typeclasses
- Naming conventions — Mathlib lemma names are *compositional and searchable*, a deliberate design for both humans and machines
- `Mathlib/Tactic/` — automation written by mathematicians for mathematicians
- `Counterexamples/` — an underrated folder: where intuition fails and why

**Skip initially:** Condensed mathematics, deep category theory, anything in `Deprecated/`.

**Why it matters:** this is the training corpus and the retrieval target for essentially every neural theorem-proving effort. Understanding its structure tells you what an LLM is being asked to predict.

---

## 3. AlphaGeometry — *how to search for proofs*

> Nature 2024. Solves 25/30 IMO geometry problems; a gold medalist averages ~25.9.

The architecture is the lesson. It is **not** `problem → LLM → answer`. It is a loop:

```
Problem
  ↓
DDAR symbolic engine  ← exhaustively derives every consequence of the premises
  ↓ (stuck?)
150M-param transformer ← proposes ONE auxiliary construction ("draw point X such that…")
  ↓
DDAR retries with the new point
  ↓
repeat, beam-searching over constructions
```

Two components, split by what each is good at:

- **DD (deductive database)** — `dd.py`. Applies geometric rules from `rules.txt` to closure. Sound but cannot invent.
- **AR (algebraic reasoning)** — `ar.py`. Gaussian elimination over linear relations between angles, ratios, distances. Catches what pure rule-matching misses.
- **DDAR** — `ddar.py`. DD and AR alternated to a fixed point. Alone it solves 14/30.
- **Language model** — `lm_inference.py`, `models.py`, `beam_search.py`. Trained purely on synthetic problems, no human proofs. Its only job is proposing auxiliary constructions, ranked by likelihood.

**What to study:** `graph.py` (the proof-state representation), `ddar.py` (the fixed-point loop), `beam_search.py` (how proposals are ranked and retried), `rules.txt` and `defs.txt` (the whole domain theory in plain text — surprisingly short).

**Skip initially:** the synthetic data generation pipeline, `numericals.py` internals.

**Why it matters:** 14/30 → 25/30 is what the neural component buys, and it buys it *without ever being trusted*. The symbolic engine verifies everything. That division of labor — LLM proposes, verifier disposes — is the single most transferable pattern here.

---

## 4. DSPy — *how to compose reasoning systems*

> "The framework for programming—not prompting—language models" — 36.5k stars, the most-used repo on this list

Not a math repo. It is about the *engineering* of multi-step reasoning. Three abstractions:

- **Signatures** (`dspy/signatures/`) — declare the input/output contract, e.g. `question -> answer`. Not a prompt; a spec.
- **Modules** (`dspy/predict/`, `dspy/primitives/`) — composable units: `Predict`, `ChainOfThought`, `ReAct`. They compose like `nn.Module` layers.
- **Optimizers / teleprompters** (`dspy/teleprompt/`) — the actual innovation. Given a metric and a handful of examples, they *search* for better prompts and few-shot demonstrations, or finetune weights. `BootstrapFewShot`, `MIPROv2`, `GEPA`.

```
Question → Retrieve → Reason → Critic → Verify → Answer
             ↑ every stage is a module; the optimizer tunes all of them jointly
```

**What to study:** `dspy/teleprompt/` — read the optimizers, that is where the ideas live. Then `dspy/predict/react.py` for how tool-use loops are expressed declaratively.

**Skip initially:** `dspy/clients/` and `dspy/adapters/` (per-provider plumbing), `dspy/experimental/`.

**Why it matters:** it reframes prompt engineering as a compilation problem with a training set and a metric. If you build reasoning systems, this is the mental model to steal even if you never import the library.

---

## 5. PySR — *how to discover mathematics from data*

> "High-Performance Symbolic Regression in Python and Julia"

Give it columns `x, y, z`. It returns `y = x² + 3` — the *equation*, not a fitted black box.

The method is evolutionary: populations of expression trees mutate, crossover, and compete. Fitness balances accuracy against complexity, so you get a Pareto front of candidate laws rather than one answer. The heavy search runs in Julia (`SymbolicRegression.jl`); `pysr/sr.py` is the scikit-learn-style Python wrapper.

**What to study**
- The accuracy/complexity Pareto tradeoff — this is Occam's razor made operational
- Custom operators and loss functions (`expression_specs.py`)
- Export paths: `export_sympy.py`, `export_torch.py`, `export_jax.py`, `export_latex.py` — discovered equations flow straight into downstream systems
- Neural-network distillation: fit a net, then symbolically regress *its* learned function to read off what it discovered

**Skip initially:** `julia_helpers.py` / `julia_import.py` plumbing, distributed-search performance tuning.

**Why it matters:** it is the only system here that *generates* mathematical statements from observation rather than proving given ones. Closest thing on the list to scientific discovery.

---

## Bonus: AIMO Progress Prize

⚠️ **Corrected link.** The commonly circulated `AI-MO/aimo-progress-prize` URL is **dead (404)**. The real repo is:

**[project-numina/aimo-progress-prize](https://github.com/project-numina/aimo-progress-prize)** (~495 stars)

Winning solution to AI Mathematical Olympiad Progress Prize 1:
- **Base model:** DeepSeekMath-Base 7B
- **Stage 1 — CoT:** fine-tuned on NuminaMath-CoT, ~860k problems with natural-language solutions (Chinese high-school exams + international olympiads)
- **Stage 2 — TIR:** Tool-Integrated Reasoning on ~70k problems where solutions *execute Python*, GPT-4-generated and filtered for correctness
- **Inference:** self-consistency decoding with code-execution feedback
- **Cost:** ~10 hours on 8× H100, then 8-bit quantized for the Kaggle submission environment

Related, worth knowing: **AIMO-2** was won with NVIDIA's [OpenMathReasoning](https://arxiv.org/abs/2504.16891) dataset, built with [NVIDIA/NeMo-Skills](https://github.com/NVIDIA/NeMo-Skills).

**Why it matters:** the only entry here that is an end-to-end, benchmarked, resource-constrained *system*. It shows what actually wins competitions today: data curation plus tool use, not clever architecture.

---

## How they fit together

```
                DISCOVER              PROVE                VERIFY
                   │                    │                     │
   PySR ───────────┘                    │                     │
   (data → equations)                   │                     │
                                        │                     │
   AlphaGeometry ───────────────────────┤                     │
   (neural proposal + symbolic search)  │                     │
                                        │                     │
   Lean 4 ──────────────────────────────┴─────────────────────┤
   (proof language, trusted kernel)                           │
                                                              │
   Mathlib4 ──────────────────────────────────────────────────┘
   (the corpus everything is stated in)

   DSPy ─── the harness that wires any of the above into a pipeline
```

- **Lean 4** — how to prove
- **Mathlib4** — how mathematics is represented formally
- **AlphaGeometry** — how to search for proofs
- **DSPy** — how to compose reasoning systems
- **PySR** — how to discover relationships from data

## Concept priority for AGI-style reasoning

| Topic | Importance | Where you'll meet it |
|-------|:----------:|----------------------|
| Logic & type theory | ★★★★★ | Lean 4 |
| Proof construction | ★★★★★ | Lean 4, Mathlib4 |
| Search algorithms | ★★★★★ | AlphaGeometry, PySR |
| Verification / soundness | ★★★★★ | Lean kernel, DDAR |
| Optimization | ★★★★☆ | DSPy, PySR |
| Combinatorics | ★★★★☆ | Mathlib4 |
| Graph theory | ★★★★☆ | AlphaGeometry (`graph.py`), Mathlib4 |
| Probability | ★★★★☆ | Mathlib4, decoding strategies |
| Linear algebra | ★★★★☆ | AlphaGeometry (`ar.py`), Mathlib4 |
| Symbolic manipulation | ★★★★☆ | PySR, DDAR |
| Geometry | ★★★☆☆ | AlphaGeometry |

## A suggested order

1. **DSPy** first — lowest barrier, immediately useful, reframes how you think about pipelines.
2. **PySR** — small, self-contained, and you get results in an afternoon.
3. **AlphaGeometry** — read `ddar.py` and `beam_search.py`. The neural/symbolic split is the highest-leverage idea in this whole list.
4. **Lean 4** — work through *Theorem Proving in Lean 4*. Slowest, deepest.
5. **Mathlib4** — not read front-to-back. Pick one area you already know and read how it was formalized.

The through-line: modern reasoning systems are **generate-and-verify loops**, not bigger next-token predictors. Every repo here is a different answer to "what generates?" and "what verifies?"
