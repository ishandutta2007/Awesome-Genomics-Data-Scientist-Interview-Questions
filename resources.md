# Additional Resources

A short list of resources for going deeper on genomics data science. (Not exhaustive — PRs welcome.) This field spans two disciplines (molecular biology and statistics/CS) and evolves quickly — reference genome versions, variant databases, and best-practice pipelines get updated regularly, so cross-check anything specific against current documentation before an interview.

## Foundational Reading
- *Bioinformatics Data Skills* — Vince Buffalo. Excellent practical grounding in the file formats, command-line tools, and workflows that underlie day-to-day genomics data work.
- *Modern Statistics for Modern Biology* — Susan Holmes & Wolfgang Huber (freely available online). Strong statistical foundations specifically framed around genomics applications.
- Broad Institute's GATK best practices documentation — the standard reference for germline and somatic variant calling pipeline design, updated periodically as recommended practices evolve.
- 10x Genomics' and Satija Lab's (Seurat) documentation and tutorials — widely used practical references for single-cell RNA-seq analysis workflows.

## Key Databases & Resources (good to be conversationally familiar with)
- gnomAD — a widely used population allele frequency reference database, important for variant interpretation and understanding population-level genetic variation.
- ClinVar — a public archive of variant-disease relationships and clinical significance classifications (with the important caveat that classifications are actively curated and can change over time).
- The GWAS Catalog — a curated database of published genome-wide association study results.
- Ensembl and UCSC Genome Browser — standard reference genome/annotation resources and browsers for exploring genomic regions.

## Tooling
- Familiarity with a workflow management system (e.g., Nextflow or Snakemake) is increasingly expected for production-oriented roles.
- RDKit is not relevant here, but familiarity with core genomics toolkits (e.g., samtools, bcftools, GATK, and a single-cell analysis framework like Scanpy or Seurat) is generally expected depending on role focus.
- R (Bioconductor ecosystem) and Python (scikit-bio, Scanpy, and similar) are both widely used; many roles expect working fluency in at least one, often both.

## Communities & Discussion
- Biostars and the Bioinformatics Stack Exchange for practical, tooling-level Q&A.
- r/bioinformatics for broader discussion threads.

## Staying Current
- Follow major genomics consortia and reference-resource update announcements (e.g., new reference genome assembly releases, pangenome reference efforts) since these can have meaningful downstream implications for pipeline design and result interpretation.
- Regulatory and clinical genomics guidance is actively evolving in multiple jurisdictions — check the relevant regulatory body's current published guidance directly for any role involving clinical-facing work, rather than relying on this repo or older sources.
