# VECTOR

`std::vector<T>` is a dynamic array container from the C++ Standard Library that encapsulates variable-size arrays. It stores elements contiguously in memory and manages storage automatically with dynamic allocation. The container provides random access to elements, efficient insertion/deletion at the end, and amortized constant-time performance for `push_back`.

**Header:** `<vector>`

**Template:** `template< class T, class Allocator = std::allocator<T> > class vector;`


![vector](vector.jpeg)

## High-level characteristics

- **Contiguous storage**: elements are stored in a single dynamically allocated contiguous block.
- **Size vs Capacity**:
  - `size()`: number of elements currently in the vector.
  - `capacity()`: number of elements the vector can hold without reallocating.
- **Amortized constant time** for `push_back` and `pop_back`.
- **Iterator invalidation**: many modifying operations can invalidate iterators, references, and pointers to elements.
- **Exception-safe**: provides exception guarantees for most operations (strong guarantee for copy operations when no reallocation occurs).

## How it works internally

Internally, a `std::vector` typically manages three pointers (or pointer-like members):
- **begin pointer**: points to the start of allocated storage
- **end pointer**: points one-past-the-last element currently contained
- **capacity pointer**: points one-past-the-last allocated slot

When you `push_back()` and `size() == capacity()`, the vector allocates a larger block of memory (implementation-defined growth strategy, often doubling or 1.5x growth), and moves or copies the existing elements to the new location.

**Growth strategy** varies by standard library implementation:
- Typical implementations use either exponential growth (2x) or 1.5x growth to reduce the number of reallocations.
- Reserve capacity using `reserve(n)` to avoid reallocations when size growth is predictable.
- Reallocation cost is O(N) but amortized cost of `push_back` is O(1).

**Exception safety**: 
- Operations that allocate memory (`insert`, `push_back` when capacity exhausted) may throw `std::bad_alloc`.
- Vector provides the strong exception guarantee for copy operations when element copying doesn't throw.
- Move operations provide the strong guarantee when moves don't throw.

## Complexity guarantees

| Operation | Complexity |
|-----------|-----------|
| Access by index (`operator[]`, `at`) | O(1) |
| `push_back` | O(1) |
| `pop_back` | O(1) |
| `insert` / `erase` at arbitrary position | O(N) (linear due to element shifting) |
| `reserve` (if causes reallocation) | O(N) |
| `reserve` (if no reallocation) | O(1) |
| `clear` | O(N) (calls destructors) |
| `size`, `capacity`, `empty` | O(1) |

## Member functions and operators

### Constructors

```cpp
vector();                                           // (1) empty vector
vector(size_type count);                            // (2) count default-constructed elements
vector(size_type count, const T& value);            // (3) count copies of value
template<class InputIt>
vector(InputIt first, InputIt last);                // (4) range [first, last)
vector(const vector& other);                        // (5) copy constructor
vector(vector&& other) noexcept;                    // (6) move constructor
vector(std::initializer_list<T> init);              // (7) initializer list
```

**Examples:**
```cpp
std::vector<int> a;                           // empty
std::vector<int> b(5);                        // {0, 0, 0, 0, 0}
std::vector<int> c(5, 42);                    // {42, 42, 42, 42, 42}
std::vector<int> d(b.begin(), b.end());       // copy from range
std::vector<int> e(std::move(b));             // move from b
std::vector<int> f = {1, 2, 3};               // initializer list
```

### Destructor

```cpp
~vector();  // destroys all elements and deallocates storage
```

### Assignment operators

```cpp
vector& operator=(const vector& other);                  // copy assignment
vector& operator=(vector&& other) noexcept;             // move assignment
vector& operator=(std::initializer_list<T> ilist);      // initializer list assignment
```

**Examples:**
```cpp
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2;
v2 = v1;                           // copy
v2 = std::move(v1);                // move
v2 = {4, 5, 6};                    // initializer list
```

### assign() — Replace contents

```cpp
void assign(size_type count, const T& value);           // fill assign
template<class InputIt>
void assign(InputIt first, InputIt last);               // range assign
void assign(std::initializer_list<T> ilist);            // initializer list assign
```

**Examples:**
```cpp
std::vector<int> v;
v.assign(3, 7);                                 // {7, 7, 7}
std::vector<int> src = {1, 2, 3};
v.assign(src.begin(), src.end());               // {1, 2, 3}
v.assign({10, 20, 30});                         // {10, 20, 30}
```

### Element access

```cpp
T& at(size_type pos);                   // bounds checking, throws std::out_of_range
const T& at(size_type pos) const;

T& operator[](size_type pos);           // unchecked access (faster, undefined if out of bounds)
const T& operator[](size_type pos) const;

T& front();                             // reference to first element (undefined if empty)
const T& front() const;

T& back();                              // reference to last element (undefined if empty)
const T& back() const;

T* data() noexcept;                     // pointer to underlying array
const T* data() const noexcept;
```

