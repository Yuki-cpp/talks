# C++ Concepts: Constraining Templates Without Losing Your Mind

**Duration:** 30-45 minutes
**Target audience:** Engineers comfortable with C++14/17, not necessarily template metaprogramming experts

---

## Talk Outline

### 1. The Problem (5 min)

Start with a motivating example that the audience already feels in their bones.

- Show a simple template function (e.g., `template<typename T> auto serialize(T val)`) called with a wrong type
- Display the resulting compiler error wall (GCC/Clang template error for a missing member, nested instantiation failure, etc.)
- Key point: **templates accept everything and fail late**, deep inside the implementation
- Briefly mention the pre-concepts workarounds:
  - `static_assert` with `type_traits` — works but manual, no overload resolution
  - SFINAE / `std::enable_if` — powerful but ugly and hard to read
  - Show a quick SFINAE example so the audience appreciates what concepts replace

### 2. What Are Concepts? (5 min)

Introduce the C++20 feature.

- A concept is a **named compile-time predicate on types** (or packs of types)
- Syntax: `template<typename T> concept Name = constraint-expression;`
- Example — build a simple concept step by step:
  ```cpp
  template<typename T>
  concept Serializable = requires(T v) {
      { v.serialize() } -> std::convertible_to<std::string>;
  };
  ```
- Walk through the `requires`-expression anatomy:
  - Simple requirements (`x + y;` — must compile)
  - Type requirements (`typename T::value_type;` — type must exist)
  - Compound requirements (`{ expr } -> concept;` — must compile AND satisfy a concept)
  - Nested requirements (`requires (condition);` — additional boolean constraint)
- Show how standard library provides ready-made concepts in `<concepts>` and `<ranges>`:
  `std::integral`, `std::floating_point`, `std::copyable`, `std::invocable`, `std::ranges::range`, etc.

### 3. Why Use Them? (5 min)

Three concrete benefits, each with a live demo/example.

**3a. Readable error messages**
- Same broken call from Section 1, but now with a concept-constrained template
- Side-by-side comparison of error output: before vs. after
- The compiler now says *"constraint not satisfied"* and points at the concept, not at line 847 of an internal header

**3b. Expressive interfaces / Self-documenting code**
- A constrained function signature tells the reader *what* a type must be without reading the body
- Compare:
  ```cpp
  // Before
  template<typename Iter, typename = std::enable_if_t<
      std::is_base_of_v<std::input_iterator_tag,
          typename std::iterator_traits<Iter>::iterator_category>>>
  void process(Iter first, Iter last);

  // After
  void process(std::input_iterator auto first, std::input_iterator auto last);
  ```

**3c. Controlled overload resolution**
- Concepts participate in overload resolution and partial ordering
- Example: `print(std::integral auto v)` vs `print(std::floating_point auto v)` — compiler picks the right one, no tag dispatch or SFINAE needed
- Mention subsumption: a more-constrained overload is preferred over a less-constrained one

### 4. How to Use Them — Syntax in Practice (10 min)

Walk through the different places you can apply concepts.

**4a. Four ways to constrain a template parameter**
```cpp
// 1. requires-clause after template parameter list
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
- Discuss when each style is appropriate (team conventions, complexity of constraint)
- Note: style 4 means each parameter could be a *different* integral type — contrast with style 3

**4b. Writing your own concepts**
- Start simple: `concept Addable = requires(T a, T b) { a + b; };`
- Compose concepts with `&&` and `||`:
  ```cpp
  template<typename T>
  concept Number = std::integral<T> || std::floating_point<T>;
  ```
- Use `requires requires` (the double-requires pattern) for inline ad-hoc constraints — explain why the keyword appears twice
- Real-world example: a concept for a "Repository" or "Logger" interface (something the audience would use in production code)

**4c. `if constexpr` + concepts**
- Show how concepts pair with `if constexpr` for compile-time branching:
  ```cpp
  template<typename T>
  std::string to_string(T const& val) {
      if constexpr (std::integral<T>) {
          return std::to_string(val);
      } else if constexpr (requires { val.str(); }) {
          return val.str();
      } else {
          static_assert(false, "Unsupported type");
      }
  }
  ```

### 5. Pitfalls and Gotchas (7-10 min)

This is where the audience learns what *not* to do.

**5a. Concepts only check syntax, not semantics**
- A concept can verify that `a + b` compiles, but it cannot verify that `+` is commutative, associative, or doesn't launch a missile
- The standard concepts (e.g., `std::regular`, `std::totally_ordered`) document *semantic* requirements in prose — the compiler only checks the syntactic part
- Takeaway: concepts are necessary but not sufficient; code review and tests still matter

**5b. Over-constraining**
- Don't require more than your function actually uses
- Example: requiring `std::random_access_iterator` when `std::forward_iterator` would suffice — you've just excluded `std::list` for no reason
- Rule of thumb: constrain to what the body calls, not to what you *think* callers will pass

**5c. Under-constraining**
- If your concept is too lax, wrong types will pass the concept check but fail inside the body — you're back to the wall-of-errors problem
- Balance is key: test your concepts with types that *should* fail

**5d. `requires requires` confusion**
- Explain the difference between a `requires`-clause (a constraint) and a `requires`-expression (a thing that produces a bool)
- When you write `requires requires(T x) { ... }`, the first is the clause, the second starts the expression
- Prefer naming your concepts to avoid this pattern when possible

**5e. Subsumption surprises**
- Only concepts composed via **conjunction (`&&`) and disjunction (`||`) at the concept level** participate in subsumption
- `std::integral<T> && true` does NOT subsume `std::integral<T>` because `true` is not a concept
- If you refactor concept logic into a helper `constexpr bool` function, subsumption breaks
- Keep the logical structure inside concept definitions if you rely on overload ordering

**5f. Compilation cost considerations**
- Concepts can increase compile time if the requires-expressions are complex or deeply nested
- Keep concepts focused and modular; compose small concepts into larger ones

### 6. Summary and Guidelines (3 min)

Distill into actionable advice.

- **Do** use standard library concepts before writing your own
- **Do** name your concepts — prefer named concepts over inline `requires requires`
- **Do** constrain to the minimum your code actually needs
- **Don't** assume concepts enforce semantics — they check syntax only
- **Don't** rely on subsumption with non-concept boolean expressions
- **Do** write negative tests: verify that types you intend to reject are actually rejected
- Concepts are the biggest upgrade to C++ generic programming since templates themselves — they make templates *usable* for non-wizards

### 7. Q&A (remaining time)

---

## Supporting Materials to Prepare

| Material | Purpose |
|---|---|
| Godbolt / Compiler Explorer live links | Show error messages side-by-side (GCC vs Clang, with and without concepts) |
| Small self-contained code samples (compilable) | Each section's example should compile and run or fail as expected |
| "Cheat sheet" handout or final slide | The four syntax forms + standard concept categories |

## Suggested Slide Count

| Section | Estimated Slides |
|---|---|
| 1. The Problem | 3-4 |
| 2. What Are Concepts | 4-5 |
| 3. Why Use Them | 4-5 |
| 4. How to Use Them | 6-8 |
| 5. Pitfalls | 5-7 |
| 6. Summary | 1-2 |
| **Total** | **23-31** |
