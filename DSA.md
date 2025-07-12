# Data Structures & Algorithms (DSA) 📊

A comprehensive guide to mastering the essential patterns and techniques for coding interviews and competitive programming.

## 📚 Table of Contents

- [Overview](#overview)
- [Learning Strategy](#learning-strategy)
- [Pattern Categories](#pattern-categories)
- [Linear Data Structure Patterns](#linear-data-structure-patterns)
- [Non-Linear Data Structure Patterns](#non-linear-data-structure-patterns)
- [Advanced Patterns](#advanced-patterns)
- [Practice Resources](#practice-resources)
- [Study Schedule](#study-schedule)

## 🎯 Overview

This guide covers **8 essential patterns** that form the foundation of most coding interview questions. These patterns are categorized into:

- **Linear Structures**: Arrays, linked lists, strings
- **Non-Linear Structures**: Trees, graphs
- **Advanced Topics**: Heaps, dynamic programming

**Key Philosophy**: Focus on understanding patterns rather than memorizing individual problems. Each pattern provides a template that can be applied to multiple similar problems.

## 🧠 Learning Strategy

1. **Understand the Pattern**: Learn the core concept and when to apply it
2. **Master the Template**: Memorize the basic code structure
3. **Practice Variations**: Solve 5-10 problems per pattern
4. **Time Complexity**: Always analyze and optimize
5. **Edge Cases**: Consider boundary conditions

## 📊 Pattern Categories

### Difficulty Progression
- **Beginner**: Two Pointers, Sliding Window
- **Intermediate**: Binary Search, BFS, DFS
- **Advanced**: Backtracking, Heaps, Dynamic Programming

### Time Investment
- **Week 1-2**: Linear patterns (Two Pointers, Sliding Window, Binary Search)
- **Week 3-4**: Tree/Graph patterns (BFS, DFS, Backtracking)
- **Week 5-6**: Advanced patterns (Heaps, Dynamic Programming)

## 🔄 Linear Data Structure Patterns

### 1. Two Pointers Pattern
**Time Complexity**: O(n) | **Space Complexity**: O(1)

**When to Use**:
- Finding pairs in sorted arrays
- Detecting cycles in linked lists
- Palindrome checking
- Merging sorted arrays

**Two Approaches**:
1. **Same Direction**: Fast and slow pointers (cycle detection, finding middle)
2. **Opposite Directions**: Left and right pointers (pair sum, palindromes)

**Template**:
```python
def two_pointers_opposite(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        current_sum = arr[left] + arr[right]
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    return [-1, -1]
```

**Common Problems**: Two Sum, 3Sum, Container With Most Water, Valid Palindrome

---

### 2. Sliding Window Pattern
**Time Complexity**: O(n) | **Space Complexity**: O(k) where k is window size

**When to Use**:
- Finding substrings/subarrays with specific properties
- Maximum/minimum in fixed-size windows
- Problems involving contiguous elements

**Two Types**:
1. **Fixed Window**: Window size remains constant
2. **Variable Window**: Window expands/contracts based on conditions

**Template**:
```python
def sliding_window_variable(s, k):
    window_start = 0
    max_length = 0
    char_frequency = {}

    for window_end in range(len(s)):
        # Expand window
        right_char = s[window_end]
        char_frequency[right_char] = char_frequency.get(right_char, 0) + 1

        # Contract window if needed
        while len(char_frequency) > k:
            left_char = s[window_start]
            char_frequency[left_char] -= 1
            if char_frequency[left_char] == 0:
                del char_frequency[left_char]
            window_start += 1

        max_length = max(max_length, window_end - window_start + 1)

    return max_length
```

**Common Problems**: Longest Substring Without Repeating Characters, Minimum Window Substring, Maximum Sum Subarray

---

### 3. Binary Search Pattern
**Time Complexity**: O(log n) | **Space Complexity**: O(1)

**When to Use**:
- Searching in sorted arrays
- Finding boundaries (first/last occurrence)
- Search in rotated sorted arrays
- Finding peak elements

**Key Insight**: Works on any monotonic function, not just sorted arrays

**Template**:
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

**Common Problems**: Search Insert Position, Find First and Last Position, Search in Rotated Sorted Array

## 🌳 Non-Linear Data Structure Patterns

### 4. Breadth-First Search (BFS)
**Time Complexity**: O(V + E) | **Space Complexity**: O(V)

**When to Use**:
- Level-order traversal
- Shortest path in unweighted graphs
- Finding minimum steps/levels

**Template**:
```python
from collections import deque

def bfs(root):
    if not root:
        return []

    result = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        current_level = []

        for _ in range(level_size):
            node = queue.popleft()
            current_level.append(node.val)

            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

        result.append(current_level)

    return result
```

**Common Problems**: Binary Tree Level Order Traversal, Minimum Depth of Binary Tree, Word Ladder

---

### 5. Depth-First Search (DFS)
**Time Complexity**: O(V + E) | **Space Complexity**: O(V)

**When to Use**:
- Tree/graph traversal
- Finding all paths
- Detecting cycles
- Topological sorting

**Three Types**:
1. **Preorder**: Process node before children
2. **Inorder**: Process left, node, right (for BST)
3. **Postorder**: Process children before node

**Template**:
```python
def dfs_recursive(root):
    if not root:
        return

    # Process current node (preorder)
    print(root.val)

    # Recurse on children
    dfs_recursive(root.left)
    dfs_recursive(root.right)

def dfs_iterative(root):
    if not root:
        return

    stack = [root]
    while stack:
        node = stack.pop()
        print(node.val)

        # Add children (right first for left-to-right processing)
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
```

**Common Problems**: Maximum Depth of Binary Tree, Path Sum, Number of Islands

---

### 6. Backtracking Pattern
**Time Complexity**: O(N!) in worst case | **Space Complexity**: O(N)

**When to Use**:
- Finding all possible solutions
- Combinatorial problems
- Constraint satisfaction problems

**Template**:
```python
def backtrack(path, choices):
    # Base case
    if is_valid_solution(path):
        result.append(path[:])  # Make a copy
        return

    # Try each choice
    for choice in choices:
        # Make choice
        path.append(choice)

        # Recurse with updated choices
        backtrack(path, get_next_choices(choice))

        # Undo choice (backtrack)
        path.pop()
```

**Common Problems**: Permutations, Combinations, N-Queens, Sudoku Solver

## 🚀 Advanced Patterns

### 7. Heaps (Priority Queue)
**Time Complexity**: O(log n) for insert/delete | **Space Complexity**: O(n)

**When to Use**:
- Top K problems
- Finding Kth smallest/largest
- Merge K sorted lists
- Median finding

**Types**:
- **Min Heap**: Smallest element at root
- **Max Heap**: Largest element at root

**Template**:
```python
import heapq

def find_k_largest(nums, k):
    # Use min heap of size k
    min_heap = []

    for num in nums:
        heapq.heappush(min_heap, num)
        if len(min_heap) > k:
            heapq.heappop(min_heap)

    return list(min_heap)
```

**Common Problems**: Kth Largest Element, Top K Frequent Elements, Merge K Sorted Lists

---

### 8. Dynamic Programming (DP)
**Time Complexity**: O(n²) typically | **Space Complexity**: O(n) or O(n²)

**When to Use**:
- Optimization problems (min/max)
- Counting problems
- Decision problems with overlapping subproblems

**Two Approaches**:
1. **Top-Down**: Recursion + Memoization
2. **Bottom-Up**: Iterative table filling

**Template**:
```python
# Top-down with memoization
def dp_top_down(n, memo={}):
    if n in memo:
        return memo[n]

    if n <= 1:
        return n

    memo[n] = dp_top_down(n-1, memo) + dp_top_down(n-2, memo)
    return memo[n]

# Bottom-up iterative
def dp_bottom_up(n):
    if n <= 1:
        return n

    dp = [0] * (n + 1)
    dp[1] = 1

    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]

    return dp[n]
```

**Common Problems**: Fibonacci, Coin Change, Longest Common Subsequence, House Robber

## 📚 Practice Resources

### Online Platforms
- **LeetCode**: Pattern-based problem sets
- **HackerRank**: Structured learning paths
- **CodeSignal**: Interview practice
- **Pramp**: Mock interviews

### Recommended Problem Sets
- **Two Pointers**: 15 problems
- **Sliding Window**: 12 problems
- **Binary Search**: 10 problems
- **BFS/DFS**: 20 problems
- **Backtracking**: 8 problems
- **Heaps**: 10 problems
- **Dynamic Programming**: 25 problems

## 📅 Study Schedule

### Week 1-2: Linear Patterns
- **Days 1-3**: Two Pointers (5 problems/day)
- **Days 4-7**: Sliding Window (4 problems/day)
- **Days 8-10**: Binary Search (3 problems/day)
- **Days 11-14**: Review and mixed practice

### Week 3-4: Tree/Graph Patterns
- **Days 1-5**: BFS (4 problems/day)
- **Days 6-10**: DFS (4 problems/day)
- **Days 11-14**: Backtracking (2 problems/day)

### Week 5-6: Advanced Patterns
- **Days 1-7**: Heaps (2 problems/day)
- **Days 8-14**: Dynamic Programming (3 problems/day)

### Daily Practice Tips
1. **Morning**: Learn new pattern (30 min)
2. **Afternoon**: Solve 2-3 problems (60 min)
3. **Evening**: Review and optimize solutions (30 min)

---

**Next Steps**: Start with [Two Pointers Pattern](./DSA/patterns/two_pointers.md) or explore [Practice Problems](./DSA/practice/)