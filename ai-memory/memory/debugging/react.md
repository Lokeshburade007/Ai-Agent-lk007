# React Debugging Playbook

The mistakes that eat the most time.

## "Cannot read property X of undefined"

The data hasn't arrived yet. You're rendering before the query resolves:
```tsx
const { data } = useQuery(...);
return <div>{data.user.name}</div>;  // 💥
```

Fix: always handle the loading/empty branches:
```tsx
if (!data) return <Spinner />;
return <div>{data.user.name}</div>;
```

## Infinite re-render loop

Symptoms: page freezes, "Maximum update depth exceeded".

Causes:
- `useEffect` with a non-primitive dependency (`{}`, `[]`, new function) — recreated each render.
- `setState` called unconditionally in render body.
- `useEffect` setting state derived from another piece of state (use `useMemo` instead).

Fix: wrap the dep in `useMemo`/`useCallback`, OR move computation out of effect entirely.

## Stale closure — "my state is the old value"

```tsx
const [count, setCount] = useState(0);
useEffect(() => {
  const id = setInterval(() => setCount(count + 1), 1000); // always uses initial 0
  return () => clearInterval(id);
}, []);
```

Fix: functional update form:
```tsx
setCount(c => c + 1);
```

## Hydration mismatch (Next.js / SSR)

"Hydration failed because the initial UI does not match what was rendered on the server."

Causes:
- `Date.now()` / `Math.random()` in render — different on server vs client.
- Reading `window`/`localStorage` during render.
- Conditional rendering on `typeof window !== 'undefined'`.

Fix: render the divergent part in `useEffect`, or use `dynamic(() => ..., { ssr: false })` in Next.

## Items in a list flicker / lose state on update

`key={index}` on a list that can reorder or have items added in the middle. React thinks element at index 0 is "the same component" even though the data changed.

Fix: use a stable, unique ID:
```tsx
items.map(item => <Row key={item.id} item={item} />)
```

## State update not reflected on screen

- You mutated state instead of replacing it:
```tsx
state.items.push(x); setState(state);   // ❌ same reference, no re-render
setState({ ...state, items: [...state.items, x] });  // ✅
```
- You used `useRef` instead of `useState` and expected a re-render.

## Form input is laggy / shows old value

- Controlled input with state in a parent — every keystroke re-renders the whole subtree. Move state down or use `react-hook-form` (uncontrolled).
- Heavy work in render — wrap in `useMemo`.

## Query fires twice (React Query / TanStack)

- React Strict Mode in dev intentionally double-invokes effects. Production behavior is normal.
- Two components on screen mounted the same query — that's correct, it's deduped.
- Network tab shows different keys — `queryKey` isn't stable. Don't compute `queryKey` from objects without memoization.

## "Each child in a list should have a unique key"

You forgot `key` prop. Add it. Don't suppress the warning — it indicates real correctness issues.

## State persists between routes when it shouldn't

You stored state in a context that wraps the router. Move the provider inside the route, or reset state on route change with `useEffect`.

## Component renders too often — find the cause

1. React DevTools → Profiler → "Why did this render?"
2. Common: parent passes a new object/array/function literal as prop on every render.
3. Fix: `useMemo` for the object, `useCallback` for the function, or pass primitives.

## Tailwind classes don't apply

- Class name is dynamic (`bg-${color}-500`). Tailwind only sees literal strings — it doesn't generate dynamic classes. Use a switch or `safelist` in config.
- Build output doesn't include the source file. Check `content` paths in `tailwind.config.js`.
- Specificity: another stylesheet is overriding. Add `!important` only as a last resort; usually re-order CSS or fix the rule.

## Production build breaks but dev works

- Tree-shaking removed a side-effect import. Mark the package as having side effects in its own `package.json`.
- Environment variable referenced as `process.env.X` but not prefixed with `VITE_` / `NEXT_PUBLIC_`. The bundler strips it.
- Case-sensitive imports broken on Linux build server.
