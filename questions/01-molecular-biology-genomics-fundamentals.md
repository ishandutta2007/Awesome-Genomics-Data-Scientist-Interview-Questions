# 1. Molecular Biology & Genomics Fundamentals

Core vocabulary and biological concepts every genomics data scientist needs, regardless of which side (computational or biological) they came from.

---

### Q: Explain the central dogma of molecular biology, and where genomic, transcriptomic, and proteomic data each fit into it. 🟢

**Answer:**
The central dogma describes the flow of genetic information: **DNA → RNA (transcription) → Protein (translation)**. DNA stores the genetic blueprint; it's transcribed into messenger RNA (mRNA); and mRNA is translated into protein, which carries out most cellular functions.

Data types map onto this flow:
- **Genomic data** (DNA sequencing) captures the underlying blueprint — the sequence itself, including variants, structural changes, and regulatory regions.
- **Transcriptomic data** (RNA sequencing) captures which genes are being actively transcribed and at what level (gene expression) at a given point in time/condition — this is dynamic and context-dependent, unlike the largely static genome.
- **Proteomic data** captures the actual protein products and their abundance/modifications, one step further removed from the DNA blueprint and often the most functionally direct but hardest and most expensive layer to measure at scale.

Understanding this hierarchy matters practically because each data type answers a different question, and a genomics data scientist needs to reason about which layer is actually relevant to a given biological question rather than assuming they're interchangeable proxies for each other.

**Follow-ups:**
- Why might a gene show high DNA-level conservation but highly variable expression across cell types, and what does that imply about which data type you'd need to study a specific biological question?

---

### Q: What's the difference between whole-genome sequencing (WGS), whole-exome sequencing (WES), and targeted panel sequencing? When would you choose each? 🟢

**Answer:**
- **WGS** sequences essentially the entire genome, including coding and non-coding regions — most comprehensive, but higher cost and more data to process/store, and much of the genome (especially non-coding regions) is harder to interpret confidently.
- **WES** sequences only the protein-coding regions (exons, roughly 1-2% of the genome) — cheaper and simpler to interpret since coding variants are generally better understood, but misses non-coding regulatory variants and structural variants that WGS can capture.
- **Targeted panel sequencing** sequences a curated, specific set of genes/regions relevant to a particular clinical or research question (e.g., a cancer gene panel) — cheapest, fastest, deepest coverage per targeted region (useful for detecting rare variants at low allele frequency), but obviously limited to the genes included on the panel.

Choice depends on the actual question: a diagnostic test for a well-characterized set of disease genes favors a targeted panel (cost-effective, high sensitivity within scope); a discovery-oriented research study looking for novel or non-coding variants favors WGS; and WES is often a reasonable middle ground when the hypothesis is coding-region-focused but the specific causal gene isn't known upfront.

**Follow-ups:**
- Why might a targeted panel achieve much higher sequencing depth than WGS for the same total sequencing budget, and why does that matter for variant detection sensitivity?

---

### Q: What is a reference genome, and what are the practical implications of the fact that it doesn't perfectly represent any single individual? 🟡

**Answer:**
A reference genome (e.g., GRCh38 for humans) is a representative, consensus genome assembly used as the coordinate system against which sequencing reads from an individual sample are aligned and variants are called — it's not any single person's actual genome, but a composite/consensus assembled from multiple sources.

