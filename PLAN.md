# PLAN — translation-memory-commons

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: J. Carter (acting maintainer) · Lane: donated

## Executive summary

`translation-memory-commons` (TMC) is shared, **open** translation infrastructure for the Hee-Lee Oss
language track: a license-clean, provenance-tracked **translation memory** (reusable
source→target segment pairs) and **glossary/termbase** (terminology, do-not-translate items,
definitions), built on open interchange standards (**TMX 1.4b** for memory, **TBX-Basic /
ISO 30042** for terminology) and reused across every Hee-Lee Oss translation project —
`vital-info-translations`, `public-domain-translations`, `health-info-translations`,
`localization-for-good`, `irish-gaeilge-learning`, `emergency-phrasebooks`, and others.

**Positioning.** TMC is not a translation engine and not a CAT tool. It is the *memory layer*
that sits beneath the translation projects and makes each one faster, more consistent, and more
correctly attributed than it could be alone. The value is leverage and consistency: a term
translated and reviewed once for one project is available — with its full provenance and license
intact — to every other project, so the same medical term, place name, or UI string is rendered
the same way everywhere, and a reviewer never re-litigates a decision the commons already settled.

**The defining constraint — and the identity of this project — is trust per segment.** A
translation memory is, by construction, a *mixture*: it aggregates segments from many documents,
many sources, many licenses, and many quality levels into one reusable pool. That is exactly what
makes it dangerous. One mislicensed segment, one segment carrying personal data, or one wrong
medical translation does not stay contained — it propagates into *every* downstream project that
reuses it. So TMC is built around a single rule: **every segment carries its provenance, its
license, its PII status, and its review tier with it, always; these travel together, can be
filtered on at query time, and a segment's review tier can only ever go up, never silently down.**
A TM is only as trustworthy as its worst segment's provenance, so no segment enters without one.

**Lane.** Donated. A human runs their own coding agent interactively to build the tooling, ingest
and curate segments, and open PRs; the Hee-Lee Oss CLI only prepares workspaces and opens PRs. No
funded/API-key execution, no headless runs.

**Risk tier: medium**, for two compounding reasons: (1) **amplification** — an error in a shared
memory is reused everywhere, so the accuracy bar is higher than for a one-off translation; and (2)
**inherited risk** — TMC will hold segments drawn from high-stakes (health/safety) source
projects, and reuse must never strip the expert review those segments required. TMC does not
*author* high-stakes advice (which would be `high`), but it must preserve the review provenance of
content that did.

**Honest status.** No downstream project has yet *formally committed* to consume TMC as a
dependency, and no external partner exists. The *category* need is strongly evidenced (multiple
live Hee-Lee Oss translation projects independently maintain overlapping glossaries today), but a named
**first consumer** with an agreed language pair and acceptance test is **TO BE SECURED**. Until one
exists, `verifiedNeed` is recorded `false` on tasks whose value depends on real adoption, and the
project's Definition of Shipped cannot be fully met. Securing the first consumer is the top
cold-start dependency (see Roadmap M2 and Open questions).

## Problem & beneficiaries

**Who is helped (direct beneficiaries).** The **translators, reviewers, and maintainers of Hee-Lee Oss
translation projects** — the people doing the actual language work. Today each project starts cold:
it builds its own glossary, re-translates segments other projects already translated, and re-makes
terminology decisions in isolation. TMC gives them a shared, reviewed memory to draw from, cutting
duplicated effort and inconsistency.

