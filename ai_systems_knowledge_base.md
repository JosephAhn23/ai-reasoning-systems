# AI Systems Complete Knowledge Base

Read this file before any formal reasoning task, proof, symbolic-math problem, or autonomous coding-agent build. All guidance is here.

Ten repositories, two disciplines. **Part I** distills five systems behind modern AI reasoning: Lean 4 (proof language), Mathlib4 (formal math corpus), AlphaGeometry (proof search), DSPy (reasoning pipelines), PySR (equation discovery). **Part II** distills five systems behind modern coding agents: OpenHands/Agent Canvas (multi-backend orchestration), Aider (repo-aware editing), SWE-agent (Agent-Computer Interface), Cline (layered SDK + IDE runtime), Claude Code (permissioned tool-use harness).

The two parts converge on one idea, stated in full at the end of each: **generate freely, verify strictly.** A kernel checking a proof, a symbolic engine checking a geometric construction, a linter checking a patch, a permission tier checking a tool call — all the same asymmetry. The engineering lives in the verifier, not the generator.

---

## Table of Contents

**Part I — Reasoning & Math**
- LOGIC.md — proof as program, the kernel, tactics
- FORMALIZATION.md — typeclass hierarchies, naming conventions, `to_additive`
- PROOF_SEARCH.md — neural proposal + symbolic verification (AlphaGeometry)
- PIPELINES.md — signatures, modules, optimizers (DSPy)
- DISCOVERY.md — evolutionary equation search (PySR)
- SYNTHESIS.md (reasoning) — the generate-and-verify loop, checklist

**Part II — Coding & Agent Systems**
- ORCHESTRATION.md — multi-backend agent control planes (Agent Canvas)
- EDIT_FORMATS.md — edit formats, reflection loops, repo maps (Aider)
- ACI.md — the Agent-Computer Interface, tool bundles, action parsing (SWE-agent)
- SDK_LAYERING.md — stateless agent loops, plan/act, checkpoints (Cline)
- TOOL_SURFACE.md — permission tiers, hooks, subagents, typed memory (Claude Code)
- SYNTHESIS.md (agents) — the generate-and-check loop, checklist

---

# PART I — REASONING & MATH

---


---

# LOGIC.md

The foundation: what a proof *is* to a machine.

## The Core Identity

A proposition is a type. A proof is a value of that type. Checking a proof is type-checking.

```
Proposition          Type
Proof of P           A term of type P
P implies Q          P → Q  (a function)
P and Q              P × Q  (a pair)
P or Q               P ⊕ Q  (a tagged union)
There exists x, P x  A pair (witness, proof)
For all x, P x       A dependent function
False                The empty type (no constructors)
True                 The unit type (one constructor)
```

This is the Curry–Howard correspondence. It is why a programming language can be a theorem prover: they are the same activity.

## The Logical Primitives

Every proposition is built from these. Note how small the list is.

```
Eq : α → α → Prop
  | refl (a : α) : Eq a a

True : Prop
  | intro : True

False : Prop
  (no constructors — this is the entire definition)

And (a b : Prop) : Prop
  | intro (left : a) (right : b)

Or (a b : Prop) : Prop
  | inl (h : a) : Or a b
  | inr (h : b) : Or a b

Nat
  | zero : Nat
  | succ (n : Nat) : Nat
```

Read these carefully — they contain the whole of propositional logic.

- `False` has **no constructors**. So you can never build one. So from `False` anything follows (there are zero cases to handle). That is the principle of explosion, and it is a consequence of the definition, not an axiom.
- `Eq` has **one** constructor, `refl : Eq a a`. The only way to prove two things equal is that they are literally the same thing. Everything else about equality (symmetry, transitivity, congruence) is derived from pattern-matching on `refl`.
- `And` is a struct with two fields. Proving `A ∧ B` is providing both. Using it is projection.
- `Or` has two constructors. Proving it is picking a side. Using it is a case split — you must handle both.
- `Nat` is zero-or-successor. This is why induction works: it is just recursion over the two constructors.

## Universes

```
Prop      = Sort 0     propositions (proof-irrelevant)
Type      = Sort 1     ordinary data
Type u    = Sort (u+1) larger data
```

`Prop` is special: proofs are *irrelevant*. Any two proofs of the same proposition are treated as equal, and proofs are erased at runtime. This is what makes it safe to have a proof carry no computational content.

## Induction Is Recursion

```
theorem add_zero (n : Nat) : n + 0 = n :=
  Nat.rec
    (motive := fun n => n + 0 = n)
    rfl                                    -- base: 0 + 0 = 0
    (fun n ih => congrArg Nat.succ ih)     -- step: assume n, prove n+1
    n
```

Every inductive type auto-generates a recursor. Mathematical induction is the recursor for `Nat`. Structural induction on lists, trees, or anything else is the same mechanism. There is no separate "induction axiom."

## The Kernel

The trust story matters more than the feature list.

```
Tactics / elaborator / macros / typeclass inference     ← large, complex, UNTRUSTED
                    ↓ produces
              a raw proof term
                    ↓ checked by
                  KERNEL                                 ← small, TRUSTED
```

The kernel does one thing: check that a term has the type it claims. It is a few thousand lines. Everything else — every tactic, every automation, every clever search — is untrusted, because its output must pass the kernel anyway.

**This is the single most important architectural idea in formal reasoning.** A buggy tactic cannot produce a false theorem. It can only fail to produce a proof. Sophistication is quarantined away from soundness.

An AI proof generator plugs in at exactly this seam: it can be as unreliable as you like, because the kernel is the judge.

## Tactic Vocabulary

Tactics are the *search interface*. They build proof terms so you do not have to write them by hand.

**Closing a goal**
```
rfl                proves x = x by computation
exact e            supply the proof term directly
assumption         find a matching hypothesis
trivial            try the obvious closers
decide             decide the goal by computation (needs decidability)
contradiction      find contradictory hypotheses
omega              linear arithmetic over Nat/Int
simp               simplify with the rewrite-rule database
```

**Structuring a proof**
```
intro x            introduce a hypothesis or bound variable
apply f            reduce goal to f's arguments (backward reasoning)
constructor        apply the type's constructor
left / right       choose a disjunct of Or
exists w           supply a witness
cases h            case-split on an inductive hypothesis
induction n        induct, generating base and step goals
have h : T := e    forward step — introduce an intermediate fact
rw [h]             rewrite using an equation
subst              eliminate a variable using an equation
symm               flip an equality goal
exfalso            replace the goal with False
by_contra h        classical proof by contradiction
```

**The two directions:**
- `apply` reasons **backward** — from the goal toward the hypotheses.
- `have` reasons **forward** — from the hypotheses toward the goal.

Real proofs interleave both. Backward for structure, forward for the actual insight.

## Worked Example

```
theorem or_comm (h : A ∨ B) : B ∨ A := by
  cases h with
  | inl ha => exact Or.inr ha
  | inr hb => exact Or.inl hb
```

`Or` has two constructors, so `cases` gives two goals. In each, flip the injection. The proof is fully mechanical once you see the shape of the type. **The type tells you the proof structure.** That is the recurring experience of working in a proof assistant.

## What to Carry Away

1. Proof = program. Verification = type-checking.
2. `False` is empty; `Eq` has one constructor. Almost everything else is derived.
3. Induction is recursion over constructors, not a separate axiom.
4. A small trusted kernel checks output from arbitrarily untrusted producers. **Generate freely, verify strictly.**

---

# FORMALIZATION.md

How an entire discipline gets encoded so both humans and machines can search it.

## The Territory

Formal mathematics organizes into these areas. This is effectively a map of the field:

```
Algebra              AlgebraicGeometry    AlgebraicTopology    Analysis
CategoryTheory       Combinatorics        Computability        Condensed
Dynamics             FieldTheory          Geometry             GroupTheory
InformationTheory    LinearAlgebra        Logic                MeasureTheory
ModelTheory          NumberTheory         Order                Probability
RepresentationTheory RingTheory           SetTheory            Topology
```

