# Chapter 7 — Sorted review

## Summary
- 21 items (C1–C19, D-new1, D-new2). Sorted into **19 errata**, **2 viva prep** and **0 holds**. E1 merges C1 and D-new2. C17 is split: its one false sentence is fixed under E11, and the rest is viva prep (V1). V2 is a new point from my own check. The errata, by impact: E1 (C1+D-new2), E2 (C9), E3 (C7), E4 (C6), E5 (C3), E6 (C4), E7 (C10), E8 (C2), E9 (D-new1), E10 (C11), E11 (C8, plus part of C17), E12 (C5), E13 (C14), E14 (C15), E15 (C13), E16 (C12), E17 (C19), E18 (C18), E19 (C16).
- **Results.** Reported numbers change only under E1 (power law), E3 (top-k ties), E6 (the RQ3 summary sentence) and E8 (x_min, whose numbers follow E1). **No RQ conclusion changes under any option.**
  - E1 / C1. Option A: α is reported for 79 graphs, not 87, with median 2.62 → 2.58, Q3 2.84 → 2.78 and max 3.00 → 2.99; the x_min and D rows change too, and so do the figure and Ch8:12. Option B: no number changes, but the bound is stated and Ch8's range is reworded. Option C: max 3.00 → 3.39 and D min 0.006 → 0.007, but this cannot be done with SBOM Lens outputs at 28113e7. Under every option the claim that "all" exponents lie in the scale-free range goes or is qualified. The heavy-tail reading stands.
  - E3 / C7, if tie-aware: zero overlaps 66 → 61 (blast) and 54 → 47 (authority). Q3 0.12 → 0.16 and 0.25 → 0.24, max 0.67 → 0.58 and 1.00 → 0.70. Medians stay 0.00.
  - E6 / C4: Ch7:134's "a fifth … depends on" becomes "a fifth is vulnerable or depends on … an eighth depends on". The 19 % and 12 % already decided are unchanged.
  - Abstract EN/PT: wording only (E2, E18). Ch8:12 (E1), Ch8:14 (E13) and Ch8:51 (E18) change. Ch8:16 does not.
- **Settled disputes.**
  - C1 / D-new2: both are right. I reproduced all 87 stored (α, x_min) pairs from the stored degree histograms. `noise_flag` is set in exactly the 8 graphs with α = 3.000 and in none of the other 79, whose maximum is 2.987. The defender is also right that the 8 capped values cannot move Q1, the median or Q3 at the stored x_min.
  - C6: the critic is right that "starker still" is wrong. The defender is right that the overlap is defined as a practical measure. I confirmed the chance baselines of 57.0 of 90 and 51.3 of 79 zeros. The expected mean overlap is k/N, a mean of 0.134 (blast) and 0.132 (authority). The observed means are 0.086 and 0.137.
  - C8: the critic is right on "largest graphs" and on the audit (Spearman ρ = 0.96). The defender is right that 1.6–151 ms rounds to "2 and 150". **The defender's cause is wrong.** Dense `eigh` on a 500-node matrix takes 0.02–0.05 s on this venv, so it cannot explain 2–3 s peaks. The cause is not established and must not go into the thesis.
  - C10: the defender is right that a log-normal win is not evidence against "heavy-tailed". **But the defender's "lighter in none" holds only for re-selected x_min.** At the stored fits, the exponential is significantly better (p < 0.1) in 2 of the 87 graphs.
  - C11: the defender is right that "23 match exactly" already uses both directions (23 exact both ways; 26 exact on precision only). MusicFree (2.9 %) is not a notable case.
  - Everything else both parties report, I recomputed and it agrees. That includes 33/24 ties, 61–67 and 47–60 zero ranges, dominated set ⊆ descendants (1,186/1,186), 1,186 of 1,565, median dominated-set size 3, minimum blast 1, |τ_b| < 0.1 in 38, 21,915 candidates with a minimum of 466 stars, 23 pnpm + 1 CalledProcessError, and lockfile misses 44/53, stackedit 20 % and wekan 14 %.
- **Four decisions are needed** (list at the end): the power-law exponents (D1, affects reported results), top-k ties (D2, affects reported results), whether to report likelihood ratios (D3), and how to scope the reproducibility claim (D4).

## Erratum

