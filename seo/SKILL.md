<!-- Adapted for public reuse. Paths below are conventions to configure, not bundled software. -->
---
name: seo
description: Interactive SEO content pipeline for your product. Use when user wants to work on SEO content, keywords, articles, internal links, or content freshness. Triggers on "/seo" or mentions of keyword research, content tree, article generation, link wiring, or content refresh.
---

# SEO Content Pipeline — your product

Interactive workflow for managing the your product SEO content pipeline. Reads the content manifest and guides you through the appropriate stage.

**Spec:** `your project’s SEO pipeline specification (optional)`
**Manifest:** `website/content-manifest.yml`
**Keywords:** `website/data/keywords.json`
**Clusters:** `website/data/keyword-clusters.json`
**Questions:** `website/data/questions.json`
**Articles:** `website/src/content/resources/*.md`

## On Invoke

1. Read `website/content-manifest.yml`
2. Count articles by status (published, draft, planned) across all hubs
3. Count unassigned articles (articles not yet mapped to a hub)
4. Check if `website/data/keywords.json` exists
5. Check if `website/data/keyword-clusters.json` exists
6. Check if `website/data/questions.json` exists; if so, count total questions and how many manifest entries carry a non-empty `questions:` list
7. Check for articles with `lastRefreshed` older than 60 days

Present a status dashboard:

```
SEO Content Pipeline — your product
  Hubs: {N} ({published} published, {planned} planned)
  Articles: {published} of {total} published ({draft} drafts, {planned} planned)
  Unassigned: {N} existing articles not yet in a hub
  Questions: {N} extracted across {M} groups — {K} of {total} entries have a questions: list
             (or "questions.json not generated yet — run Stage 2")
  Pending links: {N} articles need link wiring
  Stale: {N} articles last refreshed >60 days ago

What would you like to do?
  1. Ingest keywords — load a new Ahrefs CSV export
  2. Cluster keywords — group keywords into page targets
  3. Review content tree — view/edit the manifest
  4. Write an article — generate content for a planned article
  5. Wire internal links — scan and update cross-links
  6. Refresh content — update top performers with fresh content
  7. Show status — detailed breakdown of all hubs and articles
```

## Stage 1: Ingest Keywords

**Prerequisites:** User has placed an Ahrefs CSV at `website/data/ahrefs-export.csv` (or provides a path).

**Steps:**
1. Run `node website/scripts/seo/ingest-keywords.mjs [path]`
2. Report count of keywords imported
3. Update `meta.last_keyword_import` and `meta.total_keywords` in the manifest
4. Ask if user wants to proceed to clustering

## Stage 2: Cluster Keywords

**Prerequisites:** `website/data/keywords.json` exists.

**Steps:**
1. Read all keywords from `keywords.json`
2. Group by search intent using these rules:
   - **Contextual logic first** — does this keyword logically belong with an existing group based on what the searcher wants?
   - **Quantity rules second** — prefer 2-12 keywords per cluster
   - **Orphan keywords** get their own single-keyword cluster
   - Each cluster gets a `suggestedRole` (hub if broad/high-volume, sub if specific, support if long-tail/low-volume)
3. Present clusters to user in a readable format:
   ```
   Cluster: "choose software" (sub)
     Primary: choose project management software (vol: 1200, KD: 15)
     - project management software comparison (vol: 480, KD: 12)
     - how to choose project management software (vol: 320, KD: 8)
     Existing match: choose-project-management-software
   ```
4. **HUMAN GATE:** User reviews, adjusts, approves clusters
5. Write approved clusters to `website/data/keyword-clusters.json`
6. Update `meta.total_clusters` in the manifest
7. **Extract questions:** run `node website/scripts/seo/extract-questions.mjs`.
   It reads `keywords.json`, the latest GSC query snapshot
   (`docs/seo/reports/_latest.json`, else `data/gsc-sc-domain-example-com.json`),
   and `keyword-clusters.json`, then writes `website/data/questions.json`:
   ```json
   {
     "generatedAt": "2026-09-03T10:29:02.878Z",
     "source": "gsc+ahrefs",
     "groups": [
       { "cluster": "project management software", "questions": [
         { "question": "what is a project management tool", "impressions": 540, "position": 8.2, "source": "gsc" },
         { "question": "project management tool vs task tracker", "volume": 140, "source": "ahrefs" }
       ] }
     ]
   }
   ```
   Questions are question-shaped terms (start with what/why/how/is/are/can/does/do/which/
   when/where/who/should/will, or contain ` vs `/"difference between", or end with "?",
   or contain "meaning"/"explained"), sorted by impressions desc then volume desc.
