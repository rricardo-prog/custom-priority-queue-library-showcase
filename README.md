# High-Performance Priority Queue Architectures (C++)

[![Language: C++17](https://img.shields.io/badge/Language-C%2B%2B17-blue.svg)](https://isocpp.org/)
[![Focus: Advanced Data Structures](https://img.shields.io/badge/Focus-Heaps%20%26%20Memory%20Layouts-orange.svg)]()
[![Type: Templated Generic Library](https://img.shields.io/badge/Design-Generic%20%26%20Functor--Driven-brightgreen.svg)]()

A generic, high-performance C++ templated library implementing multiple priority queue paradigms from scratch under a unified Abstract Data Type (ADT) interface. The suite features array-backed heaps, binary heaps with custom percolation mechanics, and multiway tree-based pairing heaps with constant-time merges and fast dynamic priority updates[cite: 16, 18, 19].

---

> ### 🔒 Academic Integrity Notice
> This project was developed as part of **EECS 281 (Data Structures and Algorithms) at the University of Michigan**[cite: 16]. In strict adherence to the University of Michigan College of Engineering Honor Code and course copyright policies, the core source headers remain private[cite: 22].
>
> The documentation below outlines the comparative system architectures, algorithmic complexities, and design trade-offs. Live demonstrations and architectural walkthroughs are available upon request for technical interview discussions.

---

## ⚡ Engineering Highlights

* **Generic Polymorphic Interface:** Extends a base Priority Queue ADT supporting arbitrary types `T` and user-defined comparator functors (defaulting to `std::less<T>` for max-priority semantics)[cite: 16].
* **Zero STL Heap Reliance:** Engineered binary heap invariants completely from scratch (0-indexed parent/child pointer arithmetic, bottom-up heapify, up-percolate, down-percolate) without relying on `<algorithm>` heap primitives (`std::make_heap`, `std::push_heap`, `std::pop_heap`).
* **Self-Adjusting Pairing Heap:** Implemented a pointer-based multiway tree utilizing the **two-pass pairing scheme** for optimal amortized deletion and constant-time merges[cite: 19].
* **Dynamic Priority Mutation:** Engineered arbitrary element updates (`updateElt` / `updatePriorities`) allowing nodes to adjust values dynamically and restore heap invariants without rebuild overhead[cite: 21].
* **Memory Leak Validation:** Tested under `valgrind` to ensure complete reclamation across multi-node recursive pointer destruction cycles[cite: 22].

---

## 🛠️ Architectural Implementations

### 1. Sorted Array Priority Queue (`SortedPQ`)
* **Layout:** Flat `std::vector<T>` maintained in continuously sorted order[cite: 18].
* **Characteristics:** Offers instantaneous $O(1)$ `top()` and `pop()` from the back of the contiguous buffer[cite: 17, 18].
* **Insertion:** Leverages `std::lower_bound` binary search to locate positions, followed by insertion[cite: 20]. Optimal for read-heavy workloads with minimal dynamic inserts.

### 2. Binary Heap Priority Queue (`BinaryPQ`)
* **Layout:** Implicit complete binary tree mapped over a contiguous 0-indexed `std::vector<T>`[cite: 18].
* **Navigation Logic:** Given index $i$:
  $$\text{Parent}(i) = \lfloor (i - 1) / 2 \rfloor, \quad \text{Left}(i) = 2i + 1, \quad \text{Right}(i) = 2i + 2$$
* **Invariant Maintenance:**
  * **Heapify:** Implemented Floyd’s bottom-up linear construction running in $O(N)$ time starting from the lowest internal nodes.
  * **Push / Pop:** Logarithmic swim/sink percolation routines using custom element swaps to satisfy comparator invariants[cite: 17, 18, 20].

### 3. Pairing Heap Priority Queue (`PairingPQ`)
* **Layout:** Node-allocated multiway tree representation utilizing `left-child, sibling-pointer` structural references[cite: 19].
* **Merge (Meld) Optimization:** Unites two disjoint heap trees in $O(1)$ time by attaching the sub-tree with the smaller root priority as the leftmost child of the other[cite: 19].
* **Two-Pass Deletion (`pop`):**
  1. *First Pass (Left-to-Right):* Sibling sub-trees are paired up into binary groups and merged.
  2. *Second Pass (Right-to-Left):* The merged pairs are accumulated right-to-left into a single unified root node to bound the amortized depth.
* **Node Handles & Priority Updates:** Exposes node pointers/handles enabling $O(1)$ insert and logarithmic priority updates (`updateElt`) by severing the modified sub-tree and re-melding it with the root[cite: 19, 21].

---

## ⏱️ Comparative Algorithmic Complexity

| Data Structure | `top()` | `pop()` | `push()` | Meld / Merge | Memory Architecture |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Unordered Array**[cite: 16] | $O(N)$[cite: 16] | $O(N)$[cite: 16, 18] | $O(1)$ | $O(N)$ | Contiguous Array[cite: 18] |
| **Sorted Array**[cite: 16] | $O(1)$[cite: 17, 18] | $O(1)$[cite: 17, 18] | $O(N)$ | $O(N)$ | Contiguous Array[cite: 18] |
| **Binary Heap**[cite: 16] | $O(1)$[cite: 17] | $O(\log N)$ | $O(\log N)$ | $O(N)$ | Contiguous Array[cite: 18] |
| **Pairing Heap**[cite: 16] | $O(1)$[cite: 17] | $O(\log N)$ amortized | $O(1)$ | $O(1)$ | Node Pointers (Child/Sibling)[cite: 19] |

---

## 💻 Tech Stack & Tooling

* **Language Standard:** C++17[cite: 13, 20]
* **Design Paradigms:** Generic Programming, Templates, Custom Functors, Node-based Graph/Tree Linking[cite: 16, 19]
* **Build & Validation:** `g++` via GNU `Make`[cite: 15, 21], static code inspection flags (`-Wall`, `-Wextra`, `-pedantic`)
* **Profiling & Testing:** `Valgrind` heap-profiling for recursive deallocation validation[cite: 22]; modular unit testing for random orderings, stability, and duplicate priority metrics[cite: 16, 21].

---

## 📈 Engineering Learnings & Trade-Offs

* **Cache Locality vs. Structural Flexibility:** Contiguous binary heaps often outperform pointer-rich heaps in basic push/pop workloads due to CPU cache hit rates, while pairing heaps excel when dynamic priority adjustments and frequent melds dominate.
* **Template Inheritance Scope:** Managed C++ dependent name lookups and two-phase compilation across templated class hierarchies using explicit `this->` resolution for base member methods and comparator functors[cite: 20].
* **Deep Copying & Memory Ownership:** Implemented deep-copy copy constructors and assignment operators for tree nodes to guarantee memory safety during container duplicates[cite: 15].
