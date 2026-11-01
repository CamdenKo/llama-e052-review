Follow-up: [bounded simplification](simplification/report.md) identifies the current candidate and fresh validation; earlier evidence below retains its historical scope.

Test-validation addendum: [adversarial-review repair](repair-report.md) supersedes the fixture coverage description below. Production kernel and historical performance evidence are unchanged.

# E052 Metal-only technical review notes

Recommendation: ready for human technical review of this local candidate, after independent evidence review. This is not an upstream-ready or publication-complete designation. Local technical evidence, not a submission description or commit message. No commit, post, push, PR, upload or settings change.

## Candidate and scope

C: `src/llama-e052-metal-only`, branch `codex/e052-metal-only`, uncommitted on fresh fetched master `dde879bc4f954b809892fd0d660515f42536394a`. The base did not advance, so no commit rebase was needed. [Narrow diff](../../patches/E052-metal-only.diff), SHA256 `651adffa954b9baeae5a46b01090a90de67279b23349597c09f963058a30cfde`: two files, 229 additions and 9 deletions. Only the Metal kernel and tests change. Public header, CPU, other backends and generic comparator/tolerances remain unchanged. A changes tests only; B adds only the per-entry Metal guard to A; C adds tile/query skipping to B. [Build receipt](metal-only-build-receipt.json), [scope checks](metal-only-scope-check.json), [source/library identities](../../raw/os-26B123/glm5-opt/e052-metal-only-ops-v2/manifest.json).

C writes exact negative infinity for every exact F16 negative-infinity mask. It skips whole hidden 64-key/query-group tiles and hidden queries in otherwise active tiles; it still computes hidden entries within an individually live query tile, then guards their output. This changes exceptional arithmetic, including finite FP32 inputs overflowing half staging. Unchanged backends and unfused score-plus-mask graphs can differ. No universal arithmetic-equivalence or portable-contract claim is made.

[Primary source review](metal-only-prior-art.md) found no explicit exceptional-propagation contract forcing scope expansion. Accepted Metal FA skipping #13493 supports backend-local work elision; SYCL #24941 is analogous numerical convention, not indexer proof. #26631 disproves a blanket simultaneous-backend requirement. Transformers v5.22.1 supports exclusion intent using a finite minimum and separate selected-validity check, not a universal negative-infinity representation. This candidate **extends** that work to indexer tile/query skipping and mixed-entry exclusion; acceptance remains human/upstream judgment. All primary links and comparison limits are in the source review.

## Correctness

[Standard raw](../../raw/os-26B123/glm5-opt/e052-metal-only-ops-v2/checks-complete.json): A finite-input overflow regression: 18/18 Metal FAIL versus 18/18 CPU OK. B and C independently pass 1,011 supported checks each, including 96 direct Metal exceptional/sentinel cases each; 18 IQ4_NL cases are unsupported. Unchanged CPU passes 933 cases. Ordinary comparisons cover eight supported K types, tails, query groups, streams/broadcasting and finite mask biases. [Independent review](metal-only-independent-review.md).

[External oracles](../../raw/os-26B123/glm5-opt/e052-metal-only-oracle-v1/complete.json): B/C each 132 dispatches and 141,240 entries pass explicit bits/classes across nine mask classes, mixed tiles/queries and tails. Exact-C [captured injection](metal-only-injection-review.md): 16 dispatches, 17,836,032 scores, 16,525,312 hidden exact-negative-infinity entries, 81,920 valid/unique selected IDs ; 148 control/foreign query-cases preserve score bits and selected sets. No fillers in these two fixtures. They are tile-aligned and lack hidden entries inside individually live query tiles; operator oracles supply those boundaries. Own-visible exceptional arithmetic can still produce NaN. Synthetic corruptions establish no natural incidence or demonstrated foreign-value access.

## Operator attribution

