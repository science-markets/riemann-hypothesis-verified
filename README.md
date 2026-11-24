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
2. Export provenance inherits that. A sandboxed re-export should hash identically if exporting is deterministic. The challenge export does match the one this repository’s CI publishes, which is a point for determinism on that side. No such comparison exists for the 61.6 GB solution export, so the sandboxed re-export that would settle it has not been run.
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

They do not appear to be the same argument scaled differently, though I cannot settle that. What I can report: the Claude route is described as automorphy lifting, with Galois representations attached to regular algebraic cuspidal automorphic representations of GL_n over CM fields and theta lifts between unitary groups. Masullo's text never says "CM field", "Eichler-Shimura", "Hilbert-Samuel" or "Moriwaki", and the words it does use heavily are "trace formula" (157 times), "Weil" (224) and "explicit formula" (59). "CM" appears 42 times, every one of them a constant like C_M.

Keyword counts are weak evidence and I am treating them as that. They cannot distinguish a genuinely different route from the same route in different notation, and settling it needs a reader who knows both. If someone shows the routes converge, the point collapses. That is the test.

If the two routes are independent, two arguments for one statement is stronger evidence than one argument checked twice.

What argues for caution on the human proof is narrower than "AI found bugs". A catalogue of local errors is kept in [mccorvie/riemann-masullo](https://github.com/mccorvie/riemann-masullo): as of 24 November, 277 entries, each a place where the printed statement fails with a worked counterexample. 226 are in the body chapters, 51 in the appendices. 187 have been rechecked against the page image by a second and then a third reader, the third tasked with refuting the second. 84 are refutations checked by Lean, meaning the failure is a theorem rather than an argument.

None of that has specialist review. The entries are machine-found and machine-checked, and a number theorist who knows the surrounding literature has not signed off on any of them. It is evidence that the draft has many local problems, not a verdict on the proof.

The draft is also unfinished: 30 bare "Return here" notes in the text, and a Chapter 13 whose main theorem is still a placeholder. One placeholder, (3.3.1.11), states a bound whose exponent omits the archimedean degree. I worked out a counterexample over Q(i) and filed it as issue #2814 before finding it was already entry F16, filed five hours earlier. That is the normal outcome for this kind of check and worth saying plainly: the useful work here is being done by several people at once, and the catalogue is the product, not any one entry.

## What this page is not

This page is an explainer. It is not research, it does not settle anything, and it is not part of how any market resolves.

**Nothing here has anything to do with settlement or pricing.** science.markets runs a market on the Riemann hypothesis, and the README you are reading is not a resolution source, not an evidence submission, and not an input to any price. It does not move the market's estimate and it is not consulted when the market settles. Settlement runs on the resolution criteria published on the market page itself, against public primary sources. Read those criteria, not this page.

I do have an interest: [science.markets lists the market in question](https://science.markets/markets/riemann-zeta-critical-line/?ref=gh). Treat that as a reason to check the numbers rather than trust the prose. Every command and count on this page is given so you can reproduce it, and where I could not check something I have said so.

Corrections are welcome as [issues](https://github.com/science-markets/riemann-hypothesis-verified/issues). Where this page and a source disagree, the source is right.