## The Central Technique: Typeclass Hierarchies

Mathematical structures are encoded as typeclasses that extend one another. A theorem proved for `Monoid` applies automatically to every group, ring, and field, because each *is* a monoid by inheritance.

**The multiplicative tower, as actually defined:**

```
Mul                                          has  a * b
  └─ Semigroup           extends Mul          +  associativity
       └─ CommSemigroup  extends Semigroup    +  commutativity
       └─ LeftCancelSemigroup                 +  a*b = a*c → b = c

One, Mul
  └─ MulOneClass         extends MulOne       +  1*a = a = a*1
       └─ Monoid         extends Semigroup, MulOneClass, NPow
            └─ CommMonoid          extends Monoid, CommSemigroup
            └─ LeftCancelMonoid    extends Monoid, LeftCancelSemigroup
            └─ DivInvMonoid                   +  a⁻¹, a/b
                 └─ Group          extends DivInvMonoid   +  a⁻¹*a = 1
                      └─ CommGroup extends Group, CommMonoid
```

**The ring tower:**

```
Distrib                       extends Mul, Add          +  distributivity
NonUnitalNonAssocSemiring     extends AddCommMonoid, Distrib
  └─ NonUnitalSemiring                                  +  * associative
  └─ NonAssocSemiring                                   +  has 1
       └─ Semiring           extends AddCommMonoid, ...
            └─ CommSemiring  extends Semiring, CommMonoid
            └─ Ring          extends Semiring, AddCommGroup, AddGroupWithOne
                 └─ CommRing extends Ring
                      └─ IsDomain  extends IsCancelMulZero
```

**Why the hierarchy is so finely sliced.** Every intermediate class (`NonUnitalNonAssocSemiring` and friends) exists because *some* real theorem needs exactly those axioms and no more. Slicing thinly maximizes reuse. This is the formal-math analogue of programming to the narrowest interface.

## Naming Is a Search Index

Lemma names are built compositionally from the symbols in the statement, read left to right. Once you know the scheme you can *guess* a lemma's name and be right.

```
mul_comm              a * b = b * a
mul_one               a * 1 = a
one_mul               1 * a = a
mul_left_comm         a * (b * c) = b * (a * c)
mul_rotate            a * b * c = b * c * a
inv_inv               a⁻¹⁻¹ = a
inv_injective         a⁻¹ = b⁻¹ → a = b
mul_eq_one_iff_eq_inv a * b = 1 ↔ a = b⁻¹
add_le_add_left       b ≤ c → a + b ≤ a + c
```

**The vocabulary:**

| Fragment | Means |
|---|---|
| `mul` `add` `sub` `div` `inv` `neg` | the operation |
| `zero` `one` | the identity element |
| `comm` | commutativity |
| `assoc` | associativity |
| `left` `right` | which side the operation acts on |
| `_iff_` | the statement is an ↔ |
| `_of_` | reads right-to-left: conclusion `_of_` hypothesis |
| `_eq_` | an equation appears |
| `injective` `surjective` `bijective` | function properties |
| `'` (prime) | a variant of the previous lemma |

`_of_` is the one that trips people up. `foo_of_bar` means "**foo**, given **bar**" — conclusion first, hypothesis second.

**Why this matters for AI:** a compositional naming scheme makes the corpus *retrievable*. A model predicting `mul_comm` from a goal containing `a * b = b * a` is doing something closer to structured lookup than open generation. Naming discipline is what makes the library machine-searchable at scale.

## The Duplication Problem and to_additive

Every multiplicative structure has an additive twin. `Monoid`/`AddMonoid`, `Group`/`AddGroup`, `mul_comm`/`add_comm`. Writing both by hand doubles the library and every future edit.

The fix is a metaprogram: write the multiplicative version, mark it, and the additive version is generated automatically — statement, proof, and name all translated.

```
@[to_additive]
theorem mul_comm (a b : α) : a * b = b * a := ...

-- automatically generates:
theorem add_comm (a b : α) : a + b = b + a := ...
```

In the core group-theory definitions alone this attribute appears ~160 times. **The general lesson: when a formal corpus has a systematic symmetry, encode the symmetry as a program rather than paying for it in duplicated text.**

## Reading Formalized Mathematics

1. **Find the definition first.** The formal definition is the ground truth and often differs from the textbook phrasing in instructive ways.
2. **Check what typeclass a theorem requires.** The hypotheses tell you exactly how general it is.
3. **Counterexamples are as valuable as theorems.** A well-maintained corpus keeps a collection of them — places where the obvious generalization fails. This is where intuition gets corrected.
4. **Don't read linearly.** Pick a topic you already understand and read how it was formalized. The gap between your mental model and the formal one is the lesson.

## What to Carry Away

1. Structures form an inheritance hierarchy; theorems attach at the weakest level that supports them.
2. Names are compositional and therefore searchable — by humans and by models.
3. Systematic symmetry (multiplicative/additive) is handled by metaprogramming, not duplication.
4. This corpus is the training data and retrieval target for neural theorem proving. Its structure determines what a model can learn.

---

# PROOF_SEARCH.md

How to search for a proof when the space is infinite. The central case study in neural–symbolic architecture.

## The Architecture

Not `problem → model → answer`. A loop with a strict division of labor:

```
Problem (premises + goal)
   ↓
┌──────────────────────────────────────────────┐
│  SYMBOLIC ENGINE                             │
│  Derive every consequence of what is known.  │
│  Sound. Complete within its rules.           │
│  Cannot invent new objects.                  │
└──────────────────────────────────────────────┘
   ↓ goal proved? → DONE
   ↓ saturated, goal not proved?
┌──────────────────────────────────────────────┐
│  LANGUAGE MODEL                              │
│  Propose ONE new auxiliary object.           │
│  "Construct point X such that ..."           │
│  Unsound. Untrusted. Ranked by likelihood.   │
└──────────────────────────────────────────────┘
   ↓ add the proposed object to the premises
   └──────────── loop ────────────────────────┘
```

**The result that justifies the design:** on a 30-problem olympiad benchmark, the symbolic engine alone solves 14. With the language model proposing constructions, 25. A human gold medalist averages about 25.9.

The model contributes 11 problems. It is never trusted for a single deductive step. It only suggests *what to try*; the symbolic engine decides what is true.

## The Two Halves of Deduction

Symbolic reasoning here splits into two complementary mechanisms, alternated to a fixed point.

**Half 1 — Rule matching (DD).** Pattern-match a fixed rule set against known facts, add conclusions, repeat until nothing new appears. Sound, exhaustive, and blind to anything not expressible as a rule.

**Half 2 — Algebraic reasoning (AR).** Some facts are relations between *quantities*, not structural patterns. Encode them as linear equations and run Gaussian elimination. Three separate coefficient tables:

| Table | Variable | Captures |
|---|---|---|
| Angle table | direction / slope of each line | angle sums, parallel and perpendicular relations |
| Ratio table | log of each length | products and ratios of lengths become sums |
| Distance table | position of a point along a line | betweenness, midpoints, collinear spacing |

Taking **logarithms of lengths** is the trick that makes ratio reasoning linear: `AB/CD = EF/GH` becomes `log AB − log CD − log EF + log GH = 0`, which Gaussian elimination handles directly.

Rule matching finds structure. Algebra finds numerical consequence. Neither subsumes the other, so you alternate:

```
loop:
    run rule matching to saturation
    if goal proved: return SOLVED
    if nothing new was added:
        run algebraic derivations
        if algebra added nothing: return SATURATED
    repeat
```

## The Complete Rule Set

The entire deductive theory is 43 rules. This is the whole domain knowledge — small enough to read in one sitting, which is itself the point.

