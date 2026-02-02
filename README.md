# A Survey of Non-Cryptographic Hash Functions

**As Implemented in SMHasher3**

By Thomas Dybdahl Ahle

A comprehensive survey of 25+ non-cryptographic hash functions, organized by primitive family:

- **MUM-based**: wyhash, rapidhash, a5hash, MuseAir, komihash, XXH3
- **ARX-based**: xxHash32/64, SipHash, HalfSipHash, SpookyHash, FarmHash
- **CLMUL/polynomial**: CLHash, UMASH, Polymur, poly-mersenne
- **AES-NI**: AquaHash, MeowHash, aHash
- **SIMD-scalable**: HalftimeHash, khashv
- **Other**: ChibiHash, Rainbow, prvhash, XMSX

Each algorithm is presented with clear pseudocode and consistent notation. The paper includes a performance comparison table with SMHasher3 test results, a structural comparison of design choices, and an appendix analyzing the MUM primitive.

## Read the paper

[hash_survey.pdf](hash_survey.pdf)

## Build

```bash
pdflatex hash_survey.tex
bibtex hash_survey
pdflatex hash_survey.tex
pdflatex hash_survey.tex
```

## Structure

- `hash_survey.tex` — Main document with package imports and macros
- `sections/` — Section files (introduction, comparison, preliminaries, per-family sections, appendices)
- `refs.bib` — Bibliography
- `STYLE.md` — Style guide for notation and pseudocode conventions
