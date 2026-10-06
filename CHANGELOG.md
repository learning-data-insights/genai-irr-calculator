# Changelog

## 2026-10-06 — 9/2 validation: nonmetric missing values, multiselect order, quantitative vs qualitative metrics

Validated against the 9/2/2026 round (three coder pairs: Alexander × Chris, Heeryung × Maggie, Aaron × Nidhi) using Aaron's own 1/0 scoring sheet. His flattened 1/0 block reproduces his reported alphas (0.754, 0.609, 0.945) with the same recoding the tool uses, so his scoring could be compared with the tool's unit by unit. Three changes follow from what that comparison found and from Aaron's answers about how he scores.

### Changed
- **Nonmetric fields that every coder left blank or marked N/A are now kept and scored as agreement.** Previously they were dropped from the calculation. Per Aaron, blank and N/A are the same answer in nonmetric fields: two coders with nothing there (both blank, both N/A, or one of each) agree, and nothing there against a real answer is a disagreement. The "every coder left it blank" pruning step now applies only to metric fields (`Metric{n}-...` and `MetricMatch{n}-...`), where it removes metric slots nobody used. On the 9/2 sheet this restored 4, 8, and 7 agreements per pair, all in `Bias / Fairness Type`, where N/A is a common dropdown choice, and moved the pair figures by less than 0.006.
- **Order-independent comparison now covers every multiselect field, not only column names ending in "list".** Added: `Metric{n}-analysis type`, `Metric{n}-category`, `Metric{n}-truth`, `AI Innovation`, `Bias / Fairness Type`, `Prompting techniques`, `Education Segment`, `Education Content`, `Training source`, and `Testing source`, matching the coding sheet's own "Multiselection" data type. Previously "Evaluative Review, Pilot testing" and "Pilot testing, Evaluative Review" counted as a disagreement. Aaron scores them as agreement. On the 9/2 sheet this moved Alexander × Chris from 0.688 to 0.706 and Heeryung × Maggie from 0.706 to 0.713.

### Added
- **Qualitative vs quantitative metrics (Item Generation and Formative Feedback only).** A metric counts as quantitative when the coder entered a real value in its min or max field, including text such as "mean=4 (sd=3.34)". When the two coders disagree on this for a matched metric, the sections are still the same metric, but a blank against an N/A in any value field (everything except `analysis type`, `category`, `type`, `assessment tool`, and `truth`) is a disagreement. Identity fields are compared normally. When both coders are quantitative, or neither is, blank against N/A remains an agreement. The rule is off for Automated Scoring and for sheets with no detected or selected version, and it needs the metric min and max columns to be selected. Records now keep whether a missing cell was truly blank or an explicit N/A-type entry, which this rule needs. On the 9/2 sheet it moved Heeryung × Maggie from 0.713 to 0.598 and left the other two pairs unchanged.
- Considered and rejected: scoring every value field as a disagreement in a mixed pair. It follows the literal wording of Aaron's rule but overshot his scoring by up to 0.03, while the narrower blank-vs-N/A rule stays within 0.011 of it.

### Validation
Aaron's 1/0 rows, restricted to the fields the tool scores (so that free-text fields the tool excludes do not count), with two metric sections in RQ 367.1 rescored as disagreements at Aaron's direction, against the tool's output. Aaron × Nidhi leaves out RQ 368.1, a rejected paper that his sheet does not contain.

| Pair | Tool before | Tool now | Aaron, tool's fields | Gap now |
|---|---|---|---|---|
| Alexander × Chris | 0.682 | 0.706 | 0.717 | −0.011 |
| Heeryung × Maggie | 0.702 | 0.598 | 0.601 | −0.002 |
| Aaron × Nidhi | 0.949 | 0.938 | 0.942 | −0.004 |