```
perp A B C D, perp C D E F, ncoll A B E => para A B E F
cong O A O B, cong O B O C, cong O C O D => cyclic A B C D
eqangle A B P Q C D P Q => para A B C D
cyclic A B P Q => eqangle P A P B Q A Q B
eqangle6 P A P B Q A Q B, ncoll P Q A B => cyclic A B P Q
cyclic A B C P Q R, eqangle C A C B R P R Q => cong A B P Q
midp E A B, midp F A C => para E F B C
para A B C D, coll O A C, coll O B D => eqratio3 A B C D O O
perp A B C D, perp E F G H, npara A B E F => eqangle A B E F C D G H
eqangle a b c d m n p q, eqangle c d e f p q r u => eqangle a b e f m n r u
eqratio a b c d m n p q, eqratio c d e f p q r u => eqratio a b e f m n r u
eqratio6 d b d c a b a c, coll d b c, ncoll a b c => eqangle6 a b a d a d a c
eqangle6 a b a d a d a c, coll d b c, ncoll a b c => eqratio6 d b d c a b a c
cong O A O B, ncoll O A B => eqangle O A A B A B O B
eqangle6 A O A B B A B O, ncoll O A B => cong O A O B
circle O A B C, perp O A A X => eqangle A X A B C A C B
circle O A B C, eqangle A X A B C A C B => perp O A A X
circle O A B C, midp M B C => eqangle A B A C O B O M
circle O A B C, coll M B C, eqangle A B A C O B O M => midp M B C
perp A B B C, midp M A C => cong A M B M
circle O A B C, coll O A C => perp A B B C
cyclic A B C D, para A B C D => eqangle A D C D C D C B
midp M A B, perp O M A B => cong O A O B
cong A P B P, cong A Q B Q => perp A B P Q
cong A P B P, cong A Q B Q, cyclic A B P Q => perp P A A Q
midp M A B, midp M C D => para A C B D
midp M A B, para A C B D, para A D B C => midp M C D
eqratio O A A C O B B D, coll O A C, coll O B D, ncoll A B C,
    sameside A O C B O D => para A B C D
para A B A C => coll A B C
midp M A B, midp N C D => eqratio M A A B N C C D
eqangle A B P Q C D U V, perp P Q U V => perp A B C D
eqratio A B P Q C D U V, cong P Q U V => cong A B C D
cong A B P Q, cong B C Q R, cong C A R P, ncoll A B C => contri* A B C P Q R
cong A B P Q, cong B C Q R, eqangle6 B A B C Q P Q R, ncoll A B C
    => contri* A B C P Q R
eqangle6 B A B C Q P Q R, eqangle6 C A C B R P R Q, ncoll A B C
    => simtri A B C P Q R
eqangle6 B A B C Q R Q P, eqangle6 C A C B R Q R P, ncoll A B C
    => simtri2 A B C P Q R
eqangle6 B A B C Q P Q R, eqangle6 C A C B R P R Q, ncoll A B C, cong A B P Q
    => contri A B C P Q R
eqangle6 B A B C Q R Q P, eqangle6 C A C B R Q R P, ncoll A B C, cong A B P Q
    => contri2 A B C P Q R
eqratio6 B A B C Q P Q R, eqratio6 C A C B R P R Q, ncoll A B C
    => simtri* A B C P Q R
eqratio6 B A B C Q P Q R, eqangle6 B A B C Q P Q R, ncoll A B C
    => simtri* A B C P Q R
eqratio6 B A B C Q P Q R, eqratio6 C A C B R P R Q, ncoll A B C, cong A B P Q
    => contri* A B C P Q R
para a b c d, coll m a d, coll n b c, eqratio6 m a m d n b n c,
    sameside m a d n b c => para m n a b
para a b c d, coll m a d, coll n b c, para m n a b => eqratio6 m a m d n b n c
```

**Predicate vocabulary:**

| Predicate | Meaning |
|---|---|
| `coll A B C` | A, B, C are collinear |
| `ncoll A B C` | not collinear (a non-degeneracy guard) |
| `para A B C D` | line AB ∥ line CD |
| `npara` | not parallel |
| `perp A B C D` | line AB ⊥ line CD |
| `cong A B C D` | segment AB ≅ segment CD |
| `cyclic A B C D` | the four points lie on one circle |
| `circle O A B C` | O is the center of the circle through A, B, C |
| `midp M A B` | M is the midpoint of AB |
| `eqangle A B C D E F G H` | angle(AB,CD) = angle(EF,GH) |
| `eqratio A B C D E F G H` | AB/CD = EF/GH |
| `simtri A B C P Q R` | triangles similar |
| `contri A B C P Q R` | triangles congruent |
| `sameside A B C D E F` | points on the same side |

The `6` suffix (`eqangle6`, `eqratio6`) marks the full-permutation variant. The `*` suffix (`simtri*`, `contri*`) marks orientation-agnostic versions.

## Where the Neural Model Actually Sits

The construction vocabulary — the set of new objects that can be introduced — is around 300 named constructions: `midpoint`, `foot`, `circumcenter`, `incenter`, `orthocenter`, `angle_bisector`, `intersection_ll`, `intersection_lc`, `reflect`, `on_circle`, `on_bline`, `tangent`, and so on.

The language model's entire job is choosing which one, applied to which points. It is a ~150M parameter transformer — small — trained **only on synthetically generated problems, with no human proofs**. It emits ranked candidates; beam search tries them highest-confidence first; the symbolic engine passes judgment on each.

**The auxiliary construction is the hard part of olympiad geometry.** "Draw the midpoint of BC and connect it to the circumcenter" is the creative leap; everything after it is mechanical. The system's design isolates exactly that leap and gives it to the neural component, keeping every other step verifiable.

## Numeric Filtering

Before a candidate fact is admitted, it is checked against randomly sampled numerical coordinates satisfying the premises. If a claimed equality fails numerically it is discarded immediately — no symbolic work wasted.

**This is a cheap, powerful pattern: use fast numerical falsification to prune before expensive symbolic verification.** It generalizes far beyond geometry.

## Proof Extraction

Facts are stored in a dependency DAG: each derived fact records which rule produced it and which facts it consumed. When the goal is reached, walk the DAG backward from the goal to the premises. That traversal *is* the proof.

A useful side effect: the traversal separates steps that used auxiliary constructions from steps that did not, showing precisely which creative additions the proof actually required. Constructions the model proposed but that ended up unused are pruned.

## What to Carry Away

1. **Split proposal from verification.** Let the unreliable component propose; let the sound component verify. Neither needs to do the other's job.
2. **The untrusted component can be small.** 150M parameters, trained on synthetic data, adds 11/30 problems.
3. **Alternate complementary reasoning modes.** Pattern matching and algebra catch different things; run both to a fixed point.
4. **Falsify cheaply before verifying expensively.** Numerical checks prune the search before symbolic work begins.
5. **The full domain theory was 43 rules.** Before scaling a model, ask whether the domain has a small complete rule set you are failing to exploit.

---

# PIPELINES.md

How to build multi-step reasoning systems that improve automatically instead of by hand-tuning prompts.

## The Reframe

Prompt engineering is manual hill-climbing on a loss function you cannot see. The alternative is to treat a reasoning pipeline the way you treat a neural network: declare the structure, define a metric, and let an optimizer search the parameters.

```
Hand-written prompts          Compiled pipeline
─────────────────────         ──────────────────────────
you write the prompt          you declare inputs → outputs
you tweak wording             an optimizer searches wording
you pick examples             an optimizer searches examples
quality is unmeasured         quality is a metric on a dev set
changing models = redo        changing models = recompile
```

The prompt becomes a *compiled artifact*, not source code.

## Layer 1: Signatures

A signature declares the contract. It is not a prompt.

```
"question -> answer"
"context, question -> answer"
"document -> summary, key_points"
"question -> reasoning, answer"
```

