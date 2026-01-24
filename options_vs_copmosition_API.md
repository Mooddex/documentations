````md
# Options API vs Composition API  
## Practical Confusion Checklist (Vue / Nuxt / Pinia)

This file documents **real-world mistakes** caused by mixing Options API thinking with Composition API usage.  
Focus: **bugs, SSR issues, broken reactivity, and architectural confusion**.

No theory. Only things to watch out for.

---

## 1. Two APIs ≠ two mental models (root cause)

### Options API
- Object-based
- `this`-driven
- Vue wires everything for you

### Composition API
- Function-based
- Variable-driven
- **You wire everything yourself**

**Rule**
> If you are writing `ref`, `computed`, or `watch`, you are in **Composition mode**.  
> Do not think in `state`, `methods`, or `this`.

---

## 2. The biggest Pinia mistake

### ❌ Treating setup stores like Options stores

```ts
defineStore("x", () => {
  const state = reactive({}) // ❌ fake state
  return { state }
});
````

**Why this is wrong**

* Pinia already tracks refs
* Manual state wrapping breaks devtools and SSR hydration

### ✅ Correct

```ts
const count = ref(0);
return { count };
```

---

## 3. “state” does NOT exist in setup stores

### ❌

```ts
defineStore("x", () => {
  return {
    state: { count: 0 } // ❌ invented object
  };
});
```

This is **not state**, just an object.

**Rule**

> In setup stores, everything returned is state unless it’s a function.

---

## 4. Getters are NOT special in Composition API

### ❌ Options thinking

```ts
getters: {
  isLoggedIn: () => true
}
```

### ✅ Composition way

```ts
const isLoggedIn = computed(() => true);
return { isLoggedIn };
```

---

## 5. Actions are just functions

### ❌ Options thinking

```ts
methods: {
  doThing() {}
}
```

### ✅ Composition truth

```ts
function doThing() {}
return { doThing };
```

No magic. No binding. No `this`.

---

## 6. `this` must NEVER appear in Composition API

If you ever write:

```ts
this.user
```

inside:

* `<script setup>`
* setup Pinia store
* composable

You are **100% wrong**.

---

## 7. SSR serialization rule (Nuxt – critical)

Anything returned from:

* Pinia stores
* `useState`
* `useAsyncData`

**Must be JSON-serializable**

### ❌ Common violations

* Returning composables
* Returning refs from libraries
* Returning Zod schemas
* Returning functions inside state

### ✅ Correct approach

* Derive values with `computed`
* Keep libraries outside state

---

## 8. Composables ≠ Stores (major confusion)

| Composable                | Store                |
| ------------------------- | -------------------- |
| Reusable logic            | App-wide state       |
| Can return anything       | Must be SSR-safe     |
| Can hold functions & refs | Must be serializable |
| No devtools               | Has devtools         |

**Rule**

> Logic → composable
> Business state → store

Auth logic belongs in a **composable**, not stored state.

---

## 9. Lifecycle hook confusion

### ❌ Options API

```ts
mounted() {}
created() {}
```

### ✅ Composition API

```ts
onMounted(() => {})
```

Never mix.

---

## 10. `watch` vs `watchEffect`

* `watch` → reacts to **explicit sources**
* `watchEffect` → reacts to **everything it touches**

**Rule**

> Default to `watch`.
> Use `watchEffect` only when you fully understand the side effects.

---

## 11. Do NOT mirror props into refs

### ❌

```ts
const localUser = ref(props.user);
```

Breaks reactivity.

### ✅

```ts
const localUser = computed(() => props.user);
```

---

## 12. Stop forcing structure

Composition API does **not** care about order.

This is valid:

```ts
function signOut() {}
const user = computed(...)
```

There is no “state first” rule.

---

## Final mental reset

> **Options API describes WHAT something is**
> **Composition API describes HOW it works**

If you remember only one thing:

> **In Composition API, everything is just JavaScript.**

```

If you want, I can also:
- Split this into **Pinia-only rules**
- Add **Nuxt SSR-specific gotchas**
- Convert it into a **checklist poster**

Just say which.
```
