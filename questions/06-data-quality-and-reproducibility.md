# 6. Data Quality, Validation & Reproducibility

Genomics pipelines involve many processing steps, external databases, and long-running compute — reproducibility and rigorous validation are core professional skills, not afterthoughts.

---

### Q: How would you ensure a genomics analysis pipeline is reproducible, both for yourself six months from now and for a colleague trying to rerun it? 🟡

**Answer:**
- **Version-control all pipeline code**, not just scripts but also configuration files defining parameters, reference genome/annotation versions, and tool versions used.
- **Use containerization or environment-management tools** (e.g., Docker/Singularity, conda environments) to pin exact software versions and dependencies, since bioinformatics tools frequently have subtle version-dependent behavior differences that can silently change results.
- **Use a workflow management system** (e.g., Nextflow, Snakemake, or similar) rather than ad hoc shell scripts, which typically provide built-in provenance tracking, easier parallelization, and more reproducible re-execution.
- **Document and pin exact reference data versions** (reference genome build, annotation version, any external database versions used for annotation/filtering) — an analysis "reproduced" against a different reference or annotation version can produce meaningfully different results without this being obvious unless carefully tracked.
- **Record random seeds** for any stochastic steps (e.g., clustering algorithms with random initialization) where exact reproducibility of a specific run matters.

**Follow-ups:**
- A pipeline you ran a year ago produces slightly different results when rerun today. What are the first things you'd check to understand why?

---

### Q: What is a "truth set" or reference standard in genomics validation, and what are the limitations of relying on one? 🟡

**Answer:**
A truth set is a sample (or set of samples) with an extensively, independently validated and high-confidence set of known variants or known biological ground truth, used as a benchmark to evaluate the sensitivity and precision of a new pipeline or method (e.g., certain widely-used reference materials with deeply characterized variant sets, developed through consortium efforts combining multiple sequencing technologies and validation methods).

Limitations: truth sets are typically limited to a small number of well-studied samples/cell lines, which may not represent the full diversity of real patient samples (different ancestries, different sample types like tumor tissue versus blood, different sample quality/degradation levels) — so strong performance on a standard truth set doesn't guarantee equally strong performance on your specific real-world sample population. Truth sets also often have better characterization in "easy," uniquely-mappable genomic regions than in difficult regions (repetitive sequences, structural variant-prone regions), so benchmark performance can look better than real-world performance specifically in the genomic regions that are hardest and often most clinically important to get right.

**Follow-ups:**
- If your pipeline is going to be applied primarily to a patient population underrepresented in standard truth sets, how would you approach validating it?

---

### Q: How would you investigate whether an unexpected or surprising result in a genomics analysis reflects genuine biology versus a technical artifact? 🟡

**Answer:**
- **Check whether the surprising pattern correlates with a known technical covariate** (batch, sample processing date, sequencing depth, sample quality metrics) before considering biological explanations — technical artifacts are a more common explanation for surprising results than genuine, unexpected biology, and should generally be ruled out first.
- **Look at the raw/intermediate data supporting the surprising result**, not just the final summary statistic — e.g., actually visualize read alignments at a surprising variant call, or look at raw counts underlying a surprising differential expression result, rather than trusting a downstream number in isolation.
- **Check for consistency across replicates or an independent dataset** — does the surprising pattern hold up in a biological or technical replicate, or in an independent cohort/dataset if available?
- **Consider whether the result could be explained by a known confounder** not yet accounted for in the analysis (e.g., an unmeasured demographic or clinical variable correlated with both the "biological" variable of interest and the outcome).
- **Maintain appropriate skepticism proportional to how surprising/novel the claim is** — an unexpected result that would represent a major, exciting biological finding deserves more scrutiny before being reported, not less, given how often such initial exciting findings turn out to be artifacts on closer investigation.

**Follow-ups:**
- Describe a time (real or hypothetical) where a surprising result turned out to be a technical artifact rather than genuine biology. What tipped you off?

---

### Q: What is confounding in the context of a genomics association study, and how would you check for and address it? 🟡

**Answer:**
Confounding occurs when a variable is associated with both the exposure/predictor of interest and the outcome, creating a spurious or distorted apparent association between the two if not properly accounted for. In genomics specifically, common confounders include **population stratification** (systematic ancestry differences between groups being compared, which can create spurious genetic associations if ancestry also correlates with the outcome for reasons unrelated to the specific genetic variants being tested), age, sex, and various technical covariates (batch, sequencing depth).

Checking for and addressing confounding: for population stratification specifically, common approaches include using principal components derived from genome-wide genotype data as covariates in association models (to capture and adjust for ancestry-related population structure), or using specialized mixed-model approaches designed to account for genetic relatedness/structure in a study population. More generally, careful experimental design (matching or randomizing with respect to known confounders where possible) is the strongest safeguard, complemented by including known confounders as covariates in statistical models where design-stage control isn't fully possible.

**Follow-ups:**
- Why is population stratification a particularly important confounder to consider in genetic association studies specifically, compared to many other epidemiological study contexts?

---

### Q: How would you validate that a new bioinformatics tool or updated pipeline version doesn't introduce a regression before adopting it in production? 🟡

**Answer:**
- **Run the new tool/version alongside the existing one** on a representative benchmark set of samples (including known truth-set samples where available, and a sample of real production data reflecting the actual diversity of samples the pipeline will process), comparing outputs directly rather than assuming an updated version is strictly better just because it's newer.
- **Focus comparison on the specific metrics/outcomes that matter for the pipeline's actual use case**, not just a generic aggregate accuracy number — investigate any discordant results individually to understand whether the new version is fixing genuine issues or introducing new ones.
- **Check release notes and known issues carefully** for any tool/pipeline update, since bioinformatics tools sometimes have documented behavior changes between versions (e.g., changes to default parameters) that can silently and meaningfully change results if not accounted for.
- **Maintain a regression test suite** — a fixed set of representative test cases (including known edge cases) that get automatically checked against expected results whenever any pipeline component changes, so future updates are caught systematically rather than requiring a full manual re-validation each time.
- **Roll out gradually** where feasible (e.g., running the new pipeline in parallel with the old one for a period before fully switching over) rather than an abrupt, all-at-once cutover, especially for pipelines feeding into high-stakes downstream decisions.

**Follow-ups:**
- A pipeline update improves sensitivity for detecting a certain variant class but changes default parameters in a way that also affects several other variant classes you weren't specifically testing. How would you catch this?