Each field has a name, a type, and an optional description. The name carries semantics — `question` and `answer` mean something to the model, so field naming is part of the design. The framework generates the actual prompt text from this declaration, and the optimizer is free to rewrite that text later.

The critical property: **the signature is stable, the prompt is disposable.** You commit to what goes in and what comes out; everything else is subject to optimization.

## Layer 2: Modules

Modules compose like neural-network layers. Each wraps a signature with a strategy.

| Module | Strategy |
|---|---|
| `Predict` | direct call — the base case |
| `ChainOfThought` | prepend a `reasoning` output field before the answer |
| `ProgramOfThought` | emit code, execute it, use the result |
| `ReAct` | interleave thought / tool call / observation until done |
| `BestOfN` | sample N times, select by a scoring function |
| `Refine` | generate, critique, revise |
| `MultiChainComparison` | generate several reasoning chains, compare them |
| `Parallel` | run branches concurrently |

**ChainOfThought is almost trivially simple.** It takes your signature and inserts one extra output field named `reasoning` ahead of the real outputs. That is the entire implementation. The model fills in `reasoning` before `answer`, and because generation is left-to-right, the answer is conditioned on the reasoning it just wrote.

The lesson: chain-of-thought is not a prompt phrasing. It is a *structural* change to the output schema. Structure beats wording.

**ReAct** builds an extended signature with three added output fields — `next_thought`, `next_tool_name`, `next_tool_args` — plus a `trajectory` input that accumulates observations. It appends a synthetic `finish` tool so "I'm done" is just another tool call. The loop runs until `finish` or an iteration cap. Tool choice is typed as an enum over the available tool names, so the model cannot hallucinate a tool that does not exist.

**Constrain at the type level rather than by instruction.** "Only use these tools" in prose is a suggestion; an enum-typed output field is enforcement.

## Layer 3: Optimizers

This is where the real leverage is. Given a program, a metric, and training examples, an optimizer searches for better prompts, better few-shot demonstrations, or better weights.

### Bootstrap Few-Shot

The foundational move: generate your own demonstrations.

```
for each training example:
    run the current program
    if the metric says the output is correct:
        keep the full execution trace as a demonstration
        (including intermediate steps of every module)
```

You get few-shot examples for *every* module in the pipeline, including inner ones you have no labels for, because a correct final answer retroactively validates the whole trace.

Typical settings: at most 4 bootstrapped demos and 16 labeled demos per module. Additional rounds re-run at temperature 1.0 with a fresh rollout id to bypass caching and collect more diverse traces.

**The insight: a correct end-to-end result is a free label for every intermediate step.**

### Instruction + Demo Search (MIPRO)

Three phases:

```
Phase 1  Bootstrap candidate demonstration sets (as above)
Phase 2  Propose candidate instruction strings, using a model that sees
         the program's code, a data summary, and the bootstrapped traces
Phase 3  Bayesian optimization over the joint space of
         (instruction choice, demo-set choice) for every module
```

Phase 3 matters most. With several modules each having several instruction and demo candidates, the combinatorics are far too large to grid-search — so it uses a Bayesian optimizer over the discrete choice space, evaluating on minibatches (default 35 examples) and periodically running a full evaluation on the best candidate so far.

Preset budgets:

| Preset | Trials | Validation size |
|---|---|---|
| light | 6 | 100 |
| medium | 12 | 300 |
| heavy | 18 | 1000 |

**Minibatch evaluation is what makes this affordable.** Full evaluation of every candidate would dominate the cost; evaluate cheaply, confirm expensively, and only for candidates that look promising.

### Introspective Mini-Batch Ascent (SIMBA)

Uses the model to analyze its own failures.

```
sample a minibatch
run the program at several temperatures
find examples with HIGH OUTPUT VARIANCE across runs
    → these are where the program is uncertain, i.e. where headroom is
for those examples, either:
    (a) ask the model to write an explicit rule explaining
        what distinguishes success from failure, and append it, or
    (b) append a successful trace as a new demonstration
keep whichever candidate scores best
```

Defaults: batch size 32, 6 candidates per step, 8 steps, at most 4 demos.

**High variance across samples is a signal for where to spend optimization effort.** Examples the program always gets right or always gets wrong teach you little; the unstable middle is where improvement lives.

### Reflective Evolution (GEPA)

Treats prompts as a population under evolutionary pressure, with a crucial addition: the metric returns **text feedback, not just a score**.

A standard metric returns `0.7`. A feedback metric returns `0.7` plus "the answer omitted the units, and cited the wrong source for the second claim." That text becomes the mutation signal — the model rewrites the instruction in response to a specific diagnosis rather than to a scalar.

The optimizer tracks candidate lineage (which candidate was mutated from which), per-instance scores rather than only aggregates, and which candidate is best on each individual validation example. That last one lets it preserve candidates that are excellent on a narrow slice even if their average is mediocre — protecting diversity the way a Pareto front does.

**A scalar reward tells you *that* you failed. Text feedback tells you *why*. The second is a much richer gradient.**

## Choosing an Optimizer

| Situation | Use |
|---|---|
| Few labels, need a quick baseline | Bootstrap Few-Shot |
| Have a dev set and a real budget | Instruction + demo search (MIPRO) |
| Metric can explain failures in words | Reflective evolution (GEPA) |
| Want the model to diagnose itself | Introspective ascent (SIMBA) |
| Have many labels and can train weights | Bootstrap-then-finetune |

## Building a Pipeline

1. **Write the metric first.** Everything downstream optimizes against it. A vague metric produces a vaguely optimized program. This is the step people skip and the step that decides the outcome.
2. **Start with `Predict`.** Get a baseline number before adding structure.
3. **Add structure only where the baseline fails.** Reasoning field, tool use, retrieval — each because a specific failure demanded it.
4. **Compile.** Let the optimizer handle the wording.
5. **Recompile when anything changes.** New model, new data distribution — recompile rather than hand-patch.

## What to Carry Away

1. Declare the contract (signature); let the wording be compiled.
2. Chain-of-thought is a schema change, not a phrasing trick.
3. A correct final answer labels every intermediate step for free.
4. Evaluate on minibatches, confirm on the full set.
5. Text feedback is a far stronger optimization signal than a scalar.
6. Constrain with types, not with polite instructions.

---

# DISCOVERY.md

How to find mathematical structure in data instead of proving structure you already have. The inverse problem to everything above.

## The Problem

Regression fits parameters to a *fixed* functional form. Symbolic regression searches over functional forms themselves.

```
Given columns  x, y, z
Discover       y = x² + 3

Not "here are 400 fitted weights."
An equation you can read, check dimensionally, and reason about.
```

Search space: all expression trees over a chosen operator set. Infinite, discrete, non-differentiable. Gradient descent is unavailable. So: evolution.

## The Algorithm

```
maintain P independent populations of expression trees
repeat:
    within each population, for many cycles:
        select parents by tournament
        mutate or cross over
        optimize any numeric constants
        replace the weakest members
    migrate top individuals between populations
    migrate from the hall of fame back into populations
return the Pareto front of (accuracy, complexity)
```

Typical scale: 31 populations of 27 individuals, 380 mutation cycles per iteration, 100 iterations. Populations are semi-isolated so they explore different regions; migration spreads discoveries without collapsing everything onto one lineage too early.

**Island-model evolution — many semi-isolated populations with occasional migration — preserves diversity better than one large panmictic population.** Premature convergence is the main failure mode in expression search.

## Tournament Selection

To pick a parent: sample 15 random individuals, sort by fitness, and choose the best with probability 0.982, the next with 0.982 × 0.018, and so on down the ranked list.

Not "always take the best" (collapses diversity), not uniform random (no selection pressure). A tunable knob between them.

## Mutation Operators and Their Tuned Weights

