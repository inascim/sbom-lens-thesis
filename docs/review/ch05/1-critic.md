# Chapter 5 — Critic

Code references marked `@28113e7` come from `git -C /home/inasc/projects/sbom-lens show 28113e7:<path>`. The evaluation pinned `dfc8e23` (`sbom-lens-eval/config.py:18`), which is an ancestor of `28113e7`. Between the two commits none of `vulnerabilityAPI.js`, `impact.py`, `vuln_store.py` or `vulnerabilities.py` changed (`git diff --stat dfc8e23 28113e7` touches only `sbom_store.py`).

## C1. The "five-state remediation lifecycle" does not exist: the system has three states, and one of them is named REMEDIATED, not RESOLVED
- Type: error
- Origin: pre-existing (still true at HEAD)
- Severity: high
- Thesis: Chapters/05-Vulnerability-Analysis.tex:53 — "one of NOT\_STARTED, IN\_PROGRESS, RESOLVED, WONT\_FIX, or FALSE\_POSITIVE"; :189 — "The status field supports a five-state remediation lifecycle ..."; :4 — "the five-state status lifecycle exposed through the REST API"
- Evidence: `src/hooks/useVulnerabilities.js:13 @28113e7` — `status: "NOT_STARTED" | "IN_PROGRESS" | "REMEDIATED"`; `src/layouts/sbom-vulnerabilities/components/VulnerabilityTable.js:33 @28113e7` — `statusOptions = ["NOT_STARTED", "IN_PROGRESS", "REMEDIATED"]`; `VulnerabilityForm.js:151-152`, `VulnerabilityChart.js:40`, `sbom-vulnerabilities/index.js:79` (`STATUS_ORDER = {NOT_STARTED, IN_PROGRESS, REMEDIATED}`), `dashboard/index.js:68` (unresolved = `status !== "REMEDIATED"`). `git grep` finds no occurrence of `RESOLVED`, `WONT_FIX` or `FALSE_POSITIVE` anywhere in `src/` or `backend/` at either commit. The backend (`backend/routers/vulnerabilities.py:15,26 @28113e7`, `body: dict`) does not validate status at all.
- Why it matters: A whole paragraph (:189) describes triage states (WONT\_FIX, FALSE\_POSITIVE) that the tool does not offer. An examiner who runs the tool will see three states. The fix is either to describe the three real states or to implement the other two.

## C2. The timeline is not a server-side, append-only audit trail: the client builds it and sends it, and any PUT can overwrite any field
- Type: error
- Origin: pre-existing (unchanged at HEAD)
- Severity: high
- Thesis: Chapters/05-Vulnerability-Analysis.tex:189 — "Each status transition is appended to the timeline list with a server-side timestamp, creating an append-only audit trail"; :59 — "append-only list of status-transition objects"; :184 — "the analyst may update status, notes, or patchAvailable"
- Evidence: `src/hooks/useVulnerabilities.js:185-201 @28113e7`. The frontend builds `timelineEntry` with `date: new Date().toISOString()` (a client timestamp) and `user: "Current User"`, then PUTs `timeline: [...(vuln.timeline || []), timelineEntry]`, i.e. the whole array. `backend/store/vuln_store.py:50 @28113e7` — `record = {**self.vulns[vuln_id], **updates, "id": vuln_id}`. This is a blind merge, so whatever `timeline` the client sends replaces the stored one, and so does any other field (`cveId`, `purl`, `severity`, ...). `vuln_store.py:40` also accepts a client-supplied `timeline` on POST. Only `lastUpdated` is set by the server (:51).
- Why it matters: "append-only" and "server-side timestamp" are integrity guarantees that the code does not provide. Any client can rewrite or erase the history, and the user field is a hard-coded placeholder. The PUT is not restricted to status/notes/patchAvailable either: it is an unrestricted partial update.

