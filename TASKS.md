# ewing-expression-reanalysis — TASKS

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) ·
> Lane: donated (primary); funded (optional, capped) · Project: `ewing-expression-reanalysis`
> Risk tier: medium (core). See `PLAN.md` for roadmap, guardrails, and Definition of Shipped.

---

## How these tasks map to Elyos

Each task below becomes an **Elyos Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug `ewing-rna-<area>-NNN`.
- `title` — short imperative.
- `project` — always `"ewing-expression-reanalysis"`.
- `type` — one of `code | research | writing | data | design-spec | maintenance`.
- `lane` — `donated` (default) or `funded` (only compute-heavy re-quantification; **requires
  `fundedBudgetUsd` cap**).
- `priority` — `high | medium | low`.
- `domain` — string array, e.g. `["cancer-research","bioinformatics","reproducibility"]`.
- `riskTier` — `low | medium | high`. Scaffolding/docs = `low`; data/pipeline/interpretation =
  `medium`. No `high` tasks here — patient-facing content is out of scope (PLAN §5/§8).
- `urgent` — boolean (all `false`; rare-disease research, not time-critical).
- `deliverable` — `pr | dataset | document | translation`.
- `tokenEstimate` — `small | medium | large`.
- `status` — `open | in-progress | review | delivered | done`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` (required, non-empty),
  `requestor`, `verifiedNeed`, `outputLicense`.
- **`verifiedNeed: false` on every task** — no partner/steward secured yet (PLAN §2).
- **`outputLicense`** — `MIT` for code, `CC-BY-4.0` for data/docs (subject to per-source license
  audit; recipe-first redistribution per PLAN §7).

License/consent reminder (binding): **open-access / de-identified data only**; controlled-access
(dbGaP/EGA/biobanks) out of scope; COSMIC/OncoKB/MSigDB-restricted = recipe-reference only.

---

## Milestone M0 — Foundation & cold-start

Goal: reproducible scaffold + verified licenses + ONE open Ewing dataset reproduced end-to-end.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-rna-repo-scaffold-001 | Repo scaffold + CI on synthetic fixture | code | small | low | pr | — | Maintainer |
| ewing-rna-data-catalog-002 | Catalog ≥15 candidate open Ewing transcriptomes | data | medium | medium | dataset | 001 | Domain reviewer |
| ewing-rna-license-audit-003 | Per-source license + consent audit | research | medium | medium | document | 002 | Reviewer (license) |
| ewing-rna-datasheet-template-004 | Datasheet + provenance-manifest spec | design-spec | small | low | document | 001 | Maintainer |
| ewing-rna-repro-harness-005 | Containers (digest-pinned) + lockfiles + provenance lint | code | medium | medium | pr | 001,004 | Maintainer |
| ewing-rna-reference-repro-006 | Reproduce ONE open dataset end-to-end | data | large | medium | dataset | 003,005 | Domain reviewer |

**Acceptance criteria — key tasks**

- **ewing-rna-repo-scaffold-001**
  - [ ] Monorepo skeleton (pipeline dir + Elyos TS/ESM glue) with `README`, `LICENSE` (MIT),
        `SECURITY.md`, responsible-use note.
  - [ ] GitHub Actions CI runs build + lint + a **synthetic/subsampled fixture** through the
        pipeline stub and goes green.
  - [ ] No `:latest` container tags; no secrets in repo/logs.
- **ewing-rna-license-audit-003**
  - [ ] Every source in the catalog has `license`, `licenseVerifiedBy` (2nd reviewer), and a
        `redistributionAllowed` ruling with rationale.
  - [ ] COSMIC/OncoKB/MSigDB-restricted explicitly marked recipe-reference-only;
        dbGaP/EGA/biobanks marked out-of-scope.
  - [ ] Any ambiguous-consent/licence dataset marked **default-exclude** pending resolution.
- **ewing-rna-reference-repro-006**
  - [ ] One open dataset quantified end-to-end (acquisition → quant → QC → provenance).
  - [ ] Complete `provenance.json` (input/output checksums, container digests, versions, seed).
  - [ ] One-command re-run documented and verified by reviewer on a clean checkout.

**Definition of Done (M0):** scaffold + CI green on fixture; ≥15 datasets catalogued with
verified licenses; datasheet/provenance spec ratified; reproducibility harness in place; one open
dataset reproduced with complete provenance; all M0 PRs reviewed, signed off (`-s`/DCO), and
released openly. (Real-world "delivered" remains pending — no steward yet.)

---

## Milestone M1 — Core reanalysis pipeline

Goal: third-party-runnable quantification + QC + differential-expression pipeline.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-rna-quant-pipeline-101 | nf-core-based quant pipeline (≥3 datasets) | code | large | medium | pr | 005,006 | Maintainer |
| ewing-rna-qc-module-102 | QC/MultiQC + outlier + inclusion-list module | code | medium | medium | pr | 101 | Domain reviewer |
| ewing-rna-de-module-103 | DE module (DESeq2 + edgeR/limma) | code | large | medium | pr | 101 | Domain reviewer |
| ewing-rna-validation-104 | Validate vs ≥1 published result (concordance) | research | medium | medium | document | 103 | Domain reviewer |
| ewing-rna-provenance-manifest-105 | Auto provenance manifest + CI lint gate | code | medium | medium | pr | 101 | Maintainer |

**Acceptance criteria — key tasks**

- **ewing-rna-quant-pipeline-101**
  - [ ] Runs on ≥3 open datasets via Nextflow (nf-core/rnaseq backbone), references pinned by
        release + checksum.
  - [ ] Prefers recount3/ARCHS4 reprocessed inputs where available; re-quantifies from open
        SRA/ENA only when license permits.
  - [ ] Reproducible: digest-pinned containers + lockfiles + seeds; documented one-command run.
- **ewing-rna-de-module-103**
  - [ ] ≥2 DE engines produce results for the same contrast; agreement reported.
  - [ ] Contrasts use only published/open sample metadata; inclusion list recorded.
  - [ ] No clinical/biomarker language; results labelled research-only.
- **ewing-rna-validation-104**
  - [ ] ≥1 published Ewing DE/expression result reproduced with a **quantitative concordance
        metric** (rank correlation / DE-set overlap).
  - [ ] Discrepancies documented as findings with caveats; no claim the original was wrong absent
        evidence.

**Definition of Done (M1):** pipeline runs on ≥3 datasets; QC + DE (≥2 engines) modules merged;
≥1 validation report with concordance metric + domain sign-off; provenance manifest auto-generated
and CI-linted; one-command re-run verified.

---

## Milestone M2 — Harmonization, signature & meta-analysis

Goal: honestly-confounded cross-dataset harmonization + EWSR1-FLI1 signature reproduction.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-rna-harmonize-201 | Harmonize ≥5 datasets (matrices + metadata) | data | large | medium | dataset | 101,105 | Domain reviewer |
| ewing-rna-batch-correction-202 | Batch correction + batch-vs-biology confound report | code | medium | medium | pr | 201 | Domain reviewer |
| ewing-rna-signature-204 | Reproduce EWSR1-FLI1 expression signature | research | large | medium | document | 202,103 | Domain reviewer |
| ewing-rna-meta-analysis-203 | Cross-dataset meta-analysis (where valid) | research | medium | medium | document | 202 | Domain reviewer |

**Acceptance criteria — key tasks**

- **ewing-rna-harmonize-201**
  - [ ] ≥5 datasets harmonized to a common gene space + annotation version; provenance per cell
        of origin preserved.
  - [ ] Redistribution of harmonized matrices only where every contributing source permits;
        otherwise recipe + accessions published instead.
- **ewing-rna-signature-204**
  - [ ] A well-cited, open EWSR1-FLI1 signature reproduced (or discrepancy documented) with a
        concordance metric and explicit limitations.
  - [ ] Framed as reproduction/replication, **not** novel discovery or clinical evidence.
- **ewing-rna-batch-correction-202**
  - [ ] Confound report quantifies batch vs biology before/after correction; over-correction risk
        discussed.

**Definition of Done (M2):** ≥5 datasets harmonized with confound report; EWSR1-FLI1 signature
reproduced/documented with concordance; meta-analysis (or a reasoned decision not to) delivered;
domain-reviewer sign-off on all interpretation; provenance complete.

---

## Milestone M3 — Reproducible release (FAIR + DOI + review)

Goal: a citable, FAIR, expert-reviewed public release, and active steward outreach.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-rna-quarto-reports-301 | Quarto report site (reproducible) | writing | medium | medium | document | 104,204 | Domain reviewer |
| ewing-rna-zenodo-release-302 | Zenodo DOI + WorkflowHub entry + release freeze | maintenance | small | low | pr | 301 | Maintainer |
| ewing-rna-fair-audit-304 | FAIR self-assessment + provenance completeness audit | maintenance | small | low | document | 301 | Reviewer |
| ewing-rna-steward-outreach-303 | Outreach to ≥3 candidate stewards (logged) | writing | small | low | document | 301 | Maintainer |

**Acceptance criteria — key tasks**

- **ewing-rna-quarto-reports-301**
  - [ ] Report site rebuilds from source in CI; every figure/table traces to a provenance
        manifest.
  - [ ] Prominent "research, not medical advice" framing; full data + software citations.
- **ewing-rna-zenodo-release-302**
  - [ ] Tagged release minted with a Zenodo DOI; WorkflowHub entry created; artifacts checksummed.
  - [ ] No restricted-source bytes in the release (audit passes).
- **ewing-rna-steward-outreach-303**
  - [ ] ≥3 candidate labs/foundations contacted; responses logged in `governance/partners.md`;
        any confirmed use flips that outcome's `verifiedNeed → true`.

**Definition of Done (M3):** report site published; Zenodo DOI minted; FAIR checklist passed;
provenance 100% complete; domain + maintainer sign-off; steward outreach begun and logged.
"Delivered to beneficiary" remains conditional on a steward confirming use.

---

## Backlog / future (M4 — sized, unscheduled)

| ID | Title | Type | Size | Risk | Deliverable | Notes |
|---|---|---|---|---|---|---|
| ewing-rna-fusion-aware-401 | Explore fusion-aware quantification (research only) | research | large | medium | document | Method exploration; NOT a diagnostic |
| ewing-rna-signature-cli-402 | Signature-scoring CLI on user-supplied open data | code | medium | medium | pr | Research tool; refuses non-open/clinical use |
| ewing-rna-scrna-handoff-403 | Single-cell integration handoff spec | design-spec | medium | medium | document | Hands to `ewing-single-cell-atlas` |
| ewing-rna-maintenance-404 | Scheduled reference/dependency refresh + CI canary | maintenance | small | low | pr | Sustainability cadence |
| ewing-rna-microarray-405 | Legacy microarray series reanalysis module | code | large | medium | pr | If M0 decides arrays in-scope |

Out of scope here (separate, higher-risk projects): patient-facing guides, trial/eligibility
tooling, drug-target *recommendations*, controlled-access analyses.

---

## Example task JSON

Complete, schema-valid Task JSON for the first M0 task (`ewing-rna-repo-scaffold-001`):

```json
{
  "id": "ewing-rna-repo-scaffold-001",
  "title": "Scaffold reproducible repo + CI on a synthetic fixture",
  "project": "ewing-expression-reanalysis",
  "type": "code",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "bioinformatics", "reproducibility", "ewing-sarcoma"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "pr",
  "tokenEstimate": "small",
  "status": "open",
  "context": "Cold-start for a reproducible reanalysis project over PUBLIC, open-access Ewing Sarcoma transcriptomes. Binding guardrails: open/aggregate/de-identified data only; controlled-access (dbGaP/EGA/biobanks) and identifiable patient data are out of scope; COSMIC/OncoKB/MSigDB-restricted are recipe-reference only; no medical advice; provenance on every assertion. This task builds only the empty, license-clean scaffold and CI — no real patient data is touched.",
  "objective": "Create the repository skeleton (Nextflow/R/Python pipeline dir + TypeScript/ESM Elyos glue), governance/license files, and a GitHub Actions CI workflow that runs build + lint and pushes a tiny synthetic/subsampled fixture through a pipeline stub to green.",
  "acceptanceCriteria": [
    "Repo skeleton committed: pipeline directory, Elyos TS/ESM glue, README, LICENSE (MIT), SECURITY.md, and a research-only / not-medical-advice responsible-use note.",
    "GitHub Actions CI runs build, lint, and a synthetic/subsampled fixture through the pipeline stub and passes (green).",
    "No container uses a ':latest' tag; container references are pinned by digest where present.",
    "No secrets, tokens, or API keys appear in the repo, logs, or CI output.",
    "Provenance-manifest and datasheet placeholders are present and referenced by the README.",
    "Commit is DCO signed-off (git commit -s)."
  ],
  "resources": [
    "C:/code/elyos/CLAUDE.md",
    "C:/code/elyos/docs/good-deed-definition.md",
    "C:/code/elyos/packages/schema/src/schemas.ts",
    "planning/projects/ewing-expression-reanalysis/PLAN.md",
    "https://nf-co.re/rnaseq"
  ],
  "output": "A pull request adding the reproducible repository scaffold and a green CI workflow running a synthetic fixture end-to-end.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "MIT"
}
```

---

## Task count summary

- M0: 6 · M1: 5 · M2: 4 · M3: 4 · Backlog: 5 → **24 tasks** (19 scheduled M0–M3 + 5 backlog).
- All `verifiedNeed: false`; all `urgent: false`; no `high`-risk tasks (patient-facing out of
  scope); funded lane unused in the scheduled set (would require `fundedBudgetUsd` if added for
  re-quantification).

---

## Generated task index

All 24 backlog rows are now materialized as schema-valid `tasks/<id>.json` (validated against
`packages/schema/src/schemas.ts`; `filename == id`; no duplicate ids; no extra keys). The
machine-readable JSON set — not this table — is what the Elyos CLI executes.

**Fan-out:** none. Each `tasks/*.json` is a single representative task per backlog row. The
candidate-dataset dimension ("≥15 open transcriptomes", "≥3 / ≥5 datasets") is **not** enumerated
to named accessions in `PLAN.md`/`TASKS.md`, so no per-dataset fan-out was fabricated; the
catalog/license-audit/harmonize tasks bound that set at execution time once sources are verified.
Per-item dataset tasks expand on catalog + license-audit confirmation (M0).

**Guardrails carried into every task:** open-access / de-identified data only; controlled-access
(dbGaP/EGA/biobanks) out of scope; COSMIC/OncoKB/MSigDB-restricted are recipe-reference only;
research-only, not medical advice; provenance on every assertion. No `high`-risk / patient-facing
tasks were authored (out of scope per PLAN §5/§8).

| Milestone | Task ids |
|---|---|
| M0 | `ewing-rna-repo-scaffold-001` (seed), `ewing-rna-data-catalog-002`, `ewing-rna-license-audit-003`, `ewing-rna-datasheet-template-004`, `ewing-rna-repro-harness-005`, `ewing-rna-reference-repro-006` |
| M1 | `ewing-rna-quant-pipeline-101`, `ewing-rna-qc-module-102`, `ewing-rna-de-module-103`, `ewing-rna-validation-104`, `ewing-rna-provenance-manifest-105` |
| M2 | `ewing-rna-harmonize-201`, `ewing-rna-batch-correction-202`, `ewing-rna-meta-analysis-203`, `ewing-rna-signature-204` |
| M3 | `ewing-rna-quarto-reports-301`, `ewing-rna-zenodo-release-302`, `ewing-rna-steward-outreach-303`, `ewing-rna-fair-audit-304` |
| M4 (backlog) | `ewing-rna-fusion-aware-401`, `ewing-rna-signature-cli-402`, `ewing-rna-scrna-handoff-403`, `ewing-rna-maintenance-404`, `ewing-rna-microarray-405` |

Acceptance criteria for rows not already itemized above (002, 004, 005, 101, 102, 105, 201, 202,
203, 301, 302, 303, 304, 401–405) were authored in their JSON from each milestone's Definition of
Done and row intent; the explicitly-itemized "key tasks" criteria above were reused verbatim into
their JSON.
