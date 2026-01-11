# `type` vs `interface` — the core idea

Both `type` and `interface` are used to **describe the shape of data**.

But they have **different strengths and purposes**.

---

# 2️⃣ Simple definitions (plain English)

### `interface`

> A **contract** that describes the shape of an object and can be **extended or merged**

### `type`

> A **label** for *any* type (object, union, function, tuple, primitive, etc.)

---

# 3️⃣ Real-life analogy 🧠

### 🏢 Interface = Office Job Role

Think of `interface` like a **job role** in a company.

```ts
interface Employee {
  name: string;
  id: number;
}
```

Now the company says:

> “All employees must have these properties.”

Later, HR updates the policy:

```ts
interface Employee {
  email: string;
}
```

👉 TypeScript **merges** them automatically.

This is called **declaration merging**.

---

### 🏷️ Type = Label / Tag

Think of `type` like a **label on a box**.

```ts
type Employee = {
  name: string;
  id: number;
};
```

Once the label is printed:

* ❌ You **cannot reopen it**
* ❌ You **cannot merge or modify it**

It’s fixed.

---

# 4️⃣ Key differences (simple table)

| Feature                   | `interface` | `type` |
| ------------------------- | ----------- | ------ |
| Can describe objects      | ✅           | ✅      |
| Can describe unions       | ❌           | ✅      |
| Can describe primitives   | ❌           | ✅      |
| Can describe tuples       | ❌           | ✅      |
| Declaration merging       | ✅           | ❌      |
| Extends / Intersects      | `extends`   | `&`    |
| Preferred for public APIs | ✅           | ❌      |
| Preferred for React props | ❌           | ✅      |

---

# 5️⃣ Simple code examples

## 🟢 Using `interface` (best for models / contracts)

```ts
interface User {
  id: number;
  name: string;
}

interface Admin extends User {
  role: "admin";
}
```

Why interface here?

* It models **entities**
* It may evolve
* It may be extended by other teams

---

## 🟢 Using `type` (best for flexibility)

```ts
type ID = string | number;

type Status = "loading" | "success" | "error";
```

❌ You **cannot** do this with `interface`.

---

# 6️⃣ Why React components usually use `type` 🤯

### Example

```tsx
type Props = {
  title: string;
  isActive?: boolean;
};

function Header({ title, isActive }: Props) {
  return <h1>{title}</h1>;
}
```

### Why not `interface`?

Because React props often need:

### ✅ Union types

```ts
type ButtonProps =
  | { variant: "primary"; onClick: () => void }
  | { variant: "link"; href: string };
```

### ✅ Conditional props

```ts
type InputProps =
  | { type: "text"; value: string }
  | { type: "checkbox"; checked: boolean };
```

### ✅ Utility types

```ts
type Props = React.ComponentProps<"button"> & {
  loading?: boolean;
};
```

👉 **Interfaces cannot do this well**.

---

# 7️⃣ Real-world React example (important)

```tsx
type ModalProps = {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
};
```

Why `type`?

* `children` is a union (`ReactNode`)
* Easy to combine with other types
* Safer & predictable
* No accidental merging

---

# 8️⃣ Why **NOT** interface for React props?

### Problem with interface in React:

```ts
interface Props {
  title: string;
}
```

Later, another file accidentally does:

```ts
interface Props {
  isAdmin: boolean;
}
```

💥 Boom — merged silently.

That can cause **bugs in large codebases**.

With `type`, this is impossible.

---

# 9️⃣ Purpose of using `type` inside React components

### 🎯 The real purpose:

1. **Prevent accidental merging**
2. **Support unions & conditional props**
3. **Better inference with hooks**
4. **Clear, closed definitions**
5. **Safer component APIs**

---

# 🔑 Rule of thumb (memorize this)

> **Use `interface` for domain models and public contracts**
> **Use `type` for React props, unions, and complex typing**

---

# 🔥 Interview-ready one-liner

> “I use `interface` for extendable object contracts and `type` for React props because props often need unions, composition, and strict closed definitions.”

If you say this, you’ll sound **very senior**.

---
