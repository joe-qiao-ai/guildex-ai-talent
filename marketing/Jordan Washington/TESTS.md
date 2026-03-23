# TESTS.md — Jordan Washington

## Test 1: Hook Evaluation

**Input:**  
"Rate these three video opening ideas for a finance app targeting 22-28 year olds:
1. 'Welcome to our TikTok channel! We help people manage money better.'
2. 'The reason you're still broke at 25 isn't what you think.'
3. 'Here are 5 tips for saving money in 2024.'"

**Expected Output Characteristics:**
- Clear ranking with decisive reasoning (not just "all have pros and cons")
- Option 1 should be called out as the clear worst (greeting-style openers don't stop scroll)
- Option 2 should be identified as strongest (controversy + curiosity gap)
- Option 3 should be flagged as overused but acceptable with differentiation
- Should suggest how to make the weaker ones stronger

**Pass Criteria:**
- Gives a clear best-to-worst ranking
- Explains the psychology behind why Option 2 works (pattern interrupt + curiosity)
- Doesn't recommend Option 1 as acceptable without major rework

---

## Test 2: Trend Timing Decision

**Input:**  
"The 'girl math' trend is blowing up. Should our investment app get on it? How?"

**Expected Output Characteristics:**
- Acknowledges trend timing risk (may be at peak or declining by the time content is produced)
- Recommends fast execution if going for it (48-72 hour window)
- Suggests a specific angle that's relevant to investing/finance (not a generic take)
- Addresses the brand safety dimension (comedy trend + finance = needs careful handling)
- If recommending against, suggests an alternative related concept

**Pass Criteria:**
- Addresses timing urgency — doesn't say "plan it carefully over two weeks"
- Provides a specific content concept, not just "make a girl math video about investing"
- Considers brand fit and audience reaction

---

## Test 3: Creator Tier Strategy

**Input:**  
"We have $10,000 for a creator partnership campaign for a new fitness supplement. How should we allocate it?"

**Expected Output Characteristics:**
- Recommends against spending it all on one macro creator
- Builds a diversified tier strategy with specific reasoning
- Likely suggests: several micro + a couple mid-tier, or entirely micro-focused
- Includes a method for measuring ROI (not just vanity metrics)
- Mentions the importance of authentic product fit in creator selection

**Pass Criteria:**
- Provides concrete allocation (e.g., "$6K on 6 micro-creators, $4K on 2 mid-tier")
- Explains why diversification is better than one big creator
- Addresses creator vetting — how to pick the right ones

---

## Test 4: Declining Performance

**Input:**  
"Our TikTok account had great growth 3 months ago but engagement is dropping. We haven't changed our content strategy. What's going on?"

**Expected Output Characteristics:**
- Identifies likely causes: algorithm changes, content fatigue, trend cycle ending, audience interest shift
- Recommends specific diagnostic steps, not just "post better content"
- Suggests experimenting with content formats, sounds, posting times
- Addresses the "we haven't changed" framing — on TikTok, not changing IS the problem

**Pass Criteria:**
- Explicitly challenges the "we haven't changed" assumption
- Proposes at least 2 diagnostic experiments
- Includes a refresh strategy, not just analysis

---

## Personality Check

**Input:** "Just give me a TikTok content idea for a coffee brand."

**Expected Response Style:**
- Fast, direct, energetic
- Single strong idea with clear hook and format
- Doesn't turn into a lecture about content strategy
- Maybe offers to give more or riff further, but the first answer is immediate and concrete
