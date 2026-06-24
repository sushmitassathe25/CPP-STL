# FORWARD LIST

`std::forward_list` is a sequence container from the C++ Standard Library that encapsulates a singly-linked list. Unlike `std::list` (which is a doubly-linked list), `std::forward_list` only maintains a single forward link between successive elements. This makes it highly optimized for memory efficiency, offering zero space overhead compared to a raw, handwritten C-style singly-linked list. It provides constant-time insertion and extraction operations but only supports forward traversal.

**Header:** `<forward_list>`


**Template:** `template< class T, class Allocator = std::allocator<T> > class forward_list;`


![forward list](forward%20list.jpeg)


## High-level characteristics

- **Singly-linked structure**: Each element (node) contains a data payload and a single pointer directed exclusively to the next element. It does not track the preceding node.
- **Maximum memory efficiency**: Erases the secondary backward pointer required by `std::list`, saving `sizeof(void*)` bytes per node. It is the go-to sequence container when memory footprints must be minimized.
- **No backward traversal or size queries**: It cannot move backwards (`--it` is invalid). Additionally, to save memory and execution overhead, it does not maintain a size counter; tracking `.size()` is completely omitted. To determine its size, you must manually count elements in $O(N)$ time.
- **Constant time updates**: Inserting, moving, or removing elements takes $O(1)$ constant time after you have targeted a specific position.
- **Distinct positioning mechanics**: Because nodes cannot look backward, modifying operations are performed after a designated node (`insert_after`, `erase_after`, `emplace_after`) rather than at a specified position.


## How it works internally

Internally, `std::forward_list` handles a linear sequence of isolated node allocations:

- **Head Node Pointer**: The container objects themselves typically wrap only a single tracking pointer referencing the first data node in the sequence.
- **Forward Chaining**: Calling an operation like `push_front()` allocates a new node on the heap, maps its internal forward pointer to the pre-existing first node, and updates the container's head pointer.

Because it lacks direct random access and backward links, common algorithms must adapt. For instance, finding an item requires scanning from the head node pointer step-by-step ($O(N)$ linear time). However, splicing or removing an item simply requires modifying two local pointers, ensuring that no elements need to be shifted in memory.


**Exception safety**: 
- Insertion procedures (`push_front`, `insert_after`) maintain the strong exception guarantee. If a memory allocation throws `std::bad_alloc`, the structure rolls back safely to its original state.

## Complexity guarantees

| Operation | Complexity |
|-----------|-----------|
| `front` | O(1) |
| `push_front` | O(1) |
| `pop_front` | O(1) |
| `insert_after` / `erase_after` | O(1) (once the iterator position is reached) |
| Arbitrary search / traversal | O(N) (linear scanning required) |
| Distance / Size computation | O(N) (requires parsing using `std::distance`) |
| `clear` | O(N) (sequentially deallocates every node) |


## Member functions and operators

### Constructors

```cpp
forward_list();                                     // (1) empty forward_list
explicit forward_list( size_type count );           // (2) count default-constructed elements
forward_list( size_type count, const T& value );    // (3) count copies of value
template< class InputIt >
forward_list( InputIt first, InputIt last );         // (4) range [first, last)
forward_list( const forward_list& other );          // (5) copy constructor
forward_list( forward_list&& other ) noexcept;      // (6) move constructor
forward_list( std::initializer_list<T> init );      // (7) initializer list
```

**Examples:**
```cpp
std::forward_list<int> fl1;                          // empty
std::forward_list<int> fl2(4, 100);                  // {100, 100, 100, 100}
std::forward_list<int> fl3 = {1, 2, 3};              // initializer list
std::forward_list<int> fl4(fl3.begin(), fl3.end());  // range copy
```

### Destructor

```cpp
~forward_list(); // Sequentially invokes destructors for all nodes and frees heap allocations
```

### Assignment operators

```cpp
forward_list& operator=( const forward_list& other );     // copy assignment
forward_list& operator=( forward_list&& other ) noexcept;  // move assignment
forward_list& operator=( std::initializer_list<T> ilist ); // initializer list assignment
```


### Element access

```cpp
T& front();                                         // reference to the first element (undefined if empty)
const T& front() const;
```
*Note: `.back()` is not provided because finding the last element requires an expensive $O(N)$ linear scan.*