**Examples:**
```cpp
std::vector<int> v = {1, 2, 3};
int x = v[1];                   // 2 (unchecked)
int y = v.at(2);                // 3 (checked)
int z = v.front();              // 1
int w = v.back();               // 3
int* ptr = v.data();            // pointer to underlying array, usable with C APIs
```

### Iterators

```cpp
iterator begin() noexcept;              // iterator to beginning
const_iterator begin() const noexcept;
const_iterator cbegin() const noexcept;

iterator end() noexcept;                // iterator to end (one-past-last)
const_iterator end() const noexcept;
const_iterator cend() const noexcept;

reverse_iterator rbegin() noexcept;     // reverse iterator to end
const_reverse_iterator rbegin() const noexcept;
const_reverse_iterator crbegin() const noexcept;

reverse_iterator rend() noexcept;       // reverse iterator to beginning
const_reverse_iterator rend() const noexcept;
const_reverse_iterator crend() const noexcept;
```

**Examples:**
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// Forward iteration
for(auto it = v.begin(); it != v.end(); ++it) {
    std::cout << *it << ' ';  // 1 2 3 4 5
}

// Reverse iteration
for(auto it = v.rbegin(); it != v.rend(); ++it) {
    std::cout << *it << ' ';  // 5 4 3 2 1
}

// Range-based for (uses begin/end internally)
for(auto x : v) {
    std::cout << x << ' ';    // 1 2 3 4 5
}
```

### Capacity

```cpp
bool empty() const noexcept;              // check if size == 0
size_type size() const noexcept;          // number of elements
size_type max_size() const noexcept;      // maximum theoretical size
void reserve(size_type new_cap);          // ensure capacity >= new_cap
size_type capacity() const noexcept;      // current allocated capacity
void shrink_to_fit();                     // request to reduce capacity to fit size
```

**Examples:**
```cpp
std::vector<int> v;
v.reserve(100);                    // allocate space for at least 100 elements
std::cout << v.size();             // 0 (no elements added)
std::cout << v.capacity();         // >= 100 (allocated space)

if (!v.empty()) {
    // v has at least one element
}

if (v.size() == v.capacity()) {
    v.shrink_to_fit();             // request to free unused capacity
}
```

### Modifiers

#### clear() — Remove all elements

```cpp
void clear() noexcept;  // removes all elements but does not change capacity
```

#### push_back() / pop_back() — Add/remove at end

```cpp
void push_back(const T& value);     // copy element to end
void push_back(T&& value);          // move element to end
void pop_back();                    // remove last element (undefined if empty)
```

**Examples:**
```cpp
std::vector<std::string> names;
names.push_back("Alice");           // copy
names.push_back(std::string("Bob")); // move
names.pop_back();                   // remove "Bob"
```

#### emplace_back() — Construct in-place at end

```cpp
template<class... Args>
reference emplace_back(Args&&... args);  // construct element at end with perfect forwarding
```

**Benefits:** Avoids temporary construction and unnecessary copy/move.

**Examples:**
```cpp
struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {}
};

std::vector<Point> points;
points.emplace_back(10, 20);        // constructs Point in-place, no temp
points.push_back(Point(30, 40));    // constructs temp, then moves
```

#### insert() — Insert elements

```cpp
iterator insert(const_iterator pos, const T& value);              // single element copy
iterator insert(const_iterator pos, T&& value);                   // single element move
iterator insert(const_iterator pos, size_type count, const T& value); // fill
template<class InputIt>
iterator insert(const_iterator pos, InputIt first, InputIt last); // range
iterator insert(const_iterator pos, std::initializer_list<T> ilist); // initializer list
```

**Examples:**
```cpp
std::vector<int> v = {1, 2, 3};
v.insert(v.begin() + 1, 10);           // {1, 10, 2, 3}
v.insert(v.end(), 3, 99);              // {1, 10, 2, 3, 99, 99, 99}

std::vector<int> src = {4, 5};
v.insert(v.begin(), src.begin(), src.end()); // {4, 5, 1, 10, 2, 3, 99, 99, 99}
v.insert(v.end(), {100, 200});         // insert initializer list
```

#### emplace() — Construct and insert in-place

```cpp
template<class... Args>
iterator emplace(const_iterator pos, Args&&... args);  // construct element at pos with perfect forwarding
```

**Examples:**
```cpp
std::vector<Point> points = {Point(1, 2)};
points.emplace(points.begin() + 1, 10, 20);  // constructs Point(10, 20) at index 1
```

#### erase() — Remove elements

```cpp
iterator erase(const_iterator pos);                    // erase single element at pos
iterator erase(const_iterator first, const_iterator last); // erase range [first, last)
```

**Examples:**
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
v.erase(v.begin() + 2);                // {1, 2, 4, 5}
v.erase(v.begin(), v.begin() + 2);     // {4, 5}
```

