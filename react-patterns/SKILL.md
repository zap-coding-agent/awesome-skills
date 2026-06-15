---
name: react-patterns
description: Use when designing React components — choosing between hooks, compound components, render props, and composition patterns; avoiding prop drilling; managing shared state; or when a component is growing too large and needs splitting. Covers idiomatic React composition patterns.
---

# React Patterns

## Core Philosophy

React components compose. The patterns that age well are the ones that lean into composition over inheritance, explicit over implicit, and colocating state with its consumers. A component is good when it's easy to delete — the sign that it has one clear job and isn't secretly load-bearing for everything else.

## Custom Hooks — Extract Behavior, Not Just Logic

When a component has non-trivial state logic, extract it to a custom hook. The hook owns the state machine; the component owns the rendering:

```tsx
// ❌ component knows too much about the loading/error lifecycle
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  useEffect(() => {
    fetchUser(userId).then(setUser).catch(setError).finally(() => setLoading(false));
  }, [userId]);
  if (loading) return <Spinner />;
  if (error) return <Error />;
  return <div>{user.name}</div>;
}

// ✅ hook owns the lifecycle; component is pure rendering
function useUser(userId) {
  const [state, setState] = useState({ status: "loading" });
  useEffect(() => {
    setState({ status: "loading" });
    fetchUser(userId)
      .then(user => setState({ status: "success", user }))
      .catch(error => setState({ status: "error", error }));
  }, [userId]);
  return state;
}

function UserProfile({ userId }) {
  const state = useUser(userId);
  if (state.status === "loading") return <Spinner />;
  if (state.status === "error") return <Error />;
  return <div>{state.user.name}</div>;
}
```

A custom hook is just a function — it's testable, composable, and shareable without React Test Renderer.

## Compound Components — Co-located, Flexible API

When a component has sub-parts that need to share state without prop drilling:

```tsx
// Consumer declares structure; implementation handles state
<Select value={value} onChange={setValue}>
  <Select.Trigger>{value || "Pick one"}</Select.Trigger>
  <Select.Options>
    <Select.Option value="a">Option A</Select.Option>
    <Select.Option value="b">Option B</Select.Option>
  </Select.Options>
</Select>
```

The parent `Select` uses Context to pass state to `Trigger`, `Options`, and `Option` — they don't need props forwarded from the consumer. The consumer decides layout and structure.

**When to use:** any time you have a parent and tightly-related children that share state. Tabs, Select, Accordion, Dialog, Form.

## Controlled vs Uncontrolled — Choose Explicitly

| | Controlled | Uncontrolled |
|---|---|---|
| State lives | Parent/caller | Component itself |
| Flexibility | Full — parent decides | Self-contained |
| Use for | Form values you submit, filters | Quick prototypes, rich text editors |

For library components (design systems), support both via a pattern:

```tsx
function Toggle({ checked: controlledChecked, defaultChecked, onChange }) {
  const [internalChecked, setInternal] = useState(defaultChecked ?? false);
  const isControlled = controlledChecked !== undefined;
  const checked = isControlled ? controlledChecked : internalChecked;
  const toggle = () => {
    if (!isControlled) setInternal(c => !c);
    onChange?.(!checked);
  };
  return <button onClick={toggle} aria-pressed={checked}>{checked ? "On" : "Off"}</button>;
}
```

## Avoiding Prop Drilling: Context vs Composition

**Before reaching for Context, try composition:** pass the already-rendered element, not the data.

```tsx
// ❌ Prop drilling: user passed through 3 layers
<App user={user}><Layout user={user}><Header user={user} /></Layout></App>

// ✅ Composition: App renders Header with user already baked in
<App>
  <Layout header={<Header user={user} />} />
</App>
```

**Context is correct for:** theme, locale, auth state, anything that many components need and that changes rarely. Not for every piece of state.

## State Colocation — Lift Only What Needs Lifting

Don't put everything in global state. State should live as close to its consumers as possible:

```
ui state (hover, open/closed, tab index)   → local useState
form values before submit                  → local or form library
server data (user, products)               → React Query / SWR / server component
global app state (auth, cart, preferences) → Zustand / Jotai / Context
```

A component that manages its own state is easier to test, move, and delete than one dependent on global state.

## Key Patterns at a Glance

| Pattern | Use when |
|---|---|
| Custom hook | Component has logic that isn't rendering |
| Compound component | Tightly related sub-parts share state |
| Render prop | Consumers need full control of rendering while sharing logic |
| Controlled/Uncontrolled | Library component needs flexibility |
| Composition over context | Prop drilling through 1-2 components |
| State colocation | Any time you're tempted to lift state globally |

## Common Mistakes

| Mistake | Fix |
|---|---|
| `useEffect` for derived state | Compute it during render; `useMemo` if expensive |
| One giant component | Split on responsibility; custom hooks for behavior |
| Global state for local UI | Collocate; lift only when shared across distant tree nodes |
| `key={Math.random()}` | Stable keys from data; index only for static lists |
| Missing deps in `useEffect` | Use the eslint-plugin-react-hooks exhaustive-deps rule |
| `useEffect` + `useState` for async data | Use React Query, SWR, or RSC instead |

**REQUIRED COMPANION:** react-internals explains *why* these patterns work (Fiber, hooks internals). react-performance covers when and how to optimize. nextjs-app-router covers RSC, which replaces many data-fetching patterns.
