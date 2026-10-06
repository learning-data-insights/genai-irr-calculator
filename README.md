# IRR Calculator

A browser-based Inter-Rater Reliability calculator for the GenAI Evidence Hub systematic literature review. Upload a coding sheet, pick your columns, and get Krippendorff's α — overall, by coder pair, and drilled down to individual papers and research questions. Single self-contained HTML file, no install, all computation happens client-side.

## Getting Started

Open `index.html` in any browser. Upload a coding sheet (`.csv`, `.xlsx`, `.xls`, `.tsv`, or `.txt`), confirm the auto-detected sheet version, choose which columns to include, and click **Calculate IRR**.

## File Requirements

- One row per coder per research question.
- `Research Question ID` identifies the unit being coded (hard-coded as the paper/RQ key).
- `Reviewer` identifies the coder (hard-coded as the coder key).
- `Paper ID` groups research questions under their source paper, for the paper-level drill-down.

## Features

- **Auto-detection** of coding sheet version (Automated Scoring / Item Generation / Formative Feedback) from signature columns, with manual override. Column-name matching is whitespace/case-tolerant, so stray spaces in real export headers don't cause a column to be silently missed.
- **Column picker** with free-text fields flagged and excluded from the default selection.
- **Coder filter** to restrict calculation to a subset of coders.
- **Results at three levels**: overall, by coder pair, and an expandable pair → paper → RQ accordion.
- **Export** to CSV or formatted Excel.

## Methodology

Krippendorff's α is computed by **recoding every (Paper, RQ, Field) unit** so that each unit gets category codes unique to itself — a matching pair of coders shares a code, a mismatching pair gets two distinct codes, and no code is ever reused by any other unit in the dataset. This matches the coding team's validated hand/R-calculated method, and is deliberately different from pooling literal field values as categories: a literal answer that happens to repeat across many unrelated units (e.g. "Yes," or a common dropdown option) would otherwise be treated as one shared category when estimating chance agreement, which distorts alpha in a way that has nothing to do with actual rater agreement.

**Metric realignment.** Metric fields (`Metric1-*` through `Metric10-*`) are matched across the two coders **by content, not by slot position**, before any comparison happens. Two coders often list the same metrics in a different order, and comparing "Metric1" to "Metric1" positionally creates false disagreements. For each research question:
- Each coder's used metric slots are matched to the other coder's by similarity across identifying fields (`analysis type`, `category`, `type`, `assessment tool`, `truth`). Matching always pairs up to the smaller side's count — a match doesn't require positive content overlap; if two coders logged the same number of metrics but described them in entirely different terms, they're still compared slot-for-slot rather than left unmatched.
- A metric with **no counterpart at all** (a genuine count imbalance — one coder logged more metrics than the other) counts as a full disagreement across every one of its sub-fields, regardless of what's actually in them.
- A metric matched to a counterpart is compared field-by-field, with blank / `N/A` / `#VALUE!` (and similar spreadsheet errors) collapsed into one "nothing here" bucket — bucket-vs-bucket is an agreement, bucket-vs-real-value is not. For Item Generation and Formative Feedback only, when one coder reports the metric quantitatively (a real value in `min` or `max`) and the other does not, a blank against an `N/A` in any value field (everything except the five identifying fields) counts as a disagreement. Identifying fields are compared normally, and the rule needs the metric min and max columns to be selected.
- A metric slot neither coder used at all is excluded from the calculation entirely.

**Missing values.** In non-metric fields, blank and `N/A` (and the other missing-value entries above) are the same answer. Two coders with nothing there agree, whether both left it blank, both chose `N/A`, or one did each. Nothing there on one side against a real value on the other is a disagreement. Metric slots that no coder used are excluded, and inside a matched metric the bucket rules above apply.

