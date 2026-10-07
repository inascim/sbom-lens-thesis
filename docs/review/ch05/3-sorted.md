# Chapter 5 — Sorted review

## Summary
- 13 items: **10 errata** (C9, C1, C2, C3, C4, C5, C8, C6, D-new1, C7), **3 viva prep** (C10, C11, D-new2), **0 holds**. Every objection has a real basis in the code. Where the defender rebutted part of one, the rebuttal narrows the fix but does not remove it.
- **Results:** no result changes unless you choose to. The 19 % vulnerability reach (Ch7:132, Ch8:16) is correct for the inclusive blast radius the code computes. If you define the blast radius as dependents only, it becomes 12 % (4 %–29 %), and Ch7:132, Ch8:16 and the reach figure change (C9). The 8,868 findings and per-band counts (Ch7:51) are correct as OSV advisory–component findings. The tool would store 8,743 (C7). No RQ2/RQ3 conclusion changes either way: 0 components change worst band.
- One defender claim was wrong and I settled it: the thesis does **not** use one blast-radius definition. Ch2:126 defines it as |anc(v)|, which excludes v, and that contradicts Ch5:219 and Ch3's table.
- Four decisions are needed (list at the end): the blast-radius definition, which system version the schema describes, how to report the finding counts, and whether to state the stale-overlay bug.

## Erratum

### E1 (from C9) Blast radius is defined two ways in the thesis; the reach result depends on which — impact: high (touches a reported result)
- Settled facts:
  - Code includes the node: `impact.py:50 @28113e7`, `list(nx.ancestors(G, node_id)) + [node_id]`. Ch5:219 and Ch3:115 (table: "`log4j-core` itself") say the same.
  - **Ch2:126 excludes it**: "|anc(v)| is the blast radius of a vulnerability in v: the number of components that would be affected". CONTEXT.md:21 and Ch1:31 also define it as dependents only. So the defender's claim that the thesis is internally consistent is false. Only the glossary was checked, not Ch2.
  - Ch7:59 (RQ2) uses dependents only (`analyse.py:94`, `len-1`). This is a constant shift, so Kendall τ is unaffected. Ch7:61/132 reach uses the inclusive set (`analyse.py:129`).
  - I recomputed from `sbom-lens-eval/data/results` and both parties' figures are exact. Inclusive reach: median 0.186 (0.095–0.349), i.e. Ch7's "19 %, 9 %–35 %". Each node excluded from its own radius: 0.118 (0.040–0.288). Vulnerable components alone: 0.079.
  - Ch7:132, "as opposed to how many components are themselves vulnerable", is misleading under either definition, because the vulnerable components are inside the 19 %.
  - Ch5:219 calls the inclusive set "the upstream consumers of B", but B is not its own consumer.
- Fix: **DECISION NEEDED** (D1). Fixes for both options:
  - Option A, keep inclusive (no result change):
    - Ch2:126 — "\(|\mathrm{anc}(v)|\) is the \textit{blast radius} of a vulnerability in \(v\): the number of components that would be affected if \(v\) were compromised" → "the \textit{blast radius} of a vulnerability in \(v\) is \(v\) together with \(\mathrm{anc}(v)\): the vulnerable component and every component that would be affected if it were compromised"
    - Ch5:219 — "the upstream consumers of B whose security posture is degraded if B is compromised" → "the upstream consumers of B whose security posture is degraded if B is compromised; the blast radius is B together with these consumers"
    - Ch7:59 — "by the size of their blast radius, the number of components that depend on them directly or transitively" → "by the size of their blast radius excluding the component itself, i.e. the number of components that depend on them directly or transitively"
    - Ch7:132 — "as opposed to how many components are themselves vulnerable" → "It includes the vulnerable components themselves (a median 8\%); counting only components that depend on at least one vulnerable component gives a median of 12\% (4\% to 29\%)."
    - CONTEXT.md:21 — "The set of components that depend on a given component" → "A given component together with every component that depends on it, directly or transitively"
  - Option B, adopt dependents only (result changes):
    - Ch5:219: state that the endpoint's list also contains the node, which the analysis excludes.
    - Ch3 table:115: drop "and `log4j-core` itself".
    - Ch7:61: define reach over dependents.
    - Ch7:132 and Ch8:16: 19 % → 12 %, range 9 %–35 % → 4 %–29 %.
    - Regenerate the reach panel of the RQ3 figure (`analysis/analyse.py` rq3 and the plot at :233). This is a change to the evaluation analysis, not to SBOM Lens.
