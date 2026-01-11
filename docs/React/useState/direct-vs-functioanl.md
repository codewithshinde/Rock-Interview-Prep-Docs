# React `useState` – Direct Update vs Functional Update

## Topic Category

* React (Core Hooks)
* JavaScript (Closures, Functions)
* TypeScript (State typing patterns)

---

## 1. What is `useState`?

`useState` is a React Hook used to add state to functional components.

```ts
const [state, setState] = useState(initialValue);
```

* `state` → current state value
* `setState` → function to update state
* Updating state triggers a re-render

---

## 2. Two Ways to Update State

### Method 1: Direct Value Update

```ts
setCounter(counter + 1);
```

### Method 2: Functional Update (Callback Form)

```ts
setCounter(prev => prev + 1);
```

Both update state, but **they behave differently internally**.

---

## 3. Why Do Two Update Methods Exist?

Because **React state updates are asynchronous and batched**.

React does NOT immediately update state when `setState` is called.
It schedules updates and applies them later for performance.

This difference became **much more important in newer React versions**.

---

## 4. How React State Updates Work (Simplified)

1. Component renders
2. React captures state values
3. Event handlers run
4. `setState` queues updates
5. React batches updates
6. React re-renders with new state

This means **state variables inside a render are snapshots**, not live values.

---

## 5. Direct State Update (Older Pattern)

### Example

```ts
setCounter(counter + 1);
```

### What Happens Internally

* Uses the `counter` value from the **current render**
* If multiple updates happen before re-render, all use the same value

### Problem Example

```ts
setCounter(counter + 1);
setCounter(counter + 1);
```

Result: counter increases by **1**, not 2

Why:

* Both updates read the same old value
* React batches them together

---

## 6. Functional State Update (Modern Pattern)

### Example

```ts
setCounter(prev => prev + 1);
```

### What Happens Internally

* React passes the **latest committed state**
* Each update runs sequentially
* No stale values

### Safe Example

```ts
setCounter(prev => prev + 1);
setCounter(prev => prev + 1);
```

Result: counter increases by **2**

---

## 7. Real-Life Analogy

### Scenario: Bank Balance

#### Direct Update

You check balance = 100
You say: “Set balance to 101”

Two transactions run together:

* Both read 100
* Both set 101
* One update is lost

#### Functional Update

You say: “Whatever the balance is, add 1”

Each transaction uses the latest balance
No updates are lost

---

## 8. Older React vs Newer React

### Older React (Before React 18)

* Less aggressive batching
* Mostly synchronous updates
* Direct updates worked most of the time

### Newer React (React 18+)

* Automatic batching everywhere
* Concurrent rendering
* Strict Mode double execution (development)
* Async rendering behavior

Functional updates are now **strongly recommended**.

---

## 9. When Direct Update Breaks (Important Cases)

### Case 1: Multiple Updates

```ts
setCounter(counter + 1);
setCounter(counter + 1);
```

### Case 2: Async Code

```ts
setTimeout(() => {
  setCounter(counter + 1);
}, 1000);
```

The `counter` value may be outdated.

### Case 3: Effects

```ts
useEffect(() => {
  setCounter(counter + 1);
}, []);
```

This captures initial state only.

---

## 10. Functional Update Fixes All These

```ts
setCounter(prev => prev + 1);
```

Works safely in:

* Event handlers
* `useEffect`
* `setTimeout`
* Promises
* Concurrent rendering

---

## 11. TypeScript Perspective

### State Typing

```ts
const [counter, setCounter] = useState<number>(0);
```

### Functional Update Type Safety

```ts
setCounter((prev: number) => prev + 1);
```

TypeScript ensures:

* `prev` is always correct type
* No accidental undefined or stale access

---

## 12. JavaScript Concept Behind This

### Closure

```ts
function handler() {
  console.log(counter);
}
```

* `counter` is captured when component renders
* Not updated dynamically
* Functional update avoids closure problems

---

## 13. Recommended Best Practice (Modern React)

Rule:

> If the next state depends on the previous state, always use functional update

Recommended:

```ts
setCounter(prev => prev + 1);
```

Avoid (unless absolutely safe):

```ts
setCounter(counter + 1);
```

---

## 14. Common Interview Questions (10)

1. Why does React provide a functional form of `setState`?
2. What is state batching in React?
3. Explain stale state in React.
4. What happens if you call `setState` twice in a row?
5. How does React 18 batching differ from earlier versions?
6. Why does `useEffect` sometimes use outdated state?
7. How do closures affect React state?
8. What problems does functional update solve?
9. Is `setState` synchronous or asynchronous?
10. When would direct state updates still be acceptable?

---

## 15. Summary

* React state updates are asynchronous
* Direct updates rely on render-time values
* Functional updates rely on React’s latest state
* Modern React strongly favors functional updates
* This concept connects React, JavaScript closures, and TypeScript safety

---

## References

* React Official Docs – State Updates
  [https://react.dev/reference/react/useState](https://react.dev/reference/react/useState)
* React 18 Automatic Batching
  [https://react.dev/blog/2022/03/08/react-18-upgrade-guide](https://react.dev/blog/2022/03/08/react-18-upgrade-guide)
* JavaScript Closures – MDN
  [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
* Dan Abramov – React State Explained
  [https://overreacted.io/a-complete-guide-to-useeffect/](https://overreacted.io/a-complete-guide-to-useeffect/)

