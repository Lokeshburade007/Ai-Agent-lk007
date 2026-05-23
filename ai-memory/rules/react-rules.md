# React Rules

Load this when working on any React frontend (Vite, Next.js, CRA-replacement, MERN frontend).

## Always

- **Functional components + hooks only.** No class components in new code.
- **TypeScript** for new files. Migrate as you touch JS files (no big-bang migrations).
- **TanStack Query (React Query)** for any data that comes from a server. Never `useEffect(() => { fetch() }, [])` for primary loads — it produces stale data, race conditions, and missing error states.
- **react-hook-form + zod** for forms. Never `useState` for form fields beyond trivial cases.
- **Tailwind** for styling. Class order: layout → spacing → typography → color → state. Use `clsx` for conditionals.
- **Loading / empty / error / success** as four explicit states for every data-driven screen.
- **Co-locate** `Component.tsx`, `Component.test.tsx`, `Component.module.css` (if any), `index.ts` in the same folder.
- **Default exports for pages, named exports for components.** Pages (file-route) is the one place a default export is OK.

## Never

- ❌ Reach for Redux / Zustand / Jotai until prop drilling actually hurts (3+ levels). Then prefer **Context** for low-frequency state, **Zustand** for high-frequency.
- ❌ Compute derived state in `useState` + `useEffect`. Compute it inline or with `useMemo` if expensive.
- ❌ Use `useEffect` for anything except subscribing to external systems (timers, sockets, browser APIs).
- ❌ Inline styles (`style={{}}`) — that bypasses the design system. Use Tailwind or a styled-component variant.
- ❌ `dangerouslySetInnerHTML` without DOMPurify.
- ❌ Mutate state directly (`state.items.push(x)` then `setState(state)` — won't re-render reliably).
- ❌ `key={index}` for lists where items can reorder/be deleted. Use a stable ID.
- ❌ Components longer than ~150 lines. Split them.

## File layout

```
src/
  app/                  # routes (Next.js) or page roots (Vite)
  components/           # shared, presentational
    Button/
      Button.tsx
      Button.test.tsx
      index.ts
  features/             # vertical slices: ui + hooks + types per feature
    auth/
      LoginForm.tsx
      useLogin.ts
      schema.ts
  hooks/                # cross-feature custom hooks
  lib/                  # framework-agnostic utilities (date, currency, etc.)
  api/                  # API clients (axios/ky/fetch wrappers + types)
  styles/               # global CSS, tailwind config
```

## Forms (the right way)

```tsx
const schema = z.object({ email: z.string().email(), password: z.string().min(8) });
type FormValues = z.infer<typeof schema>;

const { register, handleSubmit, formState: { errors, isSubmitting } } =
  useForm<FormValues>({ resolver: zodResolver(schema) });

const onSubmit = async (data: FormValues) => { /* call mutation */ };

return <form onSubmit={handleSubmit(onSubmit)}>...</form>;
```

## Server state (the right way)

```tsx
const { data, isLoading, isError, error } = useQuery({
  queryKey: ['user', id],
  queryFn: () => api.getUser(id),
});

if (isLoading) return <Spinner />;
if (isError) return <ErrorBanner error={error} />;
if (!data) return <EmptyState />;
return <UserCard user={data} />;
```

## Accessibility — non-negotiable

- Every form input has a `<label htmlFor>`.
- Buttons say what they do (`Save changes` not `Submit`).
- Color contrast WCAG AA minimum.
- Focus visible on all interactive elements.
- Keyboard navigable: Tab, Enter, Esc all work.
- Semantic HTML: `<button>` not `<div onClick>`, `<nav>`, `<main>`, `<header>`.

## Testing

- **Vitest + React Testing Library.** Test from the user's perspective: "user types into email, clicks submit, sees success".
- Don't test implementation details (don't assert on internal state, props passed to children).
- Mock the API layer, not React Query / hooks.
