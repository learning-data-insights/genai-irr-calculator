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

- **Auto-detection** of coding sheet version (Automated Scoring / Item Generation / Formative Feedback) from signature columns, with manual override. Column-name matching is whitespace/case-tolerant, so stray spaces in real export headers don't cause a column to be silently missed. The version cards and the results header show which template version the tool's column list was built from.
- **Column picker** with free-text fields flagged and excluded from the default selection.
- **Coder filter** to restrict calculation to a subset of coders.
- **Results at three levels**: overall, by coder pair, and an expandable pair → paper → RQ accordion.
- **Export** to CSV or formatted Excel.

## Supported Coding Sheet Templates

The column lists in the tool were last checked against these template versions on 2026-10-01.

| Domain | Template version | Last updated | Columns in the tool | Source of the IRR column list |
|---|---|---|---|---|
| Automated Scoring | 1.5.0 | Not recorded | 161 | Coding Sheet headers and the IRR rule used by the other two domains (see below) |
| Item Generation | 0.0.8 | 2026-07-15 | 201 | "Used for IRR?" column on the Feature descriptions tab |
| Formative Feedback | 0.0.2 | 2026-09-14 | 191 | "Used for IRR?" column on the Feature descriptions tab |

- The Item Generation and Formative Feedback columns include the per-metric `evaluation results` and `Additional evaluation method notes` fields. They appear in the column picker as free-text and are not selected by default.
- The Automated Scoring workbook has no "Used for IRR?" column and no version stamp on its definitions tab. Its version number comes from the last entry in its "Coding sheet history" log, which has no date. Its column list follows the same rule as the other two domains (study design, fairness, research question design and metric fields, without comments, reviewer details and inclusion fields) and matches the Coding Sheet headers exactly.
- The "Required?" column in each template describes what coders must fill in. The calculator does not use it.
- The Field lists tab (dropdown options) matters here in one place only: it identifies which fields allow several choices. See Normalization.

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

**Normalization.** Numeric values are normalized so `85`, `85.0`, and `85%` compare correctly. Multiselect cells are compared order-independently, so `Few-shot, Zero-shot` and `Zero-shot, Few-shot` count as the same answer. The fields treated this way are `AI Innovation`, `Prompting techniques`, `Bias / Fairness Type`, and each metric's `analysis type`, `category` and `truth`. They are the fields marked Multiselection on the coding sheets that can be selected for IRR. `Education Content` is left out on purpose because it is single-select in Automated Scoring and one of its options contains a comma.

**Overall figure.** The Overall α shown at the top is the **simple average of the per-pair α figures** below it — each coder pair counts equally, regardless of how many research questions it contributed.

## Validation

The methodology has been checked twice against independent rounds of real coding data, both times by reconstructing a coder pair's raw data directly from the source spreadsheet (including Aaron's own cell highlighting as ground truth) and running it through the same recoding, realignment, and bucket-equivalence logic the tool uses:
- **6/18/2026 round:** matched a hand-calculated figure to within 0.0013.
- **7/15/2026 round** (5 coder pairs): three of five pairs matched Aaron's highlighted ground truth within 0.01–0.04. The remaining two — both the pairs with the fewest shared research questions — showed a larger gap (0.04–0.14) that further investigation didn't fully explain; see Known Limitations.

## Known Limitations

- Two coder pairs in the 7/15/2026 validation round (the two with the fewest shared RQs) still show a real, not-fully-explained gap between the tool's output and Aaron's highlighted ground truth, after direct unit-level tracing turned up no further bug. Likely sample-size sensitivity, but not confirmed.
- Automated Scoring has no "Used for IRR?" column in its template, so its column list cannot be confirmed against an explicit IRR flag. It matches the Coding Sheet headers exactly and follows the rule used for the other two domains. The Automated Scoring template did not receive the v0.0.7 fairness renames, so its fairness columns are still `Fairness acknowledged` and `Bias / Fairness Evaluated`. Worth confirming with Aaron that this list is the intended IRR set.
- Formative Feedback's template marks `Metric{n}-Additional evaluation method notes` as used for IRR, and still does in v0.0.2. Item Generation marks it as not used. The tool still excludes it as free text for every domain, so it appears unchecked in the picker for Formative Feedback. Worth confirming with Aaron whether Formative Feedback should include it.
- Item Generation's Feature descriptions tab still lists the older names `Fairness acknowledged` and `Bias / Equity Evaluated`. Its Coding sheet headers and version log use `Bias/Fairness acknowledged` and `Bias / Fairness Type`, so the tool follows the headers. The Feature descriptions tab needs the same rename.
- Formative Feedback auto-detection uses `Feedback usage reported`, `Feedback usage numbers` and `Feedback Content` to separate it from Item Generation. None of the three appears in the current Item Generation or Automated Scoring templates. This has been checked against the templates only. The first real Formative Feedback export run through auto-detect should confirm it.
- The order-independent comparison of multiselect fields (see Normalization) has not been validated against Aaron's hand-calculated figures. Earlier validation rounds ran before it applied to any field, so figures for sheets with several selections in one cell may differ slightly from earlier rounds.
- Metric realignment uses a greedy best-match assignment, not a formally optimal one. This is a reasonable approximation given the small number of metrics typically coded per research question, but could in principle pick a slightly suboptimal pairing when a research question has an unusually large number of metrics.
- Near-duplicate dropdown vocabulary in the coding scheme itself will still register as a mismatch. This is a coding-scheme issue that the calculator cannot resolve. One current example is the metric `ROGUE` in the Automated Scoring and Item Generation lists and `ROUGE` in the Formative Feedback list. It only matters if sheets from different domains are combined.
- Source data integrity problems (e.g. dropped reviewer entries when new rows are added to a coding sheet) will still produce a misleading alpha for the affected pair. The tool can't detect this on its own — it can only compute honestly on the data it's given.

---

*Created by Melanie Kurimchak in collaboration with Claude (Anthropic). Provided for exploratory use — verify results against established IRR software before formal or published reporting.*
