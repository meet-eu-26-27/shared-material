# BRCA1 variant effect and mechanism 

Everything needed to start the project. Assembled 5 October 2026.

```
brca1_variant_project_plan.pdf                    the project description — read this first
brca1_variant_project_plan.docx                   same, editable
data/
  brca1_clinvar_2026-10-05_predict.tsv            4,606 variants to predict
  brca1_clinvar_2026-10-05_reference_labels.tsv     696 confidently classified variants
  brca1_clinvar_2026-10-05_all.tsv                6,463 records — the full ClinVar pull
  README_brca1_clinvar_dataset.md                 dataset card: provenance, columns, exclusions
```

## The three fixed meetings

| | When | Where |
|---|---|---|
| Kick-off | Friday 16 October 2026, 14:00 | Zoom |
| Mid-term checkpoint | Friday 4 December 2026, 14:00 | Zoom |
| Final workshop | Friday 29 January 2027, full day | In person, Warsaw |

Submission freeze: **Tuesday 26 January 2027**. The Zoom link for both online meetings is in
section 10 of the project description.

## The task in one paragraph

ClinVar holds 4,606 BRCA1 missense variants that nobody can currently classify — 2,674 with
conflicting submissions and 1,932 flatly uncertain. Each team predicts, for every one of them,
whether it damages BRCA1 function and with what confidence, and for those called damaging,
by what mechanism. Predictions are frozen and submitted; some of these variants will be
reclassified by ClinVar over the following year, and the cohort finds out together how it did.

## Starting points

```python
import pandas as pd
pred = pd.read_csv("data/brca1_clinvar_2026-10-05_predict.tsv", sep="\t", comment="#")
ref  = pd.read_csv("data/brca1_clinvar_2026-10-05_reference_labels.tsv", sep="\t", comment="#")
```

Read the header of either TSV (the `#` lines) before writing any code against it — the column
documentation, the transcript, the genome build and the exclusion rules are all in there.

Three things that catch people out, all explained in the dataset card:

1. `ref` and `alt` are **plus-strand** alleles. BRCA1 is on the minus strand, so they are the
   complement of the bases in `hgvs_c`.
2. 47 variants are multi-nucleotide (`len(ref) > 1`). They change a single amino acid, but
   tools that expect one-base VCF records will mishandle them.
3. The ClinVar query matched "missense" on any BRCA1 transcript, so the full file contains 75
   records that are not single amino-acid substitutions in NM_007294.4. They are flagged in
   `variant_class` and already excluded from the prediction set.

## Rules worth knowing before you start

The dataset is **frozen**. Do not re-pull ClinVar during the project — the whole design depends
on predicting against this snapshot and comparing with what ClinVar does later.

Nothing is held back: the confidently classified variants and the multiplexed assay data are
all yours to use. In exchange, each team designs and **declares its own train / validation /
test split before fitting anything**, ships it as a file, and does not revise it. How you cut
the data is part of what is assessed.

A submission may not be the output of a single third-party predictor. Published scores are
welcome as features and required as baselines, but your model must combine at least two
independent kinds of evidence.
