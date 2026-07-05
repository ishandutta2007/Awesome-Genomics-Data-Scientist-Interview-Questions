# 3. Statistical & ML Methods for Genomics

Genomics is a "small n, large p" statistical minefield — high-dimensional data with relatively few samples, making rigorous statistical thinking essential.

---

### Q: What is the multiple testing problem, and how does it specifically show up in genomics analyses like GWAS or differential expression? 🟡

**Answer:**
When you run a huge number of independent statistical tests (e.g., testing millions of genetic variants for association with a trait in a GWAS, or testing tens of thousands of genes for differential expression), the probability of getting at least one false positive purely by chance grows rapidly with the number of tests, even if each individual test uses a standard significance threshold (e.g., p < 0.05).

In genomics this is addressed with corrections: **Bonferroni correction** (dividing the significance threshold by the number of tests) is simple but conservative, and in GWAS specifically, a stricter genome-wide significance threshold (commonly around p < 5×10⁻⁸) is used, reflecting the very large effective number of independent tests across the genome given linkage disequilibrium structure. For differential expression and other genomics contexts, **false discovery rate (FDR) control** (e.g., Benjamini-Hochberg) is more commonly used than strict family-wise error control, since it's less conservative while still providing a principled bound on the expected proportion of false positives among significant results — a reasonable tradeoff when some false positives are tolerable in exchange for greater statistical power to detect true effects.

**Follow-ups:**
- Why is FDR control often preferred over Bonferroni correction for differential expression analysis, and what's the tradeoff?

---

### Q: What is a batch effect, and why is it one of the most persistent and dangerous sources of error in genomics data analysis? 🔴

**Answer:**
A batch effect is systematic, non-biological variation introduced by technical factors related to when/where/how samples were processed — different sequencing runs, different reagent lots, different technicians, different processing dates — that can masquerade as, or obscure, genuine biological signal if not properly accounted for.

It's especially dangerous because: **it can be perfectly confounded with the biological variable of interest** if experimental design isn't careful (e.g., if all "disease" samples happened to be processed in one batch and all "control" samples in another, no statistical correction can fully disentangle batch effect from true biological signal after the fact — this needs to be prevented at the experimental design stage, not fixed downstream). Even when not perfectly confounded, batch effects can be large enough to dominate the actual biological signal of interest in unsupervised analyses (e.g., samples clustering by processing batch rather than by the biology being studied), leading to misleading conclusions if not explicitly checked for and corrected.

Mitigation: randomize sample processing across batches with respect to the biological variable of interest at the experimental design stage (the single most important safeguard); include batch as a covariate in statistical models where a randomized design wasn't fully possible; use batch-correction methods (e.g., ComBat and similar tools) for exploratory/visualization purposes, while being aware that batch correction has real limitations and can occasionally overcorrect and remove genuine biological signal, especially in complex designs.

**Follow-ups:**
- You inherit a dataset where disease and control samples were unfortunately processed in almost entirely separate batches. What would you do, and what could you not fully recover from a purely statistical correction?

---

### Q: What is a GWAS (genome-wide association study), and what does a "hit" (significant association) actually tell you — and not tell you — about causality? 🟡

**Answer:**
A GWAS tests genetic variants (typically common SNPs) across the genome for statistical association with a trait or disease across a large cohort, aiming to identify genomic regions where variation correlates with the trait.

A significant hit tells you there's a statistically robust **association** between variation at that locus and the trait in the studied population. It does **not** automatically tell you: that the specific tested variant is the causal one (it may just be in linkage disequilibrium with the true causal variant — see section 1); the actual biological mechanism connecting the variant to the trait; that the association generalizes to other populations, especially if the study was conducted in a genetically homogeneous cohort (a well-documented limitation, since much of GWAS research historically has been skewed toward populations of European ancestry, limiting generalizability and equity of resulting insights/tools); or the magnitude of individual-level predictive value, since most common-variant GWAS hits individually explain only a small fraction of trait variance, even when statistically robust at a population level.

**Follow-ups:**
- Why does a GWAS conducted primarily in one ancestral population often generalize poorly to other populations, and what does this imply for building equitable polygenic risk models?

