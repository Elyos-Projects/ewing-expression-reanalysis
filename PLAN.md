# ewing-expression-reanalysis — PLAN

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) ·
> Lane: donated (primary); funded (optional, capped — see §6) · Track: 8a — Ewing's Sarcoma ·
> Risk tier: **medium** (project), with **high**-tier gates for any interpretive/clinical content.

---

## Executive summary

Ewing Sarcoma is a rare, aggressive bone-and-soft-tissue cancer that mostly strikes children,
teenagers, and young adults. It is driven in ~85% of cases by the **EWSR1-FLI1** fusion oncogene
(and in most of the rest by **EWSR1-ERG** or other EWSR1-ETS fusions). Because it is rare, the
public molecular data that exists is precious — and chronically **under-reused**. Dozens of
transcriptome studies sit in public archives (GEO, SRA/ENA, GDC/TARGET open tier, recount3,
ARCHS4, Expression Atlas, Treehouse, DepMap), each processed with a different, often
under-documented pipeline, different reference genome, different normalization, and different
software versions. The result: published expression findings are **hard to reproduce, hard to
compare across studies, and hard for the next researcher to build on.**

`ewing-expression-reanalysis` builds **reproducible, fully-provenanced reanalysis pipelines** for
**public, open-access** Ewing transcriptomes, and republishes the harmonized, license-clean
intermediate products (QC reports, uniformly-quantified expression matrices where redistribution
is permitted, differential-expression tables, reproducibility manifests) as an open scientific
resource. The deliverable is **infrastructure for reproducibility, not a clinical claim**: a
versioned, containerized, checksum-verified pipeline plus a catalog of datasets it has been run
on, every assertion carrying provenance back to its source accession, license, and software
version.

This is **not** a diagnostic, a biomarker, or patient guidance. It produces no medical advice.
It uses **only open-access / aggregate / de-identified data**; controlled-access individual-level
data (dbGaP, EGA, biobanks) is categorically out of scope. The intended beneficiaries are the
Ewing research and patient-advocacy ecosystem — academic labs, methods developers, citizen
scientists, and foundations — and, indirectly and over a long horizon, the **families facing
this disease**, whose children are best served by science that other scientists can actually
trust and reuse.

We do not yet have a named research or advocacy partner; the **verified need is TO BE SECURED**
(see §2). Until it is, the project proceeds on the well-evidenced general need for reproducible
rare-cancer bioinformatics, every task ships `verifiedNeed: false`, and no output is promoted as
"used by partner X" until a partner confirms it.

---

## Problem & beneficiaries

**The disease.** Ewing Sarcoma is the second most common bone cancer in children and young
adults. It is rare (on the order of ~200 new pediatric cases per year in the US; a similar order
in Europe), with markedly worse outcomes for metastatic and relapsed disease. Rarity is exactly
why every public dataset matters and why wasted, non-reproducible reanalysis is a real cost to
the field — there is no large cohort to fall back on.

**The reproducibility problem (the need this project addresses).**
- Public Ewing transcriptomes are scattered across archives with **heterogeneous processing**:
  different aligners/quantifiers, genome builds (GRCh37 vs GRCh38), annotation versions, and
  normalization choices.
- Published differential-expression and "EWSR1-FLI1 signature" results frequently **cannot be
  reproduced** from the deposited data + methods section alone (missing versions, missing seeds,
  missing parameters, missing exact sample inclusion lists).
- Cross-study comparison is unreliable because **batch effects and pipeline differences are
  confounded** with biology.
- Newcomers (advocacy-funded labs, students, citizen scientists) re-do the same plumbing badly
  instead of building on a trustworthy base.

This matches the published evidence base on the bioinformatics reproducibility crisis and FAIR
data; it is a **real, documented, general need**. What is **not yet verified** is a *specific*
partner who will adopt and steward the output (see below).

**Beneficiaries (in order of directness).**
1. **Ewing methods developers & researchers** — get a trustworthy, reusable, uniformly-processed
   substrate and a reference pipeline they can cite and extend.
2. **Patient-advocacy-funded research** (e.g. small foundation-funded labs) — lower the fixed
   cost of doing credible molecular analysis.
3. **Students / citizen scientists / open-science community** — a worked, runnable example of
   rigorous rare-cancer reanalysis.
4. **Indirect, long-horizon: patients and families** — better, more trustworthy, faster-building
   science. We are explicit that the benefit to families is *indirect*; this project will not
   over-claim a direct patient impact.

