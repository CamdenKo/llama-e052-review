# E052 full-model validation feasibility result

2026-11-01. **Not executed: no eligible complete local model.** This is a prerequisite result, not negative quality/performance evidence. No download, GPU dispatch, tuning, production change, commit or publication. CI retains exclusive GPU ownership; coordination acknowledged in its chat. No independent reviewer was used.

## Verified identity

Candidate HEAD/base dde879bc4f954b809892fd0d660515f42536394a. Actual uncommitted binary-format Git diff equals patches/E052-metal-only-comment-cleanup.diff byte-for-byte, SHA256 b4023881f4207aef8c758196c0ea8580190a7457d22931ae64bbaadc0fbbbe69. Two changed files, 218 insertions/9 deletions; diff whitespace check passes. Source/policy hashes and exact diff are in inventory.json. The historical native A checkout has a modified test file; do not call it a pristine source control. Existing builds are not newly identity-certified full-model executables.

Read HANDOFF.md, metal-only-report.md, repair-report.md, simplification/report.md, submission-source-refresh.md and candidate AGENTS.md/CONTRIBUTING.md. Policies require full CI, model PPL/performance, two backend operator checks, failing-before regression and human review/ownership before publication. This packet is internal validation planning, not submission prose. Prior source-refresh evidence is reused without a new novelty claim.

## Inventory and coverage

GGUF headers/tensor names inspected without loading weights onto GPU:

| Local artifact | Bytes | Architecture / layers | Eligibility |
|---|---:|---|---|
| GLM-4.7-Flash Q4_K_M | 18,312,339,808 | deepseek2 / 47 | Complete-model candidate, zero indexer tensors; deepseek2.cpp has no lightning_indexer call |
| Qwen3-30B-A3B Q4_K_M | 18,556,686,912 | qwen3moe / 48 | Complete-model candidate, zero indexer tensors; qwen3moe.cpp has no lightning_indexer call |
| Four GLM5 pair GGUF variants | 8,387,056,192 to 8,579,994,176 | glm5-next / 2 | Seven indexer tensors each, but synthetic embedding/output and incomplete layers; excluded |

MLX weight directories contain versions of the same unrelated complete GLM4.7/Qwen3 models. GLM5 safetensors are selected range downloads, not all 62 original shards. Known Hugging Face/Ollama/LM Studio cache roots are absent (cache-inventory.json). Inventory is repository plus these standard caches, not proof that no model exists anywhere on the filesystem.

Nine model implementations have static ggml_lightning_indexer calls; full list/line numbers saved in inventory.json. GLM5-next.cpp:958 is the relevant slice caller. Changed Metal kernel scans exact half negative-infinity masks, skips fully hidden key/query tiles, skips hidden query rows inside live tiles, and guards hidden entries in otherwise live query tiles. No eligible complete-model graph/pipeline/mask trace was acquired; full-model path coverage is **unestablished**, not inferred from operator tests or model naming.

Tools found: llama-perplexity/llama-bench in build/llama-e052-upstream, batched-bench/bench in build/llama. Native/candidate A/C builds currently contain libraries and test-backend-ops, not the requested three tool executables. Their mere presence elsewhere does not bind them to the current patch. No WikiText test corpus or dedicated data/datasets directory found; get-wikitext-2.sh is a downloader script, not data. Raw search outputs and tool paths are saved.

## Resources and execution boundary

sw_vers: macOS27.2 build26B123. Read-only preflight: M5 Max40 GPU cores,48GB, 0 FAIL/16 WARN; complete block retained in preflight.txt and PREFLIGHT_LOG.md. Physical RAM51,539,607,552bytes. Disk preflight1092GiB free. AC confirmed. Memory snapshot includes compressed memory and historical paging: available physical RAM is not equal to free capacity for an entire model. No full-model resident fit was measured. No claim that every eligible model is too large: **none is available locally**. Do not download a large model to discover whether it fits.

Process audit found active E052 CI compilation. No GPU work was started; CI was explicitly told this task will launch none. Thus there are no measured repeats, warmups, timing windows or clocks/power samples to report. Read-only preflight does not qualify a future session. No failure or regression was discarded; existing visible-operator regressions and numerical limitations remain in the earlier reports.

## Smallest credible next execution and feasible alternatives

protocol.md specifies the bounded full-model experiment, before acquisition: matched clean native/current candidate, exact artifact/data manifests, complete-model path trace, eight PPL chunks and five native/candidate quality repetitions, visible and independent multi-sequence workloads, five paired timing processes, fixed numerical/3% visible regression criteria and telemetry gates. The prospective configuration must be finalized against an approved eligible artifact and installed CLI help before acquisition; templates have not been run.

First prerequisite: supply or authorize acquisition of a complete supported indexer model with original learned embedding/head/all layers and documented quantization. Local eligibility budget is <=36GiB total working memory and <=100GiB additional disk, with no swap reliance. Required corpus is a pinned WikiText-2 test file (or an approved existing equivalent). Source/data download permission is absent; nothing downloaded.

Concrete alternative if the complete GLM5-next artifact cannot fit: run this exact two-source comparison on an authorized larger-memory Apple GPU Mac already holding a complete GLM5-next quantization. Reserve at least artifact size plus KV/scratch plus25% headroom, confirm Metal full offload, and use the same path gates before accepting performance. This is technically executable with existing llama.cpp tools, but host/artifact access and actual capacity remain prerequisites, not verified resources. Do not project M5 Max timings onto that host. If such access is unavailable, the feasible local fallback is the already-completed exact operator oracle plus real-weight two-layer replay, explicitly labeled limited evidence; another slice campaign would not close the requested gap. The two complete local unrelated models can check generic runtime smoke only and are not worth GPU time for this claim.

## Exactly what submission claims are supported

This new packet supports source identity and the finding that suitable local full-model prerequisites are missing. It adds **no** model-quality, end-to-end speedup, full-model non-regression or energy claim.

Previously saved evidence continues to support bounded Metal operator correctness/regression sensitivity and common-state real-weight two-layer replay results, with original binary/source identities and limitations. The current production kernel hash is recorded here; comment/test changes do not manufacture new full-model evidence. Historical visible operator slowdowns remain relevant; full-model impact is unknown. No assertion of preserved complete-model PPL/KL, improved user-facing tokens/s, full CI clearance, upstream readiness, older-GPU compatibility, native exceptional equivalence, Ultra/fleet transfer or production suitability is supported by this task.