Aaron's own reported figures were 0.754, 0.609, and 0.945. The difference from the "tool's fields" column is mostly the free-text fields he scores by meaning (about 0.037 for Alexander × Chris and 0.019 for Heeryung × Maggie). Aaron confirmed that RQ 367.1 metrics 5 and 6 should have been scored as disagreements, so his Heeryung × Maggie figure was too high and needs recalculating.

### Known open items
- **A coder who skips a whole section.** Aaron scores this as a disagreement across the section (for example, the fairness fields in RQ 363.1). The tool does not implement it: because blank and N/A are the same answer in nonmetric fields, a skipped section against an N/A in a dependent field scores as agreement. Implementing it needs a definition of "section".
- **How unrelated metrics are paired is unconfirmed.** The tool pairs metrics that share none of the identifying fields as one matched metric, based on a 6/18 example. Aaron has not yet said whether he would count them as one matched metric or as two unmatched ones.
- **Rejected papers.** The combined coding sheet still contains rejected papers, and the tool cannot tell (the Inclusion Decision column read "Strong yes" or "Weak yes" for RQ 368.1). Remove their rows before uploading. Including RQ 368.1 raised Aaron × Nidhi by about 0.012.
- Alexander × Chris is still 0.011 below Aaron on the same fields. The difference is concentrated in RQs 353.1 and 356.1 and is probably metric pairing, but it has not been traced.
- The multiselect field list is hard-coded. A new multiselect field in a future coding sheet version needs to be added to it.

## 2026-09-30 — Main file renamed to index.html

Renamed `irr_calculator.html` to `index.html` so the tool is served at the repository root when hosted (for example, on GitHub Pages). No code changes: the tool does not reference its own filename, and exported files (`irr_results.csv`, `irr_results.xlsx`) are named independently of it.

### Changed
- Main tool file is now `index.html`.
- README "Getting Started" now says to open `index.html`.
- Links, bookmarks, or documents that point to `irr_calculator.html` need to be updated to `index.html`.

## 2026-09-02 — Formative Feedback enabled in the UI, IG/FF auto-detect tiebreak

Confirmed the 191-field Formative Feedback column definition (added 2026-07-15) against a fresh copy of the domain's Feature Descriptions template — exact match, no additions or removals needed. However, the UI itself still had the Formative Feedback option locked out from before that column definition existed, which meant the completed backend was unreachable.

### Fixed
- **Formative Feedback card was hardcoded disabled.** The version-selector card carried a `disabled` class with a "Soon" badge and "Not yet available" subtitle left over from before the FF column definition was populated. Card is now active like Automated Scoring and Item Generation.
- **Formative Feedback couldn't be selected at all.** `renderVersionCards()` only attached a click handler to the Automated Scoring and Item Generation cards (`if (k !== 'ff')`), so clicking the FF card did nothing even once its `disabled` styling was removed. Now attaches to all three.
- **Auto-detection could never surface Formative Feedback, even after the above fixes.** Item Generation and Formative Feedback had byte-identical `signature` arrays (`Metric1-analysis type`, `Metric1-category`, `Bias / Fairness Type`). `detectVersion` keeps the first version to reach the top score on ties, and `ig` is declared before `ff` in `SHEET_VERSIONS`, so any FF sheet would have silently auto-detected as Item Generation. Added `Feedback usage reported` — a Formative-Feedback-only column from the domain's Feature Descriptions template (Research Question Design section, not itself used for IRR) — to FF's signature as a tiebreaker. Unconfirmed whether this column name is guaranteed absent from real Automated Scoring / Item Generation exports; worth checking the auto-detect banner the next time an actual FF sheet is uploaded.

## 2026-07-15 — Multi-round validation, metric-matching fix, schema updates for two more domains

Validated the tool against a second, independent round of real coding data (7/15/2026, five coder pairs) using Aaron's actual cell highlighting as ground truth, the same method used to validate the previous release. That process surfaced one more real algorithm bug and several coding-scheme schema updates.

