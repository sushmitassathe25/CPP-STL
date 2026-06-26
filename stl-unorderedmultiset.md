# UNORDERED_MULTISET

`std::unordered_multiset` is an associative container from the C++ Standard Library that contains a collection of objects of type `Key`. Unlike `std::unordered_set`, **it allows for multiple elements with the exact same value (duplicates)**. Elements are not sorted alphabetically or numerically; instead, they are organized into buckets based on their hash values to allow for constant average time complexity ($O(1)$) operations.

**Header:** `<unordered_set>`

**Template:** 
```cpp
template<
    class Key,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator<Key>
> class unordered_multiset;
```

![unordered multiset](unordered%20multiset.jpeg)

## High-level characteristics

- **Duplicates allowed**: This is the primary difference from `std::unordered_set`. You can insert the same value as many times as you want. All identical values will be grouped together within the same bucket.
- **Hash Table Structure**: Elements are physically placed in memory based on a mathematical hash function, completely ignoring sequential or sorted ordering.
- **Constant Time Average Performance**: Lookups, insertions, and deletions generally take $O(1)$ time, assuming a well-distributed hash function and manageable collision rates.
- **Immutable keys**: The values inside the multiset act as their own keys and are locked as `const`. Modifying an element directly would corrupt its hash value and break the container's bucket mapping.

## How it works internally

Internally, `std::unordered_multiset` uses a **Hash Table** with Separate Chaining.

- **Bucket Array**: The container manages a dynamic underlying array of "buckets."
- **Hashing and Grouping**: When a value is inserted, `std::hash` generates a hash code, and modulo arithmetic assigns it to a bucket. Crucially, the C++ standard guarantees that equivalent elements (duplicates) are always placed in the same bucket and are strictly adjacent to one another when iterating.
- **Separate Chaining**: Hash collisions (different values sharing the same bucket) and duplicate values are managed by linking the elements together in a linked list attached to the bucket.
- **Rehashing**: To maintain $O(1)$ performance, the table automatically grows and re-distributes elements (rehashes) when the `load_factor` exceeds the `max_load_factor`.  

**Exception safety**:
- Provides strong exception guarantees for single-element insertions. If rehashing fails due to `std::bad_alloc`, the multiset remains unchanged.

## Complexity guarantees

| Operation | Average Complexity | Worst-Case Complexity (Severe Collisions) |
|-----------|-----------|---------------------------------------------------|
| Lookup (`find, count, contains`) | O(1) | O(N) |
| Insertion (`insert, emplace`) | O(1) | O(N) |
| Erasure by value| O(count(key)) | O(N) |
| Erasure by iterator | O(1) | O(N) |
| `size`, `empty` | O(1) | O(1) |
| `clear` | O(N) | O(N) |


## Member functions and operators

### Constructors

```cpp
unordered_multiset();                                    // (1) empty multiset
explicit unordered_multiset( size_type bucket_count );   // (2) pre-allocated with at least bucket_count buckets
template< class InputIt >
unordered_multiset( InputIt first, InputIt last );       // (3) range [first, last)
unordered_multiset( const unordered_multiset& other );        // (4) copy constructor
unordered_multiset( unordered_multiset&& other ) noexcept;    // (5) move constructor
unordered_multiset( std::initializer_list<value_type> init ); // (6) initializer list
```


**Examples:**
```cpp
std::unordered_multiset<int> ums1;                      // empty
std::unordered_multiset<int> ums2 = {1, 2, 2, 3, 1};    // {1, 1, 2, 2, 3} (duplicates are kept!)
std::unordered_multiset<std::string> ums3(1000);        // instantly reserves 1,000 buckets
```

### Destructor

```cpp
~unordered_multiset(); // Destroys all nodes and frees heap allocations
```


### Element access

Because it only contains keys and is not sorted, it does NOT provide `operator[], .at(), .front(), or .back()`. Elements must be accessed via Iterators or Lookup functions.

### Iterators

Iterators traverse the container in an unpredictable order, except that identical elements are guaranteed to be adjacent.

```cpp
iterator begin() noexcept;                          // iterator to the first element
iterator end() noexcept;                            // iterator to end (one-past-last element)
const_iterator cbegin() const noexcept;
const_iterator cend() const noexcept;
```


### Capacity and Hash Policy (Specialized Methods)

```cpp
bool empty() const noexcept;                        // checks if size == 0
size_type size() const noexcept;                    // total number of elements (including duplicates)

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
*(Note: Unlike `unordered_set`, `insert` on a multiset does not return a `std::pair<iterator, bool>` because insertion of duplicates can never fail/be rejected).*

#### erase() — Remove elements

```cpp
iterator erase( const_iterator pos );                 // erase a SINGLE element at iterator (O(1) average)
iterator erase( const_iterator first, const_iterator last ); // erase a range
size_type erase( const Key& key );                    // erase ALL occurrences of 'key', returns number of erased elements
```


#### extract() and merge() (C++17) 

```cpp
node_type extract( const key_type& x );               // unlinks a single node matching x from the multiset
void merge( unordered_multiset& source );             // moves nodes from another unordered_multiset into this one
```

#### Lookup

Because duplicates exist, lookup functions often focus on finding ranges or counts.

```cpp
size_type count( const Key& key ) const;              // returns the number of times 'key' appears
iterator find( const Key& key );                      // returns an iterator to ONE of the elements matching 'key'
bool contains( const Key& key ) const;                // (C++20) returns true if at least one 'key' exists