### Iterators

```cpp
iterator begin() noexcept;                          // iterator to the first element
const_iterator begin() const noexcept;
const_iterator cbegin() const noexcept;

iterator end() noexcept;                            // iterator to the end (one-past-last / nullptr)
const_iterator end() const noexcept;
const_iterator cend() const noexcept;

// Special member functions for forward_list:
iterator before_begin() noexcept;                   // iterator to the position before the first element
const_iterator before_begin() const noexcept;
const_iterator cbefore_begin() const noexcept;
```

**Examples:**
```cpp
std::forward_list<int> fl = {10, 20, 30};

// Standard forward loop iteration
for (auto it = fl.begin(); it != fl.end(); ++it) {
    std::cout << *it << ' ';                        // 10 20 30
}
```

### Capacity

```cpp
bool empty() const noexcept;                        // checks whether the container is empty
size_type max_size() const noexcept;                // returns the maximum theoretical number of elements
```
*Note: `size()` does not exist for `std::forward_list`.*


### Modifiers

#### clear() — Remove all elements

```cpp
void clear() noexcept; // Clears the list and deallocates memory for all nodes
```

#### push_front() / pop_front() — Boundary updates

```cpp
void push_front( const T& value );                  // copy element to the front
void push_front( T&& value );                       // move element to the front
void pop_front();                                   // remove the first element (undefined if empty)
```


#### insert_after() — Insert elements after an iterator

```cpp
iterator insert_after( const_iterator pos, const T& value );              // single copy
iterator insert_after( const_iterator pos, T&& value );                   // single move
iterator insert_after( const_iterator pos, size_type count, const T& value ); // fill
template< class InputIt >
iterator insert_after( const_iterator pos, InputIt first, InputIt last ); // range
iterator insert_after( const_iterator pos, std::initializer_list<T> ilist ); // initializer list
```


**Examples:**
```cpp
std::forward_list<int> fl = {1, 2, 4};
auto it = fl.begin();                               // points to 1

// Inserts 99 AFTER 1
fl.insert_after(it, 99);                            // fl = {1, 99, 2, 4}

// Inserts at the very beginning using before_begin()
fl.insert_after(fl.before_begin(), 0);              // fl = {0, 1, 99, 2, 4}
```

#### emplace_after() — Construct in-place after an iterator

```cpp
template< class... Args >
iterator emplace_after( const_iterator pos, Args&&... args ); // constructs element inline via perfect forwarding
```

#### erase_after() — Remove elements after an iterator

```cpp
iterator erase_after( const_iterator pos );                             // erases single node after pos
iterator erase_after( const_iterator first, const_iterator last );       // erases range (first, last)
```

**Examples:**
```cpp
std::forward_list<int> fl = {10, 20, 30, 40};
fl.erase_after(fl.begin());                         // Removes the element after 10 (removes 20) -> {10, 30, 40}
```

#### resize() — Change the number of elements

```cpp
void resize( size_type count );
void resize( size_type count, const T& value );
```

#### swap() — Exchange contents

```cpp
void swap( forward_list& other ) noexcept;          // swaps head node reference pointers instantly
```

### Operations (List Manipulation)

```cpp
void merge( forward_list& other );                  // merges two sorted lists
void splice_after( const_iterator pos, forward_list& other ); // moves nodes from other into this list
void remove( const T& value );                      // removes all elements matching value
template< class Predicate >
void remove_if( Predicate p );                      // removes all elements matching predicate
void reverse() noexcept;                            // reverses the order of elements in-place
void unique();                                      // removes consecutive duplicate elements
void sort();                                        // sorts elements using merge sort (O(N log N))
```


## Iterator and reference invalidation rules

`std::forward_list` features highly stable memory persistence layouts during structural updates:

| Operation | Invalidation |
|-----------|---|
| `push_front`/ `emplace_front` | None. Existing pointers, references, and iterators remain valid. |
| `pop_front` | Only pointers, references, and iterators referencing the removed element are invalidated. |
| `insert_after` / `emplace_after` | None |
| `erase_after` | Only pointers, references, and iterators referencing the erased nodes are invalidated. |
| `clear` / Destruction | All pointers, references, and iterators are invalidated. |


