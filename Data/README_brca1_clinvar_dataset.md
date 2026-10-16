# BRCA1 missense variants — frozen ClinVar dataset

Dataset for the student project *Predicting the effect of missense variants in BRCA1*.
**Pulled 5 October 2026. Frozen — do not refresh it during the project.** The whole point of
the freeze is that ClinVar keeps moving: predictions made against this snapshot can be
checked later against reclassifications that appear after it.

## Files

| File | Rows | What it is |
|---|---|---|
| `brca1_clinvar_2026-10-05_all.tsv` | 6,463 | everything the query returned, with a `project_set` column saying what each record is for |
| `brca1_clinvar_2026-10-05_predict.tsv` | **4,606** | **the prediction set** — this is what every team submits predictions for |
| `brca1_clinvar_2026-10-05_reference_labels.tsv` | 696 | variants with a confident ClinVar label (232 P/LP, 464 B/LB), for calibration and sanity checks |

All three are tab-separated, UTF-8, with a commented provenance header (lines starting `#`)
followed by one column-name row and then the data. Same 22 columns everywhere.

```python
import pandas as pd
df = pd.read_csv("brca1_clinvar_2026-10-05_predict.tsv", sep="\t", comment="#")
```

## Provenance

* Source: ClinVar (NCBI), web search API `www.ncbi.nlm.nih.gov/clinvar/data/search/`
* Query: `BRCA1[gene] AND "missense variant"[molecular consequence]`
* Reference transcript `NM_007294.4`, protein `NP_009225.1` (1,863 aa)
* Genome build GRCh38, chromosome 17, 1-based positions in `grch38_pos`
* `ref`/`alt` are **plus-strand** alleles; BRCA1 is transcribed from the minus strand, so they
  are the complement of the bases in `hgvs_c` (checked: 0 mismatches across all 6,388
  substitutions)

## Columns

`variation_id`, `accession`, `name`, `hgvs_c`, `hgvs_p`, `aa_ref`, `aa_pos`, `aa_alt`, `chr`,
`grch38_pos`, `ref`, `alt`, `canonical_spdi`, `germline_classification`, `review_status`,
`review_stars`, `last_evaluated`, `n_submissions`, `dbsnp_id`, `conditions`, `variant_class`,
`project_set`. Each file's header documents them line by line.

`aa_ref` / `aa_pos` / `aa_alt` are the one-letter reference amino acid, the residue number and
the variant amino acid — the three fields most prediction tools actually want. They were
derived from `hgvs_p` and independently cross-checked against the codon implied by `hgvs_c`
(0 disagreements).

## What got excluded, and why

The ClinVar query matches "missense variant" on *any* BRCA1 transcript, so not every hit is a
single amino-acid substitution in `NM_007294.4`:

| `variant_class` | n |
|---|---|
| `single_aa_substitution` | 6,388 |
| `synonymous_on_NM_007294.4` | 44 |
| `not_protein_altering_on_NM_007294.4` | 14 |
| `multi_codon_delins` | 9 |
| `nonsense_on_NM_007294.4` | 8 |

The 75 non-substitutions are marked `project_set = excluded_not_single_aa_substitution`. A
further 1,086 substitutions have no usable ClinVar classification (1,085 with none at all, 19
"not provided") and are marked `unclassified_excluded` — they are kept in the `_all` file in
case anyone wants them, but they are not part of the exercise.

47 of the 6,388 substitutions are **multi-nucleotide variants** (`hgvs_c` is a `delins` or an
`inv`, `ref`/`alt` are 2–5 bases). They change one amino acid, but tools that consume one-base
VCF records will mishandle them. Find them with `len(ref) > 1`.

## The prediction set, in numbers

4,606 variants: 2,674 *Conflicting classifications of pathogenicity* and 1,932 *Uncertain
significance*, spread over 1,778 distinct residues from position 2 to 1,863.

Review confidence is thin, which is the honest situation and part of what makes the problem
interesting: 3,879 are 1-star, 690 are 2-star, 26 are 0-star and only 11 carry an expert-panel
review.

Position coverage against the approximate UniProt P38398 feature boundaries:

| region | n |
|---|---|
| RING, 24–64 | 84 |
| coiled coil, 1391–1424 | 81 |
| BRCT1, 1646–1736 | 207 |
| BRCT2, 1756–1855 | 215 |
| outside these four | 4,019 |

So roughly seven out of eight variants in the prediction set sit in regions with no resolved
structure and no crisp domain assignment. Any approach that only works inside the RING and
BRCT domains will cover about 12% of the set — worth knowing before a team commits to a
structure-only pipeline.

## Reference labels

`brca1_clinvar_2026-10-05_reference_labels.tsv` holds the 696 substitutions ClinVar does call:
232 pathogenic-side (71 Pathogenic, 108 Likely pathogenic, 53 Pathogenic/Likely pathogenic)
and 464 benign-side (153 Benign, 293 Likely benign, 18 Benign/Likely benign). Use them for
calibration and for catching a pipeline that is pointing the wrong way.

Treat them with suspicion as a benchmark, though: many published predictors were trained on
ClinVar, so good performance here can be memorisation rather than prediction. The project
plan's evaluation section has the longer version of this argument.
