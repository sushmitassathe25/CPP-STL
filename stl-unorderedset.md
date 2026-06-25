# UNORDERED_SET

`std::unordered_set` is an associative container from the C++ Standard Library that contains a collection of unique objects of type `Key`. Unlike `std::set`, the elements are not sorted alphabetically or numerically. Instead, they are organized into buckets based on their hash values. This structure allows for incredibly fast retrieval, insertion, and removal of elements, typically executing in constant average time complexity ($O(1)$).

**Header:** `<unordered_set>`

**Template:** 
```cpp
template<
    class Key,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator<Key>
> class unordered_set;
```

![unordered set](unordered%20set.jpeg)

## High-level characteristics

- **Hash Table Structure**: Elements are not arranged in any sequential or sorted order. Their physical memory location is determined mathematically by a hash function.
- **Constant Time Average Performance**: Lookups, insertions, and deletions scale uniformly, staying blazing fast regardless of the container's size, assuming a high-quality hash function.
- **Uniqueness strictly enforced**: A `std::unordered_set` cannot contain duplicate values. If you attempt to insert a value that already exists, the operation is ignored. (For duplicates, use `std::unordered_multiset`).
- **Immutable keys**: The values inside the set are the keys, and they are locked as `const`. Modifying an element directly would change its hash value and corrupt the internal bucket mapping.


## How it works internally

Internally, `std::unordered_set` is implemented as a Hash Table using Separate Chaining to handle collisions.

- **Bucket Array**: The container manages a dynamic underlying array of "buckets."
- **The Hash Function**: When you insert a value (e.g., the number 42 or the string "Apple"), `std::hash` mathematically converts it into a large integer (a hash code). The container applies a modulo operation to this hash code to assign the element to a specific bucket index.
- **Separate Chaining (Linked Lists)**: If two different values hash to the exact same bucket (a "Hash Collision"), the container stores both elements in a singly or doubly linked list attached to that specific bucket.
- **Rehashing**: As the set fills up (monitored by the `load_factor`), the container will automatically allocate a much larger array of buckets and recalculate the placement of every single element to maintain its $O(1)$ collision-free access times.


**Exception safety**:
- Provides strong exception guarantees for single-element insertions. If rehashing fails due to a memory allocation error (`std::bad_alloc`), the set's original state is preserved flawlessly.

## Complexity guarantees

| Operation | Average Complexity | Worst-Case Complexity (Severe Collisions) |
|-----------|-----------|---------------------------------------------------|
| Lookup (`find, count, contains`) | O(1) | O(N) |
| Insertion (`insert, emplace`) | O(1) | O(N) |
| Erasure by key or iterator | O(1) | O(N) |
| `size`, `empty` | O(1) | O(1) |
| `clear` | O(N) | O(N) |


## Member functions and operators

### Constructors

```cpp
unordered_set();                                    // (1) empty set
explicit unordered_set( size_type bucket_count );   // (2) empty set pre-allocated with at least bucket_count buckets
template< class InputIt >
unordered_set( InputIt first, InputIt last );       // (3) range [first, last)
unordered_set( const unordered_set& other );        // (4) copy constructor
unordered_set( unordered_set&& other ) noexcept;    // (5) move constructor
unordered_set( std::initializer_list<value_type> init ); // (6) initializer list
```


**Examples:**
```cpp
std::unordered_set<std::string> us1;                // empty
std::unordered_set<int> us2 = {1, 2, 3, 2, 1};      // {1, 2, 3} (duplicates dropped)
std::unordered_set<int> us3(10000);                 // instantly reserves 10,000 buckets to avoid rehashing
```

### Destructor

```cpp
~unordered_set(); // Destroys all nodes and frees heap allocations
```


### Element access

Because a `std::unordered_set` only contains keys and is not sorted, it does NOT provide `operator[], .at(), .front(), or .back()`. Elements must be accessed via Iterators or Lookup functions.

### Iterators

Because `std::unordered_set` is not sorted, iterators traverse the container in a seemingly random, unpredictable order based on the internal memory layout of the buckets.

```cpp
iterator begin() noexcept;                          // iterator to the first element
iterator end() noexcept;                            // iterator to end (one-past-last element)
const_iterator cbegin() const noexcept;
const_iterator cend() const noexcept;
```

*(Note: `unordered_set` does not provide reverse iterators like `rbegin()` because traversing backwards through an unsequenced hash table is generally meaningless).*

### Capacity and Hash Policy (Specialized Methods)

```cpp
bool empty() const noexcept;                        // checks if size == 0
size_type size() const noexcept;                    // number of unique elements

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
```


#### erase() — Remove elements

```cpp
iterator erase( const_iterator pos );                 // erase element at iterator (O(1) average)
iterator erase( const_iterator first, const_iterator last ); // erase range
size_type erase( const Key& key );                    // erase by value (O(1) average), returns 1 if erased, 0 if not
```


#### extract() and merge() (C++17) 

```cpp
node_type extract( const key_type& x );               // unlinks a node from the set without destroying memory
void merge( unordered_set& source );                  // moves nodes from another unordered_set into this one
```

#### Lookup

```cpp
size_type count( const Key& key ) const;              // returns 1 if found, 0 if not
iterator find( const Key& key );                      // returns iterator to element, or end() if not found
bool contains( const Key& key ) const;                // (C++20) returns true if key exists
```


## Iterator and reference invalidation rules

Because `std::unordered_set` relies on a dynamic bucket array holding pointers to nodes, its invalidation rules strongly favor references over iterators:

| Operation | Iterator Invalidation | Pointer/Reference Invalidation |
|-----------|---|-------------------------------|
| `insert / emplace` | Invalidated ONLY IF the insertion triggers a rehash. Otherwise, none. | None. Pointers and references to elements are NEVER invalidated by insertions or rehashing. |
| `rehash / reserve` | All iterators are invalidated. | None. |
| `erase` | Only iterators to the erased elements are invalidated. | Only pointers/references to the erased elements are invalidated. |
| `clear` / Destruction | All iterators are invalidated. | All are invalidated. |    


**Key takeaway:** Rehashing completely shuffles the bucket array, breaking all active iterators. However, the physical memory nodes holding your actual values do not move, meaning references and pointers to your data remain perfectly safe and stable.

## Typical pitfalls and best practices

1. **Custom Object Hashing**: By default, `std::unordered_set` only knows how to hash fundamental types (`int, double, std::string`). If you want to store a custom `struct or class`, you MUST provide a custom hash function injection (`std::hash<MyStruct>`) and an equality operator (`operator==`).

2. **Rehashing Performance Spikes**: If you `push` 100,000 items into an unordered_map, it will lag intermittently as it pauses to reallocate its bucket array. If you know the size in advance, always call `um.reserve(100000)` on initialization to bypass this massive performance penalty.

3. **Worst-Case $O(N)$ Traps**: If a bad actor knows your hash function, or you write a terrible custom hash function (e.g., returning the same integer for everything), every single key will collide into the exact same bucket. The Hash Table devolves into a single Linked List, turning fast $O(1)$ lookups into crippling $O(N)$ linear scans.

4. **Meaningless Iteration Order**: Never write logic that relies on the sequence of elements returned by iterating over an `std::unordered_set`. The order appears random and will completely change the moment the container decides to rehash.


## Common idioms and patterns

### Lightning-Fast Existence Checking

Unordered sets are the absolute best choice when you frequently need to check "Have I seen this item before?"

```cpp
std::unordered_set<std::string> blocked_ips = {"192.168.1.1", "10.0.0.5"};

bool is_allowed(const std::string& ip) {
    // C++20 .contains() performs an O(1) existence check
    return !blocked_ips.contains(ip); 
}
```

### Fast Deduplication without Sorting

If you need to remove duplicates from a vector but don't care about the final sorted order, dumping it into an `unordered_set` is much faster than `std::sort + std::unique`:

```cpp
std::vector<int> v = {4, 1, 4, 2, 8, 1, 2};

// Dump into unordered_set to strip duplicates in O(N) total time
std::unordered_set<int> unique_vals(v.begin(), v.end());

// Assign back to vector
v.assign(unique_vals.begin(), unique_vals.end());
// v is now {8, 2, 1, 4} (Exact order may vary)
```


## Real-world use cases

- **Graph Algorithms (BFS/DFS)**: Maintaining a "Visited" set to track which nodes have already been processed to prevent infinite loops in cyclic graphs.

- **Spell Checkers / Dictionaries**: Loading an entire English dictionary into memory to instantly check if a typed word is spelled correctly in $O(1)$ time.

- **Unique Identifier Tracking**: Ensuring user IDs, transaction hashes, or session tokens generated by a system are completely unique before committing them to a database.

- **Database Query Filtering**: Quickly filtering out items that match a massive exclusion list without writing complex nested loops.


## Useful headers and related features

| Header | Functionality |
|--------|---|
| `<unordered_set>` | Provides `std::unordered_set` and `std::unordered_multimset` |
| `<set>` | Red-Black Tree equivalent (slower $O(\log N)$ lookups, but keeps data sorted) |
| `<functional>` | Provides `std::hash` used by the container internally |


## Full example program

```cpp
#include <iostream>
#include <unordered_set>
#include <string>
#include <vector>

int main() {
    // 1. Initialization
    std::unordered_set<std::string> active_sessions = {
        "user_alice_99",
        "user_bob_12",
        "user_charlie_42"
    };

    // 2. Fast O(1) Lookup (C++20 .contains)
    std::string incoming_request = "user_bob_12";
    if (active_sessions.contains(incoming_request)) {
        std::cout << "Access Granted for: " << incoming_request << "\n\n";
    }

    // 3. Safe Insertion
    // "user_alice_99" already exists, so insertion is ignored.
    auto [it, success] = active_sessions.insert("user_alice_99");
    if (!success) {
        std::cout << "Duplicate blocked: " << *it << " is already logged in.\n\n";
    }

    // 4. Pre-allocating buckets for a massive influx of data
    std::cout << "Current bucket count: " << active_sessions.bucket_count() << '\n';
    active_sessions.reserve(500); // We expect 500 new logins, reserve space now
    std::cout << "Bucket count after reserve(500): " << active_sessions.bucket_count() << "\n\n";

    // 5. Fast Deduplication of incoming raw data
    std::vector<int> raw_sensor_data = {101, 105, 101, 102, 105, 109, 101};
    std::unordered_set<int> unique_data(raw_sensor_data.begin(), raw_sensor_data.end());
    
    std::cout << "--- Processed Sensor Data (Unordered) ---\n";
    for (int reading : unique_data) {
        std::cout << "Reading: " << reading << '\n';
    }

    return 0;
}
```

**Output:**
```
Access Granted for: user_bob_12

Duplicate blocked: user_alice_99 is already logged in.

Current bucket count: 8
Bucket count after reserve(500): 503

--- Processed Sensor Data (Unordered) ---
Reading: 109
Reading: 102
Reading: 105
Reading: 101
```
*(Note: Because std::unordered_set uses hash tables, the exact order of the Processed Sensor Data and the bucket metrics may vary slightly depending on your specific compiler's hashing algorithm).*

---





