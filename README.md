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

The tool was compared with the statistical lead's hand-calculated figures at the coder-pair level, using the current version of the tool. Across the 18 coder pairs in the 6/18, 7/15, 9/2, and 9/18 rounds, the tool's figure differed from the hand calculation by a mean of 0.029 (median 0.018, range 0.001 to 0.110), and 16 of the 18 pairs differed by 0.05 or less. The tool was lower than the hand calculation for 14 of the 18 pairs. Validation measures how closely the tool reproduces the statistical lead's method. It does not test that method against other IRR software.

Two causes of the remaining differences have been identified:
- The statistical lead scores some free-text fields on meaning, and the tool excludes them (see Free-text exclusion). In the 9/2 round, restricting his scoring to the fields the tool scores, with RQ 368.1 (a rejected paper) removed, brought two of the three pairs within 0.011 of the tool.
- The hand calculation contains errors. In the 9/2 round, two metrics in one research question were scored as agreement and should have been disagreements, which affected the third pair.

The 8/12 round, the first on the Formative Feedback domain and coding sheet template, had low agreement by both methods (overall 0.422 by hand and 0.361 by the tool). Eight of its ten pairs differed by 0.014 to 0.143 (mean 0.074). The other two pairs differed by 0.348 and 0.821, and their hand-calculated figures were not recalculated.

## Known Limitations

- The tool assumes the uploaded sheet is correctly structured and computes on the data it is given. It does not warn when the columns in a file differ from the selected domain's list, so absent columns are silently left out of the calculation, and combined sheets coded with earlier template versions have produced results far from the hand-calculated figures, because columns were renamed, added, and removed between versions. It also cannot detect source data problems, such as reviewer entries lost when rows are added to a coding sheet, or that a paper was rejected. Remove rejected papers' rows before uploading, because their agreeing fields raise alpha (RQ 368.1 raised one 9/2 pair by about 0.012).
- Domain detection counts only how many signature columns match, so a sheet from an earlier template version can be assigned to the wrong domain. A 5/7 sheet whose headers match 199 of Item Generation's 201 columns was detected as Automated Scoring, because it still used an older fairness field name that is in the Automated Scoring signature. The effect on that sheet's alpha was small, but the tool gave no warning. Formative Feedback is separated from Item Generation by a single column, `Feedback usage reported`, so a Formative Feedback sheet without it is detected as Item Generation.
- The Automated Scoring and Formative Feedback templates mark `Metric{n}-Additional evaluation method notes` as used for IRR. The statistical lead does not include them, so the tool excludes them by default.
- Near-duplicate dropdown labels in the coding scheme count as disagreements, even where the statistical lead treats them as equivalent. An example is `ROGUE`, a misspelling of `ROUGE` that appeared in some template versions and was later corrected, so sheets coded on different template versions can disagree on it. This is a coding-scheme issue, not something the calculator can resolve.
- Metric realignment runs only for research questions coded by exactly two coders. On a sheet where every coder codes every research question, such as the 5/7 sheet, metric slots are compared by position, whereas the statistical lead rearranges the metrics by hand to match them. Within a pair, metrics are paired one at a time, best score first, rather than by searching for the best overall set of pairings. This is a reasonable approximation given the small number of metrics typically coded per research question, but could in principle pick a slightly suboptimal pairing when a research question has an unusually large number of metrics.
- When two coders each logged a metric and the two share none of the identifying fields, the tool pairs them as one matched metric and compares them field by field. The statistical lead treats such metrics as two different metrics, each disagreeing on every field. This should not happen when coders complete an initial review together, but it still does. It did not occur in the 9/2 round.
- A coder who skips a group of related fields, such as the fairness fields in RQ 363.1, is a disagreement in the statistical lead's scoring. The tool does not implement this. Because blank and `N/A` are the same answer in non-metric fields, a skipped group of fields against an `N/A` scores as agreement. Implementing it would need a definition of which fields form a group.
- The tool reports alpha for coder pairs only. It does not report a figure for all coders together or for each paper across all coders, which is how the 5/7 round was scored. Percent agreement is not shown alongside alpha.
- The list of multiselect fields is hard-coded: any field whose name ends in `list`, plus a fixed set of fields the coding sheet declares as multiselect. A multiselect field added in a future coding sheet version has to be added to that set, or its choices will be compared in the order entered.
- The statistical lead scores several descriptive fields by meaning (see Free-text exclusion). Those are excluded here, so the tool's alpha will not match his figure on his full field set even when every other rule agrees. The 9/2 gap from this alone was up to about 0.04.

---

*Created by Melanie Kurimchak in collaboration with Claude (Anthropic). Provided for exploratory use — verify results against established IRR software before formal or published reporting.*
