# Competitive & Improvement Analysis — `ewing-expression-reanalysis`

> Scope reviewed: `PLAN.md` (v0.1.0, 2026-06-28) + `TASKS.md` (24 tasks, M0–M3 + backlog).
> Project thesis: reproducible, fully-provenanced re-analysis of **open** Ewing Sarcoma
> transcriptomes (GEO/SRA/ENA, recount3, ARCHS4, Expression Atlas, Treehouse-open, DepMap),
> emitting derived results + code + manifests — never patient-level data. Cancer guardrails:
> open/de-identified only; per-source license verify; provenance on every assertion.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually mature for a v0.1 draft: guardrails lead the data section, provenance is a
CI gate, `verifiedNeed:false` is honest, recipe-first redistribution is locked, and ≥2 DE engines
are mandated. The following are concrete gaps, errors, weak metrics, and risks that should be
closed before M0 task creation.

### 1.1 Batch effects & cross-study confounding (the central scientific risk — under-specified)
- **Confounding is acknowledged but not operationalized.** PLAN §6 layer 5 and TASKS
  `ewing-rna-batch-correction-202` require a "batch-vs-biology confound report," but give no
  **decision rule** for the common Ewing case where batch (study/GEO series) is *completely or
  partially confounded with biology* (e.g., one series is all fusion-positive tumors, another is
  all cell lines/controls). ComBat-seq's own authors warn its negative-binomial model "may not
  work well on data with severely or even completely confounded study designs"
  (https://pmc.ncbi.nlm.nih.gov/articles/PMC7518324/). The plan needs an explicit **confound
  guardrail**: quantify the batch/condition association (e.g., a contingency/χ² or canonical
  correlation, plus guided PCA / PVCA), and **refuse to batch-correct (and refuse to pool)** when
  the design is confounded beyond a stated threshold — falling back to within-study DE + a
  meta-analytic (random-effects) combination instead of a corrected mega-matrix.
- **ComBat vs ComBat-seq error.** §6 names "ComBat/limma `removeBatchEffect`." For raw RNA-seq
  **counts** feeding DESeq2/edgeR, classic ComBat (Gaussian) is inappropriate; the correct tool is
  **ComBat-seq** (negative binomial, preserves integer counts)
  (https://academic.oup.com/nargab/article/2/3/lqaa078/5909519). The plan should also state the
  **standard best practice of NOT pre-correcting counts before DE** — instead include `batch` as a
  covariate in the DESeq2/edgeR/limma design matrix, and reserve `removeBatchEffect`/ComBat-seq
  output strictly for *visualization and clustering*, never as the DE input. This distinction is
  currently missing and is a frequent source of inflated significance.
- **No "double-dipping" / over-correction safeguard beyond a narrative.** Add a quantitative
  over-correction check (e.g., does correction collapse known biological variance? does it inflate
  DE-gene counts on a negative-control permutation?).

### 1.2 Platform heterogeneity (microarray vs RNA-seq) — left as an open question, but it's structural
- Open Q5 ("arrays in v1 or RNA-seq-first?") defers a decision that shapes the whole harmonization
  layer. **Microarray and RNA-seq are not directly poolable**: array intensities (log-scale,
  bounded dynamic range, probe-level) vs RNA-seq counts (over-dispersed, open-ended) require either
  rank-based integration, z-score/quantile harmonization, or per-platform DE + meta-analysis. The
  "uniformly shaped harmonization" literature exists precisely because naive merging destroys
  biology (https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10511763/). The plan should commit to
  **platform-stratified analysis with cross-platform integration only at the rank/effect-size
  level**, and say so before M2. Note the canonical Ewing knockdown signatures (e.g., GSE27524 /
  GDS4962 A673 EWSR1-FLI1 knockdown, https://pubmed.ncbi.nlm.nih.gov/28008375/) are **microarray**,
  so "reproduce the EWSR1-FLI1 signature" (metric #3, M2) likely *forces* array support — making
  the "RNA-seq-first" option partly illusory. This tension is unacknowledged.

### 1.3 Reference genome / annotation versioning — pinned, but cross-source mismatch not handled
- The plan correctly pins GENCODE/Ensembl by release + checksum. But its **preferred substrates,
  recount3 and ARCHS4, are processed against fixed, *different* builds** (recount3 uses GRCh38 +
  its own gene model via Monorail; ARCHS4 uses its own kallisto-based GENCODE pipeline;
  https://rna.recount.bio/ , https://archs4.org/). You **cannot** re-pin those to your chosen
  GENCODE release without re-quantifying from raw — which defeats the "prefer reprocessed to save
  compute" decision. The plan needs an explicit policy for **reconciling gene identifiers and
  builds across pre-processed sources** (stable Ensembl gene IDs, drop version suffixes, handle
  retired/merged IDs, GENCODE-vs-Ensembl annotation deltas) and must record *which* genome model
  each derived matrix actually came from — not just "the pinned release."
- **GRCh37↔GRCh38 liftover** of legacy Ewing series is mentioned as a problem in §3 but no
  resolution policy is given (re-quantify from raw on GRCh38 where open; exclude where not).

### 1.4 Normalization choices — named but not pinned to contrasts
- §6 lists RMA (Affy), DESeq2/edgeR/limma-voom, tximport, but does not state **which normalization
  is canonical for which output** (e.g., TPM/CPM for visualization/harmonization vs raw counts for
  DE; VST/rlog for clustering). The provenance schema captures `parameters` but the plan should
  enumerate the **normalization decision per artifact type** so reviewers can audit it. "Different
  normalization choices" is named as a core reproducibility failure in §3 yet the plan doesn't fully
  inoculate against re-introducing it.

### 1.5 Fusion-status metadata reliability — the weakest data-quality assumption
- The headline contrast ("EWSR1-FLI1-high vs -low / fusion-positive vs reference," §6 layer 4)
  **depends on fusion-status metadata that public GEO records frequently lack, mislabel, or report
  inconsistently** (free-text, no controlled vocabulary; GEO "metadata exist as unstructured plain
  text with inconsistent, non-standardized terminologies,"
  https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12205978/). The plan has **no fallback when fusion
  status is absent or untrusted.** Options it should adopt: (a) derive/confirm fusion status from
  the expression data itself where reads exist (fusion-aware calling is in backlog `-401` but is
  needed *earlier* as a metadata-QC cross-check), (b) use a curated, expression-based EWSR1-FLI1
  *activity score* rather than a binary label, (c) mark any sample whose fusion status can't be
  verified as `fusion-status: unverified` and exclude it from fusion contrasts. As written, metric
  #3 could silently rest on wrong labels.
- **Cell-line vs primary-tumor conflation.** Many "Ewing" GEO series are A673/SK-N-MC cell lines,
  not tumors; DepMap/CCLE are cell lines. Pooling cell lines with tumors without a recorded
  `sampleType` covariate is a classic confound. The provenance spine should add `sampleType`
  (tumor / cell line / PDX / organoid) as a first-class, required field — currently absent.

### 1.6 Multiple-testing — implied, never specified
- DESeq2/edgeR/limma all apply BH-FDR by default, but the plan **never states an FDR threshold,
  LFC shrinkage policy (apeglm/ashr), or independent-filtering stance**, and — critically — gives
  **no multiple-testing correction policy for the meta-analysis / cross-dataset layer** (combining
  p-values across studies, e.g. Fisher/Stouffer, needs its own FDR control). For a project whose
  entire value proposition is statistical trustworthiness, the absence of an explicit, pre-registered
  significance/effect-size policy is a real gap. Add it to `ewing-rna-de-module-103` acceptance
  criteria.

### 1.7 Reproducibility / containerization — strong, with two holes
- Excellent: digest-pinned images, conda-lock, renv, seeds, provenance lint as CI gate, synthetic
  fixture. Two gaps: (a) **Determinism is asserted but multi-threaded salmon/STAR and some BLAS
  backends are not bitwise-deterministic across thread counts/CPUs**; the "bit-comparable results"
  goal (Goal 1) should be softened to "documented numerical tolerance with a fixed thread count and
  recorded hardware/BLAS," which §ExecSummary partly does but Goal 1 over-promises. (b) **CI proves
  the pipeline *runs* on a synthetic fixture, not that it's *correct*.** Add at least one tiny
  **frozen real-data regression fixture** (a sub-sampled open series with known expected DE genes
  within tolerance) so CI catches scientific regressions, not just crashes.

### 1.8 Compute scale on donated chunks — the lane economics are hand-waved
- The "prefer recount3/ARCHS4" decision is the right compute mitigation, but **re-quantifying any
  series from raw SRA/ENA (salmon/STAR + nf-core/rnaseq) is tens-to-hundreds of GB of download and
  many CPU-hours per study** — not realistically a single donated interactive chunk. The plan never
  estimates per-dataset compute, never defines what fits the donated lane vs what *must* go funded,
  and never sets a per-task budget cap figure for the funded re-quantification it gestures at. M1's
  "runs on ≥3 datasets via nf-core/rnaseq" could quietly require funded compute. Add a **compute
  budget table** (per-dataset GB + CPU-hours estimate, lane assignment, funded cap) to M0.

### 1.9 Metric & process weaknesses
- **Metric #1 (independent reproducibility) has no baseline and a soft target** ("≥1 external
  re-run") that depends on an external volunteer who may never appear — it's outside the team's
  control, like the steward problem. Consider an *internal* clean-room re-run (different person,
  fresh checkout, different machine) as the floor, with external re-run as stretch.
- **No negative-control / sanity benchmark.** There's no plan to run the pipeline on a
  permuted-label or null dataset to confirm it reports ~0 DE genes — a cheap, powerful correctness
  check that should be a CI or M1 deliverable.
- **"≥15 catalogued, ≥5 reprocessed" (metric #2)** — is 15 even the size of the open Ewing
  transcriptome universe? The plan should sanity-check supply (how many *truly open* Ewing series
  exist) so the target isn't arbitrary. Treehouse/TARGET open-tier Ewing coverage is flagged as
  uncertain (Open Q7) yet metrics depend on it.
- **DOI granularity** unspecified: one DOI for the repo, or versioned DOIs per dataset/release?
  Zenodo supports concept-DOIs; pick one for citability.
- **Annotation of the "canonical signature" (Open Q4)** is a blocker for metric #3 but is left
  open; recommend pre-selecting (e.g., the Kinsey/Lessnick-lineage A673 knockdown signatures with
  open GEO inputs) so M2 isn't gated on a late decision.

### 1.10 Minor/editorial
- DepMap/CCLE listed as CC BY 4.0 "verify current" — correct to verify; note CCLE/DepMap terms have
  shifted over releases, so pin the *release* you cite.
- "recount3/ARCHS4 … CC0/permissive — verify": recount3 redistributes GEO/SRA-derived data; its
  redistribution permissions are *inherited* from the underlying studies, which is exactly the
  recipe-first concern — flag that "recount3 is open to *use*" ≠ "every derived matrix is free to
  *re-host*."

---

## 2. Competitive & adjacent landscape

These are uniform-reprocessing resources, expression databases, and reanalysis platforms whose
existence the plan must answer ("why build this when X exists?"). Citations are real and current.

| Resource | What it is | Strengths | Weaknesses / gaps vs this project |
|---|---|---|---|
| **recount3** (JHU/Langmead) | ~750k human+mouse runs uniformly processed by the Monorail pipeline; gene/exon/junction counts + coverage; R/Bioconductor access (https://rna.recount.bio/ ; https://doi.org/10.1186/s13059-021-02533-6) | Massive, truly uniform, junction-level, analysis-ready, free, scriptable | **Not disease-curated** — no Ewing view, no fusion metadata, no curated contrasts; fixed build/annotation you can't re-pin; provenance is pipeline-level not per-assertion; no QC/DE/signature deliverables — it's a substrate, not an analysis |
| **ARCHS4** (Ma'ayan Lab) | All GEO/SRA human+mouse RNA-seq uniformly reprocessed (kallisto), H5 + web viewer + search (https://archs4.org/) | Huge, fast access, gene+transcript, search/predict tools | Sample **metadata is largely uncurated/auto-extracted**; no Ewing-specific curation; no DE/signature reproduction; no per-sample license/provenance chain; cell-line vs tumor not disambiguated |
| **refine.bio** (Alex's Lemonade / CCDL) | Harmonized GEO/ArrayExpress/SRA microarray+RNA-seq, normalized via standardized pipelines (https://www.refine.bio/ ; https://docs.refine.bio/en/latest/main_text.html) | **Includes microarray** (most resources don't), quantile-normalized, childhood-cancer-aligned org, easy bulk download | Generic across all tissues; metadata harmonization is shallow; no fusion/Ewing curation; no per-study license rulings; no published-result reproduction; CCDL has pivoted attention toward single-cell (ScPCA) so refine.bio momentum is a question (https://www.ccdatalab.org/refine-bio) |
| **GREIN / iLINCS** (UC) | Interactive web reanalysis of >6,000 GEO RNA-seq series; QC, DE signatures, power analysis, LINCS connectivity (https://www.nature.com/articles/s41598-019-43935-8 ; https://pmc.ncbi.nlm.nih.gov/articles/PMC6527554/) | Push-button DE from raw GEO, dockerized, power analysis, signature→drug connectivity | GUI-driven, **not a reproducible CLI/manifest pipeline**; per-session not version-pinned/provenanced; no Ewing curation; no cross-study harmonization or meta-analysis; no license/redistribution discipline |
| **EBI Expression Atlas** | Manually curated, ontology-annotated, re-analyzed baseline + differential experiments; standardized contrasts (https://www.ebi.ac.uk/gxa/ ; https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12807774/) | **Gold-standard curation** of experimental "contrasts," ontology terms, FAIR/open, requires ≥3 reps + control | Curates a *selected* subset; not Ewing-exhaustive; contrasts are generic DE, **not fusion-status or signature-reproduction**; you can't re-run their exact pipeline locally on new series; no cross-study harmonized Ewing matrix |
| **GEO / GREIN-less GEO** (NCBI) | Primary archive of submitter-processed series + raw | The source of truth; vast Ewing coverage | **Heterogeneous, un-uniform, badly-annotated** — this is the *problem* the project addresses, not a competitor |
| **Treehouse Childhood Cancer (UCSC) + UCSC Xena** | ~12.7k uniformly processed pediatric+adult tumor RNA-seq compendium incl. TARGET/TCGA; explored via Xena (https://treehousegenomics.ucsc.edu/public-data/ ; https://www.nature.com/articles/s41597-025-05376-z) | Pediatric-focused, consistently processed, public subset, Xena visualization, recent (2025) 50-source compendium | **Pan-pediatric, not Ewing-deep**; clinical-site samples are controlled; public subset's exact Ewing/open coverage is uncertain (your Open Q7); no fusion-aware curation; no reproducible re-run pipeline you control |
| **GTEx** | Uniform normal-tissue reference expression (https://gtexportal.org/) | Best normal-tissue baseline; uniform | **No tumors / no Ewing**; useful only as a normal reference for contrasts |
| **cBioPortal** | Cancer genomics portal incl. some sarcoma expression+clinical (https://www.cbioportal.org/) | Easy multi-omic exploration, clinical linkage, widely used | Curated study-by-study, **not uniformly reprocessed**; no reproducible pipeline; limited open Ewing expression; visualization not reanalysis |
| **R2 Genomics (AMC)** | Free web genomics analysis/viz over 3,000+ public datasets, 850+ citations (https://hgserver1.amc.nl/cgi-bin/r2/ ) | Non-bioinformatician-friendly, rich interactive tools, has classic Ewing datasets | Closed/black-box pipeline, **not reproducible or version-pinned**; GUI not code; no provenance manifest; no redistribution of harmonized matrices |
| **Bgee** | Curated cross-species expression of *healthy* samples (https://www.bgee.org/) | Rigorous curation, calls of present/absent expression | Healthy-only, no tumors, no Ewing |
| **OmicsDI** | Cross-archive dataset discovery index (https://www.omicsdi.org/) | Finds datasets across GEO/AE/PRIDE/etc. | Discovery only — no processing, QC, DE, or provenance; complements (not competes with) the catalog layer |
| **Galaxy / Toil-UCSC recompute** | Workflow platforms / large recompute efforts (https://galaxyproject.org/) | Reproducible workflow execution, sharable histories | General-purpose engines, **not an Ewing resource**; you'd build *on* them, not compete; no curated Ewing deliverable |

**Net read:** there are excellent *general* uniform-reprocessing substrates (recount3, ARCHS4,
refine.bio, Treehouse) and excellent *curated* DE databases (Expression Atlas), and excellent
*interactive* reanalysis GUIs (GREIN, R2). **None of them is a disease-deep, fusion-aware,
per-assertion-provenanced, independently-re-runnable Ewing resource.** That negative space is real.

---

## 3. Gaps we can fill

1. **Rare-cancer depth where the big resources are shallow.** recount3/ARCHS4/refine.bio are
   tissue-agnostic; none provides a curated, exhaustive, license-audited inventory of the *open*
   Ewing transcriptome universe with per-series rulings. A definitive "every open Ewing series,
   verified" catalog is a genuine first.
2. **Fusion-aware curation that no general resource has.** EWSR1-FLI1/ERG status, fusion *type*,
   and an expression-derived activity score as first-class, harmonized, human-verified metadata —
   cross-checked against fusion-calling on samples where reads exist. General compendia carry none
   of this.
3. **Per-assertion provenance + redistribution rulings.** recount3 gives pipeline-level provenance;
   nobody gives "this gene in this DE table traces to accession X, license Y, build Z, container
   digest W," enforced in CI. That auditability is a differentiated artifact.
4. **Reproduction of published Ewing results with a concordance metric.** No competitor *reproduces
   the literature*; they provide substrates. Turning "this 2015 EWSR1-FLI1 signature
   reproduces/doesn't, here's the rank-correlation and the discrepancy" into a citable artifact is
   unfilled ground.
5. **Honest cross-study harmonization with a confound report.** Competitors either don't harmonize
   Ewing or harmonize silently; a harmonized Ewing matrix shipped *with* its batch-vs-biology
   confound analysis and explicit "do not pool these" rulings is novel.
6. **Microarray + RNA-seq unified for a rare cancer.** Because canonical Ewing signatures are
   array-era, a resource that correctly bridges legacy arrays and modern RNA-seq (rank/effect-size
   integration, not naive merge) serves the field where pure-RNA-seq resources can't.
7. **A re-runnable artifact, not a website.** GREIN/R2/Xena are GUIs you can't reproduce offline;
   shipping a one-command, container-pinned pipeline + DOI'd manifests is reusable infrastructure
   the GUIs structurally can't be.

---

## 4. Differentiators to win

1. **Provenance-as-CI-gate (auditable trust).** The release literally *cannot* ship an assertion
   without a complete accession→license→build→version→checksum chain. This is the project's
   strongest, most defensible claim — "trust, verified" — and no competitor enforces it.
2. **Fusion-aware, rare-cancer specialization.** Deep where the giants are broad: a curated,
   verified EWSR1-ETS view (status + type + activity score) is something recount3 will never build.
3. **Reproduction-of-the-literature deliverables.** A growing ledger of "published Ewing result X:
   reproduced / partially / discrepant, with concordance metric + caveats" is a unique, citable,
   field-serving output that doubles as a methods-trust signal.
4. **Recipe-first, license-clean by construction.** Default redistribution = pipeline + checksums +
   accessions; bytes only when permitted. This sidesteps the COSMIC/OncoKB/MSigDB redistribution
   traps that quietly poison many compendia.
5. **Independently re-runnable, not a hosted black box.** Digest-pinned containers + lockfiles +
   DOI snapshots survive maintainer turnover and let any third party reconstruct results — the
   opposite of GUI portals.
6. **Radical honesty as a brand.** `verifiedNeed:false`, "indirect benefit to families," discrepancy
   findings reported not buried, no novel-discovery over-claims. In a field full of irreproducible
   hype, *credibility* is the moat.

---

## 5. Claude API leverage

### Where Claude clearly helps (high-value, low-risk with human verification)
1. **Harmonizing GEO/SRA sample metadata into a controlled vocabulary.** This is the single best
   fit: GEO metadata is unstructured free-text with non-standard terminology, and LLM-assisted
   curation now outperforms traditional approaches and is an active research area
   (https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12205978/ ;
   https://www.biorxiv.org/content/10.1101/2024.10.26.620145.full.pdf). Claude can map free-text
   sample fields → schema (`sampleType`, `fusionStatus`, `fusionType`, `cellLine`, `tissue`,
   `treatment`, `timepoint`), draft ontology term mappings (EFO/Uberon/NCIt), and flag ambiguous
   records for human review — **as proposals, never as final labels.**
2. **Writing & maintaining pipeline code + tests.** Nextflow/Snakemake glue, nf-core config,
   tximport/DESeq2/edgeR R scripts, the synthetic + real-data regression fixtures, provenance-lint
   logic, the TS/ESM Elyos schema-validation glue, and CI workflows. Claude is strong at this and
   it's directly checkable by CI + review.
3. **Drafting QC and reproducibility reports.** Turning MultiQC/DESeq2 numeric outputs into Quarto
   narrative, datasheets (per *Datasheets for Datasets*), confound-report prose, and the FAIR
   self-assessment — Claude composes the words *around* pipeline-computed numbers.
4. **License/consent triage drafting.** Claude can pre-read per-series terms and *draft* the
   license-audit table and redistribution rationale for the second human reviewer to verify —
   accelerating, not replacing, the audit.
5. **Documentation, onboarding, and citation hygiene.** READMEs, responsible-use notes, contributor
   guides, data/software citation blocks, and "how to re-run" walkthroughs.

### Where Claude must NOT decide (hard guardrails — make these explicit in CONTRIBUTING)
1. **No fabricated or LLM-estimated results.** Every number, gene, fold-change, p-value, and matrix
   value comes from the **pipeline**, never from the model. Claude may format/explain results but
   may not produce, infer, "fill in," or round them.
2. **Statistical/numeric outputs are pipeline-authoritative.** DE calls, concordance metrics,
   batch-confound statistics, FDR — computed by DESeq2/edgeR/limma/code, not by Claude. CI should
   diff Claude-touched reports against the manifest to catch any number not traceable to an output.
3. **Metadata curation is human-verified.** Claude *proposes* harmonized labels; a human (domain
   reviewer) ratifies. Fusion status especially must not be set by the LLM — it's load-bearing for
   the headline contrast and must be human-verified or expression-derived.
4. **License/access determinations are human-verified (second reviewer).** Claude drafts; the
   binding ruling — including "open vs controlled," "redistribute vs recipe-only" — is a human
   decision recorded with `licenseVerifiedBy`. Never let an LLM clear a dataset for redistribution.
5. **No interpretive/clinical leap.** Claude must not phrase any output as biomarker, prognostic,
   or treatment-relevant; the "research, not medical advice" framing and reproduction-not-discovery
   language is mandatory, and any drift triggers the STOP/high-tier escalation in PLAN §8.

---

## 6. Ten concrete optimizations

1. **Add a confound decision rule + refuse-to-pool threshold** (quantify batch/condition
   association; if confounded beyond threshold → meta-analysis, not batch correction). Bake into
   `-202` acceptance criteria.
2. **Fix the correction stack:** mandate **ComBat-seq for counts**, forbid pre-correcting DE inputs
   (use `batch` as a model covariate), reserve corrected matrices for viz/clustering only.
3. **Add `sampleType` and `fusionStatus/fusionType/fusionVerifiedBy` to the provenance spine** as
   required fields; exclude `unverified` fusion samples from fusion contrasts.
4. **Pre-register the statistics:** FDR threshold, LFC shrinkage (apeglm/ashr), independent
   filtering, and the **meta-analysis p-combination + its own FDR** — written into `-103`/`-203`.
5. **Add a frozen real-data regression fixture + a permuted-label negative control** to CI, so the
   pipeline proves *correctness* (expected DE within tolerance; ~0 DE on null), not just that it runs.
6. **Replace "bit-comparable" with a documented numerical-tolerance + fixed-thread + recorded-BLAS
   policy**; pin thread counts in the manifest for determinism.
7. **Adopt an expression-derived EWSR1-FLI1 *activity score*** (continuous) as the primary signal,
   with binary fusion labels only as a cross-check — robust to missing/wrong metadata.
8. **Publish a per-dataset compute budget table** (GB download + CPU-hours, donated-vs-funded lane,
   funded cap $) in M0; explicitly route raw-SRA re-quantification to the funded lane with a cap.
9. **Record the *actual* source build/annotation per derived matrix** (recount3 GRCh38/Monorail vs
   ARCHS4 vs your GENCODE re-quant), with an ID-reconciliation step (stable Ensembl IDs, retired/
   merged ID handling) — don't assume one pinned release covers reprocessed substrates.
10. **Pre-select the canonical signature now** (a well-cited, open A673 EWSR1-FLI1 knockdown
    signature with reproducible GEO inputs, e.g. the GSE27524/GDS4962 lineage,
    https://pubmed.ncbi.nlm.nih.gov/28008375/) so M2 isn't blocked on Open Q4, and pin its exact
    inputs + DE method.

---

## 7. Parallel & perpendicular spin-offs

- **`ewing-open-data-catalog` (extract the catalog as a standalone good).** The license-audited,
  fusion-annotated inventory of open Ewing transcriptomes is independently valuable (think
  OmicsDI-for-Ewing with verified rulings). Ship it as its own citable dataset; it's the foundation
  every sibling reuses.
- **`ewing-fusion-detection-benchmark`.** A benchmark of fusion callers (Arriba — DREAM SMC-RNA
  winner — STAR-Fusion, FusionCatcher, CICERO) on open Ewing RNA-seq with truth from orthogonal
  evidence (https://github.com/suhrig/arriba ;
  https://www.biorxiv.org/content/10.1101/2025.01.20.633905v1.full). Directly feeds this project's
  fusion-status QC cross-check and is a clean methods paper.
- **`ewing-single-cell-atlas` handoff (already in backlog `-403`).** Integration spec to the
  CCDL/ScPCA-style single-cell world; bulk deconvolution validated against scRNA proportions.
  Keeps single-cell out of v1 core while wiring the bridge.
- **`ewsr1-fli1-knowledge-graph`.** Link harmonized DE/signature results to GG/microsatellite
  targets, ChIP-seq binding, and pathway nodes — a graph that makes the expression resource
  queryable as biology (recipe-reference restricted sources; never re-host MSigDB/COSMIC/OncoKB).
- **Reusable rare-cancer reprocessing template.** Generalize the whole stack (catalog schema +
  provenance spine + nf-core backbone + confound report + CI gates) into a **cookiecutter for any
  rare cancer** (rhabdomyosarcoma, osteosarcoma, DSRCT…). This is arguably the *highest-leverage*
  spin-off: the methodology, not the Ewing instance, is the durable contribution.
- **MCP server serving harmonized expression matrices.** An MCP endpoint exposing the
  license-clean harmonized matrices + metadata + provenance to agents (query by gene/sample/contrast,
  always returning provenance), with hard refusal on controlled/closed data. Makes the resource
  agent-native and showcases Elyos's Claude-API story — while honoring recipe-first (serve derived
  data only where redistribution is permitted; otherwise serve the recipe).

---

## 8. Open questions for the maintainer

1. **Confound policy:** what quantitative threshold flips a dataset group from "batch-correct &
   pool" to "meta-analyze only"? Who signs off the refuse-to-pool calls?
2. **Array scope (sharpens Open Q5):** given the canonical EWSR1-FLI1 signatures are microarray, is
   "RNA-seq-first" actually viable, or must M2 include arrays from the start? If included, what's
   the cross-platform integration method (rank-based? effect-size meta-analysis)?
3. **Fusion-status source of truth:** when GEO metadata lacks/contradicts fusion status, do we
   (a) exclude, (b) call fusions from reads, (c) use an expression activity score — and in what
   precedence?
4. **Compute lane reality:** which datasets are genuinely re-quantifiable on donated interactive
   compute vs which *require* the funded lane, and what is the per-task escrow cap?
5. **Reprocessed-substrate build reconciliation:** are we comfortable that recount3 (GRCh38/Monorail)
   and ARCHS4 outputs can't be re-pinned to our GENCODE release, and how do we record/reconcile that
   honestly in provenance?
6. **Determinism bar:** do we commit to a fixed-thread + recorded-hardware numerical-tolerance
   standard (and drop "bit-comparable"), or invest in fully deterministic execution?
7. **Statistics pre-registration:** will we publish the FDR/LFC/meta-analysis correction policy as a
   pinned, versioned document before generating any DE result (to avoid post-hoc threshold drift)?
8. **Canonical signature lock (Open Q4):** can we commit to the specific published signature + its
   exact open inputs now, so M2 reproduction isn't gated on a late decision?
9. **Steward (Open Q1, still the biggest gap):** is there a realistic path to a named Ewing
   lab/foundation steward, or should the project explicitly reframe its success as
   infrastructure-delivered with adoption as stretch? Candidate: CCDL/Alex's Lemonade ecosystem,
   which already funds open childhood-cancer data tooling.
10. **Catalog supply check:** how many *truly open* Ewing series actually exist — is "≥15
    catalogued / ≥5 reprocessed" conservative or optimistic against the real universe (incl.
    Treehouse/TARGET open-tier, Open Q7)?
