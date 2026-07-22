# TASKS — translation-memory-commons

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: J. Carter (acting maintainer) · Lane: donated

The backlog for the `translation-memory-commons` (TMC) good-deed project. Read alongside
[PLAN.md](./PLAN.md). Milestones (M0–M3) match the roadmap there.

## How these tasks map to Hee-Lee Oss

Each task below becomes an **Hee-Lee Oss Task JSON** validated against
`packages/schema/src/schemas.ts` (AJV / JSON Schema draft-07). Field mapping:

- **id** — stable slug id, e.g. `translation-memory-commons-spec-001` (table column `ID`).
- **title** — the task title.
- **project** — always `translation-memory-commons`.
- **type** — one of `code | research | writing | data | design-spec | maintenance` (table `Type`).
- **lane** — `donated` for all current tasks (no funded/API execution). A funded task would require
  `fundedBudgetUsd`; none here.
- **priority** — `high | medium | low`.
- **domain** — array, e.g. `["translation","tooling","licensing","privacy"]`.
- **riskTier** — `low | medium | high`. **Medium** for tasks touching translation accuracy,
  license/PII propagation, or seeded language content (amplification + inherited risk); **low** for
  pure tooling/standards/process tasks. No `high` tasks (TMC never authors high-stakes content; it
  preserves the expert review of content that does).
- **urgent** — boolean (default false; no live emergency).
- **deliverable** — `pr | dataset | document | translation` (table `Deliverable`). Code/tooling →
  `pr`; seeded memory/glossary/logs → `dataset`; specs/runbooks/procedures → `document`.
- **tokenEstimate** — `small | medium | large` (table `Size`).
- **status** — `open | in-progress | review | delivered | done` (all start `open`).
- **context / objective** — why + what.
- **acceptanceCriteria[]** — checkable bullets (listed below tables for key tasks).
- **resources[]** — links/paths (standards, allow-list, sibling project, schemas).
- **output** — path/description of the produced artifact.
- **requestor** — first consumer/requestor; `TO BE SECURED` until committed.
- **verifiedNeed** — boolean; **`false`** wherever value depends on an unsecured consumer.
- **outputLicense** — **MIT** for code/tooling and for project-metadata datasets (schemas,
  allow-list, license matrix); **per-partition source-compatible license + attribution** for
  *published TM/glossary content* (PD / CC-BY / CC-BY-SA / CC-BY-NC + any carried disclaimer),
  **never** a blanket relicense.

---

## Milestone M0 — Foundation & cold-start (glossary wedge; no consumer required)

Goal: lock standards + data model, stand up the license/PII/quality machinery, and prove it
end-to-end on the **glossary/termbase** — ingest → store → match → standards-compliant export with
provenance/license/PII/tier intact. All tasks consumer-independent; `verifiedNeed: false`.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| translation-memory-commons-spec-001 | Adopt interchange standards (TMX 1.4b, TBX-Basic) + define TM-unit/term data model (provenance, license, PII, tier) | design-spec | medium | low | document | — | Maintainer |
| translation-memory-commons-schema-001 | Implement content JSON Schemas + AJV validators + CI structural checks | code | medium | low | pr | spec-001 | Maintainer |
| translation-memory-commons-license-001 | Source allow-list + license compatibility matrix + partition/ND/NC rules | research | medium | medium | dataset | spec-001 | License reviewer / Maintainer |
| translation-memory-commons-pii-001 | Minimal PII scanner (default-exclude) + ingest gate | code | small | medium | pr | spec-001 | Maintainer |
| translation-memory-commons-ingest-001 | Ingest/normalize/segment/dedup skeleton (JSONL store) | code | medium | low | pr | schema-001, license-001, pii-001 | Maintainer |
| translation-memory-commons-glossary-001 | Seed termbase (≥50 terms, 1 language pair) with full provenance | data | medium | medium | dataset | ingest-001, license-001 | Qualified reviewer |
| translation-memory-commons-export-001 | Standards-compliant TBX export + round-trip conformance test | code | small | low | pr | schema-001, glossary-001 | Maintainer |
| translation-memory-commons-quality-001 | Quality-tier model + "tier-up-only" + "no-auto-apply-safety" rules | design-spec | small | medium | document | spec-001 | Maintainer / Qualified reviewer |

**Acceptance criteria — key M0 tasks**

`spec-001` (standards + data model) — *first M0 task; see Example task JSON*
- Selects and documents **TMX 1.4b** (memory) and **TBX-Basic / ISO 30042** (terms) as the
  interchange standards, with rationale and the specific fields used to carry provenance/license.
