<!-- .slide: data-background-image="horsemen.png" -->
# The 4 horsemen of the `const`-pocalyse

- `const`
- `constexpr` <!-- .element: class="fragment" data-fragment-index="0" -->
- `constinit` <!-- .element: class="fragment" data-fragment-index="1" -->
- `consteval` <!-- .element: class="fragment" data-fragment-index="2" -->

---

### `const` ensures variables imutability

```cpp [1-2|4-5|7-9]
const int x = 24; 
const int y; //Error: Must be initialized

const int z = 42;
++z; //Error: Can't be modified

const char * ptr = /*...*/;
char const * ptr = /*...*/;
char * const ptr = /*...*/;
```

---

### `const` ensures object state imutability

```cpp
struct Horsemen{
    bool has_horse_;
    void galop() const{
        has_horse_ = false; //Error: galop is marked const
    }
}
```

---

### `constexpr` is for **compile-time** constant expressions

```cpp [1,3-4|1,6-8|1,10-12]
constexpr int foo(int i) {return 42 + i;}

constexpr int x = 42;       // const is implicit
constexpr auto y = foo(x);

int x = 42;
constexpr auto y = foo(x);  // Error: not a compile-time
                            // constant expression

const int x = 42;
auto y = foo(x);            // Ok, y does not require
                            // compile time expression

```

---

### `constinit` ensures compile time initialization

- Only for `thread` or `static` storage duration
- Can't be used with `constexpr`

```cpp [1-3|5,7-9|5,11-13]
constinit int x = 41;
x += 1;                     // OK: constinit doesn't 
                            // imply const

constexpr int foo(int i) {return 42 + i;}

int x = 42;
constinit auto y = foo(x);  // Error: not a compile-time
                                   // constant expression

constexpr int x = 42;
constinit auto y = foo(x);  //OK
```

---

### `consteval` ensures compile-time evaluation

```cpp [1,3-5|1,7-9|1,10-12]
consteval int foo(int i) {return 42 + i;}

const int x = 42;
constexpr auto y = foo(x);  // Error: not a compile-time
                            // expression

int x = 42;
auto y = foo(x);            // Error: not a compile-time
                            // expression

constexpr int x = 42;
auto y = foo(x);            // OK
```