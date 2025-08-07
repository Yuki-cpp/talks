
# The `decl` duo

- `decltype`      <!-- .element: class="fragment" data-fragment-index="0" -->
- `std::declval`  <!-- .element: class="fragment" data-fragment-index="1" -->

---

### `decltype` gets the type of an expression

- Creates unevaluated context
- Returns the type of provided expression

------

### `decltype`

```cpp [1-5| 1-5,8-9|1-5,11-12]
template <bool b>
auto foo() {
  if constexpr (b) {return 42;} 
  else {return "42";}
}

int main() {
  decltype(foo<true>()) i = 42;       // OK
  decltype(foo<true>()) s = "Hello";  // Error

  decltype(foo<false>()) s = "Hello"; // OK
  decltype(foo<false>()) i = 42;      // Error
}
```

---

### `std::declval` makes a value from a type

- Returns a value of the given type
- Only usable in unevaluated contexts

------

### `declval`

```cpp [1-2|4-9|6,8|7|11-12]
template<typename T>
concept HasF = requires(T t){t.f();};

template<HasF T>
auto default_f_result(){
  return decltype(
    std::declval<T>().f()
  ){};
}

struct A{bool f(){return true;}};
auto val = default_f_result<A>(); // val is false
```