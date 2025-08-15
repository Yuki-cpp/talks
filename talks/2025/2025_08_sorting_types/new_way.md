## Can we do better?
# Can we do simpler? <!-- .element: class="fragment" data-fragment-index="0" -->

---

<!-- .slide: data-background-image="elmo_eureka.png" -->
### A new approach

1. Convert each UnitComponent type into a value
2. Convert an Unit into an array of values
3. Sort/Process the array via `consteval` functions
4. Convert the array back into a new Unit type

---

### Starting by the end: The sort function

```cpp [1-17|7-15]
// Pair<uid, power>
using UnitComponentPair = std::pair<std::size_t, int>;

template <size_t N>
consteval auto sort_units(std::array<UnitComponentPair, N> units)
{
    std::ranges::sort(units, [](const auto &a, const auto &b)
                      {
                        if(b.second == 0){
                            return true;
                        }
                        if(a.second == b.second){
                            return a.first < b.first;
                        }
                        return a.second > b.second; });
    return units;
}
```


---

### The making of `UnitComponentPair`

```cpp [1-5|7-11]
// Pair<uid, power>
using UnitComponentPair = std::pair<std::size_t, int>;

template <IsUnitComponent... Ts>
struct Unit{/**/};

template <IsUnitComponent... Ts>
consteval auto serialize_units(std::tuple<Ts...>)
{
    return std::to_array<UnitComponentPair>({{Ts::tag::priority, Ts::power}...});
}
```
 <!-- .element: class="full-bleed" -->

---

### How do we go back? (1/2)

```cpp [1-7|9-14]
template <UnitComponentPair unit>
consteval auto deserialize_unit()
{
    constexpr auto priority = unit.first;
    constexpr auto power = unit.second;
    return UnitComponent<decltype(unit_tag_from_priority<priority>()), power>();
}

struct m
{static constexpr std::size_t priority = 0;};
template <>
consteval auto unit_tag_from_priority<0>()
{return m();}
```

---

### How do we go back? (2/2)

We need to write the C++ compile time equivalent of:
```python
return tuple(*type_list)
```

- `tuple(...)` ↔️ `std::make_tuple(...)`

- `*type_list` ↔️ Pack expansion
    - Need a pack to expand

------

```cpp [1-10]
template <std::size_t N, std::array<UnitComponentPair, N> units, std::size_t... Is>
consteval auto deserialize_units(std::integer_sequence<std::size_t, Is...>)
{
    return std::make_tuple(deserialize_unit<units[Is]>()...);
}
```
<!-- .element: class="full-bleed" -->

```cpp [1-10]
template <std::size_t N, std::array<UnitComponentPair, N> units>
consteval auto deserialize_units()
{
    return deserialize_units<units.size(), units>(std::make_index_sequence<N>());
}
```
<!-- .element: class="fragment full-bleed" data-fragment-index="0" -->

---

### Putting things together

```cpp [1-11|13-25]
template <IsUnitComponent... Ts>
consteval auto canonicalize_units(std::tuple<Ts...> units)
{
    constexpr auto serialized = serialize_units(units);
    constexpr auto merged = reduce_duplicated_units<serialized.size(), 
                                                    serialized>();
    constexpr auto sorted = sort_units(merged);
    constexpr auto filtered = remove_null_powers<sorted.size(), 
                                                 sorted>();
    return deserialize_units<filtered.size(), filtered>();
}

static auto canonical(double value)
{
    return from_tuple_t<Unit,
                        decltype(canonicalize_units(
                            std::tuple<Ts...>()))>(value);
}
```
<!-- .element: class="full-bleed tall" -->