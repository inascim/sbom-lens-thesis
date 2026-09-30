# RQ2 compares structural rankings with a severity-band ranking, using Kendall τ-b

RQ2 asks whether a structural view reorders a project's vulnerable components compared with flat-inventory analysis. The flat-inventory baseline is the ranking an analyst actually sees: vulnerable components ordered by their worst **severity band** (CRITICAL > HIGH > MEDIUM > LOW). The five bands produce many ties, so agreement is measured with Kendall's τ-b, which is designed for tied ranks, together with the overlap between the CRITICAL components and the same number of top components by each structural measure. The structural measures are kept to two, **blast-radius size** and **HITS authority**, and one τ-b per SBOM and measure is reported as a distribution over the dataset.

## Considered Options

- Ranking by numeric CVSS v3 score: rejected. The stored findings keep only the band, so this would have meant changing SBOM Lens and recollecting after the dataset was fixed, and records without a v3 score (UNKNOWN) would still have no score. It is also not what a flat inventory shows the analyst.
- Four structural measures (adding dominated-set size and the spectral exposure score): reduced to two to keep the evaluation simple. Dominance is covered in RQ3 through vulnerable chokepoints.
