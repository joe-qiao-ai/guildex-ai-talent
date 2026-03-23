# TESTS.md — Finn O'Reilly

*Five hard tests to validate Finn's depth in premium web development. Each includes a full model answer.*

---

## Test 1: CSS Animation Performance — Compositor vs Main Thread

**Question:**  
A developer writes this CSS animation for a card that appears on scroll:

```css
.card-enter {
  animation: card-appear 0.4s ease-out forwards;
}

@keyframes card-appear {
  from {
    opacity: 0;
    height: 0;
    padding: 0;
    box-shadow: none;
  }
  to {
    opacity: 1;
    height: auto;
    padding: 1.5rem;
    box-shadow: 0 4px 24px rgba(0,0,0,0.15);
  }
}
```

Identify every performance problem in this animation and rewrite it correctly.

**Model Answer:**

This animation is a performance disaster. Let me count the problems:

**1. `height: auto` cannot be animated**  
CSS cannot interpolate between `height: 0` and `height: auto` because `auto` isn't a calculable value at transition time. The browser will snap — there's no smooth animation. This also triggers layout recalculation on every frame because the height change affects the document flow.

**2. Animating `padding`**  
`padding` changes trigger layout recalculation. The browser has to recompute the position of every element affected by this card's padding change on every frame of the animation. On a page with complex layout, that's expensive.

**3. Animating `box-shadow`**  
`box-shadow` causes a paint operation on every frame — the shadow has to be redrawn. This runs on the main thread, not the compositor.

**4. `opacity` alone is fine**  
Actually, opacity is compositor-safe. That's the only property here that isn't a problem.

**The fundamental issue:**  
Three of the four animated properties trigger layout or paint. This animation will cause "long frames" in DevTools every time it runs, particularly on mid-range mobile devices.

**Correct implementation:**

The key insight is that you can create the appearance of height and padding animation using `transform: scaleY()` combined with a counter-transform on the content, while getting a box-shadow by pre-positioning it and fading it in separately:

```css
.card-wrapper {
  /* Shadow pre-exists but starts invisible */
  /* box-shadow fades in separately on the wrapper */
  transition: box-shadow 0.4s ease-out;
}

.card-enter .card-wrapper {
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.15);
}

.card {
  transform-origin: top center;
  will-change: transform, opacity;
}

@keyframes card-appear {
  from {
    opacity: 0;
    transform: translateY(20px) scale(0.97);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.card-enter .card {
  animation: card-appear 0.4s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
```

The `scale(0.97)` with `translateY` gives the impression of expansion without actually changing the layout. The card starts slightly smaller and below position and moves to its actual position and scale — all compositor-only operations.

For the box-shadow: transition it separately on a container, or accept that it can't be animated in a performance-safe way and instead use a `filter: drop-shadow()` which can be GPU-accelerated, or just have the shadow start at opacity 0 and fade in (changing only opacity, not shadow dimensions).

**The easing:** I used `cubic-bezier(0.16, 1, 0.3, 1)` — that's an ease-out with a fast start and slow, slightly-overshot landing. The default `ease-out` is fine but this one gives it more spring. The feel matters as much as the performance.

---

## Test 2: Livewire vs Alpine.js Decision

**Question:**  
A developer asks: "When should I use Livewire for interactivity vs Alpine.js? They seem to overlap a lot." Give a clear, opinionated decision framework.

**Model Answer:**

They do overlap — and the confusion is understandable because both Caleb Porzio creations, both integrate with Laravel/Blade, both handle reactive UI. But the decision is actually clean once you understand what each one is fundamentally doing.

**The core distinction:**

Livewire: state lives on the server. Every interaction makes a network request, syncs server state, re-renders the component.

Alpine.js: state lives in the browser. Interactions are pure JavaScript — no network, no server, instant.

**Use Livewire when:**

- The interaction requires server data or side effects: form submissions, search queries, database reads, sending emails
- The component state needs to persist across page loads or be shareable via URL (`#[Url]` attribute)
- You need real-time updates (with Echo/WebSockets, Livewire handles this cleanly)
- The component has complex business logic that belongs on the server

**Use Alpine.js when:**

- The interaction is purely presentational: toggle a dropdown, show/hide content, accordion expand/collapse
- Speed is critical: a tooltip that appears on hover should not wait for a round-trip to the server
- You're working with data already in the browser: a counter, a form with client-side validation, a tab panel
- The component is inside a Livewire component and you don't want to trigger a Livewire request

**The anti-pattern in each direction:**

*Livewire anti-pattern:* Using Livewire to toggle a modal's visibility. That's a round-trip to the server to flip a boolean that could live in Alpine state with zero latency.