#### resize() — Change the number of elements

```cpp
void resize(size_type count);                   // resize to count, default-initialize new elements
void resize(size_type count, const T& value);   // resize to count, fill new with value
```

**Examples:**
```cpp
std::vector<int> v = {1, 2, 3};
v.resize(5);                    // {1, 2, 3, 0, 0}
v.resize(2);                    // {1, 2}
v.resize(4, 99);                // {1, 2, 99, 99}
```

#### swap() — Exchange contents

```cpp
void swap(vector& other) noexcept;  // swap contents and capacities with another vector
```

**Examples:**
```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = {4, 5};
a.swap(b);                      // a = {4, 5}, b = {1, 2, 3}
std::swap(a, b);                // non-member swap function (equivalent)
```

### Comparison operators

```cpp
bool operator==(const vector& lhs, const vector& rhs);   // element-wise equality
bool operator!=(const vector& lhs, const vector& rhs);   // element-wise inequality
bool operator<(const vector& lhs, const vector& rhs);    // lexicographical comparison
bool operator<=(const vector& lhs, const vector& rhs);
bool operator>(const vector& lhs, const vector& rhs);
bool operator>=(const vector& lhs, const vector& rhs);
```

**Examples:**
```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = {1, 2, 3};
std::vector<int> c = {1, 2, 4};

if (a == b) { std::cout << "equal\n"; }       // true
if (a < c) { std::cout << "less\n"; }         // true
if (a != c) { std::cout << "not equal\n"; }   // true
```

### Non-member functions

```cpp
template<class T, class Alloc>
void swap(vector<T, Alloc>& lhs, vector<T, Alloc>& rhs) noexcept;  // swap two vectors
```

## Iterator and reference invalidation rules

| Operation | Invalidation |
|-----------|---|
| `push_back` | If reallocation occurs, **all** iterators/references/pointers invalidated. Otherwise, only `end()` invalidated. |
| `emplace_back` | Same as `push_back`. |
| `pop_back` | Only the `end()` iterator invalidated. References/pointers to erased elements are invalidated. |
| `insert` | If reallocation occurs, **all** iterators/references/pointers invalidated. Otherwise, iterators/references at or after insertion point are invalidated. |
| `emplace` | Same as `insert`. |
| `erase` | Iterators/references/pointers to erased elements are invalidated. Iterators/references/pointers to elements after erase point are invalidated. |
| `clear` | All iterators/references/pointers are invalidated. Capacity remains unchanged. |
| `reserve` | If reallocation occurs, **all** iterators/references/pointers invalidated. Otherwise, none. |
| `resize` | If size increases and reallocation occurs, **all** iterators/references/pointers invalidated. Otherwise, none. |
| `shrink_to_fit` | If reallocation occurs, **all** iterators/references/pointers invalidated. Otherwise, none. |

**Key takeaway:** After any modifying operation, assume iterators/references are invalid unless you're certain no reallocation occurred.

## Typical pitfalls and best practices

1. **Use `reserve()` for known sizes**: If you know how many elements you'll add, call `reserve(n)` first to avoid repeated reallocations.

2. **Prefer `emplace_back()` over `push_back()`** when constructing objects to avoid unnecessary copies/moves (especially for complex objects).

3. **Accessing empty vectors is undefined behavior**: Always check `empty()` before calling `front()`, `back()`, or `pop_back()`.

4. **Invalidated iterators cause undefined behavior**: Do not use iterators after operations that may invalidate them.

5. **Use iterators or ranges instead of indices for insertions/deletions** to maintain clarity about what's being modified.

6. **For stable references/pointers**, use `std::list` or `std::deque`:
   - `std::list` provides stable pointers/references across all modifying operations.
   - `std::deque` is faster than list but still invalidates some references on insertion/removal.

7. **Avoid `shrink_to_fit()` in performance-critical code**: It's a non-binding request that may or may not reduce capacity and can be expensive.

## Common idioms and patterns

### Erase-remove idiom

Remove all elements matching a condition without using `erase()` in a loop (which is inefficient due to repeated shifting).

```cpp
#include <algorithm>

std::vector<int> v = {1, 2, 3, 2, 4, 2, 5};
// Remove all 2s
v.erase(std::remove(v.begin(), v.end(), 2), v.end());
// v = {1, 3, 4, 5}
```

### Building with reserved capacity

```cpp
std::vector<int> make_range(int n) {
    std::vector<int> v;
    v.reserve(n);  // avoid reallocations
    for (int i = 0; i < n; ++i) {
        v.push_back(i);
    }
    return v;  // move elision / RVO
}
```

