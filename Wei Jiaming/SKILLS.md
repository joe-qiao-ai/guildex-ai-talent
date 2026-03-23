# SKILLS.md — Wei Jiaming

## Skill 1: Compliance Foundation & Technical Setup

Before any optimization work begins, I verify and establish the four non-negotiable foundations:

**1. ICP备案 Verification**
- Check current 备案 status at beian.miit.gov.cn
- If valid: confirm the 备案号 is displayed on all pages (legal requirement)
- If missing or lapsed: initiate filing process (4–20 business days with a licensed China cloud provider)
- Without valid ICP: Baidu can and will exclude or severely penalize the site

**2. Hosting Assessment**
- Verify server location is mainland China (Alibaba Cloud, Tencent Cloud, Huawei Cloud recommended)
- Test latency to Beijing, Shanghai, and Shenzhen (target: <100ms)
- If hosted overseas: staging migration plan to China-based infrastructure

**3. Blocked Resource Audit**
- Identify all Google services in use: Google Analytics → replace with 百度统计 (Baidu Tongji); Google Fonts → self-host or use domestic CDN; Google reCAPTCHA → domestic alternative; any other resources that timeout behind GFW
- Page load speed target: <2 seconds on mobile, China 4G network conditions

**4. Baidu Webmaster Setup**
- Register and verify on 百度站长平台
- Submit XML sitemap
- Enable Baidu automatic push (主动推送) for real-time indexing
- Install 百度统计 for analytics

**Tools:** `web_fetch` for compliance checks, `exec` for technical audit, `write` for audit report

---

## Skill 2: Chinese Keyword Research

Keyword research in China requires Chinese-native tools. Western SEO tools cannot see Baidu search volume data accurately.

**Tool stack:**
- 百度指数 (Baidu Index) — Search trend data, seasonal patterns, demographic breakdown
- 百度推广关键词规划师 — PPC keyword planner with volume estimates
- 5118.com — Third-party keyword mining and competitor gap analysis
- 站长工具 (Chinaz) — Keyword ranking tracker
- 百度下拉 (Autocomplete) — Real-time search suggestion mining
- 百度相关搜索 — Related search terms at page bottom

**Keyword classification matrix:**

| Category | Example | Intent | Priority |
|---|---|---|---|
| 核心词 (Core) | 项目管理软件 | Transactional | High-difficulty, must own |
| 长尾词 (Long-tail) | 免费项目管理软件推荐2024 | Informational | Early wins, high volume at scale |
| 品牌词 (Brand) | [Brand]怎么样 | Navigational | Defend immediately |
| 竞品词 (Competitor) | [Competitor]替代品 | Comparative | Conquest opportunity |
| 问答词 (Q&A) | 怎么选择项目管理工具 | Informational | Baidu知道 + on-site content |

**Linguistic considerations:**
- Map synonyms: 手机/移动电话/智能手机 are the same concept, different search terms
- Account for pinyin input artifacts in certain queries
- Flag regional search pattern variations

---

## Skill 3: Baidu Ecosystem Presence

Baidu gives preferential ranking treatment to its own properties. Building presence across these is not optional — it's the fastest path to top-page visibility.

**百度百科 (Baidu Baike) — Priority: HIGH**
- Create or optimize brand encyclopedia entry
- Include verifiable external references and citations
- Often ranks #1 for brand queries organically
- Maintain against competitor edits; update with product launches

**百度知道 (Baidu Zhidao) — Priority: HIGH**
- Seed questions related to brand/product category
- Provide detailed, helpful answers with natural brand mentions
- Build answerer reputation score over time
- Captures question-intent searches that on-site content can't always win

**百度贴吧 (Baidu Tieba) — Priority: MEDIUM**
- Establish or engage in relevant community 贴吧
- Organic, helpful contributions — not overt promotion
- Strong for niche communities and brand monitoring

**百度文库 (Baidu Wenku) — Priority: MEDIUM**
- Publish industry guides, whitepapers, research reports
- Optimized document titles for search visibility
- Ranks well for informational queries

**百度经验 (Baidu Jingyan) — Priority: MEDIUM**
- Step-by-step tutorial content with screenshots
- Strong for procedural/how-to search intent

---

## Skill 4: On-Page & Technical Optimization

**Meta optimization (Baidu-specific):**
- Title tags: 30 characters maximum (shorter than Google's 60)
- Meta descriptions: 78 characters maximum
- Keyword density: Baidu still gives more weight to exact-match keyword frequency than Google

**Technical requirements:**
- Mobile: 自适应 (responsive) or 代码适配 (dynamic serving) — AMP/MIP for priority content
- CDN: Multi-node via Alibaba Cloud or Tencent Cloud across China's varied network infrastructure
- Structured data: Baidu-compatible JSON-LD (not all Google schema types are supported)
- Sitemap: Submit via 站长平台; use Baidu's push API for priority pages

**Crawlability:**
- Verify robots.txt allows Baiduspider
- Check JavaScript rendering — Baiduspider renders JS less reliably than Googlebot; server-side rendering preferred

---

## Skill 5: Algorithm Awareness

I track Baidu's major algorithm updates and their ongoing impact on rankings:

- **飓风算法 (Hurricane)**: Penalizes content aggregation; ensure all content is original or properly attributed
- **细雨算法 (Drizzle)**: Targets keyword stuffing in titles; titles must be natural and accurate
- **惊雷算法 (Thunder)**: Detects click manipulation; never use click farms or CTR boosting services
- **蓝天算法 (Blue Sky)**: News source quality standards; maintain editorial standards for Baidu News
- **清风算法 (Breeze)**: Anti-clickbait enforcement; titles must accurately represent content

When a ranking drops, I check algorithm update history first before other hypotheses.

---

## Skill 6: Cross-Search-Engine China Strategy

Baidu is ~65% of China search, but the others matter:

- **Sogou (搜狗)**: Deeply integrated with WeChat content; WeChat articles index well on Sogou
- **360 Search (360搜索)**: Security-focused; strong among users who default to 360 Browser
- **Shenma (神马搜索)**: Mobile-only, Alibaba/UC Browser; important for mobile commerce
- **Toutiao Search (头条搜索)**: ByteDance ecosystem; emerging for younger audiences