- Also touches: Ch2:126, Ch3:115, Ch7:59/61/132, Ch8:16, CONTEXT.md:21. Ch1:31 ("all components that depend on it") is fine under B and loose under A.
- Viva answer: "The tool's blast radius includes the vulnerable component, because it is itself exposed. RQ2 uses the dependents-only size, which differs by a constant and leaves τ unchanged. Under the dependents-only definition, reach is 12 % instead of 19 %. The conclusion that a substantial share of each application inherits a known vulnerability holds either way."

### E2 (from C1) The five-state lifecycle does not exist: three states, the terminal one is REMEDIATED — impact: high
- Settled facts:
  - `git grep` at 28113e7 over `src backend` finds only `NOT_STARTED`/`IN_PROGRESS`/`REMEDIATED`. Examples: `useVulnerabilities.js:13`, `VulnerabilityTable.js:33`, `VulnerabilityForm.js:152`, `sbom-vulnerabilities/index.js:79`, `dashboard/index.js:68`.
  - No `RESOLVED`, `WONT_FIX` or `FALSE_POSITIVE` exists in either commit.
  - The backend does not validate status: `vulnerabilities.py:15,26`, `body: dict`.
- Fix:
  - Ch5:4 — "the five-state status lifecycle" → "the three-state status lifecycle"
  - Ch5:53 — "one of \texttt{NOT\_STARTED}, \texttt{IN\_PROGRESS}, \texttt{RESOLVED}, \texttt{WONT\_FIX}, or \texttt{FALSE\_POSITIVE}" → "one of \texttt{NOT\_STARTED}, \texttt{IN\_PROGRESS}, or \texttt{REMEDIATED}"
  - Ch5:189, first five sentences ("The \texttt{status} field supports a five-state … does not actually affect the component as used.") → "The \texttt{status} field supports a three-state remediation lifecycle. \texttt{NOT\_STARTED} is the initial state assigned to every newly created record. \texttt{IN\_PROGRESS} indicates that a remediation effort is under way. \texttt{REMEDIATED} is the terminal state, indicating that the vulnerability has been patched or mitigated. The backend stores the field without validating it, so triage states such as a deliberate decision not to fix, or a false positive, could be added to the interface without changing the API; the evaluated version does not offer them."
- Also touches: none. Optionally add triage states to future work in Ch8.
- Viva answer: "The lifecycle is a UI convention over an unvalidated field. Won't-fix and false-positive are natural extensions and are future work."

### E3 (from C2) The timeline is not a server-side, append-only audit trail — impact: high
- Settled facts:
  - `useVulnerabilities.js:187-201 @28113e7` builds the entry with a client `new Date().toISOString()` and `user: "Current User"`, then PUTs the whole `timeline` array.
  - `vuln_store.py:50`, `{**self.vulns[vuln_id], **updates, ...}`, lets any PUT replace any field, including `timeline`, `cveId` and `purl`.
  - Only `lastUpdated` is set by the server (:51).
  - The defender's point holds: when a POST carries no timeline, which is the case for every OSV record (`vulnerabilityAPI.js:150-159`), the server writes the first entry with a server timestamp and `user: "System"` (`vuln_store.py:40-42`).
  - The UI's only PUT sends `status`, `lastUpdated` and `timeline`. It never sends `notes` or `patchAvailable`: the note goes into the timeline entry, not the `notes` field.
