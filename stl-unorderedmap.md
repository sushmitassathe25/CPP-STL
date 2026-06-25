# UNORDERED_MAP

`std::unordered_map` is an associative container from the C++ Standard Library that contains key-value pairs with unique keys. Unlike `std::map`, elements are not sorted by their keys. Instead, they are organized into buckets depending on the hash values of the keys. This allows for incredibly fast retrieval, insertion, and removal of individual elements, typically running in constant average time complexity ($O(1)$). 
  
**Header:** `<unordered_map>`

**Template:** 
```cpp
template<
    class Key,
    class T,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator< std::pair<const Key, T> >
> class unordered_map;
```

![unordered map](unordered%20map.jpeg)

## High-level characteristics

- **Key-Value association**: Every element stored is a `std::pair<const Key, T>`.
- **Hash Table Structure**: Elements are not arranged in any sequential or alphabetical order. Their physical memory location is determined mathematically by a hash function.
- **Constant Time Average Performance**: Lookups and modifications scale uniformly, staying blazing fast regardless of whether the map has 10 elements or 10 million elements, assuming a good hash function.
- **Uniqueness strictly enforced**: A `std::unordered_map` cannot contain duplicate keys. For duplicate keys, use `std::unordered_multimap`.
- **Immutable keys**: The key (`Key`) is locked as `const`. Modifying a key would corrupt its calculated hash and bucket placement.


## How it works internally

Internally, `std::unordered_map` is implemented as a Hash Table using Separate Chaining for collision resolution.

- **Bucket Array**: The map allocates an underlying dynamic array of "buckets".
- **The Hash Function**: When you insert a key, `std::hash` mathematically converts the key (e.g., the string "Alice") into a massive integer (a hash code). The map uses the modulo operator on this hash code to assign the element to a specific bucket index.
- **Separate Chaining (Linked Lists)**: If two different keys accidentally hash to the exact same bucket (a "Hash Collision"), the map stores both of them in a singly or doubly linked list attached to that bucket.
- **Rehashing**: As the map gets crowded (tracked by the `load_factor`), the map will automatically allocate a much larger array of buckets and recalculate the placement of every single element to maintain $O(1)$ collision-free access.


**Exception safety**:
- Provides strong exception guarantees for single element insertions. If rehashing fails due to `std::bad_alloc`, the map's original state is preserved perfectly.

## Complexity guarantees

| Operation | Average Complexity | Worst-Case Complexity (Severe Collisions) |
|-----------|-----------|---------------------------------------------------|
| Lookup (`at, operator[], find, count`) | O(1) | O(N) |
| Insertion (`insert, emplace, operator[]`) | O(1) | O(N) |
| Erasure by key or iterator | O(1) | O(N) |
| `size`, `empty` | O(1) | O(1) |
| `clear` | O(N) | O(N) |


## Member functions and operators

### Constructors

```cpp
unordered_map();                                    // (1) empty map
explicit unordered_map( size_type bucket_count );   // (2) empty map pre-allocated with at least bucket_count buckets
template< class InputIt >
unordered_map( InputIt first, InputIt last );       // (3) range [first, last)
unordered_map( const unordered_map& other );        // (4) copy constructor
unordered_map( unordered_map&& other ) noexcept;    // (5) move constructor
unordered_map( std::initializer_list<value_type> init ); // (6) initializer list
```


**Examples:**
```cpp
std::unordered_map<std::string, int> um1;           // empty
std::unordered_map<std::string, int> um2 = {        // initializer list
    {"Alice", 25}, 
    {"Bob", 30}
};
std::unordered_map<int, std::string> um3(1000);     // reserves 1000 buckets instantly to avoid rehashing overhead
```

### Destructor

```cpp
~unordered_map(); // Destroys all nodes and frees heap allocations
```


### Element access

```cpp
T& at( const Key& key );                            // bounds-checked access, throws std::out_of_range if not found
const T& at( const Key& key ) const;

T& operator[]( const Key& key );                    // returns reference to value. INSERTS default-constructed value if key doesn't exist
T& operator[]( Key&& key );
```

### Iterators

Because `std::unordered_map` is not sorted, iterators traverse the container in a seemingly random, unpredictable order based on internal memory layout.