These relative likelihoods were themselves found by hyperparameter search, which is why they are oddly specific. The *ratios* encode real information about what makes expression search work.

| Operator | Weight | Effect |
|---|---|---|
| rotate tree | 4.26 | restructure at a random node, preserving semantics locally |
| add node | 2.47 | grow the expression |
| delete node | 0.870 | shrink the expression |
| mutate operator | 0.293 | swap `+` for `*`, `sin` for `cos` |
| do nothing | 0.273 | pass through unchanged |
| swap operands | 0.198 | reorder arguments of a binary operator |
| mutate feature | 0.100 | point a variable at a different input column |
| mutate constant | 0.035 | perturb a numeric literal |
| insert node | 0.011 | wrap a subtree in a new operator |
| simplify | 0.002 | algebraically reduce constant subexpressions |
| randomize | 0.0005 | discard and regenerate from scratch |

Crossover probability: 0.0259.

**Read the ratios.** Tree rotation is weighted ~8,500× more heavily than full randomization and ~120× more than constant mutation. Structural rearrangement of what already works dominates; wholesale replacement is nearly never useful. The search is overwhelmingly *local and structural*.

Note also that constant mutation is weighted low — because constants are handled better by a dedicated numerical optimizer (BFGS, 8 iterations, 2 restarts, applied with probability 0.14 per iteration) than by blind evolutionary perturbation.

**Use the right tool per parameter type: discrete structure by evolution, continuous constants by gradient-based optimization.** Hybridizing beats either alone.

## Complexity and the Pareto Front

Every expression has a complexity score — roughly a node count, with per-operator costs configurable (you can declare `exp` more expensive than `+`).

The output is not one equation. It is the **Pareto front**: for each complexity level, the most accurate expression achieving it.

```
complexity   loss      expression
    1        0.412     2.31
    3        0.183     x + 1.04
    5        0.021     x*x + 2.98
    7        0.0207    x*x + 2.98 + 0.001*y      ← more complex, barely better
```

Read the front and pick the knee — the point past which added complexity stops buying accuracy. Here that is complexity 5. The complexity-7 equation is fitting noise.

**This is Occam's razor made operational.** You do not choose a complexity penalty in advance and hope; you see the whole tradeoff curve and choose after looking.

There is also adaptive parsimony: the search tracks how many expressions exist at each complexity level and penalizes crowded levels, pushing exploration toward under-explored sizes rather than letting everything pile up at the cheapest one.

## Operators

Standard vocabulary: `+ - * /`, `sqrt`, `exp`, `log`, `sin`, `cos`, `tan`, `abs`, `sign`, `pow`, plus arbitrary user-defined operators.

**Operator choice is the single highest-leverage decision.** Including `exp` and `log` when the true relation is polynomial wastes the entire budget searching a space that cannot help. Include what the domain plausibly contains and nothing more. If the data is physical, dimensional-analysis constraints prune enormous swaths of the space for free.

## Use Cases

1. **Scientific law discovery** — recover a governing equation from observations.
2. **Neural network distillation** — train a network, then symbolically regress *its learned function* to read off what it discovered. This turns an opaque model into a readable hypothesis.
3. **Feature engineering** — discover nonlinear combinations that matter, then feed them to a conventional model.
4. **Model compression** — replace a network with a closed-form expression that evaluates in nanoseconds.

Works best in low dimensions (roughly under 10 features). The search space grows explosively with feature count; select features first.

## What to Carry Away

1. Evolution handles discrete structure where gradients cannot go.
2. Split parameter types: structure by evolution, constants by numerical optimization.
3. Island populations plus migration beats one big population.
4. Return a Pareto front, not a single answer — let the human pick the knee.
5. Constrain the search space with domain knowledge (operators, dimensions) before scaling compute.

---

# SYNTHESIS.md (Reasoning)

What the five systems share, and how to combine them.

## The Common Shape

Every one of these is a **generate-and-verify loop**. They differ only in what plays each role.

| System | Generator | Verifier |
|---|---|---|
| Lean 4 | tactics, elaboration, automation | the trusted kernel |
| Mathlib4 | human formalizers | the kernel, plus CI on the whole corpus |
| AlphaGeometry | 150M-parameter transformer | symbolic deduction engine |
| DSPy | the language model | the metric on a dev set |
| PySR | evolutionary mutation | loss on held-out data |

**The generator is allowed to be unreliable. The verifier is not allowed to be wrong.** Every system here gets its strength from that asymmetry, and every one of them keeps the verifier small and simple enough to trust.

This is the structural answer to LLM hallucination. You do not make the generator perfectly reliable. You make its output checkable, and you check it.

## Building a Reasoning System — Checklist

**1. What is the verifier?**
Answer this first. A kernel, a symbolic engine, a test suite, a metric, held-out loss. If nothing can check the output, you do not have a reasoning system — you have a text generator.

**2. Is the verifier small enough to trust?**
Large verifiers have bugs, and a buggy verifier is worse than none because it manufactures false confidence. Keep the trusted base small and push complexity into the untrusted generator.

**3. Can you falsify cheaply before verifying expensively?**
Numerical spot-checks, type checks, dimensional analysis, quick heuristics. Prune before you pay.

**4. What exactly is the generator for?**
The sharpest designs isolate the one genuinely creative step. In geometry that is the auxiliary construction — everything after it is mechanical. Find the analogous step in your domain and give the model only that.

**5. Do you have complementary reasoning modes?**
Pattern matching and algebra catch different things. Run both to a fixed point rather than committing to one.

**6. Is your search space constrained by domain knowledge?**
43 rules covered olympiad geometry. Operator choice decides equation search. Typeclass hierarchies decide theorem applicability. Constraining the space beats scaling the search.

**7. Do you get text feedback or only a scalar?**
"Wrong, 0.3" is a weak signal. "Wrong because the units do not match" is a strong one. Design metrics that explain.

**8. Are you returning one answer or a front?**
When there is a real tradeoff (accuracy vs. complexity, speed vs. rigor), return the curve and let the human choose the knee.

## Concept Priority

| Topic | Weight | Why |
|---|---|---|
| Logic and type theory | ★★★★★ | the only rigorous definition of "correct" |
| Proof construction | ★★★★★ | verification is the bottleneck, not generation |
| Search algorithms | ★★★★★ | every system here is a search |
| Verification and soundness | ★★★★★ | the asymmetry the whole design rests on |
| Optimization | ★★★★☆ | how pipelines and equations improve |
| Combinatorics | ★★★★☆ | search-space sizing |
| Graph and DAG structure | ★★★★☆ | proof states, dependency tracking, traceback |
| Probability | ★★★★☆ | ranking proposals, decoding strategies |
| Linear algebra | ★★★★☆ | algebraic reasoning is Gaussian elimination |
| Symbolic manipulation | ★★★★☆ | rewriting, simplification, expression trees |
| Geometry | ★★★☆☆ | a case study, not a foundation |

## Reading Order

1. **Pipelines** — lowest barrier, immediately useful, reframes how you think about multi-step systems.
2. **Discovery** — small and self-contained; you get results the same day.
3. **Proof search** — the neural/symbolic split is the highest-leverage idea in this document.
4. **Logic** — slowest and deepest. Work an actual tutorial; reading about it does not transfer.
5. **Formalization** — never front to back. Pick one topic you already know and read how it was encoded.

## The One-Sentence Version

Modern reasoning systems are generate-and-verify loops, not larger next-token predictors — and the engineering is almost entirely in the verifier.


---

# PART II — CODING & AGENT SYSTEMS

---

---

# ORCHESTRATION.md

How to run many agents, of different kinds, from one control plane. Case study: OpenHands / Agent Canvas.

## The Reframe

The original design was one agent, one loop, one sandbox. The current design assumes the opposite: an operator wants to run *several different agents* — possibly from different vendors — against different backends, and switch between them without losing context.