Practical implications:
- **Reference bias:** reads containing a variant relative to the reference can align slightly less well than reads matching the reference exactly, which can subtly bias variant calling, especially at highly variable/polymorphic loci — this is a known, actively studied issue (e.g., part of the motivation behind pangenome reference efforts that aim to represent population diversity rather than a single consensus sequence).
- **Reference gaps/errors:** some genomic regions are poorly represented or missing in a given reference version, meaning certain variants or structural features simply can't be reliably captured or interpreted using that reference.
- **Version consistency matters enormously:** analyses and coordinate positions (e.g., a variant's genomic position) are only meaningful relative to a specific reference genome version — mixing data or annotations across reference versions without proper coordinate conversion (liftover) is a common and serious source of error.

**Follow-ups:**
- What is "liftover," and what risks does it introduce when converting coordinates between reference genome versions?

---

### Q: What's the difference between a SNP, an indel, and a structural variant? Why does this distinction matter for how you'd detect each computationally? 🟢

**Answer:**
- **SNP (single nucleotide polymorphism):** a single base-pair difference from the reference at a specific position.
- **Indel:** a small insertion or deletion of one or more bases (typically defined as smaller than structural variants, though the exact size cutoff is somewhat convention-dependent).
- **Structural variant (SV):** larger-scale genomic alterations — deletions, duplications, inversions, translocations, copy number variants — typically defined as spanning at least 50 base pairs, though definitions vary.

This distinction matters computationally because each requires different detection strategies: SNPs and small indels are well-detected by standard short-read alignment and variant calling; structural variants are much harder to detect reliably from short-read data alone (since a short read often can't span or fully capture a large rearrangement) and typically benefit from specialized SV-calling algorithms, long-read sequencing technologies, or complementary approaches (read-depth analysis, split-read/discordant-pair evidence) — a genomics data scientist needs to know which variant classes a given pipeline is actually well-suited to detect, since a standard short-read SNP-calling pipeline will systematically miss or misrepresent many structural variants.

**Follow-ups:**
- Why are long-read sequencing technologies generally better suited to detecting structural variants than short-read technologies?

---

### Q: What is linkage disequilibrium (LD), and why is it important to understand when interpreting GWAS results or designing a genotyping panel? 🟡

**Answer:**
Linkage disequilibrium refers to the non-random association between alleles at different genomic loci, typically because they're physically close together on a chromosome and tend to be inherited together rather than independently assorting — nearby variants are often correlated (in LD) with each other across a population.

This matters for interpreting GWAS (genome-wide association study) results because a statistically significant association between a genetic variant and a trait doesn't necessarily mean that specific variant is causal — it might simply be in LD with the true causal variant nearby. This is why GWAS "hits" typically implicate a genomic region/locus rather than pinpointing a single definitively causal variant, and why follow-up fine-mapping work is often needed to narrow down the actual causal variant(s) within an associated LD block. It also matters for genotyping panel design, since a smaller set of "tag" SNPs can capture information about many other correlated variants in the same LD block without needing to genotype every single variant directly.

**Follow-ups:**
- What is fine-mapping, and how does it attempt to narrow down a GWAS association signal to the likely causal variant?

---

### Q: What's the difference between germline and somatic variants, and why does this distinction fundamentally change how you'd approach variant calling and interpretation? 🟡

**Answer:**
**Germline variants** are present in essentially every cell of an individual (inherited from parents or arising very early in development), typically present at ~50% or ~100% allele frequency in a diploid genome (heterozygous or homozygous) — relevant to inherited disease risk and passed on to offspring.

**Somatic variants** arise in individual cells during an individual's lifetime (e.g., in a tumor) and are present only in a subset of cells (often at low, variable allele frequency depending on what fraction of sampled cells carry the mutation) — relevant to conditions like cancer, where the tumor is a mixture of cell populations with different accumulated mutations (clonal heterogeneity).

This distinction fundamentally changes variant calling approach: germline calling generally assumes a clean diploid model (expecting allele frequencies near 0%, 50%, or 100%), while somatic calling must account for **variable, often low allele frequencies**, tumor purity/contamination from normal tissue, and typically requires comparing a tumor sample against a matched normal sample from the same individual to distinguish true somatic mutations from germline variants or sequencing artifacts — using a standard germline-calling pipeline on tumor data without this comparison would produce highly unreliable results.

**Follow-ups:**
- Why is a matched normal sample so important for reliable somatic variant calling, and what would you do if one wasn't available?