```cpp
iterator begin() noexcept;                          // iterator to the first element
iterator end() noexcept;                            // iterator to end (one-past-last element)
const_iterator cbegin() const noexcept;
const_iterator cend() const noexcept;
```

*(Note: `unordered_map` does not provide reverse iterators like `rbegin()` because traversing backwards through an unsequenced hash table is generally meaningless).*

### Capacity and Hash Policy (Specialized Methods)

```cpp
bool empty() const noexcept;                        // checks if size == 0
size_type size() const noexcept;                    // number of key-value pairs

// Hash Policy Methods
float load_factor() const;                          // Average number of elements per bucket (size / bucket_count)
float max_load_factor() const;                      // Maximum load factor before automatic rehashing occurs
void max_load_factor( float ml );                   // Sets the maximum load factor trigger
void rehash( size_type count );                     // Forces the bucket array to resize to at least 'count'
void reserve( size_type count );                    // Sets bucket size to accommodate 'count' elements without rehashing
```


### Modifiers

#### insert() / emplace() — Insert elements

```cpp
std::pair<iterator, bool> insert( const value_type& value );  
template< class... Args >
std::pair<iterator, bool> emplace( Args&&... args );

// C++17 advanced insertions:
template <class... Args>
std::pair<iterator, bool> try_emplace(const Key& k, Args&&... args); // Constructs value ONLY if key is missing
template <class M>
std::pair<iterator, bool> insert_or_assign(const Key& k, M&& obj);   // Inserts if missing, overwrites if exists
```


#### erase() — Remove elements

```cpp
iterator erase( const_iterator pos );                 // erase element at iterator (O(1) average)
iterator erase( const_iterator first, const_iterator last ); // erase range
size_type erase( const Key& key );                    // erase by key (O(1) average), returns 1 if erased, 0 if not
```


#### extract() and merge() (C++17) 

```cpp
node_type extract( const key_type& x );               // unlinks a node from the map without destroying memory
void merge( unordered_map& source );                  // moves nodes from another unordered_map into this one
```

#### Lookup

```cpp
size_type count( const Key& key ) const;              // returns 1 if found, 0 if not
iterator find( const Key& key );                      // returns iterator to element, or end() if not found
bool contains( const Key& key ) const;                // (C++20) returns true if key exists
```


## Iterator and reference invalidation rules

Because `std::unordered_map` relies on a dynamic bucket array holding pointers to nodes, its invalidation rules are slightly different from `std::map`:

| Operation | Iterator Invalidation | Pointer/Reference Invalidation |
|-----------|---|-------------------------------|
| `insert / emplace / operator[]` | Invalidated ONLY IF the insertion triggers a rehash. Otherwise, none. | None. Pointers and references to elements are NEVER invalidated by insertions or rehashing.
| `rehash / reserve` | All iterators are invalidated. | None. |
| `erase` | Only iterators to the erased elements are invalidated. | Only pointers/references to the erased elements are invalidated. |
| `clear` / Destruction | All iterators are invalidated. | All are invalidated. |    


**Key takeaway:** Rehashing completely shuffles the bucket array, breaking all active iterators. However, the physical memory nodes holding your actual key-value pairs do not move, meaning references and pointers to your data remain perfectly safe and stable.

## Typical pitfalls and best practices

1. **Custom Object Keys**: Out of the box, `std::unordered_map` only knows how to hash fundamental types (int, double, std::string). If you want to use a custom struct/class as a Key, you MUST provide a custom hash function injection (`std::hash<MyStruct>`) and an equality operator (`operator==`).

2. **Rehashing Performance Spikes**: If you `push` 100,000 items into an unordered_map, it will lag intermittently as it pauses to reallocate its bucket array. If you know the size in advance, always call `um.reserve(100000)` on initialization to bypass this massive performance penalty.

3. **Worst-Case $O(N)$ Traps**: If an attacker knows your hash function, or you write a terrible custom hash function (e.g., everything returns 1), every single key will collide into the exact same bucket. The Hash Table devolves into a single Linked List, turning $O(1)$ lookups into crippling $O(N)$ lookups.