### Added
- **Descriptive metric fields excluded as free text.** `Metric{n}-min source`, `Metric{n}-max source`, `Metric{n}-min baseline`, and `Metric{n}-max baseline` (for any metric number) added to the free-text exclusion list. Confirmed against Aaron's own designation rows: he scores these as agreement based on whether they describe the same underlying thing (e.g. "KNN(all without subtopic)" vs. "KNN-all~T", or "Neuroticism subscale" vs. "neuroticism"), not on exact wording — a paraphrase judgment no exact-match tool can replicate. Excluding them avoids manufacturing disagreements Aaron never made. (Note: this does *not* cover `...baseline value`, `...baseline difference`, or `...baseline significant` — those hold actual numbers/verdicts and are still compared normally.)
- **Percent-vs-decimal numeric equivalence.** A numeric field entered as a percentage point by one coder and the same quantity as a decimal fraction by the other (e.g. `3.89` vs. `0.0389`) now compares as equal. Confirmed against Aaron's own designation rows, which score these as agreement rather than disagreement.
- **Formative Feedback column definition.** This sheet version previously had an empty placeholder column list. Populated with the 191 fields marked "Used for IRR? = Y" in the domain's Feature Descriptions template, with the same v0.0.7 field renames applied as Item Generation (see below). Flagged for follow-up: this domain's template marks `Metric{n}-Additional evaluation method notes` as used for IRR, unlike Item Generation — currently still excluded as free text pending confirmation with Aaron on whether that's intentional or another documentation lag.
- Whitespace/case-tolerant column-name matching (`normColName`) applied consistently across version auto-detection, default column pre-selection, and the "does this metric column have any data" check. Real exports have contained stray leading/trailing spaces on headers (e.g. `"Domain "`, `"Metric1-min baseline "`); these no longer cause a column to be silently treated as absent or empty.

### Fixed
- **Metric matching no longer requires positive content overlap to pair two metrics.** Previously, if two coders logged the *same number* of metrics but described them in completely different terms (zero shared identity-field content), the matcher left them "unmatched," which triggered the harsher forced-full-disagreement rule intended for genuine count imbalances. Confirmed against a real case where two coders' single metric shared no identity-field agreement at all, yet Aaron scored it as an ordinary (if total) mismatch, not a forced-absence disagreement. The matcher now always pairs up to the smaller side's count, falling back to same-slot-number pairing when content gives no signal; "unmatched" is reserved for a metric with no counterpart slot left at all.
- `getMetricColsWithData` was doing an exact-key lookup against raw row data, which could silently fail (and wrongly conclude a column had no data anywhere) if the actual header had different whitespace than the version definition. Now resolves through the same whitespace-tolerant matching as everything else.

### Changed
- **Item Generation field names updated to v0.0.7.** `Fairness acknowledged` → `Bias/Fairness acknowledged`, `Bias / Fairness Evaluated` → `Bias / Fairness Type`, confirmed against the domain's Feature Descriptions template ("Used for IRR?" column) cross-referenced with the coding sheet's own version-history log.

### Known open items
- **Automated Scoring's field names have not been updated** and still reference the pre-v0.0.7 names. Unconfirmed whether this domain received the same rename — needs the same kind of Feature Descriptions template used for Item Generation and Formative Feedback before it can be updated with confidence.
- Two of the five 7/15 coder pairs (the two with the fewest shared research questions) still show a real gap between the tool's output and Aaron's highlighted ground truth (~0.04–0.14) after this round of fixes. No further bug was identified after direct unit-level tracing; flagged as likely sample-size sensitivity rather than a known-and-unfixed defect, but not fully explained.

### Validation
Re-ran the full validation process against the 7/15/2026 round (5 coder pairs, 6 coders, 36 RQs), reconstructing each pair's alpha independently from Aaron's actual cell highlighting and comparing to the tool's output. Three of five pairs landed within 0.01–0.04 of ground truth; see "Known open items" above for the two that didn't.

