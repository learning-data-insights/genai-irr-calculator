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
- A metric matched to a counterpart is compared field-by-field, with blank / `N/A` / `#VALUE!` (and similar spreadsheet errors) collapsed into one "nothing here" bucket — bucket-vs-bucket is an agreement, bucket-vs-real-value is not.
- A metric slot neither coder used at all is excluded from the calculation entirely.

**Missing values.** Outside of metric realignment, a blank cell on one side compared against a real value on the other counts as a disagreement. A field left blank by both coders is excluded from the calculation — there's nothing to agree or disagree on.

**Numeric equivalence.** Besides the missing-value bucket above, a value entered as a percentage point by one coder and the same quantity as a decimal fraction by the other (e.g. `3.89` vs. `0.0389`) compares as equal — confirmed against cases where Aaron scores these as agreement rather than disagreement.

**Free-text exclusion.** `Research Question`, `Tested LLM Model & Version Used`, `Tested Model & Techniques List`, `Baseline list`, every `Metric{n}-evaluation results` / `Metric{n}-Additional evaluation method notes` field, and every `Metric{n}-min source` / `Metric{n}-max source` / `Metric{n}-min baseline` / `Metric{n}-max baseline` field are exact-string matched and prone to false mismatches from paraphrasing — confirmed against real cases where Aaron scores two very differently-worded descriptions as agreement because they refer to the same underlying thing. They're shown in the column picker with a free-text warning badge but excluded from the default selection. (This does not cover the baseline `...value`, `...difference`, or `...significant` sub-fields, which hold actual numbers/verdicts and are compared normally.)

**Normalization.** Numeric values are normalized so `85`, `85.0`, and `85%` compare correctly. Multiselect/list cells are compared order-independently.

**Overall figure.** The Overall α shown at the top is the **simple average of the per-pair α figures** below it — each coder pair counts equally, regardless of how many research questions it contributed.

## Validation

The methodology has been checked twice against independent rounds of real coding data, both times by reconstructing a coder pair's raw data directly from the source spreadsheet (including Aaron's own cell highlighting as ground truth) and running it through the same recoding, realignment, and bucket-equivalence logic the tool uses:
- **6/18/2026 round:** matched a hand-calculated figure to within 0.0013.
- **7/15/2026 round** (5 coder pairs): three of five pairs matched Aaron's highlighted ground truth within 0.01–0.04. The remaining two — both the pairs with the fewest shared research questions — showed a larger gap (0.04–0.14) that further investigation didn't fully explain; see Known Limitations.

## Known Limitations

- Two coder pairs in the 7/15/2026 validation round (the two with the fewest shared RQs) still show a real, not-fully-explained gap between the tool's output and Aaron's highlighted ground truth, after direct unit-level tracing turned up no further bug. Likely sample-size sensitivity, but not confirmed.
- **Automated Scoring's column definition has not been updated to the current (v0.0.7) field names** and may be stale in the same way Item Generation's was before its update — unconfirmed pending the same kind of Feature Descriptions template used to fix Item Generation and Formative Feedback.
- Formative Feedback's own template marks `Metric{n}-Additional evaluation method notes` as used for IRR, unlike Item Generation. The tool currently still excludes it as free text everywhere; worth confirming with Aaron whether that's the intended behavior for this domain specifically.
- Formative Feedback's auto-detection relies on `Feedback usage reported` as a tiebreaker column against Item Generation (their other signature columns are identical). This is unconfirmed against a real Automated Scoring or Item Generation export — if that column name ever turns up there too, an FF sheet could still misdetect as IG. Worth a quick check the first time a real FF sheet is run through auto-detect.
- Metric realignment uses a greedy best-match assignment, not a formally optimal one. This is a reasonable approximation given the small number of metrics typically coded per research question, but could in principle pick a slightly suboptimal pairing when a research question has an unusually large number of metrics.
- Near-duplicate dropdown vocabulary in the coding scheme itself (e.g. "Item rating" vs. "Item quality rating") will still register as a mismatch — this is a coding-scheme issue, not something the calculator can resolve.
- Source data integrity problems (e.g. dropped reviewer entries when new rows are added to a coding sheet) will still produce a misleading alpha for the affected pair. The tool can't detect this on its own — it can only compute honestly on the data it's given.

---

*Created by Melanie Kurimchak in collaboration with Claude (Anthropic). Provided for exploratory use — verify results against established IRR software before formal or published reporting.*
