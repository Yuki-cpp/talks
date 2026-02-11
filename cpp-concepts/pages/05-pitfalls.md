---
layout: section
---

# Pitfalls and Gotchas

What concepts **don't** do, and how to avoid common mistakes

---

# Pitfall 1: Concepts check syntax, not semantics

```cpp
template<typename T>
concept Addable = requires(T a, T b) { a + b; };
```

<v-click>

This only checks that `a + b` **compiles**. It says nothing about:

- Whether `+` is commutative
- Whether `+` has O(1) complexity
- Whether `+` actually "adds" anything

</v-click>

<v-click>

```cpp
struct Missile {
    Missile operator+(Missile const&) { launch(); return *this; }
};

static_assert(Addable<Missile>);  // true!
```

</v-click>

<v-click>

Standard concepts like `std::regular` document semantic requirements in prose — but the compiler only enforces the syntactic part.

**Concepts reduce bugs. They don't eliminate the need for tests.**

</v-click>

<!--
This is the most important thing to understand about concepts.
They are syntactic checks only. The standard documents semantic
requirements, but the compiler can only verify that expressions compile.
-->

---

# Pitfall 2: Over-constraining

Don't require more than your function body actually uses.

```cpp
// Too strict: excludes std::list, std::set, etc.
void print_all(std::random_access_iterator auto first,
               std::random_access_iterator auto last) {
    for (auto it = first; it != last; ++it)
        std::cout << *it << '\n';
}
```

<v-click>

The body only uses `++`, `!=`, and `*` — that's `std::input_iterator` territory.

```cpp
// Just right
void print_all(std::input_iterator auto first,
               std::sentinel_for<decltype(first)> auto last) {
    for (; first != last; ++first)
        std::cout << *first << '\n';
}
```

</v-click>

<v-click>

**Rule of thumb:** constrain to what the body calls, not to what you *expect* callers to pass.

</v-click>

<!--
Over-constraining is the concept equivalent of taking a concrete type
when you should take an interface. Only require what you actually use.
-->

---

# Pitfall 3: Under-constraining

Too-lax concepts push errors back inside the function body.

```cpp
template<typename T>
concept HasValue = requires { typename T::value_type; };

template<HasValue T>
void process(T const& container) {
    for (auto const& elem : container) { // may fail here
        std::cout << elem << '\n';
    }
}
```

<v-click>

A type with a `value_type` alias isn't necessarily iterable.

```cpp
struct Fake { using value_type = int; };
process(Fake{});  // passes the concept, fails inside the body
```

</v-click>

<v-click>

**Test your concepts with types that should *fail*** — if they pass, your concept is too lax.

</v-click>

<!--
If your concept doesn't check enough, you're back to cryptic template
errors inside the function body. Write negative tests for your concepts.
-->

---

# Pitfall 4: `requires requires` confusion

```cpp
template<typename T>
    requires requires(T a) { a.foo(); }
void bar(T);
```

Why twice?

<v-clicks>

- `requires` **#1** — introduces a *requires-clause* (constraint on the template)
- `requires` **#2** — begins a *requires-expression* (evaluates to `bool`)
- A requires-clause expects a boolean; a requires-expression produces one

</v-clicks>

<v-click>

```cpp
// Equivalent, but clearer:
template<typename T>
concept HasFoo = requires(T a) { a.foo(); };

template<HasFoo T>
void bar(T);
```

</v-click>

<v-click>

When you see `requires requires`, consider whether a named concept would be clearer.

</v-click>

<!--
The double requires trips everyone up. Just remember:
the first is the clause, the second starts the expression.
If it bugs you, name the concept.
-->

---

# Pitfall 5: Subsumption surprises

Subsumption only works when atomic constraints **originate from the same concept definition**.

```cpp
template<typename T>
constexpr bool is_integral_v = std::is_integral_v<T>;

template<typename T>
concept IntLike = is_integral_v<T>;  // wraps a type trait, not a concept

template<typename T> requires IntLike<T>
void f(T);

template<typename T> requires std::integral<T>
void f(T);
```

<v-click>

```
error: call to 'f' is ambiguous
```

`IntLike` uses a `constexpr bool` — the compiler can't trace it back to `std::integral`'s atomic constraints. They look equivalent but are **opaque** to subsumption.

</v-click>

<v-click>

The fix: build concepts **from** other concepts, not from raw type traits:

```cpp
template<typename T>
concept IntLike = std::integral<T>;  // now shares atomic constraints
```

</v-click>

<v-click>

**Rule:** if you rely on overload ordering, compose concepts from concepts — never from `constexpr bool` or raw type traits.

</v-click>

<!--
Subsumption is the mechanism that lets the compiler prefer a more-constrained overload.
It works by tracing constraints back through concept definitions to atomic constraints.
If two constraints originate from the same concept definition, the compiler can compare them.
But constexpr bools, raw type traits, and duplicate requires-expressions are opaque —
the compiler treats them as unrelated even if they check the same thing.
-->

---

# Pitfall 6: Compilation cost

Complex `requires`-expressions are evaluated during overload resolution.

<v-clicks>

- Every candidate function with constraints must have those constraints checked
- Deeply nested or large concepts multiply the work
- **Mitigation:** keep concepts focused and modular

</v-clicks>

<v-click>

```cpp
// Prefer small, composable concepts
template<typename T>
concept Printable = requires(std::ostream& os, T val) { os << val; };

template<typename T>
concept Serializable = requires(T val) {
    { val.to_string() } -> std::convertible_to<std::string>;
};

// Compose when needed
template<typename T>
concept Loggable = Printable<T> || Serializable<T>;
```

</v-click>

<!--
Concepts aren't free. The compiler evaluates them during overload resolution.
Keep them small and compose them rather than writing one giant requires-expression.
-->
