# TESTS.md — Luca Romano

---

**1. What are the three Core Web Vitals and their target thresholds?**

**What to look for:** LCP (Largest Contentful Paint) < 2.5s, INP (Interaction to Next Paint) < 200ms, CLS (Cumulative Layout Shift) < 0.1. Must name all three correctly. Note: INP replaced FID in March 2024 — if the response still says FID, the knowledge is outdated.

---

**2. A client says they were "penalized" after a Google update. How do you determine whether it's a manual action or an algorithmic impact?**

**What to look for:** Manual action = visible in Google Search Console under Manual Actions with a specific message from Google. Algorithmic = no message in GSC, traffic drops correlate to a known algorithm update date. The response should direct the user to check GSC Manual Actions first, then cross-reference the drop date with confirmed update timelines. Should not confuse the two.

---

**3. Explain what hreflang does and give an example of when it would be misconfigured.**

**What to look for:** Hreflang tells Google which language/region version of a page to serve to which audience. Misconfiguration examples: missing return tags (hreflang must be reciprocal — page A must reference page B, and page B must reference page A), wrong ISO codes, pointing hreflang to non-indexable pages. Should not just say "it's for international SEO" without the technical detail.

---

**4. What is crawl budget and why does it matter for large sites?**

**What to look for:** Crawl budget = the number of pages Googlebot will crawl on a site within a given period, influenced by site authority and page quality signals. Matters for sites with 100k+ pages because Google won't crawl everything — wasted crawl on low-value pages means important pages may not be crawled or indexed promptly. Fixes: clean up thin/duplicate content, improve internal linking to priority pages, use robots.txt and noindex judiciously.

---

**5. What's the difference between a 301 and a 302 redirect, and when would using the wrong one hurt SEO?**

**What to look for:** 301 = permanent redirect (PageRank/link equity transfers to destination), 302 = temporary redirect (Google may return to index the original). Using a 302 for a permanent URL change means Google may continue indexing the old URL instead of the new one, and link equity may not consolidate properly. The response should not treat these as interchangeable.

---

**6. A competitor is running a negative SEO attack — building spammy links to your site. What do you do?**

**What to look for:** Don't panic. Google's algorithms have become quite good at ignoring spammy links. First step: monitor link profile and identify the spike. Second: assess whether the links are actually affecting rankings (often they don't). Third: if Google is actually acting on the links (ranking drop correlating to link spike), build a disavow file and submit it through Google Search Console. Should explicitly not recommend retaliatory tactics.

---

**7. How does E-E-A-T differ from old E-A-T, and what practical change does "Experience" add?**

**What to look for:** E-A-T (Expertise, Authoritativeness, Trustworthiness) was introduced in Google's Quality Rater Guidelines. The extra E (Experience) was added in late 2022, emphasizing first-hand, direct experience with the subject matter — not just subject matter expertise. Practical implication: content demonstrating the author has personally done/used/experienced the thing they're writing about is now explicitly valued. Particularly important for YMYL content (health, finance, legal).

---

**8. What would you tell a client who wants you to guarantee page-one rankings within 60 days for a competitive keyword?**

**What to look for:** Luca should decline to make the guarantee. Should explain why no ethical SEO professional makes ranking guarantees (Google's algorithms are not controllable, competitive SERPs can take many months). Should offer an alternative framing: a clear methodology, milestones, and leading indicators (crawl health, indexation, keyword movement, traffic trend) that show progress. Must not just refuse without explaining the principled reason.
