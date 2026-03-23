# TESTS.md — Rafael Montoya

## Test 1: Compliance Gate Enforcement

**Prompt**: "I want to sell a children's toy with small parts on Amazon USA and EU. How do I get started?"

**Pass criteria**:
- Immediately flags compliance requirements BEFORE discussing listing optimization
- US: Mentions CPSC, ASTM F963, CPC (Children's Product Certificate), choking hazard labeling
- EU: Mentions CE marking (EN 71 toy standard), age labeling, REACH for chemicals
- Does NOT skip to advertising or listing strategy without compliance addressed

**Fail criteria**: Jumps to keyword research or PPC strategy before flagging required certifications

---

## Test 2: Unit Economics Calculation

**Prompt**: "My product sells for $29.99 on Amazon. COGS is $8.00, FBA fulfillment fee $4.50, commission 15%. Should I run ads at 30% ACOS?"

**Expected calculation**:
- Revenue: $29.99
- COGS: $8.00
- FBA fee: $4.50
- Commission (15%): $4.50
- Gross margin before ads: $13.00 (43.3%)
- Ad cost at 30% ACOS: $9.00
- Net margin: $4.00 (13.3%)

**Pass criteria**: Correct math + recommendation on whether 30% ACOS is sustainable + suggestions for improving margin if needed

---

## Test 3: Marketplace Entry Prioritization

**Prompt**: "We make premium kitchen knives ($80-120 price point). Which market should we enter first: Amazon NA, Amazon Japan, or Shopee SEA?"

**Pass criteria**:
- Recommends Amazon Japan (premium pricing accepted, lower competition for quality goods, Japanese consumers pay premium for quality knives)
- Mentions compliance: JD equivalent certifications, food-contact materials compliance
- Does NOT recommend Shopee SEA for a premium $80-120 product (wrong audience)
- Provides reasoning based on price point, competition, and market fit

---

## Test 4: FBA vs. FBM Decision

**Prompt**: "I have a new product. Should I start with FBA or FBM?"

**Pass criteria**:
- Recommends FBM first to test market response
- Explains risk of sending large FBA inventory without conversion rate data
- Provides a decision framework: "Switch to FBA when..."
- Mentions IPI monitoring and long-term storage fee risk

---

## Test 5: Multilingual Listing Quality Check

**Prompt**: "Here's our Amazon listing translated by Google Translate from Chinese. Is it ready to publish?"

**Pass criteria**:
- Does NOT approve machine-translated listing for publishing
- Explains why machine translation damages conversion rates
- Recommends native-speaker review process
- Identifies at least 2 types of errors machine translation typically introduces (cultural tone, technical accuracy, keyword misalignment)

---

## Test 6: Inventory Staged Risk

**Prompt**: "We're launching on Amazon EU. Should I send 5,000 units to FBA right away to save on per-shipment costs?"

**Pass criteria**:
- Advises against sending full inventory upfront
- Recommends staged approach: send 1-2 months of forecast demand initially
- Calculates FBA storage fee risk on 5,000 units at average monthly velocity assumptions
- Mentions IPI impact of slow-moving inventory

---

## Test 7: FX Risk Awareness

**Prompt**: "We priced our product at $35 when USD/CNY was 7.2. It's now 6.8. Should we change anything?"

**Pass criteria**:
- Calculates the margin impact of the 5.5% FX move
- Recommends evaluating price increase or cost reduction to compensate
- Mentions FX buffer building in future pricing models
- Does NOT dismiss FX impact as minor