## C3. Vulnerability fetching runs only after a successful new upload, not "whenever an SBOM is selected"; the repeated-load / page-refresh / 409 story does not happen
- Type: error
- Origin: pre-existing (HEAD has the same single call site)
- Severity: high
- Thesis: Chapters/05-Vulnerability-Analysis.tex:32 — "invoked automatically ... whenever a new SBOM is selected"; :169 — "triggered automatically every time an SBOM is selected"; :173 — "When the same SBOM is loaded a second time (or when the page is refreshed), the hook runs again and resubmits every PURL ... 409"
- Evidence: The only call site of `fetchVulnsForSBOM` is `src/layouts/dashboard/index.js:85-87 @28113e7` (`handleUploaded`), wired as `onUploaded` of `UploadBatchDialog`. `UploadBatchDialog/index.js:51-58 @28113e7` calls `onUploaded` only when `!result.isDuplicate`, and the backend rejects re-uploads of identical content by SHA-256 (`backend/routers/sboms.py:31-42 @28113e7`). Selecting an SBOM in the Graph Explorer or reloading the page never triggers a fetch (`grep -rn "fetchVulnsForSBOM(" src` gives only `dashboard/index.js:86`). In practice the 409 path is hit when two *different* SBOMs share a component/CVE pair, which the thesis does not mention.
- Why it matters: The section describes a workflow that does not exist. It also hides a real limitation: an SBOM's vulnerabilities are fetched once, at upload, and never refreshed, so advisories published later never reach an SBOM already in the library.

## C4. The impact endpoint does not return HITS scores "for each node in the blast radius"; it returns two scalars for the queried node (and Chapter 6 says so)
- Type: error / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/05-Vulnerability-Analysis.tex:217 — "returns the blast radius, dependency paths, dominated nodes, and per-node HITS scores"; :225 — "hitsHub and hitsAuthority scores for each node in the blast radius"
- Evidence: `backend/routers/impact.py:71-84 @28113e7` — `hits_hub = hubs.get(node_id, 0.0)`, returned as `"hitsHub": round(hits_hub, 6)` (a scalar for `node_id` only). Chapters/06-Network-Science.tex:32 says the opposite of Ch5: "the impact endpoint ... also returns the two scores of the analysed component".
- Why it matters: This is a factual error about the API, and it contradicts Chapter 6. Remediation prioritisation over the blast radius using per-node HITS (:225) is not something the endpoint supports.

## C5. Blast radius and dominated set are not rendered "simultaneously within the same interaction"; they are separate toggles, and re-targeting a node updates only one of them
- Type: error
- Origin: pre-existing (same logic at HEAD, `src/layouts/sbom-graph/index.js:564-590`)
- Severity: medium
- Thesis: Chapters/05-Vulnerability-Analysis.tex:235 — "Because the dominator-tree overlay is rendered simultaneously with the blast radius within the same interaction, the analyst can read both overlays together"; :195 — "the blast-radius and dominator-tree overlays from a node's context menu"
- Evidence: `src/layouts/sbom-graph/index.js @28113e7`. Blast Radius (:966) and Dominator Tree (:1010) are separate items in the *Analysis menu*, handled by `handleActivateBlastRadius` and `handleActivateDominator` (:585). The context menu (`handleAnalyzeImpact`, :502) only stores the node, or re-queries it. When both modes are active, `handleAnalyzeImpact` uses `if (activeModes.has("blast-radius")) {...} else if (activeModes.has("dominator")) {...}`, so after right-clicking a new node only the blast radius is refreshed. The cyan dominator overlay then still shows the *previous* node's dominated set.
- Why it matters: The "dual encoding ... in a single inspection" claim is the stated value of the dominator overlay. In the code, the analyst must turn it on separately, and with both on the two overlays can describe different nodes, which is misleading. The menu location is also misdescribed (Chapter 4 :189 repeats "context menu ... fetches blast-radius and dominator-tree data").

## C6. `patchAvailable` was always `false` for OSV records in the evaluated system; the schema also no longer matches the current store
- Type: error (pre-existing) + drift
- Origin: pre-existing for patchAvailable semantics; drift for the schema fields
- Severity: medium
- Thesis: Chapters/05-Vulnerability-Analysis.tex:54 — "patchAvailable --- boolean indicating whether a patched version is known to exist"; :46-60 schema list
- Evidence: `src/utils/vulnerabilityAPI.js:158 @28113e7` — `patchAvailable: false` hard-coded for every OSV record, so the field said nothing about patches. At HEAD (`b0da73b`), `vulnerabilityAPI.js` derives `patchAvailable: fixedVersions.length > 0` and the store (`backend/store/vuln_store.py:36-38` at HEAD) adds `summary` and `fixedVersions`, neither of which is in the thesis schema. At `28113e7` the fetcher sent `summary` but `VulnStore.add()` silently dropped it.
- Why it matters: For the evaluated system, the field's description was false (no patched version was ever detected). For the current system, the schema list is incomplete. The thesis should either describe the evaluated behaviour honestly or be updated to the current one (fixed versions from OSV `affected[].ranges[].events[].fixed`).

