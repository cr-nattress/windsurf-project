ReactJS Best Practices and Patterns for Windsurf Cascade Agent
Component Structure and Naming Conventions
Keeping components well-structured and consistently named makes your codebase easier to navigate and maintain. Modern React favors functional components for their simplicity and use of Hooks (over class components)
telerik.com
. Follow these conventions for clarity:
Component Naming: Use PascalCase for React component names (e.g. UserProfile), and file names that match the component. Variables and functions inside can use camelCase. Use self-explanatory names that describe the component’s purpose
uxpin.com
uxpin.com
. For example, prefer UserList over Ulst.
Folder and File Naming: Choose a consistent naming scheme. Many teams use PascalCase or kebab-case for filenames (e.g. user-profile.js or UserProfile.js) – the key is consistency
uxpin.com
. Directories should also be named clearly (e.g. components, utils, hooks).
Component File Structure: Organize the content of a component file in a logical order:
Imports (React, libraries, then local modules).
Component Definition: Define state with Hooks (if needed) at the top, then any helper functions or event handlers, then the return JSX.
Export: Export the component (default export for single component per file).
Keeping this order (state -> handlers -> JSX) makes components predictable.
One Component per File: Generally aim for one React component per file. This encourages focus and reusability. If you have very small sub-components only used by a parent, you can nest them in the same file or a subfolder for that parent.
JSX Formatting: Use proper indentation and readable JSX. Wrap multiline JSX in parentheses and fragment (<>...</>) if needed. This makes the return statement cleaner. Also, include prop types or TypeScript interfaces for props if using those for documentation.
Example – A simple functional component with proper structure:
// File: src/components/PrimaryButton.jsx

import React from 'react';

function PrimaryButton({ label, onClick }) {
  // Event handler defined outside of JSX to avoid re-creating on every render
  const handleClick = () => {
    onClick?.();
  };

  // JSX return: descriptive text and proper semantic element
  return <button className="btn-primary" onClick={handleClick}>{label}</button>;
}

export default PrimaryButton;
In the example above, the component PrimaryButton is defined with PascalCase naming, the file name matches the component, and the internal structure is clear. The event handler is declared outside the JSX (preventing unnecessary re-creations each render), and we use a semantic <button> element with a clear class name.
State Management (Local and Global)
Effective state management is crucial for React applications. Local state is state confined to a single component (via useState or useReducer), while global state is shared across components (via Context or external libraries). Follow these best practices:
Keep State Local When Possible: Manage state in the closest component that needs it, and lift it up only when necessary. This makes components simpler and avoids unnecessary global complexity. Studies have found that apps which kept non-critical data in local component state saw improved responsiveness compared to those pushing everything into a global store
moldstud.com
. In practice, this means avoid global state for something that only a couple of components use.
Lifting State Up: If two sibling components need to share state, lift that state to their parent and pass it down via props. This localizes shared state at a common ancestor without introducing global state, following React’s data flow.
Context for Global State: For data that truly needs to be global (like user authentication info, theme, or locale), use the React Context API. Context provides a way to pass values deeply without prop drilling. Instead of threading props through many layers, wrap your app (or a subtree) with a context provider and consume it via useContext
uxpin.com
. This simplifies state sharing across distant components. However, be mindful that any context update will re-render all consuming components – so avoid putting extremely frequently-changing values in a single context.
useReducer with Context: For more complex global state logic, combine Context with useReducer. Define a reducer for state transitions and provide the [state, dispatch] via context. This pattern centralizes logic (like a mini Redux) and avoids excessive prop drilling
moldstud.com
. It’s great for situations where multiple components need to both read and update shared state (e.g. a global cart in an e-commerce app).
External State Libraries: If your app grows large, consider dedicated state management libraries. Redux (predictable single-store state), MobX (observable-based state), Zustand, Recoil, etc., can handle complex global state with advanced capabilities. Use these when the built-in context/state becomes hard to scale. For example, Redux remains popular for large apps due to its predictable state updates and debugging tools
moldstud.com
. Choose a solution that fits your team’s familiarity and the app’s complexity.
Immutable Updates: Always update state immutably. Never mutate state variables directly – instead, use setState functions to produce new arrays/objects. This ensures React can properly detect changes and update UI. Utilities like the spread operator or immer library can help with producing new copies of state.
Avoid Redundant State: Derive state from props or other state when possible, rather than duplicating it. Keeping a single source of truth prevents inconsistencies. For example, instead of keeping separate firstName and fullName state, keep firstName and derive fullName in render or via a memoized selector.
Global State with Context Example:
import React, { createContext, useContext, useState } from 'react';

// Create a Context for theme (light/dark)
const ThemeContext = createContext();

// Provider component to wrap the app (or part of it)
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");
  const toggleTheme = () => setTheme(prev => prev === "light" ? "dark" : "light");

  const value = { theme, toggleTheme };
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Consumer hook for ease of use
const useTheme = () => useContext(ThemeContext);

// Example component consuming global theme state
function ThemedButton() {
  const { theme, toggleTheme } = useTheme();
  return (
    <button onClick={toggleTheme} className={`btn-${theme}`}>
      Current theme: {theme}
    </button>
  );
}

