# 2. Bioinformatics Data Types & File Formats

The unglamorous but essential plumbing knowledge every genomics data scientist needs to actually work with real data.

---

### Q: Walk me through the main file formats you'd encounter in a typical NGS analysis, from raw sequencing output to final variant calls. 🟢

**Answer:**
A typical progression:
1. **FASTQ:** raw sequencing reads — each read has a sequence of bases and a corresponding per-base quality score, but no positional/alignment information yet.
2. **SAM/BAM (or the more modern CRAM):** reads aligned to a reference genome. SAM is the human-readable text format; BAM is its compressed binary equivalent (much smaller, used in practice); CRAM compresses further, often by referencing the genome sequence itself to avoid redundantly storing matching bases.
3. **VCF (Variant Call Format):** the variants (SNPs, indels, sometimes structural variants) identified by comparing aligned reads against the reference, along with associated quality metrics, genotypes, and often annotations.
4. **Annotation formats (GTF/GFF):** describe genomic features (genes, exons, transcripts) and their coordinates, used to interpret what a variant's position actually overlaps with (e.g., is it in a coding exon, and if so, in which gene/transcript).

Understanding this pipeline of formats matters because bugs and data quality issues can be introduced at any stage, and being able to inspect and sanity-check intermediate files (not just trust the final output) is a core practical skill.

**Follow-ups:**
- If a variant looks suspicious in the final VCF, what intermediate file(s) would you go back and inspect to investigate, and why?

---

### Q: What information does a FASTQ file contain, and what does the per-base quality score actually represent? 🟢

**Answer:**
Each read in a FASTQ file has four lines: a read identifier/header, the nucleotide sequence itself, a separator line, and a string of quality scores (one character per base, using an encoding like Phred+33) representing the sequencer's confidence in each base call.

The quality score (typically a Phred quality score) represents the estimated probability that a given base call is incorrect, on a logarithmic scale — e.g., a Phred score of 30 corresponds to roughly a 1-in-1000 chance of that specific base being called incorrectly, and a score of 20 corresponds to roughly a 1-in-100 chance. This matters practically because quality scores are used throughout downstream processing (read trimming/filtering, variant calling confidence) — low-quality base calls or reads are often trimmed or given less weight, since blindly trusting every base call as correct would introduce substantial noise into downstream analysis.

**Follow-ups:**
- How would you decide on an appropriate quality score threshold for trimming/filtering reads, and what's the risk of setting the threshold too aggressively in either direction?

---

### Q: What does it mean for reads in a BAM file to be "sorted" and "indexed," and why does this matter practically? 🟢

**Answer:**
**Sorting** typically means ordering the aligned reads by their genomic coordinate (chromosome and position) rather than the arbitrary order they came off the sequencer or through the alignment process. **Indexing** (creating a companion .bai or .csi file) builds a lookup structure over the sorted BAM file that allows rapid, random-access retrieval of reads overlapping a specific genomic region, without needing to scan through the entire file sequentially.

This matters practically because most downstream tools (variant callers, visualization tools like genome browsers, coverage calculators) require sorted, indexed BAM files to work efficiently or, in some cases, to work at all — attempting to run these tools on an unsorted or unindexed BAM file will either fail outright or require the tool to do a much slower full-file scan. It's a basic but frequently-tested practical/operational competency.

**Follow-ups:**
- If a colleague hands you a BAM file and a downstream tool throws an error about a missing index, what would you do?

---

### Q: What is a VCF file, and what are the key fields you'd inspect when evaluating whether a called variant is likely to be real versus an artifact? 🟡

**Answer:**
A VCF (Variant Call Format) file records variant positions relative to a reference genome, along with per-variant and per-sample information including the reference and alternate alleles, a quality score, filter status, and various annotation fields (read depth, allele balance, mapping quality, strand bias, and more, depending on the calling pipeline).

Key fields to inspect for likely artifacts:
- **Read depth (DP):** very low depth at a variant site means less confidence in the call — a single supporting read is much less trustworthy than dozens.
- **Allele balance/frequency:** for a germline heterozygous call, you'd generally expect roughly 50% of reads to support the alternate allele; a wildly skewed ratio can suggest an artifact, a somatic/mosaic variant, or a copy number abnormality at that locus rather than a straightforward heterozygous germline variant.
- **Mapping quality (MQ):** low mapping quality suggests the reads supporting the variant may not be confidently and uniquely aligned to this genomic location (e.g., in a repetitive region), reducing confidence in the call.
- **Strand bias:** if a variant is supported almost exclusively by reads from one strand (forward or reverse) when both should contribute roughly evenly, that's a common artifact signature, particularly for certain sequencing chemistries.
- **FILTER field:** many pipelines apply a set of hard filters and flag variants that fail them (e.g., "LowQual") — these flags should be understood and respected, not ignored.

**Follow-ups:**
- You see a variant call with high read depth but very low mapping quality across all supporting reads. What would you suspect about this genomic region?

---

### Q: What's the difference between GTF and GFF3 annotation formats, and what kind of information do they encode? 🟢

**Answer:**
Both GTF (Gene Transfer Format) and GFF3 (General Feature Format version 3) describe genomic features — genes, transcripts, exons, and their coordinates and relationships (e.g., which exons belong to which transcript, which transcripts belong to which gene) — but differ in some syntactic conventions (particularly how attributes/metadata are formatted in the final column) and in exactly how hierarchical relationships between features are represented.

This annotation information is essential for interpreting raw genomic coordinates in a biologically meaningful way — e.g., translating "this variant is at chromosome 7, position X" into "this variant falls within the coding exon of gene Y, and would cause this predicted amino acid change." Mismatches or version inconsistencies between the annotation file and the reference genome/gene model version used elsewhere in a pipeline are a common, serious source of subtle errors (e.g., using an outdated gene annotation that doesn't match current transcript definitions).

**Follow-ups:**
- What issues could arise if your annotation file (GTF) and your reference genome FASTA file are based on different genome assembly versions?

---

### Q: What is a count matrix in the context of RNA-seq or single-cell RNA-seq analysis, and what does each dimension typically represent? 🟢

**Answer:**
A count matrix is the standard tabular data structure for expression data: rows typically represent genes (or transcripts), and columns represent samples (in bulk RNA-seq) or individual cells (in single-cell RNA-seq); each cell of the matrix contains a count (or normalized value) representing the expression level of that gene in that sample/cell — typically derived from the number of sequencing reads that mapped to that gene's exons.

This is the standard input format for most downstream statistical and ML analyses (differential expression, clustering, dimensionality reduction), so understanding its structure, and the fact that raw counts require normalization before most meaningful comparisons (to account for differences in total sequencing depth between samples/cells, and other technical factors), is foundational.

**Follow-ups:**
- Why can't you directly compare raw read counts for a gene across two samples with very different total sequencing depth without normalizing first?
