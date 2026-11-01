# E052 private review evidence

This packet supports the draft PR in CamdenKo/llama-e052-review. It is a saved local evidence snapshot, not a production-readiness certification. Codex prepared the private review text with Camden's explicit authorization on November 1, 2026. Camden is learning the implementation; independent line-by-line human review remains outstanding. No reviewer has been invited yet.

## Start here

The PR compares `codex/e052-review` with `e052-base`, pinned to upstream `dde879bc4f954b809892fd0d660515f42536394a`. It changes only the Metal shader and existing operator tests. This `e052-evidence` branch holds review material separately.

| File | Purpose and limits |
| --- | --- |
| [candidate.diff](candidate.diff) | Exact current patch; SHA256 b4023881f4207aef8c758196c0ea8580190a7457d22931ae64bbaadc0fbbbe69 |
| [metal-only-report.md](metal-only-report.md) | Narrow candidate's methods, A/B/C controls, measurements, and limitations; follow its simplification/repair history through the files below |
| [simplification-report.md](simplification-report.md) | Removes a synthetic performance pattern; production kernel unchanged |
| [repair-report.md](repair-report.md) | Test sensitivity repairs, failing mutants, sentinel checks |
| [comment-cleanup.json](comment-cleanup.json) | Final comment-only cleanup and patch identity |
| [operator-analysis.json](operator-analysis.json) | Five paired operator blocks and visible-work regressions; original source identities retained |
| [replay-summary.json](replay-summary.json) | Exact-C B1/B16 common-state two-layer replay; not full-model validation |
| [regression-complete.json](regression-complete.json) | Fresh repair run commands, counts, exits, and hashes; includes deliberately failing mutants |
| [prior-art.md](prior-art.md) | Pinned primary precedents and semantic comparison limits |
| [submission-source-refresh.md](submission-source-refresh.md) | Upstream policy/duplicate-work refresh, November 1, 2026; its older CI readiness status is superseded by ci-summary.md |
| [ci-summary.md](ci-summary.md) / [ci-report.md](ci-report.md) | Completed CPU/Metal drivers pass; hosted leak/autorelease and offline BF16 failures remain |
| [ci-result-matrix.json](ci-result-matrix.json) | Per-stage counts, unsupported cases, PPL gates/exemption, and terminal statuses |
| [full-model-report.md](full-model-report.md) | Feasibility result: no eligible complete target model locally; no target full-model measurement |

These are copies of selected technical records. Their original relative links and absolute acquisition paths refer to the local research workspace and may not resolve here. Raw telemetry, model weights, and the full experiment workspace are not uploaded. The selected analyses are saved evidence, not a new measurement or verification of the current remote commit's binaries.

## Minimal before/after reproduction

No model weights are needed. Requires an Apple GPU Mac with working Metal, CMake, and an appropriate Xcode toolchain. Use fresh directories; these commands have been prepared from the source and saved receipts but were not executed as a new acquisition during private PR setup.

```sh
git clone --branch codex/e052-review https://github.com/CamdenKo/llama-e052-review.git
cd llama-e052-review
sw_vers
xcodebuild -version
xcrun --show-sdk-version
git worktree add --detach ../e052-before origin/e052-base
git show HEAD:tests/test-backend-ops.cpp > ../e052-before/tests/test-backend-ops.cpp
cmake -S ../e052-before -B build/e052-before -DCMAKE_BUILD_TYPE=Release -DGGML_METAL=ON -DLLAMA_BUILD_TESTS=ON -DLLAMA_BUILD_EXAMPLES=OFF -DLLAMA_BUILD_SERVER=OFF -DLLAMA_BUILD_TOOLS=OFF -DLLAMA_BUILD_APP=OFF -DLLAMA_BUILD_MTMD=OFF
cmake --build build/e052-before --target test-backend-ops -j 8
cmake -S . -B build/e052-after -DCMAKE_BUILD_TYPE=Release -DGGML_METAL=ON -DLLAMA_BUILD_TESTS=ON -DLLAMA_BUILD_EXAMPLES=OFF -DLLAMA_BUILD_SERVER=OFF -DLLAMA_BUILD_TOOLS=OFF -DLLAMA_BUILD_APP=OFF -DLLAMA_BUILD_MTMD=OFF
cmake --build build/e052-after --target test-backend-ops -j 8
build/e052-before/bin/test-backend-ops test -b MTL0 -o LIGHTNING_INDEXER -j 1 -p 'nh=4,kv=63,nb=1,.*finite_overflow=1'
build/e052-after/bin/test-backend-ops test -b MTL0 -o LIGHTNING_INDEXER -j 1 -p 'nh=4,kv=63,nb=1,.*finite_overflow=1'
build/e052-after/bin/test-backend-ops test -b CPU -o LIGHTNING_INDEXER -j 1 -p 'nh=4,kv=63,nb=1,.*finite_overflow=1'
```

The before worktree intentionally uses candidate tests with the unmodified production kernel. Expect one executed ordinary regression: before Metal FAIL/nonzero, after Metal OK, CPU OK. Check actual case/device/exit status; no Metal device or zero executed cases is not a pass. For all 18 finite-overflow shapes, use `-p finite_overflow=1`.

For the explicit exceptional oracle:

```sh
build/e052-before/bin/test-backend-ops test -b MTL0 -o LIGHTNING_INDEXER -j 1 -p metal_exclusion=1
build/e052-after/bin/test-backend-ops test -b MTL0 -o LIGHTNING_INDEXER -j 1 -p metal_exclusion=1
```

Require `96 cases run`; saved native fails72/96, candidate fails0/96. This oracle encodes the proposed Metal convention; it does not establish that upstream wants that convention.

For ordinary operator coverage and visible/equal-session performance cells:

```sh
build/e052-after/bin/test-backend-ops test -b MTL0 -o LIGHTNING_INDEXER -j 1
build/e052-after/bin/test-backend-ops test -b CPU -o LIGHTNING_INDEXER -j 1
build/e052-before/bin/test-backend-ops perf -b MTL0 -o LIGHTNING_INDEXER -p 'mask_pattern=[12]'
build/e052-after/bin/test-backend-ops perf -b MTL0 -o LIGHTNING_INDEXER -p 'mask_pattern=[12]'
```

Each performance arm selects four cells: B1/B16, visible/equal-session. A single invocation is a screen, not a repeat of the five-block measurement. Use AC power, no competing GPU work, matched warmup, fresh preflight/telemetry, at least five randomized paired process blocks, and saved raw logs before making a new performance claim. Nominal FLOP/s includes work skipped and is not executed throughput.

## Decisions still open

- Whether exact F16 -inf should override nonfinite score propagation in this Metal operator; whether to separate guard-only correctness work from skipping.
- Whether masked-work benefit justifies the fully visible operator regressions in realistic complete-model workloads.
- Eligible target full-model quality/performance evidence, older Apple GPU/toolchain coverage, resolution or upstream acceptance of the retained CI failures, and human code understanding/maintenance readiness.
- A bounded duplicate search is not a uniqueness proof. Nearby GLM/Qwen issues have not been reproduced and fixed by this patch.

The private draft is for technical inspection. Do not reuse its AI-authored prose as an upstream submission.
