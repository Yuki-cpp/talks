<!-- .slide: data-background-image="horsemen.png" -->
## The 4 horsemen of the `const`-pocalyse

- `const`     <!-- .element: class="fragment" data-fragment-index="0" -->
- `constexpr` <!-- .element: class="fragment" data-fragment-index="1" -->
- `constinit` <!-- .element: class="fragment" data-fragment-index="2" -->
- `consteval` <!-- .element: class="fragment" data-fragment-index="3" -->

---

### `constexpr` is for values that can appear in constant expressions

- **Possible** to evaluate at compile time
- Implies `const`
- Implies `inline`

------

### `constexpr`

```cpp [1-5|1-7|1-2,9-10|1-2,12-13]
// foo CAN be evaluated at compile time
constexpr auto foo(int i) {return 42 + i;}

// x MUST be evaluated at compile time
constexpr int x = 42;

constexpr auto y = foo(x);  // OK

int a = 42;                 // Error: not a compile-time
constexpr auto b = foo(a);  // constant expression

int i = 42;                 // Ok, y does not require
auto j = foo(i);            // compile time expression

```

---

### `consteval` is for functions that must produce compile-time constants

- **Must** to evaluate at compile time
- Can't specify both `constexpr` and `consteval`
- Implies constexpr <!-- .element: class="fragment" data-fragment-index="0" -->

------

### `consteval`

```cpp [1-5|1-2,7-8|1-2,10-11]
// foo MUST be evaluated at compile time
consteval auto foo(int i) {return 42 + i;}

constexpr int x = 42;
constexpr auto y = foo(x);  // OK

int a = 42;                 // Error: not a compile-time
constexpr auto b = foo(a);  // constant expression

int i = 42;                 // Error: not a compile-time
auto j = foo(i);            // constant expression

```
