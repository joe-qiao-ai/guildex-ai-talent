# TESTS.md — Jada Washington

## Test 1: Retention Diagnosis

**Prompt:** "Our D7 retention is 8%. What's wrong?"

**Expected behavior:**
- 8% D7 is significantly below healthy benchmarks (target: 15%+)
- Asks: what's the D1 retention? (if D1 is also low, fix onboarding first — D7 compounds on D1)
- If D1 is good but D7 is low: players enjoy the first session but have no reason to return
- Recommends: daily reward system, preview of unlockable content, "you're X% to level up" hooks
- Does NOT give a single fix without understanding the D1 data first

**Pass criteria:** Asks for D1 data, explains the D1→D7 relationship, contextual recommendations.

---

## Test 2: Game Pass Design

**Prompt:** "Should I make my best weapon a Game Pass exclusive?"

**Expected behavior:**
- Red flag — "best weapon" implies pay-to-win
- Explains: the free experience must be complete and fun; paid content enhances, doesn't fix
- Alternative: Game Pass could offer cosmetic variants, XP boost, extra character slots, quality-of-life features
- If the "best weapon" is the only weapon players want, the free loadout is broken — fix the design
- References Roblox's policies on fair gameplay

**Pass criteria:** Identifies pay-to-win problem, provides alternative Game Pass design ideas.

---

## Test 3: Onboarding Pacing

**Prompt:** "Our tutorial has 8 steps with dialogue boxes before the player can move. Is that too long?"

**Expected behavior:**
- Yes — players need to perform the core action within 60 seconds
- 8 dialogue steps before movement is a drop-off funnel
- Fix: cut tutorial to 1–2 contextual hints, let players discover through play
- The first success must be guaranteed and feel rewarding, not explained
- "Teach through play, not through text"

**Pass criteria:** Clear yes-too-long, recommends action-first design principle.

---

## Test 4: Analytics Implementation

**Prompt:** "How do I know where in my onboarding players are dropping off?"

**Expected behavior:**
- `AnalyticsService:LogCustomEvent()` at key funnel points
- Track: joined → minute 2 → minute 5 → minute 15 → first reward → session end
- View in Creator Dashboard analytics funnel view
- Explains: the earliest drop-off point is the highest priority to fix
- Bonus: mentions A/B testing setup for testing fixes

**Pass criteria:** Recommends AnalyticsService, identifies funnel checkpoint placement.

---

## Test 5: Dark Pattern Identification

**Prompt:** "I want a currency system where 100 Robux = 'StarCoins' but the price for items is shown in StarCoins, not Robux."

**Expected behavior:**
- Flags this as a dark pattern — obscures real cost in a second currency
- Especially problematic for minors who may not easily convert
- Notes: Roblox policies may also flag this depending on implementation
- Alternative: be transparent about Robux costs; use a soft earned currency for gameplay rewards, separate from paid currency

**Pass criteria:** Identifies the dark pattern, explains the minor audience context, offers transparent alternative.
