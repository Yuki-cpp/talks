---
layout: section
---

# How to Use Them

Four syntax forms, custom concepts, and powerful combinations

---

# Four ways to constrain a template

```cpp {all|1-3|5-7|9-11|13-14|all}
// 1. requires-clause
template<typename T> requires std::integral<T>
T gcd(T a, T b);

// 2. Trailing requires-clause
template<typename T>
T gcd(T a, T b) requires std::integral<T>;

// 3. Constrained template parameter
template<std::integral T>
T gcd(T a, T b);

// 4. Abbreviated function template (terse syntax)
auto gcd(std::integral auto a, std::integral auto b);
```

<!--
There are four ways to apply a concept. They're mostly equivalent,
but there's one important difference I want to highlight.
-->

---

# Terse syntax: a subtle difference

Styles 1-3 enforce that `a` and `b` are the **same type** `T`.

Style 4 does **not** — each parameter gets its own type.

```cpp
// These are both integral, but could be different types
auto gcd(std::integral auto a, std::integral auto b);
```

```cpp
gcd(42, 17L);  // OK: int and long are both integral
```

<v-click>

To enforce the same type, use the explicit template form:

```cpp
// Style 3 is the simplest way to enforce the same type
template<std::integral T>
T gcd(T a, T b);
```

</v-click>

<!--
This catches people off guard. With the terse syntax, each `auto`
is its own template parameter. So you could get an int and a long.
If you need both to be the same type, use the explicit template form.
-->

---

# Writing your own concepts

Start simple, compose from there.

```cpp
template<typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::same_as<T>;
};
```

<v-click>

Compose with `&&` and `||`:

```cpp
template<typename T>
concept Number = std::integral<T> || std::floating_point<T>;

template<typename T>
concept Arithmetic = Number<T> && std::regular<T>;
```

</v-click>

<v-click>

Build domain concepts for your codebase:

```cpp
template<typename T>
concept Repository = requires(T repo, int id) {
    { repo.find(id) } -> std::convertible_to<std::optional<Entity>>;
    { repo.save(std::declval<Entity const&>()) } -> std::same_as<bool>;
    { repo.remove(id) } -> std::same_as<bool>;
};
```

</v-click>

<!--
Your own concepts can be as simple or as rich as you need.
The real power is in domain-specific concepts that encode
your architecture's contracts.
-->

---

# The `requires requires` pattern

Sometimes you need an ad-hoc constraint without naming a concept.

```cpp
template<typename T>
    requires requires(T a) { a.serialize(); }
void save(T const& obj);
```

<v-clicks>

- The first `requires` introduces a **requires-clause** (the constraint)
- The second `requires` starts a **requires-expression** (the predicate)
- It looks odd, but it's logically consistent

</v-clicks>

<v-click>

Prefer naming your concepts when the constraint is reused:

```cpp
// Better: give it a name
template<typename T>
concept HasSerialize = requires(T a) { a.serialize(); };

template<HasSerialize T>
void save(T const& obj);
```

</v-click>

<!--
requires requires is syntactically ugly but sometimes useful for one-off constraints.
If you find yourself writing the same requires-expression twice, extract it into a named concept.
-->

---

# Concepts + `if constexpr`

Compile-time branching based on type properties.

```cpp
template<typename T>
std::string to_string(T const& val) {
    if constexpr (std::integral<T>) {
        return std::to_string(val);
    }
    else if constexpr (std::floating_point<T>) {
        std::ostringstream oss;
        oss << std::fixed << std::setprecision(2) << val;
        return oss.str();
    }
    else if constexpr (requires { val.str(); }) {
        return val.str();
    }
    else {
        static_assert(false, "No conversion to string for this type");
    }
}
```

<v-click>

Concepts and inline `requires`-expressions work naturally as `if constexpr` conditions — no need for separate type traits.

</v-click>

<!--
Concepts work perfectly with if constexpr. You can use named concepts
or inline requires-expressions as conditions. This replaces a lot
of the manual type_traits checks people used to write.

Note: static_assert(false) in a discarded if-constexpr branch was
technically ill-formed NDR before CWG2518. Modern compilers (GCC 13+,
Clang 17+, MSVC) accept it in all language modes as a retroactive DR fix.
-->
