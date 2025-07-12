### Summary to learn all data structures patterns

8 important patterns for coding interviews split into two categories:

- Linear structures: arrays, linked lists, strings.
- Nonlinear structures: trees, graphs.

Focus on pre-built code templates for these patterns.

*Linear Data Structure Patterns:*

1. Two Pointers:
    1. Reduces time complexity to linear time \(O(n)\).
    2. Two methods:
        1. Same direction: scans data in a single pass (e.g., fast and slow pointers to detect cycles or find middle elements).
        2. Opposite directions: finds pairs (e.g., sum of two numbers in a sorted array).
2. Sliding Window:
    1. Uses two pointers to manage a dynamic window of elements.
    2. Expands or contracts window based on conditions (e.g., longest substring without repeating characters).
    3. Often used with hashmaps.
3. Binary Search:
    1. Finds target in logarithmic time \(O(\log n)\).
    2. Extends to lists with monotonic conditions, not just sorted numbers.
    3. Example: finding minimum in a rotated sorted array.

*Nonlinear Data Structure Patterns*

1. Breadth-First Search (BFS):
    1. Explores nodes level by level.
    2. Uses queue to track visited nodes (ideal for level order traversal).
2. Depth-First Search (DFS):
    1. Explores one complete path before backtracking.
    2. Uses recursion efficiently to explore all paths.
    3. Example: counting islands in a grid.
3. Backtracking:
    1. Extends DFS to explore all possible solutions.
    2. Builds solutions dynamically by making decisions and backtracking on invalid paths
    3. Example: letter combinations of a phone number.

*Heaps (Priority Queue)*

1. Heaps
    1. Used for questions related to top K, K smallest/largest
    2. Min heap: Smallest value at the root
    3. Max heap: Largest value at the root
    4. Max heap is used to kind K smallest values, and viceversa for K largest

*Dynamic programming (DP)*

Optimizes solutions by breaking problems into overlapping subproblems.

Two approaches:

1. Top-down: Recursive with memoization to store results
2. Bottom-up: Solves smaller subproblems iteratively using a table