**Verified need & partner — TO BE SECURED.**
We have **no signed partner or requestor yet.** Candidate stewards to approach (none contacted,
none committed): pediatric/sarcoma research labs publishing on open data; the open-science /
reproducibility community (e.g. recount/Bioconductor ecosystems); and Ewing/sarcoma patient
foundations that fund research (named targets to be recorded in `governance/partners.md` once
outreach begins). **Until a steward confirms an outcome they will use, `verifiedNeed` is `false`
on every task** and the Definition of Shipped's "handed to beneficiary" criterion is unmet by
design (§8).

---

## Goals and non-goals

**Goals**
1. A **reproducible RNA-seq/array reanalysis pipeline** for public Ewing transcriptomes —
   containerized, version-pinned, seed-controlled, CI-tested, runnable by a third party to
   bit-comparable (or documented-tolerance) results.
2. A **provenance-first dataset catalog + datasheets** of public Ewing transcriptomes, each with
   verified accession, license, consent/ethics basis, genome build, and a redistribution
   decision (may we republish derived matrices, or only the recipe?).
3. **Harmonized derived products** (QC, uniformly-quantified matrices where licensing permits,
   DE tables, the reproduced EWSR1-FLI1 expression signature) released openly with DOIs.
4. **Reproducibility manifests** for every run: input checksums, container digests, parameter
   files, software versions, and a one-command re-run.
5. **Provenance on every assertion** and a FAIR-compliant, citable release.

**Non-goals (explicit)**
- **No medical advice, diagnosis, prognosis, treatment guidance, or biomarker recommendation.**
  No patient-facing interpretation in this repo.
- **No controlled-access data.** No dbGaP/EGA/biobank individual-level data, no attempt to
  re-identify, no pooling that could re-identify.
- **No primary discovery claims.** The project's claim is *reproducibility and harmonization*,
  not "we found a new Ewing driver." Any novel observation is framed as a reproduction/replication
  result with caveats, never as clinical evidence.
- **No raw-read redistribution** unless the source license unambiguously permits it; default is
  to redistribute *recipes + checksums*, not bytes.
- **No fusion-calling clinical tool.** Fusion-aware methods may appear as analysis steps but never
  as a diagnostic.
- **No for-profit primary benefit.** Outputs are public; we will not tailor deliverables to a
  single company's commercial pipeline.
- **No web "drug/target finder" or trial tooling here** — those are separate, higher-risk Ewing
  projects (`ewing-drug-target-evidence`, `ewing-trial-finder`) with their own expert gates.

---

## Success metrics (outcomes)

Outcome-based and beneficiary-centric, not vanity counts. Baselines are mostly **unknown until
M0 measurement**; targets are first-release goals.

| # | Outcome (what changes for a beneficiary) | Baseline | Target (v1.0) | How measured |
|---|---|---|---|---|
| 1 | **Independent reproducibility**: a third party re-runs a pipeline and gets matching results | none today | ≥1 external re-run reproduces ≥1 dataset within documented tolerance | Logged external re-run issue + checksum/transcript comparison report |
| 2 | **Datasets made reusable**: open Ewing transcriptomes catalogued with verified license+provenance | 0 | ≥15 catalogued; ≥5 reprocessed end-to-end | Catalog + datasheets in repo, license field non-empty & verified |
| 3 | **Published result reproduced**: at least one literature EWSR1-FLI1 signature/DE result reproduced (or discrepancy documented) | 0 | ≥1 reproduced with concordance metric + caveats | Reproduction report w/ concordance (e.g. rank correlation, overlap of DE gene sets) |
| 4 | **Adoption/citation by the field** | 0 | ≥1 external lab/citizen-scientist reuses pipeline or data (issue, fork, citation, or DOI download evidence) | GitHub forks/issues, Zenodo download stats, citation alert |
| 5 | **Provenance completeness** | n/a | 100% of released assertions trace to source accession + license + version | Automated provenance-manifest lint in CI |
| 6 | **FAIR compliance** | n/a | Release scores "F+A+I+R" on a published checklist; has a DOI | FAIR self-assessment checklist + Zenodo DOI |
| 7 | **Partner/steward outcome (gated)** | none | ≥1 steward confirms they used an output for a real research task | Steward confirmation recorded; flips `verifiedNeed → true` for that line |

We will **not** report "number of tasks merged" as success. Merged ≠ delivered (§8).

---

## Scope

**In scope**
- Discovery, license-verification, and cataloguing of **open-access** Ewing transcriptome
  datasets (bulk RNA-seq and microarray; GEO/SRA/ENA/GDC-open/recount3/ARCHS4/Expression
  Atlas/Treehouse-open/DepMap).
- A containerized, version-pinned **quantification + QC + differential-expression** pipeline
  (built on community-standard, openly-licensed tooling — see §6).
