# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LaTeX survey paper: "A Survey of Non-Cryptographic Hash Functions — As Implemented in SMHasher3" by Thomas Dybdahl Ahle. Covers 28+ non-cryptographic hash functions, organized by primitive family (MUM, ARX, CLMUL, polynomial, AES-NI, SIMD).

## Build

```bash
pdflatex hash_survey.tex
```

No Makefile or build automation exists. Run from the repo root. Build artifacts (`.aux`, `.log`, `.out`, `.toc`, `.pdf`) are untracked.

## Structure

- `hash_survey.tex` — Main document with package imports, custom macros, and `\input` calls
- `sections/` — 12 section files included in order: introduction, comparison, preliminaries, round_functions, wyhash_family, a5hash_family, xxhash_family, siphash_family, other_notable, clmul_polynomial, aes_based, simd_scalable
- `STYLE.md` — Detailed style guide (READ THIS before editing any section)

## Key Macros (defined in hash_survey.tex)

- `\ut{w}` — Word types in pseudocode (e.g., `\ut{64}` renders as u64)
- `\Rot{k}{x}` — Rotation: `rot_k(x)`
- `\Mum`, `\Round`, `\Merge`, `\SipRound`, `\Clmul`, `\Aesenc`, `\Aesdec`, `\ShiftMix`, `\Fmix` — Clickable function references (hyperlinked to their definitions)
- `\Xorshift{r}` — Parameterized xorshift
- `\Read{w}` — Word read primitive
- `\cmark` / `\xmark` — Green checkmark / red X for tables

## Style Guide (STYLE.md) — Critical Rules

All content must follow STYLE.md. Key points:

**Notation** — Use consistently: `x_i` (message words), `v_i` (evolving state), `s_i` (fixed constants/secrets), `k_i` (explicit secret keys), `seed` (user seed), `p` (input pointer), `n` (input length). Do not invent ad-hoc names like `acc` or `mul0`.

**Pseudocode** — No compound operators (`⊕←`); write `v \gets v \oplus x`. Loop headers must define input parsing via typed tuples: `\For{each block $(x_0,\ldots,x_{k-1}) \in \ut{w}^k$}`. Avoid implicit reads.

**Per-hash structure** — Each hash subsection follows: source/variant, state+constants, core primitive(s), single algorithm block (init → main loop → tail → lane collapse → finalize → return), finalizer/output description.

**Scope** — Focus on main/long-path. Summarize short-key paths and tails briefly unless they affect mathematical structure. Define all named functions locally, not globally. Use common building blocks (XMS, ShiftMix) for finalizers.

**MUM formatting** — Thin space after comma: `\Mum(a,\, b)`.