// Usage in app root
function App() {
  return (
    <ThemeProvider>
      <Toolbar />
      <ThemedButton />
    </ThemeProvider>
  );
}
In this example, ThemeProvider holds a global theme state and provides a toggler. Any component can get the current theme and toggleTheme without drilling props. This pattern is suitable for app-wide settings. For more complex scenarios, useReducer could replace useState to manage state transitions (e.g., multiple actions on a global store) – this combination significantly reduces prop drilling in deeply nested components
moldstud.com
.
Reusable Component Patterns
Reusable components and patterns help avoid duplication and keep your code DRY. React encourages composition over inheritance for reuse – build small, focused components and combine them to create more complex UIs
legacy.reactjs.org
. Key patterns and techniques include:
Composition (Children Props): Rather than subclassing components, pass React elements as children or props to create flexible wrappers. For example, a generic <Card> component can accept arbitrary children content. This allows you to compose UIs by nesting components:
function Card({ children }) {
  return <div className="card">{children}</div>;
}
// Using composition:
function WelcomeMessage() {
  return (
    <Card>
      <h1>Welcome</h1>
      <p>Thanks for visiting our app!</p>
    </Card>
  );
}
The <Card> doesn’t need to know what it contains; it simply renders children. This is preferred over trying to create base classes. React’s powerful composition model means you can solve reuse cases by combining components instead of inheritance
legacy.reactjs.org
.
Specialization via Props: A variation of composition is passing components or render functions as props (sometimes called Render Props pattern). For example, a <Modal> component might accept a prop like renderFooter which is a function returning JSX for the footer. This way, the Modal is generic but you can inject custom content. This pattern was more common pre-Hooks, but you may still see it.
Higher-Order Components (HOC): An HOC is a function that takes a component and returns a new component, adding additional props or behavior. This pattern can be used for cross-cutting concerns (like logging, error handling, or context injection). For example, withUser(ProfileComponent) could inject user data into a Profile. With Hooks, HOCs are less needed (since you can use context or hooks directly in a component), but they are still useful in some cases. If using HOCs that pass refs, use React.forwardRef inside the HOC to forward the ref to the wrapped component
uxpin.com
. Always name HOCs clearly (e.g. withAuth) and document what they add.
Custom Hooks for Reusable Logic: Hooks allow extracting logic into reusable functions. Create custom hooks (functions starting with use) to share behavior between components without duplicating code. For example, a useFormInput hook can handle form input state and onChange logic (see Hooks section below). Custom hooks let components remain focused on rendering while the hook manages the stateful logic, and you can use the hook in multiple components
telerik.com
. This promotes separation of concerns and makes testing easier (you can test the hook’s logic independently
telerik.com
). Always prefix custom hooks with "use" so that the Rules of Hooks apply.
Presentational and Container Components: A classic pattern is splitting components into presentational (UI only, no business logic, often just receive props) and container (logic/state, passes data to presentational). For example, a CommentList container might fetch comments and pass them to a CommentListView presentational component for rendering. This separation can clarify what each piece does and improve reusability of the presentational parts (which could be used with different data sources). With Hooks, you can often achieve a similar separation by using custom hooks for data fetching and keeping components mostly for rendering.
Controlled vs Uncontrolled Components: For form inputs and similar elements, decide whether the component manages its own state (uncontrolled, using refs or internal state) or receives value and onChange from parent (controlled). As a reusable pattern, controlled components are more flexible – they allow parent components or state management to control them. Use controlled components for forms in larger apps (so you can validate or manipulate input centrally), but use uncontrolled with refs for simple cases or third-party integration.
Utility Components (Fragments & Portals): Use React Fragments (<>...</>) to group multiple children without extra DOM nodes – this is a small pattern that keeps the DOM clean
uxpin.com
. Use Portals for reusing components that need to render outside the normal hierarchy (e.g. modals or tooltips to render at the document body).
Remember, reusable doesn’t always mean “global” – it often means reusable within different contexts of your app. Aim to write components that are decoupled (e.g., a Button that just accepts props like onClick and children, rather than assuming global state inside). Use props to parameterize behavior and appearance, and composition to allow insertion of child content. Avoid premature abstraction: only generalize a component once you see it used in a few places with similar structure.
Hooks Usage (Custom and Built-In)
React Hooks (available since React 16.8) are essential for modern functional components. They allow you to use state and other React features without classes. Best practices for using Hooks include following the rules, leveraging built-in hooks appropriately, and creating custom hooks for reusable logic.
Follow the Rules of Hooks: Hooks must be called at the top level of your component or custom hook – never inside loops, conditions, or nested functions
react.dev
. This ensures Hooks are always called in the same order on every render. Only call Hooks from React function components or other custom hooks (not normal JS functions)
react.dev
. The official ESLint plugin (eslint-plugin-react-hooks) can enforce these rules to avoid mistakes.
Common Built-in Hooks: Familiarize yourself with the standard hooks and use them for their intended purposes:
useState – for local state. Initialize with a value, and it returns a state variable and setter. Use multiple useState calls for independent pieces of state instead of one complex object when possible.
useEffect – for side effects (data fetching, subscriptions, timers, logging, etc.). It runs after render and optionally cleanup on unmount or dependency change. Always specify a dependency array to avoid unintended repeated effects. If an effect doesn’t need to run on every render, include only the necessary dependencies (or an empty array for run-on-mount).
useContext – to consume a value from a React Context provider. This avoids prop drilling and directly accesses global or shared data.
useReducer – for complex state or state that involves numerous sub-values or transitions. It’s an alternative to useState, taking a reducer function and initial state. Great for when the next state depends on the previous state, or to encapsulate state update logic.
useRef – for mutable values that persist across renders (like an instance variable) or to hold DOM references. useRef is commonly used to reference a DOM element (e.g., ref={inputRef} on an <input> to focus it imperatively) or to keep any mutable value without causing re-renders when it changes.
useMemo – to memoize expensive calculations and avoid recomputation on every render. Provide a function and dependency list; it will recompute only when dependencies change
uxpin.com
. Use this for performance optimization of derived values.
useCallback – to memoize callback functions. It returns the same function instance between renders as long as dependencies haven’t changed. This is useful to prevent unnecessary re-rendering of child components that depend on a callback prop (especially when those children are wrapped in React.memo). Only use when needed (overuse can complicate code); primarily when passing callbacks deep or to optimized components.
useLayoutEffect – similar to useEffect but runs synchronously after DOM mutations. Use this only for measurements or DOM manipulations that must happen before the browser paints (e.g., reading layout, adjusting scroll). In most cases, useEffect is sufficient.
useImperativeHandle – used with React.forwardRef to customize the instance value that is exposed to parent via ref. Only needed when building components that need to expose a custom imperative API.
useDebugValue – a hook for custom hooks to label their output in React DevTools. This is for debugging and not commonly used in application code.
Custom Hooks: As mentioned, custom hooks are a powerful way to reuse logic. If you find a pattern of using several hooks together for a specific purpose in multiple components, consider extracting them into a custom hook. For example, a useOnlineStatus hook could subscribe to a browser online/offline event and return a boolean. Custom hooks can call other hooks (following the same Rules) and can return anything (value, functions, even JSX). This makes your component code cleaner (encapsulating logic in the hook) and more testable
telerik.com
. When writing custom hooks:
Name them with use... so it’s clear they are hooks.
Parameterize them as needed (e.g., function useFetch(url) { ... } can take an URL and handle fetching).
Optionally, return state and updater functions if the hook manages state (e.g., const [value, setValue] = useMyHook()).
Document their behavior and any expected conventions.
Effect Cleanup: Always clean up effects that create subscriptions or timers. Return a cleanup function from useEffect to avoid memory leaks (for example, remove event listeners or clear intervals on unmount or before next effect run).
Dependency Arrays: Be mindful of what you include in dependency arrays for useEffect, useMemo, and useCallback. Include all values that are used inside the hook callback and that come from props or state. This ensures consistency. If including something causes unnecessary calls (e.g., a function that changes on every render), consider wrapping that function in useCallback or extract the code in a way that doesn’t require dependency or disable the exhaustive-deps rule only as a last resort (with a comment explaining why).
Concurrent Features (React 18+): React 18 introduced hooks to better handle transitions and deferred updates:
useTransition – allows marking state updates as non-urgent (transitions). It returns [isPending, startTransition]. You wrap the less urgent state updates in startTransition(() => {...}). This tells React these updates can be deferred so more urgent renders (like user input feedback) can update first. Using transitions can prevent blocking the UI on large renders – it prioritizes what’s urgent. For instance, updating a filter list might be done in a transition so that a typing indicator or input box can update immediately without lag
vercel.com
vercel.com
.
useDeferredValue – allows you to take a value that may be changing quickly and defer its update. It returns a deferred version of a value that will lag behind the original if the system is busy. This is useful for throttling expensive re-renders. For example, you can defer a search query value so that as the user types quickly, your heavy search results component doesn’t re-render on every keystroke, only when idle.
These concurrent hooks improve perceived performance by keeping the interface responsive. Use them for expensive operations triggered by interactions – for example, wrapping a large state update (like setting a big list) in startTransition to avoid locking the UI, or using useDeferredValue for debouncing a live-search input.
Example – Custom Hook for Form Input:
// A custom hook to manage form field state with a reset capability
import { useState } from "react";
function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);
  const onChange = e => setValue(e.target.value);
  const reset = () => setValue(initialValue);
  return { value, onChange, reset };
}