### Using vector with C APIs (contiguous buffer)

```cpp
#include <vector>
#include <cstdio>

void write_raw_bytes(const char* buf, std::size_t len);

int main() {
    std::vector<char> buf;
    buf.reserve(1024);
    // fill buf...
    write_raw_bytes(buf.data(), buf.size());  // pass pointer to C function
}
```

### Efficient construction with emplace_back

```cpp
struct Person {
    std::string name;
    int age;
    Person(const std::string& name, int age) : name(name), age(age) {}
};

std::vector<Person> people;
people.emplace_back("Alice", 30);    // constructs Person in-place
people.push_back(Person("Bob", 25)); // constructs temp, then moves
```

### Inserting/appending ranges

```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = {4, 5, 6};
a.insert(a.end(), b.begin(), b.end());  // a = {1, 2, 3, 4, 5, 6}

// C++11 and later
a.insert(a.end(), {7, 8, 9});           // a = {1, 2, 3, 4, 5, 6, 7, 8, 9}
```

### Creating a 2D vector (matrix)

```cpp
std::vector<std::vector<int>> matrix(rows, std::vector<int>(cols, 0));
// 'rows' x 'cols' matrix initialized with 0
```

## Real-world use cases

- **Dynamic arrays**: situations where the number of elements changes at runtime.
- **Random access requirements**: algorithms that need quick indexing (sorting, binary search, random selection).
- **Buffers**: IO buffers, serialization buffers where contiguous memory is required.
- **Graph adjacency lists**: `std::vector<std::vector<int>>` for graph representation.
- **Game engine loops**: list of entities/objects updated each frame with efficient iteration and random access.
- **Interop with C libraries**: passing contiguous data to C APIs via `data()` pointer.
- **Cache-friendly storage**: contiguous layout improves CPU cache utilization.

## Performance tips

1. **Reserve early if size is known**: `v.reserve(N)` dramatically reduces reallocations.

2. **Use move semantics and `emplace`**: avoid unnecessary copies.

3. **For frequent insertions/deletions in the middle**:
   - Consider `std::list` (but loses random access).
   - Consider `std::deque` (provides good insertion at front/back).
   - Consider custom data structures (rope, gap buffer, B-tree) for specialized needs.

4. **Batch operations when possible**: e.g., insert a range at once instead of inserting elements one by one.

5. **Monitor reallocation**: use `capacity()` to understand allocation behavior and optimize accordingly.

## Useful headers and related features

| Header | Functionality |
|--------|---|
| `<vector>` | vector container |
| `<algorithm>` | `sort()`, `find()`, `remove()`, `transform()`, etc. |
| `<numeric>` | `accumulate()`, `inner_product()`, etc. (for numeric operations on vectors) |

## Full example program

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>

int main() {
    // Create and populate a vector
    std::vector<int> v;
    v.reserve(10);

    for (int i = 0; i < 5; ++i) {
        v.push_back(i * 10);
    }

    std::cout << "Initial vector: ";
    for (auto x : v) std::cout << x << ' ';
    std::cout << "\nSize: " << v.size() << ", Capacity: " << v.capacity() << '\n';

    // Insert elements
    v.insert(v.begin() + 2, 99);
    std::cout << "After insert: ";
    for (auto x : v) std::cout << x << ' ';
    std::cout << '\n';

    // Erase element
    v.erase(v.begin() + 1);
    std::cout << "After erase: ";
    for (auto x : v) std::cout << x << ' ';
    std::cout << '\n';

    // Resize
    v.resize(8, 42);
    std::cout << "After resize: ";
    for (auto x : v) std::cout << x << ' ';
    std::cout << '\n';

    // Find and remove (erase-remove idiom)
    v.push_back(42);
    v.push_back(42);
    v.erase(std::remove(v.begin(), v.end(), 42), v.end());
    std::cout << "After removing 42s: ";
    for (auto x : v) std::cout << x << ' ';
    std::cout << '\n';

    // Sum with std::accumulate
    int sum = std::accumulate(v.begin(), v.end(), 0);
    std::cout << "Sum: " << sum << '\n';

    // Reverse iteration
    std::cout << "Reverse: ";
    for (auto it = v.rbegin(); it != v.rend(); ++it) {
        std::cout << *it << ' ';
    }
    std::cout << '\n';

    return 0;
}
```

**Output:**
```
Initial vector: 0 10 20 30 40
Size: 5, Capacity: 10
After insert: 0 10 99 20 30 40
After erase: 0 99 20 30 40
After resize: 0 99 20 30 40 42 42 42
After removing 42s: 0 99 20 30 40
Sum: 189
Reverse: 40 30 20 99 0
```

---