## 2026-07-14 — Metric realignment, corrected missing-value handling, validated against hand-calculated figures

This is the most significant methodology change since the tool's initial pooled-coincidence-matrix engine. Every item below was diagnosed against the coding team's actual hand-calculated IRR sheet and confirmed field-by-field before implementation.

### Added
- **Metric realignment.** Metric fields are now matched across coders by content (via shared identifying fields — analysis type, category, type, assessment tool, truth) before comparison, instead of comparing `Metric1` to `Metric1` by slot position. Two coders who list the same metrics in a different order no longer produce false disagreements.
- **Forced disagreement for unmatched metrics.** A metric identified by one coder with no counterpart on the other side now counts as a full disagreement across all of its sub-fields, regardless of the sub-fields' own content — matching the coding team's actual practice.
- **Missing-value equivalence bucket.** Within a matched metric, blank / `N/A` / `#VALUE!` (and other spreadsheet error strings) are now treated as one equivalence class: bucket-vs-bucket is scored as agreement, bucket-vs-real-value as disagreement.
- **Blank-vs-filled scored as disagreement.** A field left blank by one coder but filled in by the other is now counted as a genuine disagreement. Previously it was silently excluded from the calculation as "missing data," which understated disagreement wherever this occurred.
- `#VALUE!`, `#N/A`, `#REF!`, and `#DIV/0!` added to the recognized missing-value tokens.
- `Metric{n}-evaluation results` and `Metric{n}-Additional evaluation method notes` (for any metric number) added to the free-text exclusion list — confirmed these are never included in the coding team's own manually-curated IRR column set.

### Changed
- **Overall figure is now a simple average of the per-pair α values**, not a single pooled calculation across every unit from every pair. Previously, a coder pair with more coded research questions had disproportionate influence on the Overall figure; now every pair counts equally, matching how the figure is actually used and reported.
- The tool now reports a **single Krippendorff's α** using the recoding methodology described above. The previous "pooled" calculation (literal field values as categories, matching a since-superseded reference script) has been removed from both the UI and the underlying computation — it was a secondary/reference figure that wasn't being used.

### Removed
- The "pooled" alpha calculation and its accompanying second column/value everywhere it was displayed (summary card, coder-pair table, paper/RQ drill-down, CSV and Excel exports).
- Vestigial column-type detection (nominal/ordinal/interval badges and manual override buttons) — confirmed dead: the underlying calculation has used nominal exact-match for every field regardless of badge for some time, so the badges no longer reflected anything the tool actually did.
- An entire disconnected metric-selector UI (Krippendorff's Alpha / Pairwise % / Cohen's κ / Fleiss' κ checkboxes) — confirmed none of the latter three were ever wired to any displayed output, and deselecting even the alpha checkbox had no effect on what was calculated.
- Several other dead functions and unused CSS left over from earlier iterations of the tool (old matrix-based pairwise agreement helper, unused parsing helpers, superseded accordion styling).

### Validation
Reconstructed a coder pair's raw coding data directly from the source spreadsheet and ran it through the current recoding + realignment + bucket-equivalence logic. Result matched the pair's hand-calculated α to within 0.0013 (0.6743 calculated vs. 0.673 hand-calculated) — the closest the tool has come to reproducing a human reference figure since this project began.

## Earlier

- Initial release: pooled coincidence-matrix engine, matched against a reference Python script (`compute_irr.py`) using nominal exact-match for all field types.
- Investigated and abandoned a metric-slot-alignment approach (v2) after concluding slot mismatches were "too rare to matter" — later revisited and disproven by the findings above, which led directly to this release's realignment work.
- Diagnosed and documented a data-integrity issue (dropped reviewer entries) affecting one coder pair's results.
- Added free-text field exclusion for narrative columns (Research Question, Tested LLM Model & Version Used, Tested Model & Techniques List, Baseline list).