- **Cross-dataset harmonization** with explicit batch-effect handling and honest confound
  reporting; reproduction of the canonical EWSR1-FLI1 expression signature.
- **Reproducibility manifests**, datasheets, Quarto/R-Markdown reports, and a DOI'd release.
- Redistribution of **derived** products only where the source license permits; otherwise the
  pipeline + checksums + accession list so anyone can regenerate them.

**Out of scope** (will NOT do)
- Any **controlled-access** data (dbGaP, EGA, individual-level biobanks) or identifiable patient
  data — full stop. No access requests, no IRB-gated work; this is donated-AI scope, not
  authorized-access scope.
- Any **clinical, diagnostic, prognostic, or treatment** output; any patient-facing medical
  guidance; any biomarker "recommendation."
- **Primary biological discovery** marketed as new clinical evidence.
- **Redistribution of restrictively-licensed resources** (e.g. COSMIC, OncoKB, MSigDB
  KEGG/curated subsets) — referenced by recipe only, never re-hosted (§7).
- **Re-identification** or linkage across datasets that could re-identify an individual.
- **Single cell** deep work, **proteomics**, **trial/eligibility/family-guide** content — handed
  off to sibling projects.
- Hosting a live web service or compute back-end for end users (a static report site is the
  ceiling for v1).

---

## Solution approach & architecture

This is a **data + code** project. The architecture is a layered, reproducible pipeline with a
provenance spine running through every layer.

**Pipeline layers**
1. **Acquisition & provenance capture.** For each dataset: record accession, source archive,
   license, consent/ethics statement, platform, organism, genome build, study design, and SHA-256
   checksums of every input. Prefer archives that already store *uniformly reprocessed* data
   (**recount3**, **ARCHS4**) to reduce compute and re-alignment variance; fall back to
   re-quantifying from raw reads (SRA/ENA) only where the license permits and the dataset is open.
2. **Quantification.**
   - RNA-seq: salmon (selective alignment) or STAR→salmon, GENCODE/Ensembl reference (pinned
     release), `tximport` to gene level. We will **reuse community-standard pipelines**
     (nf-core/rnaseq) rather than hand-rolling, for auditability.
   - Microarray (older GEO series): standard normalization (RMA for Affymetrix; appropriate
     normalization for others) via openly-licensed Bioconductor packages, with platform-annotation
     provenance pinned.
3. **QC & sample governance.** FastQC/MultiQC, library-complexity, mapping-rate, and
   expression-based outlier detection; an explicit, recorded **sample inclusion/exclusion list**
   (the single most common source of irreproducibility).
4. **Differential expression & signature.** DESeq2 / edgeR / limma-voom (method recorded per
   run), with the **EWSR1-FLI1-high vs -low / fusion-positive vs reference** contrasts where the
   public metadata supports them; reproduction of a published Ewing signature with a concordance
   metric.
5. **Harmonization & meta-analysis.** Cross-dataset combination with explicit batch correction
   (e.g. ComBat/limma `removeBatchEffect`) **and** a confound report (batch vs biology), plus a
   meta-analytic combination where appropriate. Harmonization is always reported *with* its
   limitations.
6. **Reporting & release.** Quarto/R-Markdown reproducible reports → MultiQC summaries → DE tables
   → datasheets → a versioned Zenodo release with DOI and a WorkflowHub entry.

**Tech stack**
- **Workflow:** Nextflow (nf-core/rnaseq as the quantification backbone) and/or Snakemake;
  one orchestrator chosen and locked in M0.
- **Containers:** Docker images pinned by **digest**; Apptainer/Singularity for HPC; no
  `:latest` tags anywhere.
- **Environments:** conda/mamba with **lockfiles** (e.g. `conda-lock`) committed; R via `renv`
  lockfile.
- **Analysis:** R/Bioconductor (DESeq2, edgeR, limma, tximport, fgsea), Python where useful
  (pandas, scanpy only if a single-cell handoff is needed — out of scope for v1 core).
- **Reproducibility infra:** GitHub Actions CI runs a **tiny synthetic/subsampled fixture**
  end-to-end on every PR; checksum + provenance-manifest linting; nf-core test profiles.
- **Reference data:** GENCODE/Ensembl + GENCODE basic annotation, pinned release; references are
  fetched by recorded URL + checksum, never vendored without license check.
- **Glue/CLI & schema validation:** TypeScript/ESM per Elyos conventions for any Elyos-facing
  tooling (task validation, registry entries); the scientific pipeline itself is
  Nextflow/R/Python (the right tools for the domain — `adapters/` boundary respected, no
  vendor/agent logic in core).

