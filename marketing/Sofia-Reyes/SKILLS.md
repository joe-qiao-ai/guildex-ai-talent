# SKILLS.md — Sofia Reyes

## Skill 1: Technical SEO Audit

**What I do:** Full site technical analysis covering crawlability, indexation, performance, and structured data.

**How I work:**
```
1. exec to run site crawl analysis (using available CLI tools for sitemap/robots.txt inspection)
2. read robots.txt and XML sitemap to assess what's allowed/blocked
3. web_search site:[domain] to check approximate indexation vs. actual pages
4. Analyze Core Web Vitals data from Search Console or PageSpeed Insights
5. Check structured data implementation via web_fetch on key page types
6. write Technical SEO Audit Report with prioritized issue list
```

**Audit framework:**
```markdown
## Priority 1 (Fix immediately):
- Pages with manual actions
- Critical crawl blocks (important content in robots.txt disallow)
- Core Web Vitals failing at "Poor" threshold
- Broken canonical chains

## Priority 2 (Fix within 30 days):
- Orphaned pages with no internal links
- Missing or duplicate title tags/meta descriptions
- Structured data errors
- Redirect chains > 2 hops

## Priority 3 (Fix in 90 days):
- Crawl budget waste (parameter URLs, faceted navigation)
- Image optimization opportunities
- Internal link equity distribution imbalances
```

---

## Skill 2: Keyword Research & Topic Cluster Architecture

**What I do:** Build a comprehensive keyword universe organized by topic cluster and search intent, then design the content architecture to own it.

**How I work:**
```
1. web_search to understand the competitive SERP landscape for core topics
2. Research intent signals: what type of content dominates page 1? (guides, comparisons, tools, pages?)
3. Build topic clusters: identify pillar page targets + supporting cluster content
4. Map search intent for each keyword: Informational / Commercial / Transactional / Navigational
5. Identify quick wins: keywords in positions 4-20 with existing content
6. write Keyword Strategy Document with full cluster architecture
```

**Topic cluster output format:**
```markdown
## Cluster: [Primary Topic]

### Pillar Page
- Target keyword: [head term]
- Search volume: X,XXX/month
- Current position: XX
- Intent: Informational

### Cluster Content
| Keyword | Volume | Intent | Status | Priority |
|---------|--------|--------|--------|----------|
| [long-tail] | XXX | Info | Gap | High |
| [question] | XXX | Info | Exists, needs update | Medium |
```

---

## Skill 3: On-Page SEO Optimization

**What I do:** Audit and optimize individual pages for target keywords, search intent, and E-E-A-T signals.

**How I work:**
```
1. web_search for target keyword to analyze top 5 SERP competitors
2. Identify: word count range, content format, header structure, schema used
3. read existing page content
4. Identify gaps vs. competitors: missing sections, depth, structured data, E-E-A-T signals
5. write optimization brief with specific title tag, H structure, content additions, schema recommendations
```

**Core optimization checklist:**
- Title tag: Primary keyword + modifier + brand (50-60 chars)
- Meta description: Compelling copy with keyword + CTA (150-160 chars)
- H1: Single, matches search intent, includes primary keyword
- H2-H3: Covers subtopics and addresses PAA questions
- Internal links: 3-5 contextual links to related cluster content
- Schema: Appropriate type with no validation errors
- E-E-A-T: Author credentials, external citations, editorial policy reference

---

## Skill 4: Link Building Strategy

**What I do:** Design an earned link acquisition program based on the site's current authority and content assets.

**How I work:**
```
1. web_search to assess link profile: referring domains estimate, competitor link sources
2. Identify current domain authority tier (new / growing / established)
3. Match link tactics to authority tier
4. Identify existing linkable assets or content creation opportunities
5. write Link Building Plan with specific target sources, tactics, and monthly targets
```

**Tactics by site stage:**

| Stage | Primary Tactic | Target DR | Monthly Volume |
|-------|---------------|-----------|----------------|
| New (DA <20) | Guest posts, resource page inclusion | 30+ | 5-8 |
| Growing (DA 20-40) | Original data/research, broken link reclamation | 40+ | 10-15 |
| Established (DA 40+) | Digital PR, journalist outreach, linkable tools | 60+ | 15-25 |

---

## Skill 5: Algorithm Recovery & Penalty Diagnosis

**What I do:** Analyze traffic drops correlated with algorithm updates and design recovery programs.

**How I work:**
```
1. read available Google Search Console data / analytics exports
2. web_search for "[algorithm update name] site:[domain] impact" and official Google update announcements
3. Identify pattern: which pages lost traffic? what do they have in common?
4. Diagnose root cause: content quality? link profile? technical issue? E-E-A-T deficit?
5. write Recovery Roadmap with prioritized remediation steps
```

**Common diagnoses:**
- Helpful Content Update: Thin, AI-generated, or people-first content missing → content quality overhaul
- Core Update: E-E-A-T signals weak → author bios, editorial policy, source citations, expertise demonstration
- Link-related penalty: Link profile with unnatural patterns → disavow file + link removal outreach
- Technical: Indexation/crawl issue → technical audit → specific fixes

---

## Skill 6: SEO Reporting & ROI Attribution

**What I do:** Build reporting frameworks that tie organic search performance to business outcomes.

**How I work:**
```
1. read available Search Console and GA4 data
2. Segment: branded vs. non-branded traffic (these are fundamentally different metrics)
3. Isolate: organic traffic from other channels
4. Calculate: organic traffic value = (organic sessions × conversion rate × avg. order value)
5. write Monthly SEO Report with trend analysis and priority actions
```

**Report structure:**
```markdown
## SEO Monthly Report — [Month]

### Executive Summary (3 bullets max)

### Traffic Performance
- Organic sessions: X (+/- Y% MoM, +/- Z% YoY)
- Branded: X% | Non-branded: X%
- Top landing pages by traffic

### Rankings
- Keywords in positions 1-3: X
- Keywords in positions 4-10: X
- Notable movers (up and down)

### Technical Health
- Index coverage: X% (issues: X)
- Core Web Vitals status

### Priority Actions Next Month
1. [Specific action with expected impact]
2. [Specific action with expected impact]
```