**Key takeaway:** Modifying a node inside a `std::forward_list` never causes other elements to shift or reallocate in memory. Pointers to un-erased nodes remain stable across all operations.

## Typical pitfalls and best practices

1. **Incorrect iterator placement**: Calling standard `insert` or `erase` paradigms will throw a compilation error. You must explicitly target the position before your modification zone and call `insert_after` or `erase_after`.

2. **High cost of calculating size**: Do not write algorithms that repeatedly query the list's size. Running `std::distance(fl.begin(), fl.end())` loops through the entire sequence in $O(N)$ linear time, which can quickly degrade performance.

3. **No random access**: Avoid using `std::forward_list` if your workflow relies heavily on index lookups (e.g., matching items via `[i]`).


## Common idioms and patterns

### Inserting at the very front using before_begin()

```cpp
std::forward_list<int> fl = {2, 3, 4};

// before_begin() returns a proxy iterator pointing to a virtual node before the head
auto initial_pos = fl.before_begin();
fl.insert_after(initial_pos, 1);                    // fl = {1, 2, 3, 4}
```

### Linear search and conditional removal

```cpp
std::forward_list<int> items = {11, 14, 21, 30, 45};

// Erase-remove idiom variant for forward_list using built-in methods
items.remove_if([](int x) { return x % 2 == 0; });  // Removes even numbers -> {11, 21, 45}
```


## Real-world use cases

- **Embedded systems memory pools**: Tracking blocks of available memory in resource-constrained hardware where every byte matters and backward pointer overhead cannot be tolerated.
- **Hash table chaining implementations**: Handling bucket collision chains inside performance-critical custom hash map architectures.
- **Adjacency lists in sparse graphs**: Modeling directed graph configurations where algorithms only step forward to traverse neighboring connections.
- **Undo history buffers (unidirectional pipelines)**: Pushing operational snapshots onto a timeline where navigation only moves sequentially in one direction.


## Useful headers and related features

| Header | Functionality |
|--------|---|
| `<forward_list>` | Singly-linked list container framework |
| `<list>` | Doubly-linked list container counterpart |
| `<iterator>` | Provides helper functions like `std::next` and `std::distance` |

## Full example program

```cpp
#include <iostream>
#include <forward_list>
#include <algorithm>
#include <iterator>

int main() {
    // Instantiate an inventory stream using a singly-linked forward_list
    std::forward_list<int> batch_ids = {1003, 1005, 1007};

    // Add elements to the front
    batch_ids.push_front(1001);                     // {1001, 1003, 1005, 1007}

    std::cout << "Initial Forward List Sequence: ";
    for (int id : batch_ids) std::cout << id << " ";
    std::cout << "\nFirst element check via .front(): " << batch_ids.front() << "\n\n";

    // Insert an item into the middle of the list
    auto current_it = batch_ids.begin();            // Points to 1001
    batch_ids.insert_after(current_it, 1002);       // Inserts after 1001 -> {1001, 1002, 1003, 1005, 1007}

    std::cout << "Sequence after insert_after: ";
    for (int id : batch_ids) std::cout << id << " ";
    std::cout << '\n';

    // Remove an item from the list
    // To remove 1005, we need an iterator pointing to the node before it (1003)
    auto search_it = batch_ids.begin();
    std::advance(search_it, 2);                      // Advances to point to 1003
    batch_ids.erase_after(search_it);               // Removes the node after 1003 (removes 1005)

    std::cout << "Sequence after erase_after (removed 1005): ";
    for (int id : batch_ids) std::cout << id << " ";
    std::cout << "\n\n";

    // Reverse the list in-place
    batch_ids.reverse();
    std::cout << "In-place reversed list layout: ";
    for (int id : batch_ids) std::cout << id << " ";
    std::cout << '\n';

    // Calculate total element count manually using std::distance
    auto element_count = std::distance(batch_ids.begin(), batch_ids.end());
    std::cout << "Calculated list size: " << element_count << '\n';

    return 0;
}
```

**Output:**
```
Initial Forward List Sequence: 1001 1003 1005 1007 
First element check via .front(): 1001

Sequence after insert_after: 1001 1002 1003 1005 1007 
Sequence after erase_after (removed 1005): 1001 1002 1003 1007 

In-place reversed list layout: 1007 1003 1002 1001 
Calculated list size: 4
```

---

