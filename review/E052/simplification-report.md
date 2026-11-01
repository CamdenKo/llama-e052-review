# E052 bounded simplification

Completed November 1, 2026. Local-only; no commit or publication. No independent review of this follow-up. Ponytail was unavailable in this session; the explicit relayed review guidance was applied through ship/lead-researcher.

## Scope decision

Keep the production kernel unchanged and all96 direct exceptional cases. Direct exceptional=1 uniquely combines finite-overflow inputs, poisoned output, adjacent guard checks and B9/B20 tile boundaries. The ordinary overflow regression lacks those guard/poison checks and uses B1/7/17. Removing it would cut meaningful validation.

Remove synthetic performance pattern3 and its initializer from the proposed PR. Keep fully visible and equal-session controls. Shorten the exceptional-oracle comment. The two-file candidate falls from225 additions/9 deletions to220 additions/9 deletions; kernel remains48 additions/4 deletions. No abstraction, dependency, option or unrelated kernel cleanup was added.

Current patch: patches/E052-metal-only-simplified.diff. Prior patches and historical raw data remain unchanged. To reproduce historical pattern3, use the saved pre-simplification patch patches/E052-metal-only-review-repaired.diff on base dde879bc4f954b809892fd0d660515f42536394a in a separate checkout. The historical scripts/e052_metal_only_ops.py expects six performance cells and must be used with that historical source, not the simplified four-cell candidate. No new performance campaign was run; unchanged production kernel preserves the relevance of historical measured effects within their original limits and identities.

## Runner repair and validation

scripts/e052_review_repair.py creates a unique fresh directory and refuses an existing directory, creates logs exclusively, starts with no cached rows, captures its exact script and binds each run to kernel/test/executable/library hashes. Explicit case/failure gates remain, as does finally restoration and rebuild. No resume path exists. Historical repair-runs was not reused or overwritten. Pre-change runner/tests are saved here as runner-before.py and tests-before.cpp.

Fresh complete execution: references/E052/repair-runs-20261101T203704872843Z. Native A fails72/96 direct cases. Tile-row mutant fails72/96, hidden-query mutant24/96, stream-scan mutant24/144. Each mutant compiles/runs and fails result checks. Restored C has1011 supported Metal passes plus18 unsupported IQ4_NL cases (450 supported indexer checks); CPU933 passes. Case totals independently parsed in completion.json. Production kernel restored to its exact prior hash; whitespace check passes. Preflight saved here with0 FAIL/14 WARN. These are correctness checks, not performance measurements.

Full-model quality, full CI, older-GPU/toolchain coverage, historical numerical failures and Ultra/fleet transfer remain open. Final submission text remains human-owned.

Visual-owner event ping returned `No Codex thread found`; no visual files were edited. Final process audit shows no owned validation/sampler jobs.