// Usage in a component
function LoginForm() {
  const username = useFormInput("");
  const password = useFormInput("");

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Logging in with", username.value, password.value);
    // ...perform login logic
    username.reset();
    password.reset();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" placeholder="Username" {...username} />
      <input type="password" placeholder="Password" {...password} />
      <button type="submit">Login</button>
    </form>
  );
}
In this example, useFormInput encapsulates the state and logic for form inputs. It returns an object with value and onChange that we can spread into an <input>. This pattern eliminates repetitive code for multiple inputs and keeps the component logic clean. The hook also provides a reset function to clear the field, demonstrating how custom hooks can offer convenient utilities. The cascade coding agent can learn this pattern to apply consistent form handling logic across the codebase.
Code Organization and Folder Structure
A well-organized project structure makes development and maintenance easier. React doesn’t enforce a specific structure, but here are best practices:
Group by Feature (or Route): Many projects organize files by feature area. All files for a feature (components, styles, tests, sub-components) live in one folder
legacy.reactjs.org
. For example, you might have folders like HomePage/, ProfilePage/, Dashboard/, each containing the React components, CSS/SCSS, and tests for that feature. This way, when working on “Profile” you find everything in one place. The definition of a “feature” can be a page, a section of the app, or a domain concept. This approach improves colocation, meaning files that change together are located together
legacy.reactjs.org
.
Group by File Type: Another approach is grouping by the type of file
legacy.reactjs.org
. For example, a components/ directory for all components, a styles/ directory for CSS, a tests/ directory for test files, etc. Within components, you might further separate presentational vs container, or by common vs specific components. This can work well for smaller projects, but be cautious: too much separation can scatter related code. Often, large projects adopt a mix – grouping by feature for major sections, but also having common directories (like a common/ or shared/ components folder for reusable components across features).
Avoid Deep Nesting: Limit folder nesting to a reasonable level (e.g., 3-4 levels deep maximum)
legacy.reactjs.org
. Deeply nested structures lead to long relative import paths (../../../Component) which are hard to maintain. If you find yourself nesting many layers of folders, consider flattening or reorganizing by feature.
Name Folders after Components (if grouping by feature): If a feature has multiple components, you might create a folder and inside it have index.js (which could export the main component by default) and other related components or utilities. Alternatively, some prefer each component has its own folder with the component file and its CSS/test files together.
Colocation: Place files close to where they are used. If a component has a specific stylesheet or test file, keep them alongside the component. For example, you might have:
Button.jsx
Button.test.jsx
Button.module.css (styles specific to Button)
in the same folder. Colocation reduces friction in finding and updating related code.
Separate Configuration and Scripts: Keep non-react logic separate (e.g., utility functions in a utils/ folder, API calls in a services/ or api/ folder). This separation of concerns makes it clear what’s UI (components) vs. business logic or data fetching.
Consistency is Key: Whichever structure you choose, ensure the whole team follows it. Document the structure in a README or wiki. Consistency allows the cascade agent (and developers) to predict where to find things.
Here’s an example of a feature-based folder structure:
src/
├── components/
│   ├── common/
│   │   ├── Avatar.jsx
│   │   ├── Avatar.css
│   │   └── Avatar.test.jsx
│   ├── feed/              # "Feed" feature folder
│   │   ├── Feed.jsx
│   │   ├── Feed.css
│   │   ├── FeedStory.jsx
│   │   └── FeedStory.test.jsx
│   └── profile/           # "Profile" feature folder
│       ├── Profile.jsx
│       ├── ProfileHeader.jsx
│       ├── ProfileHeader.css
│       └── ProfileAPI.js
├── hooks/
│   └── useAuth.js
├── utils/
│   └── formatDate.js
└── App.jsx
In this hypothetical structure, the feed and profile folders encapsulate those features’ components and tests. A common folder holds shared components (like Avatar) used across features. We also have dedicated folders for hooks and utilities. Notice the relatively shallow nesting. This is just one example – adjust based on your app’s needs, but keep it logical and maintainable. The official React docs suggest not overthinking structure at project start; it can evolve as you discover patterns in file usage
legacy.reactjs.org
.
Error Handling and Boundary Patterns
Robust React applications anticipate and handle errors gracefully. This includes runtime errors during rendering as well as asynchronous errors (like network failures). Key techniques include Error Boundaries and proper try-catch handling:
Error Boundaries: An Error Boundary is a React component that catches JavaScript errors anywhere in its child component tree and displays a fallback UI instead of crashing the whole app. In React 16+, error boundaries catch errors during rendering, lifecycle methods, and in constructors of child components. They do not catch errors in event handlers or asynchronous code (e.g., setTimeout, fetch promises) – those need separate handling. Use error boundaries to prevent a minor part of the UI from breaking the entire app
uxpin.com
. Typically, you create a class component with static getDerivedStateFromError and/or componentDidCatch to handle the error:
import React from "react";
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  static getDerivedStateFromError(error) {
    // Update state so the next render shows fallback UI
    return { hasError: true };
  }
  componentDidCatch(error, info) {
    // Log error details, e.g., to an external logging service
    console.error("ErrorBoundary caught an error", error, info);
  }
  render() {
    if (this.state.hasError) {
      // Fallback UI when an error is caught
      return <h2>Something went wrong.</h2>;
    }
    // Otherwise, render children normally
    return this.props.children;
  }
}
export default ErrorBoundary;
You would wrap this around parts of your app. For example, if ProfilePage might crash, render it as <ErrorBoundary><ProfilePage/></ErrorBoundary>. That way, if an error occurs in ProfilePage or its children, the ErrorBoundary catches it and shows a friendly message instead of a blank screen
refine.dev
. It effectively acts like a try-catch for React render errors, safeguarding your application from unexpected crashes and preserving the user experience
uxpin.com
.
Placement of Error Boundaries: You can have a top-level error boundary (to catch anything in the entire app) and more granular ones for specific sections. Granular boundaries ensure that if (for example) a widget in a sidebar crashes, the rest of the page can still work. Design your fallback UI to be helpful – e.g., “Something went wrong – please try again later.” Possibly provide a reload button or report mechanism.
Handling Event Handler Errors: Errors inside event handlers (like an onClick) do not automatically propagate to error boundaries. You must handle these with try/catch inside the handler if needed. For instance:
function handleSave() {
  try {
    // ...attempt some action that might throw
  } catch (err) {
    console.error("Failed to save:", err);
    setErrorMessage("Save failed!");
  }
}
In the above, we catch the error and set some component state to display an error message. This keeps the error from causing a full app crash. Similarly, for async operations (promises), attach a .catch() to handle rejections.
Asynchronous Errors and Rejections: If you have asynchronous code (like data fetching with fetch or Axios), use .catch to handle promise rejections or wrap the logic in an async function inside a try/catch. Update state to reflect the error (e.g., set an error state variable and display it). This way users get feedback instead of a silent failure.
Logging and Monitoring: In componentDidCatch, or the catch blocks of your async calls, log the error details. Use console.error during development, and consider integrating a logging service (like Sentry or LogRocket) for production to record errors and user context. This helps track down issues.
Third-Party Error Boundary Components: The community has packages like react-error-boundary (by Brian Vaughn) which provide a ready-to-use ErrorBoundary component with extras (like reset capability). You can use these to avoid writing your own class component. For example, react-error-boundary lets you wrap components and provides an onReset prop to reset the error state easily.
Error Boundaries and Hooks: Currently, only class components can be error boundaries (as they rely on lifecycle methods). There is no built-in hook for error boundary behavior. However, you can create a wrapper component (class) or use the aforementioned library in your functional component apps when you need an error boundary.
Example – Using an Error Boundary:
<ErrorBoundary>
  <Dashboard />   {/* Dashboard and its children errors will be caught */}
