---
layout: section
---

# The Problem

Templates accept everything — and fail *late*

<!--
Let's start with something we've all experienced.
-->

---

# A simple template

```cpp
template<typename T>
std::string serialize(T const& val) {
    return val.to_string();
}
```

Looks fine. Let's call it.

```cpp
serialize(42);  // int has no to_string() member
```

What does the compiler say?

<!--
This looks perfectly reasonable. But if someone passes an int,
we get... well, let's see.
-->

---

# The compiler says...

<div class="text-sm overflow-auto max-h-96">

```
In instantiation of 'std::string serialize(const T&) [with T = int]':
error: request for member 'to_string' of 'val', which is of
  non-class type 'const int'
   |     return val.to_string();
   |            ~~~~^~~~~~~~~
note: in expansion of template argument 'int'
note: while substituting deduced template arguments
...
```

</div>

<v-click>

This is a **simple** case. Now imagine 3 levels of template nesting.

</v-click>

<!--
And this is the EASY case. One level deep.
In real codebases with layered templates, you get pages of errors
that point at internal implementation details, not at the call site.
-->

---

# The old workarounds

<div class="grid grid-cols-2 gap-8">

<div>

### `static_assert`

```cpp
template<typename T>
std::string serialize(T const& val) {
    static_assert(
        has_to_string_v<T>,
        "T must have to_string()"
    );
    return val.to_string();
}
```

<v-click>

Better message, but **no overload resolution**.

</v-click>

</div>

<div>

### SFINAE / `enable_if`

```cpp
template<typename T,
    typename = std::enable_if_t<
        has_to_string_v<T>>>
std::string serialize(T const& val) {
    return val.to_string();
}
```

<v-click>

Overload-friendly, but **hard to read and write**.

</v-click>

</div>

</div>

<!--
Before concepts, we had two approaches.
static_assert gives decent errors but doesn't interact with overload resolution.
SFINAE works with overloading but the syntax is painful.
Neither is great.
-->

---
layout: statement
---

# There must be a better way.

<v-click>

There is. Since C++20.

</v-click>