**Data model (provenance spine).** Every dataset record and every released artifact carries:
`accession`, `sourceArchive`, `license`, `licenseVerifiedBy`, `consentEthicsBasis`,
`redistributionAllowed` (bool + rationale), `genomeBuild`, `annotationVersion`,
`pipelineVersion`, `containerDigests[]`, `inputChecksums[]`, `outputChecksums[]`, `seed`,
`generatedAt`, `parameters`. A **provenance manifest** (`provenance.json`) is emitted per run and
linted in CI; a release fails if any released assertion lacks a complete chain.

**Key decisions (locked).**
- **Reuse, don't reinvent:** build on nf-core/rnaseq + Bioconductor rather than bespoke aligners.
- **Recipe-first redistribution:** default output is *pipeline + checksums + accession list*;
  derived matrices are republished **only** when the source license clearly permits.
- **Provenance is a CI gate**, not documentation politeness.
- **Synthetic fixture for CI:** real Ewing data is too large for CI; a tiny synthetic/subsampled
  fixture proves the pipeline runs and stays green.
- **Two reference DE engines minimum** (DESeq2 + one of edgeR/limma) so results aren't an
  artifact of one method.

---

## Data, licensing & compliance

> ### CANCER GUARDRAILS (BINDING — these lead this section and override anything below)
> 1. **Open-access / aggregate / de-identified data ONLY.** Controlled-access sources —
>    **dbGaP, EGA, individual-level biobanks** — and **ANY identifiable patient data** are
>    **OUT OF SCOPE.** They require authorized access + IRB oversight, which is *not* what
>    donated AI tasks are for. We do not request, download, mirror, or analyze them.
> 2. **Verify every source license before use.** TCGA/GDC **open-tier** and **GEO** are open and
>    usable with attribution. **COSMIC and OncoKB are non-commercial / custom-licensed — FLAG and
>    do not redistribute**; reference by recipe only. (Also flagged: **MSigDB** curated/KEGG
>    subsets, which carry their own restrictions.) No source is used until its license is recorded
>    and verified by a second reviewer.
> 3. **No medical advice.** Any patient-facing content is **education only, fully sourced, labelled
>    "not medical advice," and gated behind oncologist + patient-advocate review (riskTier
>    `high`).** The core of this project produces **no** patient-facing content; if any is ever
>    proposed it leaves this repo and enters a high-tier sibling project.
> 4. **Provenance on every assertion.** Every released number, gene, matrix, and claim traces to
>    its source accession, license, genome build, and software version, enforced in CI.

**Sources and their licenses (each verified in M0 before use; this table is the *starting
assumption*, not the final ruling).**

| Source | What | License / terms (to verify) | Use & redistribution stance |
|---|---|---|---|
| **GEO** (NCBI) | Public expression series (array + RNA-seq processed) | NCBI public data; generally free to use, cite; no blanket redistribution license — per-series check | Use; redistribute derived only if series terms allow; always cite + record |
| **SRA / ENA** | Raw reads for open studies | Open where deposited open; per-study | Re-quantify only if open; redistribute recipe+checksums, not raw reads by default |
| **GDC / TARGET open tier** | Open-tier pediatric expression (gene-level) | NIH Genomic Data Sharing — open-access tier usable w/ attribution | Use open tier ONLY; controlled tier OUT OF SCOPE |
| **recount3 / ARCHS4** | Uniformly reprocessed public RNA-seq | Open (recount/ARCHS4 terms; CC0/permissive — verify) | Preferred substrate; cite; redistribute derived per terms |
| **Expression Atlas (EBI)** | Curated expression | EBI terms (generally CC0/CC-BY — verify) | Use; cite |
| **Treehouse Childhood Cancer (UCSC)** | Pediatric compendium (public subset) | Public subset terms — verify each | Use public subset only; verify before any redistribution |
| **DepMap / CCLE** | Ewing cell-line expression + dependencies | **CC BY 4.0** (verify current) | Use w/ attribution; redistribute derived w/ attribution |
| **GENCODE / Ensembl** | Reference genome/annotation | Open (Ensembl/GENCODE terms) | Use; pin release + checksum |
| **MSigDB** | Gene sets for enrichment | **Mixed — KEGG/curated subsets restricted** | **FLAG**; use only permissively-licensed sets; recipe-reference others |
| **COSMIC** | Somatic mutation catalogue | **Non-commercial / custom** | **FLAG — do not redistribute**; recipe-reference only if used at all |
| **OncoKB** | Precision-oncology annotations | **Restricted / custom** | **FLAG — do not redistribute**; out of core scope |
| **dbGaP / EGA / biobanks** | Controlled individual-level | Controlled-access | **OUT OF SCOPE — not used** |

