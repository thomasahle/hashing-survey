# Style Guide for `hash_survey.tex`

This document records the presentation conventions used across the survey, to keep notation and pseudocode consistent as the note evolves.

## Goals

- Prefer clear, high-level pseudocode over low-level implementation details.
- Use consistent notation across hashes, even when upstream code uses different names.
- Make dataflow obvious: what is input (`x_i`), what is evolving state (`v_i`), what is fixed/secret (`s_i`), what is a user seed (`seed`).

## Notation

Use these names consistently unless a section explicitly says otherwise.

- `p`: pointer to input bytes
- `n`: input length in bytes
- `seed`: user-provided seed (may be secret in keyed hashes)
- `x_i`: message words for the current block (word width stated by the loop header)
- `v_i`: evolving internal state (accumulators/lanes/state words)
- `s_i`: fixed constants, secret tables, or key material derived from the seed
- `k_i`: *only* for explicit secret key words in keyed constructions (e.g. SipHash’s `k_0,k_1`)
- Word types in pseudocode: `\ut{w}` (e.g. `\ut{32}`, `\ut{64}`, `\ut{128}`)

Avoid introducing ad-hoc names like `acc`, `keylength`, `mul0`, etc. Prefer expressing them as either `v`/`v_i` (state) or `s_i` (constants/keys). If a hash naturally has a single accumulator, prefer `v` over `acc`.

## Pseudocode: loops and reads

We avoid explicit `Read32/Read64/...` helper calls in algorithm blocks. Instead, the loop header defines how the input is parsed.

### Assignment operators

- Avoid compound “update” operators like `\oplus\gets` (or “⊕←”) since they aren’t standard in this note.
- Write XOR-updates explicitly as: `v \gets v \oplus x` (and similarly `v \gets v \oplus a \oplus b` when XORing several terms).

### Main loop (block iteration)

Preferred form (typed block tuples):

```tex
\For{each block $(x_0,\ldots,x_{k-1}) \in \ut{w}^k$}
    \State ...
\EndFor
```

- The tuple `(x_0,…,x_{k-1})` is the block payload.
- The type `\ut{w}^k` makes word width and block size unambiguous.
- Use `block` for the main-loop unit. Use `stripe` only where the hash’s own terminology is essential (XXH3).

For large blocks, prefer vector notation to avoid long tuples:

```tex
\For{each block $(x_i)_{i=0}^{k-1} \in \ut{w}^k$}
    \State ...
\EndFor
```

and then use index loops for structured updates:

```tex
\For{$i \in \{0,\ldots,k/2-1\}$} \Comment{Word pairs}
    \State use $(x_{2i}, x_{2i+1})$
\EndFor
```

### Tail handling

Prefer stating tail processing at a high level if it’s case-heavy:

```tex
\State Handle remaining bytes
```

If the tail is naturally word-based, use typed iterators:

```tex
\For{each remaining word $x \in \ut{32}$}
    \State ...
\EndFor
\For{each remaining byte $b \in \ut{8}$}
    \State ...
\EndFor
```

### Avoid “implicit reads”

Don’t write updates like `v_i ← Round(v_i, w_i)` without defining `w_i`. Either:

- bind `x_i` in the loop header, or
- write an explicit “interpret the next … bytes as …” line.

### Short-key special casing and tails (survey policy)

Many high-performance hashes have multiple size-specialized code paths (e.g. “short/medium/long”) and very detailed tail logic (overlapping loads, switch-on-length, alignment tricks).

In this survey:

- We **focus on the main/long-path structure** and present it in full.
- We typically **summarize** short-key paths with one sentence (or a brief bulleted list) describing how they differ, rather than reproducing every case.
- We typically **summarize tails** as “Handle remaining bytes” unless the tail itself is conceptually important to the design.
- When short-key/tail behavior affects the *mathematical structure* (e.g. different primitives or a distinct finalizer), we mention that difference explicitly.

## Per-hash structure

Each hash (or hash family member) should present the same high-level phases, so readers can quickly compare algorithms.

### Recommended order inside a hash subsection

- **Source / variant**: name the implementation file and, if relevant, the version/variant.
- **State + constants**: list the evolving state words (`v_i`) and fixed/secret material (`s_i`).
- **Core primitive(s)**: give one or two equations if the hash’s update step isn’t obvious from the pseudocode.
- **Algorithm pseudocode (single block)**: one `algorithmic` block should include:
  - initialization,
  - the main loop,
  - tail handling (explicit loops or a short summary),
  - lane collapse (if any),
  - finalization and the `\Return` line.
