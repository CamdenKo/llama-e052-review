Follow-up: [bounded simplification](simplification/report.md) identifies the current candidate and fresh validation; earlier evidence below retains its historical scope.

# E052 adversarial-review repair

Completed November 1, 2026. Local validation only; no commit or external publication. No owned GPU/sampler job remains. No independent reviewer was used for this repair.

## Changes

The candidate test file uses B20 instead of B17 in the mixed-mask and direct-exclusion loops. The final four-query tile contains visible queries 16, 17 and 19 and hidden query 18 for mask stream zero. Mixed-mask visibility also depends on mask stream, making an incorrect stream index observable deterministically. The direct oracle uses the shared operation-filter helper. Its before guard is allocated immediately before the output, with runtime address assertions for both guard allocations. The after guard follows backend alignment padding; padding bytes are not checked.

The production Metal kernel is unchanged byte-for-byte from the measured revision. Optional kernel cleanups were deferred. The Metal-specific NaN/+infinity checks remain; no universal backend contract is asserted. Native A received the same repaired test file; its prior source identity is preserved in the historical v2 diff. B and all historical raw measurements remain unchanged. The old patch remains saved; the repaired candidate is patches/E052-metal-only-review-repaired.diff.

## Measured correctness evidence

- Repaired C direct oracle: 96/96 pass, including runtime guard adjacency checks.
- Native A negative control: 96 direct cases execute, 72 fail as expected.
- Wrong fully-hidden tile row mutant: 72/96 direct cases fail.
- Wrong hidden-query output mutant: 24/96 direct cases fail.
- Wrong visibility-scan mask stream mutant: 24/144 mixed-mask comparisons fail.
- Restored C relevant suite: 1,011 supported checks pass, 18 IQ4_NL checks unsupported. Indexer coverage is 450 supported checks (354 ordinary plus 96 direct), rather than 1,011 indexer checks.
- CPU relevant suite: 933/933 pass, including 372 indexer comparisons.
- Parenthesized operation filter: 96 direct cases execute and pass.

All mutations compiled and ran; expected failures are numerical assertion failures, not build errors or zero coverage. These checks corroborate the existing candidate behavior and extend validation sensitivity; they are not new performance or silicon findings. Guard-only B passing remains expected: correctness tests must reject wrong skipping, not the absence of an optimization.

Evidence: repair-runs/results.json, complete.json, count-validation.json, identities.json, individual build/console logs, and kernel-preservation.txt in this directory. Contract: repair-contract.md. Preflight: repair-preflight.txt. Machine/software configuration remains M5 Max 40 GPU cores, 48 GB, macOS27.2/26B123, Xcode27.2/27B52, SDK27.2, Apple clang21.0.0; see attached preflight and prior configuration records. These are correctness runs, not warmup/repeated performance measurements; no new timing/power conclusion is made.

## Failures and limits

The first sandbox attempt exposed no Metal device and executed zero cases; its successful exit is invalid coverage. Host-access rerun executes all96. The mutation controller initially rejected an ambiguous stream anchor (two occurrences) before applying that mutant, restored the kernel and rebuilt. The anchor was narrowed to the visibility scan; completed evidence retains both earlier successful mutants.

Historical performance evidence remains attached to its original source/test/binary identities. The revised test binary is not represented as the historical measured binary. No new performance campaign was needed because production kernel code is unchanged. Full-model quality, full CI, older GPU/toolchain coverage, historical numerical failures and Ultra/fleet transfer remain open. The next step is human technical review; no queued experiment.

Visual-owner event ping returned `No Codex thread found`; no visual files were edited.