</ErrorBoundary>
If an error in Dashboard occurs (e.g., trying to access an undefined property), the ErrorBoundary will catch it and render the fallback UI (<h2>Something went wrong.</h2> as coded above) instead of breaking the entire app. The console will show "ErrorBoundary caught an error" with details, thanks to our logging in componentDidCatch. Remember that error boundaries do not catch: errors thrown in event handlers, async code, server-side rendering, or errors inside the error boundary itself
refine.dev
refine.dev
. You must handle those separately. For event handlers, it’s safe to let errors throw because React will still only affect that event (not the render process), but you should provide user feedback. For async, always handle rejections. By combining error boundaries and diligent try/catch handling, you create an app that fails gracefully. Users see a helpful message or an intact remainder of the UI instead of a broken page, and you can guide the cascade agent on how to pattern-match such robust error handling across the codebase.
Testing Best Practices
Testing is vital to ensure your components and logic work as intended and continue to work as the code evolves. Modern React testing often uses Jest as a test runner and React Testing Library (RTL) for unit/integration tests of components. Here are best practices for testing React:
Test Behavior, Not Implementation: Write tests from the user’s perspective. This means asserting what the user would see or experience, rather than internal component state or calls. React Testing Library encourages this by letting you query the DOM in ways similar to how a user would (by text content, roles, labels, etc.)
medium.com
medium.com
. For example, instead of checking that state.isOpen is true after a click (implementation detail), test that the modal element is now visible on screen. This makes tests more robust to refactoring – you can change how the component works internally and the test still passes as long as the user-facing behavior remains correct.
Use Accessible Queries: Prefer RTL queries that correspond to how users identify elements:
getByRole – find elements by ARIA role (buttons, headings, textbox, etc.). You can also filter by name (accessible name).
getByText – for finding text content in the render.
getByLabelText – for form fields associated with <label> text.
getByPlaceholderText, getByAltText, etc., for placeholders and image alt text.
These queries ensure you are selecting elements meaningfully. Avoid overly specific queries like getByTestId unless necessary – test IDs are fine as a fallback, but they tie the test to the implementation. For instance, prefer:
// Using role and name (preferred)
expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
over:
// Using test-id (less preferred)
expect(screen.getByTestId('submit-button')).toBeInTheDocument();
The first approach verifies that there is a button labeled "Submit", which is what matters to the user
medium.com
.
Use screen and userEvent: The screen object from RTL provides access to the rendered DOM globally, so you don’t need to extract queries from render. This improves readability (e.g., screen.getByText('Hello')). For simulating user interactions, use @testing-library/user-event which more closely mimics real user events than fireEvent
medium.com
. For example, userEvent.type(inputElement, 'hello') types each character like a real user (triggering onChange for each), whereas fireEvent.change(input, { target: { value: 'hello' } }) just sets the value. Using userEvent can also handle things like clicking (which might fire blur on another element, etc. in sequence).
Async Actions: When testing components that involve asynchronous behavior (like data fetching or delayed UI updates), use findBy queries or waitFor. For example, if a component shows a loading spinner then eventually displays data:
const { findByText } = render(<FetchComponent />);
expect(await findByText(/data loaded successfully/i)).toBeInTheDocument();
The findByText will retry until the text appears or timeout, and the await pauses the test until then
medium.com
. Alternatively, you can use waitFor to wait for a certain state change. Avoid using act manually in most cases – RTL’s async utilities typically handle it for you. Use await with findBy* or wrap code in waitFor when an assertion depends on something that changes after an event loop tick or two.
Isolation vs Integration: For pure components (no external behavior), simple unit tests are fine. But many React components have side effects (API calls, context usage). Use mocks or MSW (Mock Service Worker) for API calls so your tests don’t hit real endpoints. For context or Redux, you can wrap the component with the provider in tests (or better, abstract that in a custom render function). Aim to test components in an environment close to real usage: if a component expects a context, render it with a test provider that supplies the context.
Test Edge Cases and Error States: Don’t just test the ideal path. Simulate error conditions (like API returning an error, or invalid user input) and ensure the component handles it gracefully (e.g., shows an error message)
medium.com
. Also test boundary conditions (empty list, max characters in input, etc.). This ensures robustness.
Snapshot Testing (Use Sparingly): You can use toMatchSnapshot() to capture component output and detect unexpected changes. However, be cautious – snapshots can be too broad and may make tests brittle if overused. It’s often better to write explicit assertions for specific elements or content, which is more intention-revealing.
Organize Test Files: Keep test files alongside components (e.g., Component.test.js in the same folder) or in a __tests__ directory mirroring the component structure. The cascade agent can then easily find tests relevant to a component. Use descriptive describe blocks to group related tests (e.g., <LoginForm /> tests)
medium.com
, and clear it/test descriptions for what behavior is being verified.
Performance of Tests: Keep tests fast by avoiding unnecessary waits or real network calls. Jest and RTL run tests in a JSDOM environment – no real browser – so some things (like layout) might not exactly match a real browser. But it’s fast and usually sufficient. Use jest.useFakeTimers() if you have setTimeouts to control time in tests.
Coverage and Continuous Integration: Aim for a high coverage on critical modules (logic, reducers, etc.), but remember 100% coverage doesn’t guarantee quality – focus on meaningful tests. Integrate tests into CI so that no regression goes unnoticed. The cascade agent can learn from these tests as examples of expected behavior for components.
Example – Testing a form submission with React Testing Library:
// LoginForm.test.jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import LoginForm from "./LoginForm";

