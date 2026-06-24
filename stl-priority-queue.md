# PRIORITY QUEUE

`std::priority_queue` is a container adaptor from the C++ Standard Library that provides constant-time lookup of the largest (by default) element. It wraps an underlying sequence container and structures the data using a heap layout (typically a max-heap). This configuration guarantees that the element with the highest priority remains at the front of the queue, ensuring an ordered extraction process regardless of the sequence in which items are inserted.

**Header:** `<queue>`

**Template:** `template<class T, class Container = std::vector<T>, class Compare = std::less<typename Container::value_type>> class priority_queue;`

*(By default, if no alternative container is specified, `std::vector<T>` is used. The default comparison predicate is std::less<T>, which structures the adapter as a max-heap where the largest element sits at the top).*


![priority queue](https://github.com/sushmitassathe25/CPP-STL/blob/main/priority%20queue.jpeg)

## High-level characteristics

- **Priority-based scheduling**: Elements are fetched based on sorting precedence rather than arrival order.
- **Container abstraction**: It modifies an underlying sequential container (like `std::vector` or `std::deque`) by running heap algorithms (`std::make_heap`, `std::push_heap`, `std::pop_heap`) over its elements.
- **Strict single-element visibility**: Only the highest priority element is visible via the `.top()` member function. Arbitrary mid-queue access or indexing is not permitted.
- **No iterator support**: `std::priority_queue` does not expose `begin()` or `end()` iterators, preventing manual loops or traversal without clearing the collection.
- **Custom order control**: Passing a custom comparator (like `std::greater<T>`) transforms the structure into a min-heap, keeping the smallest element at the top.

## How it works internally

`std::priority_queue` performs heap operations on its underlying sequential storage container to maintain its strict properties:

- `push(val)` Appends `val` to the end of the container using container `.push_back(val)` and then triggers `std::push_heap` to bubble the element up to its correct location in $O(\log n)$ time.
- `pop()` Swaps the top element with the last element via `std::pop_heap` in $O(\log n)$ time, moving the target element to the end of the container where it is safely expelled using container`.pop_back()`.  
- `top()`: Directly returns a reference to the first element (`container.front()`) in $O(1)$ constant time.


The underlying container must provide random access iterators along with `front()`, `push_back()`, and `pop_back()`. While `std::vector` and `std::deque` fit this specification perfectly, `std::list` cannot be used because it lacks random access iterators ($O(1)$ indexing) required by heap layout math.

**Exception safety**: 
- Reflects the exception safety profiles of the underlying container and comparison predicates. If the container's `push_back` or structural move algorithms fail, standard rollover rules apply (e.g., strong exception guarantee under `std::bad_alloc`).


## Complexity guarantees


| Operation | Complexity |
|-----------|-----------|
| `top` | O(1) |
| Heap generation from raw container | O(N) (linear time optimization when passing a full container to the constructor) |
| `push` / `emplace` | O(log N) |
| `pop` | O(log N) |
| `size`, `empty` | O(1) |

## Member functions and operators

### Constructors

```cpp
priority_queue();                                   // (1) default empty priority queue
explicit priority_queue( const Compare& compare );  // (2) empty queue with specific comparator
priority_queue( const Compare& compare, const Container& cont ); // (3) builds heap from pre-existing cont
template< class InputIt >
priority_queue( InputIt first, InputIt last, const Compare& compare = Compare(), const Container& cont = Container() ); // (4) range construct [first, last)

// Since C++23: range construction support without container parameters
template< class ContainerAllocator >
explicit priority_queue( const ContainerAllocator& alloc );
```

**Examples:**
```cpp
std::vector<int> vec = {10, 5, 20, 15};

std::priority_queue<int> pq1;                       // Default max-heap configuration
std::priority_queue<int, std::vector<int>, std::greater<int>> min_pq; // Configuration for a min-heap

std::priority_queue<int> pq2(vec.begin(), vec.end()); // Generates a max-heap from range in O(N) time
```

### Destructor

```cpp
~priority_queue(); // Cleans up the queue along with elements managed inside the backing container
```

### Assignment operators

```cpp
priority_queue& operator=( const priority_queue& other );            // copy assignment
priority_queue& operator=( priority_queue&& other ) noexcept;         // move assignment
```


### Element access

```cpp
const_reference top() const;                        // returns a const reference to the top element (undefined if empty)
```
*Note: Unlike other adapters, `.top()` returns a const reference. You cannot modify the top element directly in-place because altering its value would break the internal heap structure.*

**Examples:**
```cpp
std::priority_queue<int> pq;
pq.push(50);
pq.push(100);

int highest = pq.top();                             // highest = 100
// pq.top() = 200;                                  // Compile Error! Modifying top element directly is banned
```

### Capacity

```cpp
bool empty() const;                                 // checks whether the container adaptor is empty
size_type size() const;                             // returns the number of elements
```

### Modifiers

#### push() / pop() — Alter heap structure

```cpp
void push( const T& value );                        // inserts element and sorts via heap rules
void push( T&& value );                             // moves element and sorts via heap rules
void pop();                                         // extracts the top element (undefined if empty)
```

**Examples:**
```cpp
std::priority_queue<int> pq;
pq.push(30);                                        // [30]
pq.push(70);                                        // [70, 30]
pq.push(10);                                        // [70, 30, 10]
pq.pop();                                           // Evicts 70, leaving [30, 10]
```

#### emplace() — Construct in-place at back

```cpp
template< class... Args >
void emplace( Args&&... args );                     // constructs element inline, then bubbles up via heap math
```

  
#### swap() — Exchange contents

```cpp
void swap( priority_queue& other ) noexcept(/* conditional */); // Swaps underlying containers instantly
```



### Non-member functions

```cpp
template< class T, class Container, class Compare >
void swap( std::priority_queue<T,Container,Compare>& lhs,
           std::priority_queue<T,Container,Compare>& rhs ) noexcept;
```

## Iterator and reference invalidation rules

Because `std::priority_queue` relies on `std::vector` (by default) or `std::deque`, it shares their reference and iterator invalidation properties:

- `std::vector` **backing (Default)**: A `push` or `emplace` operation that triggers a capacity reallocation invalidates all external pointers and references to data elements. If no reallocation occurs, or if you use `std::deque`, existing references to untouched elements remain stable in memory.
- **Element reassignment note**: Even if references stay technically stable in memory, their positional indexing order changes significantly during a `push` or `pop` as heap elements bubble up or down.


## Typical pitfalls and best practices

1. **Popping or reading an empty priority queue triggers Undefined Behavior**: Running `.top()` or `.pop()` on an empty priority queue causes a segmentation fault or memory corruption. Always check `pq.empty()` before these operations.

2. **Read-only top elements**: The top element is read-only (`const`). If you need to alter an element, you must `pop()` it out, update it, and `push()` it back in to let the heap re-sort itself properly.

3. **Misconfiguring min-heaps**: When declaring a min-heap, ensure you explicitly provide all three template arguments: `std::priority_queue<Type, std::vector<Type>, std::greater<Type>> min_heap;`. Skipping the container argument breaks the compilation.


## Common idioms and patterns

### Processing records by priority

```cpp
std::priority_queue<int> scheduler;
// populate scheduler...

while(!scheduler.empty()) {
    int urgent_task = scheduler.top(); // Access highest item safely
    
    // Execute task logic...
    std::cout << "Handling task importance rank: " << urgent_task << "\n";
    
    scheduler.pop();                   // Remove the processed element
}
```
### Custom object sorting via lambda predicates

```cpp
struct Task { int id; int priority; };

// Define a custom comparator lambda for a min-heap setup
auto comp = [](const Task& a, const Task& b) { return a.priority > b.priority; };

// Pass the lambda type and object to the constructor
std::priority_queue<Task, std::vector<Task>, decltype(comp)> task_queue(comp);
```

## Real-world use cases

- **Operating System process schedulers**: Managing active threads or CPU tasks, ensuring high-priority system cycles execute before lower-priority background jobs.
- **Graph searching algorithms**: Powering the active frontier queue in Dijkstra's shortest path algorithm and A* search algorithms to repeatedly fetch the closest unvisited node.
- **Huffman encoding compression**: Building optimal character prefix trees by repeatedly extracting the two lowest-frequency alphanumeric components.
- **Data stream filtering (Top-K elements)**: Tracking the top $K$ largest or smallest values in an ongoing data stream by maintaining a min-heap of fixed size $K$.


## Useful headers and related features

| Header | Functionality |
|--------|---|
| `<queue>` | Priority queue and standard FIFO queue container adaptors|
| `<algorithm>` | Core heap subroutines (`make_heap`, `push_heap`, `pop_heap`) used by the adaptor |
| `<functional>` | Standard comparison functions like `std::less` and `std::greater` |

## Full example program

```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <string>

// Struct representing a patient in an emergency room triage system
struct Patient {
    std::string name;
    int severity; // Scale from 1 (low) to 10 (critical)

    // Overload the < operator to configure a max-heap based on severity
    bool operator<(const Patient& other) const {
        return severity < other.severity;
    }
};

int main() {
    // Instantiate a priority queue managing patients via max-heap configuration
    std::priority_queue<Patient> triage_line;

    // Simulate patients arriving out of priority order
    triage_line.push({"John (Mild Fever)", 2});
    triage_line.push({"Mary (Chest Pain)", 10});
    triage_line.push({"Alice (Deep Cut)", 6});
    triage_line.push({"David (Sprained Wrist)", 3});

    std::cout << "Triage Queue Initial Status: " << (triage_line.empty() ? "Empty" : "Active") << '\n';
    std::cout << "Number of patients waiting: " << triage_line.size() << '\n';
    std::cout << "Most urgent patient (Top): " << triage_line.top().name 
              << " [Severity: " << triage_line.top().severity << "]\n\n";

    // Process patients based on medical urgency
    std::cout << "--- ER Treatment Pipeline ---\n";
    while (!triage_line.empty()) {
        Patient current = triage_line.top();
        std::cout << "Treating: " << current.name << " (Severity: " << current.severity << ")\n";
        triage_line.pop(); // Evicts the patient from the top of the heap
    }

    std::cout << "\nAll patients treated. Remaining line count: " << triage_line.size() << '\n';

    return 0;
}
```

**Output:**
```
Triage Queue Initial Status: Active
Number of patients waiting: 4
Most urgent patient (Top): Mary (Chest Pain) [Severity: 10]

--- ER Treatment Pipeline ---
Treating: Mary (Chest Pain) (Severity: 10)
Treating: Alice (Deep Cut) (Severity: 6)
Treating: David (Sprained Wrist) (Severity: 3)
Treating: John (Mild Fever) (Severity: 2)

All patients treated. Remaining line count: 0
```

---