- Defines the `TmUnit` and `TermEntry` data models with **mandatory** `provenance`, `license`,
  `pii`, `qualityTier`, `srcLang`/`tgtLang`, `domain`, `safetyCritical`, and content hashes.
- States the locked decisions: JSONL source-of-truth + disposable SQLite index; per-partition
  licensing; ND excluded / NC labeled; tier can only go up; no auto-apply to safety-critical.
- A unit lacking complete provenance is, by spec, invalid and cannot enter the commons.

`schema-001` (schemas + CI)
- Exports `tmUnitSchema`, `termEntrySchema`, `provenanceSchema`, `allowListSchema` from
  `packages/schema/src/schemas.ts`; compiled and exposed via `validate.ts` (AJV draft-07 +
  ajv-formats), exactly like `taskSchema`.
- A structural-check script validates all JSONL artifacts and **fails CI** (`pnpm test`) on any
  malformed/non-conformant unit (missing provenance, invalid enum, etc.).
- Unit tests cover valid + invalid fixtures for each schema.

`license-001` (allow-list + matrix)
- `sources/allow-list.yaml` lists ≥ 3 license-verified sources (incl. ≥ 1 PD and ≥ 1 CC variant),
  each with name, URL, version/date, retrieval date, license name + URL, **snapshot of license
  text**, `snapshotHash` (SHA-256), web-archive URL where available, derivatives/commercial/share-
  alike flags, required attribution, `verifiedBy`/`verifiedDate`.
- `licenses/compat.yaml` encodes the compatibility matrix + partition rules: **ND excluded**, **NC
  labeled**, share-alike honored, most-restrictive-wins *within* a partition, **no relicensing**.
- Any source with unclear/incompatible terms is marked **excluded** with a reason.

`pii-001` (PII gate)
- A scanner detects emails, phone numbers, government IDs, and name/number heuristics over candidate
  units; **any hit → unit excluded by default** and routed to a flagged-for-review queue file.
- The ingest pipeline refuses to store a unit until its `pii.status` is `clean` or reviewer-cleared.
- Tests cover positive (PII present → excluded) and negative cases.

`glossary-001` (seed termbase)
- ≥ 50 source→target term entries in one language pair, each with full provenance, license
  (inherited from an allow-listed source / existing Hee-Lee Oss project glossary), domain, and quality
  tier ≥ T1; reviewed by a qualified reviewer for the pair.
- `safetyCritical` flagged where applicable (e.g., do-not-translate drug names, negation-bearing
  instruction terms).
- Validates against `termEntrySchema`; exports to TBX and round-trips with no metadata loss.

**M0 Definition of Done:** standards + data-model spec merged; content schemas + validators fail CI
on malformed units; allow-list with ≥ 3 verified sources (snapshot hashes) + license compatibility
matrix merged; minimal PII scanner with default-exclude wired into ingest; termbase seeded (≥ 50
provenance-complete terms, tier ≥ T1, 1 pair); TBX export round-trips with no metadata loss
(conformance test green); quality-tier + tier-up-only + no-auto-apply-safety rules documented.
License/PII "100%/0" gates are **manual + structural in M0, automated from M1**. All M0 deliverables
carry `verifiedNeed: false` (no consumer yet).

---

## Milestone M1 — Matching, dedup & the segment-level TM

Goal: add fuzzy/exact matching + dedup/conflict resolution, automate the license + PII gates, and
extend the proven machinery from terms to **segment-level TM (TMX)**.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| translation-memory-commons-match-001 | Exact + fuzzy matcher + query API + match report (langpair/license/tier/domain filters) | code | medium | low | pr | ingest-001, export-001 | Maintainer |
| translation-memory-commons-dedup-001 | Dedup + conflict-resolution rules (variants, tier/provenance ordering) | code | medium | medium | pr | match-001 | Maintainer / Qualified reviewer |
| translation-memory-commons-tm-001 | Seed segment-level TM (≥200 TUs, tier ≥ T1) from a reviewed Hee-Lee Oss translation set | data | large | medium | dataset | ingest-001, license-001 | Qualified reviewer |
| translation-memory-commons-license-002 | Automate license gate in CI (allow-list + per-unit + partition enforcement) | code | medium | medium | pr | license-001, schema-001 | License reviewer / Maintainer |
| translation-memory-commons-pii-002 | Strengthen PII detection (multi-language) + human-review queue workflow | code | medium | medium | pr | pii-001 | Maintainer |
| translation-memory-commons-watcher-001 | Automated source-change watcher (hash-diff vs. stored snapshots) | code | small | low | pr | license-001 | Maintainer |
| translation-memory-commons-review-001 | Reviewer/promotion process (T0→T1→T2) + qualification criteria | writing | small | low | document | quality-001 | Maintainer |
| translation-memory-commons-runbook-001 | Contributor + consumer runbook (how to add to / consume the commons) | writing | small | low | document | match-001, export-001 | Maintainer |

