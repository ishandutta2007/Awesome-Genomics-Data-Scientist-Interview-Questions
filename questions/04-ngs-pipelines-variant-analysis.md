# 4. NGS Pipelines & Variant Analysis

The practical, end-to-end mechanics of turning raw sequencing data into interpretable, trustworthy variant calls.

---

### Q: Walk me through a standard germline variant calling pipeline, from raw FASTQ files to a final annotated VCF. 🟡

**Answer:**
A representative pipeline (specific tools vary, but the stages are broadly standard):
1. **Quality control on raw reads:** assess read quality, adapter contamination, and other issues (e.g., using a tool like FastQC), and trim/filter low-quality bases or adapter sequences as needed.
2. **Alignment:** map reads to a reference genome (e.g., using an aligner like BWA-MEM for short reads), producing a SAM/BAM file.
3. **Post-alignment processing:** mark/remove PCR duplicates (reads that originated from the same original DNA fragment amplified during library prep, which shouldn't be double-counted as independent evidence), and optionally perform base quality score recalibration.
4. **Variant calling:** identify positions where the sample differs from the reference (e.g., using a caller like GATK HaplotypeCaller or similar tools), producing a raw VCF with candidate variants and quality metrics.
5. **Variant filtering:** apply hard filters or a trained filtering model to remove likely artifacts based on quality metrics (depth, mapping quality, strand bias, etc. — see section 2).
6. **Annotation:** annotate the filtered variants with functional consequence (which gene/transcript, coding effect), population allele frequency (e.g., from reference databases), and other relevant metadata to support interpretation.

Understanding this pipeline holistically matters because a genomics data scientist needs to know where in this chain a given quality issue is likely to originate, rather than treating the whole thing as an opaque black box that either "works" or "doesn't."

**Follow-ups:**
- At which stage of this pipeline would you look first if you noticed an unusually high rate of false-positive variant calls specifically in regions of segmental duplication?

---

### Q: Why is marking (or removing) PCR duplicates an important step in NGS data processing, and what could go wrong if you skipped it? 🟡

**Answer:**
During library preparation, DNA fragments are typically amplified (PCR-amplified) before sequencing, meaning multiple sequencing reads can originate from the exact same original DNA molecule rather than representing independent sampling of the genome. If these duplicate reads aren't identified and appropriately down-weighted/removed, they can be mistakenly treated as independent supporting evidence for a variant call, artificially inflating confidence in that call (or, in RNA-seq contexts, artificially inflating apparent expression levels for a gene).

Skipping this step risks: **false-positive variant calls** driven by a PCR error that got amplified and appears in many "reads" that are actually all descendants of one original erroneous molecule, and **general miscalibration of quality/confidence metrics** throughout downstream analysis, since most tools' statistical models implicitly assume reads represent independent observations.

**Follow-ups:**
- How does duplicate marking differ between PCR-based library preparation and PCR-free protocols, or between DNA-seq and single-cell RNA-seq (where unique molecular identifiers, UMIs, are often used)?

---

### Q: What are unique molecular identifiers (UMIs), and what problem do they solve that standard duplicate marking doesn't fully address? 🟡

**Answer:**
UMIs are short, random nucleotide sequences attached to individual DNA/RNA molecules before PCR amplification, so that all amplified copies of the same original molecule carry an identical UMI tag. This allows duplicates to be identified with much higher confidence than standard duplicate-marking approaches (which typically rely on alignment position alone, assuming reads starting at the exact same genomic coordinates are duplicates) — because two genuinely independent original molecules can coincidentally align to the same position (especially likely for lowly-expressed genes or shallow sequencing), and position-based duplicate marking alone can't distinguish this coincidence from a true PCR duplicate, whereas the UMI tag can.

This is especially important in **single-cell RNA-seq and other low-input applications**, where the number of original molecules per gene/cell can be very small, making the risk of falsely merging (or falsely failing to merge) reads based on position alone much more consequential for accurate quantification.

**Follow-ups:**
- Why does UMI-based deduplication matter more for low-input/single-cell applications than for typical bulk DNA sequencing?

---

### Q: How would you evaluate the quality of a new variant calling pipeline before trusting it for production use? 🔴

**Answer:**
- **Benchmark against a well-characterized truth set** — e.g., a sample with an extensively validated, high-confidence set of known variants (such as reference materials from genome-in-a-bottle-style consortium efforts), computing standard metrics like sensitivity (recall) and precision against this known truth.
- **Stratify performance by variant type and genomic context** — a pipeline's performance often varies substantially between SNPs versus indels, and between "easy," uniquely-mappable regions of the genome versus difficult regions (repetitive sequences, segmental duplications, GC-extreme regions) — an aggregate accuracy number can hide serious weaknesses concentrated in specific, often clinically important, genomic contexts.
- **Compare against an existing, trusted pipeline** on real production-representative samples, investigating any discordant calls to understand whether the new pipeline is finding genuine improvements or introducing new errors.
- **Assess reproducibility** — does the pipeline produce consistent results when re-run on technical replicates or when minor, non-meaningful environmental differences (software versions, compute environment) are introduced? Unexpected non-determinism is itself a red flag worth investigating.
- **Validate a sample of calls experimentally** (e.g., via orthogonal confirmation methods like Sanger sequencing for a subset of variants) where the stakes justify the additional cost, particularly for any pipeline whose output will inform clinical or other high-stakes decisions.

**Follow-ups:**
- Your new pipeline shows higher overall sensitivity than the old one but at the cost of somewhat lower precision. How would you decide whether this is an acceptable tradeoff?

---

### Q: What's the difference between hard filtering and machine-learning-based variant filtering (e.g., a variant quality score recalibration approach), and when would you prefer one over the other? 🔴

**Answer:**
**Hard filtering** applies fixed, manually chosen thresholds on specific quality metrics (e.g., "exclude any variant with read depth below 10, or quality score below 30") — simple, transparent, and easy to reason about, but somewhat crude since it treats each metric independently rather than learning how they jointly relate to true-positive versus false-positive calls.

**ML-based filtering approaches** train a model (often using a set of high-confidence known-true and known-false variants) to jointly learn how multiple quality metrics combine to predict whether a candidate variant is likely real, producing a more nuanced, continuous confidence score rather than a series of independent hard cutoffs. These can achieve better sensitivity/precision tradeoffs than naive hard filtering, especially when quality metrics interact in non-obvious ways.

Preference depends on context: hard filtering is often preferred when transparency/explainability is important (e.g., needing to clearly justify why a specific variant was excluded, which matters in some clinical or regulatory contexts) or when there isn't a sufficiently large, reliable training set available for the specific sequencing protocol/context; ML-based approaches are preferred when there's enough good training data available and marginal gains in sensitivity/precision are valuable enough to justify the added complexity and reduced interpretability.

**Follow-ups:**
- If a clinical lab needed to justify to a regulator or auditor exactly why a specific variant call was excluded, would you recommend a hard-filtering or ML-based approach, and why?

---

### Q: What is variant annotation, and why can the same physical variant be reported with different predicted functional consequences depending on which transcript or annotation source is used? 🟡

**Answer:**
Variant annotation adds biological/functional context to a raw variant call — which gene and transcript(s) it overlaps, its predicted effect on the protein (e.g., missense, nonsense, synonymous, frameshift), and often population frequency data from reference databases and (where available) clinical significance classifications from curated databases.

The same variant can get different predicted consequences across sources because: **many genes have multiple transcript isoforms** (due to alternative splicing), and the same genomic position can fall in a coding exon of one transcript but an intron or untranslated region of another — so a variant's predicted functional impact genuinely depends on which specific transcript is being considered as the reference, and annotation tools may default to different "canonical" transcript choices or report multiple transcript-specific consequences that a downstream user needs to interpret carefully rather than assuming there's one single, universal "the" functional consequence for a given variant.

**Follow-ups:**
- How would you decide which transcript to prioritize when reporting a variant's functional consequence for a clinical report, if different transcripts give conflicting predicted effects?