test("allows the user to log in successfully", async () => {
  render(<LoginForm />);

  // Simulate user typing into the username and password fields
  userEvent.type(screen.getByLabelText(/username/i), "myUser");
  userEvent.type(screen.getByLabelText(/password/i), "secret123");
  // Simulate clicking the login button
  userEvent.click(screen.getByRole('button', { name: /log in/i }));

  // After login, suppose a welcome message "Welcome, myUser" is displayed
  const welcomeMessage = await screen.findByText(/welcome, myUser/i);
  expect(welcomeMessage).toBeInTheDocument();
});
In this test, we render the LoginForm component, fill out the form using userEvent.type (which fires the proper events for each keystroke), and click the submit button. We then use findByText to asynchronously wait for the welcome message to appear (indicating a successful login flow). We assert that this message is in the document, confirming the login process worked. Notice we didn’t inspect any internal state like “isLoggedIn” – we only checked what a user would see, which is the recommended approach
medium.com
. This test is also resilient to minor presentational changes (we used case-insensitive regex for text). Writing tests like this ensures your components and flows work and helps the cascade agent learn correct usage patterns (like how to interact with form controls via label text and roles).
Performance Optimization Techniques
Performance is important for large or complex React applications to ensure a smooth user experience. React is fast by default, but you should be mindful of expensive operations and re-renders. Consider these optimization techniques and patterns:
Code Splitting and Lazy Loading: Don’t load all your code upfront if it isn’t needed. Utilize dynamic import() and React.lazy with <Suspense> to split your bundle and lazy-load components when they are rendered
uxpin.com
. For example, you might lazy-load a settings panel that the user only opens occasionally:
const SettingsPanel = React.lazy(() => import('./SettingsPanel'));
// ...
<Suspense fallback={<Spinner />}>
  {showSettings && <SettingsPanel />}
