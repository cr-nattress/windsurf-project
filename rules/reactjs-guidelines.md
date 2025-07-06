# Windsurf IDE Rules: ReactJS Best Practices

## Component Structure
- Enforce PascalCase naming for React components.
- One component per file (default export).
- Organize imports: external libraries first, then internal.
- Components must start with hooks, then handlers, then return JSX.
- Require clear, semantic file names (no abbreviations).

## State Management
- Prefer local state (`useState`, `useReducer`) over global unless shared.
- Use Context for shared state; combine with `useReducer` for complex updates.
- Disallow redundant derived state (derive from props where possible).
- State updates must be immutable (no mutation of arrays/objects).

## Hooks Usage
- Enforce "Rules of Hooks" (top-level calls only, in components or hooks).
- Disallow using Hooks in conditions or loops.
- Custom hooks must start with `use`.
- Recommend `useCallback` and `useMemo` for memoization only when needed.
- Warn on unnecessary re-creation of functions or objects in JSX.

## Reusable Patterns
- Encourage use of composition via children or render props.
- Promote custom hooks for repeated logic.
- Avoid inheritance or class-based reuse.
- Prefer controlled components for forms.

## Error Handling
- Require use of Error Boundaries for critical render trees.
- Disallow throwing errors in event handlers without try/catch.
- Encourage fallback UIs for async and boundary errors.

## Folder and File Structure
- Use feature-based folder organization (`/feature/components/...`).
- Keep component styles and tests co-located with components.
- Avoid deep nesting of folders (>4 levels).
- Use relative paths with aliases or absolute imports.

## Testing
- Prefer `@testing-library/react` with user-centric queries (`getByRole`, etc.).
- Avoid testing implementation details.
- Disallow `act()` outside of RTL context.
- Encourage `userEvent` over `fireEvent`.

## Performance
- Recommend `React.memo` for pure functional components.
- Require use of `key` prop for list rendering.
- Suggest `useTransition` and `useDeferredValue` for large updates.
- Lazy-load non-critical routes/components via `React.lazy`.

## Accessibility
- Require semantic HTML (e.g., `<button>`, not `<div onClick>`).
- Form inputs must have associated labels.
- Enforce keyboard navigability (tabIndex, key handlers).
- Alt text is required for all images.
- Warn if ARIA roles/attributes misused or redundant.

## General
- All components must be functional (no class components).
- Prefer TypeScript or PropTypes for type safety.
- Enforce code formatting via Prettier + ESLint (with react/recommended, jsx-a11y).