In React development environments, Strict Mode intentionally mounts and unmounts components twice. This frequently triggers an immediate `Error: Duplicate DailyIframe instances are not allowed` if developers attempt to instantiate a Daily call instance inside a standard `useEffect`.

You need to implement a foundational `App.tsx` React component that safely initializes a headless Daily call object without throwing duplicate instance exceptions during development.

**Constraints:**
- You MUST use the `@daily-co/daily-react` library.
- You MUST utilize the `useCallObject()` hook to orchestrate the singleton initialization rather than manually calling `DailyIframe.createCallObject()`.
- You MUST wrap the child components (e.g., a dummy `<CallUI />` component) inside a `<DailyProvider>` and pass the initialized call object to it.