### E1 (from C1, D-new2) The maximum α = 3.00 is powerlaw 2.0.0's default bound; 8 fits sit on it, and the package marks them failed — impact: high (touches reported results: Tab. 7.2, Fig. 7.1, Ch7:77, Ch8:12; Ch6:170)
- Settled facts (my scripts: scratchpad `c1.py`, `c1s.py`, `c10.py`; refits from the stored `scaleFree.logLogData`, which the defender showed equals the SBOM's in-degree histogram):
  - `graph_analysis.py:128 @28113e7` calls `powerlaw.Fit(degrees, discrete=True, verbose=False)` with no `parameter_ranges`. powerlaw 2.0.0 sets `Power_Law.DEFAULT_PARAMETER_RANGES = {'alpha': [0, 3]}`.
  - A default refit reproduces all 87 stored (α, x_min) pairs. Eight have α = 3.000: fabric.js, chat-ui, ignite, readme-md-generator, color-thief, AnotherRedisDesktopManager, johnny-five and typeorm. All eight, and no other graph, have `noise_flag` set ("No valid values for xmin found"). The largest valid α is 2.987.
  - The backend never checks the flag (`graph_analysis.py:127-134`). So the tool returns, as exponents, fits the package itself marks failed. This makes Ch6:170, "a fit that fails … also returns no exponent", false for these 8 (D-new2).
  - With the stored x_min and the bound lifted (`[1, 10]` or `[1, None]`, same result), the eight are 3.029–3.386. Q1, the median and Q3 are unchanged. KS D changes for these 8 (e.g. AnotherRedisDesktopManager 0.006 → 0.056).
  - The 3 graphs without a fit have in-degrees {1, 2, 3}, and `Fit` raises ValueError. Ch7:77's reason for them is right.
  - Ch2:150 gives γ ∈ [2, 3] "for most empirical cases", so values of 3.03–3.39 do not contradict a heavy tail. They contradict only "all lie … inside the range".
  - The stored outputs contain α (rounded to 3 decimals), x_min, σ, D, nTail and the degree histogram. They do not contain `noise_flag`. In this dataset, α = 3.000 is exactly equivalent to the flag.
- Fix: **DECISION NEEDED** (D1). Ch6:170 changes under every option:
  - Ch6:170 — "a fit that fails, as it can when the in-degrees take only a few distinct values, also returns no exponent." → "a fit that fails, as it can when the in-degrees take only a few distinct values, also returns no exponent. The implementation does not, however, check the package's flag for a fit in which no candidate \(x_{\min}\) gives a valid estimate; with \texttt{powerlaw}~2.0.0, whose default range bounds \(\alpha\) at 3, such a fit is returned with \(\alpha = 3\) (\autoref{sec:eval-rq1})."
  - **Option A — treat the 8 as failed fits; report α for 79 of 90.**
    - Where the change is made: `analyse.py` only, from the stored outputs. In `rq1()`, set `alpha`, `xmin` and `ksD` to NaN when the stored α equals 3.0, the package bound, with a comment naming powerlaw 2.0.0 and the noise flag. Change the extra print to "alpha fitted in 79 of 90". No backend re-run is needed.
    - This keeps the rule that every value is an SBOM Lens output (`analyse.py:3`): the script withholds 8 outputs and computes none. The thesis must say so (Ch7:77 below and the Ch6:170 sentence).
    - Caveat: a genuine estimate in [2.9995, 3) would also be dropped, but none exists here (max 2.987).
    - Tab. 7.2 (`tab_rq1_profile.tex:17-19`, regenerated, never hand-edited):
      - α: "87 & 2.62 & 2.50 & 2.84 & 2.09 & 3.00" → "79 & 2.58 & 2.50 & 2.78 & 2.09 & 2.99"
      - x_min: "87 & 1 & 1 & 1 & 1 & 3" → "79 & 1 & 1 & 2 & 1 & 3". Q3 is 1.5 and the 0-digit format prints 2. Consider 1 digit for this row.
      - D: "87 & 0.024 & 0.018 & 0.037 & 0.006 & 0.089" → "79 & 0.024 & 0.018 & 0.037 & 0.007 & 0.089"
    - Fig. 7.1, middle panel: 79 values, range 2.09–2.99. The last bin drops from 12 to 5, and the tallest bin becomes 2.51–2.57 (11).
    - Ch7:77, from "A power law could be fitted" to "(\autoref{sec:scale-free})" — "A power law could be fitted to the in-degree distribution of 87 graphs; in the remaining three, the in-degrees take too few distinct values. The fitted exponents are tightly clustered around \(\alpha = 2.62\) (interquartile range 2.50--2.84) and all lie between 2.09 and 3.00, inside the range typical of scale-free networks (\autoref{sec:scale-free})." → "A power law could be fitted to the in-degree distribution of 79 graphs. In three, the in-degrees take too few distinct values. In the other eight, no candidate lower bound \(x_{\min}\) gives an estimate inside the \texttt{powerlaw} package's default range \(\alpha \leq 3\), and the package marks the fit as failed; SBOM Lens returns these fits with \(\alpha = 3\) (\autoref{sec:ns-powerlaw}), and they are counted here as having no fit. The fitted exponents are clustered around \(\alpha = 2.58\) (interquartile range 2.50--2.78) and lie between 2.09 and 2.99, inside the range typical of scale-free networks (\autoref{sec:scale-free})."
    - Ch7:86 caption — "(87 SBOMs)" → "(79 SBOMs)"
    - Ch8:12 — "In 87 of the 90 graphs the in-degree distribution is heavy-tailed, with fitted exponents between 2.1 and 3.0," → "A power law could be fitted to the in-degree distribution of 79 of the 90 graphs, with exponents between 2.1 and 3.0 (median 2.6),"
    - The E8 (x_min) numbers become 59 of 79, 20 and 11 %–40 %.
  - **Option B — keep the 87 stored values; state the bound.**
    - Where the change is made: thesis text only. `analyse.py`, the table cells and the figure are unchanged. If a table note is wanted, it goes in the caption string at `analyse.py:276`, never in the .tex.
    - Ch7:77, same span as in A → "A power law could be fitted to the in-degree distribution of 87 graphs; in the remaining three, the in-degrees take too few distinct values. The median fitted exponent is \(\alpha = 2.62\) (interquartile range 2.50--2.84), and the exponents lie between 2.09 and 3.00. The upper end is not a measurement: the \texttt{powerlaw} package (version~2.0.0) bounds \(\alpha\) at 3 by default, and in eight graphs the estimate sits on that bound, a fit the package itself marks as having no valid \(x_{\min}\) (\autoref{sec:ns-powerlaw}). The other 79 lie between 2.09 and 2.99, inside the range typical of scale-free networks (\autoref{sec:scale-free})."
    - Optional, B′: add "Refitted outside SBOM Lens without the bound and at the same \(x_{\min}\), the eight exponents lie between 3.03 and 3.39, and the median and quartiles are unchanged." This is the defender's fix. The values are not SBOM Lens outputs, so the sentence must say so, as it does.
    - Ch7:86 caption — after "(87 SBOMs)" add "; eight sit on the fitting package's upper bound of 3"
    - Ch8:12 — "In 87 of the 90 graphs the in-degree distribution is heavy-tailed, with fitted exponents between 2.1 and 3.0," → "A power law could be fitted to the in-degree distribution of 87 of the 90 graphs, with a median exponent of 2.6; eight of these estimates sit on the fitting package's upper bound of 3,"
    - The E8 numbers stay 66 of 87, 21 and 10 %–40 %.
  - **Option C — report the unbounded values (3.03–3.39) for the 8.**
    - This cannot be done at 28113e7 from the stored SBOM Lens outputs. The stored α is the bounded 3.000.
    - (i) `analyse.py` could refit from the stored `logLogData` and x_min with `parameter_ranges={'alpha': [1, None]}`. The inputs are tool outputs, but the reported α would be the script's, which breaks `analyse.py:3`. It would also add `powerlaw` to the analysis dependencies and contradict Ch7:57 ("with the measures of Ch6").
    - (ii) Re-running `graph-theory` with a patched call leaves 28113e7, against the decision already taken.
    - (iii) Re-running 28113e7 under powerlaw 1.5, which has no upper bound, does not isolate the 8: x_min changes in 74 of 87 and all 90 get a fit (V2).
    - If chosen anyway via (i):
      - Tab. 7.2 α: "… & 3.00" → "… & 3.39" (n, median, Q1, Q3 and min unchanged). D: min 0.006 → 0.007. x_min unchanged.
      - Fig. 7.1: range to 3.39; the tallest bin becomes 2.43–2.52 (13).
      - Ch7:77: "between 2.09 and 3.39; 79 of the 87 lie within the range 2 to 3 typical of scale-free networks, and the other eight, refitted without the \texttt{powerlaw} package's default bound of 3, between 3.03 and 3.39".
      - Ch8:12: "between 2.1 and 3.4".
      - Ch7:169 must state that the script refits these eight.
  - **RQ1 conclusion under each option:** unchanged. "Heavy-tailed, a few components depended on by many" holds in all three. What changes is the sub-claim that every fitted exponent lies in 2–3. It survives in A for 79 graphs with fits (but 11 have none), and it is qualified in B and C.
- Also touches: Ch6:170 (all options); `analyse.py` (A, C); Tab. 7.2 and Fig. 7.1 (A, C); Ch7:86 (A, B); Ch8:12 (all); Ch7:169 (C); E8's numbers (A). The Abstract and Ch1 report no α.
- Viva answer: "The fit uses powerlaw's defaults. Version 2.0.0 bounds α at 3, and in eight graphs every candidate x_min hit that bound, which the package flags as no valid fit. The backend did not check the flag. Without the bound, those eight are 3.03–3.39, and the median and quartiles do not move. The heavy-tail reading does not depend on the cap. The claim that all exponents lie in 2–3 did."

### E2 (from C9) The abstract claims effectiveness for prioritising that Ch7 disclaims; PT says "independent" — impact: high (Abstract EN/PT)
- Settled facts: the defender's split is right. "Unrelated" restates Ch7:107. "Proved" is stronger than a descriptive median τ_b of 0.00 (Ch7:63: no significance claim). "Independentes" names statistical independence, which nothing tests. Ch7:111 says τ_b ≈ 0 does not show structural ranking is better, and Ch7 has no outcome measure of prioritisation quality.
- Fix:
  - Abstract EN :21 — "structural and severity rankings of vulnerable components proved unrelated" → "structural and severity rankings of vulnerable components were essentially unrelated"
  - Abstract EN :21 — "provide an effective foundation for contextualising and prioritising security risk in modern software supply chains" → "provide context for security risk in modern software supply chains that severity alone does not, and a basis for prioritising it"
  - Abstract PT :8 — "revelaram-se independentes" → "mostraram-se praticamente não relacionadas"
  - Abstract PT :8 — "constituem uma abordagem eficaz para priorizar riscos de segurança em cadeias de fornecimento de software modernas" → "fornecem, para os riscos de segurança em cadeias de fornecimento de software modernas, um contexto que a severidade, por si só, não oferece, e uma base para os priorizar"
- Also touches: none (Ch8:33 already says "were found to be unrelated").
- Viva answer: "The evaluation shows that structure carries information severity does not, and that most exposure is structural. Whether prioritising by structure gives better outcomes needs ground truth, such as exploitation data or an analyst study. That is future work."

### E3 (from C7) Top-k ties at the k-th place are broken alphabetically in 33 of 90 (blast) and 24 of 79 (authority) — impact: medium (touches reported results: Tab. 7.3, Ch7:107)
- Settled facts: `analyse.py:116` sorts by `(-value, node)`. I recomputed with `analyse.py`'s own functions and agree with both parties:

  | Variant | Blast: zeros / Q3 / max | Authority: zeros / Q3 / max |
  |---|---|---|
  | Reported (node-ID tie-break) | 66 / 0.12 / 0.67 | 54 / 0.25 / 1.00 |
  | Ties against CRITICAL | 67 / 0.08 | 60 / 0.00 |
  | Ties for CRITICAL | 61 / 0.25 | 47 / 0.33 |
  | Expected under random tie-break | 61 / 0.16 / 0.58 | 47 / 0.24 / 0.70 |

  Medians are 0.00 in every variant. The mean is 0.086 → 0.094 (blast) and 0.137 → 0.120 (authority).
- Fix: **DECISION NEEDED** (D2).
  - Option A, tie-aware (change `analyse.py` `rq2()`: overlap = (CRITICAL strictly above the k-th value + free slots × CRITICAL tied / tied) / k):
    - Ch7:59 — "the share of those \(k\) components that are also among the \(k\) components ranked highest by the structural measure" → "the share of those \(k\) components that are also among the \(k\) components ranked highest by the structural measure (when components tie at the \(k\)-th place, the expected share under random tie-breaking)"
    - Ch7:107 — "zero in 66 of the 90 SBOMs for blast radius and in 54 of the 79 rankable SBOMs for authority" → "zero in 61 of the 90 SBOMs for blast radius and in 47 of the 79 rankable SBOMs for authority"
    - Tab. 7.3 (regenerated): blast row "90 & 0.00 & 0.00 & 0.12 & 0.00 & 0.67" → "90 & 0.00 & 0.00 & 0.16 & 0.00 & 0.58"; authority row "79 & 0.00 & 0.00 & 0.25 & 0.00 & 1.00" → "79 & 0.00 & 0.00 & 0.24 & 0.00 & 0.70".
  - Option B, keep the script: Ch7:59, after "ranked highest by the structural measure", add "(ties at the \(k\)-th place, which occur in 33 SBOMs for blast radius and 24 for authority, are broken by component identifier; resolving them for or against the \textsc{critical} components gives 61--67 and 47--60 zero overlaps)".
- Also touches: E4's wording (the baseline sentence differs by option). Ch8:14 ("in most applications none …") holds under both (61/90). τ_b, Fig. 7.2 and the Abstract are unaffected.
- Viva answer: "The medians do not depend on the tie-break. The zero counts move by at most six SBOMs and stay above half."

### E4 (from C6) "Starker still": zero overlap is the expected outcome for an unrelated ranking at these k/N — impact: medium (interpretation of a reported result)
- Settled facts: the median k is 3 and the median N is 25. Under a random ordering, the expected zeros are 57.0 of 90 (blast) and 51.3 of 79 (authority). The expected mean overlap is k/N: 0.134 and 0.132. The observed means are 0.086 and 0.137. So blast is somewhat below chance and authority is at chance. The defender's permutation p = 0.008 for blast is plausible but unverified by me, and it is not needed in the text. The method (:59) defines the overlap as a practical measure, so the fix is to the reading, not the measure.
- Fix:
  - Ch7:107 — "The practical measure is starker still." → "The practical measure shows what this means for an analyst."
  - Ch7:107, after "for authority", add (D2 Option B): ". With so few \textsc{critical} components, this is close to what an unrelated ranking would give (an expected 57 of 90 and 51 of 79 zero overlaps under a random ordering), so the overlap restates the \(\tau_b\) result in practical terms rather than adding to it"
  - Under D2 Option A, the zero-count baseline no longer applies to the tie-aware measure. Use instead: ". With so few \textsc{critical} components, an unrelated ranking would place on average a share \(k/N\) of them in the top \(k\) (a mean of 0.13); the observed means are 0.09 for blast radius and 0.12 for authority, so the overlap restates the \(\tau_b\) result in practical terms rather than adding to it"
- Also touches: Ch8:14 is fine as worded. The added numbers are analysis-side statistics of tool outputs (like τ_b), and are subject to D4.

### E5 (from C3) A vulnerable chokepoint dominates only its own private dependencies; "remediation is hardest" and "irreplaceable" are unsupported — impact: medium (interpretation of a reported RQ3 result)
- Settled facts: I confirmed dominated set ⊆ `nx.descendants(G, v)` for all 1,186 vulnerable chokepoints. Median size 3 (Q1 1, Q3 5), and 26.8 % have size 1. 1,186 of the 1,565 vulnerable components with a dependency are chokepoints (75.8 %). No Ch7 measure relates to remediation effort. The defender's replacement ("a single upgrade … also settles the exposure") overclaims the other way: a new version may keep the same vulnerable dependency. I give a neutral fix.
- Fix:
  - Ch7:130 — "so they could not be removed or replaced without also losing those parts" → "so those parts arrive in the application only through them"
  - Ch7:130 — "These are the components for which remediation is hardest --- the ones that cannot simply be dropped --- and a flat inventory gives no indication of which they are." → "Removing or replacing such a component also removes the parts it dominates, which are its own dependencies; a flat inventory gives no indication of which components these are."
  - Ch7:134 — "nearly half of them are structurally irreplaceable" → "nearly half of them are the only route to some of the application's dependencies"
- Also touches: none. Ch8:16, CONTEXT.md:36-40 and Ch5:233 ("removed or replaced", decided in Ch5) are consistent with the fix.
- Viva answer: "A chokepoint's dominated set is its private subtree, a median of three packages. That is leverage for one change, not resistance to it. The thesis does not measure remediation effort."

### E6 (from C4) The RQ3 summary says a fifth "depends on" a vulnerable component; that share is an eighth — impact: medium (touches a reported claim)
- Settled facts: recomputed. Inclusive reach median 0.186 (0.095–0.349). Dependents-only 0.118 (0.040–0.288). Vulnerable components 0.079. The 19 %/12 % reporting is decided; only :134 contradicts it.
- Fix: Ch7:134 — "and roughly a fifth of each application depends on at least one of them" → "and in the median application a fifth of the components is vulnerable or depends on a vulnerable component, and an eighth depends on one"
- Also touches: none. Ch8:16 ("lies in the blast radius") is correct for the inclusive measure.

### E7 (from C10) "It confirms" contradicts the caveat in the same sentence; Ch8 counts fits as heavy tails — impact: medium (RQ1 claim; Ch8:12)
- Settled facts (my `c10.py`, normalised LR, p < 0.1):

  | Fit | PL vs exponential: PL better / exp better / inconclusive | PL vs log-normal: PL better / LN better / inconclusive |
  |---|---|---|
  | Stored x_min, 87 | 47 / 2 / 38 | 0 / 22 / 65 |
  | Stored x_min, 79 (D1-A) | 47 / 2 / 30 | 0 / 21 / 58 |
  | Re-selected x_min, bound lifted, 87 | 50 / 0 / 37 | 0 / 15 / 72 |

  The defender is right that a log-normal win does not count against "heavy-tailed". The defender's "lighter in none" is true only for the re-selected fits, not for the fits the tool reported. Ch6:171 and Ch8:49 say SBOM Lens does no LR test.
- Fix:
  - Ch7:77 — "but it confirms that a few components in each application are depended upon by many others" → "it is consistent with a few components in each application being depended upon by many others, but does not show that the tail follows a power law rather than another heavy-tailed distribution"
  - Ch8:12 is reworded under E1 (D1), which removes "heavy-tailed in 87 of the 90". The bold headline "heavy-tailed structure" can stand.
  - **DECISION NEEDED** (D3): whether to add LR counts. If yes, use the stored-fit rows (e.g. "a likelihood-ratio comparison, computed by the analysis script rather than by SBOM Lens, finds the tail significantly heavier than exponential in 47 of the 87 graphs and lighter in 2, and cannot separate a power law from a log-normal in most"). This needs `powerlaw` in `analyse.py`, and the numbers follow D1.
- Also touches: Ch8:12 (via E1). Ch8:49 under D3-yes ("in the tool").
- Viva answer: "The claim is a heavy tail, not a power law. At a few hundred components, power law and log-normal usually cannot be told apart, and the thesis does not depend on which."

### E8 (from C2) x_min is 1 in 66 of 87, not "almost every"; in the other 21 the tail is 10 %–40 % of components with a consumer — impact: medium (touches a reported claim; numbers follow D1)
- Settled facts: stored x_min is 1 in 66, 2 in 17 and 3 in 4. Tail share for x_min > 1 is 9.6 %–40.1 %. Under D1-A: 59 / 16 / 4, and 10.9 %–40.1 %.
- Fix: Ch7:77 — "The lower bound \(x_{\min}\) is 1 for almost every graph, so the fit covers nearly all components with at least one consumer, and" → (D1 B or C) "The lower bound \(x_{\min}\) is 1 in 66 of the 87 graphs, where the fit covers every component with at least one consumer; in the other 21 it is 2 or 3 and the fitted tail holds 10\% to 40\% of those components. The Kolmogorov--Smirnov distances are small (median \(D = 0.024\)). As …" — under D1-A: "1 in 59 of the 79 graphs … in the other 20 … 11\% to 40\%". Remove the original ", and the Kolmogorov--Smirnov distances are small (median \(D = 0.024\))", which is now its own sentence.
- Also touches: none beyond D1.

### E9 (from D-new1) Almost every pnpm project was lost to the generator, and the threats section does not say so — impact: medium (external validity of all results)
- Settled facts: all 23 "root has no dependency edges" rejections carry `(pnpm-lock.yaml)`. `run.py` comments that "cdxgen does not link the root to its dependencies for some pnpm lockfile formats". One pnpm project is accepted. The skip log does not record the lockfile for later stages, so "every pnpm project" cannot be claimed.
- Fix: Ch7:161, after "Each filter shapes the result." add "The generator shaped it too: cdxgen produced no dependency edges from the root for 23 projects with a \texttt{pnpm} lockfile, so only one \texttt{pnpm} project remains, and the findings describe \texttt{npm} and \texttt{yarn} installations."
- Also touches: Tab. 7.1 row :40 (E15). Optionally Ch8:51 ("npm and yarn").

### E10 (from C11) The lockfile check reports only one direction; production packages and declared dependencies missing from the graph go unreported — impact: medium (internal validity)
- Settled facts: over 53 npm-lockfile projects, the median miss is 0 %, and 44 miss none. stackedit misses 67/331 (20.2 %) and wekan 53/366 (14.5 %); promptfoo misses 7.7 %. "23 match exactly" already means both directions. `declaredMatching.unmatchedDeclared` lists 14 declared production dependencies in 7 SBOMs (bit 5, lx-music-mobile 4, and yapi, MusicFree, AriaNg, stf and wekan 1 each).
- Fix: Ch7:159, after "than npm resolves." add "In the other direction, 44 of the 53 graphs contain every package the lockfile marks as production, but two miss more than a tenth of them (14\% and 20\%). In seven SBOMs, the generated root also lacks one to five of the production dependencies declared in \texttt{package.json}, so their subtrees are absent and the size, reach and vulnerability counts of those graphs are lower bounds."
- Also touches: none.

### E11 (from C8, and the unmeasured claim of C17) Timing sentences: the audit does scale with size; spectral peaks are not on the largest graphs — impact: medium (reported performance claims)
- Settled facts: audit ρ = 0.96 with node count. The three slowest epidemic runs are on 393, 361 and 469 nodes. All 11 runs above 0.5 s (each spectral endpoint) are on 250–476 nodes. On the 21 graphs above 500 nodes, epidemic ≤ 0.039 s and optimizer ≤ 0.063 s. Medians run from 1.6 ms (graph-analysis) to 151 ms (impact), so "2 and 150" is a fair rounding. The cause of the 2–3 s peaks is unknown: dense `eigh` at 500 nodes takes 0.02–0.05 s here, so the defender's eigensolver explanation fails. The share of audit time spent in the unmaintained check was never measured.
- Fix: Ch7:151 — "Nearly all of it is the audit, and nearly all of the audit is the unmaintained-package check, which queries the npm registry once per package; its time depends on the network rather than on the graph. Every other analysis had a median between 2 and 150 milliseconds, including the impact analysis of all vulnerable components of an SBOM; the slowest were the two spectral analyses, which took at most 2.9 seconds on the largest graphs." → "Nearly all of it is the audit, whose unmaintained-package check queries the npm registry once per package, so the audit's time grows with the number of packages and with network latency. Every other analysis had a median between 2 and 150 milliseconds, including the impact analysis of all vulnerable components of an SBOM. The slowest single runs were of the two spectral analyses, at most 2.9 seconds; they occurred on graphs of 250 to 476 nodes, while on the graphs above 500 nodes these analyses took under 0.07 seconds, so the peaks are not an effect of size."
- Also touches: none (Ch8:18 "at most a few seconds" holds).

### E12 (from C5) "Every number … is produced by one analysis script" is false — impact: medium (reproducibility claim)
- Settled facts: `analyse.py` `main()` writes `per_sbom.csv`, Tabs. 7.2–7.5, four figures and five extra counts. Tab. 7.1 is hand-typed from `skipped.csv`. Dataset facts (:51) and most in-text counts (:107, :128, :130, :132, :159) are computed elsewhere. All are correct (recomputed by both parties and by me for those I checked).
- Fix: **DECISION NEEDED** (D4).
  - Option A, reword:
    - Ch7:4 — "Every number in this chapter is produced by one analysis script from the published dataset" → "Every number in this chapter is computed from the published dataset"
    - Ch7:169 — "Rerunning the analysis script on the published data regenerates every table and figure of this chapter" → "Rerunning the analysis script on the published data regenerates \autoref{tab:eval-rq1} to \autoref{tab:eval-performance} and every figure of this chapter; the selection funnel and the remaining counts in the text are computed from the same data"
  - Option B: extend `analyse.py` to print the funnel and every in-text count, and keep the claim. This is more work, but it is also the natural home for E3-A, E4's baselines and D3's LR counts.
- Also touches: E3, E4, E7 (where new numbers would come from).

### E13 (from C14) "A leaf package on which nothing else depends" never occurs — impact: low (RQ2 reading; Ch8:14)
- Settled facts: every non-root node has a consumer by construction. The minimum blast radius excluding the node is 1. The median blast share is 0.78 % for CRITICAL and 0.83 % for the others, so "as likely" stands.
- Fix:
  - Ch7:111 — "as likely to be in a leaf package on which nothing else depends" → "as likely to be in a package that a single other component uses"
  - Ch8:14 — "as likely to affect a leaf package" → "as likely to affect a package with a single consumer"
- Also touches: Ch8:14.

### E14 (from C15) "Conventionally read as negligible" has no source; τ_b = 1 is not attainable with ties — impact: low (RQ2 wording)
- Settled facts: no Kendall entry and no effect-size source in the bib. |τ_b| < 0.1 holds in 38 of 90 (recomputed). Neither party's threshold is sourced. The τ_b ceiling has median 0.79 (critic and defender agree; not re-run by me).
- Fix:
  - Ch7:107 — "64 of the 90 have \(|\tau_b| < 0.2\), a level conventionally read as negligible association. Where the association is not negligible it is more often negative than positive" → "64 of the 90 have \(|\tau_b| < 0.2\), and 38 have \(|\tau_b| < 0.1\). Where \(|\tau_b| \geq 0.2\), the association is more often negative than positive"
  - Ch7:59 — "\(\tau_b = 1\) means the structural ranking orders the components exactly as the severity bands do, \(\tau_b = -1\) the reverse" → "\(\tau_b\) is positive when the structural ranking tends to order the components as the severity bands do and negative when it tends to reverse them (with ties, \(|\tau_b|\) cannot reach 1)"
  - Recommended: cite Kendall at the first τ_b on :59. Kendall 1938 (Biometrika 30:81–93) and Kendall 1945 (Biometrika 33:239–251, ties) must be verified before adding them to the bib.
- Also touches: Bibliography.bib (if cited).
- Viva answer: "The conclusion rests on the median 0.00 and the IQR −0.17 to 0.11, not on a label."

### E15 (from C13) The funnel and filter text omit cases the code applies — impact: low
- Settled facts: the cdxgen stage holds 23 "root has no dependency edges (pnpm-lock.yaml)" and 1 "cdxgen failed: CalledProcessError". All 103 production rejections are "no production dependencies declared". `config.py:33-34` also accepts `npm-shrinkwrap.json` and treats `pnpm-workspace.yaml`/`lerna.json` as monorepo markers. Tab. 7.1 is hand-written, so editing it is allowed.
- Fix:
  - Ch7:40 — "Root without dependency edges in the generated SBOM" → "Generator failure (1) or root without dependency edges (23, all \texttt{pnpm})"
  - Ch7:39 — "No production dependencies, or an empty production graph" → "No production dependencies declared"
  - Ch7:16 — "(\texttt{package-lock.json}, \texttt{yarn.lock} or \texttt{pnpm-lock.yaml}), it is not a workspaces monorepo" → "(\texttt{package-lock.json}, \texttt{npm-shrinkwrap.json}, \texttt{yarn.lock} or \texttt{pnpm-lock.yaml}), it is not a monorepo (no \texttt{workspaces} field, \texttt{pnpm-workspace.yaml} or \texttt{lerna.json})"
- Also touches: Ch7:38 ("Workspaces monorepo") can be "Monorepo".

### E16 (from C12) The candidate pool is the ~10,000 most-starred per language, not all repositories ≥ 100 stars — impact: low
- Settled facts: `config.py:14` `MAX_CANDIDATES_PER_LANGUAGE = 10000`. There are 21,915 candidates, the least-starred with 466 stars. Candidate 1,281 has 10,968 stars. No result changes.
- Fix: Ch7:12 — "repositories on GitHub with at least 100 stars that are neither archived nor forks, retrieved through the GitHub search API most-starred first --- 21{,}915 repositories in total" → "repositories on GitHub with at least 100 stars that are neither archived nor forks: the roughly 10{,}000 most-starred of each language returned by the GitHub search API, 21{,}915 repositories in total"
- Also touches: none.

### E17 (from C19) Nothing ties the evaluated commit dfc8e23 to the described 28113e7 — impact: low
- Settled facts: `git diff --stat dfc8e23 28113e7 -- backend src/utils` touches only `.env.example`, `README.md`, `main.py`, `routers/analysis.py` (new), `store/sbom_store.py` and `utils/severity.js` (new). The analysis routers and `vulnerabilityAPI.js` are unchanged. "28113e7" appears nowhere in Chapters/.
- Fix: Ch7:169 — after "SBOM Lens at commit \texttt{dfc8e23}" add "; its analysis code and \texttt{vulnerabilityAPI.js} are unchanged in commit \texttt{28113e7}, the version described in Chapters~4 to~6, which adds only interface features and an endpoint the evaluation does not use"
- Also touches: none.

### E18 (from C18) Smaller cross-chapter mismatches — impact: low (Abstract PT, Ch8:51)
- Fix:
  - Abstract PT :8 — "Em 90 aplicações reais" → "Em 90 aplicações npm reais"
  - Ch8:51 — "where duplicate versions are rarer and dependency trees shallower" → "where duplicate versions are rarer"
  - Ch8:51 — "The evaluation covers npm applications only, at release tags" → "The evaluation covers popular npm applications only, not organised as monorepos, at release tags"
- Also touches: none.

### E19 (from C16) "Five severity bands" vs the four in the ranking and the data — impact: low
- Settled facts: there are four bands in the data and none UNKNOWN. `analyse.py:27` would rank UNKNOWN as 0. The case does not arise.
- Fix: Ch7:59 — "which accounts for the many ties that five severity bands produce" → "which accounts for the many ties that four severity bands produce"
- Also touches: none.

## Viva prep

### V1 (from C17) How were the times measured?
- Likely question: "On what machine, how many repetitions, and what does the impact time include?"
- Answer: each time is one wall-clock HTTP round trip from the pipeline to a local backend, once per SBOM (`lens.py:82-88`). It includes JSON serialisation. The impact row is the sum of one request per vulnerable node, and each request recomputes HITS on the whole graph. The times are end-to-end latencies, which is what a user waits for. They are single measurements, so they indicate scale, not precise cost. `run.json` records no CPU, memory or OS. If the author has the machine specifics, add them to Ch7:140: "measured by the collection pipeline as the wall-clock time of one REST request per analysis (one per vulnerable component for the impact analysis), against a local backend on a single workstation (<CPU, RAM, OS>), once per SBOM". The unmeasured "nearly all of the audit is the unmaintained check" is removed under E11.

### V2 (new, from my check for E1) The exponents depend on the powerlaw version
- Likely question: "Would another version of the fitting library give the same exponents?"
- Answer: no. I ran the same 28113e7 call under powerlaw 1.5 (no upper bound; same Python and NumPy, cause not investigated). x_min differs in 74 of 87. The median α is 2.79 (IQR 2.56–3.36, max 9.43), the median D is 0.047, and the three graphs without a fit get one. So the α row describes powerlaw 2.0.0's fit, which is why Ch7:169 records the library versions. The reading "heavy tail with a few hubs" does not depend on the version. The exact numbers do.

## Holds
- None. Each objection is supported by the code or the data. The defender's partial rebuttals (C6, C8, C9, C10, C11) narrow the fixes but dismiss no item. Two defender corrections were themselves wrong (the C8 cause, and "lighter in none" in C10), and the fixes above avoid them.

## Decisions needed
1. **Power-law exponents (E1/C1/D-new2; affects reported results).** Ch6:170 changes under every option, and the RQ1 conclusion does not change.
   - (A) Treat the 8 as failed fits. `analyse.py` only, from stored outputs, within the SBOM Lens-output rule. Tab. 7.2 α: 87/2.62/2.50/2.84/2.09/3.00 → 79/2.58/2.50/2.78/2.09/2.99. x_min: n 79, Q3 1 → 2. D: n 79, min 0.006 → 0.007. Fig. 7.1 is regenerated. Ch7:77, the Ch7:86 caption 87 → 79, Ch8:12, and E8 → 59 of 79.
   - (B) Keep the 87 and state the bound. Text only (Ch7:77, the Ch7:86 caption, Ch8:12); no number changes. Optionally B′, the refit range 3.03–3.39, labelled as computed outside SBOM Lens.
   - (C) Report the unbounded values. This is not possible from SBOM Lens outputs at 28113e7: it needs a script refit, which breaks the rule, or a patched backend, which leaves 28113e7. If done: α max 3.00 → 3.39, D min 0.006 → 0.007, the figure is regenerated, Ch7:77 says "79 of 87 within 2–3", Ch8:12 says "2.1 and 3.4", and Ch7:169 is extended.
2. **Top-k ties (E3/C7; affects reported results).** (A) Tie-aware expected overlap in `analyse.py`: 66 → 61 and 54 → 47 zeros, Tab. 7.3 Q3 0.12 → 0.16 and 0.25 → 0.24, max 0.67 → 0.58 and 1.00 → 0.70, and E4 uses the k/N baseline. (B) Keep the script and add the tie sentence with the 61–67 and 47–60 ranges; E4 uses the 57/51 baseline. Medians and conclusions are unchanged either way.
3. **Likelihood-ratio counts (E7/C10).** (A) Wording only. (B) Also report LR counts from the stored fits (47 heavier than exponential, 2 lighter, of 87; or 47/2 of 79 under D1-A), computed in `analyse.py` and labelled as not SBOM Lens output. Ch8:49 is then qualified.
4. **Reproducibility claim (E12/C5).** (A) Reword Ch7:4 and Ch7:169 to scope the claim to Tabs. 7.2–7.5 and the figures. (B) Extend `analyse.py` to emit the funnel and every in-text count, and keep the claim. B also hosts any new numbers from D2-A, E4 and D3-B.