- Fix:
  - Ch5:59 — "append-only list of status-transition objects, each carrying a timestamp and the new status value, supporting an audit trail of remediation history" → "list of status-transition objects, each carrying a timestamp, the new status value and a note, recording the remediation history of the record"
  - Ch5:184 — "partial update of an existing record; the analyst may update \texttt{status}, \texttt{notes}, or \texttt{patchAvailable}; the \texttt{lastUpdated} timestamp is refreshed automatically by the server" → "partial update of an existing record: every field sent replaces the stored value; the interface uses it to change \texttt{status} and extend \texttt{timeline}. The \texttt{lastUpdated} timestamp is set by the server"
  - Ch5:189, last sentence ("Each status transition is appended … for each record.") → "The first \texttt{timeline} entry is written by the server when the record is created; each later status change made through the interface appends an entry with a client-side timestamp and sends the extended list back. The server does not enforce this, since \texttt{PUT} replaces any field it receives, so the timeline is a remediation history for a single-user prototype rather than a tamper-proof audit trail."
- Also touches: none.
- Viva answer: "There is no authentication and one user, so integrity was out of scope. The timeline is append-only by convention, because the only UI path appends."

### E4 (from C3) Fetching runs once, at upload, not "whenever an SBOM is selected"; the reload/409 story is misattributed — impact: high
- Settled facts:
  - The only call site is `dashboard/index.js:86 @28113e7` (`handleUploaded`).
  - `UploadBatchDialog/index.js:53-58` calls `onUploaded` only when `!result.isDuplicate`.
  - Identical content is rejected by SHA-256 (`sboms.py:31-37`).
  - Selecting an SBOM or refreshing the page never fetches.
  - The defender is right that the 409 path is real and common:
    - `delete_sbom` (`sboms.py:114-121`) does not touch `vuln_store`, so a re-upload after deletion resubmits every pair.
    - Shared components: I recomputed 3,789 distinct (cveId, purl) pairs, of which 1,267 occur in more than one SBOM. That gives 4,954 POSTs that would get a 409 if all 90 were uploaded.
  - Records are never refreshed after upload.
- Fix:
  - Ch5:32 — "whenever a new SBOM is selected in the frontend" → "whenever an SBOM is uploaded to the library"
  - Ch5:32, last sentence ("On subsequent loads of the same SBOM, … no duplication.") → "When an upload contains pairs already in the store --- because another SBOM shares the component, or because the same SBOM is uploaded again after being deleted --- the backend rejects each with HTTP 409, and the frontend hook treats 409 as a no-op, so the analyst observes no duplication."
  - Ch5:38 — "the fetch runs again every time an SBOM is loaded" → "the fetch runs for every SBOM uploaded, and SBOMs in one library often share components"
  - Ch5:169 — "triggered automatically every time an SBOM is selected in the frontend" → "triggered automatically when an SBOM is uploaded to the library". In the same paragraph, "from the loaded SBOM" → "from the uploaded SBOM".
  - Ch5:173, whole paragraph → "Because records are keyed by CVE ID and PURL rather than by SBOM, an upload that shares components with an SBOM already in the library resubmits pairs that already exist in \texttt{VulnStore}; so does an SBOM uploaded again after being deleted, since deleting an SBOM does not delete its vulnerability records. Each such pair receives a 409 Conflict response, which the hook treats as a no-op. Vulnerabilities are fetched once, at upload: selecting an SBOM or reloading the page does not query OSV again, so an advisory published later reaches an SBOM only if it is uploaded again."
  - Optional, for consistency: Ch5:4 "automated fetching on SBOM load", Ch5:163 "on SBOM load" and the subsection title at :166 → "upload".
- Also touches: none.
- Viva answer: "OSV is queried once per SBOM, not on every navigation. The cost is staleness, which is now stated. Deduplication is exercised heavily, because a third of the pairs recur across SBOMs."

