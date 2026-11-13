# AJC Mock Taxonomy v0.17

Files in this folder
- [taxonomy.json](taxonomy.json): the machine-readable taxonomy (nodes, entity types, decisions log, interest model).
- [taxonomy-explorer.html](taxonomy-explorer.html): searchable browser generated from the JSON (`viewer.template.html` is its source).
- [auto-tagging-plan.md](auto-tagging-plan.md): draft plan to tag existing and new content against this taxonomy.
- [Leadership brief](https://claude.ai/artifact/AMxXCbq2thibV5DoRSKnoE): "AJC Content Taxonomy: Leadership Brief", a Claude Doc. It is private to its owner until shared from the doc's Share menu, so the link will not open for others before then.

Owners: Zachary Firestone, Alex Schrange, Corbin Fricker. Change process: decided later, after the pilot.

## Sources
| Source | What it gave us |
|---|---|
| Amplitude `AJC [PROD SDK]` (764792) | Content properties (`site_section`, `content_sections`, `content_tags`, `content_type`, `content_sub_type`, `content_byline`, `article_type`). Events: Page Viewed, Content Started/Completed, Newsletter Subscribed, User Onboarding Topic Selected/Removed, Search Completed. |
| Amplitude `AJC News App` (631765) | Airship push (about 29M sends logged), push category values (`push_breaking`, `push_politics`, `push_news`, `push_politically-georgia`, `geo_georgia`, `geo_atlanta`), user properties `push_opt_in` and notification permission, and a `topic` property on `user_action` with 29 values (Sports, Weather, Crime & Public Safety, Daily Digest, Braves Report, Hawks Report, Southern Fried Soccer, and others). |
| Tableau Newsroom Dashboard (Quality Points Trended) | About 55 Site Section Names, 9 Pods, 7 Content Subtypes, Editorial Team, Byline, Content Tags. |
| Tableau Programming Manager Dashboard (Byline Content Production, Geographic Content Breakout) | Section performance, byline quality points, 4-tier Target Region, city strings. |

Not used: Piano, GA4, Arc directly; Tableau tabs Underperforming Content, Heat Map, Byline Talent; the full Editorial Team list. Unverified: what `user_action.topic` records in the app (opt-in or just a tap).

## Design: six facets
| Facet | Describes | Example |
|---|---|---|
| Topics (2 levels) | What a story is about | Sports > Football; Politics > Issues > Immigration |
| Entities | Named people, teams, organizations, places, events, legal cases | Atlanta Falcons, a columnist, a Georgia officeholder |
| Geo | Where | Metro Atlanta > Fulton County > Atlanta > a neighborhood; Georgia regions (North, South, Coastal, Central); school and legislative districts as overlays |
| Formats and page types | Shape of content (story formats) and kind of page (page types) | Story format: live blog, video, gallery, newsletter issue. Page type: homepage, section page, scores, profile |
| Programs | Named series and products | Investigations, Politically Georgia, Daily Digest, team reports |
| Authors | Launch nodes: staff with 10+ articles (about 100 of 737) plus all opinion authors; others auto-promoted. Staff table keyed to CMS ids. Org bylines are typed: AJC Editorial Board is a followable opinion author; AJC Staff, Associated Press, partners (ProPublica, The 74, ArtsATL) and Channel 2 are source types, not followable. Readers Write (144 letters, all blank bylines) is a program with no author. Bylines are parsed into name, affiliation and role. An author's beat is derived from the last 90 days of stories. |

IDs are immutable (`ajc:<facet>/<path>`); existing section, pod, tag and byline values map in through a `legacy` field so current reporting still reconciles.

## Decisions made in review (Nov 13, 2026)
| Area | Decision |
|---|---|
| Topics | 58 topics at v0.13. Politics: arenas (Georgia, Legislature, Elections, Statewide Offices, U.S. Congress, Courts & Justice, Parties) plus issue topics. News: Metro, Georgia, Southeast, Local Government, Courts & Trials, Nation & World, Environment & Energy, Crime, Breaking, Traffic. Business: Economy/Jobs/Prices, Retail & Real Estate, Data Centers & Energy, Aviation & Airport. Weather is its own L1. Food & Dining: Restaurants & Openings, Food Events & Pop-ups. Things To Do: Festivals & Concerts, Weekend Guides (low confidence). Lifestyle is an editors-only grouping. Sport is a topic; pro/college/high school is a level attribute. Opinion pieces carry their subject topic plus an Opinion label (an attribute). High school: GHSA class and region as attributes (values from the sports desk). |
| Entities | Rule: curated seed plus auto-promote with newsroom approval. Added in review: Atlanta Dream and other pro teams, high schools, parties as organizations, World Cup 2026 as an event tagged on top of normal topics. v1 types: sports teams, authors, Georgia officeholders and candidates, organizations, places and venues, events, legal cases. Sports stops at team; athletes deferred. Coaches and athletes are auto-promote only (no seed list). Seed groups: major employers and brands, venues, parks and airport, cultural institutions and festivals, government bodies (names to be confirmed by the desks). Offices (Governor of Georgia, Mayor of Atlanta, Attorney General) are entities with dated holders, loaded from an authoritative source. Recurring events are series with yearly occurrences (Peachtree Road Race, Shaky Knees). Hybrid upkeep: auto-extraction proposes, newsroom approves and merges aliases. Politician roles are dated and not hardcoded. |
| Geo | Levels: target region, Georgia region (North, South, Coastal, Central; boundaries from the state desk), county, city, neighborhood; school and legislative districts as overlays. Neighborhoods start with the City of Atlanta. Primary geo is where the event happened, from body and dateline (headline mentions alone do not decide it), plus up to three secondary geos. A required scope attribute: local, state, regional, national, world. Seeds: 16 cities from the Tableau breakout and five metro counties; the rest auto-promoted. Home geo from an onboarding picker and subscriber address/ZIP, not from reading. |
| Formats, page types and programs | Two attributes: page type (homepage, section page, scores, standings, profile, flat page; ignored by interest inference) and story format (article, live coverage, gallery, video, wire, blog, newsletter issue, list, special presentation). Each entity node has a hub page and existing team and player profile pages become hubs.  Followable: newsletters, live coverage, Daily Digest and ePaper. Programs (26 at v0.14): v1 kinds are newsletters, columns/cartoons/series, and special projects and event updates (A.M. ATL, Mike Luckovich, Readers Write, The Win Column, America at 250, Peachtree Road Race updates, and others). Data products (AJC Poll, Where They Stand, Varsity rankings, Maxwell projections) are deferred. A required CMS program field going forward, backfilled by headline-prefix rules; Politically Georgia needs a CMS tag. Followable only with a newsletter or push channel or real demand; low-traffic programs (Sweet Tea, Kick It) are tag-only. One id per program; newsletter, push and follow are channels. |
| Authors | Launch nodes: staff with 10+ articles (about 100 of 737) plus all opinion authors; others auto-promoted. Staff table keyed to CMS ids. Org bylines are typed: AJC Editorial Board is a followable opinion author; AJC Staff, Associated Press, partners (ProPublica, The 74, ArtsATL) and Channel 2 are source types, not followable. Readers Write (144 letters, all blank bylines) is a program with no author. Bylines are parsed into name, affiliation and role. An author's beat is derived from the last 90 days of stories. |
| Interest model | Explicit v1: topic follow, push category opt-in, newsletter subscription, onboarding picks. Inferred interest used silently to rank. Anonymous readers get a device profile from the first visit, merged into the account on login (privacy review needed). Signal weights are calibrated against outcomes (return visits, subscription starts) before launch. |
| Policy | No inferred or offered reader partisan identity (Democrat/Republican/Moderate). Party is a fact about a politician only. Pending legal/privacy review. |

### Entity rule and seeds (added v0.9)
- **Rule:** curated seed list plus auto-promote: a name that appears in N stories over 90 days (N set from the article export) is proposed, and an editor approves before it is followable.
- **Seeds:** the current six teams plus Atlanta Dream and other pro teams (newsroom to list); every high school with a varsity program (linked to county and school district); Georgia officeholders and declared candidates from official rosters with dated roles, newsroom-confirmed; organizations, venues, events and legal cases from a newsroom seed.
- **Why high schools:** `/sports/varsity` is the 3rd-largest section (672,862 page views in 30 days) and had no entity type.

### Coverage check (Nov 13, 2026; Amplitude 764792, 30 days, top 100 `site_section` values)
About 47% of views are homepage and `/`. About 98% of the rest map to an existing node by a hand-assigned proxy. Biggest gaps: Nation & World (68.6K views), aging/senior living (17K), homes and real-estate features (13K), Pulse (10K), Black history and culture (6K), Atlanta Dream (3K). Limit: section paths cannot show child-level coverage; that needs an article-level export (section, tags, headline, byline, page views, 90 days).

### Article export findings (Nov 13, 2026)
Source: Tableau Byline Table (Trend), 7,803 articles, Apr 1 to Nov 12 2026. Columns: Content Id, publish date, byline, pod, headline, quality points, page views, subscription starts, registrations. No section or tags. `Content Id` equals Amplitude `content_id` on Page Viewed (checked on one article: 87,286 vs 87,557 views), so behavior data can be joined. The raw CSV stays in Downloads.
- **Pods by page-view share:** Politics 29.0%, Metro/State 20.2%, Pro & College Sports 18.4%, Lifestyle 9.0%, Varsity 9.0%, Business 8.2%, Opinion 5.1%, Investigative 1.0%.
- **Authors:** 737 distinct; 5.3% of articles have a blank byline; the top 50 authors account for 86% of author-credited views and the top 100 for 96%. Seed about 100 author tags, auto-promote the rest.
- **Neighborhoods are almost absent from headlines** (Buckhead 16 articles, Midtown 24, Beltline 31, West End 0), while 849 onboarding picks chose Neighborhoods. Neighborhood tags need body text, dateline or location data, not headlines.
- **Aliases matter:** UGA appears as "uga", "bulldogs", "georgia football", "georgia baseball", "kirby smart", and "georgia" is ambiguous with the state.
- **Candidates found** (all need newsroom confirmation; full counts in the JSON under `candidates_from_export`): people (for example Keisha Lance Bottoms, Jon Ossoff, Kirby Smart, Mike Luckovich); programs (Readers Write 144 articles, A.M. ATL 121, Win Column, Weekend Reflections, Braves Report, AJC Poll); events (World Cup 150 articles, WNBA Finals, Peachtree Road Race, 2026 governor and Senate races, early voting); topics (data centers, guns, courts and trials); orgs (Georgia Power, Delta, Six Flags).
- **Caveats:** headline keyword counts only; the window includes the 2026 election peak; not yet joined to section or tags.

## Onboarding evidence (Amplitude 764792, `selection_text`, Aug 1 to Nov 13 2026, event totals)
Selected: News 1,308 · Politics 1,292 · Sports 1,239 · Arts & Entertainment 1,176 · Food & Dining 1,132 · Business 1,079 · Things To Do 1,050 · Neighborhoods 849 · Travel 843 · Weather 816 · Education 770 · Wellness 725 · Opinion 707. Removals: Sports 55 (4.4% of its selections), Opinion 25 (3.5%), Business 32 (3.0%), Neighborhoods 24 (2.8%), Education 21 (2.7%), A&E 29 (2.5%); others 1.2% to 2.0%. "Clear all" (214) is a UI action, not a topic.

Takeaways: Neighborhoods is the 8th most-picked topic with no data behind it; Weather rivals Education and Wellness; Sports has the highest removal rate, which may mean readers want teams (unconfirmed). Counts are events, not unique users, and onboarding offers a fixed menu of 13.

## Interest signals
| Signal | Kind | Tracked today |
|---|---|---|
| Topic follow | Explicit | No (new Topic Followed/Unfollowed event) |
| Onboarding pick | Explicit | Yes |
| Newsletter subscribe | Explicit | Yes (web) |
| Push category opt-in | Explicit | Partly: Airship categories and `push_opt_in` exist; per-user topic opt-in unconfirmed |
| Read complete / start / view | Inferred | Yes |
| Search click | Inferred | Yes |

Volumes (Amplitude 764792, Oct 14 to Nov 13 2026): 2.25M people viewed a page; 708K started an article; 126K completed one (about 18% of starters); 27.9K logged in (about 1.2%); 1.5K subscribed to a newsletter; 19K searches. Explicit signals are rare, so inferred reading signals carry the scale.

Format evidence (Page Viewed by `content_type`, 30 days): homepage 4.9M, article 4.4M, section page 1.2M, scores 193K, newsletter 147K, live blog 134K, gallery 106K, wire 78K, blog 69K, video on demand 40K, team profile 23K. `content_sub_type` is essentially empty on the web; Tableau subtypes come from the CMS.

Rules: explicit outranks inferred; inferred scores roll up to parents at 50%; each score keeps its source; obituaries and non-editorial sections never generate inferred interest. Weights and decay windows in the JSON are placeholders to tune against retention and conversion.

## API shapes (proposed)
- Personalization: `GET /users/{id}/interests?facet=topics` returns `[{node_id, score, sources, last_signal_ts}]`; rank content by weighted overlap of its tag ids and ancestors with user scores.
- Search: index every tagged node's label and aliases per article; expose facets topic, entity, geo, author, format, program.
- Content tagging contract (new CMS fields): `primary_topic_id`, `secondary_topic_ids`, `entity_ids`, `geo_ids`, `author_ids`, `format_id`, `program_ids`, `access`.

## Data problems found
1. No topic or neighborhood field exists; topics are inferred from section and URL path.
2. Geo strings are messy (`atlanta,georgia`); a blank-city `,georgia` bucket holds 3.6% of geo-tagged content.
3. Bylines are lowercase, comma-joined text with no ids.
4. `content_tags` mixes real tags with operational ones (`teamchris`, `runautolink-...`, `homepage`).
5. `content_type` is inconsistent and overlaps `content_sub_type`.
6. Tableau sections, Tableau pods, Amplitude `site_section` and Airship push categories all disagree today.
7. Non-editorial sections (sponsor, contests, About Us) must be excluded from interest inference.

## Confidence
| Part | Confidence | Why |
|---|---|---|
| L1 topics, teams, columnists, cities | High | Direct from existing sections and Tableau |
| L2 topics with a legacy section | Medium-High | Map to real section names |
| Politics issue topics, counties | Low / Medium | My additions from the review, not from AJC data; newsroom to confirm lists |
| Neighborhoods, organizations, venues, politician names | Low | Illustrative or not yet populated; need gazetteer and extraction pass |
| Program channel ids, newsletter ids | Unknown | Not found; `null` in JSON |
| Signal weights | Low | Placeholders |

### Programs and authors evidence (export, Apr 1 to Nov 12 2026)
- **Programs by headline prefix:** A.M. ATL 121 articles (only 3.7K views; its value is email), Mike Luckovich 117 (65K views), Readers Write 144, The Win Column 51, Weekend Reflections 26 (10.5K), Braves Report 24, Sweet Tea 21 (245 views), America at 250 16, AJC Poll 13 (56K views), AJC Varsity Rankings 12 (34K), Maxwell Projections about 50 mentions (30K).
- **Authors:** 737 distinct; 493 have one article; 100 have 10 or more; 22 of those write across 3 or more pods. Blank bylines: 410, of which 144 are Readers Write letters; the other 266 (Metro 186, Unassigned 42) look like a data-quality queue. Non-person bylines include AJC Staff, AJC Editorial Board, Associated Press, ProPublica, The 74, Channel 2 Action News.

## Auto-tagging (draft plan; full detail in [auto-tagging-plan.md](auto-tagging-plan.md))
Decisions (Nov 13, 2026): rules plus dictionary plus a model that reads the body and returns only taxonomy ids, with human review; high-confidence tags auto-apply and the rest go to an editor queue (new entities and new geo always reviewed); backfill the whole archive in one pass (the staged option was recommended, so the plan lists the risks: era batches, retired sections, dated roles, newest first, idempotent re-runs); evaluate with about 500 editor-labeled articles and per-facet precision and recall, with launch targets set before shipping.

Rules-only baseline on 7,803 articles (headlines only, a floor and not an accuracy measure): top-level topic from the pod for 98.9% of articles (100% of page views); any deeper tag (entity, place or program) for 52% of articles (48% of views); top-level topic only for 47%. Child topics and neighborhoods need body text.

Not built. Needs CMS API access with bodies, the archive size, a model and budget (with a terms review for sending article text), editor time for labeling, a review tool and owners.

## Governance and tagging practice (from Adobe's tags/taxonomy/metadata guide)
The guide confirms the design (controlled vocabulary with synonyms, hierarchy not flat, many categories per item, core approval team). Note: despite its URL it is not a site-hierarchy guide. Seven practices adopted in v0.8:
1. Tag at creation: required tag fields in authoring; the 500-article pilot is a one-time backfill test.
2. Plain metadata (content type, free/paid access) stays a filter; followable formats (newsletter, live coverage) are modeled as programs.
3. Review cadence, glossary as authoritative vocabulary, usage audits, and a retirement path for stale terms.
4. Per-field documentation: how filled (extracted, derived, manual), maintainer, cost to obtain, onboarding for new staff.
5. Internal tags (`team*`, `runautolink-*`) never reach the personalization or search APIs.
6. Retire, do not delete: archive terms and keep ids.
7. Evaluate IPTC Media Topics and schema.org alignment. This is our suggestion, not from the guide, and not yet compared to our topics.

## Open items
1. Define the change process and who approves additions (see governance practices above).
2. Newsroom review of unmapped sections (Pulse, Access Atlanta, Inspire Atlanta) and the "Trump Administration" name.
3. Source City of Atlanta neighborhood boundaries; state desk defines Georgia region boundaries (especially Central) and assigns regions and counties to the eight newly seeded cities.
4. Build the staff author table and byline parser; map Airship categories and newsletter ids to taxonomy ids; set the traffic threshold for followable programs; add a required program field to the CMS.
5. Privacy review: subscriber ZIP for home geo, device profiles merged on login, silent inferred interest, no-partisan-inference rule. Run the weight calibration (which signals predict return visits and subscription starts). Revisit silent inference before launch.
6. Confirm what `user_action.topic` records in the app; add Topic Followed and Push Topic Subscribed events.
7. Auto-tagging (see plan): get CMS API access with article bodies, count the archive, choose a model and budget, label about 500 articles, and build the review queue before any CMS change.
