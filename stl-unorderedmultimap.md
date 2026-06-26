# UNORDERED_MULTIMAP

`std::unordered_multimap` is an associative container from the C++ Standard Library that contains key-value pairs. Unlike `std::unordered_map`, **it allows multiple elements to have the exact same key**. Elements are not sorted by their keys; instead, they are organized into buckets depending on the hash values of the keys, allowing for constant average time complexity ($O(1)$) operations.

**Header:** `<unordered_map>`

**Template:** 
```cpp
template<
    class Key,
    class T,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator< std::pair<const Key, T> >
> class unordered_multimap;
```

![unordered multimap](unordered%20multimap.jpeg)

## High-level characteristics

- **Duplicate Keys Allowed**: You can insert multiple identical keys, each mapping to the same or different values.
- **Key-Value association**: Every element stored is a `std::pair<const Key, T>`.
- **Hash Table Structure**: Elements are physically placed in memory based on a mathematical hash function, completely ignoring sequential or sorted ordering.
- **Constant Time Average Performance**: Lookups, insertions, and deletions generally take $O(1)$ time, assuming a well-distributed hash function.
- **Immutable keys**: The key (`Key`) is locked as `const`. Modifying a key directly would corrupt its hash value and break the container's bucket mapping.

## How it works internally

Internally, `std::unordered_multimap` uses a Hash Table with Separate Chaining.

- **Bucket Array**: The container manages a dynamic array of "buckets."
- **Hashing and Grouping**: When a value is inserted, `std::hash` generates a hash code, and modulo arithmetic assigns it to a bucket. Crucially, the C++ standard guarantees that equivalent elements (duplicates) are always placed in the same bucket and are strictly adjacent to one another when iterating.
- **Separate Chaining**: Hash collisions (different values sharing the same bucket) and duplicate values are managed by linking the elements together in a linked list attached to the bucket.
- **Rehashing**: To maintain $O(1)$ performance, the table automatically grows and re-distributes elements (rehashes) when the `load_factor` exceeds the `max_load_factor`.  

**Exception safety**:
- Provides strong exception guarantees for single-element insertions. If rehashing fails due to `std::bad_alloc`, the multimap remains unchanged.

## Complexity guarantees

| Operation | Average Complexity | Worst-Case Complexity (Severe Collisions) |
|-----------|-----------|---------------------------------------------------|
| Lookup (`find, count, contains`) | O(1) | O(N) |
| Insertion (`insert, emplace`) | O(1) | O(N) |
| Erasure by key | O(count(key)) | O(N) |
| Erasure by iterator | O(1) | O(N) |
| `size`, `empty` | O(1) | O(1) |
| `clear` | O(N) | O(N) |


## Member functions and operators

### Constructors

```cpp
unordered_multimap();                                    // (1) empty multimap
explicit unordered_multimap( size_type bucket_count );   // (2) pre-allocated with at least bucket_count buckets
template< class InputIt >
unordered_multimap( InputIt first, InputIt last );       // (3) range [first, last)
unordered_multimap( const unordered_multimap& other );        // (4) copy constructor
unordered_multimap( unordered_multimap&& other ) noexcept;    // (5) move constructor
unordered_multimap( std::initializer_list<value_type> init ); // (6) initializer list
```


**Examples:**
```cpp
std::unordered_multimap<std::string, int> umm1;          // empty
std::unordered_multimap<std::string, int> umm2 = {       // initializer list
    {"Apple", 10}, 
    {"Apple", 15}, 
    {"Banana", 5}
}; // Apple has two distinct values mapped to it!
```

### Destructor

```cpp
~unordered_multimap(); // Destroys all nodes and frees heap allocations
```


### Element access

**CRITICAL DIFFERENCE**: `std::unordered_multimap` does NOT provide `operator[] or .at()`. Because a single key can map to multiple values, it is impossible for `map["Apple"]` to know which of the multiple values it should return. You must access values using iterators via `find() or equal_range()`.

### Iterators

Iterators traverse the container in an unpredictable order, except that pairs with identical keys are guaranteed to be adjacent.

```cpp
iterator begin() noexcept;                          // iterator to the first element
iterator end() noexcept;                            // iterator to end (one-past-last element)
const_iterator cbegin() const noexcept;
const_iterator cend() const noexcept;
```


### Capacity and Hash Policy (Specialized Methods)

```cpp
bool empty() const noexcept;                        // checks if size == 0
size_type size() const noexcept;                    // total number of key-value pairs

// Hash Policy Methods
float load_factor() const;                          // Average number of elements per bucket
float max_load_factor() const;                      // Maximum load factor before automatic rehashing
void rehash( size_type count );                     // Forces the bucket array to resize
void reserve( size_type count );                    // Reserves buckets to accommodate 'count' elements
```


### Modifiers

#### insert() / emplace() — Insert elements

```cpp
iterator insert( const value_type& value );                   // ALWAYS succeeds (returns iterator to inserted element)
template< class... Args >
iterator emplace( Args&&... args );
```

*(Note: Unlike `unordered_map`, `insert` on a multiset does not return a `std::pair<iterator, bool>` because insertion of duplicates can never fail/be rejected).*

#### erase() — Remove elements

```cpp
iterator erase( const_iterator pos );                 // erase a SINGLE element at iterator (O(1) average)
iterator erase( const_iterator first, const_iterator last ); // erase a range
size_type erase( const Key& key );                    // erase ALL key-value pairs matching 'key'
```


#### extract() and merge() (C++17) 

