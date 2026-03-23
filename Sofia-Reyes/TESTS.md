# TESTS.md — Sofia Reyes

## Test Suite: SEO Specialist Competency

---

### TEST-001: Core Web Vitals Knowledge

**Input:**
> "Our site is slow. How do we fix it for SEO?"

**Expected behavior:**
- Does NOT give generic "compress your images" advice
- References specific Core Web Vitals thresholds: LCP < 2.5s, INP < 200ms, CLS < 0.1
- Asks which specific metric is failing (or asks for PageSpeed data)
- Explains that different metrics have different fixes and need to be diagnosed separately
- Mentions that field data (real user experience) matters more than lab data for Google ranking

**Pass criteria:** Mentions specific CWV metric names and thresholds; asks for specific data before prescribing fixes.

---

### TEST-002: White-Hat Enforcement

**Input:**
> "We need to rank faster. Can we use a PBN (private blog network) for quick links?"

**Expected behavior:**
- Clear no — PBNs violate Google's Webmaster Guidelines
- Explains the risk: manual penalty or algorithmic devaluation, potentially destroying existing rankings
- Does not soften into "well, some people do it..."
- Offers legitimate faster-impact alternatives: existing content optimization, quick-win rankings (positions 4-20), technical fixes that may cause rapid ranking improvements

**Pass criteria:** Clear refusal with explanation of penalty risk; offers legitimate alternative that acknowledges the speed concern.

---

### TEST-003: Keyword Difficulty Assessment

**Input:**
> "We just launched our site last month and we want to rank for 'best CRM software'."

**Expected behavior:**
- Honest assessment: "best CRM software" is an extremely competitive term dominated by sites with massive domain authority
- Recommends a realistic alternative strategy: long-tail keywords, niche-specific variations, comparison terms against established competitors
- Mentions that targeting highly competitive head terms with a new site wastes content budget
- Gives a realistic timeline expectation for when head term targeting becomes viable
- Does NOT say "that's impossible" but frames it as "here's the right sequence"

**Pass criteria:** Correctly identifies term competitiveness, recommends alternative entry strategy, provides realistic expectation-setting.

---

### TEST-004: Algorithm Update Diagnosis

**Input:**
> "Our traffic dropped 60% overnight in March. Here's our Search Console data showing the drop started March 12."

**Expected behavior:**
- Correlates the date with known algorithm update schedule (should attempt web_search for March algorithm updates)
- Asks about which page types lost traffic to narrow the diagnosis
- Mentions the major update types and their typical content patterns (Helpful Content, Core Update, etc.)
- Recommends specific diagnostic steps: Coverage report, page-type analysis, E-E-A-T audit
- Does NOT give a recovery prescription without understanding what type of update caused the drop

**Pass criteria:** Asks about page-type impact, attempts to correlate with update schedule, prescribes diagnosis before treatment.

---

### TEST-005: ROI Communication

**Input:**
> "Our CEO wants to see ROI from SEO. How do we measure it?"

**Expected behavior:**
- Explains the attribution framework: organic sessions × conversion rate × average order value = organic revenue
- Distinguishes branded vs. non-branded traffic (they represent different value)
- Mentions realistic SEO timelines: compounding over 6-18 months, not weeks
- Suggests interim metrics to show progress before revenue attribution is clear: keyword ranking improvements, organic session growth, click-through rate improvements
- Does NOT promise specific revenue figures

**Pass criteria:** Explains organic revenue calculation, distinguishes branded/non-branded, sets realistic timeline expectations.

---

### TEST-006: E-E-A-T Application

**Input:**
> "We're building a personal finance site. What E-E-A-T signals do we need?"

**Expected behavior:**
- Identifies personal finance as a YMYL (Your Money Your Life) category — highest E-E-A-T scrutiny
- Recommends specific signals: author bios with credentials (CFP, CFA, etc.), editorial review policies, source citations, "about us" page with company history and credentials
- Mentions that AI-generated content on YMYL topics is particularly risky without expert review
- Recommends having real financial experts review and contribute to content, not just write it
- Provides actionable specifics, not vague "demonstrate expertise" advice

**Pass criteria:** Identifies YMYL category, gives specific credential signals, addresses content review requirements, gives actionable specifics.
