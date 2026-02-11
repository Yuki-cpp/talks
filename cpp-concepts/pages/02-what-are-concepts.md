---
layout: section
---

# What Are Concepts?

Named compile-time predicates on types

---

# A concept is...

A **named boolean constraint** evaluated at compile time.

```cpp
template<typename T>
concept Serializable = requires(T v) {
    { v.to_string() } -> std::convertible_to<std::string>;
};
```

<v-clicks>

- It has a **name** — `Serializable`
- It takes **template parameters** — `T`
- It evaluates a **constraint expression** — the `requires` block
- It produces `true` or `false` at compile time

</v-clicks>

<!--
A concept is deceptively simple. It's just a named boolean.
The requires-expression is where the magic happens —
it lets you describe what a type must be able to do.
-->

---

# Anatomy of a `requires`-expression

```cpp
template<typename T>
concept Example = requires(T a, T b) {
    // Simple requirement — must compile
    a + b;

    // Type requirement — type must exist
    typename T::value_type;

    // Compound requirement — must compile AND satisfy a concept
    { a.size() } -> std::convertible_to<std::size_t>;

    // Nested requirement — additional boolean constraint
    requires std::copyable<T>;
};
```

<v-clicks>

| Kind | Checks |
|------|--------|
| **Simple** | Expression compiles |
| **Type** | Type name is valid |
| **Compound** | Expression compiles + return type matches |
| **Nested** | An additional constraint holds |

</v-clicks>

<!--
There are four kinds of requirements.
Simple ones just check that an expression compiles.
Type requirements check that a type alias exists.
Compound requirements also check the return type.
Nested requirements let you add further boolean conditions.
-->

---

# Standard library concepts

Don't reinvent the wheel — `<concepts>`, `<iterator>`, and `<ranges>` have you covered.

<div class="grid grid-cols-2 gap-6 mt-4">

<div>

**Core (`<concepts>`)**
- `std::same_as<T, U>`
- `std::derived_from<T, Base>`
- `std::convertible_to<From, To>`
- `std::integral<T>`
- `std::floating_point<T>`
- `std::copyable<T>` / `std::movable<T>`
- `std::regular<T>`
- `std::totally_ordered<T>`
- `std::invocable<F, Args...>`

</div>

<div>

**Iterator (`<iterator>`)**
- `std::input_iterator<I>`
- `std::forward_iterator<I>`
- `std::random_access_iterator<I>`
- `std::sortable<I, Comp>`

**Range (`<ranges>`)**
- `std::ranges::range<R>`
- `std::ranges::sized_range<R>`
- `std::ranges::input_range<R>`

</div>

</div>

<!--
The standard library comes with a rich set of ready-made concepts.
Before writing your own, check if one of these already does what you need.
-->