**Provenance model.** Per §6 data model — a `provenance.json` per run + a `datasheet.md` per
dataset (following *Datasheets for Datasets*), both required for release; CI rejects incomplete
chains.

**Privacy / PII stance.** All in-scope data is open-access and **de-identified at source**. We
add no linkage that could re-identify; we do not attempt re-identification; we do not combine
datasets in ways that increase identifiability. Sample-level metadata is used only as published.
If any dataset's "open" status is ambiguous, **default to exclusion** pending verification.

**Attribution.** Every dataset, tool, and reference is cited in releases and reports per its
license (data citations with accessions; software citations; CC-BY attributions). Output code is
**MIT**; output data/reports are **CC-BY-4.0** unless a source's share-alike (e.g. ODbL-style) or
non-commercial term forces a stricter or recipe-only treatment — recorded per artifact.

**Compliance posture.** Conservative by default: when license or consent basis is unclear, the
dataset is **catalogued but not reprocessed/redistributed** until resolved. The license audit
(`*-license-audit-003`) is a hard dependency of every reprocessing task.

---

## Quality, review & risk gates

**Risk tier: medium** for the core (open data, no patient-facing claims, requires domain
accuracy). **High** for any interpretive or patient-facing content — which is **out of scope
here** and, if ever proposed, must move to a high-tier sibling project with credentialed expert
sign-off before merge.

**Required review before a deed is "done":**
- **All tasks:** standard maintainer code/data review + CI green (build, tests, lint, provenance
  lint, checksum verification).
- **License-touching tasks** (any new source, any redistribution decision): **second-reviewer
  license verification** recorded in the datasheet — non-negotiable.
- **Biological-interpretation tasks** (DE results, signature reproduction, meta-analysis
  conclusions): **domain reviewer with bioinformatics/cancer-genomics expertise** signs off that
  methods and caveats are sound. This is the medium-tier expert gate.
- **Any drift toward patient-facing or clinical claims:** STOP — escalate to high tier
  (oncologist + patient-advocate review) and likely relocate out of scope.

**Definition of Shipped.** A deliverable is *shipped* when: acceptance criteria met **and** CI
green **and** maintainer review approved **and** (for interpretation tasks) domain-reviewer
sign-off **and** provenance/license chain complete **and** the artifact is published openly with
its license **and**, for the project to claim *real-world outcome*, a **steward/beneficiary has
received and confirmed use** of it. Until a partner exists, the last clause is openly **unmet**;
we ship *infrastructure* honestly and mark outcome lines as pending.

---

## Roadmap & milestones

Phased, with measurable EXIT CRITERIA. M0 is a deliberately thin, license-clean cold-start.

**M0 — Foundation & cold-start (provenance + one reproduced dataset).**
Goal: stand up the reproducible scaffold, verify licenses, and reproduce **one** open Ewing
dataset end-to-end as the proof of concept.
Exit criteria: repo scaffold + CI green on a synthetic fixture; ≥15 datasets catalogued with
license field populated; per-source **license audit** complete and second-reviewed; datasheet
template ratified; **one** open Ewing dataset reprocessed end-to-end with a complete
`provenance.json`; reproducibility harness (containers pinned by digest, conda/renv lockfiles)
in place.

**M1 — Core reanalysis pipeline.**
Goal: a third-party-runnable quantification + QC + DE pipeline.
Exit criteria: nf-core-based quant pipeline runs on ≥3 open datasets; QC/outlier module +
recorded inclusion lists; DE module with ≥2 engines (DESeq2 + edgeR/limma); validation report
comparing to ≥1 published result with a concordance metric; provenance manifest auto-generated
and CI-linted; documented one-command re-run.

**M2 — Harmonization, signature & meta-analysis.**
Goal: cross-dataset, honestly-confounded harmonization and reproduction of the EWSR1-FLI1
signature.
Exit criteria: ≥5 datasets harmonized with explicit batch-vs-biology confound report;
EWSR1-FLI1 expression signature reproduced (or discrepancy documented) with concordance metric;
meta-analysis where appropriate, with limitations; domain-reviewer sign-off on interpretation.

**M3 — Reproducible release (FAIR + DOI + review).**
Goal: a citable, FAIR, expert-reviewed public release.
Exit criteria: Quarto report site published; Zenodo DOI minted; WorkflowHub entry; FAIR
checklist passed; all released assertions provenance-complete; domain-reviewer + maintainer
sign-off; outreach to ≥3 candidate stewards begun and logged.

**M4 — Sustaining & extensions (backlog; see TASKS).**
Goal: maintenance cadence + opt-in extensions (fusion-aware quant exploration, signature-scoring
CLI, single-cell handoff spec). No patient-facing scope without high-tier relocation.