</Suspense>
Here, the SettingsPanel code is loaded only when needed, improving initial load time. Lazy loading defers loading of non-critical components, reducing the amount of JS that must be parsed on first load
uxpin.com
. Always provide a fallback UI for Suspense (like a spinner or placeholder) to display while the component is loading.
Avoid Unnecessary Re-renders: React re-renders a component when its state or props change. Minimizing needless re-renders can boost performance:
React.memo: For functional components that render the same output given the same props, wrap them in React.memo(Component). This shallowly compares props and skips re-render if props haven’t changed
uxpin.com
. Use this for pure presentational components that get frequently re-rendered with the same props. Be cautious: if the props are objects or arrays that change reference every time (like inline functions or new arrays), memo won’t help unless those are memoized too.
useMemo and useCallback: Use useMemo to memoize expensive calculations so they don’t run on every render without need
uxpin.com
. Use useCallback to memoize callback props so they don’t trigger child re-renders every time. For instance, if you pass a handler prop to a child and that handler is defined inline, wrap it in useCallback to preserve the same function instance across renders (unless its dependencies change). This prevents the child (especially if it’s memoized) from thinking “props changed”.
Avoid Arrow Functions or Object Literals in JSX: Defining functions in JSX props (e.g., <List itemRenderer={() => {...}}> ) or creating objects/arrays in JSX (like <Comp style={{ margin: 0 }} />) will create new references on every render. This can cause child components to re-render unnecessarily. Instead, define the function outside or use useCallback, and define static objects outside or use useMemo
uxpin.com
.
Split Large State: If a component holds a large piece of state that always changes altogether, all parts of that component will re-render whenever it changes. If possible, split state into multiple state variables so that changing one doesn’t re-render unrelated parts of the UI. Alternatively, break the component into multiple components so that each one subscribes to only the part of state it needs.
Keys in Lists: Provide stable key props when rendering lists. A unique key helps React identify items that changed. Avoid using array indices as keys, as this can lead to issues during re-ordering and can hurt performance for insertion/deletion (because React might unnecessarily re-render list items)
uxpin.com
. Use an ID or unique value from the data if possible.
Throttling Expensive Operations: For actions like filtering a list as the user types, performing the filter on every keystroke can be expensive. Use useDeferredValue or manually throttle/debounce such operations. For example, you can use lodash.debounce to update state 300ms after the last key press rather than on every key stroke. React 18’s useDeferredValue can also be used to defer the update of the filter results so the input updates immediately and the filtering happens slightly later.
useTransition for Large Updates: As described in Hooks, wrap non-critical updates in startTransition. This tells React those updates can be interrupted if more urgent updates (like user input) come in. It leads to smoother, more responsive UIs under heavy load
vercel.com
. For example, when switching a tab that loads a big list, you could do startTransition(() => setActiveTab(tab)) so that if the user quickly clicks another tab, React can abandon rendering the first huge list and switch to the next. Transitions are a new way to prioritize rendering and avoid blocking the main thread with long tasks
vercel.com
.
Virtualize Long Lists: If you need to display very long lists (hundreds or thousands of items), consider list virtualization (using libraries like react-window or react-virtualized). These render only the visible items and a small buffer, rather than all items, drastically improving performance and memory usage.
Avoid Heavy Computation in Render: Intensive calculations (sorting large arrays, computing layouts) should be done outside of rendering or cached. If you must do them in render, use useMemo to cache the result. If the computation is truly heavy and independent of React, consider moving it to a web worker.
Profile and Measure: Use React’s Developer Tools Profiler to identify slow components or frequent re-renders. The profiler can show you which components render often and how long they take
moldstud.com
. This data helps target optimizations. Also track performance metrics (like interaction response times). Tools like Lighthouse or web vitals (TTI, TTFB, etc.) can show if your app is lagging.
Optimizing External Dependencies: Sometimes performance issues come from outside React (like a poorly performing third-party library). Profile to see if any such code is a bottleneck. For example, heavy animation libraries or complex canvas drawing might need optimization or moving off the main thread.
Use Production Build: Always test performance with the production build of React (which removes development warnings and does optimizations). Development mode is much slower due to extra checks.
Example – Using React.memo and useCallback:
const Item = React.memo(function Item({ value, onDelete }) {
  console.log("rendering Item", value);
  return (
    <li>
      {value} <button onClick={() => onDelete(value)}>x</button>
    </li>
  );
});

