# 5. Single-Cell & Multi-Omics Analysis

Single-cell technologies and multi-modal data integration have transformed genomics analysis in the last several years — a near-guaranteed interview topic for many roles.

---

### Q: Walk me through a typical single-cell RNA-seq (scRNA-seq) analysis workflow, from raw count matrix to biological interpretation. 🟡

**Answer:**
A representative workflow:
1. **Quality control:** filter out low-quality cells (e.g., very low total counts/genes detected, suggesting empty droplets or dying cells) and, often, cells with unusually high mitochondrial gene content (a common signal of cellular stress or death), as well as filtering out genes detected in very few cells.
2. **Normalization:** correct for differences in sequencing depth/total counts between cells, since raw counts aren't directly comparable across cells without this correction.
3. **Feature selection:** identify highly variable genes that carry the most informative biological signal for downstream analysis, reducing dimensionality and noise from genes that don't vary meaningfully across cells.
4. **Dimensionality reduction:** typically PCA first (to reduce to a more manageable, denoised set of dimensions), often followed by a non-linear embedding (UMAP or t-SNE) for visualization.
5. **Clustering:** group cells into clusters presumed to represent distinct cell types or states, typically using graph-based clustering methods on the reduced-dimension representation.
6. **Cluster annotation:** assign biological identity to clusters, typically by examining marker genes (genes specifically enriched in each cluster) and comparing them to known cell-type marker gene sets.
7. **Downstream analysis:** differential expression between clusters/conditions, trajectory/pseudotime analysis, cell-cell communication analysis, etc., depending on the biological question.

**Follow-ups:**
- At which step in this workflow would a batch effect across multiple sequencing runs be most likely to distort your results if not corrected for, and how would you check for it?

---

### Q: Why is normalization particularly tricky for single-cell RNA-seq data compared to bulk RNA-seq? 🟡

**Answer:**
Single-cell data has additional sources of technical variation beyond simple sequencing depth differences: **dropout** (genes that are truly expressed in a cell but fail to be detected at all, due to the very small amount of starting RNA material per cell and the inherent inefficiency of capture during library preparation) means single-cell count matrices are extremely sparse, with many true zeros mixed in with "missing" zeros that don't reflect true absence of expression — a challenge bulk RNA-seq (which pools RNA from many thousands of cells, averaging out this stochastic dropout) doesn't face nearly as severely.

This means standard bulk normalization methods don't transfer cleanly, and single-cell-specific normalization approaches need to explicitly account for this sparsity and dropout structure, rather than simply assuming a smoothly scaled version of the bulk normalization problem.

**Follow-ups:**
- What is "dropout" specifically, and how might it lead you to incorrectly conclude a gene isn't expressed in a given cell?

---

### Q: How would you decide whether an apparent cell cluster in your single-cell analysis represents a genuine distinct biological cell type/state, versus a technical artifact? 🔴

**Answer:**
- **Check for known, biologically sensible marker genes** specific to that cluster — does it express genes consistent with a known or biologically plausible cell type/state, evaluated against prior biological knowledge, not just a statistical clustering result in isolation?
- **Check whether the cluster correlates with a technical covariate** (batch, sequencing depth, percent mitochondrial content) rather than a biological one — a cluster that aligns suspiciously well with which sequencing batch a cell came from is a red flag for a technical artifact rather than genuine biology.
- **Assess robustness to clustering parameters** — does the cluster persist across a reasonable range of clustering resolution/parameter choices, or does it appear/disappear with small parameter changes (suggesting it may be an artifact of a specific parameter choice rather than a robust biological signal)?
- **Consider whether it could reflect doublets** (two or more cells accidentally captured and sequenced together as if they were one cell, common in droplet-based single-cell technologies) — a cluster showing co-expression of markers from two otherwise-distinct, biologically incompatible cell types is a classic doublet signature, and dedicated computational doublet-detection tools are commonly run as part of QC.
- **Where feasible, validate orthogonally** — e.g., via an independent experimental technique (flow cytometry, imaging, or a targeted follow-up assay) confirming the proposed cell population actually exists.

**Follow-ups:**
- What computational signatures would specifically suggest a cluster is actually a doublet artifact rather than a genuine rare cell type?

---

### Q: What is multi-omics data integration, and what are the main computational challenges in combining data from different modalities (e.g., scRNA-seq and scATAC-seq) measured on the same or different cells? 🔴

**Answer:**
Multi-omics integration combines multiple types of molecular measurements (e.g., gene expression from RNA-seq, chromatin accessibility from ATAC-seq, protein abundance, DNA methylation) to build a more complete picture of cellular state than any single modality provides alone.

Key challenges:
- **Different modalities have very different statistical properties** (different noise structures, sparsity levels, and dynamic ranges), so naively combining them (e.g., simple concatenation) without modality-appropriate preprocessing can let one modality's technical characteristics dominate the combined analysis inappropriately.
- **Matching cells across modalities when data comes from different cells/experiments** ("unpaired" integration, as opposed to genuinely multi-modal measurements from the exact same individual cells) requires computational methods to infer correspondences between cells measured in different experiments, which is inherently more uncertain than truly paired, simultaneous multi-modal measurement.
- **Batch effects compound across modalities** — each modality can have its own technical batch structure, and integration methods need to correct for these while preserving genuine cross-modal biological relationships, which is a harder problem than single-modality batch correction alone.
- **Interpretability**: a joint low-dimensional representation combining multiple modalities can be harder to interpret biologically than a single-modality analysis, since it's not always clear which modality is driving a given pattern in the integrated representation.

**Follow-ups:**
- Would you trust a computational method that infers cell correspondences between an scRNA-seq dataset and a separately-collected scATAC-seq dataset (from different cells) as much as data collected using a technology that measures both modalities simultaneously in the same cells? Why or why not?

---

### Q: What is pseudotime/trajectory analysis, and what assumption is it making about the underlying biology that may not always hold? 🔴

**Answer:**
Pseudotime/trajectory analysis orders cells along an inferred continuum representing a dynamic biological process (e.g., cellular differentiation, a stress response, or disease progression), based on the assumption that cells captured at a single snapshot in time actually represent different points along a shared, continuous underlying process — since single-cell technologies typically only capture a static snapshot, not a true longitudinal time series of the same cells over time.

A key assumption this makes: that the observed cell population genuinely reflects a single, continuous underlying trajectory (or a definable branching set of trajectories) rather than, for example, several genuinely discrete, non-continuously-related cell states, or a more complex/cyclic biological process (e.g., the cell cycle) that a simple linear/branching trajectory model doesn't represent well. Trajectory inference methods can produce a plausible-looking ordering even when the underlying biology doesn't actually follow a clean continuous trajectory, so results should be interpreted cautiously and, where possible, validated against known biological markers of the process being studied (e.g., known early versus late markers of a differentiation process) rather than trusted as a definitive, unbiased description of the true biological dynamics.

**Follow-ups:**
- How would you sanity-check whether a trajectory inferred from your single-cell data actually reflects the real biological process, rather than being an artifact of the trajectory-inference method's assumptions?
