---
description: "Web development standards — React, TypeScript, accessibility, performance, security, testing patterns"
applyTo: "**/*.{ts,tsx,jsx,css}"
---
# Web Development — Coding Standards

## React

### State Management
- `useState` → simple, local state
- `useReducer` → complex transitions, next-state-depends-on-previous
- Context → truly global UI state (theme, locale, auth) — NOT frequently-changing data
- TanStack Query → all server/async state (caching, deduplication, refetching). UI state still needs local state, context, or a lightweight store.
- External store (Zustand, Jotai) → only when Context performance is a measured problem

### Performance
- **With React Compiler** (`babel-plugin-react-compiler`): manual memoization is rarely needed — the compiler handles it
- **Without React Compiler**: use `useMemo`/`useCallback`/`React.memo` only when you've profiled and measured a problem. Don't cargo-cult.
- Code split routes with `React.lazy()` + `Suspense`
- Virtualize lists over 50 items (TanStack Virtual, react-window)
- `loading="lazy"` on below-fold images

### Anti-Patterns
- `useEffect` for event handlers → move logic into the handler directly
- Derived state in `useState` → compute during render instead
- `useEffect` + `fetch` → use TanStack Query
- `key={index}` on dynamic lists → use stable unique `key={item.id}`
- Prop drilling beyond 2-3 levels → use composition or Context

### React 19
- `ref` is a regular prop — `forwardRef` is deprecated
- `<Context value={...}>` replaces `<Context.Provider value={...}>`
- `use()` reads promises/context in render (can be conditional, unlike hooks)
- Actions: async functions in transitions that manage pending/error/optimistic states
- Document metadata (`<title>`, `<meta>`) renders anywhere — auto-hoisted to `<head>`

## TypeScript + React

- `interface` for component props and extendable shapes; `type` for unions and intersections
- Never use `any` — use `unknown` + type narrowing
- `React.ReactNode` for children (not `JSX.Element`)
- `React.ComponentPropsWithoutRef<'button'>` to extend native HTML element props
- `as const` for literal unions instead of enums
- Explicit return types on exported functions; inferred fine for internal
- Narrow errors in catch blocks: `if (error instanceof Error)` before accessing `.message`

## Accessibility (WCAG 2.2 AA Minimum)

- Use semantic HTML: `<button>` not `<div onClick>`, `<a>` for navigation, `<nav>/<main>/<header>/<footer>` for landmarks
- Every `<img>` needs `alt` text (or `alt=""` for decorative)
- Every form input needs a visible, associated `<label>`
- All interactive elements keyboard-reachable and operable — no keyboard traps
- Focus indicator visible on all interactive elements
- Color is not the sole means of conveying information
- Text contrast ≥ 4.5:1 (large text ≥ 3:1); UI component contrast ≥ 3:1
- Touch targets ≥ 24×24 CSS pixels (WCAG 2.2 addition)
- Dynamic content changes announced via `role="status"` or `aria-live="polite"`
- First rule of ARIA: don't use ARIA if native HTML can express the semantics
- React-specific: use `htmlFor` (not `for`) on labels, ensure router links have anchor semantics

## Security

- 🔴 **Never** use `dangerouslySetInnerHTML` without DOMPurify sanitization
- 🔴 **Never** put secrets/API keys in frontend code or `VITE_` env vars (they're in the bundle)
- 🔴 Reject `javascript:` protocol in `href`/`src` attributes
- Validate and sanitize all user input before rendering
- Use `HttpOnly` cookies for auth tokens, not localStorage
- Run `npm audit` regularly; don't add dependencies for trivial functionality

## Testing

### Query Priority (Testing Library — from official docs)
1. `getByRole` — top preference (reflects accessibility tree)
2. `getByLabelText` — best for form fields
3. `getByText` — for non-interactive content
4. `getByTestId` — **escape hatch only**

### What to Test
- ✅ User-visible behavior and outcomes
- ✅ Loading, error, and empty states
- ✅ Form submission and validation
- ❌ Implementation details (state variables, internal methods)
- ❌ Styling and CSS classes
- ❌ Snapshot tests

### Patterns
- Use `userEvent` over `fireEvent` for realistic interactions
- Mock APIs at network level (MSW), not module level
- Test error states alongside happy paths
- ~80% integration tests (component + hooks + API mocking), ~20% unit tests for pure logic

## Data Fetching (TanStack Query)

- Handle all three states: `isPending`, `isError`, `isSuccess` — loading/error check FIRST
- Query keys must include all variables the query function depends on
- Set `staleTime > 0` to prevent unnecessary refetches (default is 0)
- Use `placeholderData` for instant UI while loading
- Use `useMutation` with `onMutate` for optimistic updates
- Prefetch on hover/focus: `queryClient.prefetchQuery()`
- Don't use `useEffect` + `fetch` when TanStack Query should be used

## CSS / Tailwind

- Mobile-first responsive: base styles, then `sm:`, `md:`, `lg:` overrides
- Use Tailwind design tokens — avoid arbitrary values (`w-[137px]`) when a token exists
- Prefer `transform` and `opacity` for animations (GPU-composited)
- No inline styles (`style={{}}`) except for truly dynamic values
- No `!important` — fix specificity instead
- Extract to components when class lists exceed ~5 utilities

## Code Organization

- Feature-based colocation is the recommended default for React SPAs (Bulletproof React pattern). Framework-imposed structures (Next.js App Router) take precedence when applicable.
- Import direction: `shared → features → app`. Features never import from other features.
- Avoid barrel exports in Vite projects — import from specific files (hurts dev server performance)
- Naming: `PascalCase.tsx` for components, `use` prefix for hooks, `UPPER_SNAKE_CASE` for constants
- Colocate test files: `ComponentName.test.tsx` alongside the component

## Core Web Vitals Thresholds

| Metric | Target | Measures |
|--------|--------|----------|
| LCP | ≤ 2.5s | Loading (largest visible element) |
| INP | ≤ 200ms | Interactivity (input to paint) |
| CLS | ≤ 0.1 | Visual stability (layout shifts) |

> Full reference with anti-patterns catalog, sources, and detailed review checklist: `web-development.detail.instructions.md`
