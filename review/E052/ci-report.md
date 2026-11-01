# E052 CI continuation result

Routine tooling and CI fixture blockers are resolved. **Both complete self-hosted local CI driver invocations pass. Overall local validation still has hosted leak/autorelease and offline BF16 failures; it is not all green.**

Tested exact base `dde879bc4f954b809892fd0d660515f42536394a` plus unchanged patch SHA256 `b4023881f4207aef8c758196c0ea8580190a7457d22931ae64bbaadc0fbbbe69`. Fresh sources: `src/e052-submission-ci-full-cpu` and `src/e052-submission-ci-full-metal`. No production change, new waiver, test exclusion, commit, push, publication, or system setting change.

| Check | Status | Evidence/result |
|---|---|---|
| Required style / EditorConfig3.0.3 | PASSED | Fresh full-driver source; whitespace checked. |
| Full unchanged CPU ci/run.sh | PASSED | All11 stages; Debug17, Release17 and model-dependent suites pass. Default operator command skips CPU; supplement below. |
| Full unchanged Metal ci/run.sh | PASSED | All11 stages; Debug18, Release18 and model-dependent suites pass; embedded-source/runtime-compiled path. |
| Explicit CPU operators, exact BLAS-OFF build | PASSED | 23579 supported;3239 unsupported; no operation/parameter filter. |
| Full Metal operators | PASSED | 18770 standard +96 direct;8048 unsupported. CPU backend skipped as designed. |
| Logical Metal device sweeps | PASSED | [405, 552, 698, 844] checks; one physical GPU. |
| Model CI: BGE/reranker/split/quantize/Qwen/context | PASSED | Full driver functions, not substitute fixtures or staged-only coverage. |
| Qwen PPL gates | PASSED | 12 precisions executed;11 enforced <=20. Existing q2_k exemption retained, with observed values in JSON. |
| Hosted Apple required CTest selector | PASSED | All18tests pass after supplying fixture; local --no-cache compiler lookup workaround retained. |
| Hosted leak check, candidate | FAILED | [{'count': 1, 'bytes': 80}] |
| Hosted leak check, clean native | FAILED | [{'count': 1, 'bytes': 80}] |
| Hosted autorelease check, candidate/native | FAILED | See exact warning classes/statuses in JSON; no Device exception broadened. |
| Offline BF16 indexer supplement, candidate/native | FAILED | Both prior checks SIGABRT(-6), missing BF16 function in offline library; no fix or exclusion. |
| Other platforms / older GPU hardware | UNSUPPORTED HERE | Not claimed covered by this Mac. |
| Separate eligible full-model validation | NOT RUN HERE | CI models do not exercise the target full-model indexer; separate owner/prerequisites remain. |

The full-driver stage ledger, CTest counts, per-precision PPL values, exact statuses and warning classes are in [result-matrix.json](result-matrix.json). Native leak findings and concrete separate follow-ups are in [native-leak-investigation.md](native-leak-investigation.md). The offline BF16 finding remains in [offline-bf16-investigation.md](../offline-bf16-investigation.md). Nothing was removed from a failing selection to get a pass.

CI fixtures were explicitly authorized within4GiB downloads/20GiB scratch. Metadata/revisions, LFS checksums and acquired payload sizes are in [download-plan.json](download-plan.json), [acquisition-receipt.json](acquisition-receipt.json) and [gemma-receipt.json](gemma-receipt.json). The complete Qwen cache reuse was verified against the real server (zero response body). A10-second scratch guard did not trigger. These learned CI models do not establish complete-model quality/performance for E052: none is the eligible target indexer model.

Machine/configuration: M5 Max40GPU/48GB, macOS27.2/26B123, Xcode/SDK27.2, clang21, Metal32023.922. Full-driver CI venv Python3.12.16/Torch2.11.0/NumPy2.2.6; private GNU wget1.25.0/OpenSSL3.5.4, GitLFS3.7.1, GNU nproc9.5. AC/high-power and full preflight warnings retained. Tests are correctness/sanity checks; no performance/energy claim from their timings.

Correction to earlier packet: its manual builds used BLAS ON while ci/run.sh sets the BLAS backend OFF by default. They were not exact CI-configuration equivalents. Old reports/raw logs remain preserved in [prior-stage-reports](prior-stage-reports/); the new unchanged-driver results supersede their readiness status. [Configuration comparison](configuration-equivalence-correction.json). The initial Debug directory collision is also retained; no candidate failure was erased.

Build identity limits: [build-receipts](build-receipts/) records future Metal build binary/library/metallib/cache hashes before the driver deletes intermediate release builds. Early CPU intermediate executable hashes were not captured; their complete source/compiler/configuration/build commands and logs remain, and final CPU binaries are hashed. Do not claim exact intermediate CPU binary identity. See [build-receipt-note.md](build-receipt-note.md).

Raw full-driver logs: [CPU](../continuation-full-cpu.log), [Metal](../continuation-full-metal.log); internal complete stage logs live in [official-cpu](official-cpu/) and [official-metal](official-metal/). All exact command arguments/environment/statuses are appended to [commands.jsonl](../commands.jsonl); [commands-transcript.sh](commands-transcript.sh) is the continuation transcript. [Reproduction commands](REPRODUCE.md).

Remaining work that cannot be cleared by fixture/tool setup: the source-level hosted failures above, eligible target full-model evidence, human review/ownership and other contribution prerequisites. Proposed production fixes are separate and unapplied, following the original instruction not to modify production code to obtain a pass.
