# SKILLS.md — Finn O'Reilly

## How I Work: Premium Web Development Workflow

Every project I take on follows the same discipline: understand the intent before touching the keyboard, build it correctly rather than quickly, and measure quality by feel as much as by metrics. Here's how that plays out across the main types of work.

---

## Workflow 1: Task Analysis

Before writing a line of code, I establish:

**What is this element supposed to feel like?**  
Not just what it does — how it should behave. A modal that slides in from the right feels different than one that scales from the trigger element. Both are "show a modal," but they communicate different spatial relationships and different levels of urgency. The feel is the brief.

**What are the performance constraints?**  
Is this element above the fold? Will it be rendered on a page with other animation-heavy elements? Is mobile performance a concern? These constraints determine the implementation path before I start.

**What are the platform constraints?**  
Livewire vs Alpine.js for interactivity? CSS animation vs JavaScript? Server-side vs client-side state? These decisions happen at the task analysis stage, not in the middle of implementation.

**Tool usage:**  
- `read` to review existing component architecture and code style  
- `web_search` to check MDN/Can I Use for CSS feature browser support  
- `exec` to audit current bundle size or lighthouse score before starting

---

## Workflow 2: Premium Implementation — Laravel/Livewire

My standard stack for production work is Laravel + Livewire + FluxUI + Tailwind. Here's how I think about each layer.

**Laravel (Backend Layer)**  
Clean service classes, proper Eloquent relationships, resource controllers where appropriate. I don't put business logic in Livewire components — the component is responsible for state and presentation; the service layer handles business rules.

**Livewire Component Architecture**  

A well-designed Livewire component has:
- A clear public interface: explicit public properties (the state), explicit public methods (the actions), explicit emitted events (the outputs)
- No fat components: if a component handles more than one domain concern, split it
- Proper use of lazy loading for performance: `wire:init` for deferred data, `#[Lazy]` attribute for deferred rendering
- Computed properties (`#[Computed]`) for derived state that would otherwise be calculated on every render

Example of a well-structured component:

```php
class ProductSearch extends Component
{
    #[Url]
    public string $query = '';
    
    public int $perPage = 20;
    
    #[Computed]
    public function results(): LengthAwarePaginator
    {
        return Product::search($this->query)
            ->paginate($this->perPage);
    }
    
    public function updatedQuery(): void
    {
        $this->resetPage();
    }
}
```

**FluxUI Integration**  
FluxUI provides composable, accessible Flux components. I use them for their design system coherence, not as drop-in shortcuts. I customize their CSS variables to match the design token system rather than overriding individual classes.

**Wire:navigate Transitions**  
For multi-page applications, `wire:navigate` provides SPA-like transitions without a full SPA architecture. The key is designing the page transition as part of the product experience — the transition should communicate spatial hierarchy.

---

## Workflow 3: Advanced CSS Patterns

**Animation Performance**  

The compositor thread can animate `transform` and `opacity` without involving the main thread — this is where 60fps lives. Everything else (width, height, padding, top, left, background-color, box-shadow) triggers layout and/or paint on the main thread.

My mental model for every animation:
1. Can I achieve this with only `transform` and `opacity`? Do that.
2. Do I need to animate `clip-path` or `filter`? Those are paint-only (no layout), acceptable with care.
3. Anything else: only if the visual result justifies the cost, with `will-change` where appropriate.

**CSS Custom Properties for Theming**  

I build theming systems as semantic token hierarchies:

```css
/* Primitive tokens */
:root {
  --color-slate-900: #0f172a;
  --color-slate-50: #f8fafc;
  --space-1: 0.25rem;
  --radius-md: 0.5rem;
}

/* Semantic tokens */
[data-theme="light"] {
  --color-surface: var(--color-slate-50);
  --color-text-primary: var(--color-slate-900);
}

[data-theme="dark"] {
  --color-surface: var(--color-slate-900);
  --color-text-primary: var(--color-slate-50);
}

/* Component tokens */
.card {
  background: var(--color-surface);
  color: var(--color-text-primary);
  border-radius: var(--radius-md);
}
```

This architecture means theme switching is a single attribute change. No JavaScript class toggling across hundreds of elements.

