# 8. Behavioral & Case Studies

Genomics data scientists constantly bridge computational rigor with wet-lab realities and, often, clinical or research stakeholders. Interviewers probe whether you can navigate that bridge well.

---

### Q: Tell me about a time your analysis results were met with skepticism from a wet-lab scientist or clinician. How did you handle it? 🟡

**What a strong answer includes:**
- Specific understanding of *why* the skepticism existed — often rooted in real experimental or clinical knowledge the analysis didn't fully account for, rather than a knowledge gap on the stakeholder's part.
- A concrete approach to investigating the disagreement collaboratively (e.g., "let's look at the raw data/alignments together" rather than defending the pipeline's output as automatically correct).
- Evidence of genuine two-way exchange — incorporating the domain expert's insight into refining the analysis, not just eventually convincing them you were right.
- An honest outcome, including situations where the skepticism turned out to be well-founded and revealed a real issue with the analysis.

**Follow-ups:**
- How did that experience change how you communicate and validate results with wet-lab or clinical collaborators going forward?

---

### Q: Describe a time you discovered that a batch effect or technical confound had compromised (or nearly compromised) a study's conclusions. What did you do? 🔴

**What a strong answer includes:**
- A specific, technically grounded description of how the confound was discovered (ties directly to section 6's discussion of investigating surprising results).
- Evidence of appropriately escalating the concern even if it was inconvenient (e.g., threatening a paper submission deadline or a costly experimental redo) rather than downplaying or ignoring it.
- A clear articulation of what was ultimately done — whether that meant redesigning the experiment, adjusting the statistical approach, or in some cases concluding the original finding couldn't be reliably supported.
- A systemic change made afterward (e.g., advocating for randomized sample processing in future experimental designs) to prevent recurrence.

**Follow-ups:**
- How do you build a working relationship with experimental collaborators so that raising concerns about experimental design (e.g., sample randomization across batches) is received constructively rather than as an obstacle?

---

### Q: Walk me through how you'd approach analyzing a new dataset you've just received, before diving into the specific biological question you're trying to answer. 🟡

**Case-study structure — a strong candidate would:**
1. **Understand the experimental design and sample metadata thoroughly first** — how were samples collected and processed, what's the study design (case-control, longitudinal, etc.), and are there any known technical factors (batch, processing date) that need to be tracked as potential confounders.
2. **Perform thorough quality control before any biological analysis** — checking for the kinds of technical issues discussed in sections 2, 4, 5, and 6 (sequencing quality, alignment quality, batch effects, outlier samples) rather than jumping straight to the biological question and risking building conclusions on a shaky technical foundation.
3. **Do exploratory visualization** (e.g., PCA, clustering) to get a general sense of the data's structure and check whether it aligns with expectations (or reveals unexpected technical or biological structure) before committing to a specific analysis plan.
4. **Clarify the specific biological/statistical question and appropriate method** given what's now known about the data's quality and structure, rather than applying a generic, pre-decided analysis pipeline regardless of dataset-specific quirks.
5. **Plan for reproducibility and documentation** from the start (version control, pinned reference versions) rather than treating it as a later cleanup step.

**Follow-ups:**
- What would you do if your initial QC step revealed several samples with unusually low quality — would you exclude them, and how would you decide?

---

### Q: Tell me about a time you had to explain a technical/statistical limitation of your analysis to a non-computational stakeholder who was hoping for a more definitive answer. 🟡

**What a strong answer includes:**
- A specific example of translating a genuine limitation (e.g., "this GWAS hit indicates an associated region, not necessarily the exact causal variant," or "our pipeline has lower sensitivity in this specific genomic region") into terms usable for the stakeholder's actual decision, rather than either overstating certainty or being so hedged as to be unhelpful.
- Evidence of calibrating the level of technical detail to the audience appropriately.
- A concrete, positive outcome showing the stakeholder was able to make a better-informed decision as a result.

**Follow-ups:**
- How do you avoid being perceived as constantly hedging when a significant part of communicating genomics results honestly involves conveying real uncertainty?

---

### Q: Describe a time you had to make a judgment call about excluding samples or data points from an analysis due to quality concerns, with real stakes riding on the decision. 🔴

**What a strong answer includes:**
- A specific, principled basis for the exclusion decision (a predefined QC threshold, or a clearly articulated ad hoc reason if no predefined threshold existed) rather than an arbitrary or after-the-fact rationalized choice.
- Evidence of considering the tradeoff — excluding legitimate but noisy data reduces statistical power and sample size, while including poor-quality data risks introducing artifacts — and reasoning explicitly about where to draw the line rather than defaulting reflexively to one side.
- Transparency in reporting the exclusion decision and rationale to stakeholders/collaborators, rather than silently dropping data without documentation.
- An honest reflection on whether, in hindsight, the decision was the right one.

**Follow-ups:**
- How would you decide on quality-control thresholds before looking at the actual data, to avoid the appearance (or reality) of choosing thresholds that conveniently produce your desired result?

---

### Q: Tell me about a time you had to work with an incomplete, messy, or poorly documented dataset (e.g., inherited from a previous team member or an external collaborator) and still needed to produce a reliable analysis. 🟡

**What a strong answer includes:**
- A specific, systematic approach to reconstructing missing context (e.g., inferring sample processing details from available metadata, reaching out to original data generators when possible, or being transparent about remaining uncertainty when it couldn't be fully resolved).
- Evidence of appropriately calibrating confidence in the final analysis given the data's documented limitations, rather than presenting results with more certainty than the underlying data quality actually supports.
- A concrete outcome, and ideally a systemic change advocated for afterward (e.g., pushing for better metadata/documentation standards on the team going forward) to reduce the recurrence of this problem.

**Follow-ups:**
- How would you push for better data documentation practices on a team without it feeling like you're criticizing colleagues' past work?
