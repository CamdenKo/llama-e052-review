# Current E052 local CI result

Routine tooling/fixture blockers are resolved. **Both full CPU and Metal CI drivers pass; overall validation is not all green because hosted failures remain.** Candidate/base/patch are unchanged. GPU/test processes are terminal and ownership released.

| Check | Result |
|---|---|
| Unchanged full CPU `ci/run.sh` | Passed all11 stages |
| Unchanged full Metal `ci/run.sh` | Passed all11 stages |
| CPU operators, explicit unfiltered operation set | 23,579 passed;3,239 unsupported |
| Metal operators | 18,770 standard +96 direct passed;8,048 unsupported |
| Logical-device sweeps, BLAS OFF | 405/552/698/844 checks passed |
| Hosted Apple required CTest selector | 18/18 passed |
| Code-style / EditorConfig3.0.3 / whitespace | Passed |
| Hosted leak + autorelease checks | Failed identically on candidate and clean native:80-byte Metal library leak; two missing-pool warning classes |
| Offline BF16 indexer supplement | Failed on candidate and native:missing function / SIGABRT |

No new test exclusions, tolerance changes, source fixes, system setting changes, commits, pushes or publication. The repository's pre-existing q2_k PPL threshold exemption is explicitly retained with its measured values. No target full-model E052 quality/performance claim is added.

[Full matrix and logs](REPORT.md) · [Exact reproduction commands](REPRODUCE.md) · [Native-control findings and separate proposed fixes](native-leak-investigation.md) · [Preservation/configuration/input/process audit](final-audit.json)

Earlier manual builds used BLAS ON and were not exact CI equivalents; their original evidence is preserved and the label corrected. These full-driver results use the repository's BLAS-OFF configuration. Early CPU intermediate executable hashes were not retained before the unchanged driver deleted/rebuilt them; full recipes/logs and final CPU hashes are saved. All subsequent Metal build hashes were captured before deletion.
