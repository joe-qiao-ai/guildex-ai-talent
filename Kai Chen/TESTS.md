# TESTS.md — Kai Chen

These tests are designed to fail a naive AI that echoes best-practice platitudes. Passing requires specific, defensible technical judgment.

---

## Test 1: The Bundle Optimization Trap

**Question:** We've added `React.memo` to every component in our app to improve performance. Bundle sizes and re-renders are still bad. What's going wrong?

**What a bad answer looks like:** "React.memo is great for performance optimization! Make sure you're also using useCallback and useMemo."

**Expected answer:**

`React.memo` only prevents re-renders when the parent re-renders — it does nothing for bundle size (it's a runtime concern, not a build concern). Wrapping every component in `React.memo` is actually a performance antipattern for several reasons:

1. **Comparison overhead** — `React.memo` runs a shallow comparison on every render cycle. For simple, cheap components, the comparison cost exceeds the render cost.
2. **It solves the wrong problem** — if a child is re-rendering unnecessarily, the root cause is usually that its parent's state is changing when it shouldn't be, or that callbacks are being recreated on each render. The fix is at the state/callback design level, not the component boundary.
3. **For bundle size** — you need code splitting, tree shaking, and dynamic imports. `React.memo` is invisible to the bundler.

The right approach: profile first with React DevTools Profiler to identify *which* components are causing expensive re-renders, and *why*. Then fix the cause, not the symptom.

---

## Test 2: The Accessibility Edge Case

**Question:** Our modal has `role="dialog"` and `aria-modal="true"`. A blind user reports they can still navigate to content behind the modal with their screen reader. Why?

**What a bad answer looks like:** "Make sure you add aria-labelledby and aria-describedby to your modal."

**Expected answer:**

`aria-modal="true"` is meant to signal to screen readers that they should restrict navigation to the modal's contents — but support is inconsistent. Several popular screen readers (particularly older NVDA versions and JAWS with certain browse mode settings) partially or fully ignore `aria-modal`. 

The robust solution is **focus trapping + `aria-hidden` on the background**:

```javascript
// When modal opens:
document.getElementById('app-root').setAttribute('aria-hidden', 'true');

// When modal closes:
document.getElementById('app-root').removeAttribute('aria-hidden');
```

Combined with a focus trap (every Tab/Shift+Tab cycles within the modal), and returning focus to the trigger element on close, this reliably prevents navigation to background content regardless of screen reader behavior.

`aria-modal` is a nice progressive enhancement but cannot be your only defense. The ARIA spec acknowledges this limitation.

---

## Test 3: The Core Web Vitals Diagnosis

**Question:** Our LCP is 5.2 seconds. Our server TTFB is 200ms and our CSS/JS are already code-split. What else should I check?

**What a bad answer looks like:** "Try using a CDN and compressing your assets."

**Expected answer:**

At a 5.2s LCP with fast TTFB and code-split assets, the most likely culprits are:

1. **LCP element is an image loaded lazily** — `loading="lazy"` on above-the-fold images is one of the most common LCP killers. The browser waits to discover lazy images until the page renders. If your LCP element (hero image, product image) has `loading="lazy"`, remove it or set `fetchpriority="high"`.

2. **LCP image is not preloaded** — Add a `<link rel="preload" as="image" href="...">` in the `<head>` for your LCP image. This starts the download before the browser finishes parsing.

3. **Render-blocking resources** — Even with code splitting, synchronous CSS in `<head>` blocks rendering. Check for `@import` in CSS files (which are render-blocking chains) and large third-party scripts loaded synchronously.

4. **LCP element discovered late via JavaScript** — If your hero content is rendered by a client-side React hydration step, the browser won't see the LCP element until JS executes. Consider SSR or SSG for above-the-fold content.

Run `exec: npx lighthouse --only-audits=lcp` and share the "LCP element" it identifies — that's the thread to pull.

---

## Test 4: The State Architecture Problem

**Question:** We have a shopping cart in React. The cart icon in the header shows item count. The cart drawer slides out from the right. The checkout page reads the cart. These are all separate route-level components. How would you structure this state?

**What a bad answer looks like:** "Use Redux for global state management since the cart is shared across components."

**Expected answer:**

This is a good use case for a lightweight global store (Zustand or Jotai), **not** Redux. Here's why and how:

The cart data meets the criteria for global state: it's needed by genuinely unrelated components (header, drawer, checkout page) at different routing levels, and changes need to propagate immediately across all consumers.

Zustand approach:
```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface CartItem { id: string; qty: number; price: number; }

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  total: () => number;
}

const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      addItem: (item) => set(state => ({
        items: [...state.items.filter(i => i.id !== item.id), item]
      })),
      removeItem: (id) => set(state => ({
        items: state.items.filter(i => i.id !== id)
      })),
      total: () => get().items.reduce((sum, i) => sum + i.price * i.qty, 0),
    }),
    { name: 'cart-storage' } // auto-persists to localStorage
  )
);
```

I'd also use TanStack Query alongside this for the checkout step — server-side cart validation before payment should not rely purely on local state.

Redux would add significant boilerplate for no additional benefit here. The cart is not complex enough to need DevTools time-travel debugging.

---

## Test 5: The Cross-Browser Compatibility Trap

**Question:** Our CSS Grid layout looks perfect in Chrome and Safari but breaks in Firefox. What's different?

**What a bad answer looks like:** "Use a CSS reset and make sure you add vendor prefixes."

**Expected answer:**

Vendor prefixes don't affect CSS Grid in any modern browser — Grid has had full unprefixed support across Chrome, Firefox, and Safari for years. The divergence you're seeing is almost certainly a behavioral difference, not a prefix issue.

The most common Firefox-specific Grid issues:

1. **`min-content` vs `auto` sizing** — Firefox and Chrome sometimes resolve `min-width: auto` on grid children differently. If a grid child contains text or images that can't shrink, Firefox may size the column differently than Chrome expects.

2. **Subgrid support** — Firefox was first to implement `subgrid` fully; Chrome and Safari came later. If you're using subgrid, check the MDN compatibility table.

3. **Gap behavior with `fr` units** — In some configurations with `min-content` tracks, Firefox calculates `fr` allocation differently.

4. **`overflow` on grid children** — Firefox is stricter about painting overflow outside grid cells.

The diagnostic step: open Firefox DevTools, go to the Layout panel, and enable Grid overlay. This shows you exactly how Firefox is calculating track sizes. Compare the track sizes with Chrome's equivalent. The discrepancy will point you to the specific property causing the difference.

Share your Grid CSS and I can identify the exact conflict.
