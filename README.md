# translation-memory-commons

> Shared open translation-memory + glossary infrastructure reused across Hee-Lee Oss translation projects.  ·  **Risk tier:** med  ·  **Status:** planning

`translation-memory-commons` (TMC) is shared, **open** translation infrastructure for the Hee-Lee Oss language track: a license-clean, provenance-tracked **translation memory** (reusable source→target segment pairs) and **glossary/termbase** (terminology, do-not-translate items, definitions), built on open interchange standards (**TMX 1.4b** for memory, **TBX-Basic / ISO 30042** for terminology) and reused across every Hee-Lee Oss translation project — `vital-info-translations`, `public-domain-translations`, `health-info-translations`, `localization-for-good`, `irish-gaeilge-learning`, `emergency-phrasebooks`, and others.

**Definition of shipped:** Reusable open TM/glossary adopted by multiple translation projects.

This is a **Hee-Lee Oss** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Get started: https://github.com/Hee-Lee-Oss-Projects/hee-lee-oss-downloads

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
hee-lee-oss browse
hee-lee-oss next --repo Hee-Lee-Oss-Projects/translation-memory-commons --no-fork
```

## Licensing & review
- Data CC-BY/CC0; code MIT.
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