Dependencies: M1 depends on M0's license audit + harness; M2 depends on M1's pipeline + provenance;
M3 depends on M2's signed-off results.

---

## Work breakdown

The itemized, schema-mapped backlog lives in **`TASKS.md`** — one section per milestone (M0–M3 +
backlog), each with a task table (`ID | Title | Type | Size | Risk | Deliverable | Depends on |
Reviewer`), acceptance criteria for the most important tasks, a Definition of Done per milestone,
and a complete schema-valid example Task JSON. ~18 tasks across milestones in the first robust
backlog; `verifiedNeed: false` throughout until a steward is secured.

---

## Governance, roles & stakeholders

- **Maintainer (Owner): TBD** — accountable for the repo, releases, and license discipline.
- **Reviewers (rotation): TBD (≥2)** — code/data review + CI gate; one must perform second-reviewer
  **license verification** on license-touching tasks.
- **Domain reviewer(s): TBD (bioinformatics / cancer-genomics)** — the medium-tier expert gate for
  any biological interpretation; must sign off DE/signature/meta-analysis conclusions.
- **High-tier reviewers (only if scope ever drifts): oncologist + patient-advocate** — required
  *before merge* for any patient-facing/clinical content; default posture is that such content
  does not belong in this repo.
- **Steward (last-mile owner): TO BE SECURED** — the partner lab/foundation that will actually
  *use* an output; their confirmation is what flips an outcome from "shipped" to "delivered."
- **Requestor / partner: TO BE SECURED** — none yet; outreach tracked in `governance/partners.md`.
- **Elyos board / community** — governs edge cases, the good-deed definition, and COI/veto.

---

## Dependencies & integrations

- **External data archives:** GEO, SRA/ENA, GDC/TARGET (open tier), recount3, ARCHS4, Expression
  Atlas, Treehouse (public subset), DepMap.
- **Reference data:** GENCODE/Ensembl (pinned release + checksum).
- **Tooling/upstream OSS:** nf-core/rnaseq, Nextflow/Snakemake, salmon, STAR, tximport, DESeq2,
  edgeR, limma, fgsea, FastQC, MultiQC, conda/mamba + conda-lock, renv, Docker/Apptainer.
- **Publishing:** Zenodo (DOI), WorkflowHub, GitHub (Actions CI + Pages for the report site).
- **Elyos pieces:** task schema (`packages/schema`), CLI/registry, the donated-lane workspace
  flow; optional funded lane via `packages/runner` (capped) for compute-heavy re-quantification.
- **Flagged/avoided:** COSMIC, OncoKB, MSigDB restricted subsets (recipe-reference only, never
  re-hosted); dbGaP/EGA/biobanks (out of scope).

---

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| A "public" dataset is actually controlled-access or has unclear consent | Medium | High | Default-exclude until license+consent verified; second-reviewer sign-off; out-of-scope list enforced | Reviewers |
| License misclassification → unlawful redistribution (COSMIC/OncoKB/MSigDB) | Medium | High | Recipe-first redistribution; per-source audit; CI blocks release without verified `license` field | Maintainer |
| Re-identification via dataset linkage | Low | High | No linkage that raises identifiability; no re-ID attempts; aggregate/de-identified only | Domain reviewer |
| Results misread as clinical/biomarker advice | Medium | High | Prominent "research, not medical advice" framing; no patient-facing content; interpretation gated by domain reviewer | Maintainer |
| Irreproducibility (versions/seeds drift) | Medium | Medium | Digest-pinned containers, lockfiles, seeds, provenance manifest as CI gate | Maintainer |
| Compute cost of re-quantification | Medium | Medium | Prefer recount3/ARCHS4 reprocessed data; subsample CI fixture; cap funded-lane budgets hard | Maintainer |
| Published result can't be reproduced (genuine discrepancy) | Medium | Medium | Treat as a *finding*: document discrepancy + caveats, don't bury; never assert original was wrong without evidence | Domain reviewer |
| No partner/steward secured → no real-world outcome | High | Medium | Honest `verifiedNeed:false`; begin outreach in M3; ship infrastructure value regardless | Maintainer |
| Over-claiming novel biology | Medium | High | Non-goal enforced; reproduction-framed language; domain review of all claims | Domain reviewer |
| Upstream tool/archive API changes break pipeline | Medium | Low | Pin versions; record fetch URLs+checksums; CI canary on fixture | Reviewers |

---

## Security & privacy

- **Threat surface:** primarily *data-handling correctness* (license/consent, no controlled data,
  no re-ID), and supply-chain (containers/dependencies). No user accounts, no PII collection, no
  live service in v1.