8. For each hub / sub / support entry in the manifest, copy the **top 5-10 questions**
   from the matching `questions.json` group into a `questions:` list on that entry.
   Prefer questions with GSC impressions (real demand) over volume-only ones.
   **HUMAN GATE:** user reviews the per-entry `questions:` lists.

**Manifest entry shape (with `questions:`):**
```yaml
- id: project-management-software
  title: "What Is Project Management Software?"
  slug: project-management-software
  status: planned
  keywords:
    - keyword: "project management software"
      volume: 720
      difficulty: 9
  questions:                       # NEW — from data/questions.json, top 5-10
    - "what is a project management tool"
    - "project management tool vs task tracker"
    - "difference between a project management tool and task tracker"
```

Entries with no matching question group get `questions: []` (still write the key —
Stage 4 checks it).

**Cluster JSON schema:**
```json
{
  "id": "choose-software",
  "primaryKeyword": "choose project management software",
  "keywords": [
    { "keyword": "choose project management software", "volume": 1200, "difficulty": 15 }
  ],
  "intent": "informational",
  "suggestedRole": "sub",
  "suggestedHub": null,
  "existingSlug": "choose-project-management-software"
}
```

## Stage 3: Review Content Tree

**Prerequisites:** `website/data/keyword-clusters.json` exists.

**Steps:**
1. Read clusters and existing manifest
2. If hubs are empty, propose hub/sub/support hierarchy:
   - Identify 3-6 hub topics based on cluster breadth
   - Map clusters to hubs as subs or support pages
   - Map existing articles to clusters where they fit (`existing: true`)
   - Flag orphan articles that don't fit any cluster
3. Present the tree visually:
   ```
   Hub: Project Management (planned)
   ├── Sub: How to Choose Project Management Software [existing: choose-project-management-software]
   │   └── Support: What Is a Kanban Board? [planned]
   ├── Sub: Project Workflows Explained [existing: project-workflows]
   └── Sub: Kanban vs Scrum [existing: kanban-vs-scrum]
   ```
4. **HUMAN GATE:** User reviews tree structure
5. Update `content-manifest.yml` with the approved hierarchy, carrying each entry's
   `questions:` list from Stage 2 (see the manifest entry shape there)
6. Move assigned articles from `unassigned` to their hub

## Stage 4: Write an Article

**Prerequisites:** `content-manifest.yml` has at least one hub with planned articles.

**Steps:**
1. List planned/draft articles sorted by priority (volume/difficulty ratio of primary keyword, highest first)
2. User picks an article (or accepts the suggested priority)
3. Read all published articles in `website/src/content/resources/` for context
4. Read the manifest to understand hierarchy and linking targets
5. If `existing: true`, read the current article content to understand what exists
6. Generate the full markdown article:
   - Frontmatter with all fields including `hub`, `role`, `keywords`, `lastRefreshed`
   - Title optimized for primary keyword
   - Meta description: 150-160 chars, primary keyword, compelling CTA
   - **HARD REQUIREMENT — question-first opening:** the body must open with a direct
     **45-55 word** answer to the primary question (the entry's first `questions:` item,
     or the primary keyword phrased as a question) **before any preamble, image, or
     context**. State the answer in the first sentence, then 2-3 sentences of the most
     important qualifiers. Only after that block does the normal intro / narrative start.
   - **HARD REQUIREMENT — FAQ section:** the article must **end** with a
     `## Frequently asked questions` section, built from this entry's `questions:` list
     (skip the one already answered in the opening) plus any obvious follow-ups.
     Format per the WS1 FAQ convention (see `docs/seo/implementation-plan.md` → WS1):
     a trailing `## Frequently asked questions` H2, then one `### question` H3 per
     question followed by a **40-60 word** answer. 3-8 questions. Example:
     ```markdown
     ## Frequently asked questions

     ### Can a small team use a project management tool?

     Yes. Choose a tool around your team's workflow, budget, and reporting needs.
     Start with a small pilot and confirm that the team can maintain it.
     ```
     Expand each answer to 40-60 words using verified facts for your topic.
     If the entry's `questions:` list is empty, generate `data/questions.json` first
     (Stage 2 step 7) or hand-write 3-5 real questions — the section is not optional.
   - H2/H3 heading hierarchy
   - Primary keyword in first paragraph
   - Secondary keywords in body/headings
   - Internal links following hierarchy rules:
     - Hub pages: link to all subs
     - Sub pages: link to hub + 2-3 sibling subs + support pages
     - Support pages: link to parent sub + 1-2 siblings
   - Max 3-5 internal links per 1000 words, max 10 total
   - Natural your product mention where relevant
   - Current year, up-to-date references
