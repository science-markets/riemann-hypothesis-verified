# What the Lean verification of the Riemann Hypothesis claim establishes

On 20 November 2026, Levent Alpöge published a purported proof of the Generalized Riemann Hypothesis, produced with Claude, together with a Lean 4 formalization. Within hours an independent verification audit appeared and a set of kernel checks ran to completion.

This page records what those checks show, what they do not, and how to reproduce each claim. Every number here was read off the audit or computed locally.

- Claim source: [alpo.ge/remark.pdf](https://alpo.ge/remark.pdf)
- Verification audit: [math.mit.edu/~drew/grh](https://math.mit.edu/~drew/grh/), by Andrew Sutherland at MIT
- Source mirror: [leanprover-community/riemann-hypothesis](https://github.com/leanprover-community/riemann-hypothesis)

## The short version

The formal statement was proved. The kernel checks ran and passed. The mathematical argument has not been read by anyone.

That separates two questions the coverage ran together: whether a formal statement follows from the axioms, and whether anyone understands why.

## Scope: which hypothesis

The audit states it plainly: **"GRH here means Dirichlet L-functions only."**

The five theorems cover the Riemann Hypothesis for zeta, plus the analogous statement for all Dirichlet L-functions. Not Artin L-functions, not Dedekind zeta functions of general number fields, not L-functions of automorphic forms. No kernel check can widen a statement.

The `riemannHypothesis` theorem is stated against Mathlib's own definition in `Mathlib/NumberTheory/LSeries/RiemannZeta.lean`, which excludes the trivial zeros and the point s = 1.

## What was checked

| Check | Result |
|---|---|
| All modules compile | 175,187 of 175,187, zero errors (Lean v4.33.1, Mathlib v4.33.1) |
| Axioms of the five theorems | propext, Classical.choice, Quot.sound only |
| Axioms outside Lean core | none (15 in environment, all in Init) |
| Mathlib vs official cache | 8,322 + 378 olean files byte-identical |
| leanchecker kernel replay | 175,187 of 175,187, zero rejections |
| nanoda (independent Rust kernel) | accepts full export, exit 0, 27m42s on 80 threads |
| con-leche | 1,468,418 declarations accepted, exit 0, 3m15s on 64 threads |
| con-leche --compare | all five theorems justified by the solution |
| lean4lean | 175,187 of 175,187 (five needed a raised step limit) |

A Lean proof can depend on `sorryAx`, a placeholder that proves anything, and still compile. `#print axioms` shows what a theorem actually rests on. These rest on three axioms that are part of Lean's foundations, the same ones any ordinary Mathlib theorem uses.

Three independent kernel implementations agreeing matters because a single kernel bug would fool only one of them.

## What was not checked

The audit's own caveats:

1. The build was unsandboxed, calling lean directly rather than through lake build. A 77-module sample is byte-identical to a real lake build; the full set was not compared.
2. Export provenance inherits that. A sandboxed re-export should hash identically if exporting is deterministic. That comparison has not been made.
3. A kernel pass is conditional on the kernels being correct and on Mathlib's definitions meaning what they are intended to mean.
4. The lake comparator and the Lean-kernel replay of the export were still running when the audit page was last updated.

Two checkers could not run for reasons unrelated to correctness. con-leche and con-ron as bundled with Lean v4.36.0 refuse a mutual inductive block with a reflexive member, which this proof contains. lean4lean bundled with v4.36.0 cannot read a v4.33 core library. Neither is a rejection.

## Checking the export yourself

```bash
curl -O https://math.mit.edu/~drew/grh/challenge.ndjson.zst
zstd -d --long=27 challenge.ndjson.zst
shasum -a 256 challenge.ndjson
# bd484ac9886a96f26b1d9a7a59ebd45b7a70d102c015d4f31f9302bbdd6ab2b9
```

That hash matches the audit's Provenance table. The file has 6,495,712 records and 183,806 string records, 30,044 unique, and not one contains a slash or backslash. The export format has no field that could carry a source path, which is why the build-directory dependence that affects olean hashes does not transfer to the export the kernels judge. Details in [issue #6](https://github.com/leanprover-community/riemann-hypothesis/issues/6).

A warning if you try it: `zstd -d` exits 0 on truncated input and reports the truncation only on stderr. Piping it with `2>/dev/null` will silently scan a fragment. Check the byte count before trusting any count.

## The other proof, and why both exist

Separately, Alessandro Maria Masullo released a 940-page human proof of GRH, announced by Ravi Vakil. Brian Conrad had been reviewing it for months and gave feedback that was being incorporated.

The two are not the same argument scaled differently. The Claude route runs through automorphy lifting: Galois cohomology growth bounds, Galois representations attached to regular algebraic cuspidal automorphic representations of GL_n over CM fields, theta lifts between unitary groups. Searching Masullo's 940 pages gives zero hits for Eichler-Shimura, CM field, Hilbert-Samuel, Moriwaki. He uses Weil's explicit formula and the trace formula instead.

Two independent arguments for the same statement is far stronger evidence than one argument checked twice. It is also falsifiable. If someone shows the routes are secretly the same, the point collapses.

What argues for caution on the human proof is narrower than "AI found bugs". A catalogue of local errors exists in [mccorvie/riemann-masullo](https://github.com/mccorvie/riemann-masullo), eleven entries marked false as stated with worked counterexamples. All eleven are in the appendices, in recaps of standard material. None has specialist review. And the draft is unfinished: 27 bare "Return here" notes across pages 199-483, and a Chapter 13 that states no theorem.

## Why this page exists

It is run by [science.markets](https://science.markets/markets/riemann-zeta-critical-line/?ref=gh), which lists a market on whether this proof holds.

That is why the manuscript and the audit were read closely, and it is a reason to read the above sceptically. I have a position. The commands and numbers are given so you do not have to trust me for them.

Corrections are welcome as issues.