```
                    ┌─────────────────────┐
                    │   Frontend (web UI)  │   http://localhost:8000
                    └──────────┬───────────┘
                               │
                    ┌──────────┴───────────┐
                    │     Agent Server      │   REST API
                    │  runs N agents on     │
                    │  one machine          │
                    └──────────┬───────────┘
                               │  Agent-Client Protocol (ACP)
              ┌────────────────┼────────────────┬───────────────┐
              │                │                │               │
        OpenHands agent   Claude Code       Codex agent     Gemini agent
```

A separate **Automation Server** sits alongside this for scheduled or event-triggered runs — the difference between "an agent I am watching" and "an agent that runs on a cron and reports back."

## The Protocol Is the Product

The interesting design decision is not the agent implementation — it is the **Agent-Client Protocol (ACP)**, a standard that lets a single control surface drive fundamentally different agents interchangeably. Whatever agent backend you plug in, the control plane only needs to speak ACP; it does not need to know the agent's internals.

**This is the same idea as a device driver interface.** The moment you have more than one agent implementation worth running, the leverage move is standardizing the boundary between "thing that plans and acts" and "thing that displays and schedules," not improving any single agent.

## Deployment Is a Spectrum, Not a Choice

The runtime explicitly supports, as different deployment targets for the *same* control plane:

```
local machine  →  Docker container (filesystem sandboxed)  →  remote VM  →  managed cloud
```

The user is meant to move an agent across this spectrum without re-learning a different tool. Sandboxing is a deployment-target property, not a feature of any one agent.

## What to Carry Away

1. Once you have more than one agent worth running, build the protocol boundary before you build the second agent.
2. Separate "watch this now" (agent server) from "run this later" (automation server) — they have different failure and notification needs.
3. Sandboxing belongs to the deployment target, not the agent logic. The same agent should run unsandboxed locally and sandboxed in Docker without code changes.
4. A control plane's job is display, scheduling, and routing — not reasoning. Keep reasoning in the agent, keep orchestration in the plane.

---

# EDIT_FORMATS.md

How to get reliable file edits out of a model that outputs text. Case study: Aider.

## The Central Problem

An LLM emits a text stream. A file edit is a structured mutation. Something has to bridge that gap, and the bridge has to survive the model getting the format slightly wrong.

Aider's answer is to maintain **multiple edit formats** and pick per-model, because different models are reliable at different formats:

| Format | Shape | Failure mode it avoids |
|---|---|---|
| `wholefile` | model re-emits the entire file | no diff-application logic needed at all — simplest, but expensive and slow for large files |
| `editblock` (SEARCH/REPLACE) | model emits `<<<<<<< SEARCH ... ======= ... >>>>>>> REPLACE` blocks | avoids line-number drift; the block is matched by content, not position |
| `udiff` | model emits unified-diff-style hunks | compact for large files; needs strict hunk-header discipline |
| `patch` | a stricter, model-specific patch grammar | for models that reliably follow one exact grammar and nothing else |

Each format has its own **coder** class and its own **prompt file** — the prompt text itself changes per format, because the instructions for "how to emit a SEARCH/REPLACE block" are entirely different from "how to emit a unified diff." The prompt is not incidental; it is load-bearing infrastructure matched to a specific parser.

**The lesson: don't pick one edit format and hope every model is good at it. Match the format to the model's actual reliability profile, and treat the format-specific prompt as part of the format, not a detail.**

## The Main Loop

```
run_one(user_message):
    message = preprocess(user_message)
    while message:
        reflected_message = None
        send_message(message)             # stream LLM output, then apply edits
        if not reflected_message:
            break                          # clean success, done
        if num_reflections >= max_reflections:
            return                         # give up, don't loop forever
        num_reflections += 1
        message = reflected_message        # feed the error back in as the next turn
```

`send_message` does three things in sequence: streams the model's response, extracts edits via `get_edits()`, and applies them via `apply_edits()` after a dry run (`apply_edits_dry_run()`) confirms the patch actually matches the file content.

## The Reflection Loop

After edits land, the system runs the project's linter and tests. If either fails, the **error output becomes the next user message**, automatically, up to `max_reflections` times.

```
edit applied
   ↓
lint_edited() / cmd_test()
   ↓ failure?
reflected_message = "<lint/test error output>"
   ↓
loop restarts with that as input — the model sees its own mistake and fixes it
```

**This turns a static edit into a checked one, without a human in the loop.** The model is not told "you might have broken something" in the abstract — it is handed the exact compiler or test failure and asked again. A capped retry count keeps this from spinning forever on an unfixable error.

## The Repo Map

Before editing, the model needs to know what exists in a codebase far larger than its context window. Aider builds a **ranked, budget-fitted map** rather than dumping the whole tree.

```
1. Parse every source file with tree-sitter (falls back to Pygments tokenization
   for languages without a tags query file).
2. Build a directed graph:
       node = file
       edge = "this file references a symbol that file defines"
3. Weight edges by:
       - reference count (square-root scaled, so one file spamming a symbol
         doesn't dominate)
       - ×50 if the referencing file is already open in the chat
       - ×10 if the symbol name looks meaningful (camelCase/snake_case, 8+ chars —
         filters out noise like `i`, `x`, `tmp`)
       - ×0.1 if more than 5 files define the same symbol name (ambiguous, less useful)
4. Run PageRank over that graph, personalized toward files already in the
   conversation.
5. Binary-search the cutoff rank so the rendered map fits the token budget
   (accept anything within ±15% of target, otherwise narrow the search).
```

**Two ideas worth stealing independently of Aider itself:**
- **Use reference graphs, not file trees, to decide what a model should see.** A file's *importance* is how much other code depends on it, which PageRank captures and a directory listing does not.
- **Fit context to a token budget by search, not by a fixed truncation rule.** Binary-searching the cutoff against an actual tokenizer call is more robust than "top 50 files" or "first N lines," because it adapts to how verbose the ranked tags happen to be for this repo.

## What to Carry Away

1. Maintain multiple edit formats; match format to model reliability, not to your own preference.
2. A dry run before applying a patch catches "the model's diff doesn't actually match the file" before it corrupts anything.
3. Feed lint/test failures back into the conversation automatically, capped, rather than surfacing them to a human every time.
4. Rank context by a reference graph (PageRank), not by directory proximity or recency alone.
5. Fit to a token budget with a search loop against the real tokenizer, not a fixed heuristic.

---

# ACI.md

The Agent-Computer Interface: designing the tool surface itself as the lever, not just the agent. Case study: SWE-agent.

## The Core Idea

The insight this project is built around: **the tools you hand an agent are as important as the model driving it.** A human-oriented shell (raw `bash`, raw `sed`, error messages meant for humans) is a bad interface for a model, the same way a badly designed API is a bad interface for a human programmer. So the tools themselves are redesigned specifically for model consumption.

This is a general principle disguised as a tool list: **when you build an agent, design the tool surface as its own artifact, tuned to the actual consumer (a model), not repurposed from tools built for humans.**

## Tool Bundles, Not a Fixed Toolkit

Tools are packaged as swappable **bundles** — each a small self-contained folder with:

```
tools/<bundle_name>/
    config.yaml     # how this bundle's commands are described to the model
    bin/            # the actual executable(s) — often small wrapper shell scripts
    install.sh      # setup, if the tool needs anything installed in the sandbox
```