- **Secrets handling:** none required for open data; **no tokens/keys in logs, receipts, or
  commits** (per CLAUDE.md). Any optional funded-lane API key lives only in the runner's
  environment with a hard per-task budget cap and is never written to artifacts.
- **PII:** in-scope data is open + de-identified; we add no identifiers and attempt no
  re-identification. Ambiguous-consent data is excluded by default.
- **Supply chain:** digest-pinned images, lockfiles, dependency provenance; CI verifies input
  checksums before processing.
- **Abuse/misuse prevention:** refusal guardrails apply — no surveillance, no re-ID, no
  clinical-advice repackaging. Outputs labelled research-only. A `SECURITY.md` + responsible-use
  note ships with the repo.

---

## Sustainability & maintenance

- **After delivery:** the maintainer + reviewer rotation keep CI green, refresh pinned references
  on a scheduled cadence, and re-run the fixture on dependency updates. Each release is a frozen,
  DOI'd artifact so reproducibility survives even if maintenance lapses.
- **Outcome tracking:** Zenodo download stats, GitHub forks/issues/citations, and a
  `OUTCOMES.md` log; when a steward confirms real use, the relevant outcome flips to delivered.
- **Bus-factor:** lockfiles + digest-pinned containers + DOI snapshots mean any third party can
  reconstruct the environment; nothing depends on a single person's machine.
- **Cost:** mostly donated compute; funded lane only for bounded re-quantification jobs with
  per-task escrow caps.

---

## Open questions

1. **Steward/partner:** which lab or foundation will actually adopt and use the output? (Blocks
   `verifiedNeed → true` and real-world outcome claims.)
2. **Redistribution rulings:** for each high-value dataset, may we republish derived matrices, or
   recipe-only? (Per-source audit resolves.)
3. **Orchestrator:** Nextflow vs Snakemake — lock in M0 (leaning Nextflow + nf-core/rnaseq).
4. **Which "EWSR1-FLI1 signature" to reproduce** as the canonical reference? (Pick a well-cited,
   open one with reproducible inputs.)
5. **Microarray scope:** include legacy array series in v1, or RNA-seq-only first?
6. **Funded lane:** is any re-quantification compute-heavy enough to justify escrow, and at what
   cap?
7. **Treehouse / TARGET open-tier Ewing coverage:** confirm exactly which Ewing samples are truly
   open-tier before relying on them.

---

## References

- Elyos: `CLAUDE.md` (work rules, lanes, guardrails); `docs/good-deed-definition.md` (criteria +
  risk tiers); `packages/schema/src/schemas.ts` (Task schema); `planning/ROADMAP.md` (Track 8a).
- Data archives: NCBI GEO; SRA/ENA; GDC / TARGET (open tier) under NIH Genomic Data Sharing;
  recount3; ARCHS4; EBI Expression Atlas; UCSC Treehouse Childhood Cancer Initiative; DepMap/CCLE.
- Methods/tooling: nf-core/rnaseq; Nextflow; salmon; STAR; tximport; DESeq2; edgeR; limma-voom;
  fgsea; FastQC/MultiQC; conda-lock; renv; Apptainer.
- Standards: *Datasheets for Datasets* (Gebru et al.); FAIR data principles (Wilkinson et al.);
  data- and software-citation practice.
- Flagged-restricted (recipe-reference only, not re-hosted): COSMIC; OncoKB; MSigDB
  (KEGG/curated subsets).

---

## Appendix A — Improvements applied

25 specific improvements identified against the first draft and **applied** above.

1. **Led the data/licensing section with the binding cancer guardrails** as a callout block, so
   the controlled-access prohibition is impossible to miss (was buried mid-section).
2. **Added an explicit "default-exclude on ambiguity" rule** for any dataset whose open status or
   consent basis is unclear — conservative-by-default made operational.
3. **Made the per-source license table a "starting assumption, not final ruling,"** with a hard
   dependency on the M0 license audit before any reprocessing.
4. **Flagged MSigDB** (not just COSMIC/OncoKB) because its KEGG/curated subsets are restricted —
   a common, easily-missed redistribution trap in enrichment analysis.
5. **Adopted recipe-first redistribution as a locked decision** (republish pipeline + checksums +
   accessions by default; derived data only when clearly permitted) — turns license risk into a
   design default.
6. **Promoted provenance to a CI gate** (`provenance.json` lint blocks release) rather than a
   documentation nicety.
7. **Specified a synthetic/subsampled CI fixture** so real (large) data isn't needed to keep CI
   green and the pipeline stays continuously runnable.
