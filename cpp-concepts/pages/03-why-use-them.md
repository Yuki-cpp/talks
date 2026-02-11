---
layout: section
---

# Why Use Them?

Better errors, clearer interfaces, smarter overloading

---
layout: two-cols-header
---

# Benefit 1: Readable error messages

::left::

**Without concepts**

```
error: request for member 'to_string'
  of 'val', which is of non-class
  type 'const int'
     return val.to_string();
            ~~~~^~~~~~~~~
note: in instantiation of
  'serialize<int>'
note: required from ...
note: required from ...
note: required from ...
```

::right::

**With concepts**

```
error: no matching function for call
  to 'serialize(int)'

note: candidate:
  'std::string serialize(const T&)'
  [with T = int]
note: constraints not satisfied

note: the required expression
  'v.to_string()' is invalid
```

<!--
Look at the difference. On the left, the error points inside the function body.
On the right, it tells you exactly which constraint failed and why.
No more digging through template instantiation stacks.
-->

---

# Benefit 2: Self-documenting interfaces

SFINAE is write-only code. Concepts read like prose.

<div class="mt-4"></div>

**Before** — what does this accept?

```cpp
template<typename Iter, typename = std::enable_if_t<
    std::is_base_of_v<std::input_iterator_tag,
        typename std::iterator_traits<Iter>::iterator_category>>>
void process(Iter first, Iter last);
```

<v-click>

**After** — immediately clear

```cpp
void process(std::input_iterator auto first, std::input_iterator auto last);
```

</v-click>

<v-click>

The constraint **is** the documentation.

</v-click>

<!--
With SFINAE, you have to mentally parse enable_if and type traits to figure out what a function accepts.
With concepts, you just read the parameter type. The constraint IS the documentation.
-->

---

# Benefit 3: Overload resolution

Concepts participate in overload resolution — no tag dispatch needed.

```cpp
void print(std::integral auto val) {
    std::cout << "Integer: " << val << '\n';
}

void print(std::floating_point auto val) {
    std::cout << "Float: " << std::fixed << val << '\n';
}

void print(std::ranges::range auto const& r) {
    for (auto const& elem : r) print(elem);
}
```

<v-click>

```cpp
print(42);               // calls integral overload
print(3.14);             // calls floating_point overload
print(std::vector{1,2}); // calls range overload
```

</v-click>

<v-click>

Each concept-constrained overload is a **distinct candidate** — the compiler matches the right one. No tag dispatch needed.

</v-click>

<!--
Each call resolves to the right overload automatically.
These concepts are non-overlapping, so ordinary overload resolution handles them.
When concepts DO overlap, subsumption kicks in: a more-constrained overload
(e.g. signed_integral vs integral) is preferred. We'll cover that in Pitfalls.
-->