**Acceptance criteria — key M1 tasks**

`match-001` (matcher)
- Returns exact and fuzzy matches (normalized edit distance / token-set), each with **score +
  full provenance + license + quality tier**.
- Query filters: language pair, **license class**, **minimum quality tier**, domain.
- Emits a per-run **match report** (matched segments, scores, sources) consumable by a downstream
  project; `safetyCritical` hits are marked **review-required** and never flagged auto-apply.

`dedup-001` (dedup + conflict resolution)
- Detects duplicate and conflicting units (same source, different target).
- Resolution order implemented: higher tier → stronger/more recent provenance → reviewer
  adjudication → **keep both as variants**; never silently auto-pick for `safetyCritical`.
- Conflict decisions and rationale recorded; tests cover each branch.

`license-002` (automated license gate)
- CI validates every published unit against its allow-list source entry and the compatibility
  matrix; **fails** if attribution/disclaimer missing, license incompatible with its partition, ND
  present, or NC unlabeled. This makes the **100% license-compliance** metric automated from M1.

`pii-002` (strengthened PII)
- Detection extended across the seeded language(s); precision/recall measured against a labeled test
  set; flagged units route to a documented **human-review queue** with a resolution workflow;
  default remains exclude-on-doubt. Makes the **0-PII** metric automated from M1.

**M1 Definition of Done:** matcher + match report with filters; dedup + conflict resolution
implemented; ≥ 200 TUs (tier ≥ T1, full provenance) seeded and TMX round-trip green; **license gate
automated in CI** (`license-002`); **PII detection strengthened + review queue** (`pii-002`);
**source-change watcher automated** (`watcher-001`); reviewer/promotion process + runbook merged.
License/PII "100%/0" metrics effective from here. **Steward named** (prerequisite for M2).

---

## Milestone M2 — First consumer integration (needs a committed consumer)

Goal: a named Hee-Lee Oss translation project consumes TMC in a **real** translation and we measure the
leverage. First true Definition of Shipped. **All tasks gated on a committed consumer**
(`verifiedNeed` flips to `true` only when confirmed).

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| translation-memory-commons-consumer-001 | Secure first consumer project; agree language pair + acceptance test | research | medium | low | document | runbook-001 | Steward / Maintainer |
| translation-memory-commons-integrate-001 | Integrate TMC into a real translation run; measure match rate + reviewer acceptance | code | large | medium | pr | consumer-001, match-001, tm-001 | Qualified reviewer / Steward |
| translation-memory-commons-outcomes-001 | Outcome tracking: match-rate, reviewer-acceptance, propagated-defect, consistency logs | data | small | low | dataset | integrate-001 | Steward |

**Acceptance criteria — key M2 tasks**

`consumer-001`
- A named Hee-Lee Oss translation project (likely `vital-info-translations` or
  `public-domain-translations`) **commits in writing** to integrate TMC, with an agreed language
  pair and a concrete **acceptance test** (target match rate + reviewer-acceptance threshold).
- **Pause/decision point:** if no consumer commits by end of M1, the maintainer makes an explicit
  **go / pivot (publish-only commons) / hold** decision before further memory-building.
- On completion, related tasks update `requestor` and set `verifiedNeed: true`.

`integrate-001`
- TMC is wired into the consumer project's workflow; a real document set is translated using TMC
  suggestions; the run produces a match report.
- Measured: **match rate ≥ 25%**, **reviewer-accepted reuse ≥ 70%** of *non-safety-critical*
  offers; **safety-critical hits re-reviewed in context** (none auto-applied).
- Served pool passes **0 PII / 100% license compliance**; **0 propagated critical defects** found
  in the reviewed output. Consumer confirms real reuse in the PR/receipt (Definition of Shipped).

**M2 Definition of Done:** consumer secured (`verifiedNeed=true`); TMC integrated into a real
translation; match-rate, reviewer-acceptance, license, and PII targets met; 0 propagated critical
defects; outcome tracking live. This is the first **shipped** event.

---

## Milestone M3 — Scale, public release & federation

Goal: multiple consumers/language pairs, versioned public releases, sustained quality.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| translation-memory-commons-release-001 | Versioned, license-partitioned public TMX/TBX releases + changelog | maintenance | medium | medium | dataset | integrate-001, license-002 | License reviewer / Maintainer |
| translation-memory-commons-scale-001 | Onboard a 2nd consumer / language pair; measure terminology consistency | data | large | medium | dataset | integrate-001 | Qualified reviewers |
| translation-memory-commons-maint-001 | Maintenance cadence: source re-verification, license drift, PII re-scan, glossary refresh | maintenance | small | low | document | watcher-001, outcomes-001 | Maintainer |