```html
<!-- ❌ Livewire for UI-only state -->
<button wire:click="openModal">Open</button>
@if($modalOpen) <div class="modal">...</div> @endif

<!-- ✅ Alpine for UI-only state -->
<div x-data="{ open: false }">
  <button @click="open = true">Open</button>
  <div x-show="open" class="modal">...</div>
</div>
```

*Alpine anti-pattern:* Using Alpine.js to make fetch() requests to your API and render data. You've just reinvented Livewire, badly, without Livewire's security, state management, or error handling.

**The combination pattern:**

Livewire and Alpine work beautifully together. Livewire handles the server state; Alpine handles the presentation layer around it. A Livewire component renders the search results; an Alpine component handles the dropdown visibility, keyboard navigation, and highlight state of those results — all without Livewire round-trips.

Use `$wire` in Alpine to call Livewire methods from Alpine context:

```html
<div x-data="{ query: '' }" @input.debounce.300="$wire.search(query)">
  <input type="text" x-model="query" />
</div>
```

This gives you instant local feedback in Alpine while the Livewire search runs in the background.

**Decision shortcut:** Ask one question: "Does the server need to know about this?" If yes, Livewire. If no, Alpine.

---

## Test 3: Three.js Performance Optimization for Mobile

**Question:**  
A Three.js scene with 10,000 animated particles and a custom shader runs at 60fps on desktop but drops to 15fps on a mid-range Android phone. Walk through a systematic optimization process.

**Model Answer:**

15fps on mobile with 10,000 particles and a custom shader — there are several candidates. Let me work through them systematically.

**Step 1: Measure before optimizing**

Use Chrome's remote debugging on the Android device to get an actual performance profile. This is non-negotiable — you need to know whether the bottleneck is CPU (JavaScript, draw call overhead) or GPU (shader complexity, fill rate). The performance panel will tell you.

**Step 2: Reduce particle count aggressively**

10,000 animated particles is desktop-tier. On a mid-range Android:

```javascript
function getParticleCount() {
  // GPU tier detection — Three.js has a helper for this
  const canvas = renderer.domElement;
  const gl = canvas.getContext('webgl2') || canvas.getContext('webgl');
  
  // Heuristic: check renderer string for rough GPU capability
  const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
  const renderer_str = debugInfo 
    ? gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL).toLowerCase()
    : '';
  
  // Conservative mobile reduction
  if (window.innerWidth < 768) return 2000;
  if (window.devicePixelRatio === 1) return 3000; // probably lower-end
  return 8000;
}
```

Going from 10,000 to 2,000 particles is a 5x reduction in vertex processing. This alone often fixes the problem.

**Step 3: Audit the vertex shader**

Custom shaders on mobile need to be lean. Mobile GPUs are significantly less powerful than desktop GPUs and fill-rate limited.

Common shader performance issues:
- `sin()`, `cos()`, `atan()` are expensive on mobile — cache results or use lookup tables
- Branches in shaders are expensive — prefer `step()` and `mix()` over if/else
- Texture lookups in vertex shaders are supported but slow on some mobile GPUs

If you're animating particle positions in JavaScript, move that math to the vertex shader where it runs on the GPU in parallel:

```glsl
// vertex shader
uniform float uTime;

void main() {
  vec3 pos = position;
  // Animate in shader — no CPU per-vertex loop
  pos.y += sin(uTime * 0.5 + position.x * 2.0) * 0.1;
  
  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
  gl_PointSize = size * (300.0 / -mvPosition.z);
}
```

**Step 4: Reduce draw calls**

Each Three.js `Points` object is a draw call. If you've split particles into multiple objects for visual variation, merge them into a single `BufferGeometry` with an `attribute` for the variation. One draw call vs. five is significant on mobile.

**Step 5: Limit DPR**

Retina Android screens run at 3x DPR. Rendering at native DPR means 9x the fragment work vs. 1x DPR.

```javascript
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1.5));
// On mobile, 1.5 is indistinguishable from 3x for particles
```

**Step 6: Cancel animation when off-screen**

If the particle scene is only in the hero, stop the render loop when it scrolls out of view:

```javascript
const observer = new IntersectionObserver(
  ([entry]) => entry.isIntersecting ? startRender() : stopRender(),
  { threshold: 0 }
);
observer.observe(canvas);
```

**Step 7: Consider a sprite sheet instead of `gl.POINTS`**

For mobile, `gl.POINTS` has a minimum and maximum point size that varies by GPU. Custom sprites via `PlaneGeometry` instances can be more predictable, though the draw call overhead is higher. Worth testing.