function ItemList({ items, onDeleteItem }) {
  // useCallback to memoize the handler so that React.memo(Item) sees the same onDelete prop for same item
  const handleDelete = React.useCallback((itemValue) => {
    onDeleteItem(itemValue);
  }, [onDeleteItem]);

  return (
    <ul>
      {items.map(item => (
        <Item key={item.id} value={item.label} onDelete={handleDelete} />
      ))}
    </ul>
  );
}
In this snippet, Item is wrapped in React.memo, so it will skip re-rendering if the value and onDelete props are the same as last time. In ItemList, we use useCallback to ensure the handleDelete function is stable across renders (unless the onDeleteItem prop changes). This way, each Item sees the same onDelete reference and won’t re-render unless its own value or the delete handler reference actually changes. This pattern prevents re-rendering all items in the list when, say, a new item is added or one item changes, by isolating the change. We can confirm via the console log that unchanged items don’t re-render. This technique, along with others like memoization, helps optimize React performance by preventing unnecessary work
uxpin.com
. Another example: implementing a search with transition:
function SearchableList({ items }) {
  const [query, setQuery] = useState("");
  const [filtered, setFiltered] = useState(items);
  const [isPending, startTransition] = useTransition();

  const handleInput = (e) => {
    const q = e.target.value;
    setQuery(q);
    // Mark the filtering update as low priority
    startTransition(() => {
      const filteredItems = items.filter(item => item.includes(q));
      setFiltered(filteredItems);
    });
  };

  return (
    <>
      <input value={query} onChange={handleInput} placeholder="Search..." />
      {isPending && <div>Updating list…</div>}
      <ul>{filtered.map(it => <li key={it}>{it}</li>)}</ul>
    </>
  );
}
In this SearchableList, as the user types, we update the input (query) immediately, but the heavy filtering logic is inside startTransition. If the list is large, filtering can be slow; using a transition means if the user keeps typing, React can drop intermediate filtering work and only render the latest filtered list, keeping the input responsive. The UI shows a fallback “Updating list…” when isPending is true (meaning a transition update is ongoing). This leads to a much smoother experience in scenarios with expensive renders, exemplifying React 18’s concurrent rendering capabilities. In summary, optimize when you have evidence of a problem – use the techniques above judiciously. Most of the time, writing clean React code and using functional components will be enough. But when you hit performance bottlenecks, these patterns (memoization, lazy loading, splitting work, etc.) are your tools to resolve them.
Accessibility and Semantic HTML
Building accessible and semantic UI is a best practice that benefits all users and improves maintainability. React supports accessibility features out of the box, but it’s the developer’s responsibility to use them correctly. Key guidelines include:
Use Semantic HTML Elements: Always prefer semantic tags that convey meaning, instead of non-semantic <div> or <span> for everything. For example, use <button> for clickable actions, <input> for inputs, <form> for form container, <nav> for navigation sections, <header>, <main>, <footer> to define page regions, and headings (<h1>...<h6>) for titles/subtitles. Using the right element provides built-in accessibility benefits (like keyboard focus handling and screen reader identification)
developer.mozilla.org
. Screen readers and browsers rely on these semantics to help users navigate. Replacing a clickable <div> with a <button> not only is semantic but also gives you keyboard focus and “click” via Enter key automatically.
Alternative Text for Non-Text Content: Ensure images have meaningful alt attributes (or use empty alt if the image is purely decorative). For icons, if using <svg> or icon fonts, provide appropriate aria-label or visually hidden text. Text alternatives are crucial so that screen reader users know what the image is conveying
developer.mozilla.org
. Example: <img src="chart.png" alt="Sales increased 20% in 2025" />. If an image is decorative, alt="" tells assistive tech to ignore it.
Form Labels and Controls: Every form control (<input>, <select>, <textarea>) should have an associated <label> or appropriate aria labeling. Use the <label for="id"> (with matching id on the input) or wrap input with <label>. This ensures screen readers announce the label when the field is focused. For custom controls (like custom styled checkboxes), use aria-label or aria-labelledby to associate text. Also, group related fields with <fieldset> and <legend> for context when needed.
Keyboard Navigation: All interactive elements must be keyboard accessible. This means using semantic controls (as mentioned) which are focusable by default. If you create a custom component that isn’t inherently focusable (like a clickable card <div>), add a tabIndex="0" to make it focusable and handle key events (like Enter or Space) to activate it
moldstud.com
. Provide “skip to main content” links at the top for screen reader and keyboard users to bypass repetitive navigation.
Focus Management: When modals or overlays open, transfer focus to them, and trap focus inside while open (so Tab doesn’t focus behind the modal). When they close, return focus to a logical place (e.g., the button that opened it). Use libraries or hooks for focus trapping if needed. Also, ensure visible focus outlines are present on navigable elements (don’t remove the default outline without providing an equivalent).
ARIA Roles and Attributes: Use ARIA attributes to supplement semantics, not replace them. First, use the correct HTML element (which often already includes a default ARIA role). If needed, add ARIA roles for custom widgets (e.g., role="dialog" on a modal container) and ARIA attributes to convey state (aria-expanded, aria-current, aria-live, etc.). For example, if you build a custom accordion component, manage aria-expanded on the trigger and aria-hidden on the content panel accordingly. ARIA can also help announce dynamic content changes – e.g., using aria-live="polite" on a container where validation messages appear will notify screen readers of new content.
Color and Contrast: Ensure text has sufficient color contrast against its background (following WCAG guidelines, typically 4.5:1 for normal text). This helps users with low vision or color blindness. Use tools like WebAIM contrast checker during design/development
moldstud.com
. Don’t rely solely on color to convey information (e.g., an error message should not just be red, it should also have an icon or text label "Error").
Responsive and Zoom: Design layouts that work with text resizing and high zoom levels. Use relative units (em, rem) for font sizes so that increasing system font size or zoom works. Test by zooming in to 200% and see if functionality is preserved.
Testing with Assistive Tech: Periodically test your app with screen readers (VoiceOver on Mac, NVDA or JAWS on Windows) and keyboard only. This helps catch things like focus order issues, missing labels, or confusing announcements. Also use automated tools: ESLint plugin jsx-a11y will warn you during development about common accessibility issues (like missing alt text or invalid ARIA usage), and tools like axe (the axe-core library or browser extension) can audit pages for issues.
Accessible Patterns: Implement known patterns accessibly. For instance, for an autocomplete input, ensure the list of suggestions is an ARIA listbox with options, and tie it to the input with aria-controls and aria-haspopup. For drag-and-drop, provide alternate keyboard mechanisms or at least announcements of drag start/drop. For spinners/loading, if content is updating dynamically, consider using aria-live="polite" to announce when new content is loaded (or assertive if it’s critical).
Semantic HTML Benefits: Using semantic HTML and accessibility best practices not only helps users with disabilities but often improves code quality and SEO. For example, headings and landmarks (<main>, <nav>, etc.) provide structure that search engines also use. Good semantics are a foundation of accessibility
developer.mozilla.org
, and React doesn’t remove or change your semantics – it’s all about how you write the JSX.
Accessibility Example – Semantic and Accessible Markup:
// BAD example (non-semantic, inaccessible):
<div onClick={openMenu} className="menu-button">Menu</div>