std::pair<iterator,iterator> equal_range( const Key& key ); // Returns a range containing ALL elements matching 'key'
```


## Iterator and reference invalidation rules
  

| Operation | Iterator Invalidation | Pointer/Reference Invalidation |
|-----------|---|-------------------------------|
| `insert / emplace` | Invalidated ONLY IF the insertion triggers a rehash. Otherwise, none. | None. Pointers and references remain valid. |
| `rehash / reserve` | All iterators are invalidated. | None. |
| `erase` | Only iterators to the erased elements are invalidated. | Only pointers/references to the erased elements are invalidated. |
| `clear` / Destruction | All iterators are invalidated. | All are invalidated. |    


## Typical pitfalls and best practices

1. **The `erase(value)` trap:** Calling `ums.erase(5)` will remove every single instance of the number 5 from the multiset. If you only want to remove one instance, you must find an iterator first and erase that specific iterator: `ums.erase(ums.find(5));`.

2. **Frequency Counting vs `unordered_map`**: While `unordered_multiset` can be used to count how many times an item appears (using `.count()`), it is incredibly memory-inefficient to store 10,000 identical strings just to know the count is 10,000. It is almost always better to use `std::unordered_map<std::string, int>` for frequency counting.

3. **Iterating over duplicates**: If you want to iterate over all instances of a specific value, do not use `find()`. Use `equal_range()`, which gives you a precise `begin` and `end` iterator bound for that specific group of identical elements.
  


## Common idioms and patterns

### Erasing exactly ONE duplicate


```cpp
std::unordered_multiset<std::string> words = {"apple", "apple", "apple"};

// We only want to remove ONE apple, not all of them.
auto it = words.find("apple");
if (it != words.end()) {
    words.erase(it); // Safe: Removes only the element pointed to by the iterator
}
// words now contains 2 apples.
```

### Processing all duplicates of a specific key


```cpp
std::unordered_multiset<int> numbers = {10, 20, 20, 20, 30};

// Get the boundaries for all '20's
auto [range_start, range_end] = numbers.equal_range(20);

for (auto it = range_start; it != range_end; ++it) {
    // Process each '20' individually
    std::cout << "Processing a 20...\n";
}
```


## Real-world use cases

- **"Bag" Data Structures**: Situations where you need an unordered collection of items where quantities matter, such as an inventory system in a game (e.g., holding 5 Health Potions, 3 Mana Potions) where each item might have unique hidden metadata or addresses but shares the same hash key.

- **Graph Edges (Multigraphs)**: Storing connections in a multigraph where two distinct nodes can be connected by multiple parallel edges.

- **Log Aggregation**: Collecting thousands of error codes generated by a system. You can store them in an `unordered_multiset` and quickly extract blocks of identical error codes for batch processing using `equal_range()`.


## Useful headers and related features

| Header | Functionality |
|--------|---|
| `<unordered_set>` | Provides `std::unordered_set` and `std::unordered_multimset` |
| `<set>` | Provides `std::multiset` (allows duplicates, but keeps data strictly sorted in $O(\log N)$ time) |


## Full example program

```cpp
#include <iostream>
#include <unordered_set>
#include <string>

int main() {
    // 1. Initialization (Duplicates are retained)
    std::unordered_multiset<std::string> shopping_bag = {
        "apple", "banana", "apple", "orange", "apple", "banana"
    };

    // 2. Checking quantities
    std::cout << "Total items in bag: " << shopping_bag.size() << '\n';
    std::cout << "Number of apples: " << shopping_bag.count("apple") << "\n\n";

    // 3. Removing ALL instances of a value
    std::cout << "Eating all bananas...\n";
    shopping_bag.erase("banana"); // Deletes every "banana" in the set
    
    // 4. Removing EXACTLY ONE instance of a value
    std::cout << "Eating exactly one apple...\n";
    auto it = shopping_bag.find("apple"); // Finds the first apple
    if (it != shopping_bag.end()) {
        shopping_bag.erase(it); // Erases only that specific node
    }

    // 5. Processing remaining items
    std::cout << "\nRemaining items in bag:\n";
    for (const auto& item : shopping_bag) {
        std::cout << "- " << item << '\n';
    }

    // 6. Using equal_range to interact with grouped duplicates
    std::cout << "\nInspecting grouped apples via equal_range:\n";
    auto [start_it, end_it] = shopping_bag.equal_range("apple");
    int instance = 1;
    for (auto i = start_it; i != end_it; ++i) {
        std::cout << "Found Apple Instance #" << instance++ << '\n';
    }

    return 0;
}
```

**Output:**
```
Total items in bag: 6
Number of apples: 3

Eating all bananas...
Eating exactly one apple...

Remaining items in bag:
- orange
- apple
- apple

Inspecting grouped apples via equal_range:
Found Apple Instance #1
Found Apple Instance #2
```

---