Real bundles observed: `registry` (core), `edit_anthropic` (editing tuned to Anthropic models' tendencies), `search`, `filemap`, `diff_state`, `windowed_edit_linting`, `windowed_edit_replace`, `submit`, `review_on_submit_m`, `web_browser`, `forfeit`.

**Why bundles instead of one fixed toolkit:** different tasks and different backend models want different editing primitives. `windowed_edit_linting` gives the model a scrolling window over the file plus post-edit lint feedback; `edit_anthropic` is phrased and shaped around how Claude models tend to specify edits. Swapping bundles is how the interface adapts to the model, without touching the agent loop itself.

## Multiple Action-Parsing Strategies

The bridge from "text the model wrote" to "a validated tool call" is not one parser — it is several, chosen per model:

| Parser | Expects |
|---|---|
| `ThoughtActionParser` | free-form reasoning, then a single command in a fenced code block — the parser takes the **last** non-nested block as the action |
| `FunctionCallingParser` | native tool-call output (via LiteLLM) — exactly one call per turn, validated against the registered command signature |
| `XMLFunctionCallingParser` | `<function=name><parameter=key>value</parameter></function>` — for models that are reliable at XML but not native function-calling |
| `JsonParser` | a JSON object with a `thought` field and a `command` field |
| `ActionOnlyParser` | the entire response is the action, no reasoning expected |
| `BashCodeBlockParser` / `SingleBashCodeBlockParser` | one or many fenced bash blocks specifically |

**Every parser ends at the same validation pipeline regardless of format:** extract the candidate action, confirm the command name exists in the currently loaded bundles, confirm required arguments are present, reject unexpected arguments, then format the arguments into the command's invocation template.

**The lesson: decouple "how the model expresses an action" from "how the action gets validated and executed."** Six input formats converge on one validation path. This is what lets you swap a model without rewriting the safety checks.

## The System Prompt Encodes a Fixed Procedure

The default configuration doesn't just say "fix the bug." It hands the agent an explicit five-step procedure: locate the relevant code, reproduce the reported error, edit the source, verify the fix, consider edge cases. And a submission checklist: re-run the reproduction script after edits, delete any temporary test scripts created along the way, revert accidental changes to test files, confirm before submitting.

**A fixed procedural scaffold in the prompt compensates for a model's tendency to declare victory early or leave scratch artifacts behind.** This is cheap and doesn't require any code — it is entirely a prompt-engineering lever, but a structural one (an ordered procedure), not a stylistic one.

## What to Carry Away

1. Treat the tool surface as a design object in its own right — not "give the agent bash," but "design what bash-equivalent access should look like for a model."
2. Make tools swappable bundles, so the interface can be retuned per task or per backend model without touching the core loop.
3. Support several action-parsing formats, but converge them all on one shared validation pipeline.
4. A written procedural checklist in the system prompt (reproduce → fix → verify → clean up → submit) is a real lever, not filler — it prevents an agent from declaring success prematurely.

---

# SDK_LAYERING.md

How to structure an agent so it can run headless, in an IDE, or in a hub, from the same core. Case study: Cline.

## The Layer Stack

```
@cline/shared     reusable low-level contracts: types, schemas, hook contracts,
                   remote-config primitives — no behavior, just shapes

@cline/llms       model/provider runtime: provider settings resolution,
                   AI-SDK-backed execution — talks to model APIs, nothing else

@cline/agents     the STATELESS runtime loop: tool orchestration, event emission
                   — explicitly holds no persistent storage

@cline/core       STATEFUL orchestration: sessions, persistence, plugin
                   discovery, hub services — the only layer allowed to remember
                   anything between calls
```

Each layer depends only on the ones below it. `@cline/agents` cannot reach into `@cline/core` for storage — if it needs to remember something, that is by construction the wrong layer to hold it.

**The one deliberate architectural decision worth naming: the agent loop is stateless by design.** State (sessions, persistence, history) lives one layer up, in `@cline/core`. This is what lets the *same* stateless loop power a one-shot CLI invocation, a long-running IDE session, and a hosted hub session — each host wraps the stateless core in whatever statefulness it needs, rather than the core assuming one hosting model.

## How a Session Actually Runs

```
Host constructs a RuntimeHost
   → starts a session
      → session creates an Agent (from @cline/agents)
         → Agent runs its loop using @cline/llms handlers
            → each iteration: orchestrate tool calls, emit lifecycle events
   → the host may intercept and modify message history or the system prompt
     BEFORE it reaches the provider call
```

That interception point — the host gets to edit history/prompt pre-flight — is what lets one host add IDE-specific context (open files, selection) and another host add nothing at all, without either patching the shared agent loop.

Tool implementations themselves live in `@cline/core` under `extensions/tools/`, not in `@cline/agents`. The agent layer only *orchestrates* calling them. Plugins register additional tools into the same seam. **Separating "the loop that decides to call a tool" from "the code that the tool actually runs" is what makes tools pluggable without touching the loop.**

## Plan / Act as a Genuine Mode Split

This lives in the IDE host (`apps/vscode`), not the shared SDK — it is a *host-level* feature built on the generic loop, which is itself informative: mode-switching is a UX pattern, not something the core loop needs to know about.

```
Plan mode:  gather information, ask clarifying questions,
            respond via a dedicated plan_mode_respond tool — no file mutation
Act mode:   execute — file edits, command execution, the real tool surface
```

Switching modes **saves the current mode's model configuration and restores the other mode's**, so a user can deliberately point Plan at a cheap/fast model and Act at a stronger one, without re-configuring anything by hand each switch.

**The lesson: separating "figure out what to do" from "do it" as an explicit, user-visible mode — each with its own tool surface and even its own model — gives a human a natural checkpoint to review a plan before anything touches the filesystem.** This is a cheaper, lower-friction version of the propose/verify split seen elsewhere: a human is the verifier, and the checkpoint is a mode boundary instead of a program.

## Checkpoints

Checkpoints are **git commits taken automatically after each tool execution**, not just at session boundaries. This gives:

- Diffing between any two points in a task, not just start-vs-now
- Restoring to any prior checkpoint, not only "undo last thing"
- Resuming a task later with full context intact, because the checkpoint chain **is** the history

**Using the version-control system already in the loop as the checkpoint mechanism is the efficient move — no bespoke snapshot format to invent or maintain.** Anywhere an agent mutates a filesystem repeatedly, "commit after every tool call" is a nearly free source of full undo/redo/diff for free.

## What to Carry Away

1. Make the core agent loop stateless; put persistence exactly one layer up, and only there.
2. Let hosts intercept and modify prompt/history before the provider call — this is how one loop serves multiple hosting shapes.
3. Separate "decide to call a tool" from "run the tool" across a layer boundary, so tools stay pluggable.
4. A plan/act mode split is a UX feature bolted onto a generic loop, not a change to the loop itself — and it can carry independent model choices per mode.
5. If you already have git, checkpoint by committing after every tool call rather than building a bespoke undo system.

---

# TOOL_SURFACE.md

Permissioned tool use, subagents, and durable memory in a terminal-native agent. Case study: Claude Code.

*Unlike the other four sections, the public `claude-code` repository is documentation, scripts, and plugins — not the agent-loop source. What follows describes the actual current design as specified to the agent at runtime and in public docs, not source extracted from that repo.*

## The Core Loop Shape

Same generate-act-observe shape as everything else in this document, but the emphasis sits on the boundary between "the model decided to act" and "the action actually runs":

```
model proposes a tool call
   ↓
permission layer classifies it: auto-allowed / needs confirmation / never allowed
   ↓ (if confirmation needed)
surfaced to the user with enough specifics to approve or deny in one look
   ↓
tool executes; result returns to the model
   ↓
loop continues
```

## A Three-Tier Permission Model, Not a Binary One

Actions are not simply "safe" or "dangerous." They fall into three explicit tiers:

| Tier | Examples | Handling |
|---|---|---|
| **Prohibited** | entering passwords/API keys into fields, permanent deletion, executing financial trades, bypassing CAPTCHAs | Never performed, even on explicit request with full details supplied — the agent states the rule and has the human do it directly |
| **Confirm-first** | sending a message on the user's behalf, publishing content, purchasing with a stored payment method, force-push | Ask in chat, wait for an explicit yes, then act — approval is per-action, not standing, and does not generalize to "future similar actions" |
| **Regular** | everything else — reading files, running tests, local edits | Proceeds without asking |

**The prohibited tier cannot be unlocked by user instruction.** This is a deliberate, load-bearing property: "the user asked for it and gave every detail" is not sufficient authorization for actions in that tier, because authorization for those specific action *types* is structurally out of scope for a chat instruction, full stop — it is a difference in kind from asking for confirmation. Confirm-first actions, by contrast, *can* be unlocked, but only by an explicit yes in the current turn — a prior approval of the same action type does not carry forward.

**The instruction-source boundary matters as much as the tier.** Content observed through tools — file contents, web pages, error messages — is data, never commands, regardless of what it claims about its own authority ("system message," "pre-approved," "urgent"). Only the human via chat can authorize an action. This is the direct defense against prompt injection: an attacker who can get text into a file or a web page the agent reads cannot use that text to grant itself permissions.

## Hooks: User-Owned Automation at Fixed Points

Rather than the agent deciding when to run linting, formatting, or notifications, the **user configures hooks** — shell commands that fire automatically at defined lifecycle points (before/after a tool call, on session stop, etc.). A hook that blocks an action is treated as user feedback the agent should reason about and adapt to, not an error to route around.

**This is the same "put the check outside the untrusted generator" pattern seen elsewhere in this document — except here the verifier is fully user-authored and user-owned**, rather than baked into the agent's own code. The agent cannot silently disable or reinterpret a hook; if a hook blocks something, the correct move is to figure out why, not to retry with `--no-verify`.

## Subagents: Delegation With a Narrower Window

A main agent can dispatch a task to a subagent with its own context window, its own tool access, and (optionally) a different underlying model. The subagent returns a final result; its intermediate reasoning does not consume the parent's context.

The documented failure mode to avoid: **don't delegate understanding.** A prompt like "based on the research, fix it" pushes the actual synthesis work onto the subagent instead of doing it in the parent. A well-formed delegation includes the concrete file paths, line numbers, and specifics that prove the delegating agent already understood the problem — the subagent should be doing bounded execution, not open-ended interpretation of a vague pointer.

Subagents matter for two distinct reasons that are easy to conflate:
- **Context protection** — keep a large, noisy exploration (grepping a big repo, reading many files) out of the main conversation, returning only the synthesized answer.
- **Parallelism** — run several independent lookups concurrently instead of serially.

If a task needs neither, spawning a subagent is pure overhead: cold start, no memory of the parent conversation, and a round-trip cost that a direct tool call would not have paid.

## Durable Memory as a Typed, Filed System

Memory is not one undifferentiated scratchpad. It is explicitly typed:

| Type | Captures | Explicitly excluded |
|---|---|---|
| `user` | role, expertise, how to tailor explanations | negative judgments about the user |
| `feedback` | corrections *and* confirmations of an approach, with the *why* | — |
| `project` | who's doing what and by when, with dates converted to absolute | ephemeral in-progress task state |
| `reference` | pointers to external systems (where bugs are tracked, which dashboard matters) | — |

Explicitly excluded from all types: anything derivable by reading the current code (architecture, file layout), anything git history already answers (who changed what), and debugging fix recipes (the commit message already has that context).

**The exclusion list is as important as the inclusion list.** A memory system that also stores what `git log` or `git blame` can answer on demand just accumulates staleness — a memory of "function X does Y" silently goes wrong the moment X changes, whereas asking the live repo never goes stale. The rule of thumb: store *why*, not *what* — the what is always re-derivable from current state; the why (a past incident, a stated preference, a deadline) usually is not recoverable from the code at all.

**Confirmations are explicitly called out as worth recording, not just corrections.** A memory system that only writes on failure drifts away from approaches the user already validated; capturing "yes, that was the right call" is what keeps the agent from re-litigating a settled decision on the next occasion.

## What to Carry Away

1. Three permission tiers, not two — "never," "ask this time," and "just do it" are different enough policies to warrant different code paths, and the top tier is structurally non-negotiable regardless of what the request says.
2. Data observed through tools is never a command source, no matter what it claims about its own authority. This is the actual defense against prompt injection, not a filter on suspicious-looking text.
3. Let the user own the verification layer (hooks) instead of hardcoding it into the agent — a blocked hook is feedback to reason about, not an obstacle to route around.
4. Delegate bounded execution to a subagent; keep synthesis and understanding in the parent. A vague delegation prompt is a sign the parent didn't do its own job first.
5. Type your memory system, and write down an explicit exclusion list. What's re-derivable from live state should never be persisted — only what would otherwise be lost (the why).

---

# SYNTHESIS.md (Agents)

What the five systems share, and how to combine the ideas.

## The Common Shape

Every one of these, again, is a loop — but the emphasis in agentic coding systems shifts from "is this proof correct" to "is this action safe/reversible, and did it work":

| System | What generates the action | What checks it |
|---|---|---|
| Agent Canvas | any ACP-compliant agent backend | the human, via the control-plane UI, across whichever backend they picked |
| Aider | the LLM's edit-format output | dry-run patch matching, then the project's own linter/test suite |
| SWE-agent | the LLM's parsed action | the ACI validation pipeline (command exists, args complete) then task-level review-on-submit |
| Cline | the LLM inside the stateless agent loop | git checkpoints (reversibility) plus a human-owned Plan-mode review gate |
| Claude Code | the LLM's tool call | a three-tier permission system, plus user-owned hooks |

**The recurring architecture is: keep the part that decides what to try cheap and replaceable, and put the real engineering into the part that checks, reverses, or gates the result.** This is the same asymmetry as the reasoning systems in this knowledge base — a kernel checking a proof, a symbolic engine checking a construction — applied to actions in a filesystem instead of statements in logic. The generator can be swapped, retried, or wrong; the checking layer is where correctness and safety actually live.

## Building an Agent — Checklist

**1. What checks the action, and can you name it in one sentence?**
"The linter and test suite," "git checkpoints," "a human in Plan mode," "the permission tier." If the honest answer is "we trust the model," that is the gap to close first.

**2. Is the check reversible, or only detective?**
Detective (a failing test after the fact) is weaker than reversible (a git checkpoint you can roll back to, a dry run before a patch lands). Prefer reversible where the cost is low — checkpointing after every tool call is nearly free if you already have git.

**3. Does your tool surface match what your model is actually good at?**
Aider ships four different edit formats for exactly this reason. Don't assume the model will adapt to your one preferred format; adapt the format to the model.

**4. Is the parsing layer decoupled from the validation layer?**
SWE-agent runs six different action-parsing strategies into one shared validation pipeline. If validation logic is duplicated per parser, a fix to one won't apply to the others.

**5. Where does state live, and is the core loop stateless?**
Cline's agent loop holds no persistent storage on purpose — sessions, history, and persistence live one layer up. A stateless core is what lets the same loop run headless, in an IDE, or in a hosted service without three different implementations.

**6. Are your permission tiers actually different policies, or one tier with exceptions bolted on?**
"Never," "ask every time," and "just proceed" are different enough in consequence that they deserve to be three real code paths, not one path with a growing list of special cases.

**7. Is anything you'd call "memory" actually just re-derivable from current state?**
If it's answerable by reading the code or running `git log`, it doesn't belong in a persisted memory — persist the *why* (a preference, an incident, a deadline), not the *what* (which is always re-checkable and therefore goes stale silently if cached).

**8. If you're running more than one agent, have you standardized the protocol between control plane and agent — or are you special-casing each backend?**
The moment a second agent implementation shows up, the ACP-style boundary pays for itself; special-casing does not scale past two.

## The One-Sentence Version

An agentic coding system is only as trustworthy as its cheapest-to-bypass check — so put the engineering into the checks (linting, dry runs, git checkpoints, permission tiers, human gates), and treat the model's output as a proposal, exactly the way the reasoning systems in this knowledge base treat a neural network's output as a proposal for a verifier to accept or reject.