**Acceptance criteria — key M3 tasks**

`release-001`
- Public releases are **partitioned by license class** (PD / CC-BY / CC-BY-SA / CC-BY-NC + any
  carried disclaimer), each with a changelog, schema version, and standard versions recorded;
  license gate green on every release; nothing relicensed.

`scale-001`
- A second consuming project / language pair is onboarded; **terminology consistency ≥ 95%** of
  shared glossary terms rendered identically across consumers (cross-project diff report).

**M3 Definition of Done:** ≥ 2 consuming projects; versioned license-partitioned public releases
with changelog; terminology-consistency ≥ 95% on shared terms; maintenance cadence in effect; named
sustainability owner.

---

## Backlog / future

Sized but unscheduled:

- **(medium) XLIFF 2.1 handoff support** — emit/consume XLIFF for downstream CAT-tool interchange,
  if a consumer requires it.
- **(large) External-contribution intake (federation)** — accept community-contributed segments with
  explicit consent + license grant + PII + reviewer gates; governance for outside contributors.
- **(medium) Vetted external-corpus subset ingestion** — case-by-case admission of a specific,
  license-clean, PII-screened slice of an open corpus (never bulk; full verification each time).
- **(small) Domain taxonomy** — standardized `domain` tags shared across consumers for better
  filtering.
- **(medium) Consistency dashboard** — automated cross-project terminology-consistency reporting.
- **(large, funded — needs escrow) Budget-capped bulk curation** — metered curation under a hard
  per-task budget; would require `fundedBudgetUsd`; out of scope for v0.1.

---

## Example task JSON

Schema-valid Task JSON for the first M0 task (`spec-001`). `verifiedNeed` is **false** (no consumer
secured); `outputLicense` is **MIT** because the deliverable is a design/spec document (project
metadata), not published translation content. Donated lane → no `fundedBudgetUsd` required.

```json
{
  "id": "translation-memory-commons-spec-001",
  "title": "Adopt interchange standards (TMX 1.4b, TBX-Basic) and define the TM-unit/term data model",
  "project": "translation-memory-commons",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["translation", "tooling", "standards", "licensing", "privacy"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "medium",
  "status": "open",
  "context": "translation-memory-commons is shared open translation-memory + glossary infrastructure reused across Hee-Lee Oss translation projects. A translation memory aggregates segments from many sources, licenses, and quality levels, so an error, a mislicensed segment, or PII propagates into every downstream project that reuses it. Before any segment is stored, the project must lock its interchange standards and a data model in which provenance, license, PII status, and quality tier travel with every unit and can be filtered at query time.",
  "objective": "Select and document the interchange standards (TMX 1.4b for memory, TBX-Basic / ISO 30042 for terminology) and define the TmUnit and TermEntry data models with mandatory provenance, license, PII, quality-tier, language-pair, domain, safety-critical, and hash fields, plus the locked decisions (JSONL source-of-truth + disposable SQLite index; per-partition licensing; ND excluded / NC labeled; tier can only go up; no auto-apply to safety-critical).",
  "acceptanceCriteria": [
    "Spec selects TMX 1.4b and TBX-Basic (ISO 30042) and documents which standard fields carry provenance, license, and attribution",
    "Defines TmUnit and TermEntry models with mandatory provenance, license, pii, qualityTier, srcLang, tgtLang, domain, safetyCritical, and content-hash fields",
    "States locked decisions: JSONL source-of-truth + disposable SQLite index, per-partition licensing, ND excluded, NC labeled, tier-up-only, no auto-apply to safety-critical content",
    "Specifies that a unit lacking complete provenance is invalid and cannot enter the commons",
    "Spec is reviewed and merged; it is the basis for schema-001 (JSON Schemas) and license-001 (allow-list + compatibility matrix)"
  ],
  "resources": [
    "C:/code/hee-lee-oss/planning/projects/translation-memory-commons/PLAN.md",
    "C:/code/hee-lee-oss/packages/schema/src/schemas.ts",
    "C:/code/hee-lee-oss/planning/projects/vital-info-translations/PLAN.md",
    "TMX 1.4b specification (OASIS/GALA)",
    "TBX-Basic / ISO 30042 specification"
  ],
  "output": "docs/spec/data-model-and-standards.md plus a fields reference mapping TmUnit/TermEntry to TMX/TBX",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "MIT"
}
```
