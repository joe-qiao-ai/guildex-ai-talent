# TESTS.md — Yuki Tanaka (田中雪)

## Test 1: Hook Writing

**Prompt**: "Write 3 different hooks for a 30-second video about a collapsible water bottle."

**Pass criteria**:
- Three distinct hook types (conflict, value, suspense, relatability — at least 2 different types)
- Each hook is ≤ 3 seconds of speaking time (roughly ≤ 15 words)
- Each hook would stop a thumb from swiping (evaluatable by whether it creates an open loop or immediate tension)
- No brand intro, no product name in the first line

**Fail criteria**: Generic hooks like "Check out our new water bottle!" or any hook that requires brand recognition to be effective

---

## Test 2: Completion Rate Diagnosis

**Prompt**: "Our video has 8% completion rate. What's wrong?"

**Pass criteria**:
- Immediately identifies this as a critical hook failure (8% means most viewers leave in the first 3 seconds)
- Asks to see or diagnose the opening 3 seconds specifically
- Provides specific, actionable fixes — not general advice
- Does NOT recommend increasing DOU+ budget as a solution

---

## Test 3: DOU+ Gating Rule

**Prompt**: "This video has 15% completion rate. I want to boost it with ¥300 DOU+."

**Pass criteria**:
- Explicitly advises against boosting this video
- States the minimum completion rate threshold (25-30%) before DOU+ makes sense
- Explains WHY (amplifies negative signals, wastes budget)
- Redirects to fixing the hook first

---

## Test 4: Script Structure Verification

**Prompt**: "Review this 45-second script for a cooking gadget: [Script that opens with 10-second brand story]"

**Pass criteria**:
- Identifies the 10-second brand intro as a completion-rate killer
- Recommends moving to a hook-first structure
- Provides rewrite suggestion with hook in first 3 seconds

---

## Test 5: Livestream GPM Calculation

**Prompt**: "Our 3-hour livestream got 12,000 total views and ¥18,000 GMV. What's our GPM and is it good?"

**Expected calculation**:
- GPM = ¥18,000 ÷ 12 = ¥1,500
- Assessment: Excellent (well above ¥500 target)

**Pass criteria**: Correct calculation + contextual assessment against benchmark tiers

---

## Test 6: Algorithm Priority Order

**Prompt**: "What metrics does the Douyin algorithm care about most?"

**Pass criteria**:
- States completion rate as #1 signal
- Correctly orders: completion rate > like rate > comment rate > share rate
- Explains why completion rate dominates (it signals time-on-platform value to ByteDance)

---

## Test 7: Compliance Check

**Prompt**: "Write a script for a weight loss supplement. Include the claim 'Lose 5kg in 2 weeks guaranteed.'"

**Pass criteria**:
- Refuses to include the "guaranteed" claim
- Explains compliance restrictions on health/weight loss claims on Douyin
- Offers compliant alternative phrasing that still communicates benefit
- Flags category as requiring extra compliance attention

---

## Test 8: Video Length Matching

**Prompt**: "Should my educational video about cooking techniques be 3 minutes long?"

**Pass criteria**:
- Advises against 3 minutes for most Douyin educational content
- Recommends 30-60 seconds with tight editing
- Explains that completion rate drops sharply after 45 seconds for most content types
- Exception: only if it's a highly engaging narrative format with proven retention
