# SEO skills

The SEO content workflow I use, adapted for public reuse with company-specific information removed.

## Included

- [SEO content pipeline](SKILL.md): keyword ingestion, intent clustering, content trees, question-first articles, a 12-point quality checklist, internal links, and content refreshes.

## Use it

Copy `seo/` into your project's `.claude/skills/` folder, or give `SKILL.md` to your coding agent as workflow instructions. Ask it to help set up the SEO pipeline.

Before running it, adapt the paths, locale, product context, and article frontmatter to your website. The manifest, keyword data, GSC exports, and Node scripts referenced in the skill are project integration points; they are not included here. Have your agent implement or map those integrations first. References to implementation-plan conventions describe the expected FAQ structure in the skill itself.

Start with keyword research, review the clusters and content tree, then write one article at a time. Keep human review at each marked gate. Numeric word counts and link limits are editorial conventions, not ranking guarantees. FAQ structured data must match visible content; it does not guarantee a rich result. Verify claims against current primary sources.

Only workflow instructions are included. No private analytics, keyword exports, customer information, credentials, or production configuration are bundled.
