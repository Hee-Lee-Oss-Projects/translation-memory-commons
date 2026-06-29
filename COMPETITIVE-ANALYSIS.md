# Competitive + Improvement Analysis — `translation-memory-commons` (TMC)

> Analyst review of PLAN.md v0.1.0 (2026-06-28) for the Elyos donated-lane good-deed project: an
> open, license-clean, provenance-tracked TM (TMX 1.4b) + glossary/termbase (TBX-Basic) commons for
> under-served languages, seeded from openly-licensed parallel material, serving translators and MT
> training. Web-researched and cited throughout. Date: 2026-06-29.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually mature for a v0.1 draft: 17 sections, enforceable gates with owners, a
glossary-first wedge, an inherited-risk doctrine, license-partitioned releases, and a hard PII
ingest gate. The core architecture (JSONL source-of-truth + disposable SQLite index, per-segment
provenance/license/PII/tier travelling together, "tier can only go up," "no auto-apply to
safety-critical") is correct and well-matched to the real failure modes of a shared memory. The
findings below are gaps, not refutations.

**1.1 Source-corpus licensing rigor — directionally correct, but the threat model understates the
problem.** The plan's instinct to *refuse bulk OPUS/ParaCrawl ingestion* and admit only
allow-listed, per-source-verified material is exactly right and is the single most defensible
decision in the document. The grounding confirms why:

- **OPUS is a heterogeneous aggregator with no central license guarantee.** Its front page lists
  1,214 corpora / ~102.9B sentence pairs across 1,005 languages, dominated by OpenSubtitles (27.2B),
  NLLB (22.7B), CCMatrix (17.1B), and ParaCrawl (4.6B) — but publishes **no per-corpus license or
  copyright disclaimer on the landing page**; licensing is delegated to each sub-corpus and the
  citation paper. OpenSubtitles in particular is film-subtitle text of dubious redistributable
  copyright status. So "OPUS-licensed" is not a thing; each sub-corpus must be verified individually.
- **"CC0" on web-crawled corpora is a *release label*, not a copyright clearance.** ParaCrawl is
  released CC0 ("no rights reserved") yet is extracted from a fresh crawl of websites selected via
  CommonCrawl — the CC0 covers ParaCrawl's *compilation*, not the upstream copyright of the crawled
  pages. TMC's policy ("verify per-source terms; if unclear, don't ingest") is correct, but the plan
  should state explicitly that **a downstream CC0/permissive label on a derived corpus does not
  launder upstream copyright**, and that allow-listing requires verifying the *original* source's
  license, not the redistributor's.
- **Gap:** the allow-list schema captures `derivativesAllowed/commercialAllowed/shareAlike/
  attribution`, but does not capture **upstream-vs-redistributor license provenance** (who is
  asserting the license, and on what basis). Add a `licenseAsserter` / `licenseBasis` field.

**1.2 Per-segment provenance + license — strong, with one structural risk.** Mandatory non-empty
provenance as an ingest gate is the right spine. Two gaps: (a) the model stores a single `license`
object per unit, but a TM unit is a *pair* drawn from a source document whose **source-side and
target-side may carry different rights** (e.g., a public-domain source text with a CC-BY-SA
translation). The schema should allow asymmetric source/target licensing or at minimum record that
the unit license is the *more restrictive join* with a note. (b) There is no **provenance integrity
/ tamper-evidence** mechanism beyond `sourceHash/targetHash`; for a withdrawal-by-provenance
guarantee (Sustainability section) to hold, provenance records themselves need to be immutable/append
-only or hash-chained.

**1.3 Alignment quality — under-specified relative to its risk.** The plan treats alignment as
"minimal sentence segmentation" and lists word-level statistical alignment as a non-goal. That is
reasonable for *human-reviewed* Elyos outputs (already aligned by the translator). But the moment
TMC seeds from any *external* parallel text (PD literary translations, government bitext), **segment
alignment becomes the dominant quality risk** — literary and legal translations are notoriously
non-sentence-aligned (1:many, reordered, omitted). The independent audit by Kreutzer/Caswell et al.
("Quality at a Glance," TACL 2022) manually audited 205 language corpora across CCAligned/ParaCrawl/
WikiMatrix/OSCAR/mC4 and found **at least 15 corpora with no usable text and many low-resource
corpora with <50% acceptable-quality sentences**, plus systematic mislabeling. The lesson for TMC:
**for any non-Elyos parallel source, alignment correctness must be an explicit, reviewed gate** (not
folded into "segmentation"), and low-resource pairs need a higher human-review sampling rate.

**1.4 PII / content leakage in TM segments — correctly prioritized; detection bar is the open
risk.** Making PII a hard default-exclude ingest gate is correct and ahead of the field (ParaCrawl
runs Biroamer/Bicleaner anonymization; MyMemory offers one-click proper-name removal — both are
*opt-in scrubbing*, not hard gates). The unresolved issue is **detection efficacy for under-served
languages**: regex for emails/phones/IDs is language-agnostic, but name/number heuristics and
ID-format patterns are deeply locale-specific, and most NER/PII tooling is weak exactly where TMC
operates (low-resource scripts). The plan's Open Question #4 acknowledges this but sets no floor.
**Recommend a per-language precision/recall floor with mandatory human spot-audit when the language
has no validated PII detector** — otherwise "0 PII" is an unfalsifiable claim for the languages that
matter most. Also add **memorization/leakage of source-document context** (a TM segment can be a
verbatim sentence from a copyrighted or sensitive document even when it contains no classic PII) —
the safety-critical/sensitive-document flag should cover this, not just named-entity PII.

**1.5 Format standards (TMX/TBX/XLIFF) — correct choices; watch the round-trip fidelity claim.** TMX
1.4b and TBX-Basic/ISO 30042 are the right, durable standards, and XLIFF 2.1 as optional handoff is
sensible. The risk: **the plan's "100% round-trip with no metadata loss" promise is hard to keep**
because TMX/TBX have no native fields for `qualityTier`, `pii.status`, `safetyCritical`, or rich
provenance — these must ride in `<prop>`/`<note>` extensions, which other CAT tools may silently
drop. The conformance suite should test round-trip **through at least one third-party tool**
(e.g., Okapi/OmegaT), not just TMC's own importer, or the guarantee only holds inside TMC.

**1.6 Dedup — policy is good; the hard case is missing.** The conflict-resolution ladder (higher
tier → stronger provenance → reviewer → keep-both-as-variants) is sound. Two missing cases:
(a) **near-duplicate / fuzzy-duplicate source segments** (not exact-equal) that should collapse but
won't under exact-hash dedup; (b) **cross-license duplicates** — the same segment arriving under two
different licenses (e.g., PD and CC-BY-SA). Under partitioned releases the unit must appear in *both*
partitions with its respective license, not be deduped to one — the plan should state that dedup is
**per-partition**, never across license classes.

**1.7 Low-resource-language scarcity — the elephant the metrics don't size.** The plan's target of
"≥25% match rate on the first integrated document set" is plausible for a *high-resource* pair but
likely **unreachable for genuinely under-served languages**, where there is little prior translated
text to match against — that is the entire reason the commons is needed. The success metrics should
**stratify targets by language-resource level** (a 25% match-rate target is reasonable for, say,
Spanish but punishing for a revitalization language), or the project risks looking like a failure
precisely on its mission languages. Relatedly, the glossary-first wedge is the right hedge here:
terminology consistency delivers value even at near-zero TM match rates.

**Completeness verdict:** all required sections present and internally consistent; gates ↔ metrics ↔
roadmap cross-reference cleanly. Highest-value corrections: (1.4) per-language PII floor + human
audit for undetectable languages, and (1.1/1.2) upstream-license provenance + asymmetric
source/target licensing.

---

## 2. Competitive landscape

TMC sits in a crowded field of parallel-text and TM resources. The crucial framing: **none of the
dominant players optimizes for per-segment license cleanliness + provenance + PII safety for
under-served languages.** They optimize for scale (OPUS, ParaCrawl, NLLB), commerce (TAUS, MyMemory),
or a single institution's output (DGT, Wikimedia).

| Competitor | What it is | Strengths | Weaknesses (vs TMC's thesis) |
|---|---|---|---|
| **OPUS** (Tiedemann, Helsinki-NLP) | The dominant open aggregator; 1,214 corpora, ~102.9B pairs, 1,005 languages | Unmatched breadth/coverage; standard in MT research; free | **Heterogeneous, per-corpus licenses with no central guarantee or landing-page disclaimer**; includes copyright-fraught sub-corpora (OpenSubtitles); not PII-screened or provenance-rich at segment level |
| **ParaCrawl** | EU-funded web-crawled bitext, 23+ EU langs | Large, CC0-released, Bicleaner-filtered, Biroamer anonymization | **CC0 label ≠ upstream copyright clearance**; web-crawl quality/PII issues; EU-language-centric (not under-served); anonymization is best-effort, not a hard gate |
| **DGT-TM** (EU JRC) | EU Acquis legislative TM, 24 langs, TMX | Clean institutional provenance; explicit reuse terms (commercial + non-commercial, attribution required); genuine TMX | **Single narrow domain** (EU law); EU languages only; attribution/ownership conditions attach; no under-served-language coverage |
| **TAUS Data Marketplace** | Commercial buy/sell language-data market; 35B+ words, 600+ pairs | Quality scoring, cleaning, anonymization services; domain-tuned | **Commercial/paywalled**; opaque per-segment provenance; serves enterprise MT, not public-good under-served languages; not an open commons |
| **MyMemory** (Translated) | "World's largest TM," ~7.5B human-translated segments, free API | Huge, free, no-key API; aggregates EU/UN + web + MT | **Mixed human/MT quality**; proper-name removal is opt-in, not a guaranteed gate; aggregated provenance is coarse; not partitioned by license |
| **Tatoeba** | Community sentence/translation collection | Open (default CC-BY 2.0 FR; CC0 option); per-sentence attribution; community-curated | **Sentences are illustrative, not document-derived TM**; CC0 *translations* disallowed (derivative-license constraint); coverage skewed to popular languages |
| **Wikimedia Content Translation** | CAT tool producing published Wikipedia translations as parallel corpora | CC-BY-SA, large, growing, real provenance via article history | **Wikipedia-domain only**; CC-BY-SA (ShareAlike) constrains reuse; quality varies; not termbase-structured |
| **FLORES-200 / NLLB datasets** (Meta) | 200-language MT eval benchmark + mined training data | Professional translations; 200 langs incl. low-resource; CC-BY-SA-4.0; now community-run as FLORES+ | **An eval benchmark, not a reusable TM/glossary**; NLLB mined data has the web-crawl quality/PII issues the audit flagged; ShareAlike |
| **Mozilla (Pontoon / Common Voice TMX)** | OSS localization platform; exposes per-locale TMX | Real OSS-localization TM; open; per-project | **UI-string domain**; software-localization register, not health/literary/vital-info; coverage follows Mozilla's locales |

**Key cited sources.** OPUS scale/heterogeneity and absence of a central license disclaimer:
[opus.nlpl.eu](https://opus.nlpl.eu/) and [Helsinki-NLP/OPUS](https://github.com/Helsinki-NLP/OPUS).
ParaCrawl CC0 + crawl provenance + Bicleaner/Biroamer:
[ParaCrawl v8](https://paracrawl.eu/v8) and [release notes](https://mailman.uib.no/public/corpora-archive/2019-September/030639.html).
Web-crawl quality/PII audit: Kreutzer, Caswell et al., "Quality at a Glance"
([arXiv:2103.12028](https://arxiv.org/abs/2103.12028), [TACL 2022](https://aclanthology.org/2022.tacl-1.4/)).
DGT-TM reuse terms (TMX, 24 langs, commercial + non-commercial, attribution + EU ownership):
[EC JRC](https://joint-research-centre.ec.europa.eu/language-technology-resources/dgt-translation-memory_en).
TAUS Data Marketplace (commercial, quality tiers, anonymization):
[taus.net](https://www.taus.net/resources/blog/taus-launches-the-data-marketplace) /
[Datarade](https://datarade.ai/data-providers/taus/profile).
MyMemory (~7.5B segments, EU/UN aggregation, one-click name removal):
[mymemory.translated.net](https://mymemory.translated.net/) /
[API spec](https://mymemory.translated.net/doc/spec.php).
Tatoeba licensing (CC-BY 2.0 FR default, CC0 option, translations-are-derivatives constraint):
[Tatoeba downloads](https://tatoeba.org/en/downloads) /
[CC0 contributions](https://en.wiki.tatoeba.org/articles/show/cc0-contributions).
Wikimedia Content Translation parallel-corpus output + licensing:
[mediawiki.org](https://www.mediawiki.org/wiki/Content_translation/Translation_tools).
FLORES-200 / FLORES+ (CC-BY-SA-4.0, 200 langs, eval benchmark):
[NLLB paper](https://arxiv.org/pdf/2207.04672).
Health-domain low-resource gap (context for vital-info ties): OpenWHO corpus
([arXiv:2508.16048](https://arxiv.org/pdf/2508.16048)).

---

## 3. Gaps we can fill

The market is bifurcated: **scale-but-dirty** (OPUS/ParaCrawl/NLLB — huge, heterogeneous license,
unscreened PII, mixed alignment) vs **clean-but-narrow-or-paywalled** (DGT = EU law only; Wikimedia =
Wikipedia only; TAUS = commercial). TMC's defensible gap is the **missing quadrant: clean *and* broad
*and* open *and* under-served-language-focused, with per-segment trust metadata.**

1. **Per-segment provenance + license + PII + quality-tier as first-class, queryable metadata.** No
   open competitor ships this travelling with every unit. OPUS gives corpus-level licenses; MyMemory
   gives coarse aggregate provenance; DGT gives institutional provenance but one license. TMC's
   filter-at-query-time (by license class, min tier, domain) is genuinely novel for an open commons.
2. **License-partitioned releases with no relicensing.** Everyone else either flattens to one label
   (ParaCrawl→CC0, DGT→reuse-terms, Wikimedia→CC-BY-SA) or leaves it to the user. Publishing
   *separate per-license pools* lets a downstream consumer pick exactly what their use permits — a
   real, underserved need (the "WHO NC-trap" the plan calls out is widespread).
3. **PII as a hard gate, not opt-in scrubbing.** ParaCrawl/MyMemory anonymize optionally; TMC
   excludes-by-default. For health/vital-info/refugee contexts this is the difference between
   publishable and not.
4. **Termbase/glossary commons with provenance** — the field is almost entirely *bitext* (sentences),
   not *terminology*. A reviewed, provenance-tracked, cross-project TBX termbase for under-served
   languages barely exists in the open. This is TMC's lowest-risk, highest-differentiation surface.
5. **Under-served-language focus with curated quality** — FLORES/NLLB chase 200 languages at
   benchmark/mined quality; TMC can offer *small but trustworthy* memory exactly where the big
   aggregators are weakest (the audit shows low-resource corpora are where quality collapses).
6. **Withdrawal-by-provenance** — because every unit traces to its source, TMC can recall a segment
   (and notify consumers) if a license changes or a defect/PII slip is found. No open corpus offers a
   recall path; they are fire-and-forget dumps.

---

## 4. Differentiators to win (vs OPUS, the dominant aggregator)

OPUS wins on **breadth**; TMC cannot and should not try to. TMC wins on **trust per segment** — the
axis OPUS structurally cannot serve because it is an aggregator of others' corpora.

1. **"Trust, not tonnage."** OPUS's value proposition is coverage; TMC's is *a segment you can ship
   to a beneficiary without re-verifying*. Every differentiator flows from this.
2. **Verified, per-segment license cleanliness** vs OPUS's "per-corpus, verify-it-yourself, no
   central guarantee." TMC's allow-list with snapshot-hash + web-archive + `verifiedBy` is auditable
   provenance OPUS does not provide.
3. **Hard PII exclusion** vs OPUS's unscreened web-mined sub-corpora (the audit documents real
   leakage and junk in exactly these).
4. **Reviewed quality tiers (T1/T2) + safety-critical no-auto-apply** vs OPUS's unlabeled mixed
   quality. A medical term from TMC carries its expert-review provenance; the same string from OPUS
   carries nothing.
5. **Glossary/termbase commons** — a product category OPUS simply does not have. This is the cleanest
   "we have something they don't" wedge.
6. **Standards-native, recallable, reproducible** — TMX/TBX with embedded attribution + a
   withdrawal path, versioned releases with changelogs. TMC is *infrastructure*, not a data dump.
7. **Composability inside Elyos** — TMC compounds in value as sibling translation projects feed and
   consume it; OPUS has no such network effect with the beneficiary projects.

The honest caveat the plan already records: **none of this matters until a first consumer commits**
(M2). The differentiators are real but unrealized until adoption — securing `vital-info-translations`
as the first consumer is the make-or-break move.

---

## 5. Claude API leverage (and hard limits)

The donated lane means the *human* runs their agent; TMC's tooling itself stays agent-neutral and
file-based (per CLAUDE.md). Where Claude (via the human's agent, or a future budget-capped funded
task) adds leverage:

**High-value, human-gated uses:**

1. **Alignment QA / segment-pair triage.** Claude flags likely-misaligned pairs (1:many, semantic
   mismatch, length/script anomalies) for human review — directly attacking the #1 external-source
   risk (§1.3). It *proposes*; the reviewer *decides*.
2. **Glossary/term extraction + candidate definitions.** From reviewed Elyos translations, Claude
   drafts candidate term entries (term, POS, do-not-translate flags, candidate translations) for the
   TBX wedge — high leverage on the lowest-risk surface.
3. **License/PII triage assistant.** Claude pre-classifies source license signals and pre-flags
   probable PII spans (names, IDs, addresses) to *route* units to the human queue faster — raising
   recall of the hard gate without ever being the gate.
4. **Format conversion + round-trip linting.** Claude assists TMX/TBX/XLIFF mapping and writes
   conformance fixtures (e.g., verifying `<prop>`-carried tier/PII metadata survives round-trips,
   incl. third-party tools per §1.5).
5. **Conflict-resolution explanation drafting** — when two segments disagree, Claude drafts the
   rationale for the reviewer to confirm/edit (never to auto-pick).

**Where Claude must NOT decide (hard rule, mirrors CLAUDE.md + the plan's gates):**

- **License determinations are human-verified.** Claude may surface signals; a license/compliance
  reviewer makes the call recorded in `verifiedBy`/`verifiedDate`. No allow-list entry enters on a
  model's say-so.
- **PII / quality clearance is human-reviewed.** Claude raises flags; default-exclude stands until a
  human clears. A model "looks clean" is never sufficient to publish.
- **No fabricated segments, ever.** Claude must not *invent* translations or term equivalents to fill
  coverage gaps; every published unit traces to a real, attributed, reviewed source. Synthetic/MT
  drafts, if ever admitted, are T0-staging-only and never published.
- **Alignment is verified, not asserted.** Claude-proposed alignments require human confirmation
  before a unit reaches tier ≥ T1.
- **Safety-critical content is never auto-applied** from a Claude match — in-context human re-review
  at the originating tier, always.

---

## 6. Ten concrete optimizations

1. **Add `licenseAsserter` / `licenseBasis` to the allow-list schema** so "CC0 on a redistributed
   crawl" can't be mistaken for upstream copyright clearance (§1.1). State the rule: a redistributor
   label never launders upstream copyright.
2. **Support asymmetric source/target licensing** in `TmUnit.license` (or record the restrictive
   join + a note) — PD source + CC-BY-SA translation is common and currently unrepresentable (§1.2).
3. **Make alignment a named gate for non-Elyos sources** with a higher human-review sampling rate on
   low-resource pairs; cite the audit's <50%-quality finding as the rationale (§1.3).
4. **Set a per-language PII detection floor + mandatory human spot-audit** for any language lacking a
   validated detector; "0 PII" is otherwise unfalsifiable for mission languages (§1.4).
5. **Add a "sensitive source document" flag** distinct from named-entity PII — verbatim segments can
   leak context even without classic PII (§1.4).
6. **Stratify success-metric targets by language-resource level** so a 25% match-rate target doesn't
   make the project look like a failure on exactly the under-served languages it exists for (§1.7).
7. **Test TMX/TBX round-trips through a third-party tool** (Okapi/OmegaT), not just TMC's own
   importer, before claiming "no metadata loss" (§1.5).
8. **Make dedup per-partition and add near-duplicate (fuzzy) collapse**; never dedup across license
   classes (§1.6).
9. **Hash-chain or append-only provenance records** so withdrawal-by-provenance and tier-up-only are
   tamper-evident, not just policy (§1.2).
10. **Ship an MCP server (read-only query) as the consumer interface** — `match`, `glossary lookup`,
    `license-filtered export` — so any agent-driven translation workflow (incl. sibling Elyos
    projects) can consume TMC with provenance/tier/license attached to every hit, lowering the M2
    integration cost that currently gates "shipped."

---

## 7. Parallel & perpendicular spin-offs

**Parallel (same machinery, adjacent content):**

- **`oncology-glossary-multilingual` / health-term commons** — a high-value, T2-expert-signed TBX
  termbase for cancer/medical terminology in under-served languages, riding TMC's provenance/tier
  machinery. Strong fit with the OpenWHO low-resource-health-translation gap
  ([arXiv:2508.16048](https://arxiv.org/pdf/2508.16048)). Inherits TMC's safety-critical/no-auto-apply
  rules natively.
- **`vital-info-translations` / `emergency-phrasebooks` shared glossary** — the most likely first
  consumer; TMC supplies consistent rendering of "do not induce vomiting," drug names, dosage terms
  across every leaflet. This is the M2 wedge.
- **`public-domain-translations` TM** — PD literary bitext is the cleanest license partition and an
  ideal seed for the segment-level TM (no NC/SA/ND complications).

**Perpendicular (different shape, same commons):**

- **MT-training-ready clean corpus** — a license-partitioned, PII-screened, provenance-stamped
  export packaged specifically for *fine-tuning* under-served-language MT (the thing OPUS/NLLB give
  you dirty). This turns TMC into the "verified subset" the audit literature keeps calling for —
  positioned as *the clean alternative to mining*, not a competitor on scale.
- **MCP server** (see §6.10) — TMC-as-a-tool for any coding/translation agent; the standards-native,
  provenance-attached query layer no competitor offers.
- **`localization-for-good` shared TM** — software-UI-string register, complementing Mozilla/Pontoon
  but for non-profit/civic OSS in under-served locales.
- **License-engine + allow-list as a reusable Elyos package** — the compatibility matrix, snapshot-
  hash allow-list, and PII gate are useful to *every* Elyos content project, not just translation;
  spin them out as shared `packages/` infrastructure.

---

## 8. Open questions

1. **External-subset stance (the pivotal one).** The plan says "never bulk; case-by-case only." But
   the under-served-language scarcity problem (§1.7) means Elyos-internal seed material may be too
   thin to hit useful match rates. Is there a disciplined path to admit *small, fully-verified,
   PII-screened, alignment-reviewed slices* of OPUS/DGT/FLORES+ for specific pairs — and what's the
   cost ceiling on that verification?
2. **Asymmetric & ShareAlike interactions.** When a PD source pairs with a CC-BY-SA translation,
   which partition does the *unit* land in, and does SA "infect" any pool it joins? The compatibility
   matrix needs explicit unit-level (not just pool-level) rules.
3. **PII detection floor for languages with no NER/PII tooling** — what is the human-audit sampling
   rate, and who is qualified to audit a language the maintainer doesn't read?
4. **Match-rate targets by resource level** — what is a *fair* match-rate target for a revitalization
   language vs a high-resource one, so the metric measures success honestly (§1.7)?
5. **MT-training spin-off licensing** — if TMC publishes a "clean corpus for fine-tuning," does
   NonCommercial labeling actually constrain commercial MT training, and how is that communicated
   without misleading reusers?
6. **Third-party round-trip conformance** — which external tool(s) define the "no metadata loss"
   bar, and what happens to TMC-specific metadata a CAT tool drops?
7. **First consumer + steward** (already the top cold-start risk) — `vital-info-translations` is the
   plan's bet; the go/pivot/hold decision at end-of-M1 needs a named decider and explicit criteria.
8. **Relationship to existing project glossaries** — migrate-into-TMC vs mirror-and-reference, to
   avoid two sources of truth (plan's Open Q #6, still unresolved).

---

### Sources

- OPUS: https://opus.nlpl.eu/ · https://github.com/Helsinki-NLP/OPUS
- ParaCrawl: https://paracrawl.eu/v8 · https://mailman.uib.no/public/corpora-archive/2019-September/030639.html
- Quality-at-a-Glance audit: https://arxiv.org/abs/2103.12028 · https://aclanthology.org/2022.tacl-1.4/
- DGT-TM: https://joint-research-centre.ec.europa.eu/language-technology-resources/dgt-translation-memory_en
- TAUS: https://www.taus.net/resources/blog/taus-launches-the-data-marketplace · https://datarade.ai/data-providers/taus/profile
- MyMemory: https://mymemory.translated.net/ · https://mymemory.translated.net/doc/spec.php
- Tatoeba: https://tatoeba.org/en/downloads · https://en.wiki.tatoeba.org/articles/show/cc0-contributions
- Wikimedia Content Translation: https://www.mediawiki.org/wiki/Content_translation/Translation_tools
- FLORES-200 / NLLB: https://arxiv.org/pdf/2207.04672
- OpenWHO health corpus: https://arxiv.org/pdf/2508.16048