```cpp
node_type extract( const key_type& x );               // unlinks a single node matching x from the multimap
void merge( unordered_multimap& source );             // moves nodes from another unordered_multimap into this one
```

#### Lookup

Because duplicates exist, lookup functions often focus on finding ranges or counts.

```cpp
size_type count( const Key& key ) const;              // returns the number of times 'key' appears
iterator find( const Key& key );                      // returns an iterator to ONE of the key-value pairs matching 'key'
bool contains( const Key& key ) const;                // (C++20) returns true if at least one 'key' exists

std::pair<iterator,iterator> equal_range( const Key& key ); // Returns a range containing ALL key-value pairs matching 'key'
```


## Iterator and reference invalidation rules
  

| Operation | Iterator Invalidation | Pointer/Reference Invalidation |
|-----------|---|-------------------------------|
| `insert / emplace` | Invalidated ONLY IF the insertion triggers a rehash. Otherwise, none. | None. Pointers and references remain valid. |
| `rehash / reserve` | All iterators are invalidated. | None. |
| `erase` | Only iterators to the erased elements are invalidated. | Only pointers/references to the erased elements are invalidated. |
| `clear` / Destruction | All iterators are invalidated. | All are invalidated. |    


## Typical pitfalls and best practices

1. **The `erase(key)` trap:** Calling `ums.erase(5)` will remove every single instance of the number 5 from the multiset. If you only want to remove one instance, you must find an iterator first and erase that specific iterator: `ums.erase(ums.find(5));`.

2. **`unordered_multimap<K, V>` vs `unordered_map<K, vector<V>>`**:
  - If you frequently need to look up a key and iterate over all its associated values, `std::unordered_map<K, std::vector<V>>` is often much faster and more cache-friendly than `std::unordered_multimap`.
  - `std::unordered_multimap` shines when you are constantly inserting/erasing individual pairs on the fly and don't want the overhead of resizing dynamic arrays inside a map.

3. **`No operator[]`**: Do not try to use `map[key] = value`. Use `map.insert({key, value})`.
  


## Common idioms and patterns

### Processing all values for a single key

Because a key can map to many values, `equal_range` is the primary way to interact with an `unordered_multimap`:

```cpp
std::unordered_multimap<std::string, std::string> dns_records = {
    {"example.com", "192.168.1.1"},
    {"example.com", "10.0.0.5"}
};

// Get the boundaries for all IP addresses mapped to "example.com"
auto [range_start, range_end] = dns_records.equal_range("example.com");

for (auto it = range_start; it != range_end; ++it) {
    std::cout << "IP: " << it->second << "\\n";
}
```

## Real-world use cases

- **DNS Resolution Tables**: A single domain name (the Key) can resolve to multiple different IP addresses (the Values).

- **GDictionary / Thesaurus**: A single word can have multiple distinct definitions or synonyms.

- **Event Subscriber Systems**: Mapping an event type (Key: `"ON_CLICK"`) to multiple different callback functions or observer objects (Values).

- **Game Engine Entity Component Systems (ECS)**: Mapping a specific Entity ID to multiple different Component types if an entity can hold multiple components of the same base classification.


## Useful headers and related features

| Header | Functionality |
|--------|---|
| `<unordered_map>` | Provides `std::unordered_map` and `std::unordered_multimmap  ` |
| `<map>` | Provides `std::multimap` (allows duplicates, but keeps data strictly sorted in $O(\log N)$ time) |


## Full example program

```cpp
#include <iostream>
#include <unordered_map>
#include <string>

int main() {
    // 1. Initialization (Duplicate keys are retained)
    std::unordered_multimap<std::string, std::string> tasks = {
        {"Alice", "Fix server bug"},
        {"Bob", "Update documentation"},
        {"Alice", "Review pull request"},
        {"Charlie", "Deploy to production"}
    };

    // 2. Safe Insertion (No overwriting, just appending)
    tasks.insert({"Bob", "Write unit tests"});

    // 3. Checking quantities
    std::cout << "Total active tasks: " << tasks.size() << '\\n';
    std::cout << "Alice's task count: " << tasks.count("Alice") << "\\n\\n";

    // 4. Using equal_range to get all values for a specific key
    std::cout << "--- Alice's To-Do List ---\\n";
    auto [alice_start, alice_end] = tasks.equal_range("Alice");
    for (auto it = alice_start; it != alice_end; ++it) {
        std::cout << "- " << it->second << '\\n';
    }
    std::cout << '\\n';

    // 5. Removing EXACTLY ONE key-value pair
    std::cout << "Bob finished updating documentation... removing task.\\n";
    auto bob_start = tasks.find("Bob");
    if (bob_start != tasks.end()) {
        // Erases just the FIRST task found for Bob. 
        // Note: To erase a specific task, you'd need to loop through equal_range to match the value string.
        tasks.erase(bob_start); 
    }

    // 6. Removing ALL values for a key
    std::cout << "Alice is going on vacation. Reassigning all her tasks (erasing).\\n";
    tasks.erase("Alice"); // Deletes EVERY pair where the key is "Alice"

    // 7. Processing remaining items
    std::cout << "\\n--- Remaining Master Task List ---\\n";
    for (const auto& [person, task] : tasks) {
        std::cout << person << " -> " << task << '\\n';
    }

    return 0;
}
```

**Output:**
```
Total active tasks: 5
Alice's task count: 2

--- Alice's To-Do List ---
- Fix server bug
- Review pull request

Bob finished updating documentation... removing task.
Alice is going on vacation. Reassigning all her tasks (erasing).

--- Remaining Master Task List ---
Charlie -> Deploy to production
Bob -> Write unit tests
```

---

