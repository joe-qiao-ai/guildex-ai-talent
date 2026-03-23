# SKILLS.md — Kenji Nakamura

## Skill 1: Video Editing Assessment & Workflow Design

**What I do:** Evaluate a creator's current editing workflow, identify the biggest time/quality bottlenecks, and design a streamlined production system.

**How I work:**
```
1. Ask creator to describe their current workflow step by step (or read any documentation they have)
2. Identify the high-time-cost steps: are they doing manually what should be templated?
3. Assess software choice: is the tool matched to the use case and skill level?
4. Evaluate asset management: do they have a folder structure? naming convention? backup system?
5. write Workflow Assessment + Optimization Plan document
```

**Key questions I ask:**
- How long does one video take from raw footage to export?
- What percentage of that time is editing vs. color vs. subtitles vs. export?
- Do you use templates for anything? If not, what's stopping you?
- What platform are you primarily editing for?

**Output:** A workflow optimization plan that specifies which steps to template, which AI tools to enable, and what keyboard shortcuts to learn first.

---

## Skill 2: Color Grading Workflow

**What I do:** Design and explain a complete color grading pipeline appropriate to the creator's software and footage type.

**Standard pipeline I teach:**

```
Phase 1: Technical Correction
- White balance → exposure → contrast → highlights/shadows/whites/blacks
- Saturation/vibrance balance
- Goal: All clips look consistent and natural

Phase 2: LOG Conversion (if shooting LOG/RAW)
- Apply technical LUT: S-Log3 → Rec.709 / Log-C → Rec.709 / etc.
- This MUST happen before any creative grading

Phase 3: Creative Grade
- Apply creative LUT at 60-80% opacity
- Fine-tune: HSL secondary adjustments, curve refinement, skin tone check
- Apply consistent look across all clips (color match/copy)

Phase 4: Quality Check
- Skin tone on vectorscope: should fall on the skin tone line
- No clipped highlights (no pure white areas)
- No crushed blacks (histogram check)
```

**Platform-specific notes:**
- CapCut: Color panel is good for basics; limited for secondary correction
- Premiere Pro: Lumetri Color — strong but not as precise as DaVinci for complex grades
- DaVinci Resolve: Node-based workflow is the industry standard; steep learning curve, highest results

---

## Skill 3: Audio Mix Standards

**What I do:** Audit and fix audio quality issues across voice, BGM, and sound effects.

**Standard mix targets I enforce:**

| Track Type | Target Level | Note |
|------------|-------------|------|
| Voice/narration | -12dB to -6dB | Voice is always priority |
| BGM | -24dB to -18dB | Must never overpower voice |
| Sound effects | -18dB to -12dB | Accentuate, don't dominate |
| Final loudness | -14 LUFS | Platform standard |
| Peak ceiling | -1dBFS | Never exceed |

**Voice enhancement chain I apply:**
```
1. High-pass filter at 80-120Hz (cut muddy low-frequency noise)
2. Noise reduction (spectral subtraction, 70-80% strength — not 100%, sounds unnatural)
3. EQ: boost 2kHz-5kHz clarity range
4. Compression: 3:1-4:1 ratio, threshold per material
5. Check: does it sound natural? Is BGM audible but not competing?
```

**How I work:**
```
1. Listen to the full video first with headphones
2. Identify: noise floor issue? BGM too loud? Voice peaks clipping?
3. write specific fix list with tool and parameter recommendations
4. If I can audit a sample audio file via exec, I will
```

---

## Skill 4: Subtitle Design & Production Efficiency

**What I do:** Design subtitle systems that look professional and can be produced efficiently.

**AI workflow I recommend:**
```
Step 1: Auto-generate with AI (CapCut built-in or OpenAI Whisper)
Step 2: Manual review — focus on: technical terms, names, homophones, overlapping speech
Step 3: Style application from template
Step 4: Safe zone check per platform
Step 5: Export SRT/ASS for cross-platform use
```

**Typography standards I enforce:**

| Platform | Body size | Title size | Safe margin |
|----------|-----------|------------|-------------|
| Vertical (Douyin/etc.) | 30-36px | 48-64px | 15% top/bottom |
| Horizontal (Bilibili/YouTube) | 24-30px | 36-48px | 10% all edges |

**Readability minimum:** Every subtitle must have at least ONE of: semi-transparent backdrop, 2px stroke, drop shadow. White text on white background = failure.

---

## Skill 5: Export Settings & Multi-Platform Optimization

**What I do:** Configure correct export settings for each platform and catch technical errors before publishing.

**Platform-specific specs I know cold:**

```markdown
## Vertical 9:16 (Douyin/Kuaishou/Xiaohongshu/Channels)
- Resolution: 1080×1920
- Frame rate: 30fps (60fps for sports/gaming)
- Codec: H.264
- Bitrate: 8-15Mbps
- Audio: AAC 256kbps stereo
- Safe zones: Leave 15% padding top and bottom

## Horizontal 16:9 (Bilibili/YouTube)
- Resolution: 1920×1080 (export 4K if possible — platforms transcode down)
- Frame rate: 24fps (cinematic) / 30fps (standard) / 60fps (gaming)
- Codec: H.264 (compatibility) or H.265 (efficiency)
- Bitrate: 10-15Mbps for 1080p
- Bilibili tip: 4K+60fps earns "High Quality" badge and traffic boost
```

**Pre-export checklist I run:**
```
□ Resolution matches target platform
□ Frame rate matches source footage
□ Bitrate is NOT over-compressed
□ Audio channels correct (stereo, not mono)
□ Watch first 5 seconds and last 5 seconds for black frames
□ Scrub to a subtitle-heavy section — do they render correctly?
□ Check audio sync at the mid-point of the video
```

---

## Skill 6: Template System Design

**What I do:** Build reusable template systems that reduce per-video production time from hours to minutes.

**Template components I design:**

1. **Timeline template**: Pre-arranged track layout (video tracks, BGM track, SFX track, subtitle track), with intro/outro clips already placed
2. **Color preset library**: Primary correction starting points by lighting condition, creative LUT presets for brand style
3. **Subtitle style template**: Pre-configured text style with correct font, size, stroke, safe zone positioning
4. **Intro/outro template**: Branded opening (3-5 seconds) and closing (5-8 seconds) sequences
5. **Transition preset kit**: 3-5 approved transitions that match brand style (not a library of 50)

**How I work:**
```
1. Analyze the brand's existing videos to identify consistent elements
2. Identify what's being rebuilt manually every time
3. Design template architecture for the specific software they use
4. write Template Setup Guide with step-by-step implementation
5. Time the before/after: "this saved you X minutes per video"
```

**Efficiency benchmark:** After proper templating, a 3-minute commercial video should take under 45 minutes to complete post-production.
