---
name: react-performance
description: Use when a React app has rendering performance problems — excessive re-renders, jank, slow list rendering, large bundle size, or slow initial load. Covers profiling first, then targeted fixes: memo/useMemo/useCallback, virtualization, code splitting, Suspense, and avoiding premature optimization.
---

# React Performance

## Core Principle: Profile Before Optimizing

> "Premature optimization is the root of all evil." — Knuth (applies doubly to React)

React's default rendering is fast enough for most UIs. `React.memo`, `useMemo`, and `useCallback` have their own cost — comparing props on every render. Adding them speculatively slows code down and makes it harder to read. **Use the profiler first; apply the fix to the measured bottleneck.**

**Profiling toolchain:**
- React DevTools Profiler → flame chart of render time per component, highlighted by re-render count.
- Chrome Performance tab → scripting time, layout/paint, long tasks.
- `why-did-you-render` library → logs which props/state triggered a render.

## Re-render Causes (understand before fixing)

A component re-renders when:
1. Its own state changes (expected, correct).
2. Its parent re-renders AND it doesn't bail out (default behavior).
3. A Context value it subscribes to changes.

**The goal is not zero re-renders. The goal is no *unnecessary* re-renders.** A component with cheap render logic doesn't need `React.memo` — the comparison overhead may exceed the render cost.

## React.memo — Bail Out of Parent Re-renders

```tsx
// Only re-renders when its own props change (shallow compare)
const ExpensiveChart = React.memo(({ data, onHover }) => {
  return <HeavyD3Chart data={data} onHover={onHover} />;
});

// ✅ stable reference for the callback — otherwise memo is useless
function Dashboard() {
  const [selection, setSelection] = useState(null);
  const handleHover = useCallback((d) => setSelection(d), []);  // stable ref
  return <ExpensiveChart data={processedData} onHover={handleHover} />;
}
```

`React.memo` does a shallow comparison of props. If a prop is a new object/array/function literal on every render, memo never bails out — you need `useMemo`/`useCallback` to stabilize those references.

## useMemo — Stabilize Expensive Computations

```tsx
// ❌ recomputes on every render
const sorted = items.sort((a,b) => a.score - b.score);

// ✅ recomputes only when items changes
const sorted = useMemo(() => 
  [...items].sort((a,b) => a.score - b.score),
  [items]
);
```

Only wrap in `useMemo` when:
- The computation is genuinely expensive (>1ms measured).
- The result is passed to a `memo`-wrapped child as a prop.
- The result is used as a `useEffect` dependency to prevent infinite loops.

Don't `useMemo` everything — you pay memory + comparison overhead on every render.

## useCallback — Stable Function References

`useCallback(fn, deps)` = `useMemo(() => fn, deps)`. Useful for:
- Callback props to `memo`-wrapped children.
- Callbacks in `useEffect` deps arrays.

```tsx
// ✅ stable ref; ExpensiveList won't re-render on parent re-renders
const handleClick = useCallback((id) => dispatch({ type: "select", id }), [dispatch]);
```

## Virtualization — Long Lists

Rendering 10,000 rows is slow not because of React — it's because the browser must lay out and paint 10,000 DOM nodes. Virtualization renders only what's visible:

- `@tanstack/react-virtual` — headless, flexible, high-perf.
- `react-window` — simpler API for fixed-height rows.

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);
  const virtualizer = useVirtualizer({
    count: items.length, getScrollElement: () => parentRef.current,
    estimateSize: () => 48,  // row height estimate
  });
  return (
    <div ref={parentRef} style={{ height: 600, overflow: "auto" }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(vRow => (
          <div key={vRow.key} style={{ transform: `translateY(${vRow.start}px)` }}>
            <Row item={items[vRow.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Code Splitting & Lazy Loading

Ship only what the current route needs:

```tsx
const HeavyEditor = lazy(() => import("./HeavyEditor"));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      {showEditor && <HeavyEditor />}
    </Suspense>
  );
}
```

Route-based splitting is the highest ROI: split at route boundaries so each page loads only its code. Component-level splitting (editors, modals, charts) is second.

## Context Performance — Avoid Unnecessary Re-renders

Every component that calls `useContext(Ctx)` re-renders when the context value changes — even if it only uses one field. Fix with:

```tsx
// Split contexts by update frequency
const ThemeContext    = createContext(null);   // changes rarely
const UserContext     = createContext(null);   // changes on auth
const CartItemsContext = createContext(null);  // changes often

// Or use a state management lib with selector-based subscriptions (Zustand, Jotai)
const count = useCartStore(state => state.count);  // re-renders only when count changes
```

## Quick Reference

| Problem | Fix |
|---|---|
| Child re-renders on every parent render | `React.memo` + stable props |
| New function/object on every render | `useCallback` / `useMemo` |
| Expensive derivation | `useMemo` |
| 1000+ list rows slow | Virtualization |
| Large initial bundle | Route-level `lazy` + `Suspense` |
| Context triggers too many re-renders | Split context by frequency; use selectors |
| Waterfall of loading states | Lift Suspense boundary up; prefetch |

**REQUIRED COMPANION:** Always profile with React DevTools before applying these fixes. react-patterns covers component architecture that avoids many performance problems at design time. nextjs-app-router moves data-fetching to the server, eliminating many client-side performance concerns.
