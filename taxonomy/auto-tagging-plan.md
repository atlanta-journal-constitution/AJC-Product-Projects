# Auto-tagging plan (draft v0.1)

Goal: tag every existing and new AJC story against the v0.16 taxonomy (topics, entities, geo, programs, authors, story format, scope) with a provenance and confidence on every tag. Status: mock plan for discussion. Nothing here is built.

## Decisions (Nov 13, 2026)
| Question | Decision |
|---|---|
| Architecture | Rules plus dictionary plus model, with human review |
| Review | Auto-apply high-confidence tags, queue the rest; entity promotions and new geo always need approval |
| Backfill | Whole archive in one pass (the recommendation was last 12 months first, then by traffic; see risks) |
| Evaluation | Editors label about 500 articles; per-facet precision and recall; targets set before launch |

## Baseline: what rules alone achieve
Measured on the Tableau export (7,803 articles, Apr 1 to Nov 12 2026, headlines only, no body text, no labeled set). This is a floor on coverage, not a measure of accuracy.

| Tag source | Articles | Page views |
|---|---|---|
| Top-level topic from the pod | 98.9% | 100% |
| At least one entity from the headline dictionary | 32% | 31% |
| At least one place from the headline | 18% | 19% |
| A program from the headline prefix | 7.6% | 2.1% |
| Any deeper tag (entity, place, program) | 52% | 48% |
| Top-level topic only, nothing deeper | 47% | 52% |

Deeper coverage from headlines by pod: Pro & College Sports 74%, Opinion 57%, Lifestyle 55%, Business 53%, Metro/State 41%, Politics 37%, Investigative 16%, Varsity 13%. Child topics and neighborhoods cannot come from a headline or pod at all.

## Pipeline
1. **Ingest** from the CMS (Arc, by content id): headline, body, section, tags, byline, dateline, publish date, content type. Needs API access and body text; neither is confirmed.
2. **Rules** (cheap, deterministic):
   - pod or section to a top-level topic;
   - headline prefixes to programs (the `detect` rules in taxonomy.json);
   - exact entity and place aliases;
   - byline parsing into name, affiliation, role, then match to the staff author table;
   - content type to story format and page type.
3. **Model** reads headline plus body (trimmed) with the candidate list from the taxonomy, and returns only taxonomy ids in a fixed schema: one primary topic, up to three secondary, entities with the evidence phrase, one primary geo and up to three secondary, scope (local, state, regional, national, world), and a confidence per tag. Output outside the taxonomy is rejected, not accepted.
4. **Resolve**: merge rule and model tags, expand aliases, and apply context rules for ambiguous names (see below).
5. **Gate** by per-facet confidence thresholds, set from the labeled sample:
   - above threshold: apply;
   - below threshold: review queue;
   - new entities, new geo nodes and promotions: always review.
6. **Write back** to CMS fields and to Amplitude event properties: `primary_topic_id`, `secondary_topic_ids`, `entity_ids`, `geo_ids`, `author_ids`, `format_id`, `program_ids`, `scope`.
7. **Learn**: every editor correction is stored as labeled data for the next evaluation.

Every stored tag carries: source (rule, model, editor), confidence, model version and taxonomy version, so any run can be reproduced and rolled back.

## Ambiguity rules (from the export)
- "Georgia" alone is the state, the team or the university: require context (football terms, coach names) before tagging UGA.
- Surnames (Jackson, Collins, Bottoms) need a first name or role in the text.
- "Atlanta" in a headline is often a brand, not a place: geo comes from where the event happened (body, dateline), per the primary-geo rule.
- Opinion pieces keep their subject topic and carry the Opinion label; Readers Write letters have no author.
- Dated roles: a person's office depends on the publish date; use the taxonomy's dated holders, not memory.

## New content
Tag at publish (CMS webhook) with a nightly sweep for misses. Required program and geo fields in the CMS let authors set them directly; the model fills the rest. Volume is small, about 1,040 articles a month in the export (about 35 a day).

## Backfill: the whole archive in one pass
Chosen over the staged option. Risks and mitigations:
- **Old content uses old names and sections** (retired sections such as the legacy COVID section, renamed beats, former officeholders). Map retired sections explicitly and tag with era-aware dated roles.
- **One big run hides errors.** Run in era batches and sample-check each before continuing; keep runs idempotent and keyed by content id and taxonomy version.
- **A partial run should still be useful.** Process newest first within the pass.
- **Cost and time are unknown.** Count the archive first and measure real token lengths on a sample before committing.

## Evaluation
- About 500 articles labeled by editors, stratified by pod, traffic and era (include old content); double-label about 100 to measure agreement between editors.
- Measure precision and recall per facet (topic, entity, geo, program, author, format, scope), not one overall score.
- Set launch targets per facet before running on the archive. No numbers are proposed here; they depend on what the labeled sample shows.
- Re-measure after each taxonomy or model change.

## Open items
1. CMS API access and body text for the archive (needs engineering).
2. Archive size and token-length sample, to estimate cost and runtime.
3. Model and vendor choice, budget, and a terms review for sending article text to a model.
4. Editor time for labeling (about 500, plus 100 double-labeled) and a review queue tool.
5. Labeling guidelines for edge cases (ambiguous names, opinion, multi-topic stories).
6. Owners: who runs the pipeline, who approves entities, who owns the thresholds.
7. Retired-section and era mapping for the archive.
