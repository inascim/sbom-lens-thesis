# Thesis completion plan

Agreed order of work (2026-09-30). Decisions behind each step are in `docs/adr/`; vocabulary is in `CONTEXT.md`.

1. **Code changes in `../sbom-lens`**
   - [x] Per-severity overlay classes (`.vuln-critical`, `.vuln-high`, `.vuln-medium`, `.vuln-low`): red / orange / yellow / green-yellow, matching Figure 5.1. Then resolve the REVIEW comment in Section 5.5.
   - [x] Root-based dominance in `backend/routers/impact.py`, using a virtual root when there are several (ADR 0002). Then resolve the REVIEW comment in Section 5.5.3.
   - [x] Fixes from `docs/research/backend-metrics.md`: spectral eigensolver sign (`eigsh`, "LA"), epidemic threshold 1/λ₁ in the response, seeded small-world sampling, `topDependedOn` by in-degree, self-loops stripped at graph build, graph-analysis caches cleared on edit/delete, version conflicts keyed by (group, name), power law by MLE with x_min (`powerlaw`), bow-tie reported only when the core has more than one node.
   - [ ] Verify in the running app (frontend untested: no Node on the dev machine), then commit in `sbom-lens`.
   - [ ] Retake Figure 5.1 with the four severity colours.
2. **`sbom-lens-eval` pipeline** (ADR 0003, ADR 0004)
   - [x] `sbom-lens` prep: severity from the CVSS v3 vector, else the GHSA label, else UNKNOWN; remove the unused NVD/GHSA code; correct Ch2/Ch5 to OSV only.
   - [ ] New local repo `../sbom-lens-eval`: Python orchestrator driving the running backend over HTTP, plus a Node script that imports the real `vulnerabilityAPI.js` from a pinned `sbom-lens` checkout; cdxgen pinned.
   - [ ] Candidates: GitHub search (JavaScript/TypeScript, by stars) → latest release tag ≤ 2024-03-31 → committed lockfile, not a workspaces monorepo → cdxgen `--required-only` → ≥1 known vulnerability, ≤1000 components. Stop at ~90 accepted; every rejection logged in `skipped.csv`.
   - [ ] Outputs: one JSON per SBOM (all metrics, `/impact` for each vulnerable node, timings), a combined CSV, the query date and the pinned commits.
   - [ ] Push to GitHub; Zenodo snapshot at submission.
3. **Thesis writing** (can overlap with step 2)
   - [x] Ch6: component, cluster and whole-graph scales.
   - [ ] Ch3: gap analysis (3.4) first, then 3.2 and 3.3 working back from it.
   - [ ] Ch8: everything except the summary of findings.
   - [ ] Ch7: dataset and pipeline, method and RQs, threats to validity.
4. **After data collection**
   - [ ] Ch7: RQ1–RQ3 results and the performance paragraph.
   - [ ] Ch8: summary of findings.
