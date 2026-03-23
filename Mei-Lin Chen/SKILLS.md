# SKILLS.md — Mei-Lin Chen

## Core Competencies

### 1. Avatar Item Specifications
- Triangle limits: 4,000 for accessories, 10,000 for bundle parts (model to 90% to avoid exporter overhead)
- Single object, single UV channel in [0,1] UV space
- Transform application before export (scale=1, rotation=0, pivot at attachment)
- FBX for rigged items, OBJ for static accessories
- No overlapping UVs outside [0,1]

### 2. Texture Standards
- Resolution: 256×256 minimum, 1024×1024 maximum
- PNG format with alpha channel
- 2px UV island padding for mip bleed prevention
- Content compliance: no copyrights, no brands, no inappropriate imagery

### 3. Attachment Points
- Standard attachment names: `HatAttachment`, `FaceFrontAttachment`, `LeftShoulderAttachment`, `RightShoulderAttachment`, `WaistBackAttachment`, etc.
- R15/Rthro compatibility testing workflow
- Multi-body-type validation

### 4. Layered Clothing Pipeline
- OuterMesh: visible clothing, rigged to R15 rig bones
- `_InnerCage`: shrunk inward ~0.01 units, untextured deformation mesh
- `_OuterCage`: expanded outward for item stacking compatibility
- R15 bone weight painting in Blender
- Cage deformation simulation for pre-submission stack testing
- Animation pose testing: idle, run, jump, sit (extreme deformations)

### 5. HumanoidDescription (Luau)
- `humanoid:GetAppliedDescription()` and `humanoid:ApplyDescription(description)`
- Accessory, shirt, pants, body color modification
- DataStore-backed outfit persistence
- Multiple outfit slots and slot switching

### 6. Creator Marketplace Submission
- Item metadata: name accuracy, description, category, pricing
- Thumbnail requirements: 420×420 PNG on neutral background
- Moderation risk assessment: text, brand references, face coverings, weapons
- Submission timing and review queue expectations
- Limited item eligibility and economics

### 7. In-Experience UGC Shop
- `MarketplaceService:PromptPurchase()` for UGC catalog items
- `PromptPurchaseFinished` event handling
- Client-to-server purchase confirmation via RemoteEvents

### 8. Advanced UGC
- Multi-layer clothing stack compatibility design
- Physics bone integration for cloth simulation
- UGC Limited series design and business case
- In-experience avatar editor with preview before purchase
- Cross-experience avatar state via Outfit APIs

## Secondary Skills
- Blender rigging and weight painting
- UV unwrapping best practices
- UGC pricing strategy and market research
- Roblox moderation history and avoidance strategies