M5 Max, 40 GPU cores, 48 GB, macOS 27.2/26B123, Xcode 27.2/27B52, SDK 27.2, Metal 32023.922, Apple clang 21.0.0. Release, native ARM, embedded Metal. D128/H32/KV32768, F32 q/K/weights, F16 mask, FP32 output, half kernel staging. AC 140 W/high-power/open-lid. Five randomized A/B/C blocks, identical deterministic inputs, standard-tool warmup and at least 1 second per case. Each cell is median[min,max] microseconds; effect columns are medians of five paired speedups.

| B | Mask | A us | B us | C us | A/B | B/C | A/C |
|---|---|---|---|---|---|---|---|
|1|visible|38.17[38.00,38.33]|38.22[38.06,38.30]|39.51[39.33,39.64]|1.0016|0.9669|0.9662|
|1|equal sessions|37.95[37.75,37.98]|38.01[37.67,38.15]|39.02[38.98,39.16]|0.9963|0.9741|0.9691|
|1|synthetic32keys|37.82[37.69,37.94]|38.08[37.80,38.45]|7.08[7.07,7.13]|0.9921|5.3465|5.3437|
|16|visible|364.39[363.85,365.56]|364.45[363.89,367.04]|370.97[369.01,371.31]|0.9998|0.9846|0.9826|
|16|equal sessions|365.41[363.84,365.57]|364.61[364.04,365.21]|54.33[54.14,54.89]|1.0015|6.7189|6.7171|
|16|synthetic32keys|365.68[364.44,366.28]|365.52[363.92,366.83]|40.66[40.58,40.87]|1.0013|8.9823|8.9808|

Guard-only B is near A at this resolution. B-to-C isolates the skipping revision, including its scans/resource changes: visible controls regress; session/sparse cells improve. B1 equal-session density is 100%; B16 is 1/16, an idealized equal-history distribution. Sparse exposes exactly 32 keys, not a session history. The B1/B16 visible paired latency regressions are 3.50%/1.78%. Neither a unique instruction-cost explanation nor physical resource bottleneck is established.

[Corrected analysis and all paired spreads](../../raw/os-26B123/glm5-opt/e052-metal-only-ops-v2/analysis-corrected.json), [raw console/telemetry](../../raw/os-26B123/glm5-opt/e052-metal-only-ops-v2/). Operator results remain descriptive: telemetry covers whole processes, not per-case clock windows. All 465 thermal samples are Nominal; GPU-usage clocks min/median/max338/1620/1620MHz, power32/58576/78646mW. Initialization/idle are included. Nominal FLOPs include skipped work; no executed throughput, MFU or energy/token claim.

## Layer replay

[Exact-C replay](../../raw/os-26B123/glm5-opt/e052-metal-only-replay-v2/model/narrow-summary.json) uses `models/glm5-next-layers/glm5-pair-primary-Q4_K.gguf`, the Q4_K real-weight two-layer GLM5-next slice with synthetic embedding/output. It uses hash-verified historical native prefix bytes at context 32768, with explicit transfer receipts and new imports. B1/B16 both pass fresh-reference/import fidelity, zero native layer envelopes and bit-identical layer/indexer/DSA checks. This is common-state replay, not new independent-prefix validation. Five paired process blocks per cell use 128-call warmup and 635 retained steady calls/process. Times are medians[min,max] of five process medians; speedup is the paired geometric mean with conditional 95% t interval (df 4).

| B | A ms | C ms | Paired A/C [95%] | Qualification |
|---|---|---|---|---|
|1|1.6303[1.6255,1.6316]|1.6350[1.6288,1.6419]|0.996570[0.992077,1.001083]|all gates pass|
|16|11.8470[11.7024,12.2208]|10.5535[10.5305,10.7965]|1.125538[1.106747,1.144648]|all gates pass|

B16 latency falls **11.15% [9.65,12.64]** in this two-layer replay. B1 is inconclusive, with a 0.34% point slowdown and 0.80% adverse 95% endpoint, below the prospective 3% concern. All 20 timing processes pass clock coverage, paired-clock spread, thermal and footprint gates with zero own page-ins. Source/build binding has no drift. Full per-window clocks/power/resource values are in the raw analysis; its inherited revision description names the old final-runtime file, so the new runtime.json/bindings and narrow-summary establish candidate identity.

