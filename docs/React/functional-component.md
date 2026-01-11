# React Functional Components – TypeScript Typing Patterns

## Topic Category

* React (Function Components)
* TypeScript (Type Inference, Function Typing)
* JavaScript (Functions, Parameters, Return Values)

---

## 1. What Is a Functional Component?

A **functional component** in React is simply a **JavaScript function** that:

* Accepts `props` as input
* Returns JSX as output

```ts
function App(props) {
  return <div>Hello</div>;
}
```

React does **not** care how you type this function.
All differences discussed below are **TypeScript-only** concerns.

---

## 2. Key Idea (Very Important)

> All versions below create the **same React component**.
>
> React behaves exactly the same.
>
> The difference is **how much TypeScript guidance or restriction you add**.

This is about **developer experience**, not runtime behavior.

---

## 3. Real-Life Analogy (Simple)

Think of driving a car.

* The car is the same
* The road is the same
* The destination is the same

What changes:

* How much guidance or restriction you have while driving

This maps directly to how we type React components.

---

## 4. Typing Approaches (From Modern to Legacy)

---

## 4.1 Plain Function with Props Type (Modern, Recommended)

```ts
type AppProps = {
  message: string;
};

const App = ({ message }: AppProps) => {
  return <div>{message}</div>;
};
```

### What This Does

* TypeScript infers the return type automatically
* React already expects JSX
* Minimal syntax
* Clean and readable

### Why This Works Well Today

* TypeScript inference is very strong (TS 4.5+)
* React components are treated as normal functions
* Less boilerplate
* Easier to refactor

### Use Case

* Most production React applications
* Component libraries
* Enterprise codebases

This is the **default and recommended approach**.

---

## 4.2 Explicit Return Type (Optional Safety)

```ts
const App = ({ message }: AppProps): React.JSX.Element => {
  return <div>{message}</div>;
};
```

### What Changes

You explicitly tell TypeScript:

> “This function must return JSX.”

### Advantages

* Prevents returning invalid values
* Can catch mistakes in complex functions
* Useful for shared libraries or public APIs

### Disadvantages

* Adds visual noise
* Redundant in most cases
* Rarely needed in app code

### When to Use

* Component libraries
* Strict API boundaries
* When return type clarity is critical

---

## 4.3 Inline Props Type (Not Scalable)

```ts
const App = ({ message }: { message: string }) => {
  return <div>{message}</div>;
};
```

### What This Does

* Props type is defined inline
* No reusable type
* No sharing across components

### Pros

* Quick
* Simple
* Fine for small demos

### Cons

* Hard to reuse
* Hard to refactor
* Poor scalability

### Use Case

* Examples
* Tutorials
* Temporary code

Avoid this in real applications.

---

## 4.4 `React.FC` / `React.FunctionComponent` (Legacy Pattern)

```ts
const App: React.FC<AppProps> = ({ message }) => {
  return <div>{message}</div>;
};
```

### What This Does

* Forces the return type
* Automatically adds `children`
* Wraps the function in extra typing

---

## 5. Why `React.FC` Was Popular (Older React)

### React 16 Era

* TypeScript inference was weaker
* `children` was commonly required
* Class components were dominant
* Functional components felt “special”

Using `React.FC` felt safer and clearer at the time.

---

## 6. Why `React.FC` Is Discouraged Now

### Problems with `React.FC`

1. `children` is added even when not needed
2. Makes props less explicit
3. Worse generic inference in many cases
4. Harder to work with `defaultProps`
5. Adds abstraction without benefit

### Modern React Philosophy

> “A React component is just a function that returns JSX.”

So we type it like a **normal function**.

---

## 7. Old React vs New React (Clear Comparison)

| Aspect       | Older React (≤16)      | Modern React (18+)       |
| ------------ | ---------------------- | ------------------------ |
| Typing style | `React.FC<Props>`      | Plain function           |
| TypeScript   | Weak inference         | Strong inference         |
| Children     | Implicit               | Explicit                 |
| Mindset      | Components are special | Components are functions |

---

## 8. Why the Shift Makes Sense

### Old Thinking

* Components are a special React concept
* They need special typing

### New Thinking

* Components are functions
* JSX is just a return value
* TypeScript can infer almost everything

This reduces complexity and improves maintainability.

---

## 9. Best Practice (Modern Recommendation)

### Recommended

```ts
type AppProps = {
  message: string;
};

const App = ({ message }: AppProps) => {
  return <div>{message}</div>;
};
```

### Avoid (Unless Required)

```ts
const App: React.FC<AppProps> = ...
```

---

## 10. Interview-Ready Explanation

“In modern React, function components are treated as plain functions. TypeScript can infer the return type automatically, so the simplest form is preferred. `React.FC` was useful in older React versions but is now discouraged due to unnecessary constraints like implicit children and weaker inference.”

This answer signals **strong practical experience**.

---

## 11. Common Interview Questions (10)

1. What is a functional component in React?
2. Does React care how you type components in TypeScript?
3. Why is `React.FC` discouraged in modern React?
4. What are the drawbacks of implicit `children`?
5. When would you explicitly type a component’s return value?
6. How does TypeScript inference help React development?
7. What changed in React 18 that affected typing practices?
8. Is `JSX.Element` the same as `React.JSX.Element`?
9. Why are components treated as plain functions now?
10. What typing style would you use in a production app and why?

---

## 12. Final Takeaway

| Pattern                     | Recommendation   |
| --------------------------- | ---------------- |
| Plain function + props type | Best choice      |
| Explicit return type        | Optional         |
| Inline props type           | Small demos only |
| `React.FC`                  | Avoid in apps    |

---

## References

* React Official Docs – Components and Props
  [https://react.dev/learn/your-first-component](https://react.dev/learn/your-first-component)
* React + TypeScript Cheatsheet
  [https://react-typescript-cheatsheet.netlify.app](https://react-typescript-cheatsheet.netlify.app)
* React 18 Upgrade Guide
  [https://react.dev/blog/2022/03/08/react-18-upgrade-guide](https://react.dev/blog/2022/03/08/react-18-upgrade-guide)
* TypeScript Handbook – Type Inference
  [https://www.typescriptlang.org/docs/handbook/type-inference.html](https://www.typescriptlang.org/docs/handbook/type-inference.html)
