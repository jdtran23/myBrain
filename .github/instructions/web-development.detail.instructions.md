---
description: "Web development deep reference — anti-patterns catalog, review checklist, accessibility spec, performance metrics, sources"
---
# Web Development — Detailed Reference

## Anti-Patterns Catalog

| # | Don't Do This | Do This Instead | Why |
|---|--------------|-----------------|-----|
| 1 | `<div onClick={handler}>` | `<button onClick={handler}>` | Div has no keyboard support, no role, no focus |
| 2 | `useEffect(() => { fetchData() }, [])` | `useQuery({ queryKey: ['data'], queryFn: fetchData })` | TanStack Query handles caching, deduplication, refetching |
| 3 | `const [fullName, setFullName] = useState(first + last)` | `const fullName = first + ' ' + last` | Derived state should be computed, not stored |
| 4 | `useMemo(() => items.length, [items])` | `items.length` | Trivial calculations don't need memoization |
| 5 | `useEffect(() => { handleClick logic }, [clicked])` | Move logic into the click handler | Effects synchronize with external systems, not events |
| 6 | `key={index}` on dynamic lists | `key={item.id}` | Index keys cause bugs on reorder/delete |
| 7 | `<img src={url} />` | `<img src={url} alt="Description" />` | Missing alt = invisible to screen readers |
| 8 | `dangerouslySetInnerHTML={{__html: userInput}}` | `dangerouslySetInnerHTML={{__html: DOMPurify.sanitize(userInput)}}` | Unsanitized HTML = XSS |
| 9 | Check data before loading state | Check loading/error FIRST, then data | Data may be stale; loading/error takes priority |
| 10 | Barrel exports (`export * from './Button'`) | Direct imports (`import { Button } from './Button'`) | Barrel exports hurt Vite dev server performance |
| 11 | `useCallback` without `memo()`-wrapped child | Plain function declaration | useCallback without matching memo() adds overhead for nothing |
| 12 | `<a href="javascript:void(0)">` | `<button onClick={...}>` | javascript: protocol is XSS risk |
| 13 | `catch (error) { setError(error.message) }` | `catch (error) { if (error instanceof Error) setError(error.message) }` | Must narrow unknown error type |
| 14 | `VITE_API_SECRET=secret` in .env | Move secret to backend; call via API proxy | VITE_ vars are inlined into the bundle |
| 15 | `style={{ width: '200px', color: '#3b82f6' }}` | `className="w-[200px] text-blue-500"` | Inline styles bypass design system, can't be responsive |

## Full Review Checklist for @absol

### Correctness
- [ ] All async operations handle loading, error, AND empty states
- [ ] Error boundaries wrap route-level components
- [ ] Key props use stable, unique identifiers (not array index)
- [ ] Effects have proper cleanup functions
- [ ] Dependency arrays are correct (no missing deps, no over-inclusion)
- [ ] State updates use functional form when depending on previous state

### TypeScript
- [ ] No `any` types (use `unknown` + narrowing)
- [ ] Component props properly typed (interface for objects)
- [ ] Event handlers explicitly typed
- [ ] Error handling narrows error type before accessing properties
- [ ] Strict null checks handled

### Accessibility
- [ ] Interactive elements are native HTML (`<button>`, `<a>`, `<input>`)
- [ ] All form inputs have visible, associated labels
- [ ] Images have meaningful alt text (or `alt=""` for decorative)
- [ ] Color is not the sole means of conveying information
- [ ] Focus management for modals, drawers, dynamic content
- [ ] Touch targets ≥ 24×24px
- [ ] Dynamic content changes announced via `aria-live`

### Performance
- [ ] Routes are code-split with `React.lazy()`
- [ ] Images use `loading="lazy"` below the fold
- [ ] Long lists (50+ items) use virtualization
- [ ] No unnecessary memoization (only when measured need)
- [ ] Bundle impact considered for new dependencies

### Security
- [ ] No `dangerouslySetInnerHTML` without sanitization
- [ ] No secrets/API keys in client code or VITE_ env vars
- [ ] No `javascript:` protocol in href/src
- [ ] User input validated before rendering

### Data Fetching
- [ ] Server state managed with TanStack Query (not useEffect + useState)
- [ ] Query keys include all variables the query function depends on
- [ ] Error and loading states handled for every query
- [ ] Mutations invalidate relevant queries

### Testing
- [ ] Tests query by role/label first (not testid)
- [ ] Tests verify behavior, not implementation
- [ ] Error states tested alongside happy paths