- **Finalizer / output**: state what finalizer is used (or that none is used) and the output width.

### Canonical “shape” to aim for

```tex
\State Initialize state $v_0,\ldots,v_{L-1}$ from $\mathit{seed}$ and $s_i$
\For{each block $(x_0,\ldots,x_{k-1}) \in \ut{w}^k$}
    \State Update $v_i$ using the core primitive
\EndFor
\State Handle remaining bytes (tail)
\State Collapse lanes (if any) into $v$
\State Apply finalizer/mixer (if any)
\State \Return output
```

### State naming rules (within a hash)

- Use `v_i` for evolving internal state throughout (including long-lived working variables).
- Use `v` for a single accumulator value (instead of `acc`).
- Use `t_i` for short-lived temporaries.
- Do **not** use `s_i` for evolving state; `s_i` is reserved for fixed constants/secret material.

### Lanes: prefer looped updates

When the algorithm has a fixed number of parallel lanes, prefer a short lane loop instead of unrolling repeated lines.

- Preferred: `\For{$i \in \{0,\ldots,L-1\}$}` with a clearly stated lane update using indexed `v_i` and `x_i`.
- Unroll only when the update is not uniform across lanes (e.g. distinct constants per lane not expressible as `s_i`, or complex cross-lane wiring that would become harder to read).

### Finalizer conventions

- The **main loop and the finalizer live in the same `algorithmic` block** (with the initialization at the top).
- Every scalar-output hash should end with an explicit finalization stage (even if it is “trivial”).
- Prefer inlining short finalizers as a few `\State` lines.
- If the finalizer is named (e.g. `XXH32_avalanche`), it is OK to call it in the final `\Return` line, but define it immediately adjacent (right after the algorithm) so it remains local.
- If the finalizer is a standard xorshift+multiply pattern, express it using the common building blocks from Preliminaries (e.g. `XMS`, `ShiftMix`) with explicit parameters.
- If there is no finalizer (or the algorithm’s reduction is already the final step), state that explicitly in one sentence.

## Functions and naming

- Define "round", "compress", and "combine" functions *where they are used* (locally in the relevant section), not in a global "Round Functions" section.
- Prefer small, named building blocks for repeated finalizers/mixers (e.g. `XMS`, `ShiftMix`) and express individual finalizers as compositions of those.
- **Every named function called in pseudocode must be defined.** Either define it as a `\Function` block, an equation, or in prose immediately before its use. Do not leave macro names (e.g. `MEOW_MIX`, `mumix`) as opaque calls — the reader should be able to see what operations they perform. Hardware intrinsics (`aesenc`, `aesdec`, `clmul`, `pshufb`) are defined once in the relevant section introduction (e.g. AES-Based Hashes) and may be used freely after that.

## MUM formatting

- Write MUM calls with thin space after the comma: `\Mum(a,\, b)`.
- If referencing the “protected”/folded variant, use the document’s definition (as stated in the Preliminaries / relevant family section).

## Constants

- Use `s_i` for fixed constants and secret tables.
- Show literal constants as `\texttt{0x...}`.
- If a hash uses multiple constant sets (e.g. 32-bit vs 64-bit primes), indicate this by superscripts (`s_1^{32}`, `s_1^{64}`) or by stating the width in text, but keep the base symbol `s_i`.

## When exceptions are OK

Some algorithms don’t naturally fit the “block as words” iterator (e.g. Polymur’s 56-bit chunking loop). In those cases:

- Keep the loop structure faithful (e.g. a `\While`), but still use `x_i` for message pieces, `v`/`v_i` for evolving state, and `s_i` for constants/keys.
- Add a short comment explaining the atypical block shape (e.g. “49 bytes per iteration” or “7×56-bit chunks”).

## Tables and guarantees (survey-wide)

- The comparison table uses **Provable width**: report effective bits `w = -log2(ε)` when a published collision bound applies; otherwise use `N/A`.
- If the bound depends on input length (e.g. `ε ≈ n/2^64`), reflect that dependence in the entry (e.g. `≈ 64 - log2 n`).
