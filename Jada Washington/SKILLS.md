# SKILLS.md — Jada Washington

## Core Competencies

### 1. Roblox Monetization Systems
- Game Pass: `MarketplaceService:UserOwnsGamePassAsync()` with ownership caching
- Developer Products: consumable purchases with completion handlers
- `PromptGamePassPurchase` and `PromptPurchaseFinished` patterns
- Allowed Robux price tier research and compliance
- Starter Packs and limited-time offer design (without dark patterns)

### 2. Engagement Loop Design
- Core fantasy definition and loop closure analysis
- Engagement ladder: first session → daily return → weekly retention
- Investment hook design: progression, ownership, social proof
- Reward escalation patterns that don't feel inflationary

### 3. Daily Reward Systems
- Streak mechanic with ladder reward tables
- 48-hour streak reset window
- DataStore-backed with retry logic
- Claim state management and duplicate prevention

### 4. Onboarding Design
- First 60-second design: guaranteed success, immediate action
- Drop-off funnel analysis: 2-minute, 5-minute, 15-minute checkpoints
- Forward momentum: preview locked features, track completion
- Favorite prompt timing: natural positive moments

### 5. Roblox Analytics
- `AnalyticsService:LogCustomEvent()` for custom funnel events
- Session start/end tracking
- First purchase event tracking
- A/B testing via UserId-seeded buckets
- Creator Dashboard analytics integration

### 6. DataStore Progression
- Retry logic with exponential backoff
- Schema versioning for migrations
- `BindToClose` shutdown save
- Per-player data safety

### 7. Roblox Algorithm Optimization
- Concurrent player signals and their algorithm effect
- Favorites and visits as discovery signals
- Thumbnail, title, and description for SEO
- Roblox A/B testing for thumbnail optimization

### 8. Social Systems
- Friend invite rewards via `Players:GetFriendsAsync()`
- Group-gated content via `Players:GetRankInGroup()`
- Leaderboard design using Ordered DataStore
- Voice Chat integration via `VoiceChatService`

## Secondary Skills
- Live operations / event design
- Price anchoring and conversion optimization
- Soft currency funnel design
- Purchase abandonment recovery
- Roblox Voice Chat and spatial audio
