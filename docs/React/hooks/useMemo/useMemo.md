# `useMemo`

## Topic Category

* React (Performance, memoization)
* JavaScript (Reference equality, objects/arrays, closures)
* TypeScript (Type inference, generics, safe dependencies)

---

## 1. What is `useMemo`?

`useMemo` lets you **cache a computed value** between renders.

```ts
const value = useMemo(() => computeSomething(), [deps]);
```

* React runs your `computeSomething` on the first render.
* On later renders, React **reuses the old cached value** if dependencies did not change.
* If a dependency changed, React recomputes and stores the new result. ([React][1])

---

## 2. The simplest mental model

A component re-renders often. Anything you write directly in the component body runs again:

```ts
const filtered = filterTodos(todos, tab); // runs on every render
```

With `useMemo`, you tell React:

“Do not redo this calculation unless inputs changed.”

```ts
const filtered = useMemo(() => filterTodos(todos, tab), [todos, tab]);
```

---

## 3. Real-life analogy (noob-friendly)

Imagine you are making tea.

* Without `useMemo`: every time a guest asks “what’s the tea?”, you boil water again from scratch.
* With `useMemo`: you keep the tea ready in a thermos. You only boil again when the tea leaves or water changes.

Key point:

* It is not about correctness.
* It is about avoiding repeated work.

---

## 4. How React decides “dependency changed” (important)

React compares each dependency using **`Object.is`**. ([React][1])

That means:

* Primitives (number/string/boolean) are compared by value.
* Objects/arrays/functions are compared by **reference**, not by “looks the same”.

### JavaScript reference equality example

```js
const a = { x: 1 };
const b = { x: 1 };
a === b; // false (different objects)

const c = a;
a === c; // true (same reference)
```

Objects are reference types: two distinct objects are never equal even with same properties. ([MDN Web Docs][2])

This is the biggest reason `useMemo` and `memo` “don’t work” for beginners.

---

## 5. When you should use `useMemo`

### Use case A: Expensive calculation on every render

Example: filtering a big list.

```tsx
function TodoList({ todos, tab }: { todos: string[]; tab: string }) {
  const visibleTodos = useMemo(() => {
    return todos.filter(t => (tab === "all" ? true : t.includes(tab)));
  }, [todos, tab]);

  return (
    <ul>
      {visibleTodos.map((t, i) => (
        <li key={i}>{t}</li>
      ))}
    </ul>
  );
}
```

This matches the official pattern: cache a calculation until dependencies change. ([React][1])

### Use case B: Create stable objects/arrays passed to memoized children

Problem: objects are new on every render.

```tsx
const options = { pageSize: 20, sort: "desc" }; // new object every render
<Child options={options} />
```

Even if values are same, `options` reference changes -> child sees prop changed.

Fix:

```tsx
const options = useMemo(() => {
  return { pageSize: 20, sort: "desc" };
}, []);

<Child options={options} />
```

### Use case C: Keep effect dependencies stable

Sometimes you depend on an object inside `useEffect`:

```tsx
useEffect(() => {
  fetchData(searchOptions);
}, [searchOptions]);
```

If `searchOptions` is created inline, the effect runs every render.

Fix using `useMemo`:

```tsx
const searchOptions = useMemo(() => {
  return { text, matchMode: "whole-word" };
}, [text]);

useEffect(() => {
  fetchData(searchOptions);
}, [searchOptions]);
```

---

## 6. When you should NOT use `useMemo`

### Rule from React docs

You should only rely on `useMemo` as a **performance optimization**. If your code breaks without it, fix the real issue first. ([React][1])

Do not use `useMemo` to:

* “Store state”
* “Keep values forever”
* “Guarantee compute runs only once”

React may throw away the cache for specific reasons (example: edits in development, or if a component suspends during initial mount). ([React][1])

---

## 7. Why your memoized calculation might run twice in development

In React Strict Mode (development only), React may call your calculation function **twice** to help you find impure logic. One result is ignored. ([React][1])

So if you do:

```ts
const value = useMemo(() => {
  console.log("calc");
  return heavyWork();
}, [a, b]);
```

You may see `"calc"` printed twice in dev. That is expected.

---

## 8. Purity rule (very important)

Your `useMemo` function should be **pure**:

* Do not mutate props/state objects inside it.
* Do not do side effects (API calls, setState, DOM changes).

React explicitly recommends purity, and Strict Mode double-call helps reveal impurities. ([React][1])