4. **Order is fundamentally meaningless**: Never write logic that relies on the sequence of elements returned by iterating over an `std::unordered_map`. The order will arbitrarily change the moment the map decides to rehash.


## Common idioms and patterns

### Extremely Fast Memoization (Caching)

Unordered maps are the gold standard for caching expensive function results:

```cpp
std::unordered_map<int, long long> fib_cache;

long long fibonacci(int n) {
    if (n <= 1) return n;
    
    // Check if we already calculated this value in O(1) time
    if (fib_cache.contains(n)) {
        return fib_cache[n];
    }
    
    // Calculate, store, and return
    long long result = fibonacci(n - 1) + fibonacci(n - 2);
    fib_cache[n] = result;
    return result;
}
```

### Pre-Allocating Capacity

```cpp
std::unordered_map<std::string, int> fast_map;
// We know we are about to load 1 million rows from a database.
// Pre-reserve the buckets to prevent expensive rehashing lag.
fast_map.reserve(1000000);
```


## Real-world use cases

- **Database Indexing**: Building fast, in-memory lookup indexes where finding a row by a unique ID string must be instantaneous.

- **Game Engines**: Managing resource pools (e.g., mapping "hero_sprite.png" to an active memory Texture object) for rapid $O(1)$ rendering lookups.
  
- **Symbol Tables in Compilers**: Tracking variable names and their current scope/type data during source code parsing.

- **Memoization**: Storing previously computed results of expensive math or recursive algorithms.


## Useful headers and related features

| Header | Functionality |
|--------|---|
| `<unordered_set>` | Provides `std::unordered_map` and `std::unordered_multimap` |
| `<map>` | Red-Black Tree equivalent (slower $O(\log N)$ lookups, but keeps data sorted) |
| `<functional>` | Provides `std::hash` used by the container internally |


## Full example program

```cpp
#include <iostream>
#include <unordered_map>
#include <string>

int main() {
    // 1. Initialization
    std::unordered_map<std::string, double> product_prices = {
        {"Laptop", 1200.50},
        {"Mouse", 25.99},
        {"Keyboard", 75.00}
    };

    // 2. Fast O(1) Lookup (C++20 .contains)
    std::string cart_item = "Mouse";
    if (product_prices.contains(cart_item)) {
        std::cout << cart_item << " costs $" << product_prices.at(cart_item) << "\n\n";
    }

    // 3. The operator[] behavior
    // "Monitor" does not exist. It is created with a default double (0.0), then assigned.
    product_prices["Monitor"] = 300.00; 

    // 4. Safe Insertion (try_emplace - C++17)
    // "Laptop" exists, so try_emplace instantly rejects the update without allocating memory.
    auto [it, success] = product_prices.try_emplace("Laptop", 500.00);
    if (!success) {
        std::cout << "Failed to update Laptop price. It is still $" << it->second << "\n\n";
    }

    // 5. Inspecting internal hash table metrics
    std::cout << "--- Hash Table Diagnostics ---\n";
    std::cout << "Total Products: " << product_prices.size() << '\n';
    std::cout << "Bucket Array Size: " << product_prices.bucket_count() << '\n';
    std::cout << "Current Load Factor: " << product_prices.load_factor() << '\n';
    std::cout << "Max Load Factor Limit: " << product_prices.max_load_factor() << "\n\n";

    // 6. Iteration (Notice the output order is effectively random!)
    std::cout << "--- Inventory List ---\n";
    for (const auto& [product, price] : product_prices) {
        std::cout << product << ": $" << price << '\n';
    }

    return 0;
}
```

**Output:**
```
Mouse costs $25.99

Failed to update Laptop price. It is still $1200.5

--- Hash Table Diagnostics ---
Total Products: 4
Bucket Array Size: 8
Current Load Factor: 0.5
Max Load Factor Limit: 1

--- Inventory List ---
Monitor: $300
Keyboard: $75
Laptop: $1200.5
Mouse: $25.99
```
*(Note: Because std::unordered_map uses hash tables, the exact order of the Inventory List and the bucket metrics may vary slightly depending on your specific compiler's hashing algorithm, perfectly illustrating the "unordered" nature of the container!)*

---




