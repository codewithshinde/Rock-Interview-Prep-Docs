# React `useCallback` (Simple Notes + Advanced Coverage)

## 1. One-line definition

`useCallback` makes React return the **same function reference** between renders, until the dependencies change.

It does not “make your app faster by default.”
It helps only in specific situations where **function identity** matters.

---

## 2. The noob problem: “Why do I need it?”

In JavaScript, every time you write this inside a component:

```js
const handleClick = () => {};
```

a **new function** is created on **every render**.

Even if the logic is identical, the function is a different object in memory.

So this is true:

```js
(() => {}) === (() => {}) // false
```

React passes props by reference. If you pass a function as a prop, the child sees it as “changed” every time.

---

## 3. Simple real-life analogy

Imagine you give your friend your “contact card.”

* Without `useCallback`: every time you meet, you print a **new card**, even though the phone number is the same. Your friend thinks “new contact.”
* With `useCallback`: you keep giving the **same card** until your number actually changes.

The content may be identical. The identity changes without `useCallback`.

---

## 4. First clean example: Why children re-render

### Step A: A child that logs when it renders

```tsx
import React from "react";

type ChildProps = {
  onAdd: () => void;
};

function Child({ onAdd }: ChildProps) {
  console.log("Child rendered");
  return <button onClick={onAdd}>Add</button>;
}
```

### Step B: Parent updates some unrelated state

```tsx
import React, { useState } from "react";

export default function App() {
  const [count, setCount] = useState(0);

  const onAdd = () => {
    console.log("Add clicked");
  };

  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Re-render parent</button>
      <Child onAdd={onAdd} />
    </>
  );
}
```

### What you observe

Every time you click “Re-render parent”, the console logs:

* `Child rendered`

Why?
Because `onAdd` is a **new function** each time the parent re-renders.

---

## 5. When `useCallback` helps (the simplest useful case)

`useCallback` helps when the child is memoized using `React.memo`, so it can skip re-render if props are the same.

### Step A: Memoize the child

```tsx
import React from "react";

type ChildProps = {
  onAdd: () => void;
};

const Child = React.memo(function Child({ onAdd }: ChildProps) {
  console.log("Child rendered");
  return <button onClick={onAdd}>Add</button>;
});

export default Child;
```

### Step B: Use `useCallback` in parent

```tsx
import React, { useCallback, useState } from "react";
import Child from "./Child";

export default function App() {
  const [count, setCount] = useState(0);

  const onAdd = useCallback(() => {
    console.log("Add clicked");
  }, []);

  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Re-render parent</button>
      <Child onAdd={onAdd} />
    </>
  );
}
```

### Result

Now clicking “Re-render parent” does NOT re-render `Child`, because:

* `Child` is memoized with `React.memo`
* `onAdd` keeps the same reference due to `useCallback`

This is the most important beginner use-case:
Stable callback + `React.memo` = fewer child re-renders.

---

## 6. Important truth (many beginners miss this)

`useCallback` does NOT stop the **parent** from re-rendering.

* Parent state change always re-renders parent.
* `useCallback` only helps you prevent **unnecessary child re-renders** when passing callbacks down.

---

## 7. The dependency rule (simple explanation)

This is the rule:

* If the callback uses a value from props/state, that value must be in the dependency array.
* Otherwise, your callback can “remember” an old value (stale value bug).

### Example: callback uses `count`

```tsx
const onAdd = useCallback(() => {
  console.log(count);
}, [count]);
```

This keeps `onAdd` updated when `count` changes.

### Common bug: wrong dependencies

```tsx
const onAdd = useCallback(() => {
  console.log(count);
}, []); // wrong if you expect latest count
```

This will keep logging the initial `count` forever.

Why? JavaScript closures capture values from the render where the function was created.

---

## 8. Advanced but still simple: reduce dependencies using functional state update

Sometimes you want a callback that updates state, but you don’t want to put state in dependencies.

### Problem version (needs `count` in deps)

```tsx
const onIncrement = useCallback(() => {
  setCount(count + 1);
}, [count]);
```

This callback changes every time `count` changes, so it’s not very “stable.”

### Better version (stable, empty deps)

```tsx
const onIncrement = useCallback(() => {
  setCount(prev => prev + 1);
}, []);
```

Why this is better:

* It doesn’t read `count` from closure.
* React gives you `prev` (the latest state).
* The callback can stay stable with `[]`.

This pattern is very common in advanced React code.

---

## 9. Another important use-case: `useEffect` dependencies

Sometimes you put a function in `useEffect` dependencies:

```tsx
useEffect(() => {
  doSomething();
}, [doSomething]);
```

If `doSomething` is recreated every render, the effect runs every render.

### Fix

```tsx
const doSomething = useCallback(() => {
  // effect logic
}, [/* dependencies */]);

useEffect(() => {
  doSomething();
}, [doSomething]);
```

This makes your effect run only when the real dependencies change.

---

## 10. When NOT to use `useCallback`

Do not add `useCallback` everywhere.

Avoid it when:

* You are not passing the function to memoized children.
* You are not using it as a dependency in `useEffect` / `useMemo`.
* The component is small and performance is fine.

Reason:

* `useCallback` adds complexity and has its own overhead (React must store and compare dependencies).

Good practice:

* Use it where it prevents real re-renders or fixes dependency loops.

---

## 11. Older React vs newer React

### Before Hooks (React classes)

* Class methods are usually stable references (e.g., `this.handleClick`).
* You didn’t face this problem as often.

### After Hooks (React 16.8+)

* Function components recreate functions each render.
* You need `useCallback` in some cases.

### React 18+ (modern behavior)

* React encourages concurrent rendering patterns.
* Strict Mode in development may render more than once to surface issues.
* `useCallback` is still the same API, but performance debugging and memo patterns are more common in large apps.

---

## 12. TypeScript notes (simple)

### Basic typing (often inferred)

```tsx
const onAdd = useCallback(() => {
  // ...
}, []);
```

### Explicit typing (if needed)

```tsx
const onAdd = useCallback((): void => {
  // ...
}, []);
```

### Passing typed callbacks

```tsx
type ChildProps = {
  onAdd: () => void;
};
```

TypeScript helps ensure:

* The child receives the correct function signature.
* You don’t pass `onAdd={123}` by mistake.

---

## 13. Common interview questions (at least 10)

1. What problem does `useCallback` solve in React?
2. Why do functions cause extra re-renders in React?
3. What is “function reference equality” and why does it matter?
4. Explain a case where `useCallback` gives no benefit.
5. How does `useCallback` help with `React.memo`?
6. What is a stale closure bug? Show an example with wrong dependencies.
7. How can you avoid adding state to dependencies when updating state inside a callback?
8. `useCallback` vs `useMemo`: what is the difference?
9. Why might `useCallback` make code slower or more complex if overused?
10. How does React 18 Strict Mode affect how you perceive renders during development?
11. How would you debug whether `useCallback` is actually helping?

---

## References

* React Docs: `useCallback`
  [https://react.dev/reference/react/useCallback](https://react.dev/reference/react/useCallback)
* React Docs: `React.memo`
  [https://react.dev/reference/react/memo](https://react.dev/reference/react/memo)
* React Docs: Hooks and closures / effects patterns (see `useEffect` and related learn pages)
  [https://react.dev/learn](https://react.dev/learn)
* MDN: JavaScript Closures
  [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
