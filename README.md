# ewing-expression-reanalysis

> Ewing Sarcoma is a rare, aggressive bone-and-soft-tissue cancer that mostly strikes children, teenagers, and young adults. It is driven in ~85% of cases by the **EWSR1-FLI1** fusion oncogene (and in m  ·  **Risk tier:** med  ·  **Status:** planning

Ewing Sarcoma is a rare, aggressive bone-and-soft-tissue cancer that mostly strikes children, teenagers, and young adults. It is driven in ~85% of cases by the **EWSR1-FLI1** fusion oncogene (and in most of the rest by **EWSR1-ERG** or other EWSR1-ETS fusions). Because it is rare, the public molecular data that exists is precious — and chronically **under-reused**. Dozens of transcriptome studies sit in public archives (GEO, SRA/ENA, GDC/TARGET open tier, recount3, ARCHS4, Expression Atlas, Treehouse, DepMap), each processed with a different, often under-documented pipeline, different reference genome, different normalization, and different software versions. The result: published expression findings are **hard to reproduce, hard to compare across studies, and hard for the next researcher to build on.**

**Definition of shipped:** design (§8).

This is an **Elyos** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/elyos

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
elyos browse
elyos next --repo Elyos-Projects/ewing-expression-reanalysis --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