// GOOD example:
<button onClick={openMenu} className="menu-button" aria-haspopup="true" aria-expanded={isMenuOpen}>
  Menu
</button>
In the bad example, a <div> is used as a button. It is not focusable by keyboard and has no semantic meaning of “button”. The good example uses a <button> element which is focusable and clickable via keyboard by default. It also uses aria-haspopup and aria-expanded to indicate that it controls a menu popup, and whether that menu is currently open (assuming isMenuOpen is a boolean state). This way, assistive tech announces it as a collapsible menu button and conveys state. Another example, an image with alt text:
<img src="profile.jpg" alt="Profile picture of John Doe" />
This provides a descriptive alt, so a screen reader user knows what the image is (as opposed to alt="" which would hide it, or no alt which would read the file name – not useful). By following these practices – semantic HTML, proper ARIA when needed, ensuring keyboard access, and testing – your React app will be more inclusive. The Windsurf cascade coding agent can also learn from these patterns to enforce or suggest accessible code (e.g., warning if a <div onClick> should be a <button> or adding labels to inputs). Accessibility is an ongoing effort, but building it in from the start is far easier than retrofitting it later. An inclusive app not only widens your audience but often has cleaner, more maintainable code as a result
developer.mozilla.org
.
Citations

React Design Patterns and Best Practices for 2025

https://www.telerik.com/blogs/react-design-patterns-best-practices
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/

Managing Global State in React Applications Best Practices 2025 | MoldStud

https://moldstud.com/articles/p-effective-strategies-for-managing-global-state-in-react-applications-best-practices-2025
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/

Managing Global State in React Applications Best Practices 2025 | MoldStud

https://moldstud.com/articles/p-effective-strategies-for-managing-global-state-in-react-applications-best-practices-2025

Managing Global State in React Applications Best Practices 2025 | MoldStud

https://moldstud.com/articles/p-effective-strategies-for-managing-global-state-in-react-applications-best-practices-2025

Composition vs Inheritance – React

https://legacy.reactjs.org/docs/composition-vs-inheritance.html

React Design Patterns and Best Practices for 2025

https://www.telerik.com/blogs/react-design-patterns-best-practices

React Design Patterns and Best Practices for 2025

https://www.telerik.com/blogs/react-design-patterns-best-practices
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/

Rules of Hooks – React

https://react.dev/warnings/invalid-hook-call-warning

Rules of Hooks – React

https://react.dev/warnings/invalid-hook-call-warning
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/

How React 18 Improves Application Performance - Vercel

https://vercel.com/blog/how-react-18-improves-application-performance

How React 18 Improves Application Performance - Vercel

https://vercel.com/blog/how-react-18-improves-application-performance

File Structure – React

https://legacy.reactjs.org/docs/faq-structure.html

File Structure – React

https://legacy.reactjs.org/docs/faq-structure.html

File Structure – React

https://legacy.reactjs.org/docs/faq-structure.html

File Structure – React

https://legacy.reactjs.org/docs/faq-structure.html

File Structure – React

https://legacy.reactjs.org/docs/faq-structure.html
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/

Error Boundaries in React - Handling Errors Gracefully | Refine

https://refine.dev/blog/react-error-boundaries/

Error Boundaries in React - Handling Errors Gracefully | Refine

https://refine.dev/blog/react-error-boundaries/

Error Boundaries in React - Handling Errors Gracefully | Refine

https://refine.dev/blog/react-error-boundaries/

Best Practices for Using React Testing Library | by Dzmitry Ihnatovich | Medium

https://medium.com/@ignatovich.dm/best-practices-for-using-react-testing-library-0f71181bb1f4

Best Practices for Using React Testing Library | by Dzmitry Ihnatovich | Medium

https://medium.com/@ignatovich.dm/best-practices-for-using-react-testing-library-0f71181bb1f4

Best Practices for Using React Testing Library | by Dzmitry Ihnatovich | Medium

https://medium.com/@ignatovich.dm/best-practices-for-using-react-testing-library-0f71181bb1f4

Best Practices for Using React Testing Library | by Dzmitry Ihnatovich | Medium

https://medium.com/@ignatovich.dm/best-practices-for-using-react-testing-library-0f71181bb1f4

Best Practices for Using React Testing Library | by Dzmitry Ihnatovich | Medium

https://medium.com/@ignatovich.dm/best-practices-for-using-react-testing-library-0f71181bb1f4

Best Practices for Using React Testing Library | by Dzmitry Ihnatovich | Medium

https://medium.com/@ignatovich.dm/best-practices-for-using-react-testing-library-0f71181bb1f4

Best Practices for Using React Testing Library | by Dzmitry Ihnatovich | Medium

https://medium.com/@ignatovich.dm/best-practices-for-using-react-testing-library-0f71181bb1f4
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/
React Best Practices – A 10-Point Guide | UXPin

https://www.uxpin.com/studio/blog/react-best-practices/

How React 18 Improves Application Performance - Vercel

https://vercel.com/blog/how-react-18-improves-application-performance

Managing Global State in React Applications Best Practices 2025 | MoldStud

https://moldstud.com/articles/p-effective-strategies-for-managing-global-state-in-react-applications-best-practices-2025

HTML: A good basis for accessibility - Learn web development | MDN

https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/HTML

Achieving Accessibility in ReactJS Applications Best Practices | MoldStud

https://moldstud.com/articles/p-achieving-accessibility-in-reactjs-applications-best-practices

Achieving Accessibility in ReactJS Applications Best Practices | MoldStud

https://moldstud.com/articles/p-achieving-accessibility-in-reactjs-applications-best-practices

HTML: A good basis for accessibility - Learn web development | MDN

https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/HTML