### E5 (from C4) The impact endpoint returns HITS scores for the queried node only — impact: medium
- Settled facts:
  - `impact.py:70-84 @28113e7` returns scalar `hitsHub`/`hitsAuthority` for `node_id`.
  - Per-node scores come only from `/global-analysis` (:105-111).
  - Ch6:32 already describes this correctly, so Ch5 contradicts Ch6.
- Fix:
  - Ch5:217 — "and per-node HITS scores in a single response" → "and the HITS hub and authority scores of the queried node in a single response"
  - Ch5:225 — "scores for each node in the blast radius. These scores quantify the structural centrality of each node in the dependency network" → "scores of the analysed component; scores for every node are returned by \texttt{GET /sboms/\{id\}/global-analysis}. These scores quantify the component's structural centrality in the dependency network"
- Also touches: none (Ch6:32 is already right).

### E6 (from C5) Blast radius and dominated set are separate toggles; re-targeting refreshes only one — impact: medium
- Settled facts:
  - Both sets come back in one response (`impact.py:79-86`), so Ch5:231 is true.
  - The overlays can coexist, because `activeModes` is a Map.
  - The node context menu has "Analyze Impact" (`GraphVisualizer.js:614`).
  - The modes are separate Analysis-menu items (`sbom-graph/index.js:561, 585`).
  - `handleAnalyzeImpact` (:502-527) uses `if blast-radius … else if dominator`. With both on, a new node refreshes only the blast radius, and the cyan overlay keeps the previous node's set. I confirmed this in the code.
  - Ch4:189 claims the context menu "fetches blast-radius and dominator-tree data". It fetches only when one of those modes is already active, and then only one of them.
- Fix:
  - Ch5:195 — "and the blast-radius and dominator-tree overlays from a node's context menu" → "and the blast-radius and dominator-tree overlays from the same menu, for the node chosen with \emph{Analyze Impact} in its context menu"
  - Ch5:217 — "triggered when the analyst selects a node and requests impact analysis from the context menu" → "triggered when the analyst chooses a node with \emph{Analyze Impact} in its context menu and switches the overlay on in the Analysis menu"
  - Ch5:235 — "Because the dominator-tree overlay is rendered simultaneously with the blast radius within the same interaction, the analyst can read both overlays together" → "When both overlays are switched on for the same node, the analyst can read them together"
  - **DECISION NEEDED** (D4): whether to add "In the evaluated version, choosing a new node while both overlays are active refreshes only the blast radius, so the dominator-tree overlay must be switched on again."
  - Ch4:189 — "which fetches blast-radius and dominator-tree data from the backend" → "which selects the node for the blast-radius and dominator-tree overlays of the Analysis menu"
- Also touches: Ch4:189.
- Viva answer: "One request returns both sets, by design. The stale-overlay case is a frontend defect in the evaluated version and does not affect the evaluation, which calls the endpoint directly."

### E7 (from C8) "Precisely the packages an attacker can exploit" and "removed or compromised" overclaim — impact: medium
- Settled facts:
  - `nx.ancestors` is graph reachability only (`impact.py:50`).
  - Ch7:157 and Ch8:45 concede the over-approximation.
  - Compromise does not remove a node. CONTEXT.md:25 and Ch7:130 say "removed or replaced".
  - The defender's reading of "precisely" as a statement about direction is plausible, but the sentence still asserts exploitability.
- Fix:
  - Ch5:219 — "and they are precisely the packages that an attacker can exploit by compromising the vulnerable node." → "and so the packages that may be affected if the vulnerable node is compromised. Because the analysis is at component level, this over-approximates exposure: a consumer may never execute the vulnerable code (\autoref{sec:rw-call-graphs})."
  - Ch5:233 — "if it were removed or compromised" → "if it were removed or replaced"
- Also touches: none.
- Viva answer: "The point of the sentence was the direction of propagation. Component-level over-approximation is stated in Ch3, Ch7 and Ch8."

