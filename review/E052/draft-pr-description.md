## Overview

Private review of E052: avoid masked work in Metal's `LIGHTNING_INDEXER` and make exact F16 negative-infinity masks produce exact F32 negative infinity, including when score arithmetic is nonfinite.

This is an experimental work sample for technical feedback, not an upstream submission or a claim of production readiness. I am still learning the implementation and completing my own review. Codex wrote this description and commit message at my request; the patch and much of the experimental tooling were developed with AI assistance.

The code diff is limited to the Metal kernel and `test-backend-ops`. Base: upstream `dde879bc4f954b809892fd0d660515f42536394a`. Patch SHA256: `b4023881f4207aef8c758196c0ea8580190a7457d22931ae64bbaadc0fbbbe69`. Upstream checked November 1, 2026: `785322d`, one dependency-only commit later. This PR preserves the measured base rather than claiming a latest-base rerun.

### Behavior and semantic decision

The original kernel stages keys to half precision, computes scores, then adds the mask. A finite F32 key such as 100000 can overflow during half staging, leading to `+inf + -inf = NaN` at an excluded entry. The candidate:

- Writes `-inf` and returns before key staging when every query in a threadgroup has a fully hidden 64-key tile.
- Skips score work for a fully hidden query tile within an otherwise active group.
- Forces each exact `0xfc00` F16 mask entry to output `-inf`, including hidden entries inside partly visible query tiles. Those individual scores may still be computed.

This changes exceptional propagation. Other backends and decomposed score-plus-mask graphs are unchanged; visible-key overflow is not repaired. A finite bias, NaN mask, or positive-infinity mask does not trigger exclusion. The synthetic reproduction does not establish natural model failure incidence or a security exploit.

The main review question is whether this Metal-local exclusion convention is appropriate, and whether the guard and skipping should remain one change or the guard should be considered separately first. Masked-work elision has [Metal FlashAttention precedent](https://github.com/ggml-org/llama.cpp/pull/13493), but that does not settle indexer semantics. A bounded duplicate search found no exact matching change; no claim is made that nearby GLM/Qwen issues are fixed.

### Measured benefit and regressions

M5 Max, 40 GPU cores, 48 GB; macOS 27.2/26B123, Xcode 27.2/27B52, SDK 27.2; AC/high-power. Operator timing uses D128/H32/KV32768, F32 inputs, F16 mask, F32 output, half staging, and five randomized paired process blocks.

| Workload | Result | Scope |
| --- | --- | --- |
| B16 equal-session operator | 6.7171x paired speedup | Idealized disjoint equal histories; not tokens/s |
| Fully visible operator, B1 / B16 | 3.50% / 1.78% latency regressions | Cost must be weighed against masked-work benefit |
| B16 two-layer real-weight replay, context32768 | 11.15% latency reduction; conditional 95% interval [9.65%, 12.64%] | Common-state GLM5-next replay with synthetic embedding/output |
| B1 two-layer replay | Inconclusive | No demonstrated benefit |

Guard-only and guard-plus-skip controls separate the skipping revision's benefit from the exclusion guard. Historical performance remains tied to its original binary/source identities; the final cleanup changes comments/tests, not production executable logic. These are not full-model quality, serving-throughput, energy, or Ultra/fleet results.

## Additional information

[Evidence index and reproduction commands](https://github.com/CamdenKo/llama-e052-review/blob/e052-evidence/review/E052/README.md) are on a separate branch to keep this diff to two files. The packet includes the source patch, operator/replay analyses, regression receipts, prior-art notes, and CI/full-model reports.

Validation available:

- Finite-input overflow regression fails before the kernel change and passes after; unchanged CPU passes. The direct exceptional oracle executes 96 Metal cases; native fails 72, candidate passes all 96. Deliberately broken tile/query/stream variants are detected.
- Both full local CPU and Metal `ci/run.sh` drivers pass all 11 stages. Explicit CPU operators pass 23,579 supported checks. Metal passes 18,770 standard plus 96 direct checks. Unsupported cases remain explicitly recorded.
- Overall validation is not all green: hosted leak/autorelease checks and the offline BF16 indexer supplement fail on both candidate and clean identical-base native. The reports retain these failures and configuration workarounds; no production fixes or new waivers are bundled here.
- No eligible complete target model was available for separate full-model perplexity/performance validation. CI models do not establish execution of the changed indexer in a complete target model. Older Apple GPUs/toolchains, broader backend semantics, and full human line-by-line review remain outstanding.

Feedback sought: synchronization and tail/index correctness, exceptional-mask semantics, test sensitivity and maintenance cost, and whether the measured tradeoff justifies the visible-work overhead. This draft is for inspection and discussion within this private repository.

## Requirements

- [ ] I have read and agree with the [contributing guidelines](https://github.com/ggml-org/llama.cpp/blob/master/CONTRIBUTING.md). Reading is confirmed; upstream contribution readiness and full independent code understanding are not certified by this private draft.
- AI usage disclosure: YES. AI assisted implementation, experiments, tests, analysis, and review preparation. Codex authored this private PR description and commit message with explicit permission. Any eventual upstream submission requires a separate human-authored description and human-owned review, understanding, and maintenance.