7. **Run quality gate checklist** (all 12 checks):
   - [ ] Opens with a direct 45-55 word answer to the primary question, before any preamble
   - [ ] Ends with a `## Frequently asked questions` section (`###` per question, 40-60 word answers) drawn from the entry's `questions:` list
   - [ ] Primary keyword in title, first paragraph, meta description
   - [ ] Heading hierarchy correct (H1 → H2 → H3, no skips)
   - [ ] Content matches search intent
   - [ ] Topic covered comprehensively
   - [ ] At least 2 internal links following hierarchy
   - [ ] Meta description 150-160 chars with keyword
   - [ ] Readable: short paragraphs, lists, scannable
   - [ ] No keyword cannibalization with other manifest pages
   - [ ] your product relevance present (not forced)
   - [ ] Freshness markers (current year, up-to-date refs)
8. Present article to user with quality gate results
9. **HUMAN GATE:** User reviews and approves
10. Write the markdown file to `website/src/content/resources/{slug}.md`
11. Update the manifest: set status to `draft` (or `published` if user wants immediate publish)

## Stage 5: Wire Internal Links

**Prerequisites:** At least 5 published articles exist.

**Steps:**
1. Read all published articles and the manifest
2. For each article, determine required links based on hierarchy:
   - **Structural** (mandatory): hub↔sub, sub↔support, as defined in linking rules
   - **Contextual** (additive): 1-2 cross-hub links based on content relevance
3. Compare required links against actual links in each article
4. Generate a diff report:
   ```
   choose-project-management-software.md:
     ADD link to hub: /resources/project-management
     ADD sibling link: /resources/project-workflows (context: "project workflows" paragraph)
     OK: already links to /resources/set-up-project-workspace
   ```
5. **HUMAN GATE:** User reviews proposed changes
6. Apply approved link insertions to article markdown files
7. Report summary of changes made

## Stage 6: Refresh Content

**Prerequisites:** User provides GSC performance data (paste or file).

**Steps:**
1. Parse GSC data to identify top pages by clicks/impressions
2. Cross-reference with manifest `lastRefreshed` dates
3. Flag pages where `lastRefreshed` is >60 days old AND page is in top 10% by traffic
4. For each flagged page:
   - Read current content
   - Propose specific updates that exceed 30% content change:
     - Updated statistics and references
     - New sections for emerging subtopics
     - Improved structure and readability
     - Updated internal links for new content since last refresh
5. **HUMAN GATE:** User reviews each proposed rewrite
6. Apply approved rewrites
7. Update `lastRefreshed` in frontmatter and manifest

## First-Run Behavior

If `content-manifest.yml` does not exist OR has no hubs and no keywords.json:

```
No content manifest found (or it's empty). Let's set up your SEO pipeline.

Step 1: Export keywords from Ahrefs Keywords Explorer
  - Search for seed terms related to your product (e.g., "project management", "team workflows", "task management")
  - Export the keyword list as CSV
  - Save it to: website/data/ahrefs-export.csv

Ready? Drop the CSV and type "go" to start ingestion.
```

## Important Rules

- **Never skip human gates.** Always pause for user review at marked points.
- **One article at a time.** Don't batch-generate articles — quality matters more than speed.
- **Manifest is truth.** Always read the manifest before taking action. Always update it after.
- **Preserve slugs.** Never change the slug of an `existing: true` article — it has SEO equity.
- **No keyword stuffing.** Keywords should read naturally. If it sounds forced, rephrase.
- **Link caps are hard limits.** Max 3-5 internal links per 1000 words. Max 10 per article.