---

### Q: What is a polygenic risk score (PRS), and what are its main statistical and practical limitations? 🔴

**Answer:**
A polygenic risk score aggregates the estimated effects of many genetic variants (often thousands to millions, each with a small individual effect) into a single score estimating an individual's genetic predisposition to a trait or disease, typically derived from GWAS summary statistics.

Key limitations:
- **Portability across ancestries:** PRS models trained primarily on one ancestral population (historically often European-ancestry cohorts) tend to have substantially reduced predictive accuracy when applied to individuals from different ancestral backgrounds — a serious, well-documented equity concern for clinical translation of PRS tools.
- **Doesn't capture gene-environment interactions or rare high-effect variants** well, since it's built from common-variant GWAS effect estimates — a PRS is not a complete picture of genetic risk, let alone total (genetic + environmental) risk.
- **Score interpretation and clinical utility remain actively debated:** a given PRS percentile doesn't translate straightforwardly into an actionable clinical risk without careful calibration and validation in the specific population and context it's being used in, and overinterpreting PRS results (e.g., presenting them as more deterministic than they are) is a recognized risk in both clinical and consumer genomics contexts.

**Follow-ups:**
- How would you evaluate whether a PRS developed on one population is appropriate to deploy for a more ancestrally diverse patient population?

---

### Q: What are common approaches to differential expression analysis, and why do RNA-seq count data require specialized statistical models rather than a standard t-test or linear regression? 🟡

**Answer:**
Common tools/approaches (e.g., DESeq2, edgeR, limma-voom) model RNA-seq counts using distributions suited to count data's specific statistical properties, rather than assuming normally distributed continuous data as a standard t-test would.

Why standard approaches fall short: RNA-seq counts are **discrete, non-negative, and typically overdispersed** (variance greater than the mean, unlike a simple Poisson assumption) — a plain t-test assumes roughly normally distributed, homoscedastic data, which doesn't hold well for raw or even simply log-transformed count data, especially for lowly-expressed genes with few counts. Specialized methods typically model counts using a negative binomial distribution (which naturally accommodates overdispersion) and use approaches like empirical Bayes shrinkage to stabilize variance estimates for genes with few replicates — genomics experiments often have very few biological replicates per condition (sometimes just 2-3), so borrowing statistical strength across genes to get more stable variance estimates is an important and specifically-motivated methodological choice, not an arbitrary complexity.

**Follow-ups:**
- Why is having only 2-3 biological replicates per condition (common in genomics due to cost) especially statistically challenging, and how do differential expression tools try to compensate?

---

### Q: How would you approach dimensionality reduction (e.g., PCA, UMAP, t-SNE) for a high-dimensional genomics dataset, and what are common pitfalls in interpreting the results? 🟡

**Answer:**
- **PCA** is a linear technique useful both as an exploratory visualization tool and as a preprocessing step (e.g., using top principal components as covariates to control for large-scale population structure in a GWAS, or batch effects in expression data) — its linear nature makes it more interpretable but less able to capture complex non-linear structure.
- **UMAP and t-SNE** are non-linear techniques primarily used for visualization (e.g., visualizing cell population structure in single-cell data) — they can reveal cluster structure well but are prone to specific interpretive pitfalls.

Common pitfalls: **distances and cluster sizes in UMAP/t-SNE plots are not reliably meaningful** — the apparent size of a cluster or the distance between clusters in the low-dimensional embedding doesn't necessarily correspond to a meaningful quantity in the original high-dimensional space, and over-interpreting these visual properties is a common and serious mistake. Results can also be quite sensitive to hyperparameter choices (e.g., perplexity in t-SNE, neighbor count in UMAP) and to random initialization, so a single embedding shouldn't be over-trusted without some sensitivity checking. Additionally, unwanted technical variation (batch effects, sequencing depth differences) can dominate the top components/dimensions if not corrected for beforehand, leading to a visualization that reflects technical rather than biological structure.

**Follow-ups:**
- A UMAP plot of your single-cell data shows what looks like a clear, distinct cluster of cells. What would you check before concluding this represents a genuine, novel cell population?