8. **Required ≥2 DE engines** (DESeq2 + edgeR/limma) so results aren't an artifact of one method.
9. **Preferred recount3/ARCHS4 (uniformly reprocessed) substrates** to cut compute cost *and*
   reduce alignment-driven variance — a reproducibility *and* sustainability win.
10. **Reframed "irreproducible published result" as a documented finding**, with an explicit rule
    never to assert the original was wrong without evidence — protects scientific fairness.
11. **Added a concordance metric requirement** (rank correlation / DE-set overlap) so
    "reproduced" is quantitative, not a vibe.
12. **Separated medium-tier (domain reviewer) from high-tier (oncologist + advocate) gates** and
    stated the core has no patient-facing content — clean risk-tier mapping.
13. **Added an explicit "STOP and escalate" rule** if scope drifts toward clinical/patient-facing
    claims (relocate to a high-tier sibling project).
14. **Made `verifiedNeed: false` and the unmet "handed to beneficiary" clause explicit and
    honest** throughout, rather than implying a partner exists.
15. **Added a steward/last-mile owner role** distinct from maintainer/reviewer, and tied the
    real-world-outcome metric to steward confirmation.
16. **Distinguished direct vs indirect beneficiaries** and explicitly refused to over-claim direct
    patient impact — honest tone per the spec.
17. **Added a confound report (batch vs biology) as a required harmonization output**, so
    cross-dataset claims aren't silently confounded.
18. **Recorded the sample inclusion/exclusion list as a first-class artifact** — naming the single
    most common irreproducibility source.
19. **Pinned references by URL + checksum + release**, never vendored without a license check.
20. **Added a FAIR self-assessment checklist + DOI as an explicit M3 exit criterion**, making
    "open scientific resource" measurable.
21. **Hardened secrets posture for the optional funded lane** (key only in runner env, hard
    per-task cap, never in artifacts) per CLAUDE.md.
22. **Added bus-factor mitigation** (lockfiles + digest-pinned containers + DOI snapshots) so
    reproducibility survives maintainer turnover.
23. **Clarified the agent-neutral-core boundary**: scientific pipeline in Nextflow/R/Python is
    correct domain tooling; Elyos-facing glue stays TS/ESM, no vendor logic in core.
24. **Tightened non-goals** to explicitly exclude single-cell/proteomics/trial/family-guide work,
    routing them to named sibling projects to prevent scope creep.
25. **Added an outreach-to-stewards task into M3** (≥3 candidates, logged) so the partner gap is
    actively worked, not just acknowledged.

---

## Review sign-off

**Reviewer:** senior staff engineer + TPM (drafting author self-review against PLAN_SPEC,
CLAUDE.md, good-deed-definition, and the binding cancer guardrails).

**Completeness check.** All 17 required H2 sections present and in spec order; Data/licensing
section leads with the binding cancer guardrails as required; Appendix A (25 applied improvements)
and this sign-off added. TASKS.md exists with milestone tables, acceptance criteria, per-milestone
DoD, a schema-valid example Task JSON, and `verifiedNeed:false` throughout.

**Correctness check.**
- **Guardrails:** controlled-access (dbGaP/EGA/biobanks) and identifiable data explicitly out of
  scope; open/aggregate/de-identified only; COSMIC/OncoKB/MSigDB flagged non-redistributable;
  GEO/GDC-open treated as open with attribution. ✓
- **No medical advice:** core produces none; any patient-facing content gated to high tier
  (oncologist + advocate) and relocated out of repo. ✓
- **Provenance:** required on every assertion and enforced as a CI gate. ✓
- **Honesty:** no partner invented; `verifiedNeed:false`; real-world-outcome clause openly unmet;
  outcome-based metrics, not vanity counts. ✓
- **Elyos fit:** lanes (donated primary, funded capped) respected; agent-neutral-core boundary
  respected; schema-mapped tasks; quality bar "delivered, not merged" honored. ✓

**Fixes applied during review.** (a) Added explicit "default-exclude on ambiguity" cross-reference
between §5 and §7; (b) ensured every Risks row has an owner; (c) aligned success-metric #7 with
the steward role and the Definition of Shipped's last clause; (d) confirmed the funded-lane
budget-cap language matches CLAUDE.md's "never exceed escrow."

**Outstanding human decisions (cannot be resolved in-plan):** secure a steward/partner (Open
Q1); per-dataset redistribution rulings (Q2); orchestrator lock (Q3); canonical signature choice
(Q4). None block M0.

**Verdict:** Plan is internally consistent, guardrail-compliant, and ready to drive M0 task
creation. Status remains **Draft v0.1.0** pending maintainer assignment and partner outreach.