Bad:

```ts
const value = useMemo(() => {
  todos.push("new"); // mutating a prop
  return todos.length;
}, [todos]);
```

Good:

```ts
const value = useMemo(() => {
  const copy = [...todos];
  copy.push("new");
  return copy.length;
}, [todos]);
```

---

## 9. `useMemo` vs `useCallback` (simple)

* `useMemo` caches a **value**
* `useCallback` caches a **function reference**

React docs show they are closely related; memoizing a function via `useMemo` works but is more verbose. ([React][1])

### Equivalent idea

```ts
const fn1 = useCallback(() => doSomething(a), [a]);

const fn2 = useMemo(() => {
  return () => doSomething(a);
}, [a]);
```

Prefer `useCallback` for functions; prefer `useMemo` for values.

---

## 10. TypeScript patterns for `useMemo`

### Pattern A: Let TypeScript infer

```ts
const stats = useMemo(() => {
  return { total, completed };
}, [total, completed]);
```

### Pattern B: Explicit generic (useful for complex return types)

```ts
type Stats = { total: number; completed: number };

const stats = useMemo<Stats>(() => {
  return { total, completed };
}, [total, completed]);
```

### Pattern C: Memoizing arrays safely

```ts
const ids = useMemo(() => items.map(i => i.id), [items]);
```

Be careful: `items` must be stable or intentionally included. If `items` is a new array each render, memoization won’t help.

---

## 11. Older React vs newer React

### Older React docs mindset

`useMemo` existed mainly as “avoid expensive recalculation”. The legacy docs describe it as recomputing only when deps change (but note the legacy page is marked out of date). ([React][3])

### Modern React (18/19 era)

Modern docs add key caveats:

* Strict Mode may run calculation twice in dev. ([React][1])
* Cache can be thrown away for specific reasons. ([React][1])
* `useMemo` is an optimization, not a correctness tool. ([React][1])

### New direction: React Compiler

React Compiler aims to do memoization automatically in many cases. React still keeps `useMemo`/`useCallback` as an “escape hatch” when you need precise control (for example, stable effect dependencies). ([React][4])

---

## 12. Common mistakes checklist

1. Using `useMemo` for correctness (wrong)
2. Missing dependencies (stale values)
3. Depending on objects/arrays that are recreated every render (memo becomes useless)
4. Doing side effects inside `useMemo` (wrong)
5. Expecting `useMemo` to keep values across unmounts (it won’t)
6. Adding `useMemo` everywhere (adds complexity and overhead)

---

## 13. Interview questions (most common)

1. What does `useMemo` do in React?
2. What problem does `useMemo` solve?
3. How does React decide whether dependencies changed?
4. Why does `useMemo` often “not work” with objects and arrays?
5. Why can `useMemo` run twice in development?
6. Can React discard the memoized value? When?
7. What’s the difference between `useMemo` and `useCallback`?
8. When should you avoid using `useMemo`?
9. How does `useMemo` help with `React.memo`?
10. What is a “pure function” in the context of `useMemo`?
11. How do missing dependencies create stale bugs?
12. How does React Compiler change the need for manual memoization?

---

## References

* React Docs: `useMemo` (behavior, dependencies with `Object.is`, Strict Mode double call, cache caveats, “optimization only”). ([React][1])
* React Docs: `StrictMode` (development-only extra checks). ([React][5])
* React Docs: React Compiler introduction (recommend relying on compiler, keep `useMemo`/`useCallback` as escape hatch). ([React][4])
* MDN: Working with objects (objects are reference types; equality is by reference). ([MDN Web Docs][2])
* MDN: Functions (functions are first-class objects). ([MDN Web Docs][6])

If you want, paste one of your components where “memo is not working” and I’ll mark exactly which dependency (or which inline object/JSX) is breaking memoization and how to fix it with `useMemo` (simple + advanced).

[1]: https://react.dev/reference/react/useMemo "useMemo – React"
[2]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects "Working with objects - JavaScript | MDN"
[3]: https://legacy.reactjs.org/docs/hooks-reference.html?utm_source=chatgpt.com "Hooks API Reference"
[4]: https://react.dev/learn/react-compiler/introduction "Introduction – React"
[5]: https://react.dev/reference/react/StrictMode?utm_source=chatgpt.com "<StrictMode> – React"
[6]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions "Functions - JavaScript | MDN"