Relative to the combined patch: the guard is now independently validated without CPU or other-backend changes, exceptional tests target Metal explicitly, and deterministic A/B/C controls isolate skipping's gains and visible-work cost. The new B16 measurement confirms a narrower-candidate benefit near the historical 11.41%, without assigning the old 15-17% B32/64K gains to C. Those larger cells were not rerun. The broad cross-backend patch remains preserved as an optional proposal.

## Commands, failures and limits

Reproduction build template (actual targets were built separately, llama then test-backend-ops): `cmake -S <source> -B build/llama-e052-metal-<A|B|C> -DCMAKE_BUILD_TYPE=Release -DGGML_METAL=ON -DLLAMA_BUILD_TESTS=ON -DLLAMA_BUILD_EXAMPLES=OFF -DLLAMA_BUILD_SERVER=OFF -DLLAMA_BUILD_TOOLS=ON`; `cmake --build <build> --target llama test-backend-ops -j 8`. Exact configure/build logs are hashed in the build receipt. All leaf commands, selected libraries, timestamps and exits are in raw JSON receipts.

```text
.venv/bin/python scripts/e052_metal_only_ops.py --stage checks
.venv/bin/python scripts/e052_metal_only_oracle.py
.venv/bin/python scripts/e052_metal_only_ops.py --stage perf
.venv/bin/python scripts/e052_metal_only_replay.py --stage prepare
.venv/bin/python scripts/e052_metal_only_replay.py --stage injection
.venv/bin/python scripts/e052_metal_only_replay.py --stage model
python3 scripts/analyze_e052_metal_only.py
```

Preserved failures: v1's wrong Metal/MTL registry predicate silently skipped the 96 direct cases; v1 is invalidated for oracle coverage. The corrected count gate rejects its zero cases, then v2 executes all 96. V1 raw/diffs/controllers remain saved. V1 replay prepared helpers only and is superseded after source freeze. The first power analysis pooled two same-named GPU Power fields/sample; original retained, corrected analysis separates sections. Initial `fetch origin master` failed because only `upstream` exists; corrected fetch succeeded. Sandboxed ps was denied; escalated read-only ps found no competing benchmark. An initial scope substring check used a stale offset after an include insertion; independently located anchors pass.

Historical native DSA numerical failures and B8 independent-prefix envelope miss remain unwaived. Incomplete two-layer GGUF slices with synthetic embedding/output are not full-model llama-bench/PPL/KL targets. Full-model quality, full local CI, older Apple GPU/toolchain coverage, fleet/Ultra transfer and universal backend parity remain unestablished. Original combined patches, branches, measured trees, raw results and failed checks remain preserved; broad backend parity is optional future work. No final submission prose or commit wording is prepared here.

[Independent final review](metal-only-independent-review.md) clears the bounded numerical, identity and operating checks. [Saved preflight blocks](metal-only-preflight-blocks.md) include every valid session. Preflight warnings retain resident Instruments, background apps/global swap and controller self-detection; no settings were changed. No owned benchmark or sampler remained in final process inspection. Four delegated workers handled source checks, tests, replay preparation/raw checks, and independent review. Agent token/runtime cost was not measured. The 20 timing processes retain 12,700 steady calls; their two acquisition cells took 275.36 seconds including preflight, import checks, initialization and timing, not pure GPU busy time.

Final read-only upstream check still returns `dde879bc4f954b809892fd0d660515f42536394a`. [Preservation checks](metal-only-preservation-check.json) verify selected historical artifacts. Official editorconfig-checker is absent; diff whitespace passes, with no official editorconfig/full-CI clearance. Final binding-check import initially failed under system Python (missing NumPy); it passes under the same `.venv` used by acquisition.