### E8 (from C6) `patchAvailable` was never set by the OSV fetch in the evaluated system — impact: low
- Settled facts:
  - `vulnerabilityAPI.js:158 @28113e7` sets `patchAvailable: false` for every OSV record.
  - The field can be set only in the manual-entry form (`VulnerabilityForm.js:26,73,159-160`). No UI path updates it afterwards.
  - At 28113e7, `VulnStore.add()` drops `summary` (`vuln_store.py:29-43`), so the Ch5:46-60 schema list is exact for the evaluated version.
  - `summary`/`fixedVersions` and derived `patchAvailable` arrived in `b0da73b`, which is post-thesis. They are not an error in the thesis.
- Fix: **DECISION NEEDED** (D2).
  - Option A, evaluated version: Ch5:54 — "boolean indicating whether a patched version is known to exist" → "boolean, set by the analyst when entering a record manually, indicating that a patched version is known to exist; records from the OSV fetch always carry \texttt{false}"
  - Option B, current version: describe the derivation from OSV `fixed` events and add `summary` and `fixedVersions` to the list. The thesis would then no longer describe the evaluated commit.
- Also touches: none.

### E9 (from D-new1) Error-handling description is inaccurate: OSV failures are not logged, and one store error abandons the rest — impact: low
- Settled facts:
  - `fetchVulnerabilitiesForSBOM` returns `[]` on a non-OK response or an exception, with no log (`vulnerabilityAPI.js:100-104`). Failed detail fetches are dropped (:122-123, :138).
  - Only a non-409 POST failure throws (`useVulnerabilities.js:140`). It is caught and logged with `console.warn` (`useVulnerabilityFetcher.js:45-47`).
  - The loop sits inside the `try`, so the remaining records of that SBOM are abandoned.
  - Ch5:32 ("caught and logged") is wrong for OSV failures. Ch5:171 ("swallows errors silently") is loose but consistent with the code comment. Together they mislead.
- Fix:
  - Ch5:32 — "errors are caught and logged at the \texttt{console.warn} level" → "no error is ever shown to the analyst"
  - Ch5:171 — "and swallows errors silently so that a network failure or an unresponsive vulnerability database never prevents the analyst from viewing the dependency graph." → "and never surfaces an error to the analyst: a failed OSV request yields an empty result, and a backend error while storing records is logged with \texttt{console.warn} and abandons the remaining records of that SBOM. A network failure therefore never prevents the analyst from viewing the dependency graph, but an empty vulnerability list cannot be told apart from a failed lookup."
- Also touches: none. This also covers the silent-failure half of C10.

### E10 (from C7) Deduplication silently merges distinct advisories that share a CVE alias; "rare case" is inaccurate — impact: low
- Settled facts:
  - I recomputed 8,868 records with 125 duplicate (cveId, purl) pairs within an SBOM, leaving 8,743 after deduplication.
  - Per-band counts change from 462/4,081/3,622/703 to 462/4,039/3,539/703.
  - **0** components change worst band, so RQ2/RQ3 are unaffected.
  - The defender is right that the :77 case does happen: there are 162 OSV-ID-fallback pairs, 38 of them in more than one SBOM. But it is not "rare".
  - The defender is also right that Ch7:20 claims identical lookup and severity derivation, not identical storage (`pipeline/vulns.mjs` bypasses `VulnStore`). So Ch7 is not false.
- Fix: Ch5:77 — "preventing duplicate OSV records in the rare case where the same OSV entry is fetched more than once." → "preventing duplicates when the same OSV entry is fetched again for another SBOM. Because the key is the first CVE alias, two distinct advisories that share that alias for the same component are also collapsed into one record, and the advisory stored second is discarded."
- Ch7:51 is **DECISION NEEDED** (D3); see the list below.
- Also touches: Ch7:51 (optional, depending on D3).
- Viva answer: "Chapter 7 counts advisories as OSV returns them, which is the more faithful exposure measure. The tool stores one row per CVE per component, so it would hold 8,743. No component's worst band, and so no RQ result, differs."