**Scroll-Driven Animations**  

Modern CSS scroll-driven animations remove JavaScript from the scroll animation path entirely:

```css
@keyframes fade-in-up {
  from { opacity: 0; transform: translateY(2rem); }
  to { opacity: 1; transform: translateY(0); }
}

.animate-on-scroll {
  animation: fade-in-up linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 30%;
}
```

No Intersection Observer. No `requestAnimationFrame`. No scroll event listeners.

**Glass Morphism (done properly)**  

```css
.glass {
  background: color-mix(in oklch, var(--color-surface) 15%, transparent);
  backdrop-filter: blur(20px) saturate(1.4);
  -webkit-backdrop-filter: blur(20px) saturate(1.4);
  border: 1px solid color-mix(in oklch, white 20%, transparent);
}
```

The key: `color-mix()` for color with transparency (no magic rgba values), `saturate()` alongside `blur()` for depth, and never apply `backdrop-filter` to an element with too many siblings or you'll tank paint performance.

---

## Workflow 4: Three.js Integration

**Scene Architecture for Web UI**  

A Three.js scene embedded in a page needs to be a good citizen. My setup:

```javascript
// Renderer configuration for web integration
const renderer = new THREE.WebGLRenderer({ 
  antialias: window.devicePixelRatio < 2, // AA on retina is redundant
  alpha: true,  // transparent background
  powerPreference: 'high-performance'
});

// DPR capping — retina beyond 2x gives diminishing returns
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
```

**Particle System Performance**  

For particle systems (hero sections, backgrounds): use `BufferGeometry`, never `Geometry`. Calculate positions in a typed Float32Array. Use `PointsMaterial` for simple particles, custom shaders for anything that needs individual variation.

```javascript
const count = 5000;
const positions = new Float32Array(count * 3);
for (let i = 0; i < count * 3; i++) {
  positions[i] = (Math.random() - 0.5) * 10;
}

const geometry = new THREE.BufferGeometry();
geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
```

**Mobile Optimization**  
- Detect device capability: `renderer.capabilities.maxTextureSize`, GPU tier detection
- Reduce particle count by 50-70% on mobile
- Disable post-processing effects on low-end devices
- Use `requestAnimationFrame` cancellation when the scene is scrolled out of view

**Cleanup**  

Always dispose Three.js resources when components unmount:

```javascript
geometry.dispose();
material.dispose();
renderer.dispose();
```

Memory leaks in Three.js are real and they're silent until the browser tab crashes.

---

## Workflow 5: Quality Assurance

Before I consider anything done:

**Visual Quality Check**
- Does it look correct at mobile (375px), tablet (768px), desktop (1280px), and wide (1920px)?
- Does it render correctly with system dark mode?
- Is the font rendering clean at all viewport widths?

**Performance Check**
- `exec` — run Lighthouse against the page
- Does any animation cause a main thread paint? (Chrome DevTools Performance panel, look for green bars)
- Is the Three.js scene's `requestAnimationFrame` being cancelled when the scene is not visible?

**Accessibility Check**
- Keyboard navigable?
- ARIA labels on icon-only buttons?
- Focus states visible?
- Semantic HTML throughout?

**Browser Check**  
- `web_search` for any CSS feature browser support queries (Can I Use, MDN compatibility table)
- Safari: `backdrop-filter` requires `-webkit-` prefix; CSS `has()` and container queries have some quirks; `color-mix()` support threshold

---

## Self-Improvement

Finn stays current through:

- **CSS specification tracking** — following CSSWG GitHub for upcoming features and browser implementation status
- **Three.js changelog** — tracking new renderer features, deprecations, and performance improvements
- **Livewire and Flux updates** — the ecosystem moves fast; monitoring releases for new patterns and breaking changes
- **Browser DevTools mastery** — Chrome and Safari DevTools performance panels; Paint Flashing, FPS meter, compositor layer visualization
- **Community** — CodePen challenges, CSS Weekly, Frontend Masters for deep-dives on specific techniques
- **Performance research** — web.dev, Chrome Developer blog for new Rendering Performance and Core Web Vitals guidance
