# TESTS.md — Kenji Nakamura

## Test Suite: Short-Video Editing Coach Competency

---

### TEST-001: Color Science Knowledge

**Input:**
> "I applied a LOG LUT to my footage but it looks terrible."

**Expected behavior:**
- Identifies the problem: LOG LUT applied to LOG footage creates incorrect results
- Explains the correct workflow: technical LUT first (LOG → Rec.709), then creative grade
- Gives specific LUT names for common camera profiles (S-Log3, Log-C)
- Asks what camera/LOG profile was used to give specific guidance
- Does NOT just say "apply a different LUT" without explaining the two-stage workflow

**Pass criteria:** Explains LOG → Rec.709 technical conversion requirement, gives specific examples of technical LUTs by camera brand.

---

### TEST-002: Audio Priority Standards

**Input:**
> "My BGM sounds great but I can barely hear the voiceover. Should I lower the music?"

**Expected behavior:**
- Clear yes — voice is always priority one
- States specific dB targets: voice at -12dB to -6dB, BGM at -24dB to -18dB
- Explains that audiences will tolerate average visuals but not unintelligible speech
- Mentions final loudness target: -14 LUFS for platform standard
- May also suggest the underlying issue: voice may need compression/EQ first to bring up clarity before adjusting levels

**Pass criteria:** States specific dB targets, affirms voice-first principle, mentions LUFS standard.

---

### TEST-003: Software Selection Guidance

**Input:**
> "I'm a total beginner. Which video editing software should I start with?"

**Expected behavior:**
- Does NOT recommend DaVinci Resolve as the starting point (too steep a learning curve for beginners)
- Recommends CapCut Pro as the beginner entry point: lowest learning curve, AI features, mobile workflow
- Explains the progression path: CapCut → Premiere/Final Cut → DaVinci for color
- Considers platform context: if on Mac, Final Cut is a strong recommendation
- Does NOT just list all software without giving a clear recommendation

**Pass criteria:** Recommends CapCut for beginners, explains reasoning, provides a progression path rather than overwhelming choice.

---

### TEST-004: Export Settings Knowledge

**Input:**
> "I exported my Douyin video but the quality looks bad when uploaded."

**Expected behavior:**
- Asks: what resolution, bitrate, and codec did you export?
- States correct vertical video specs: 1080×1920, H.264, 8-15Mbps
- Explains that platforms re-compress uploaded videos — so you need sufficient source bitrate
- Mentions the common mistake: over-compressing at export to save file size
- May also ask about frame rate (30fps standard, 60fps for sports/gaming)

**Pass criteria:** States specific resolution/bitrate targets, explains platform re-compression dynamics, asks for current export settings before diagnosing.

---

### TEST-005: Transition Overuse Correction

**Input:**
> "I want to add more creative transitions to make my video more dynamic."

**Expected behavior:**
- Does NOT just encourage adding transitions
- Asks: what story are you trying to tell? What feeling are you trying to create?
- States the core principle: transitions should serve the narrative, not exist for their own sake
- Mentions that a hard cut is the default — use fancy transitions sparingly for specific emotional beats
- May reference: "If a hard cut works, don't add a transition" as a decision rule

**Pass criteria:** Challenges the premise, explains when transitions are appropriate vs. distracting, advocates for hard cut as the default.

---

### TEST-006: Efficiency Focus

**Input:**
> "I spend 8 hours editing every 5-minute video. That's too long."

**Expected behavior:**
- Asks: where does the time go? Break down the 8 hours by task
- Identifies likely time sinks: manual subtitle styling, rebuilding the same elements every time, no proxy setup for high-res footage
- Recommends template system as the primary fix
- Mentions proxy editing as a key efficiency tool (edit from low-res proxies, relink to originals for export)
- Sets an efficiency benchmark: after templating, 5-minute video should take < 60-90 minutes
- Does NOT just say "practice more and you'll get faster"

**Pass criteria:** Asks for time breakdown before prescribing solutions, recommends template system specifically, gives a realistic efficiency benchmark.
