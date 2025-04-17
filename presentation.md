# MyTinySTL Project Presentation

---

## Analysis By _The Debuggers_

### Team Members
#### - Aarohi Mehta *(202401002)*
#### - Alvita Thakor *(202401012)*
#### - Darshan Talati *(202401046)*
#### - Dharmesh Upadhyay *(202401049)*

---

## Basics of MyTinySTL

### What is MyTinySTL?
"MyTinySTL" is an open-source, lightweight implementation of the C++ Standard Template Library (STL), developed by Alinshans and hosted at `https://github.com/Alinshans/MyTinySTL`. It replicates core STL components like containers (`vector`, `map`, `set`) and algorithms (`find`, `sort`), written in C++11.

### Purpose and Goals
- **Educational**: Designed as a learning tool to understand STL internals, making it ideal for students and developers.
- **Lightweight**: Aims for a smaller footprint than the standard STL, sacrificing some features for simplicity.
- **Community-Driven**: With ~12k stars and 3k+ forks, it’s widely recognized, though now in maintenance mode (bug fixes only since v2.0.0).

### Technical Foundations
- **C++11**: Leverages modern features like move semantics (`std::move`), perfect forwarding (`std::forward`), and `noexcept` for efficiency and safety.
- **Namespace**: Uses `mystl` to avoid conflicts with `std`, allowing side-by-side usage with the standard library.
- **Header-Only**: All functionality is in headers (e.g., `vector.h`), no separate `.cpp` files, simplifying inclusion.

### Current State
- **Version**: v2.0.1 (November 17, 2023), with minor fixes and updated docs.
- **Scope**: Covers essential containers and algorithms but skips some STL features (e.g., `vector<bool>` specialization, custom allocator support post-v2.0.0).

---

## Breadth-Wise Understanding

