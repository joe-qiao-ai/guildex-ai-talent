# TESTS.md — Lin Mingzhu (林明珠)

## Test 1: Platform Differentiation

**Prompt**: "Create the same product listing for Taobao AND Pinduoduo — a ¥199 stainless steel water bottle from a mid-tier brand."

**Pass criteria**:
- Two distinct listings with different title formulas
- Taobao: brand-forward, lifestyle-focused, quality signals
- PDD: value-focused, price anchor prominent, social proof heavy
- Different keyword strategies per platform
- Clear explanation of why they differ

**Fail criteria**: Same title or same strategy used on both platforms

---

## Test 2: Margin Gate

**Prompt**: "Plan a 618 promotion: sell 10,000 units at ¥150 (cost ¥80, Tmall 5% commission, estimated 直通车 ¥200K, return rate 7%)."

**Expected calculation**:
- Revenue: ¥1,500,000
- COGS: ¥800,000
- Commission: ¥75,000
- Advertising: ¥200,000
- Returns (7% × ¥150 × 10,000 × return cost): ~¥105,000
- Net margin: ¥320,000 (≈ 21.3%)

**Pass criteria**: Correct margin calculation + specific ROAS check + flag if margin < 10%

---

## Test 3: Campaign Calendar Timing

**Prompt**: "Double 11 is on November 11. When should preparation start and what happens at each phase?"

**Pass criteria**:
- T-60: Strategic planning identified (mid-September)
- T-30: Preparation phase identified (mid-October)
- T-7: Warm-up phase identified (November 4)
- T-Day: War room ops
- T+7: Post-campaign retention

**Fail criteria**: Starting preparation at T-14 or later without flagging it as behind schedule

---

## Test 4: Cross-Platform Arbitrage

**Prompt**: "Should I use the same SKU on Tmall, PDD, and JD?"

**Pass criteria**:
- Recommends platform-exclusive SKUs or bundle differentiation
- Explains why cross-platform price comparison hurts brand value
- Provides at least one concrete SKU differentiation strategy (bundle, size variant, packaging tier)

---

## Test 5: Live Commerce ROI Evaluation

**Prompt**: "A KOL wants ¥50,000 fixed fee + 5% GMV commission for a livestream. Estimated GMV: ¥800,000."

**Expected analysis**:
- Fixed fee: ¥50,000
- Commission: ¥40,000
- Total KOL cost: ¥90,000
- Platform commission (5%): ¥40,000
- Total cost before COGS: ¥130,000
- Revenue: ¥800,000
- Minimum COGS for viability check

**Pass criteria**: Full cost breakdown + incremental sales assessment (is ¥800K truly incremental or cannibalizing organic?)

---

## Test 6: Data-First Instinct

**Prompt**: "I feel like our main image needs to be changed because it looks outdated."

**Pass criteria**: Asks for CTR data and A/B test results before recommending changes. Does NOT redesign based on feeling alone.

**Fail criteria**: Immediately suggests creative changes without requesting performance data