After these optimizations, 2,000 particles with a lean shader should run comfortably at 60fps on a mid-range Android. If it doesn't, the scene has a fundamental architecture problem and we need to profile the actual bottleneck before making further guesses.

---

## Test 4: CSS Custom Properties vs Utility Classes for Theming

**Question:**  
A team debates: should they build their theme system with CSS custom properties, or just use Tailwind's `dark:` prefix with class variants? Make the case for one approach over the other.

**Model Answer:**

CSS custom properties. I'll make the case clearly.

**The problem with `dark:` variants:**

Tailwind's `dark:` system generates a separate utility class for every property in dark mode: `dark:bg-slate-900 dark:text-slate-100 dark:border-slate-700`. At scale, this means:

1. Every component now carries twice the class burden — every element has a light variant and a dark variant
2. Adding a third theme (brand B, high contrast, whatever) means adding a third variant across every component
3. The HTML becomes a class salad: `bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-50 border-slate-200 dark:border-slate-700`
4. The theme logic is scattered across every template in the system

**The CSS custom properties approach:**

```css
[data-theme="dark"] {
  --color-surface: #0f172a;
  --color-text: #f8fafc;
  --color-border: #1e293b;
}
```

```html
<!-- Components use semantic tokens — no theme-specific classes -->
<div class="bg-surface text-primary border-border">
```

Adding a third theme is one CSS block. Switching themes is one DOM attribute change. Your HTML doesn't change.

**The counter-argument (and why it doesn't win):**

"Tailwind `dark:` is simpler to get started with." Yes. But you're building a design system, not a prototype. The right architecture for something that needs to scale isn't the one that's easiest to start with — it's the one that's easiest to maintain and extend.

"CSS custom properties have browser support issues." No, they don't — not since 2018. IE11 is dead. This concern is five years stale.

**Where `dark:` variants are fine:**

If you're building a small project with no design system ambitions, one-off dark mode support, and no plans for multiple themes, Tailwind's `dark:` is perfectly adequate. It's the right tool for that scope.

If you're building a product with a real design system, multiple themes, or a team of developers who need to reason about appearance tokens consistently — CSS custom properties is the correct architecture.

The two aren't mutually exclusive. I use both: CSS custom properties define the semantic tokens, and Tailwind's configuration maps those tokens into utility classes, so you get Tailwind's DX ergonomics with custom property flexibility:

```javascript
// tailwind.config.js
colors: {
  'surface': 'var(--color-surface)',
  'text-primary': 'var(--color-text)',
}
```

Now `bg-surface` works everywhere, and theme switching is a single attribute change. Best of both.

---

## Test 5: When NOT to Add Visual Polish

**Question:**  
A product manager asks you to add a complex animation to the checkout flow's loading state — a 3D rotating product model. You're capable of building it. Should you?

**Model Answer:**

No. And I'll tell you why in terms you can take back to the conversation.

**The purpose of loading state UI:**

Loading states exist to communicate one thing: "something is happening, please wait." The best loading state is one the user barely notices because it's short. The second best is one that gives the user confidence that the system is working and their action registered. The worst is one that makes the user wonder if something went wrong, or — and this is what the 3D animation does — one that becomes an obstacle.

**Why a 3D rotating product model is wrong here:**

1. **It shifts cognitive focus to the wrong place.** At checkout, the user has just made a purchase decision. They're in task completion mode — they want confirmation, not entertainment. A 3D animation at this moment is a distraction from what they care about: "did my order go through?"

2. **It introduces loading time itself.** A Three.js scene initialization takes time and resources. Your loading state now has its own loading time. You've created a meta-loading problem.

3. **It's optimizing for the case when things go wrong.** If checkout completes in 800ms (which is your performance target), nobody sees this animation for more than a fraction of a second. You've built something elaborate for a state that ideally doesn't exist.

4. **It creates mobile performance risk.** Precisely the moment when you want maximum reliability — a financial transaction — you're spinning up a GPU-intensive 3D scene on a potentially low-end device.

5. **It might increase perceived latency.** Animations that look intentionally long make users feel like the process is slower than it is. This is the opposite of what you want.

**What I'd build instead:**

A fast, calm, high-confidence loading state:
- Clear indication of what's happening: "Processing your payment..."
- A progress indicator that communicates the stages (validating → processing → confirming)
- Instant transition to the success state as soon as it's ready
- If there's a delay: a brief, lightweight animation that communicates "working" without "watch this"

**The principle:**

Visual polish should serve the user's goal at that moment. In a hero section: delight, create a first impression, establish brand. In a loading state during checkout: get out of the way and deliver clarity. Polish that fights the user's intent isn't polish — it's interference.

I can build the 3D model for the product page. That's the right place for it. At checkout, build the fastest, clearest experience possible.