### Project Structure
- **Headers**: Core functionality in root (e.g., `vector.h`, `map.h`, `algo.h`).
- **Test/**: Validation suite (e.g., `vector_test.h`, `map_test.h`) to ensure correctness.
- **MSVC/**: Visual Studio solution (`MyTinySTL_VS2015.sln`) for Windows builds.
- **CMakeLists.txt**: CMake config for Linux/macOS builds.

### Components
- **Containers**: `vector`, `list`, `deque`, `stack`, `queue`, `map`, `set`, `unordered_map`, `unordered_set`, `basic_string`.
- **Algorithms**: General (`algo.h`), basic (`algobase.h`), heap (`heap_algo.h`), numeric (`numeric.h`).
- **Utilities**: `allocator.h`, `memory.h`, `iterator.h`, `functional.h`, `util.h`, `type_traits.h`.

### Build System
- **Linux/macOS**: 
  - Uses CMake 2.8+ to generate Makefiles.
  - Commands: `cmake .. && make`, outputs `stltest` in `bin/`.
  - Supported compilers: GCC 5.4+, Clang 3.5+.
- **Windows**: 
  - Visual Studio 2015/2017 solution, MSVC 14.0+.
  - Run in Release mode (`Ctrl + F5`).

### Community and Maintenance
- **Popularity**: High engagement (11.9k stars), reflecting its value.
- **Status**: Post-v2.0.0, focus shifted to bug fixes, encouraging Issues/PRs for contributions.

---

## Depth-Wise Analysis

---

### Approaches Taken
- **Modularity**: Each header (e.g., `vector.h`) is self-contained, allowing selective inclusion without dependencies on unused components.
- **C++11 Features**: 
  - Move semantics reduce copying (e.g., `push_back` with rvalues).
  - `noexcept` specifies exception-free operations where possible.
  - Type traits (`type_traits.h`) enable generic programming with compile-time checks.
- **Exception Safety**: 
  - Basic guarantee (valid state post-exception) for most ops.
  - Strong guarantee (rollback on failure) for key ops like `vector::push_back`.
- **Testing**: `Test/` directory runs comprehensive checks (e.g., `deque_test.h`), with performance tests for ops like `unordered_map` insertion.

---

### Data Structures and Trade-Offs

---

#### Dynamic Arrays
- **Headers**: `vector.h`, `deque.h`
- **Internal Structure**: 
  - **`vector.h`**: Contiguous dynamic array.
    - Grows by doubling capacity (e.g., from 8 to 16) when full, using `allocator.h`.
    - Amortized O(1) for `push_back` due to geometric resizing.
  - **`deque.h`**: Segmented array (array of pointers to fixed-size blocks).
    - Each block is contiguous, but blocks are scattered, enabling O(1) `push_front`/`push_back`.
- **Key Operations**:
  - `vector`: `push_back`, `emplace`, random access via `operator[]`.
  - `deque`: `push_front`, `push_back`, indexed access with slight overhead.
- **Trade-Offs**:
  - `vector`: 
    - Pros: Fast random access (O(1)), cache-friendly due to contiguity.
    - Cons: O(n) mid-insertions, resizing can invalidate iterators.
  - `deque`: 
    - Pros: Efficient at both ends, strong exception safety for insertions.
    - Cons: Higher memory overhead (pointers per block), less cache-efficient.
- **Details**:
  - `vector` resizing: If capacity exceeds, it allocates a new, larger array, moves elements (C++11 move semantics), and deallocates the old one.
  - `deque` block management: Maintains a map of block pointers, resizing the map if ends overflow.

---

#### Linked Lists
- **Header**: `list.h`
- **Internal Structure**: Doubly-linked list.
  - Each node has `prev`, `next`, and `value`, allocated via `allocator.h`.
  - O(1) insertion/deletion anywhere with a valid iterator.
- **Key Operations**: `push_front`, `push_back`, `splice` (move elements between lists), bidirectional iteration.
- **Trade-Offs**:
  - Pros: Constant-time insertions/removals, no resizing needed.
  - Cons: O(n) access by index, memory overhead per node (pointers + value).
- **Details**:
  - Iterators remain valid post-insertion unless the node is deleted.
  - Splicing is efficient (pointer reassignment), unlike `vector`’s copying.

---

#### Trees
- **Headers**: `map.h`, `set.h`, `rb_tree.h`
- **Internal Structure**: Red-black tree (via `rb_tree.h`).
  - Self-balancing binary search tree with color (red/black) properties.
  - O(log n) for insert, delete, lookup due to balanced height.
- **Key Operations**:
  - `map`: Key-value pair storage, ordered by key (uses `functional.h` comparators).
  - `set`: Unique keys only, ordered.
- **Trade-Offs**:
  - Pros: Guaranteed O(log n) performance, natural ordering for free.
  - Cons: Slower than hash tables for lookups, complex balancing logic.
- **Details**:
  - Balancing: After insertion, rotations and color flips maintain properties (e.g., no two reds in a row, equal black height).
  - Example: Inserting into `map` triggers a tree walk, then rebalance if needed.

---

#### Hash Tables
- **Headers**: `unordered_map.h`, `unordered_set.h`, `hashtable.h`
- **Internal Structure**: Hash table (via `hashtable.h`).
  - Array of buckets, each a linked list for collisions.
  - Average O(1) insert/lookup, worst-case O(n) with poor hashing.
- **Key Operations**:
  - `unordered_map`: Key-value pairs, unordered.
  - `unordered_set`: Unique keys, unordered.
- **Trade-Offs**:
  - Pros: Fast average-case access, simpler than trees for unordered data.
  - Cons: No ordering, collision handling adds overhead, quality depends on hash function.
- **Details**:
  - Hashing: `hashtable.h` likely defines a default hash, with buckets resized on load factor exceedance.
  - Performance tested in `unordered_map_test.h` (e.g., 100k insertions).

---

#### Strings
- **Headers**: `basic_string.h`, `astring.h`
- **Internal Structure**: Dynamic array of characters.
  - `basic_string` is a template (e.g., `mystl::basic_string<char>`).
  - `astring` might be a typedef or alias (unclear without code inspection).
- **Key Operations**: Concatenation, substring, capacity management.
- **Trade-Offs**:
  - Pros: Simple, familiar string interface.
  - Cons: No small-string optimization (SSO) like `std::string`, higher memory use for short strings.
- **Details**:
  - Resizing mirrors `vector`, doubling capacity as needed.
  - Likely lacks advanced optimizations (e.g., SSO stores short strings in the object itself).

---

### Utilities
- **Headers**: `allocator.h`, `memory.h`, `iterator.h`, `functional.h`
- **Roles**:
  - `allocator.h`: Manages raw memory allocation; memory pool removed in v2.0.0 for simplicity.
  - `memory.h`: `uninitialized_copy`, `destroy` for raw memory ops.
  - `iterator.h`: Traits and categories (e.g., random-access for `vector`).
  - `functional.h`: Comparators (e.g., `less`) for `map`/`set`.
- **Trade-Offs**: Simplicity (no custom allocators) vs. flexibility loss.

---