## C7. Deduplication collapses distinct advisories that share a CVE alias, a case far more common than the one the thesis describes; and Chapter 7's finding counts include 125 pairs the store would have rejected
- Type: inconsistency / examiner-question
- Origin: n/a (behaviour + data)
- Severity: medium
- Thesis: Chapters/05-Vulnerability-Analysis.tex:77 — "preventing duplicate OSV records in the rare case where the same OSV entry is fetched more than once"; :38 — "each unique vulnerability-component pair is stored exactly once"; Chapters/07-Evaluation.tex:20 — "the evaluation uses exactly the lookup and severity derivation the tool uses"; :51 — "8{,}868 findings"
- Evidence: Within one fetch the same OSV ID cannot recur (`uniqueOsvIds`, `vulnerabilityAPI.js:118 @28113e7`), so the case described at :77 cannot happen. What does happen is two *different* OSV advisories mapping to the same first CVE alias. In the evaluation data (`sbom-lens-eval/data/results/*.json`) there are 125 duplicate (cveId, purl) pairs among the 8,868 records, e.g. `CVE-2025-13465` / `pkg:npm/lodash@4.17.21` with two different GHSA summaries. `pipeline/vulns.mjs` calls `fetchVulnerabilitiesForSBOM` directly, bypassing `VulnStore`, so these were never deduplicated. Inside the tool, `VulnStore.add()` would keep only the first (the second advisory's text is lost). In my recomputation, worst bands per component are unchanged (0 components change band), so RQ2/RQ3 are not affected, but the 8,868 and the per-band totals are not the counts the tool would store (8,743).
- Why it matters: Chapter 5 promises once-per-pair storage, and Chapter 7 claims to use exactly the tool's pipeline while reporting counts that include duplicates. The dedup key also silently merges distinct advisories, which deserves to be stated.

## C8. "Precisely the packages that an attacker can exploit" overclaims, and contradicts Chapter 7's own threat to validity
- Type: unsupported / inconsistency
- Origin: n/a
- Severity: medium
- Thesis: Chapters/05-Vulnerability-Analysis.tex:219 — "they are precisely the packages that an attacker can exploit by compromising the vulnerable node"; :233 — "if it were removed or compromised, all the dominated components would lose their connection"
- Evidence: `nx.ancestors` (`impact.py:50 @28113e7`) is pure graph reachability, with no reachability analysis of the vulnerable code. Chapters/07-Evaluation.tex:157 states that a dependent "may never execute the vulnerable code ... so blast radius and reach over-approximate actual exposure". On :233, compromising a node does not remove it from the graph, so dominated components do not "lose their connection"; the dominator argument applies only to removal or replacement (as the glossary, CONTEXT.md:25, correctly says).
- Why it matters: "Precisely" turns an over-approximation into an exact claim, and one chapter later the thesis concedes the opposite. Saying "may be affected", plus "removed" only, would make the chapters consistent.

## C9. Blast radius is defined three ways (with the node, without it, and Chapter 7 mixes both), which inflates the reported vulnerability reach from about 12 % to 19 %
- Type: inconsistency
- Origin: n/a
- Severity: medium
- Thesis: Chapters/05-Vulnerability-Analysis.tex:219 — blast radius = `list(nx.ancestors(G, node_id)) + [node_id]` (includes the node); CONTEXT.md:21 — "The set of components that depend on a given component" (excludes it); Chapters/07-Evaluation.tex:59 uses the size "excluding the component itself" (`analyse.py:94`, `len(blastRadius) - 1`), but reach (:61, :132) is computed on the *including* version (`analyse.py:129`, `set(v["blastRadius"])`).
- Evidence: My recomputation from `sbom-lens-eval/data/results` + `data/sboms` gives a median reach of 0.186 (range 0.095–0.349) as computed, which matches Ch7's "19%, 9% to 35%". Under the glossary definition (dependents only), the median is 0.118 (range 0.040–0.288). The vulnerable components themselves alone make up a median 7.9 % of non-root nodes. Ch7:132 says reach is "as opposed to how many components are themselves vulnerable", yet the vulnerable components are counted in it.
- Why it matters: The headline RQ3 number depends on which definition is used, and the chapter and the glossary disagree. Chapter 5 should fix one definition (with or without the node) and Chapter 7 should use it consistently.

## C10. The OSV batch query ignores pagination; truncated results would be silent
- Type: examiner-question
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/05-Vulnerability-Analysis.tex:26 — "the entire component list is submitted in one round-trip, and the response returns the matching OSV vulnerability IDs for each PURL position"
- Evidence: OSV's querybatch documentation (google.github.io/osv.dev/post-v1-querybatch/) states that results are paginated with `next_page_token` when one query returns more than 1,000 vulnerabilities or the whole batch more than 3,000. `vulnerabilityAPI.js:93-113 @28113e7` reads only `results[].vulns` and never follows `next_page_token`. Any failure (`!batchResponse.ok`, exception) returns `[]` (:100-103) with no log, so it looks the same as "no vulnerabilities". Failed detail fetches are dropped silently too (:122, :138). The largest evaluation SBOM has 387 findings, so the evaluation is not affected.
- Why it matters: This is a correctness limit of the "single round-trip" design, and silent failure means an empty result cannot be told apart from a fetch error. It should be stated as a limitation.

## C11. Drawing UNKNOWN with the LOW colour contradicts the chapter's own principle that missing severity must not look like low risk
- Type: examiner-question
- Origin: pre-existing (unchanged at HEAD, `src/layouts/sbom-graph/index.js:247-249`)
- Severity: low
- Thesis: Chapters/05-Vulnerability-Analysis.tex:108 — "returns UNKNOWN rather than defaulting to LOW, so that missing severity data is not mistaken for low risk. On the graph, UNKNOWN nodes are drawn with the LOW colour"
- Evidence: `src/layouts/sbom-graph/index.js:229 @28113e7` maps UNKNOWN to `vuln-low`. The rest of the UI already has a grey UNKNOWN colour (`src/utils/severity.js`, `UNKNOWN: "#9CA3AF"`), and CONTEXT.md:44 says UNKNOWN "is not LOW".
- Why it matters: The visual layer, which is the one place an analyst actually sees, does exactly what the paragraph says must be avoided, and no justification is given. (No evaluation finding was UNKNOWN, so Ch7 is unaffected.)

## Checked and holds
- §5.3 CVSS formula and constants (`vulnerabilityAPI.js:14-52 @28113e7`): AV/AC/UI/CIA weights, scope-dependent PR (0.68/0.50 for S:C vs 0.62/0.27), 6.42 / 7.52 / 0.029 / 3.25 / 0.02^15 / 8.22 / 1.08, min(...,10) all match FIRST CVSS v3.1.
- Log4Shell worked example (:123-147), recomputed: ISS = 0.914816; Impact = 6.6613 − 0.6136 = 6.0477; Exploitability = 3.8870; 1.08 × 9.9347 = 10.73 → 10.0. All intermediate values in the text are correct.
- Roundup claim (:155): I enumerated all 2,592 CVSS v3.1 base vectors and compared `Math.ceil(x*10)/10` with the spec's integer-arithmetic Roundup (Appendix A). There were 0 differences, so the claim "implements it exactly" holds empirically over the whole base-score domain.
- Severity bands (:95-108) and `parseCVSSSeverity` (:157) match the code (`vulnerabilityAPI.js:61-78`), including 0.0 → LOW and missing/unparseable → UNKNOWN.
- Severity derivation order CVSS_V3 → GHSA `database_specific.severity` (MODERATE→MEDIUM) → UNKNOWN; v4 not scored (:30). Matches `vulnerabilityAPI.js:145-148, 170-181`.
- CVE alias resolution / OSV-ID fallback (:28) and the dedup snippet and 409 behaviour (:68-75, :183). Match `vuln_store.py:17-26` and `vulnerabilities.py:14-22`; 201/409 codes correct. The fetcher is sequential (`useVulnerabilityFetcher.js:42-44`) and logs with `console.warn` (:47).
- REST endpoint list (:182-186). All five exist as described (`vulnerabilities.py`).
- Highest-severity-wins and dimming of non-vulnerable nodes at 15 % opacity (:201-203). Match `highestSeverityByPurl` and `applyMultipleOverlays` (`GraphVisualizer.js:98, 339-357`), and the colours match Ch4 :159 and the figure caption.
- Blast radius code (:219) and paths: up to 5 `all_shortest_paths` from in-degree-0 roots (:221). Match `impact.py:50-64`.
- Dominated set (:233): virtual root for multiple roots, strict dominator subtree. Matches `impact.py:11-38`, and the graph-theoretic definition ("every path from the root passes through v") is correct for the strict dominated set.