### Code Organization
- [ ] No cross-feature imports (features don't import from other features)
- [ ] No barrel file re-exports
- [ ] Naming follows conventions (PascalCase components, use-prefix hooks)
- [ ] Constants extracted for magic numbers/strings

## Severity Guide for @absol

| Category | Issue | Severity |
|----------|-------|----------|
| Error Handling | Missing error boundary around route boundaries | 🔴 |
| Error Handling | Missing loading/error states on async operations | 🔴 |
| Memory Leaks | Uncleared `setInterval`/`setTimeout` in effects | 🔴 |
| Memory Leaks | Missing cleanup for event listeners / subscriptions | 🔴 |
| Race Conditions | Async operations that don't handle component unmount | 🔴 |
| Race Conditions | Stale closure from incorrect dependency array | 🔴 |
| Lists | Missing or non-unique `key` prop | 🔴 |
| Security | `dangerouslySetInnerHTML` without sanitization | 🔴 |
| Security | Secrets/keys in client code | 🔴 |
| Accessibility | `<div onClick>` without keyboard support | 🔴 |
| Accessibility | Missing form labels | 🔴 |
| Accessibility | Images without alt text | 🟡 |
| Performance | No code-splitting on routes | 🟡 |
| Performance | Large lists without virtualization | 🟡 |
| Performance | Unnecessary memoization (cargo-culting) | 🔵 |
| Types | Usage of `any` | 🟡 |
| Types | Missing error type narrowing in catch | 🟡 |
| State | Derived state in `useState` | 🟡 |
| State | `useEffect` for event handler logic | 🟡 |
| State | `useEffect` + fetch instead of TanStack Query | 🟡 |
| Testing | `getByTestId` when `getByRole` is possible | 🟡 |
| Testing | Testing implementation details | 🟡 |
| Imports | Cross-feature imports | 🟡 |
| Imports | Barrel file re-exports in Vite | 🔵 |
| Hardcoding | Magic numbers/strings without constants | 🔵 |

## WCAG 2.2 AA Quick Reference

| Criterion | Requirement | Level |
|-----------|------------|-------|
| 1.1.1 | Non-text content has text alternative | A |
| 1.4.1 | Color not sole means of conveying info | A |
| 1.4.3 | Text contrast ≥ 4.5:1 (large text ≥ 3:1) | AA |
| 1.4.11 | UI component contrast ≥ 3:1 | AA |
| 2.1.1 | All functionality operable via keyboard | A |
| 2.1.2 | No keyboard traps | A |
| 2.4.7 | Focus visible on interactive elements | AA |
| 2.4.11 | Focus not obscured by sticky headers/modals (new in 2.2) | AA |
| 2.5.7 | Dragging operations have non-drag alternatives (new in 2.2) | AA |
| 2.5.8 | Touch targets ≥ 24×24px (new in 2.2) | AA |
| 3.1.1 | `<html lang="en">` set | A |
| 3.3.1 | Errors identified in text, not just color | A |
| 3.3.2 | Labels or instructions for input | A |
| 4.1.3 | Status messages via `role="status"` or `aria-live` | AA |

## TanStack Query Key Organization Pattern

```typescript
const cardKeys = {
  all: ['cards'] as const,
  lists: () => [...cardKeys.all, 'list'] as const,
  list: (filters: CardFilters) => [...cardKeys.lists(), filters] as const,
  details: () => [...cardKeys.all, 'detail'] as const,
  detail: (id: string) => [...cardKeys.details(), id] as const,
};
```

## Sources

| Source | URL | Date |
|--------|-----|------|
| React 19 Release | https://react.dev/blog/2024/12/05/react-19 | Dec 2024 |
| React `use()` API | https://react.dev/reference/react/use | 2024 |
| React Compiler | https://react.dev/learn/react-compiler | 2024 |
| TanStack Query Guides | https://tanstack.com/query/latest/docs/framework/react/guides/queries | 2026 |
| WCAG 2.2 Quick Reference | https://www.w3.org/WAI/WCAG22/quickref/ | Sep 2025 |
| Web.dev Core Web Vitals | https://web.dev/articles/vitals | Oct 2024 |
| Web.dev Optimize INP | https://web.dev/articles/optimize-inp | Sep 2025 |
| Testing Library Priority | https://testing-library.com/docs/queries/about | Mar 2024 |
| Bulletproof React | https://github.com/alan2207/bulletproof-react | 2023 |
| TkDodo Query Keys | https://tkdodo.eu/blog/effective-react-query-keys | 2022 |
| OWASP XSS Prevention | https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html | Reference |
| Vite Performance | https://vite.dev/guide/performance | 2026 |