**Numeric equivalence.** Besides the missing-value bucket above, a value entered as a percentage point by one coder and the same quantity as a decimal fraction by the other (e.g. `3.89` vs. `0.0389`) compares as equal — confirmed against cases where the statistical lead scores these as agreement rather than disagreement. This applies only to fields whose names contain `min`, `max`, or `difference` (a metric's min and max, and the baseline value and difference fields that belong to them), so unrelated counts such as `Total number of models tested` are never treated as equal because they differ by a factor of 100.

**Free-text exclusion.** `Research Question`, `Tested LLM Model & Version Used`, `Tested Model & Techniques List`, `Baseline list`, every `Metric{n}-evaluation results` / `Metric{n}-Additional evaluation method notes` field, and every `Metric{n}-min source` / `Metric{n}-max source` / `Metric{n}-min baseline` / `Metric{n}-max baseline` field are exact-string matched and prone to false mismatches from paraphrasing — confirmed against real cases where the statistical lead scores two very differently-worded descriptions as agreement because they refer to the same underlying thing. They're shown in the column picker with a free-text warning badge but excluded from the default selection. (This does not cover the baseline `...value`, `...difference`, or `...significant` sub-fields, which hold actual numbers/verdicts and are compared normally.)

**Normalization.** Numeric values are normalized so `85`, `85.0`, and `85%` compare correctly. Multiselect cells are compared order-independently, so "X, Y" equals "Y, X". This covers any column name ending in "list" and the fields the coding sheet declares as multiselect: `Metric{n}-analysis type`, `Metric{n}-category`, `Metric{n}-truth`, `AI Innovation`, `Bias / Fairness Type`, `Prompting techniques`, `Education Segment`, `Education Content`, `Training source`, and `Testing source`.

**Overall figure.** The Overall α shown at the top is the **simple average of the per-pair α figures** below it — each coder pair counts equally, regardless of how many research questions it contributed.

## Validation

The methodology has been checked three times against independent rounds of real coding data, each time by reconstructing a coder pair's raw data directly from the source spreadsheet (including the statistical lead's own cell highlighting as ground truth) and running it through the same recoding, realignment, and bucket-equivalence logic the tool uses:
- **6/18/2026 round:** matched a hand-calculated figure to within 0.0013.
- **7/15/2026 round** (5 coder pairs): three of five pairs matched the statistical lead's highlighted ground truth within 0.01–0.04. The remaining two — both the pairs with the fewest shared research questions — showed a larger gap (0.04–0.14) that further investigation didn't fully explain; see Known Limitations.
- **9/2/2026 round** (3 coder pairs): on the fields the tool scores, and with RQ 368.1 (a rejected paper) removed, the tool is within 0.011 of the statistical lead's scoring for all three pairs (Pair A 0.706 vs 0.717, Pair B 0.598 vs 0.601, Pair C 0.938 vs 0.942; Pair C includes the statistical lead's own coding). The statistical lead's own reported figures (0.754, 0.609, 0.945) also include free-text fields the tool excludes. He confirmed two metric sections in RQ 367.1 were scored too leniently, so his Pair B figure needs recalculating.

## Known Limitations

- Two coder pairs in the 7/15/2026 validation round (the two with the fewest shared RQs) still show a real, not-fully-explained gap between the tool's output and the statistical lead's highlighted ground truth, after direct unit-level tracing turned up no further bug. Likely sample-size sensitivity, but not confirmed.
- **Automated Scoring's column definition has not been updated to the current (v0.0.7) field names** and may be stale in the same way Item Generation's was before its update — unconfirmed pending the same kind of Feature Descriptions template used to fix Item Generation and Formative Feedback.
- Formative Feedback's own template marks `Metric{n}-Additional evaluation method notes` as used for IRR, unlike Item Generation. The tool currently still excludes it as free text everywhere; worth confirming with the statistical lead whether that's the intended behavior for this domain specifically.
- Formative Feedback's auto-detection relies on `Feedback usage reported` as a tiebreaker column against Item Generation (their other signature columns are identical). This is unconfirmed against a real Automated Scoring or Item Generation export — if that column name ever turns up there too, an FF sheet could still misdetect as IG. Worth a quick check the first time a real FF sheet is run through auto-detect.
- Metric realignment uses a greedy best-match assignment, not a formally optimal one. This is a reasonable approximation given the small number of metrics typically coded per research question, but could in principle pick a slightly suboptimal pairing when a research question has an unusually large number of metrics.
- Near-duplicate dropdown vocabulary in the coding scheme itself (e.g. "Item rating" vs. "Item quality rating") will still register as a mismatch — this is a coding-scheme issue, not something the calculator can resolve.
- Source data integrity problems (e.g. dropped reviewer entries when new rows are added to a coding sheet) will still produce a misleading alpha for the affected pair. The tool can't detect this on its own — it can only compute honestly on the data it's given.
- A coder who skips a whole section is a disagreement in the statistical lead's scoring (for example, the fairness fields in RQ 363.1). The tool does not implement this. Because blank and `N/A` are the same answer in non-metric fields, a skipped section against an `N/A` scores as agreement. Implementing it needs a definition of "section".
- Metrics that share none of the identifying fields are still paired as one matched metric, based on a 6/18 example. Whether the statistical lead would count them instead as two unmatched metrics is unconfirmed.
- The tool cannot tell that a paper was rejected. Remove rejected papers' rows before uploading, otherwise their agreeing fields raise alpha (RQ 368.1 raised one 9/2 pair by about 0.012).
- The qualitative-vs-quantitative rule applies only when Item Generation or Formative Feedback is the detected or selected version, and only when the metric min and max columns are selected.
- The list of multiselect fields is hard-coded. A new multiselect field in a future coding sheet version has to be added to it, or its choices will be compared in the order entered.
- The statistical lead scores several descriptive fields by meaning (see Free-text exclusion). Those are excluded here, so the tool's alpha will not match his figure on his full field set even when every other rule agrees. The 9/2 gap from this alone was up to about 0.04.

---

*Created by Melanie Kurimchak in collaboration with Claude (Anthropic). Provided for exploratory use — verify results against established IRR software before formal or published reporting.*
