# Thesis completion plan

Agreed order of work (2026-09-30). Decisions behind each step are in `docs/adr/`; vocabulary is in `CONTEXT.md`.

1. **Code changes in `../sbom-lens`**
   - [ ] Per-severity overlay classes (`.vuln-critical`, `.vuln-high`, `.vuln-medium`, `.vuln-low`): red / orange / yellow / green-yellow, matching Figure 5.1. Then resolve the REVIEW comment in Section 5.5.
   - [ ] Root-based dominance in `backend/routers/impact.py`, using a virtual root when there are several (ADR 0002). Then resolve the REVIEW comment in Section 5.5.3.
2. **`sbom-lens-eval` pipeline** (ADR 0003)
   - [ ] New repo: committed project list (~30 each for npm, PyPI and Maven, at 2–3-year-old release tags), cdxgen SBOM generation, and filters (≥1 known vulnerability, ≤1000 components).
   - [ ] Vulnerability lookup through `../sbom-lens/src/utils/vulnerabilityAPI.js` under Node (NVD + OSV + GHSA; needs an NVD API key and a GitHub token), with results posted to the backend.
   - [ ] Dump every metric plus timings to JSON/CSV; record the vulnerability-query date and the pinned `sbom-lens` commit.
   - [ ] Push to GitHub; Zenodo snapshot at submission.
3. **Thesis writing** (can overlap with step 2)
   - [ ] Ch6: component, cluster and whole-graph scales (see the chapter's TODO comment).
   - [ ] Ch3: gap analysis (3.4) first, then 3.2 and 3.3 working back from it.
   - [ ] Ch8: everything except the summary of findings.
   - [ ] Ch7: dataset and pipeline, method and RQs, threats to validity.
4. **After data collection**
   - [ ] Ch7: RQ1–RQ3 results and the performance paragraph.
   - [ ] Ch8: summary of findings.