## Viva prep

### V1 (from C10) The OSV batch query ignores pagination
- Likely question: "Your single round-trip design assumes OSV returns everything. What happens for a large SBOM?"
- Answer: OSV paginates above 1,000 vulnerabilities per query or 3,000 per batch, and `vulnerabilityAPI.js:93-113` does not follow `next_page_token`, so results above those limits would be silently truncated. The evaluation is far below them: the largest SBOM has 387 findings (recomputed). Optional one-sentence addition after Ch5:26: "The implementation does not follow OSV's pagination tokens, issued above 1,000 vulnerabilities per query or 3,000 per batch; no evaluated SBOM came near these limits." Silent failure is fixed under E9.

### V2 (from C11) UNKNOWN drawn with the LOW colour
- Likely question: "You say UNKNOWN must not look like low risk, then draw it in the LOW colour. Why?"
- Answer: Ch5:108 discloses the mapping (`sbom-graph/index.js:227-229`). It is a display-only shortcut to keep the four severity classes of Ch4, and the data and the evaluation keep UNKNOWN separate (no evaluation finding was UNKNOWN). A grey UNKNOWN colour already exists elsewhere in the UI (`utils/severity.js`), so this is a known limitation. Optional text change at Ch5:108, after "drawn with the \textsc{low} colour": add "to keep the four overlay classes of \autoref{sec:cytoscape-rendering}; this is a known limitation, since such a node is indistinguishable from a \textsc{low} one".

### V3 (from D-new2) Records are global, so a status applies across SBOMs
- Likely question: "If I mark a CVE remediated in application A, what happens in application B?"
- Answer: Records are keyed by (cveId, purl) with no SBOM identifier (`vuln_store.py:29-43`), and this is by design: `useVulnerabilities.js:5` says "they apply across all SBOMs containing that component". A status therefore applies to every SBOM containing that exact component version, which is correct if remediation means replacing that version. Records also outlive their SBOM (`sboms.py:114-121`). Ch5:182 already says "global view". The E4 rewrite of :173 now states the keying. Optionally add to §5.4.2: "A status therefore applies to every SBOM containing that component."

## Holds
- None. Each objection is supported by the code or the data. The defender's partial rebuttals (C3, C5, C6, C7, C8, C9) narrow the fixes above but do not dismiss any item.

## Decisions needed
1. **Blast-radius definition (E1/C9; affects a reported result).** (A) Keep the inclusive definition the code uses: fix Ch2:126, Ch5:219, Ch7:59, Ch7:132 and CONTEXT.md:21; 19 % stands. (B) Adopt dependents only: reach becomes 12 % (4 %–29 %) in Ch7:132 and Ch8:16, the reach figure is regenerated, and Ch3:115 and Ch5:219 change.
2. **Which version the schema describes (E8/C6).** (A) The evaluated `28113e7`: `patchAvailable` is analyst-set, false from OSV, and the schema stays as is. (B) The current HEAD: add `summary` and `fixedVersions`, and say `patchAvailable` is derived from OSV `fixed` events. The thesis would then no longer match the evaluated commit.
3. **How Ch7:51 reports findings (E10/C7).** (A) Keep 8,868 and add "(advisory–component pairs as returned by OSV; \texttt{VulnStore} would store 8{,}743, since 125 share a CVE and component with another, without changing any component's worst band)". (B) Report the stored counts, 8,743: 462 / 4,039 / 3,539 / 703. (C) Leave Ch7:51 unchanged and rely on the Ch5:77 fix plus a viva answer.
4. **Whether to state the stale-overlay defect (E6/C5).** (A) Add the one-sentence limitation at Ch5:235. (B) Correct only the description and keep the defect as a viva answer.
