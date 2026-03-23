# TESTS.md — Mei-Lin Chen

## Test 1: Triangle Budget

**Prompt:** "My hat mesh has 3,950 triangles. Is that okay to submit?"

**Expected behavior:**
- Technically within the 4,000 limit but risky — exporters add overhead
- Recommends: target 3,800 maximum to leave buffer for export artifacts
- Explains: the hard limit is 4,000 but exporter (Blender FBX) can add geometry
- Does NOT say "it's fine" — pushes for a safe margin

**Pass criteria:** Flags the risk, recommends 3,800 target.

---

## Test 2: Attachment Name

**Prompt:** "I named my hat attachment 'HeadAttachment' — will that work?"

**Expected behavior:**
- No — the correct name for a hat is `HatAttachment`
- `HeadAttachment` is not a standard Roblox attachment point name for hats
- Lists correct standard attachment names for common item types
- Explains: wrong attachment name means the item won't attach correctly to the avatar

**Pass criteria:** Identifies the wrong name, provides the correct Roblox standard name.

---

## Test 3: Layered Clothing Missing Cage

**Prompt:** "My layered jacket submitted fine but it clips through the avatar's arms during animations."

**Expected behavior:**
- Primary cause: `_InnerCage` mesh doesn't match the jacket geometry, or is missing from the rig
- Check: is the inner cage volume sufficient to enclose all animated positions?
- Fix: adjust inner cage inward offset, re-test on run and jump animations
- Mentions: bone weights must be identical between jacket and cage

**Pass criteria:** Identifies inner cage as the issue, explains the fix.

---

## Test 4: Moderation Risk

**Prompt:** "I want to put a Nike swoosh on my sneaker UGC item. Will that pass moderation?"

**Expected behavior:**
- Clear answer: No — real-world brand logos cause moderation rejection
- Roblox policy explicitly prohibits copyrighted brand logos
- Fix: create an original logo design inspired by the aesthetic, not the actual brand
- Notes: this is not a gray area — it's an automatic rejection

**Pass criteria:** Clear no with policy reason and practical alternative.

---

## Test 5: Pricing Strategy

**Prompt:** "How should I price my new hat?"

**Expected behavior:**
- Asks: have you searched comparable items on Creator Marketplace?
- Research-first: look at similar hats by style, complexity, and creator reputation
- Guidance: price within 15% of comparable items for a new creator without brand recognition
- Notes: pricing too high without an established store delays sales significantly
- Does NOT give a number without asking about the item type and market research

**Pass criteria:** Asks for market research, provides a methodology rather than a magic number.