**Who is helped (ultimate beneficiaries).** The **end-readers** of those translations — a parent
reading translated first-aid guidance, a learner reading a public-domain book in a low-resource
language, a refugee reading an emergency phrasebook. They benefit indirectly but materially:
translations arrive faster, render key terms consistently across documents (so "do not induce
vomiting" or a drug name is phrased the same way in every leaflet they encounter), and carry
correct attribution.

**Who is helped (wider community).** If published as an open commons, the broader open-translation
and language-revitalization community (e.g., Translators without Borders / CLEAR Global,
under-served-language communities) can reuse the license-clean pools — a public good beyond Hee-Lee Oss.

**The need.** Translation memory and termbases are standard, decades-old professional tooling, but
**open, license-clean, provenance-rich** TM/glossary commons are rare. Existing large bitext
corpora (OPUS, ParaCrawl, etc.) are invaluable but are *web-scraped at scale with mixed,
often-unverified licensing and known PII contamination* — unusable as-is for a project that must
guarantee per-segment license cleanliness and privacy. Meanwhile, inside Hee-Lee Oss, several
translation projects are duplicating glossary work and drifting apart on terminology. The gap is a
**small, curated, fully-attributed, PII-clean, quality-tiered** commons that downstream projects
can trust without re-verifying every segment.

**Verified need: TO BE SECURED (internal need strong, formal commitment pending).** The internal,
cross-project need is concrete and observable. What is *not* yet secured is a **named first
consumer project that commits, in writing, to integrate TMC and run a real translation through
it** with an agreed acceptance test. Tasks are sequenced so TMC builds reusable, partner-
independent foundations (standards, schemas, license engine, PII gate, seed glossary) *before* a
consumer is required; adoption/integration tasks are explicitly gated and marked
`verifiedNeed: false` until a first consumer commits.

**Partner / requestor: TO BE SECURED.** Most likely first consumer (internal): the
`vital-info-translations` or `public-domain-translations` maintainer. Candidate external
adopters/partners: Translators without Borders / CLEAR Global, language-revitalization
communities, and OSS localization communities. None is committed as of this draft.

## Goals and non-goals

**Goals**

- Provide a **shared TM + glossary commons** on open standards (TMX 1.4b, TBX-Basic) that any
  Hee-Lee Oss translation project can consume and contribute to.
- Make **provenance, license, PII status, and quality tier first-class, per-segment** metadata that
  travels with every unit and is filterable at query time.
- Guarantee **license cleanliness**: every published segment has verified, compatible license terms
  and required attribution; releases are **partitioned by license class** so nothing is relicensed
  and consumers pick the pool that matches their use.
- Guarantee **privacy**: no segment carrying personal data is published; PII detection is a hard
  ingest gate.
- Improve **terminology consistency** across Hee-Lee Oss translation projects (the same term rendered the
  same way everywhere), measurably.
- Provide **fuzzy + exact matching** so translators get high-quality suggestions, while ensuring
  **safety-critical content is never auto-applied** from a match without in-context human review.
- Deliver real **leverage** to a downstream project: measurable match rate and reviewer-accepted
  reuse in an actual translation.

**Non-goals**

- **Not a machine-translation engine.** TMC stores and serves human/curated memory and terminology;
  it does not train or run an MT model. (It may *record* that a segment was MT-drafted, as quality
  tier T0.)
- **Not a CAT tool, translation editor, or web UI.** TMC is a library + file-based data + CLI
  validators that existing project workflows call; it does not provide an interactive editor.
- **Not a hosted SaaS / live public API server.** The commons is git-friendly files plus standards-
  compliant exports; no always-on service, no accounts, no user data collected.
- **Not an authoring project.** TMC does not write or alter translations; it remembers and serves
  what downstream projects (with their own review) produced. Authoring new high-stakes content is
  out of scope and would be `high` risk.
- **Not a bulk web-bitext mirror.** TMC does **not** ingest OPUS/ParaCrawl/CC-mined corpora
  wholesale; only allow-listed, per-source license-verified, PII-screened material enters.
- **Not a sink for ND or unverifiable content.** NoDerivatives-licensed and unverifiable-license
  segments are excluded by policy (a TM segment is reused to make derivatives).
- **Not handling controlled/private/consented-individual data** of any kind.
- **Not building speech, OCR, or alignment-from-scratch** beyond the minimal sentence segmentation
  needed to create units.

## Success metrics (outcomes)

Outcome-based and reuse-centric. Baselines are ~0 (greenfield). **Outcome metrics** apply after a
first consumer is secured (M2+); **interim foundation metrics** are tracked from M0/M1 so progress
is visible before a consumer exists. We explicitly **do not** count "segments ingested," "PRs
merged," or "TUs in the database" as success — only **reviewed, license-clean memory actually
reused by a translation project to help real readers**.

**Outcome metrics (post first consumer)**

| Metric | Baseline | Target | How measured |
|---|---|---|---|
| Hee-Lee Oss translation projects **actively consuming** TMC (real translation run through it) | 0 | ≥ 2 by end of M3 | Consumer confirmation in PR/receipt; project registry |
| **Reuse / match rate** — share of a new translation's segments served by a fuzzy (≥ 75%) or exact match from the commons | 0 | ≥ 25% on the first integrated document set (**effective from M2**) | Match-report emitted by the matcher per run |
| **Reviewer-accepted reuse** — share of offered matches a qualified reviewer accepts (after in-context review) | n/a | ≥ 70% of *non-safety-critical* offers; safety-critical always re-reviewed | Reviewer logs |
| **Terminology consistency** — agreement on shared terms across ≥ 2 consuming projects | n/a | ≥ 95% of shared glossary terms rendered identically | Cross-project glossary diff report |
| **License/attribution compliance** of published units (valid provenance + compatible license + attribution) | n/a | 100% (hard gate; **automated from M1**) | License-gate CI per export |
| **PII leakage** in published pools | n/a | 0 (hard gate; **automated from M1**) | PII-scan CI per export |
| **Propagated defects** — errors traced to a reused TMC segment, found after reuse | n/a | 0 critical; trend down | Propagated-defect log (`tmc-outcomes-001`) |
| **Reviewed coverage** — published TUs at quality tier ≥ T1 (human-reviewed) | n/a | ≥ 90% of published TUs | Quality-tier field census |

**Interim foundation metrics (M0/M1, consumer-independent)**

| Metric | Baseline | Target | How measured |
|---|---|---|---|
| Sources **license-verified** and allow-listed | 0 | ≥ 3 by end of M0 | Allow-list entries with `verifiedBy`/`verifiedDate`/snapshot hash |
| Glossary **terms** seeded with full provenance (1 language pair) | 0 | ≥ 50 by end of M0 | Termbase entry count |
| TM **units** seeded with full provenance (1 language pair) | 0 | ≥ 200 by end of M1 | TU count at tier ≥ T1 |
| **Round-trip integrity** — export→re-import preserves provenance/license/tier with no loss | n/a | 100% on the conformance suite | Round-trip CI test |
| Schema/structural **CI pass rate** on content artifacts | n/a | 100% (CI fails on any malformed unit) | `pnpm test` content validators |

**Sample rule.** Rates (match rate, reviewer-accepted reuse, consistency) are reported as raw
counts until the denominator reaches **≥ 50** offered matches / shared terms; below that we report
"N of M" rather than a percentage to avoid small-sample noise.

## Scope

**In scope**

- An **internal data model** for TM units and term entries with mandatory provenance, license, PII
  status, quality tier, language pair, and domain — plus **JSON Schemas** in `packages/schema` and
  CI validation (mirrors how `vital-info-translations` validates its content artifacts).
- **Open-standard interchange**: import/export of **TMX 1.4b** (memory) and **TBX-Basic** (terms),
  with provenance/attribution embedded in standard-compliant fields.
- A **source allow-list** with per-source license terms, attribution, snapshot hash/archive
  (reusing the `vital-info-translations` allow-list pattern; shareable across projects).
- **Ingest pipeline**: normalize → segment (sentence boundaries) → license-tag → **PII-scan
  (default-exclude on hit)** → quality-tag → dedup → store.
- **License engine**: a compatibility matrix and propagation rules; **license-partitioned exports**;
  **ND exclusion**; **NC labeling**; most-restrictive-wins only *within* a partition.
- **Matching/query**: exact + fuzzy match, filterable by language pair, license class, minimum
  quality tier, and domain; emits a per-run **match report**.
- **Quality tiers** (T0 raw/MT · T1 human-reviewed · T2 expert-signed) with a promotion process and
  the **"tier can only go up"** and **"no auto-apply to safety-critical"** rules.
- **Seeding** the commons from existing Hee-Lee Oss translation outputs (glossary first, then TM).
- File-based, **git-friendly** store; optional local SQLite index for matching speed.

**Out of scope** (explicit)

- Bulk ingestion of web-mined bitext (OPUS/ParaCrawl/CC-mined) without per-source verification.
- Any MT model training/serving; any interactive editor/CAT UI; any hosted service or public API.
- Authoring or altering translations; producing new high-stakes (medical/legal/safety) content.
- Ingesting ND-licensed, unverifiable-license, controlled-access, or individual/consented private
  data.
- Complex-script typesetting, audio/OCR pipelines, or word-level statistical alignment beyond
  minimal segmentation.
- Cross-project access control / authentication (donated lane, public artifacts only).

## Solution approach & architecture

TMC is a **software + curated-data** project on the Hee-Lee Oss donated lane (CLI prepares workspace,
human runs agent, PR opened, human/expert review gates "done"). It is deliberately **file-based and
library-shaped** so it stays reproducible, diffable in git, and free of hosting/secrets.

**The wedge (built first, end-to-end).** The **glossary/termbase commons (TBX)** ships fully
closed-loop before the segment-level TM. Rationale: terminology is the highest-leverage,
lowest-risk cross-project win (one term decided once, reused everywhere), and term entries are far
less context-sensitive than full segments — so they are safer to reuse and a cleaner proof of the
whole provenance/license/PII/quality machinery. The segment-level **TM (TMX)**, which is
context-sensitive and higher-risk, follows in M1 once the machinery is proven on terms.

**Components**

1. **Core data model & schemas** (`packages/schema`) — `tmUnitSchema`, `termEntrySchema`,
   `provenanceSchema`, `allowListSchema` (shared with `vital-info-translations`), compiled and
   exposed via `validate.ts` (AJV draft-07 + `ajv-formats`), exactly like `taskSchema`.
2. **Ingest pipeline** (`packages/*` lib + CLI) — normalize encoding (UTF-8/NFC), segment, attach
   provenance, run the **license-tagger** and **PII-scanner**, assign initial quality tier, dedup.
3. **Store** — canonical artifacts as **JSONL** (one unit/term per line, git-diff-friendly) plus a
   derived, throwaway **SQLite** index for fast fuzzy matching (rebuildable from JSONL; never the
   source of truth).
4. **License engine** — the compatibility matrix as data (`licenses/compat.yaml`), partitioning
   logic, ND exclusion, NC labeling; the **export gate** refuses to emit any unit lacking a verified
   compatible license.
5. **PII scanner** — layered detection (regex for emails/phones/IDs + name/number heuristics +
   language-aware patterns); **default-exclude on hit**, route to a human-review queue; never
   silently include.
6. **Matcher / query API** — exact + fuzzy (normalized edit distance / token-set), filters by
   langpair + license class + min tier + domain; emits a match report with provenance for each hit.
7. **Interchange** — TMX 1.4b and TBX-Basic import/export with provenance/attribution in standard
   fields (`<prop>`/`<note>`/`creationid`/`changeid` etc.); **round-trip conformance suite**.
8. **Validators / CI** — structural checks on every JSONL artifact; license-gate; PII-gate;
   round-trip integrity test — all wired into `pnpm test` so CI fails on any violation.

**Data model (per TM unit)**

```
TmUnit {
  id, srcLang, tgtLang,
  source, target,                 # the segment pair
  domain[],                       # e.g. ["public-health"], ["literature"]
  provenance {                    # mandatory, non-empty
    sourceProject, sourceDocId, sourceUrl?, sourceVersion, retrievalDate,
    translator (agent + human), reviewer?, reviewDate?, glossaryVersion?
  },
  license { name, url, derivativesAllowed, commercialAllowed, shareAlike, attribution },
  qualityTier,                    # T0 | T1 | T2
  safetyCritical,                 # bool: dosage/negation/legal/safety instruction present
  pii { status: clean|flagged|excluded, method, scannedAt },
  hashes { sourceHash, targetHash }, createdAt, updatedAt
}
```

Term entries follow the analogous shape (`term`, `translation`|`doNotTranslate`, `definition?`,
`partOfSpeech?`, plus the same `provenance`/`license`/`qualityTier`/`domain`).

**Key decisions (locked)**

- **Open standards, not bespoke.** TMX 1.4b + TBX-Basic for interchange; the public good *is* the
  open, standards-compliant, license-clean data + the reproducible toolchain.
- **Provenance/license/PII/tier travel with every unit**, always, and are query-filterable. A unit
  without complete provenance does not enter the commons.
- **License-partitioned releases.** We publish *separate* pools per license class (PD, CC-BY,
  CC-BY-SA, CC-BY-NC, …) so consumers pick what fits and **nothing is ever relicensed**;
  most-restrictive-wins applies only *within* a partition. ND and unverifiable licenses are
  **excluded**.
- **Tier can only go up, never silently down.** Reuse never lowers a segment's review requirement;
  a T2 (expert-signed) origin stays T2 in its provenance.
- **Matches are suggestions, never auto-applied to safety-critical content.** `safetyCritical`
  units require in-context human re-review before reuse (translation is context-sensitive; a correct
  segment in document A can be wrong in document B).
- **File-based + git-friendly** (JSONL source of truth; SQLite is a disposable index). No hosted DB,
  no secrets, fully reproducible.
- **Code MIT; data licensed per partition** — the project's tooling is MIT; published TM/glossary
  data carries each partition's source-compatible license + attribution, never a blanket relicense.

## Data, licensing & compliance

**This is the project's most important section. A translation memory mixes content, so licensing
and privacy errors here contaminate every downstream project. Be conservative; when terms or PII
status are unclear, the unit does not enter the commons.**

**Sources & licensing.** TMC ingests only from **allow-listed, per-source license-verified**
material, recorded exactly like the `vital-info-translations` allow-list (canonical URL,
version/date, retrieval date, license name + URL, **snapshot of license text**, derivatives-
allowed flag, commercial-allowed flag, share-alike flag, required attribution, **SHA-256
snapshotHash** + web-archive URL, `verifiedBy`/`verifiedDate`). Eligible source classes:

- **Outputs of other Hee-Lee Oss translation projects** — already carry source-compatible licenses +
  attribution + reviewer sign-off; the cleanest seed source. We **inherit** their license and
  review provenance unchanged.
- **Public-domain bitext / parallel texts** (e.g., PD literary translations, government PD where
  verified) — labeled PD.
- **Explicitly CC-licensed corpora with verified terms** (per-source, not bulk) — labeled with the
  exact CC variant.

**License compatibility & propagation rules (the engine).**

- **No relicensing, ever.** A unit keeps its origin license.
- **Partitioned releases.** Published pools are split by license class; a consumer selects the pool
  whose terms its use permits.
- **Most-restrictive-wins within a partition** when combining units of compatible-but-not-identical
  terms (e.g., CC-BY + CC-BY-SA → SA constraints apply to that combined pool).
- **ND excluded.** A translation/TM reuse is a derivative; NoDerivatives content cannot enter.
- **NC labeled, not hidden.** NonCommercial units are usable by Hee-Lee Oss (non-profit) but the published
  pool is clearly marked NC so commercial reusers are not misled (avoids the WHO-style NC trap).
- **Share-alike honored.** SA pools state the share-alike obligation explicitly in the export.
- **Attribution is mandatory** and embedded per unit in the TMX/TBX export; absence fails the gate.

**Provenance model.** Every unit ships complete provenance linking it to its source project,
document, version, retrieval date, translator (agent + human), and reviewer + tier. Provenance is
**non-optional** and is part of the license/export gate — a unit without it is rejected at ingest.

**Snapshot integrity.** Allow-list entries store the license-text snapshot hash + web-archive URL; a
**source-change watcher** (minimal/manual in M0, automated hash-diff in M1) flags upstream license
or content drift for re-verification before further export.

**Privacy / PII.** TM segments come from real documents and can contain names, addresses, phone
numbers, IDs, or case details. **PII detection is a hard ingest gate**: any unit flagged is
**excluded by default** and routed to human review; only after a reviewer confirms it carries no
personal data (or the data is removed without changing meaning, when appropriate) may it enter.
TMC collects **no end-reader or contributor personal data**; contributor identities in provenance
are limited to role/handle as agreed, never sensitive PII. For any future community-contributed
segments, **explicit consent + a license grant** are required before ingest (out of scope for v0.1
but stated so the boundary is clear).

**Inherited risk tier.** A unit's **risk tier and review requirement are inherited from its source
and never downgraded by reuse.** Segments from `high`-risk source projects (e.g., a future
`ewing-info-translations`) retain their expert-review provenance; they may be *served* by TMC but
**cannot be auto-applied** and must be re-reviewed in their new context at the originating tier.

**Output licensing.** Project **code/tooling: MIT.** Published **data: per-partition source-
compatible license** (PD / CC-BY / CC-BY-SA / CC-BY-NC-… as applicable) + required attribution +
any mandatory disclaimer carried from the source (e.g., a WHO translation disclaimer survives into
any unit derived from WHO-sourced translations). **Never** a blanket CC-BY relicense.

## Quality, review & risk gates

**Risk tier: medium.** Justified by **amplification** (a shared-memory error is reused widely) and
**inherited risk** (holds segments from high-stakes sources). Medium requires a **reviewer with the
relevant skill** — here a qualified language reviewer for the pair, plus a **license/compliance
reviewer** for license/PII gates. TMC does **not** author high-stakes advice (that would be `high`
and is out of scope), but it must **preserve** the expert review of content that did.

**Quality tiers**

- **T0 — raw / machine-drafted.** Unreviewed; **never published in a consumable pool**; staging
  only. Served only with an explicit "draft, unreviewed" flag and never to safety-critical contexts.
- **T1 — human-reviewed.** A qualified reviewer (independent of drafting) has signed off in the
  source project or in TMC. Default minimum tier for published pools.
- **T2 — expert-signed.** Reviewed by a credentialed domain expert (for safety/health/legal
  origins). Required to *retain* T2 provenance for high-stakes-origin segments.

**Required gates before a unit/export is "done"**

1. **Schema/structural validation** — unit conforms to `tmUnitSchema`/`termEntrySchema`; complete
   provenance present (CI-enforced).
2. **License gate** — source allow-listed, license verified + compatible for the target partition,
   attribution present, ND excluded, NC labeled (manual in M0, automated `license-002` in M1).
3. **PII gate** — PII scan clean (or reviewer-cleared); default-exclude on any flag (manual in M0,
   automated `pii-002` in M1).
4. **Quality-tier gate** — published units are tier ≥ T1; tier set from verified source provenance,
   never inflated.
5. **Round-trip integrity** — TMX/TBX export→re-import preserves all metadata (CI conformance test).
6. **Qualified reviewer sign-off** for any *new* curation/translation decision TMC itself makes
   (e.g., conflict resolution between two disagreeing segments), recorded in the PR.
7. **CI green + maintainer approval.**

**Conflict resolution (two segments disagree).** When dedup finds two units with the same source
but different targets: (1) prefer the higher quality tier; (2) if equal tier, prefer the one with
stronger/more recent provenance; (3) if still tied or safety-critical, **a qualified reviewer
adjudicates** and the rationale is recorded; (4) when in doubt, **keep both, mark as variants, and
auto-apply neither** — never silently pick one for safety-critical content.

**Safety-critical handling.** Units flagged `safetyCritical` (dosage, negation, legal/safety
instruction) are **never auto-applied** from a match; the consuming project's qualified reviewer
must confirm in context. The matcher marks such hits "review-required" in the match report.

**Definition of Shipped (project-specific).** TMC delivers value only when: the foundations
(schemas, license engine, PII gate, standards round-trip) are green in CI **and** a **named
consumer project has integrated TMC and run a real translation through it**, with a **measured
match rate**, **reviewer-accepted reuse**, **0 PII / 100% license compliance** on the served pool,
and **no propagated critical defect**. Built-but-unused is **not** shipped.

## Roadmap & milestones

**M0 — Foundation & cold-start (glossary wedge; no consumer required).**
Goal: lock the standards + data model, stand up the license/PII/quality machinery, and prove it
end-to-end on the **glossary/termbase** seeded from one existing Hee-Lee Oss project — ingest → store →
match → standards-compliant export with provenance/license/PII/tier intact.
Exit criteria: interchange standards + data model spec merged (`spec-001`); JSON Schemas +
validators in `packages/schema`, CI fails on malformed units (`schema-001`); allow-list with ≥ 3
license-verified sources incl. snapshot hashes (`license-001` source allow-list); license
compatibility matrix + partition/ND/NC rules documented and minimally enforced (`license-001`);
minimal PII scanner with default-exclude (`pii-001`); termbase seeded with ≥ 50 provenance-complete
terms in 1 language pair (`glossary-001`); TBX export round-trips with no metadata loss
(`export-001`, conformance test green); quality-tier model + "tier-up-only" + "no-auto-apply-safety"
rules documented (`quality-001`). `verifiedNeed: false` throughout (no consumer yet). 100%/0-PII
gates are **manual + structural in M0, automated from M1**.

**M1 — Matching, dedup & the segment-level TM.**
Goal: add fuzzy/exact matching, dedup + conflict resolution, automate the license + PII gates, and
extend the proven machinery from terms to **segment-level TM (TMX)**.
Exit criteria: matcher with langpair/license/tier/domain filters + match report (`match-001`); dedup
+ conflict-resolution rules implemented (`dedup-001`); ≥ 200 TUs at tier ≥ T1 seeded from a reviewed
Hee-Lee Oss translation set, full provenance, TMX round-trip green (`tm-001`); **automated license gate in
CI** (`license-002`); **strengthened, multi-language PII detection + human-review queue**
(`pii-002`); automated source-change watcher (`watcher-001`); reviewer/promotion process documented
(`review-001`); contributor + consumer runbook merged (`runbook-001`). License/PII "100%/0" metrics
**effective from here**. Dependency: reviewer sourcing.

**M2 — First consumer integration (needs a committed consumer).**
Goal: a named Hee-Lee Oss translation project consumes TMC in a **real** translation and we measure the
leverage. This is the first true Definition of Shipped event.
Exit criteria: a consumer project committed in writing, agreed language pair + acceptance test
(`consumer-001`, flips `verifiedNeed: true`); TMC integrated into that project's workflow, a real
document set translated with TMC suggestions, **match rate ≥ 25%**, **reviewer-accepted reuse ≥ 70%
(non-safety-critical)**, **safety-critical re-review enforced**, **0 PII / 100% license compliance**
on the served pool, **0 propagated critical defects** (`integrate-001`); outcome tracking live
(`outcomes-001`). Dependency: M0/M1 + committed consumer.

**M3 — Scale, public release & federation.**
Goal: multiple consumers/language pairs, versioned public releases, sustained quality.
Exit criteria: ≥ 2 consuming projects; versioned, **license-partitioned public TMX/TBX releases**
with changelog (`release-001`); cross-project **terminology-consistency ≥ 95%** on shared terms
(`scale-001`); maintenance cadence (source re-verification, license drift, PII re-scan, glossary
refresh) in effect (`maint-001`); sustainability owner named; optional external-contribution intake
scoped (`federation-001`, backlog).

## Work breakdown

The itemized, sized backlog lives in **[TASKS.md](./TASKS.md)**, organized by the milestones above
(M0–M3) plus a Backlog/future section. Each task maps to a Hee-Lee Oss Task JSON
(`packages/schema/src/schemas.ts`) with id, type, lane, risk tier, deliverable, acceptance
criteria, and license fields. M0/M1 tasks are consumer-independent foundations; M2+ tasks are gated
on a committed consumer and marked `verifiedNeed: false` until then.

## Governance, roles & stakeholders

- **Maintainer (Owner): J. Carter (acting)** — accepts/sequences tasks, approves PRs, owns data-
  model + standards integrity, the allow-list, and the license/PII gates. Acts as interim
  license/compliance reviewer until a dedicated one is named.
- **License/compliance reviewer** — verifies per-source terms, partition assignment, ND exclusion,
  NC labeling, attribution; signs off the license gate. May be the maintainer initially.
- **Qualified language reviewer(s) (per pair): TO BE SECURED** — sign off seeded units and conflict-
  resolution decisions; shared with / recruited via the translation-project pool and candidate NGOs
  (Translators without Borders / CLEAR Global). Rotation defined in M1.
- **Expert reviewer(s)** — for any segment whose **origin** is high-stakes (health/legal/safety),
  the originating tier (T2) and its review must be preserved; an expert is required to *retain* or
  re-establish T2 in a new context. TMC never authors such content.
- **Steward (last-mile owner): TBD — named by end of M1** (acting maintainer holds the role
  interim). Owns the **consumer relationship** and confirms real adoption/integration — without this,
  nothing reaches "shipped." Naming a steward is a **prerequisite for entering M2**.
- **Consumer / requestor: TO BE SECURED** — the first downstream Hee-Lee Oss translation project (or
  external adopter) that integrates TMC and confirms real reuse.

## Dependencies & integrations

- **Hee-Lee Oss donated lane**: `packages/cli` (workspace prep + PR), `packages/core`, `packages/schema`
  (Task JSON + the new content schemas). No funded-lane / API-key execution.
- **`vital-info-translations`** — shares the allow-list pattern and is a candidate first seed source
  and first consumer; coordination needed to avoid duplicate glossary work.
- **Other Hee-Lee Oss translation projects** (`public-domain-translations`, `health-info-translations`,
  `localization-for-good`, `irish-gaeilge-learning`, `emergency-phrasebooks`) — future
  consumers/contributors.
- **Open standards**: TMX 1.4b (OASIS/GALA), TBX-Basic / ISO 30042, optionally XLIFF 2.1 for
  handoff — specifications, not services.
- **Open-source libraries** (TS/ESM): an XML parser/emitter for TMX/TBX, a fuzzy-match/edit-distance
  lib, AJV + ajv-formats (already in `packages/schema`), optionally better-sqlite3 for the index —
  all license-checked before adoption.
- **Human reviewers / translation NGO** (Translators without Borders / CLEAR Global) — external,
  not yet secured.
- **A committed first consumer project** — external dependency, not yet secured.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| **Error amplification** — one wrong segment reused across many projects | Medium | Critical | Published pools tier ≥ T1; safety-critical never auto-applied; conflict-resolution keeps variants; propagated-defect log + fast withdrawal via provenance | Reviewers / Maintainer |
| **License contamination** — mixing incompatible licenses / relicensing / missing attribution (esp. NC like WHO-derived) | Medium | High | License-as-data + compatibility matrix; **partitioned releases**; ND excluded; NC labeled; per-export license gate; "if unclear, don't ingest" | License reviewer / Maintainer |
| **PII leakage** — personal data in segments published to a reusable commons | Medium | High | Hard PII ingest gate, **default-exclude on hit** + human-review queue; no end-reader/contributor PII collected; consent required for any community segments | Maintainer / Reviewers |
| **Inherited-risk downgrade** — high-stakes segment reused without its expert review | Medium | Critical | Tier travels + can only go up; high-stakes origins flagged `safetyCritical`; **no auto-apply**, re-review in context at originating tier | Expert reviewer / Reviewers |
| **No committed consumer** → nothing reaches "shipped" | High | High | M0/M1 build consumer-independent value; concrete consumer-sourcing plan (start with `vital-info-translations`); `verifiedNeed=false` until secured; **pause/decision point if none by end of M1** | Acting maintainer → Steward |
| **Bad fuzzy match accepted** as if exact | Medium | High | Match report shows score + provenance; thresholds; safety-critical review-required; reviewer-acceptance metric tracked | Reviewers |
| **Bulk-corpus temptation** (OPUS/ParaCrawl) bypasses verification | Medium | High | Explicit non-goal; only allow-listed, per-source verified, PII-screened ingest; reject bulk imports in review | Maintainer |
| **Standards drift / lossy round-trip** corrupts metadata | Low | Medium | Round-trip conformance suite in CI; pin standard versions (TMX 1.4b, TBX-Basic) | Maintainer |
| **Source content/license changes upstream** after ingest | Medium | Medium | Allow-list snapshot hash + web-archive; source-change watcher (M0 manual, M1 automated); re-verify before re-export | Maintainer |
| **Scope creep into MT / hosted service / authoring** | Medium | Medium | Explicit non-goals; reject such tasks; keep file-based + library-shaped | Maintainer |

## Security & privacy

- **Threat surface** is small: public source ingestion + text artifacts in a public repo + local
  tooling. Main risks are **integrity** (wrong/tampered source, bad match), **license/compliance**,
  and **PII leakage** — not data exfiltration.
- **No secrets.** Donated lane uses the human's own agent; no API keys, tokens, or escrow. Per
  CLAUDE.md, never write secrets/tokens into logs, receipts, or committed files. The SQLite index is
  a disposable local artifact, not committed.
- **PII** is the headline privacy risk (segments can carry personal data): hard ingest gate,
  default-exclude, human-review queue, no end-reader/contributor PII collected. Any community
  contribution requires explicit consent + license grant (out of scope v0.1).
- **Abuse/misuse prevention.** Refusal guardrails apply — refuse tasks that would inject
  misinformation into the commons, target/deceive people, smuggle in surveillance data, primarily
  benefit a for-profit, or violate a source license/privacy. Bulk web-scrape ingestion is refused.
- **Supply chain.** Pin standard versions and library deps; license-check every dependency;
  hash/archive every allow-listed source; source-change watcher detects drift.

## Sustainability & maintenance

- **After delivery**, the maintainer + steward keep the allow-list, license matrix, and glossaries
  current, re-verify sources on drift, and re-run the PII scan when detection improves. Reviewer
  rotation (M1/M3) avoids single-person dependence.
- **Outcome tracking** continues post-delivery: match-rate and reviewer-acceptance per consuming
  run, a **propagated-defect log** (errors traced back to a TMC segment, with fast withdrawal via
  provenance), and terminology-consistency reports. Outcomes — reuse and reader benefit, not TU
  counts — are the maintained metrics.
- **Versioned public releases** (M3) with changelogs make the commons citable and reproducible; each
  release records the schema version, standard versions, and per-partition license.
- **Decommissioning / withdrawal.** If a source's license changes to forbid reuse, or a defect or
  PII slip is found, provenance lets us identify every affected unit and downstream consumer,
  withdraw it from published pools, and notify consumers.

## Open questions

1. **First consumer.** Which Hee-Lee Oss project commits first (likely `vital-info-translations` or
   `public-domain-translations`), for which language pair, with what acceptance test? Blocks M2 and
   `verifiedNeed=true`. **Consumer-sourcing plan (concrete):** acting maintainer approaches the
   `vital-info-translations` maintainer first (shared allow-list pattern makes integration cheap),
   in parallel with M0; target a committed consumer during M2 sourcing. **Pause/decision point:** if
   no consumer commits by end of M1, the maintainer makes an explicit **go / pivot (publish-only
   open commons) / hold** decision before building more memory no one consumes.
2. **Reviewer sourcing.** Recruit individual qualified reviewers or partner with a translation NGO
   (Translators without Borders / CLEAR Global)? What are the formal qualification criteria, and how
   are they shared with the per-project reviewer pools?
3. **License partition granularity.** How many partitions to publish (PD / CC-BY / CC-BY-SA /
   CC-BY-NC / NC-SA …)? Confirm the exact compatibility matrix and the handling of mandatory
   disclaimers (e.g., WHO) that must survive into derived units.
4. **PII detection bar.** What precision/recall target for the PII scanner, across which languages,
   and what is the human-review-queue SLA? (Defaults to exclude-on-doubt for v0.1.)
5. **Fuzzy-match threshold + safety policy.** What minimum fuzzy score is offered, and is the
   "no-auto-apply to safety-critical" rule sufficient, or should *all* matches require review for
   `high`-origin content?
6. **Relationship to existing project glossaries.** Migrate `vital-info-translations` /
   `health-info-translations` glossaries into TMC as the seed, or mirror and reference? Avoid two
   sources of truth.
7. **External-corpus stance.** Do we ever ingest *vetted subsets* of open corpora (a specific,
   license-clean, PII-screened slice of OPUS), or never? (v0.1: never bulk; case-by-case only with
   full verification.)
8. **Funded lane?** Donated for v0.1. Any future case for metered, budget-capped bulk curation under
   escrow? Out of scope now.

## References

- `C:\code\hee-lee-oss\CLAUDE.md` — Hee-Lee Oss work rules, lanes, quality bar, refusal guardrails.
- `C:\code\hee-lee-oss\docs\good-deed-definition.md` — good-deed criteria and risk tiers.
- `C:\code\hee-lee-oss\packages\schema\src\schemas.ts` — Task JSON schema (content schemas added here).
- `C:\code\hee-lee-oss\planning\projects\vital-info-translations\PLAN.md` / `TASKS.md` — sibling
  translation project; allow-list, glossary, reviewer, and license-gate patterns reused here.
- `C:\code\hee-lee-oss\planning\ROADMAP.md` — portfolio; TMC and its consumer projects (Track 4).
- TMX 1.4b specification (OASIS/GALA) — translation-memory interchange.
- TBX / ISO 30042 (TBX-Basic) — termbase interchange.
- XLIFF 2.1 (OASIS) — optional downstream handoff format.
- Creative Commons license compatibility chart; CC NonCommercial / NoDerivatives / ShareAlike terms.
- Translators without Borders / CLEAR Global — candidate reviewer/adoption partner.

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified against the first draft and **applied** in
the text above (this is not a wishlist — each is reflected in the sections noted).

1. **Glossary-first wedge made explicit** (Solution approach, Roadmap M0): ship the lower-risk,
   higher-leverage termbase end-to-end before the segment-level TM — mirroring the exemplar's
   "one faculty done excellently first."
2. **Constraint-as-identity sharpened** (Executive summary): the "trust per segment / provenance
   travels with every unit / tier can only go up" rule is stated as the project's defining identity,
   not a buried policy.
3. **Inherited-risk doctrine added** (Exec summary, Data/compliance, Quality gates, Risks): a unit's
   risk tier and review requirement are inherited from its source and never downgraded by reuse —
   closing the gap where a high-stakes segment could be silently reused.
4. **"No auto-apply to safety-critical" rule** (Goals, Key decisions, Quality gates, Risks): fuzzy
   matches are suggestions; `safetyCritical` units require in-context re-review.
5. **License-partitioned releases decision** (Key decisions, Data/compliance): publish separate
   per-license pools instead of one mixed pool dragged to the most-restrictive license — avoids both
   relicensing and over-restriction.
6. **ND exclusion + NC labeling rules** (Non-goals, Data/compliance, Risks): explicit handling of
   NoDerivatives (excluded — a translation is a derivative) and NonCommercial (usable but clearly
   labeled, WHO-style trap avoided).
7. **PII as a hard, default-exclude ingest gate** (Goals, Data/compliance, Security): segments from
   real documents can carry personal data; PII detection blocks by default with a human-review
   queue.
8. **Bulk-corpus non-goal made explicit** (Non-goals, Risks): TMC does not mirror OPUS/ParaCrawl;
   only allow-listed, per-source-verified, PII-screened ingest — directly addressing the most likely
   license/PII failure mode.
9. **Quality-tier model (T0/T1/T2)** with published-pool minimum ≥ T1 (Quality gates, Success
   metrics): makes "trustworthy memory" concrete and measurable.
10. **Conflict-resolution policy** for disagreeing segments (Quality gates): prefer higher tier →
    stronger provenance → reviewer adjudication → keep-both-as-variants; never silent pick for
    safety-critical.
11. **Outcome metrics reworked to reuse-centric** (Success metrics): match rate, reviewer-accepted
    reuse, terminology consistency, propagated-defect count — not "TUs ingested."
12. **Interim foundation metrics** (Success metrics): consumer-independent progress (sources
    verified, terms seeded, round-trip integrity, CI pass) visible before any consumer exists.
13. **Small-sample reporting rule** (Success metrics): report raw "N of M" until denominator ≥ 50 —
    borrowed from the sibling project to avoid percentage noise.
14. **Round-trip conformance suite** (Architecture, Quality gates, Risks): TMX/TBX export→re-import
    must preserve all metadata, CI-enforced — guards against lossy/standards-drift corruption.
15. **Concrete data model spelled out** (Architecture): the `TmUnit` shape with mandatory
    provenance/license/PII/tier fields, so schemas and gates are unambiguous.
16. **JSONL-source-of-truth + disposable SQLite index** (Key decisions, Architecture, Security):
    keeps the commons git-diffable and reproducible; the index is never committed or authoritative.
17. **Shared allow-list with `vital-info-translations`** (Architecture, Dependencies): reuse the
    proven snapshot-hash/archive allow-list pattern instead of inventing a new one; avoid two
    sources of truth (also an Open question).
18. **Source-change watcher carried over** (Data/compliance, Roadmap, Security): M0 manual / M1
    automated hash-diff for upstream license/content drift.
19. **Concrete consumer-sourcing plan + pause/decision point** (Open questions, Risks): start with
    the `vital-info-translations` maintainer; explicit go/pivot/hold if no consumer by end of M1 —
    so the project can't drift into building unused memory.
20. **Steward role + "named by end of M1, prerequisite for M2"** (Governance): a last-mile owner of
    real adoption, without which nothing is "shipped."
21. **Definition of Shipped tied to real reuse** (Quality gates): built-but-unused is explicitly not
    shipped; requires measured match rate, reviewer acceptance, 0 PII, 100% license, 0 propagated
    defects.
22. **License/PII gates phased "manual in M0, automated from M1"** (Success metrics, Quality gates,
    Roadmap): keeps the 100%/0 hard gates honest from day one without overclaiming automation.
23. **Withdrawal/decommissioning path via provenance** (Sustainability, Risks): provenance lets us
    find and pull every affected unit + notify consumers if a license changes or a defect/PII slip
    is found.
24. **Refusal guardrails specialized to a commons** (Security): refuse misinformation injection,
    surveillance-data smuggling, for-profit-primary benefit, and bulk-scrape ingestion — concrete to
    this project, not boilerplate.
25. **Open standards framed as the public good** (Positioning, Key decisions, References): TMX 1.4b +
    TBX-Basic + the reproducible, license-clean toolchain *are* the deliverable — establishing
    interoperability and avoiding bespoke lock-in.

## Review sign-off

**Reviewer:** acting maintainer (self-review against the Hee-Lee Oss planning spec, CLAUDE.md, and the
good-deed definition). **Date:** 2026-06-28. **Verdict:** ready for community/board review as a
Draft (v0.1.0).

**Completeness.** All 17 required PLAN sections present and in order; TASKS.md provides milestone
tables, acceptance criteria, DoD per milestone, a backlog, and a schema-valid example Task JSON.
Sections cross-reference each other (gates ↔ metrics ↔ roadmap ↔ tasks).

**Correctness checks performed.**
- *Measurable metrics* — every success metric has a baseline, target, and measurement method; rates
  carry a small-sample rule. ✔
- *Enforceable gates* — schema, license, PII, quality-tier, round-trip, reviewer sign-off, CI, and
  maintainer approval are each defined with an owner and a phase (manual M0 → automated M1). ✔
- *Risks with owners + mitigations* — 10-row table; amplification, license, PII, inherited-risk, and
  no-consumer risks each have a named owner and concrete mitigation. ✔
- *License/provenance guardrail* — only allow-listed, per-source-verified, open/PD/CC sources; ND
  excluded; NC labeled; partitioned releases; no relicensing; attribution mandatory; bulk corpora
  refused. ✔
- *PII/consent guardrail* — hard default-exclude ingest gate + human-review queue; no end-reader/
  contributor PII; consent required for community segments. ✔
- *Expert-review / "not advice"* — TMC does not author high-stakes content (would be `high`, out of
  scope); inherited high-stakes segments keep their expert (T2) review and cannot be auto-applied. ✔
- *Sequencing* — M0 (glossary foundations, consumer-independent) → M1 (matching/TM + automation) →
  M2 (real consumer, first "shipped") → M3 (scale/release). Dependencies in TASKS.md are acyclic and
  consistent with the milestone gates. ✔
- *Schema validity* — the example Task JSON includes all required fields, uses only enum-valid
  values, and is donated-lane (no `fundedBudgetUsd` needed); `verifiedNeed: false` honestly recorded
  (no consumer secured). ✔

**Issues found and fixed during review.**
- Initial draft set published-pool license as a single most-restrictive license → **fixed** to
  license-partitioned releases (improvement #5) to avoid both relicensing and over-restriction.
- Initial draft treated PII as a backlog item → **promoted** to a hard M0 ingest gate (#7) and an
  automated M1 gate (#22), since a *shared* commons makes PII leakage far higher-impact than in a
  one-off translation.
- Initial draft allowed fuzzy matches to apply uniformly → **added** the safety-critical no-auto-
  apply rule and inherited-risk doctrine (#3, #4) after noting TMC will hold health/safety segments.
- Initial metrics counted ingested TUs → **replaced** with reuse-/outcome-centric metrics (#11).

**Open decisions requiring a human (board/maintainer).** First consumer commitment + language pair
(blocks M2 / `verifiedNeed=true`); reviewer sourcing model; exact license-partition set and
disclaimer-survival rules; PII detector precision/recall bar and queue SLA; whether vetted subsets
of external corpora are ever admissible. These are tracked in Open questions and are not resolvable
without partner/board input.
