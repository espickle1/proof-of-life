# Getting Started with Lean 4 via Claude Code

A handoff guide for using Claude Code to develop formally verified mathematics in Lean 4, with a focus on information-theoretic problems.

---

## 1. Installation

```bash
# Install elan (Lean version manager) — like nvm for Node
curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh
source ~/.profile   # or restart terminal

# Verify
lean --version
lake --version
```

---

## 2. Create a New Project

```bash
lake new myproject
cd myproject

# Add Mathlib (Lean's standard math library, 60k+ theorems)
lake add mathlib

# Pull prebuilt Mathlib binaries — CRITICAL, saves hours of compilation
lake exe cache get
```

Your project structure will look like:
```
myproject/
├── lakefile.lean       # project config (like package.json)
├── myproject.lean      # main file
├── Myproject/          # module directory
└── lake-manifest.json  # dependency lockfile
```

---

## 3. VS Code Setup (for local work)

Install the **lean4** extension by `leanprover`. It gives you:
- Live proof state at each line
- Goal display as you write tactics
- Error highlighting in real time

This is your best feedback loop when working locally.

---

## 4. GitHub Setup

```bash
git init
git add .
git commit -m "initial lean project"
gh repo create myproject --public
git push -u origin main
```

Add a `.gitignore`:
```
.lake/
```

### CI with GitHub Actions

Create `.github/workflows/build.yml`:

```yaml
name: Build
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: leanprover/lean-action@v1
        with:
          build-args: "--wfail"
```

A green checkmark means every proof in the repo is **mathematically verified**. This is a stronger guarantee than any test suite.

---

## 5. Starting Claude Code

```bash
cd myproject
claude
```

### Recommended opening context for Claude Code

Paste this at the start of each session:

```
We are working in Lean 4 with Mathlib. The domain is information theory 
applied to biological sequences (DNA/protein alphabets).

Rules:
- Before writing any proof, search Mathlib for relevant existing lemmas 
  using `exact?`, `apply?`, or grep.
- Always run `lake build` after changes and fix all errors before moving on.
- Mark any unproven steps with `sorry` and a comment explaining what's needed.
- Prefer building on Mathlib's existing probability and measure theory 
  infrastructure (Mathlib.Probability, Mathlib.MeasureTheory).
- When stuck on a proof, outline the strategy as comments before writing tactics.
```

---

## 6. Key Mathlib Modules for Information Theory

| Module | Contents |
|---|---|
| `Mathlib.Probability.ProbabilityMassFunction.Basic` | Discrete probability distributions |
| `Mathlib.MeasureTheory.Measure.MeasureSpace` | Measure spaces |
| `Mathlib.Analysis.SpecialFunctions.Log.Basic` | `Real.log`, log inequalities |
| `Mathlib.Algebra.BigOperators.Basic` | `Finset.sum` notation |
| `Mathlib.Data.Fintype.Basic` | Finite types (e.g. a 4-letter alphabet) |

Import them at the top of any `.lean` file:
```lean
import Mathlib.Probability.ProbabilityMassFunction.Basic
import Mathlib.Analysis.SpecialFunctions.Log.Basic
```

---

## 7. The Core Workflow

### Step 1 — State what you want to prove (even with `sorry`)

```lean
/-- Shannon entropy of a finite distribution is non-negative -/
theorem entropy_nonneg (p : Fin n → ℝ) (hp : ∀ i, 0 ≤ p i) 
    (hn : ∑ i, p i = 1) :
    0 ≤ -∑ i, p i * Real.log (p i) := by
  sorry
```

This compiles (with a warning). You have a verified *interface* before the *proof*.

### Step 2 — Ask Claude Code to fill it in

> "Prove `entropy_nonneg`. Search Mathlib for lemmas about `Real.log` 
> and `Finset.sum_nonneg`. Run `lake build` after each attempt."

### Step 3 — Build iteratively bottom-up

Break hard theorems into helper lemmas, prove the easy ones first, 
keep `sorry` only on the genuinely novel parts.

---

## 8. Useful Lean Tactics Cheatsheet

| Tactic | What it does |
|---|---|
| `exact h` | Close goal with hypothesis `h` |
| `apply lemma_name` | Apply a lemma, generating subgoals |
| `simp` | Simplify using known identities |
| `ring` | Prove algebraic equalities |
| `linarith` | Prove linear arithmetic goals |
| `norm_num` | Prove numeric computations |
| `induction n` | Induct on a natural number |
| `exact?` | Search Mathlib for a matching lemma |
| `apply?` | Search for applicable lemmas |
| `simp?` | Find which simp lemmas close the goal |
| `sorry` | Placeholder — proof accepted but flagged |

---

## 9. Useful Claude Code Prompts for Lean

**Finding existing Mathlib lemmas:**
> "Search Mathlib for lemmas about `Real.log` being negative on (0,1). 
> Use `grep -r` in the Mathlib source and also try `exact?` in a scratch file."

**Fixing type errors:**
> "Here is the error from `lake build`: [paste error]. 
> The expected type is X, we have Y. Find the right coercion or lemma."

**Proof strategy:**
> "I want to prove [theorem]. Don't write the proof yet — outline it as 
> a sequence of mathematical steps and identify which steps Mathlib 
> likely already covers."

**Checking `sorry` status:**
> "Run `grep -r 'sorry' .` and list every unproven gap with a comment 
> on what kind of mathematics each one needs."

---

## 10. The `sorry`-Driven Research Workflow

For novel mathematics (your biological information theory results):

```
1. State the full theorem with sorry
2. Ask Claude Code to decompose it into lemmas
3. For each lemma, check if Mathlib has it (exact? / apply?)
4. Prove the Mathlib-covered lemmas fully
5. sorry only remains on genuinely new mathematical content
6. Those are your actual research contributions
```

The gaps that resist automation are where the mathematics is truly new.

---

## 11. Resources

| Resource | URL |
|---|---|
| Lean 4 documentation | https://leanprover.github.io/lean4/doc/ |
| Mathlib docs | https://leanprover-community.github.io/mathlib4_docs/ |
| Theorem Proving in Lean 4 (free book) | https://leanprover.github.io/theorem_proving_in_lean4/ |
| Mathlib source on GitHub | https://github.com/leanprover-community/mathlib4 |
| Physlib (physics + quantum info) | https://github.com/leanprover-community/PhysLib |
| Lean Zulip chat (community) | https://leanprover.zulipchat.com |
| Lean web playground | https://live.lean-lang.org |

---

## 12. Next Step

Open a new chat and describe your problem domain concretely:
- What are your mathematical objects? (sequences, distributions, alphabets)
- What properties do you want to prove?
- What would a meaningful first theorem look like?

The translation from biological intuition to Lean-ready mathematics 
is its own step — get that right before writing any code.
